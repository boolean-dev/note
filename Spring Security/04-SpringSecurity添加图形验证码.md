# SpringSecurity添加图形验证码

## 1. 前言

前面已经使用了用户名密码登录，这里再添加一个功能，用户名密码登录的时候，添加一个图形验证码的验证

## 2. 编写思路

图形验证码的功能相信大家都知道原理，大概就是把验证码信息存入`session`,用户在登录验证之前，首先验证`session`中的验证码是否和用户传入的一样，如果一样，则进行用户名密码验证。这是在常规的web项目中采用的做法，同理，在`SpringSecurity`中，也是一样，就是在`Spring`执行帐号密码验证之前，执行验证码拦截器，然后再将我们自己写的拦截器的结果，交给`SpringSecury`去处理

## 3. 准备工作

### 3.1 ImageCode实体类

```java
/**
 * @ClassName ImageCode
 * @Descriiption 图形验证码
 * @Author yanjiantao
 * @Date 2019/7/11 17:17
 **/
@Data
public class ImageCode {
    /**
     * 图片
     */
    private BufferedImage image;

    /**
     * 验证码
     */
    private String code;

    /**
     * 过期时间
     */
    private LocalDateTime expireTime;

    public ImageCode(BufferedImage image, String code, Integer expireIn) {
        this.image = image;
        this.code = code;
        this.expireTime = LocalDateTime.now().plusMinutes(expireIn);
    }
}
```

### 3.2 验证码生成工具类ImageCodeUtils

```java
/**
 * @ClassName ImageCodeUtils
 * @Descriiption 图形验证码工具类
 * @Author yanjiantao
 * @Date 2019/7/11 17:21
 **/
public class ImageCodeUtils {

    public static ImageCode generate(ServletWebRequest request) {
        int width = 67;
        int height = 23;
        BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);

        Graphics g = image.getGraphics();

        Random random = new Random();

        g.setColor(getRandColor(200, 250));
        g.fillRect(0, 0, width, height);
        g.setFont(new Font("Times New Roman", Font.ITALIC, 20));
        g.setColor(getRandColor(160, 200));
        for (int i = 0; i < 155; i++) {
            int x = random.nextInt(width);
            int y = random.nextInt(height);
            int xl = random.nextInt(12);
            int yl = random.nextInt(12);
            g.drawLine(x, y, x + xl, y + yl);
        }

        String sRand = "";
        for (int i = 0; i < 4; i++) {
            String rand = String.valueOf(random.nextInt(10));
            sRand += rand;
            g.setColor(new Color(20 + random.nextInt(110), 20 + random.nextInt(110), 20 + random.nextInt(110)));
            g.drawString(rand, 13 * i + 6, 16);
        }

        g.dispose();

        return new ImageCode(image, sRand, 60);
    }

    /**
     * 生成随机背景条纹
     *
     * @param fc
     * @param bc
     * @return
     */
    private static Color getRandColor(int fc, int bc) {
        Random random = new Random();
        if (fc > 255) {
            fc = 255;
        }
        if (bc > 255) {
            bc = 255;
        }
        int r = fc + random.nextInt(bc - fc);
        int g = fc + random.nextInt(bc - fc);
        int b = fc + random.nextInt(bc - fc);
        return new Color(r, g, b);
    }
}
```

### 3.3 获取验证码接口

```java
@GetMapping("/user/login/code/image")
    @ResponseBody
    public void codeImage(HttpServletRequest request, HttpServletResponse response) throws IOException {
        ServletWebRequest webRequest = new ServletWebRequest(request);
        ImageCode imageCode = ImageCodeUtils.generate(webRequest);
        sessionStrategy.setAttribute(webRequest, SESSION_KEY, imageCode);
        ImageIO.write(imageCode.getImage(), "JPEG", response.getOutputStream());
    }
```

### 3.4 登录界面signIn.ftl

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>登录</title>
</head>
<body>
<h2>浏览器登录页</h2>
<h3>表单登录</h3>
<form action="/user/login" method="post">
    <table>
        <tr>
            <td>用户名:</td>
            <td><input type="text" name="username"></td>
        </tr>
        <tr>
            <td>密码:</td>
            <td><input type="password" name="password"></td>
        </tr>
        <tr>
            <td>图形验证码:</td>
            <td>
                <input type="text" name="imageCode">
                <img src="/user/login/code/image">
            </td>
        </tr>
        <tr>
            <td colspan="2"><button type="submit">登录</button></td>
        </tr>
    </table>
</form>
</body>
</html>
```

## 4. 核心代码

### 4.1 验证码拦截器ValidateCodeFilter

```java
/**
 * @ClassName ValidateCodeFilter
 * @Descriiption 验证码拦截器
 * @Author yanjiantao
 * @Date 2019/7/11 18:07
 **/
@Data
public class ValidateCodeFilter extends OncePerRequestFilter {

    // 验证失败handler
    private AuthenticationFailureHandler authenticationFailureHandler;

    // session
    private SessionStrategy sessionStrategy = new HttpSessionSessionStrategy();

    /**
     * 验证码拦截器
     * @param request   request
     * @param response  respon
     * @param filterChain   filter拦截
     * @throws ServletException ServletException
     * @throws IOException  IOException
     */
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {

        if (StringUtils.equals("/user/login", request.getRequestURI()) &&
                StringUtils.equalsIgnoreCase("POST", request.getMethod())) {
            try {
                this.validate(new ServletWebRequest(request));
            } catch (ValidateImageCodeException e) {
                authenticationFailureHandler.onAuthenticationFailure(request, response, e);
                return;
            }
        }
        filterChain.doFilter(request, response);
    }

    /**
     * 校验图形验证码
     * @param servletWebRequest request
     * @throws ServletRequestBindingException   exception
     */
    private void validate(ServletWebRequest servletWebRequest) throws ServletRequestBindingException {

        // 从session中得到imageCode
        ImageCode imageCodeSession = (ImageCode) sessionStrategy.getAttribute(servletWebRequest, SESSION_KEY);

        // 从request中得到验证码
        String imageCode = ServletRequestUtils.getStringParameter(servletWebRequest.getRequest(), "imageCode");

        if (StringUtils.isEmpty(imageCode)) {
            throw new ValidateImageCodeException("图形验证码异常");
        }

        if (LocalDateTime.now().isAfter(imageCodeSession.getExpireTime())) {
            sessionStrategy.removeAttribute(servletWebRequest, SESSION_KEY);
            throw new ValidateImageCodeException("图形验证码已过期");
        }

        if (!StringUtils.equalsIgnoreCase(imageCodeSession.getCode(), imageCode)) {
            throw new ValidateImageCodeException("验证码异常");
        }

        sessionStrategy.removeAttribute(servletWebRequest, SESSION_KEY);
    }


}
```

### 4.2 验证失败的异常

```java
/**
 * @ClassName ValidateImageCodeException
 * @Descriiption 图形验证码验证异常
 * @Author yanjiantao
 * @Date 2019/7/12 14:10
 **/
public class ValidateImageCodeException extends AuthenticationException {
    public ValidateImageCodeException(String msg, Throwable t) {
        super(msg, t);
    }

    public ValidateImageCodeException(String msg) {
        super(msg);
    }
}
```



### 4.3 自定义失败处理器

```java
/**
 * @ClassName MyAuthenctiationFailureHandler
 * @Descriiption 自定义登录失败处理器
 * @Author yanjiantao
 * @Date 2019/7/10 17:26
 **/
@Slf4j
@Component("myAuthenctiationFailureHandler")
public class MyAuthenctiationFailureHandler extends SimpleUrlAuthenticationFailureHandler {

    @Autowired
    private SecurityProperties securityProperties;

    @Override
    public void onAuthenticationFailure(HttpServletRequest request,
                                        HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        logger.info("登录失败...\ne={}", exception);
        Gson gson = new Gson();
        if ("JSON".equalsIgnoreCase(securityProperties.getBrowser().getLoginType())) {
            response.setStatus(HttpStatus.OK.value());
            response.setContentType("application/json;charset=UTF-8");
            response.getWriter().write(gson.toJson(Result.errorMsg()));
        }else {
            super.onAuthenticationFailure(request, response, exception);
        }
    }
}
```

我这里没怎么写，主要写的是`json`返回的处理类，如果是`html`的话，就直接调用父类的处理器，如果你有一些特殊的需求的话，是可以直接在自定义的

### 4.4 SpringSecurity的配置

```java
/**
 * @ClassName WebSecurityConfig
 * @Descriiption Spring Security配置类
 * @Author yanjiantao
 * @Date 2019/7/8 10:45
 **/
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private SecurityProperties securityProperties;

    @Autowired
    private MyAuthenctiationFailureHandler myAuthenctiationFailureHandler;

    @Autowired
    private MyAuthenticationSuccessHandler myAuthenticationSuccessHandler;


    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        // 图形验证码拦截器
        ValidateCodeFilter validateCodeFilter = new ValidateCodeFilter();
        validateCodeFilter.setAuthenticationFailureHandler(myAuthenctiationFailureHandler);

        // 图形验证码拦截器执行
        http.addFilterBefore(validateCodeFilter, UsernamePasswordAuthenticationFilter.class).
                formLogin()
                .loginPage(securityProperties.getBrowser().getLoginPage())
                .loginProcessingUrl("/user/login")
                .successHandler(myAuthenticationSuccessHandler)
                .failureHandler(myAuthenctiationFailureHandler)
                .and()
                .authorizeRequests()
                .antMatchers(securityProperties.getBrowser().getLoginPage(), "/user/login/code/image")
                .permitAll()
                .anyRequest()
                .authenticated()
                .and()
                .csrf().disable();
    }

}
```

这里面有些配置是不需要的，我后期可能会进行一些重构，所以，关键代码只是这一行`        http.addFilterBefore(validateCodeFilter, UsernamePasswordAuthenticationFilter.class)`，将自定义处理器在`UsernamePasswordAuthenticationFilter`之前调用

## 5 结语

以上就是`SpringSecurity`图形验证码的实现，还有很大的改进空间，例如现在都是微服务，session可能用得比较少，所以可以把验证码存储在redis，或者分布式session。代码有些乱，后期我会再进行重构，并且添加手机号登录的验证，不过思路大概相同。[代码地址](https://github.com/boolean-dev/security/tree/v04-07)