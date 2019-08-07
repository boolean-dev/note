# SpringSecurity添加短信验证码

## 1. 前言

在之前的博客中，我们已经使用了图形验证码登录，大概的思路就是在用户名登录之前，添加一个拦截器，然后将验证码与我们存在session中的验证码进行对比，如果相同，则放行，进行用户名密码登录，如果不相同，则拦截，返回错误

这一节我讲介绍使用手机验证码方式登录

## 2. 整体思路

图形验证码其实只是在用户名密码登录之前，验证验证码是否正确错误，底层还是`UsernamePasswordAuthenticationFilter`，而短信验证码完全不同，它是在执行完短信验证`ValidateSmsCodeFilter`过滤器之后，自行构建一个用户`SmsCodeAuthenticationToken`，并且标记该用户已进行认证，因此，我们需要重写`SpringSecurity`的认证过程,重写的类包括过滤器`AbstractAuthenticationProcessingFilter`（和用户名密码登录的`UsernamePasswordAuthenticationFilter`作用类似），用户信息`AbstractAuthenticationToken`（和用户名密码登录中的`UsernamePasswordAuthenticationToken`作用类似）,重写`AuthenticationProvider`，（和用户名密码登录的`DaoAuthenticationProvider`类似）。重写完之后，如果是手机号验证码登录的话，就会走我们这个流程。接下来开始coding

## 3. 核心代码编写

### 3.1 ValidateSmsCodeFilter（验证码拦截器）

在请求进入`SpringSecurity`之前，将会执行这个拦截器，进行手机验证码的验证



```java
/**
 * @ClassName ValidateSmsCodeFilter
 * @Descriiption 手机验证码拦截器
 * @Author yanjiantao
 * @Date 2019/7/12 16:39
 **/
@Slf4j
public class ValidateSmsCodeFilter extends AbstractAuthenticationProcessingFilter {

    private SessionStrategy sessionStrategy = new HttpSessionSessionStrategy();


    public ValidateSmsCodeFilter() {
        super(new AntPathRequestMatcher("/authentication/phone", "POST"));
    }


    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {
        if(!StringUtils.equalsIgnoreCase("post", request.getMethod())){
            throw new AuthenticationServiceException("登录请求只支持POST方法");
        }

        String phone = ServletRequestUtils.getStringParameter(request, "phone");

        if(StringUtils.isBlank(phone)){
            throw new AuthenticationServiceException("手机号不能为空");
        }

        String code = ServletRequestUtils.getStringParameter(request, "smsCode");
        SmsCode smsCode = (SmsCode) sessionStrategy.getAttribute(new ServletWebRequest(request), SESSION_SMS_KEY);

        if (smsCode == null) {
            throw new AuthenticationServiceException("验证码不存在");
        }

        if (LocalDateTime.now().isAfter(smsCode.getExpireTime())) {
            throw new AuthenticationServiceException("验证码已过期");
        }

        if (!StringUtils.equalsIgnoreCase(code, smsCode.getCode())) {
            throw new AuthenticationServiceException("验证码错误");
        }

        SmsCodeAuthenticationToken authRequest = new SmsCodeAuthenticationToken(phone);

        authRequest.setDetails(authenticationDetailsSource.buildDetails(request));

        return getAuthenticationManager().authenticate(authRequest);
    }


}
```

### 3.2 SmsCodeAuthenticationToken（用户信息）

```java
/**
 * @ClassName SmsCodeAuthenticationToken
 * @Descriiption sms登录信息
 * @Author yanjiantao
 * @Date 2019/7/12 17:40
 **/
public class SmsCodeAuthenticationToken extends UsernamePasswordAuthenticationToken {

    public SmsCodeAuthenticationToken(Object principal) {
        super(principal, null);
    }
    public SmsCodeAuthenticationToken(Object principal, Object credentials) {
        super(principal, credentials);
    }

    public SmsCodeAuthenticationToken(Object principal, Object credentials, Collection<? extends GrantedAuthority> authorities) {
        super(principal, credentials, authorities);
    }
}
```

### 3.3 SmsCodeFilter(验证码登录拦截器)

这一步和前面那个拦截器功能不同，这个拦截器主要的功能是生成一个用户，并且将其设置为已登入用户

```java
/**
 * @ClassName SmsCodeFilter
 * @Descriiption 验证码过滤器
 * @Author yanjiantao
 * @Date 2019/7/12 18:00
 **/
@Slf4j
public class SmsCodeFilter extends AbstractAuthenticationProcessingFilter {

    public SmsCodeFilter() {
        super(new AntPathRequestMatcher("/authentication/phone", "POST"));
    }


    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {

        if (!StringUtils.equalsIgnoreCase("POST", request.getMethod())) {
            throw new ValidateImageCodeException("请求方法必须为POST");
        }

        String phone = ServletRequestUtils.getRequiredStringParameter(request, "phone");

        if (StringUtils.isEmpty(phone)) {
            throw new ValidateImageCodeException("电话号为空");
        }

        SmsCodeAuthenticationToken smsToken = new SmsCodeAuthenticationToken(phone);
        smsToken.setDetails(authenticationDetailsSource.buildDetails(request));
        return getAuthenticationManager().authenticate(smsToken);
    }
}
```



### 3.4 SmsCodeAuthenticationProvider

```java
/**
 * @ClassName SmsCodeAuthenticationProvider
 * @Descriiption 验证码Provider
 * @Author yanjiantao
 * @Date 2019/7/12 17:52
 **/
@Data
public class SmsCodeAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider {

    private UserDetailsService userDetailsService;

    private SessionStrategy sessionStrategy = new HttpSessionSessionStrategy();


    @Override
    public boolean supports(Class<?> authentication) {
        return  SmsCodeAuthenticationToken.class.isAssignableFrom(authentication);
    }

    @Override
    protected void additionalAuthenticationChecks(UserDetails userDetails, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {

    }

    @Override
    protected UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
        UserDetails user = userDetailsService.loadUserByUsername(username);

        if(user == null) {
            throw new InternalAuthenticationServiceException("未获取到用户信息");
        }

        return user;
    }
}
```

## 4. 配置SpringSecurity

### 4.1 手机验证码配置

```java
@Component
public class TempConfig  extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity> {

    @Autowired
    private UserDetailsService userDetailsService;

    @Autowired
    private AuthenticationSuccessHandler imoocAuthenticationSuccessHandler;

    @Autowired
    private AuthenticationFailureHandler imoocAuthenticationFailureHandler;

    @Override
    public void configure(HttpSecurity http) throws Exception {

        ValidateSmsCodeFilter smsCodeAuthenticationFilter = new ValidateSmsCodeFilter();
        smsCodeAuthenticationFilter.setAuthenticationManager(http.getSharedObject(AuthenticationManager.class));
        smsCodeAuthenticationFilter.setAuthenticationSuccessHandler(imoocAuthenticationSuccessHandler);
        smsCodeAuthenticationFilter.setAuthenticationFailureHandler(imoocAuthenticationFailureHandler);

        SmsCodeAuthenticationProvider authenticationProvider = new SmsCodeAuthenticationProvider();
        authenticationProvider.setUserDetailsService(userDetailsService);

        http.authenticationProvider(authenticationProvider)
                .addFilterAfter(smsCodeAuthenticationFilter, AbstractPreAuthenticatedProcessingFilter.class);

    }
}
```

### 4.2 SpringSecurity配置类

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

    @Autowired
    private TempConfig tempConfig;


    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        // 图形验证码拦截器
        ValidateCodeFilter validateCodeFilter = new ValidateCodeFilter();
        validateCodeFilter.setAuthenticationFailureHandler(myAuthenctiationFailureHandler);
//
        SmsCodeFilter smsCodeFilter = new SmsCodeFilter();
        smsCodeFilter.setAuthenticationFailureHandler(myAuthenctiationFailureHandler);


        http.addFilterBefore(validateCodeFilter, UsernamePasswordAuthenticationFilter.class)
            .addFilterBefore(validateCodeFilter, UsernamePasswordAuthenticationFilter.class)
            .formLogin()
            .loginPage(securityProperties.getBrowser().getLoginPage())
            .loginProcessingUrl("/user/login")
            .successHandler(myAuthenticationSuccessHandler)
            .failureHandler(myAuthenctiationFailureHandler)
                .and()
                .apply(tempConfig)
            .and()
            .authorizeRequests()
            .antMatchers(securityProperties.getBrowser().getLoginPage(), "/user/login/code/image", "/user/login/code/sms","/authentication/phone")
            .permitAll()
            .anyRequest()
            .authenticated()
            .and()
            .csrf().disable();
    }

}
```

## 5. 其余代码

### 5.1 发送短信验证码接口

```java
    @GetMapping("/user/login/code/sms")
    @ResponseBody
    public void codeSms(String phone, HttpServletRequest request) throws IOException {

        ServletWebRequest webRequest = new ServletWebRequest(request);

        String code = RandomStringUtils.randomNumeric(4);
        SmsCode smsCode = new SmsCode(phone, code, 30);
        log.info("发送短信，phone={},code={}", phone, code);
        sessionStrategy.setAttribute(webRequest, SESSION_SMS_KEY, smsCode);
    }
```

### 5.2 前端界面

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


<h3>短信登录</h3>
<form action="/authentication/phone" method="post">
    <table>
        <tr>
            <td>手机号:</td>
            <td><input type="text" name="phone" value="1857911****"></td>
        </tr>
        <tr>
            <td>短信验证码:</td>
            <td>
                <input type="text" name="smsCode">
                <a href="/user/login/code/sms?phone=1857911****">发送验证码</a>
            </td>
        </tr>
        <tr>
            <td colspan="2"><button type="submit">登录</button></td>
        </tr>
    </table>
</form>
</body>

```

## 6. 演示

### 6.1 登录界面

![image.png](https://img.hacpai.com/file/2019/07/image-49b7bc96.png)

### 6.2 发送验证码

![image.png](https://img.hacpai.com/file/2019/07/image-23d9d85d.png)

### 6.3 登录成功

![image.png](https://img.hacpai.com/file/2019/07/image-0ed6dbf0.png)

## 7. 代码地址

[SpringSecurity分支v04-08](https://github.com/boolean-dev/security/tree/v04-08)

