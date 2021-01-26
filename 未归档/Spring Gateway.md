**1. 为什么是Spring Cloud Gateway**

一句话，Spring Cloud已经放弃Netflix Zuul了。现在Spring Cloud中引用的还是Zuul 1.x版本，而这个版本是基于过滤器的，是阻塞IO，不支持长连接。Zuul 2.x版本跟1.x的架构大一样，性能也有所提升。既然Spring Cloud已经不再集成Zuul 2.x了，那么是时候了解一下Spring Cloud Gateway了。

![img](https://img2018.cnblogs.com/blog/874963/201907/874963-20190704132254746-702819017.png)

可以看到，最新的Spring Cloud中的Zuul还是1.3.1版本

而且，官网中也明确说了不再维护Zuul了

![img](https://img2018.cnblogs.com/blog/874963/201907/874963-20190704132403793-534668759.png)

（PS：顺便补充几个名词： 服务发现（Eureka），断路器（Hystrix），智能路由（Zuul），客户端负载均衡（Ribbon））

**2. API网关**

> API网关是一个服务器，是系统的唯一入口。从面向对象设计的角度看，它与外观模式类似。API网关封装了系统内部架构，为每个客户端提供一个定制的API。它可能还具有其它职责，如身份验证、监控、负载均衡、缓存、请求分片与管理、静态响应处理。API网关方式的核心要点是，所有的客户端和消费端都通过统一的网关接入微服务，在网关层处理所有的非业务功能。通常，网关也是提供REST/HTTP的访问API。

网关应当具备以下功能：

- 性能：API高可用，负载均衡，容错机制。
- 安全：权限身份认证、脱敏，流量清洗，后端签名（保证全链路可信调用）,黑名单（非法调用的限制）。
- 日志：日志记录（spainid,traceid）一旦涉及分布式，全链路跟踪必不可少。
- 缓存：数据缓存。
- 监控：记录请求响应数据，api耗时分析，性能监控。
- 限流：流量控制，错峰流控，可以定义多种限流规则。
- 灰度：线上灰度部署，可以减小风险。
- 路由：动态路由规则。

目前，比较流行的网关有：Nginx 、 Kong 、Orange等等，还有微服务网关Zuul 、Spring Cloud Gateway等等

对于 API Gateway，常见的选型有基于 Openresty 的 Kong、基于 Go 的 Tyk 和基于 Java 的 Zuul。这三个选型本身没有什么明显的区别，主要还是看技术栈是否能满足快速应用和二次开发。

![img](https://img2018.cnblogs.com/blog/874963/201907/874963-20190704125902757-1187231766.png)

![img](https://img2018.cnblogs.com/blog/874963/201907/874963-20190704125915409-48439275.png)

![img](https://img2018.cnblogs.com/blog/874963/201907/874963-20190704125930349-802179280.png)

以上说的这些功能，这些开源的网关组件都有，或者借助Lua也能实现，比如：Nginx + Lua

那要Spring Cloud Gateway还有什么用呢？

其实，我个人理解是这样的：

- 像Nginx这类网关，性能肯定是没得说，它适合做那种门户网关，是作为整个全局的网关，是对外的，处于最外层的；而Gateway这种，更像是业务网关，主要用来对应不同的客户端提供服务的，用于聚合业务的。各个微服务独立部署，职责单一，对外提供服务的时候需要有一个东西把业务聚合起来。
- 像Nginx这类网关，都是用不同的语言编写的，不易于扩展；而Gateway就不同，它是用Java写的，易于扩展和维护
- Gateway这类网关可以实现熔断、重试等功能，这是Nginx不具备的

所以，你看到的网关可能是这样的：

![img](https://img2018.cnblogs.com/blog/874963/201907/874963-20190704131304876-2104304055.png) 

**2.1. Netflix Zuul 1.x VS Netflix Zuul 2.x**

![img](https://img2018.cnblogs.com/blog/874963/201907/874963-20190704132559449-1231366972.png) ![img](https://img2018.cnblogs.com/blog/874963/201907/874963-20190704132610001-258462769.png)

**3. Spring Cloud Gateway**

3.1. 特性

- 基于Spring Framework 5、Project Reactor和Spring Boot 2.0构建
- 能够在任意请求属性上匹配路由
- predicates（谓词） 和 filters（过滤器）是特定于路由的
- 集成了Hystrix断路器
- 集成了Spring Cloud DiscoveryClient
- 易于编写谓词和过滤器
- 请求速率限制
- 路径重写

3.2. 术语

**Route** ： 路由是网关的基本组件。它由ID、目标URI、谓词集合和过滤器集合定义。如果聚合谓词为true，则匹配路由

**Predicate** ： This is a Java 8 Function Predicate

**Filter** ： 是GatewayFilter的一个实例，在这里，可以在发送下游请求之前或之后修改请求和响应

3.3. 原理

![img](https://img2018.cnblogs.com/blog/874963/201906/874963-20190627174309384-221370917.png)

（PS：看到这张图是不是很熟悉，没错，很像SpringMVC的请求处理过程）

客户端向Spring Cloud Gateway发出请求。如果Gateway Handler Mapping确定请求与路由匹配，则将其发送给Gateway Web Handler。这个Handler运行通过特定于请求的过滤器链发送请求。过滤器可以在发送代理请求之前或之后执行逻辑。执行所有的“pre”过滤逻辑，然后发出代理请求，最后执行“post”过滤逻辑。

3.4. Route Predicate Factories

- Spring Cloud Gateway 包含许多内置的 Route Predicate Factories
- 所有这些predicates用于匹配HTTP请求的不同属性
- 多个 Route Predicate Factories 可以通过逻辑与（and）结合起来一起使用

3.4.1. After Route Predicate Factory

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个路由匹配“美国丹佛时间2017-01-20 17:42”之后的任意请求

3.4.2. Header Route Predicate Factory

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个路由匹配“请求头包含X-Request-Id并且其值匹配正则表达式\d+”的任意请求

3.4.3. Method Route Predicate Factory

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个路由匹配任意GET请求

3.4.4. Path Route Predicate Factory

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Path=/foo/{segment},/bar/{segment}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个路由匹配这样路径的请求，比如：/foo/1 或 /foo/bar 或 /bar/baz

3.4.5. Query Route Predicate Factory

这个Predicate有两个参数：一个必须的参数名和一个可选的正则表达式

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=baz
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个路由匹配“查询参数中包含baz”的请求

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=foo, ba.
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个路由匹配“查询参数中包含foo，并且其参数值满足正则表达式ba.”的请求，比如：bar，baz

3.4.6. RemoteAddr Route Predicate Factory

这个路由接受一个IP（IPv4或IPv6）地址字符串。例如：192.168.0.1/16，其中192.168.0.1，16是子网掩码

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: https://example.org
        predicates:
        - RemoteAddr=192.168.1.1/24
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里路由匹配远程地址是这样的请求，例如：192.168.1.10

3.5. GatewayFilter Factories（网关过滤器）

路由过滤器允许以某种方式修改传入的HTTP请求或传出HTTP响应。路由过滤器的作用域是特定的路由。Spring Cloud Gateway包含许多内置的网关过滤器工厂。

3.5.1. AddRequestHeader GatewayFilter Factory

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        filters:
        - AddRequestHeader=X-Request-Foo, Bar
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

对于所有匹配的请求，将会给传给下游的请求添加一个请求头 X-Request-Foo:Bar

3.5.2. AddRequestParameter GatewayFilter Factory

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: https://example.org
        filters:
        - AddRequestParameter=foo, bar
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

对于所有匹配的请求，将给传给下游的请求添加一个查询参数 foo=bar

3.5.3. AddResponseHeader GatewayFilter Factory

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: add_response_header_route
        uri: https://example.org
        filters:
        - AddResponseHeader=X-Response-Foo, Bar
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

对于所有匹配的请求，添加一个响应头 X-Response-Foo:Bar

3.5.4. Hystrix GatewayFilter Factory

Hystrix网关过滤器允许你将断路器引入网关路由，保护你的服务免受级联失败的影响，并在下游发生故障时提供预备响应。

为了启用Hystrix网关过滤器，你需要引入 spring-cloud-starter-netflix-hystrix

Hystrix网关过滤器需要一个name参数，这个name是HystrixCommand的名字

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: hystrix_route
        uri: https://example.org
        filters:
        - Hystrix=myCommandName
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

给这个过滤器包装一个名字叫myCommandName的HystrixCommand

Hystrix网关过滤器也接受一个可选的参数fallbackUri，但是目前只支持forward:前缀的URL。也就是说，如果这个fallback被调用，请求将被重定向到匹配的这个URL。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: hystrix_route
        uri: lb://backing-service:8088
        predicates:
        - Path=/consumingserviceendpoint
        filters:
        - name: Hystrix
          args:
            name: fallbackcmd
            fallbackUri: forward:/incaseoffailureusethis
        - RewritePath=/consumingserviceendpoint, /backingserviceendpoint
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

当fallback被调用的时候，请求将被重定向到/incaseoffailureusethis

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: ingredients
        uri: lb://ingredients
        predicates:
        - Path=//ingredients/**
        filters:
        - name: Hystrix
          args:
            name: fetchIngredients
            fallbackUri: forward:/fallback
      - id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在这个例子中，专门定义了一个端点来处理/fallback请求，它在localhost:9994上。也就是说，当fallback被调用的时候将重定向到http://localhost:9994/fallback

3.5.5. PrefixPath GatewayFilter Factory

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - PrefixPath=/mypath
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

所有匹配的请求都将加上前缀/mypath。例如，如果请求是/hello，那么经过这个过滤器后，发出去的请求变成/mypath/hello

3.5.6. RequestRateLimiter GatewayFilter Factory

RequestRateLimiter网关过滤器使用一个RateLimiter实现来决定是否当前请求可以继续往下走。如果不能，默认将返回HTTP 429 - Too Many Requests

这个过滤器接受一个可选的参数keyResolver，这个参数是一个特定的rate limiter

keyResolver是实现了KeyResolver接口的一个Bean。

在配置的时候，使用SpEL按名称引用Bean。#{@myKeyResolver}是一个SpEL表达式，表示引用名字叫myKeyResolver的Bean。

**KeyResolver.java** 

```
1 public interface KeyResolver {
2 	Mono<String> resolve(ServerWebExchange exchange);
3 }
```

KeyResolver默认的实现是PrincipalNameKeyResolver，它从ServerWebExchange中检索Principal，并调用Principal.getName()方法。 

默认情况下，如果KeyResolver没有找到一个key，那么请求将会被denied（译：否认，拒绝）。这种行为可以通过spring.cloud.gateway.filter.request-rate-limiter.deny-empty-key (true or false) 和 spring.cloud.gateway.filter.request-rate-limiter.empty-key-status-code 属性来进行调整. 

**Redis RateLimiter**

需要引用 spring-boot-starter-data-redis-reactive

这个逻辑使用令牌桶算法

- redis-rate-limiter.replenishRate ： 允许用户每秒处理多少个请求。这是令牌桶被填充的速率。
- redis-rate-limiter.burstCapacity ： 用户在一秒钟内允许执行的最大请求数。这是令牌桶可以容纳的令牌数量。将此值设置为0将阻塞所有请求。

一个稳定的速率是通过将replenishRate 和 burstCapacity设为相同的值来实现的。也可以将burstCapacity设得比replenishRate大，以应对临时爆发的流量。在这种情况下，需要允许速率限制器在突发事件之间间隔一段时间，因为连续两次突发事件将导致丢弃请求（HTTP 429 - Too Many Requests）

**application.yml**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            redis-rate-limiter.replenishRate: 10
            redis-rate-limiter.burstCapacity: 20
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**Config.java**

```
1 @Bean
2 KeyResolver userKeyResolver() {
3     return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
4 }
```

这里定义了每个用户的请求速率限制为10。允许使用20个请求，但是在接下来的一秒中，只有10个请求可用。

这个例子中只是简单地从请求参数中获取"user"，在实际生产环境中不建议这么做。

我们也可以通过实现RateLimiter接口来自定义，这个时候，在配置中我们就需要引用这个Bean，例如：#{@myRateLimiter} 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            rate-limiter: "#{@myRateLimiter}"
            key-resolver: "#{@userKeyResolver}"
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

3.5.7. Default Filters

如果你想要添加一个过滤器并且把它应用于所有路由的话，你可以用spring.cloud.gateway.default-filters。这个属性接受一个过滤器列表。

```
spring:
  cloud:
    gateway:
      default-filters:
      - AddResponseHeader=X-Response-Default-Foo, Default-Bar
      - PrefixPath=/httpbin
```

3.6. Global Filters（全局过滤器）

GlobalFilter接口的方法签名和GatewayFilter相同。这些是有条件地应用于所有路由的特殊过滤器。

3.6.1. GlobalFilter和GatewayFilter的顺序

当一个请求过来的时候，将会添加所有的GatewayFilter实例和所有特定的GatewayFilter实例到过滤器链上。过滤器链按照org.springframework.core.Ordered接口对该链路上的过滤器进行排序。你可以通过实现接口中的getOrder()方法或者使用@Order注解。

Spring Cloud Gateway将过滤器执行逻辑分为“pre”和“post”阶段。优先级最高的过滤器将会是“pre”阶段中的第一个过滤器，同时它也将是“post”阶段中的最后一个过滤器。

**ExampleConfiguration.java** 

```
 1 @Bean
 2 @Order(-1)
 3 public GlobalFilter a() {
 4     return (exchange, chain) -> {
 5         log.info("first pre filter");
 6         return chain.filter(exchange).then(Mono.fromRunnable(() -> {
 7             log.info("third post filter");
 8         }));
 9     };
10 }
11 
12 @Bean
13 @Order(0)
14 public GlobalFilter b() {
15     return (exchange, chain) -> {
16         log.info("second pre filter");
17         return chain.filter(exchange).then(Mono.fromRunnable(() -> {
18             log.info("second post filter");
19         }));
20     };
21 }
22 
23 @Bean
24 @Order(1)
25 public GlobalFilter c() {
26     return (exchange, chain) -> {
27         log.info("third pre filter");
28         return chain.filter(exchange).then(Mono.fromRunnable(() -> {
29             log.info("first post filter");
30         }));
31     };
32 }
```

3.6.2. LoadBalancerClient Filter

LoadBalancerClientFilter查找exchange属性中查找ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR一个URI。如果url符合lb schema（例如：lb://myservice），那么它将使用Spring Cloud LoadBalancerClient 来解析这个名字到一个实际的主机和端口，并替换URI中相同的属性。原始url中未被修改的部分被附加到ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR属性列表中。

**application.yml**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: myRoute
        uri: lb://service
        predicates:
        - Path=/service/**
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

默认情况下，当一个服务实例在LoadBalancer中没有找到时，将返回503。你可以通过配置spring.cloud.gateway.loadbalancer.use404=true来让它返回404。

3.7. 配置

**RouteDefinitionLocator.java** 

```
1 public interface RouteDefinitionLocator {
2 	Flux<RouteDefinition> getRouteDefinitions();
3 }
```

默认情况下，PropertiesRouteDefinitionLocator通过@ConfigurationProperties机制加载属性

下面两段配置是等价的

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      routes:
      - id: setstatus_route
        uri: https://example.org
        filters:
        - name: SetStatus
          args:
            status: 401
      - id: setstatusshortcut_route
        uri: https://example.org
        filters:
        - SetStatus=401
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

下面用Java配置

**GatewaySampleApplication.java** 

```
 1 // static imports from GatewayFilters and RoutePredicates
 2 @Bean
 3 public RouteLocator customRouteLocator(RouteLocatorBuilder builder, ThrottleGatewayFilterFactory throttle) {
 4     return builder.routes()
 5             .route(r -> r.host("**.abc.org").and().path("/image/png")
 6                 .filters(f ->
 7                         f.addResponseHeader("X-TestHeader", "foobar"))
 8                 .uri("http://httpbin.org:80")
 9             )
10             .route(r -> r.path("/image/webp")
11                 .filters(f ->
12                         f.addResponseHeader("X-AnotherHeader", "baz"))
13                 .uri("http://httpbin.org:80")
14             )
15             .route(r -> r.order(-1)
16                 .host("**.throttle.org").and().path("/get")
17                 .filters(f -> f.filter(throttle.apply(1,
18                         1,
19                         10,
20                         TimeUnit.SECONDS)))
21                 .uri("http://httpbin.org:80")
22             )
23             .build();
24 }
```

这种风格允许自定义更多的谓词断言，默认是逻辑与（and）。你也可以用and() ， or() ， negate() 

再来一个例子

```
 1 @SpringBootApplication
 2 public class DemogatewayApplication {
 3 	@Bean
 4 	public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
 5 		return builder.routes()
 6 			.route("path_route", r -> r.path("/get")
 7 				.uri("http://httpbin.org"))
 8 			.route("host_route", r -> r.host("*.myhost.org")
 9 				.uri("http://httpbin.org"))
10 			.route("hystrix_route", r -> r.host("*.hystrix.org")
11 				.filters(f -> f.hystrix(c -> c.setName("slowcmd")))
12 				.uri("http://httpbin.org"))
13 			.route("hystrix_fallback_route", r -> r.host("*.hystrixfallback.org")
14 				.filters(f -> f.hystrix(c -> c.setName("slowcmd").setFallbackUri("forward:/hystrixfallback")))
15 				.uri("http://httpbin.org"))
16 			.route("limit_route", r -> r
17 				.host("*.limited.org").and().path("/anything/**")
18 				.filters(f -> f.requestRateLimiter(c -> c.setRateLimiter(redisRateLimiter())))
19 				.uri("http://httpbin.org"))
20 			.build();
21 	}
22 }
```

3.8. CORS配置

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
spring:
  cloud:
    gateway:
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins: "https://docs.spring.io"
            allowedMethods:
            - GET
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面的例子中，所有原始为docs.spring.io的GET请求均被允许跨域请求。

**4. 示例**

![img](https://img2018.cnblogs.com/blog/874963/201907/874963-20190704140751886-512659222.png)

 

本例中又4个项目，如下图：

![img](https://img2018.cnblogs.com/blog/874963/201907/874963-20190704140954746-1893213585.png)

4.1. cjs-eureka-server

**pom.xml**

```
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 3          xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
 4     <modelVersion>4.0.0</modelVersion>
 5     <parent>
 6         <groupId>org.springframework.boot</groupId>
 7         <artifactId>spring-boot-starter-parent</artifactId>
 8         <version>2.1.6.RELEASE</version>
 9         <relativePath/> <!-- lookup parent from repository -->
10     </parent>
11     <groupId>com.cjs.example</groupId>
12     <artifactId>cjs-eureka-server</artifactId>
13     <version>0.0.1-SNAPSHOT</version>
14     <name>cjs-eureka-server</name>
15 
16     <properties>
17         <java.version>1.8</java.version>
18         <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
19     </properties>
20 
21     <dependencies>
22         <dependency>
23             <groupId>org.springframework.cloud</groupId>
24             <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
25         </dependency>
26         <dependency>
27             <groupId>ch.qos.logback</groupId>
28             <artifactId>logback-classic</artifactId>
29             <version>1.2.3</version>
30         </dependency>
31     </dependencies>
32 
33     <dependencyManagement>
34         <dependencies>
35             <dependency>
36                 <groupId>org.springframework.cloud</groupId>
37                 <artifactId>spring-cloud-dependencies</artifactId>
38                 <version>${spring-cloud.version}</version>
39                 <type>pom</type>
40                 <scope>import</scope>
41             </dependency>
42         </dependencies>
43     </dependencyManagement>
44 
45     <build>
46         <plugins>
47             <plugin>
48                 <groupId>org.springframework.boot</groupId>
49                 <artifactId>spring-boot-maven-plugin</artifactId>
50             </plugin>
51         </plugins>
52     </build>
53 
54 </project>
```

**application.yml**

```
 1 server:
 2   port: 8761
 3 
 4 spring:
 5   application:
 6     name: cjs-eureka-server
 7 
 8 eureka:
 9   client:
10     service-url:
11       defaultZone: http://10.0.29.92:8761/eureka/,http://10.0.29.232:8761/eureka/
12 
13 logging:
14   file: ${spring.application.name}.log
```

**Application.java**

```
 1 package com.cjs.example;
 2 
 3 import org.springframework.boot.SpringApplication;
 4 import org.springframework.boot.autoconfigure.SpringBootApplication;
 5 import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
 6 
 7 /**
 8  * @author chengjiansheng
 9  * @date 2019-06-26
10  */
11 @EnableEurekaServer
12 @SpringBootApplication
13 public class CjsEurekaServerApplication {
14 
15     public static void main(String[] args) {
16         SpringApplication.run(CjsEurekaServerApplication.class, args);
17     }
18 
19 } 
```

4.2. cjs-gateway-server

**pom.xml**

```
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 3          xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
 4     <modelVersion>4.0.0</modelVersion>
 5     <parent>
 6         <groupId>org.springframework.boot</groupId>
 7         <artifactId>spring-boot-starter-parent</artifactId>
 8         <version>2.1.6.RELEASE</version>
 9         <relativePath/> <!-- lookup parent from repository -->
10     </parent>
11     <groupId>com.cjs.example</groupId>
12     <artifactId>cjs-gateway-server</artifactId>
13     <version>0.0.1-SNAPSHOT</version>
14     <name>cjs-gateway-server</name>
15 
16     <properties>
17         <java.version>1.8</java.version>
18         <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
19     </properties>
20 
21     <dependencies>
22         <dependency>
23             <groupId>org.springframework.cloud</groupId>
24             <artifactId>spring-cloud-starter-gateway</artifactId>
25         </dependency>
26 
27         <dependency>
28             <groupId>org.springframework.boot</groupId>
29             <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
30         </dependency>
31         <dependency>
32             <groupId>ch.qos.logback</groupId>
33             <artifactId>logback-classic</artifactId>
34             <version>1.2.3</version>
35         </dependency>
36     </dependencies>
37 
38     <dependencyManagement>
39         <dependencies>
40             <dependency>
41                 <groupId>org.springframework.cloud</groupId>
42                 <artifactId>spring-cloud-dependencies</artifactId>
43                 <version>${spring-cloud.version}</version>
44                 <type>pom</type>
45                 <scope>import</scope>
46             </dependency>
47         </dependencies>
48     </dependencyManagement>
49 
50     <build>
51         <plugins>
52             <plugin>
53                 <groupId>org.springframework.boot</groupId>
54                 <artifactId>spring-boot-maven-plugin</artifactId>
55             </plugin>
56         </plugins>
57     </build>
58 
59 </project>
```

**application.yml**

```
 1 server:
 2   port: 8080
 3   servlet:
 4     context-path: /
 5 spring:
 6   application:
 7     name: cjs-gateway-server
 8   redis:
 9     host: 10.0.29.187
10     password: 123456
11     port: 6379
12   cloud:
13     gateway:
14       routes:
15         - id: header_route
16           uri: http://10.0.29.187:8080/
17           predicates:
18             - Header=X-Request-Id, \d+
19 #        - id: path_route
20 #          uri: http://10.0.29.187:8080/
21 #          predicates:
22 #            - Path=/foo/{segment},/bar/{segment}
23         - id: query_route
24           uri: http://10.0.29.187:8080/
25           predicates:
26             - Query=baz
27 #      default-filters:
28 #        - AddResponseHeader=X-Response-Foo, Bar
29 #        - AddRequestParameter=hello, world
30 
31 logging:
32   file: ${spring.application.name}.log
```

**Application.java**

```
 1 package com.cjs.example.gateway;
 2 
 3 import org.springframework.boot.SpringApplication;
 4 import org.springframework.boot.autoconfigure.SpringBootApplication;
 5 import org.springframework.cloud.gateway.filter.ratelimit.KeyResolver;
 6 import org.springframework.cloud.gateway.filter.ratelimit.RedisRateLimiter;
 7 import org.springframework.cloud.gateway.route.RouteLocator;
 8 import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
 9 import org.springframework.context.annotation.Bean;
10 import org.springframework.web.bind.annotation.RestController;
11 import reactor.core.publisher.Mono;
12 
13 /**
14  * @author chengjiansheng
15  */
16 @RestController
17 @SpringBootApplication
18 public class CjsGatewayServerApplication {
19 
20     public static void main(String[] args) {
21         SpringApplication.run(CjsGatewayServerApplication.class, args);
22     }
23 
24     @Bean
25     public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
26         return builder.routes()
27                 .route("path_route", r -> r.path("/price/**")
28                         .filters(f -> f.addRequestHeader("hello", "world")
29                                 .addRequestParameter("name", "zhangsan")
30                                 .requestRateLimiter(c -> c.setRateLimiter(redisRateLimiter())))
31                         .uri("http://10.0.29.232:8082/price"))
32                 .route("path_route", r -> r.path("/commodity/**").uri("http://10.0.29.92:8081/commodity"))
33                 .build();
34     }
35 
36     @Bean
37     public RedisRateLimiter redisRateLimiter() {
38         return new RedisRateLimiter(2, 4);
39     }
40 
41     @Bean
42     KeyResolver userKeyResolver() {
43 //        return exchange -> Mono.just(exchange.getRequest().getHeaders().getFirst("userId"));
44         return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("userId"));
45     }
46 
47 }
```

其余代码就不一一贴出来了，都在git上

https://github.com/chengjiansheng/cjs-springcloud-example

截两张图吧

![img](https://img2018.cnblogs.com/blog/874963/201907/874963-20190704144548248-1605699133.png)

![img](https://img2018.cnblogs.com/blog/874963/201907/874963-20190704144603476-1689105688.png)

 

下面看效果

效果一：正常路由

![img](https://img2018.cnblogs.com/blog/874963/201907/874963-20190704143206935-889394161.png)

![img](https://img2018.cnblogs.com/blog/874963/201907/874963-20190704143242693-846622558.png)

效果二：限流

```
1 ab -n 20 -c 10 http://10.0.29.187:8080/price/index/test01?userId=123
```

观察控制台会看到

```
1 2019-07-03 18:21:23.946 DEBUG 34433 --- [ioEventLoop-4-1] o.s.c.g.f.ratelimit.RedisRateLimiter     : response: Response{allowed=false, headers={X-RateLimit-Remaining=0, X-RateLimit-Burst-Capacity=4, X-RateLimit-Replenish-Rate=2}, tokensRemaining=-1}
2 2019-07-03 18:21:23.946 DEBUG 34433 --- [ioEventLoop-4-1] o.s.w.s.adapter.HttpWebHandlerAdapter    : [53089629] Completed 429 TOO_MANY_REQUESTS
```

**5. 文档**

https://spring.io/projects/spring-cloud-gateway

https://spring.io/guides/gs/gateway/

https://cloud.spring.io/spring-cloud-gateway/spring-cloud-gateway.html

https://github.com/spring-cloud/spring-cloud-gateway/tree/master/spring-cloud-gateway-sample

https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.1.2.RELEASE/single/spring-cloud-netflix.html

 

https://stripe.com/blog/rate-limiters

https://en.wikipedia.org/wiki/Token_bucket

 

https://blog.csdn.net/qingmengwuhen1/article/details/80742654

https://blog.mkfree.com/archives/236