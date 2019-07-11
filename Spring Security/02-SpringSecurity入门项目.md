# 02-SpringSecurity入门

## 1. 前言

本篇博客主要介绍SpringSecurity初始化demo

## 2. 编写代码

### 2.1 引入pom

```xml
<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-freemarker</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
```



### 2.1 编写WebSecurityConfig

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

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

### 2.2 编写SecurityController测试Controller

```java
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

### 2.3 编写index.ftl主页

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>index</title>
</head>
<body>
    <h1>index</h1>
</body>
</html>
```

## 3 测试

启动项目，可以看见SpringSecurity默认的密码，

![Snipaste20190710115841.png](https://img.hacpai.com/file/2019/07/Snipaste20190710115841-ce1749af.png)

使用默认的用户`user`登录

![Snipaste20190710122425.png](https://img.hacpai.com/file/2019/07/Snipaste20190710122425-7daefeb8.png)

登录后可看到主页

![Snipaste20190710122527.png](https://img.hacpai.com/file/2019/07/Snipaste20190710122527-15bdcd58.png)