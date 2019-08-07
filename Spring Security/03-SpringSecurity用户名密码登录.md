# SpringSecurity用户名密码登录

## 1. 前言

前面我已经介绍了如何使用SpringSecuriry进行用户登录，只写了几行代码，就可以登录认证了，但是，在我们项目中，这样是完全无法使用的，接下来，我就介绍，如何连接数据库，来进行用户名和密码的验证

## 2. 准备工作

### 2.1 建立用户数据库

数据库结构很简单，只有用户名和密码

![image.png](https://img.hacpai.com/file/2019/07/image-efaabb28.png)

建表脚本

```sh
CREATE TABLE `tb_user` (
	`id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
	`username` VARCHAR(50) NOT NULL DEFAULT '0' COMMENT '用户名',
	`password` VARCHAR(100) NOT NULL DEFAULT '0' COMMENT '密码',
	PRIMARY KEY (`id`),
	UNIQUE INDEX `username` (`username`)
)
COLLATE='utf8_general_ci'
ENGINE=InnoDB
AUTO_INCREMENT=3
;

```

### 2.2 添加包依赖

之前已经添加过`security`的包了，这里只添加`mybatis`的包

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-freemarker</artifactId>
		</dependency>
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>2.0.0</version>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
```

### 2.3 编写用户的查询接口

service

```java
/**
 * @ClassName UserService
 * @Descriiption UserService
 * @Author yanjiantao
 * @Date 2019/7/10 14:02
 **/
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    public User findByUserName(String username) {
        return userMapper.findByUserName(username);
    }
}
```

mapper

```java
/**
 * @ClassName User
 * @Descriiption 用户
 * @Author yanjiantao
 * @Date 2019/7/10 13:56
 **/
@Mapper
public interface UserMapper {

    /**
     * 通过用户名查找用户
     * @param username  username
     * @return  user
     */
    @Select("select * from tb_user where username = #{username}")
    User findByUserName(String username);
}

```

yml配置文件

```yaml
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/security?useUnicode=true&characterEncoding=utf8&characterSetResults=utf8&serverTimezone=GMT%2B8
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
server:
  port: 8888
```

## 3. Security密码认证

### 3.1 自定义用户密码认证

如果需要自定义用户的认证，Spring Security提供了一个接口`UserDetailsService`，只需继承这个接口，然后实现`loadUserByUsername(String username)`方法即可，`Spring`便会采用我们自定义的认证方法

```java
/**
 * @ClassName MyUserDetailsService
 * @Descriiption 自定义密码校验
 * @Author yanjiantao
 * @Date 2019/7/10 13:45
 **/
@Slf4j
@Component
public class MyUserDetailsService implements UserDetailsService {

    @Autowired
    private UserService userService;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        log.info("--------->登录用户名为:username={}", username);
        User user = userService.findByUserName(username);
        String password = passwordEncoder.encode(user.getPassword());
        return new org.springframework.security.core.userdetails.User(username, user.getPassword(),
                true, true, true, true,
                AuthorityUtils.commaSeparatedStringToAuthorityList("admin"));
    }
}
```

这里采用的密码加密方法是`BCryptPasswordEncoder`，这个加密算法即使密码是`123456`，不同时间加密后也会不同，据说加密前会自动生成`salt`值，再把`salt`值和密码进行`hash`加密，加密后的密码串包含了`salt`和密码。之后解密再把salt值拿出来进行解密。详细解释见[链接](https://www.zhihu.com/question/54720851/answer/288322108)

### 3.2 配置授权登录

```java
/**
 * @ClassName WebSecurityConfig
 * @Descriiption Spring Security配置类
 * @Author yanjiantao
 * @Date 2019/7/8 10:45
 **/
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {


    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
                .and()
                .authorizeRequests()
                .anyRequest()
                .authenticated();
    }

}
```

### 3.3 配置controller

```java
/**
 * @ClassName SecurityController
 * @Descriiption 测试Controller
 * @Author yanjiantao
 * @Date 2019/7/8 10:18
 **/
@Controller
public class SecurityController {

    @ResponseBody
    @GetMapping("/user/{id}")
    public Object user(@PathVariable String id) {
        Map<String, Object> result = new HashMap<>();
        result.put("code", 200);
        result.put("msg", "success");
        return result;

    }

    @GetMapping("/index")
    public String index() {
        return "index";
    }
}

```

## 4. 结尾

没有写得特别复杂，所以就没有写用户的添加，所以用户加密后的密码可以使用

`BCryptPasswordEncoder`进行加密，然后再直接写入数据库。[代理地址](https://github.com/boolean-dev/security/tree/v04-03)