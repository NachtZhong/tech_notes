

## 1. 简介

Spring Cloud Gateway 是 Spring Cloud 的一个全新项目，该项目是基于 Spring 4.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。

Spring Cloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是替代 Netflix Zuul，其不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/指标，和限流

### 1.1 术语

- **Route（路由）**：这是网关的基本构建块。它由一个 ID，一个目标 URI，一组断言和一组过滤器定义。如果断言为真，则路由匹配。
- **Predicate（断言）**：这是一个 Java 8 的 Predicate。输入类型是一个 ServerWebExchange。我们可以使用它来匹配来自 HTTP 请求的任何内容，例如 headers 或参数。
- **Filter（过滤器）**：这是org.springframework.cloud.gateway.filter.GatewayFilter的实例，我们可以使用它修改请求和响应。

### 1.2 流程

![image-20190927162112460](/Users/nacht/Documents/Talking-To-The-Moon/tech_notes/microservice/assets/image-20190927162112460.png)

客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑。



### 1.3 springcloud gateaway和zuul

zuul1是阻塞的, 性能不是特别理想, 而zuul2基于netty, 是非阻塞的, 但是由于它的"连续跳票", 在zuul2开源的时候springcloud已经开发出自己的网关springcloud gateaway来作为zuul的替代品

![image-20190927162137941](/Users/nacht/Documents/Talking-To-The-Moon/tech_notes/microservice/assets/image-20190927162137941.png)

官方的测试项目对比了springcloud gateaway和zuul1.x的性能差异, 结果显示springcloud gateaway的RPS(Request Per Second)是zuul1.x版本的1.6倍



## 2. 入门示例

springcloud gateaway网关路由有两种配置方式

- 在配置文件yml中配置
- 通过@Bean自定义RouteLocator, 在启动主类Application中配置

一般采用yml方式进行配置

### 2.1 项目依赖

```xml
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
```



### 2.2 配置文件

```yaml
server:
  port: 8080
spring:
  application:
    name: spring-cloud-gateway
  cloud:
    gateway:
      routes:
        - id: hello_route
          uri: http://localhost:8009
          predicates:
            - Path=/hello
        - id: csdn_route
          uri: https://www.jianshu.com
          predicates:
            - Path=/p/f4cca5ce055a
        - id: post_route
          uri: http://localhost:8009
          predicates:
            - Method=POST
        - id: get_route
          uri: http://localhost:8009
          predicates:
            - Method=GET
logging:
  level:
    org.springframework.cloud.gateway: TRACE
    org.springframework.http.server.reactive: DEBUG
    org.springframework.web.reactive: DEBUG
    reactor.ipc.netty: DEBUG
```



id : 自定义路由名字, 唯一标识

uri: 目标地址(ip+端口或者域名+端口, 需带协议)

predicates: 匹配规则, 有多个维度的匹配规则,根据实际场景选用



## 3. springcloud内置的predicates

Spring Cloud Gateway里面包含了许多内置的路由匹配规则工厂, 这些匹配规则可以用来匹配HTTP请求的不同属性, 这些匹配规则可以通过逻辑运算符and等进行组合



### 3.1 Before路由匹配规则

Before路由匹配规则只有一个日期作为参数, 这个匹配规则会匹配在请求时间在该日期之前的所有请求

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: before_route
        uri: https://example.org
        predicates:
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```



### 3.2 After路由匹配规则

After路由匹配规则只有一个日期作为参数, 这个匹配规则会匹配在请求时间在该日期之后的所有请求

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```



### 3.3 Between路由匹配规则

Between路由匹配规则有两个参数, datetime1和datetime2, 这个匹配规则会匹配请求时间在datetime1和datetime2之间的所有请求(datetime2必须在datetime1之后)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: https://example.org
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```



### 3.4 Cookie路由匹配规则

Cookie路由匹配规则有两个参数, cookie的name和一个正则表达式, 这个匹配规则匹配name相同并且值和正则表达式匹配的cookie对应的请求

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: https://example.org
        predicates:
        - Cookie=chocolate, ch.p
```



### 3.5 Header路由匹配规则

Header路由匹配规则有两个参数, header的name和一个正则表达式, 这个匹配规则匹配header的name相同并且值符合正则表达式的Header对应的请求

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```



### 3.6 Host路由匹配规则

Host路由匹配规则有一个参数, 一个hostname pattern的列表, 这个匹配规则匹配Header的Host符合列表里pattern的请求

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Host=**.somehost.org,**.anotherhost.org
```



### 3.7 Method路由匹配规则

Method路由匹配规则只有一个参数, HTTP调用方法, 这个匹配规则匹配HTTP调用方法符合参数配置的请求

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET
```



### 3.8 Path路由匹配规则

Path路由匹配规则有两个参数, 一个Spring PathMatcher的Pattern列表和一个是否自动填充斜线进行匹配的标志位

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Path=/foo/{segment},/bar/{segment}
```

这个匹配规则可以匹配请求的path为例如`/foo/1` or `/foo/bar` or `/bar/baz`的请求, 它会把URI里面模板参数(例如上面配置中的segment)提取出来, 然后把它放到ServerWebExchange.getAttributes()里面, key会被定义在ServerWebExchangeUtils.URI_TEMPLATE_VARIABLES_ATTRIBUTE里面

一个常用的获取这些参数的方法如下:

```Java
Map<String, String> uriVariables = ServerWebExchangeUtils.getPathPredicateVariables(exchange);

String segment = uriVariables.get("segment");
```



### 3.9 Query路由匹配规则

Query路由匹配规则有两个参数, 一个为查询参数名, 另一个是可选的, 查询参数的值(正则表达式)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=baz
```



```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=foo, ba.
```



### 3.10 RemoteAddr路由匹配规则

RemoteAddr路由匹配规则参数是一个list,  里面是远程地址的列表(包含ip地址和子网掩码)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: https://example.org
        predicates:
        - RemoteAddr=192.168.1.1/24
```



### 3.11 Weight路由匹配规则

Weight匹配规则有两个参数, 组名和权重

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: weight_high
        uri: https://weighthigh.org
        predicates:
        - Weight=group1, 8
      - id: weight_low
        uri: https://weightlow.org
        predicates:
        - Weight=group1, 2
```

上述配置会将大约80%流量转发到https://weighthigh.org, 20%流量转发到https://weightlow.org



各种 Predicates 同时存在于同一个路由时，请求必须同时满足所有的条件才被这个路由匹配。



> 一个请求满足多个路由的谓词条件时，请求只会被首个成功匹配的路由转发



## 4. 过滤器GatewayFilter

springcloud gateway和zuul相似, 有pre和post两种方式的filter, 客户端的请求先经过 “pre” 类型的 filter，然后将请求转发到具体的业务服务，收到业务服务的响应之后，再经过“post”类型的filter处理，最后返回响应到客户端。在“pre”类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等，在“post”类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等。

与 Zuul 不同的是，filter 除了分为 “pre” 和 “post” 两种方式的 filter 外，在 Spring Cloud Gateway 中，filter 从作用范围可分为另外两种，一种是针对于单个路由的 gateway filter，它需要像上面 application.yml 中的 filters 那样在单个路由中配置；另外一种是针对于全部路由的global gateway filter，不需要单独配置，对所有路由生效。

springcloud gateway内置了许多常用的filter工厂

### 4.1 AddRequestHeader GatewayFilter Factory

采用一对名称和值作为参数  **application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: http://example.org
        filters:
        - AddRequestHeader=X-Request-Foo, Bar
```

对于所有匹配的请求，这将向下游请求的头中添加 `x-request-foo:bar` header

### 4.2 AddRequestParameter GatewayFilter Factory

采用一对名称和值作为参数  **application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: http://example.org
        filters:
        - AddRequestParameter=foo, bar
```

对于所有匹配的请求，这将向下游请求添加`foo=bar`查询字符串

### 4.3 AddResponseHeader GatewayFilter Factory

采用一对名称和值作为参数

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: http://example.org
        filters:
        - AddResponseHeader=X-Response-Foo, Bar
```

对于所有匹配的请求，这会将`x-response-foo:bar`头添加到下游响应的header中

### 4.4 Hystrix GatewayFilter Factory

[Hystrix](https://github.com/Netflix/Hystrix) 是Netflix开源的断路器组件。Hystrix GatewayFilter允许你向网关路由引入断路器，保护你的服务不受级联故障的影响，并允许你在下游故障时提供fallback响应。

要在项目中启用Hystrix网关过滤器，需要添加对 `spring-cloud-starter-netflix-hystrix`的依赖 [Spring Cloud Netflix](https://cloud.spring.io/spring-cloud-netflix/).

Hystrix GatewayFilter Factory 需要一个name参数，即`HystrixCommand`的名称。

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: hystrix_route
        uri: http://example.org
        filters:
        - Hystrix=myCommandName
```

这将剩余的过滤器包装在命令名为“myCommandName”的`HystrixCommand`中。

hystrix过滤器还可以接受可选的`fallbackUri` 参数。目前，仅支持`forward:` 预设的URI，如果调用fallback，则请求将转发到与URI匹配的控制器。

**application.yml.**

```yaml
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

当调用hystrix fallback时，这将转发到`/incaseoffailureusethis` uri。注意，这个示例还演示了（可选）通过目标URI上的'lb`前缀,使用Spring Cloud Netflix Ribbon 客户端[负载均衡](https://cloud.tencent.com/product/clb?from=10680)。

主要场景是使用`fallbackUri` 到网关应用程序中的内部控制器或处理程序。但是，也可以将请求重新路由到外部应用程序中的控制器或处理程序，如：

**application.yml.**

```yaml
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

在本例中，gateway应用程序中没有 `fallback` 实现，但是另一个应用程序中有一个接口实现，注册为“[http://localhost:9994](http://localhost:9994/)”。

在将请求转发到fallback的情况下，Hystrix Gateway过滤还支持直接抛出`Throwable` 。它被作为`ServerWebExchangeUtils.HYSTRIX_EXECUTION_EXCEPTION_ATTR`属性添加到`ServerWebExchange`中，可以在处理网关应用程序中的fallback时使用。

对于外部控制器/处理程序方案，可以添加带有异常详细信息的header。可以在 [FallbackHeaders GatewayFilter Factory section](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.0.RELEASE/single/spring-cloud-gateway.html#fallback-headers).中找到有关它的更多信息。

hystrix配置参数（如 timeouts）可以使用全局默认值配置，也可以使用[Hystrix wiki](https://github.com/Netflix/Hystrix/wiki/Configuration)中所述属性进行配置。

要为上面的示例路由设置5秒超时，将使用以下配置：

**application.properties**

```properties
hystrix.command.fallbackcmd.execution.isolation.thread.timeoutInMilliseconds: 5000
```

### 4.5 FallbackHeaders GatewayFilter Factory

`FallbackHeaders`允许在转发到外部应用程序中的`FallbackUri`的请求的header中添加Hystrix异常详细信息，如下所示：

**application.yml.**

```yaml
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
        filters:
        - name: FallbackHeaders
          args:
            executionExceptionTypeHeaderName: Test-Header
```

在本例中，在运行`HystrixCommand`发生执行异常后，请求将被转发到 `localhost:9994`应用程序中的 `fallback`终端或程序。异常类型、消息（如果可用）cause exception类型和消息的头，将由`FallbackHeaders` filter添加到该请求中。

通过设置下面列出的参数值及其默认值，可以在配置中覆盖headers的名称：

-  `executionExceptionTypeHeaderName` (`"Execution-Exception-Type"`)
-  `executionExceptionMessageHeaderName` (`"Execution-Exception-Message"`)
-  `rootCauseExceptionTypeHeaderName` (`"Root-Cause-Exception-Type"`)
-  `rootCauseExceptionMessageHeaderName` (`"Root-Cause-Exception-Message"`)

Hystrix 如何实现的更多细节可以参考 [Hystrix GatewayFilter Factory section](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.0.RELEASE/single/spring-cloud-gateway.html#hystrix).

### 4.6 PrefixPath GatewayFilter Factory

PrefixPath GatewayFilter 只有一个 `prefix` 参数.

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: http://example.org
        filters:
        - PrefixPath=/mypath
```

这将给所有匹配请求的路径加前缀`/mypath`。因此，向`/hello`发送的请求将发送到`/mypath/hello`。

### 4.7 PreserveHostHeader GatewayFilter Factory

该filter没有参数。设置了该Filter后，GatewayFilter将不使用由HTTP客户端确定的host header ，而是发送原始host header 。

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: preserve_host_route
        uri: http://example.org
        filters:
        - PreserveHostHeader
```

### 4.8 RequestRateLimiter GatewayFilter Factory

RequestRateLimiter使用`RateLimiter`实现是否允许继续执行当前请求。如果不允许继续执行，则返回`HTTP 429 - Too Many Requests` （默认情况下）。

这个过滤器可以配置一个可选的`keyResolver` 参数和rate limiter参数（见下文）。

`keyResolver` 是 `KeyResolver` 接口的实现类.在配置中，按名称使用SpEL引用bean。`#{@myKeyResolver}` 是引用名为'myKeyResolver'的bean的SpEL表达式。

**KeyResolver.java.**

```java
public interface KeyResolver {
    Mono<String> resolve(ServerWebExchange exchange);
}
```

`KeyResolver`接口允许使用可插拔策略来派生限制请求的key。在未来的里程碑版本中，将有一些`KeyResolver`实现。

`KeyResolver`的默认实现是`PrincipalNameKeyResolver`，它从`ServerWebExchange`检索`Principal`并调用`Principal.getName()`。

默认情况下，如果`KeyResolver` 没有获取到key，请求将被拒绝。此行为可以使用 `spring.cloud.gateway.filter.request-rate-limiter.deny-empty-key` (true or false) 和 `spring.cloud.gateway.filter.request-rate-limiter.empty-key-status-code`属性进行调整。

>  说明  无法通过"shortcut" 配置RequestRateLimiter。以下示例*无效*  

**application.properties.**

```properties
# INVALID SHORTCUT CONFIGURATION
spring.cloud.gateway.routes[0].filters[0]=RequestRateLimiter=2, 2, #{@userkeyresolver}
```

#### 4.8.1 [Redis](https://cloud.tencent.com/product/crs?from=10680) RateLimiter

Redis的实现基于 [Stripe](https://stripe.com/blog/rate-limiters)实现。它需要使用 `spring-boot-starter-data-redis-reactive` Spring Boot starter。

使用的算法是[Token Bucket Algorithm](https://en.wikipedia.org/wiki/Token_bucket).。

`redis-rate-limiter.replenishRate`是你允许用户每秒执行多少请求，而丢弃任何请求。这是令牌桶的填充速率。

``redis-rate-limiter.burstCapacity`是允许用户在一秒钟内执行的最大请求数。这是令牌桶可以保存的令牌数。将此值设置为零将阻止所有请求。

稳定速率是通过在`replenishRate` 和 `burstCapacity`中设置相同的值来实现的。可通过设置`burstCapacity`高于`replenishRate`来允许临时突发流浪。在这种情况下，限流器需要在两次突发之间留出一段时间（根据`replenishRate`），因为连续两次突发将导致请求丢失 (`HTTP 429 - Too Many Requests`).。

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: http://example.org
        filters:
        - name: RequestRateLimiter
          args:
            redis-rate-limiter.replenishRate: 10
            redis-rate-limiter.burstCapacity: 20
```

**Config.java.**

```java
@Bean
KeyResolver userKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
}
```

这定义了每个用户10个请求的限制。允许20个突发，但下一秒只有10个请求可用。`KeyResolver`是一个简单的获取`user`请求参数的工具（注意：不建议用于生产）。

限流器也可以定义为`RateLimiter`接口的实现 bean。在配置中，按名称使用SpEL引用bean。`#{@myRateLimiter}`是引用名为'myRateLimiter'的bean的SpEL表达式。

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: http://example.org
        filters:
        - name: RequestRateLimiter
          args:
            rate-limiter: "#{@myRateLimiter}"
            key-resolver: "#{@userKeyResolver}"
```

### 4.9 RedirectTo GatewayFilter Factory

该过滤器有一个 `status` 和一个 `url`参数。status是300类重定向HTTP代码，如301。该URL应为有效的URL，这将是 `Location` header的值。

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: http://example.org
        filters:
        - RedirectTo=302, http://acme.org
```

这将发送一个302状态码和一个`Location:http://acme.org` header来执行重定向。

### 4.10 RemoveNonProxyHeaders GatewayFilter Factory

RemoveNonProxyHeaders GatewayFilter Factory 从转发请求中删除headers。删除的默认头列表来自 [IETF](https://tools.ietf.org/html/draft-ietf-httpbis-p1-messaging-14#section-7.1.3).

**The default removed headers are:**

- Connection
- Keep-Alive
- Proxy-Authenticate
- Proxy-Authorization
- TE
- Trailer
- Transfer-Encoding
- Upgrade  要更改此设置，请将 `spring.cloud.gateway.filter.remove-non-proxy-headers.headers`属性设置为要删除的header名称。

### 4.11 RemoveRequestHeader GatewayFilter Factory

有一个`name`参数. 这是要删除的header的名称。

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestheader_route
        uri: http://example.org
        filters:
        - RemoveRequestHeader=X-Request-Foo
```

这将在`X-Request-Foo` header被发送到下游之前删除它。

### 4.12 RemoveResponseHeader GatewayFilter Factory

有一个`name`参数. 这是要删除的header的名称。

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removeresponseheader_route
        uri: http://example.org
        filters:
        - RemoveResponseHeader=X-Response-Foo
```

这将在返回到网关client之前从响应中删除`x-response-foo`头。

### 4.13 RewritePath GatewayFilter Factory

包含一个 `regexp`正则表达式参数和一个 `replacement` 参数. 通过使用Java正则表达式灵活地重写请求路径。

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewritepath_route
        uri: http://example.org
        predicates:
        - Path=/foo/**
        filters:
        - RewritePath=/foo/(?<segment>.*), /$\{segment}
```

对于请求路径`/foo/bar`，将在发出下游请求之前将路径设置为`/bar`。注意,由于YAML规范，请使用 `$\`替换 `$`。

### 4.14 RewriteResponseHeader GatewayFilter Factory

包含 `name`, `regexp`和 `replacement` 参数.。通过使用Java正则表达式灵活地重写响应头的值。

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewriteresponseheader_route
        uri: http://example.org
        filters:
        - RewriteResponseHeader=X-Response-Foo, , password=[^&]+, password=***
```

对于一个`/42?user=ford&password=omg!what&flag=true`的header值，在做下游请求时将被设置为`/42?user=ford&password=***&flag=true`，由于YAML规范，请使用 `$\`替换 `$`。

### 4.15 SaveSession GatewayFilter Factory

SaveSession GatewayFilter Factory将调用转发到下游之**前**强制执行`WebSession::save` 操作。这在使用 [Spring Session](https://projects.spring.io/spring-session/) 之类时特别有用，需要确保会话状态在进行转发调用之前已保存。

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: save_session
        uri: http://example.org
        predicates:
        - Path=/foo/**
        filters:
        - SaveSession
```

如果你希望要将[Spring Security]（https://projects.spring.io/Spring Security/）与Spring Session集成,并确保安全详细信息已转发到远程的进程，这一点至关重要。

### 4.16 SecureHeaders GatewayFilter Factory

SecureHeaders GatewayFilter Factory 将许多headers添加到reccomedation处的响应中，从[this blog post](https://blog.appcanary.com/2017/http-security-headers.html).

**添加以下标题（使用默认值分配）:**

- `X-Xss-Protection:1; mode=block`
- `Strict-Transport-Security:max-age=631138519`
- `X-Frame-Options:DENY`
- `X-Content-Type-Options:nosniff`
- `Referrer-Policy:no-referrer`
- `Content-Security-Policy:default-src 'self' https:; font-src 'self' https: data:; img-src 'self' https: data:; object-src 'none'; script-src https:; style-src 'self' https: 'unsafe-inline'`
- `X-Download-Options:noopen`
- `X-Permitted-Cross-Domain-Policies:none`

要更改默认值，请在`spring.cloud.gateway.filter.secure-headers` 命名空间中设置相应的属性：

**Property to change:**

- `xss-protection-header`
- `strict-transport-security`
- `frame-options`
- `content-type-options`
- `referrer-policy`
- `content-security-policy`
- `download-options`
- `permitted-cross-domain-policies`

### 4.17 SetPath GatewayFilter Factory

SetPath GatewayFilter Factory 采用 `template`路径参数。它提供了一种通过允许路径的模板化segments来操作请求路径的简单方法。使用Spring Framework中的URI模板，允许多个匹配segments。

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setpath_route
        uri: http://example.org
        predicates:
        - Path=/foo/{segment}
        filters:
        - SetPath=/{segment}
```

对于一个 `/foo/bar`请求，在做下游请求前，路径将被设置为`/bar`

### 4.18 SetResponseHeader GatewayFilter Factory

SetResponseHeader GatewayFilter Factory 包括 `name` 和 `value` 参数.

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setresponseheader_route
        uri: http://example.org
        filters:
        - SetResponseHeader=X-Response-Foo, Bar
```

此GatewayFilter使用给定的名称替换所有header，而不是添加。因此，如果下游服务器响应为`X-Response-Foo:1234`，则会将其替换为`X-Response-Foo:Bar`,这是网关客户端将接收的内容。

### 4.19 SetStatus GatewayFilter Factory

SetStatus GatewayFilter Factory 包括唯一的 `status`参数.必须是一个可用的Spring `HttpStatus`。它可以是整数值`404`或字符串枚举`NOT_FOUND`。

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setstatusstring_route
        uri: http://example.org
        filters:
        - SetStatus=BAD_REQUEST
      - id: setstatusint_route
        uri: http://example.org
        filters:
        - SetStatus=401
```

在这个例子中，HTTP返回码将设置为401.

### 4.20 StripPrefix GatewayFilter Factory

StripPrefix GatewayFilter Factory 包括一个`parts`参数。 `parts`参数指示在将请求发送到下游之前，要从请求中去除的路径中的节数。

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: nameRoot
        uri: http://nameservice
        predicates:
        - Path=/name/**
        filters:
        - StripPrefix=2
```

当通过网关发出`/name/bar/foo`请求时，向`nameservice`发出的请求将是`http://nameservice/foo`。

### 4.21 Retry GatewayFilter Factory

Retry GatewayFilter Factory包括 `retries`, `statuses`, `methods`和 `series` 参数.

-  `retries`: 应尝试的重试次数
-  `statuses`: 应该重试的HTTP状态代码，用`org.springframework.http.HttpStatus`标识
-  `methods`: 应该重试的HTTP方法，用 `org.springframework.http.HttpMethod`标识
-  `series`: 要重试的一系列状态码，用 `org.springframework.http.HttpStatus.Series`标识

**application.yml.**

```yaml
spring:
	cloud:
		gateway:
			routes:
				- id: retry_test
			  uri: http://localhost:8080/flakey
				predicates:
					- Host=*.retry.com
			filters:
				- name: Retry
				args:
					retries: 3
					statuses: BAD_GATEWAY
```



>  注意  retry filter 不支持body请求的重试，如通过body的POST 或 PUT请求  
>
>  注意  在使用带有前缀为 `forward:` 的`retry filter`时，应仔细编写目标端点，以便在出现错误时不会执行任何可能导致将响应发送到客户端并提交的操作。例如，如果目标端点是带注解的controller，则目标controller方法不应返回带有错误状态代码的`ResponseEntity`。相反，它应该抛出一个`Exception`，或者发出一个错误信号，例如通过`Mono.error(ex)` 返回值，重试过滤器可以配置为通过重试来处理。  

### 4.22 RequestSize GatewayFilter Factory

当请求大小大于允许的限制时，RequestSize GatewayFilter Factory可以限制请求不到达下游服务。过滤器以`RequestSize`作为参数，这是定义请求的允许大小限制(以字节为单位)。

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: request_size_route
      uri: http://localhost:8080/upload
      predicates:
      - Path=/upload
      filters:
      - name: RequestSize
        args:
          maxSize: 5000000
```

当请求因大小而被拒绝时， RequestSize GatewayFilter Factory 将响应状态设置为`413 Payload Too Large`，并带有额外的header `errorMessage` 。下面是一个 `errorMessage`的例子。

```
errorMessage` : `Request size is larger than permissible limit. Request size is 5.0 MB where permissible limit is 4.0 MB
```

>  注意  如果未在路由定义中作为filter参数提供，则默认请求大小将设置为5 MB。  

### 4.23 Modify Request Body GatewayFilter Factory

这个过滤器被定义为beta版本，将来API可能会改变。

此过滤器可用于在请求主体被网关发送到下游之前对其进行修改。

>  注意  只能使用Java DSL配置此过滤器  

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_request_obj", r -> r.host("*.rewriterequestobj.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyRequestBody(String.class, Hello.class, MediaType.APPLICATION_JSON_VALUE,
                    (exchange, s) -> return Mono.just(new Hello(s.toUpperCase())))).uri(uri))
        .build();
}

static class Hello {
    String message;

    public Hello() { }

    public Hello(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

### 4.24 Modify Response Body GatewayFilter Factory

这个过滤器被定义为beta版本，将来API可能会改变。  此过滤器可用于在将响应正文发送回客户端之前对其进行修改。

>  注意  只能使用Java DSL配置此过滤器  

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_response_upper", r -> r.host("*.rewriteresponseupper.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyResponseBody(String.class, String.class,
                    (exchange, s) -> Mono.just(s.toUpperCase()))).uri(uri)
        .build();
}
```



## 5. 全局过滤器GlobalFilter

`GlobalFilter`接口与`GatewayFilter`具有相同的签名。是有条件地应用于所有路由的特殊过滤器



### 5.1 过滤器的排序规则

当请求进入网关, 匹配到路由的时候, Web Handler会将所有的filter实例添加到filter chain, filter组合的排序由org.springframework.core.Ordered接口决定, 可以通过实现getOrder方法或者使用@Order注解来设置

由于Spring Cloud Gateway将用于执行过滤器逻辑区分为“前置”和“后置”阶段，具有最高优先级的过滤器将是“前置”阶段的第一个，而“后置”阶段的最后一个。

> Lower values have higher priority.

代码配置全局过滤器示例

```Java
@Bean
@Order(-1)
public GlobalFilter a() {
    return (exchange, chain) -> {
        log.info("first pre filter");
        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            log.info("third post filter");
        }));
    };
}

@Bean
@Order(0)
public GlobalFilter b() {
    return (exchange, chain) -> {
        log.info("second pre filter");
        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            log.info("second post filter");
        }));
    };
}

@Bean
@Order(1)
public GlobalFilter c() {
    return (exchange, chain) -> {
        log.info("third pre filter");
        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            log.info("first post filter");
        }));
    };
}
```



### 5.2 Forward Routing Filter

`ForwardRoutingFilter`在exchange属性`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`中查找URI。如果URL有一个`forward`scheme (如 `forward:///localendpoint`)，它将使用Spring `DispatcherHandler` 来处理请求。请求URL的路径部分将被转发URL中的路径覆盖。未修改的原始URL将附加到 `ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR`属性中的列表中。

### 5.3 LoadBalancerClient Filter

`LoadBalancerClientFilter`在exchange属性`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`中查找URI。如果URL有一个`lb`scheme (如 `lb://myservice`），它将使用Spring Cloud `LoadBalancerClient` 将名称（在前一个示例中为'myservice`）解析为实际主机和端口，并替换URI。未修改的原始URL将附加到`ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR`属性中的列表中。过滤器还将查看`ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR`属性，查看它是否等于`lb`，然后应用相同的规则。

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: myRoute
        uri: lb://service
        predicates:
        - Path=/service/**
```

>  注意  默认情况下，如果一个服务实例在`LoadBalancer` 中没有发现，则返回503。可以通过设置`spring.cloud.gateway.loadbalancer.use404=true`来让网管返回404.  
>
>  注意  

从`LoadBalancer`返回的`ServiceInstance`的`isSecure` 值将覆盖在对网关发出的请求中指定的scheme。例如，如果请求通过`HTTPS`进入网关，但`ServiceInstance`表示它不安全，则下游请求将通过`HTTP`协议。相反的情况也适用。但是，如果在网关配置中为路由指定了`GATEWAY_SCHEME_PREFIX_ATTR`，则前缀将被删除，并且路由URL生成的scheme将覆盖`ServiceInstance`配置。

### 5.4 Netty Routing Filter

如果位于 `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`属性中的URL具有`http` 或`https` 模式，则会运行Netty Routing Filter。它使用Netty `HttpClient` 发出下游代理请求。响应放在 `ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR` exchange属性中，以便在以后的过滤器中使用。（有一个实验阶段不需要Netty的相同的功能的Filter，`WebClientHttpRoutingFilter`）

### 5.5 Netty Write Response Filter

如果`ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR` exchange属性中存在 Netty `HttpClientResponse`，则运行 `NettyWriteResponseFilter` 。它在其他所有过滤器完成后将代理响应写回网关客户端响应之后运行。（有一个不需要netty的实验性的`WebClientWriteResponseFilter`执行相同的功能）

### 5.6 RouteToRequestUrl Filter

如果`ServerWebExchangeUtils.GATEWAY_ROUTE_ATTR` exchange属性中存在`Route`对象，`RouteToRequestUrlFilter`将运行。它基于请求URI创建一个新的URI，使用`Route`对象的uri属性进行更新。新的URI被放置在`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` exchange属性中。

如果该URI有一个前缀scheme，例如`lb:ws://serviceid`，则会从该URI中剥离该 `lb` scheme，并将其放置在`ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR`中，以便稍后在过滤器链中使用。

### 5.7 Websocket Routing Filter

如果`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`exchange属性中有 `ws` 、 `wss`scheme，则Websocket Routing Filter将被运行。它使用Spring Web Socket基础模块将Websocket转发到下游。

URI前缀为`lb`的Websockets可以被负载均衡，如 `lb:ws://serviceid`.

>  注意  如果使用 [SockJS](https://github.com/sockjs) 作为普通HTTP的fallback，则应配置普通HTTP路由以及WebSocket路由。  

**application.yml.**

```yaml
spring:
  cloud:
    gateway:
      routes:
      # SockJS route
      - id: websocket_sockjs_route
        uri: http://localhost:3001
        predicates:
        - Path=/websocket/info/**
      # Normwal Websocket route
      - id: websocket_route
        uri: ws://localhost:3001
        predicates:
        - Path=/websocket/**
6.8 Gateway Metrics Filter
```

### 5.8 Gateway Metrics Filter

要启用网关指标，请将spring-boot-starter-actuator添加为项目依赖项。然后，默认情况下，只要属性`spring.cloud.gateway.metrics.enabled`未设置为`false`，网关指标过滤器就会运行。此过滤器添加名为“gateway.requests”的计时器指标，并带有以下标记：

-  `routeId`: The route id
-  `routeUri`:  API 将被转发的URI
-  `outcome`: 结果分类依据 [HttpStatus.Series](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/HttpStatus.Series.html) 
-  `status`: 返回client的请求的Http Status

这些指标可以从`/actuator/metrics/gateway.requests`中获取，可以很容易地与Prometheus集成以创建[Grafana](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.0.RELEASE/single/images/gateway-grafana-dashboard.jpeg) [dashboard](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.0.RELEASE/single/gateway-grafana-dashboard.json).

>  注意  要将pometheus启用，需要添加 micrometer-registry-prometheus为项目依赖。  

### 5.9 Making An Exchange As Routed

网关路由`ServerWebExchange`之后，它将通过向Exchange属性添加`gatewayAlreadyRouted`，将该exchange标记为“routed”。一旦一个请求被标记为routed，其他路由过滤器将不会再次路由该请求，将跳过该过滤器。有一些方便的方法可以用来将exchange标记为routed，或者检查exchange是否已经routed。

-  `ServerWebExchangeUtils.isAlreadyRouted` 有一个 `ServerWebExchange`对象并检查它是否已"routed"
-  `ServerWebExchangeUtils.setAlreadyRouted` 有一个 `ServerWebExchange` 对象并将其标记为"routed"



## 6. Java定义路由api

配置实例

```java
// static imports from GatewayFilters and RoutePredicates
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder, ThrottleGatewayFilterFactory throttle) {
    return builder.routes()
            .route(r -> r.host("**.abc.org").and().path("/image/png")
                .filters(f ->
                        f.addResponseHeader("X-TestHeader", "foobar"))
                .uri("http://httpbin.org:80")
            )
            .route(r -> r.path("/image/webp")
                .filters(f ->
                        f.addResponseHeader("X-AnotherHeader", "baz"))
                .uri("http://httpbin.org:80")
            )
            .route(r -> r.order(-1)
                .host("**.throttle.org").and().path("/get")
                .filters(f -> f.filter(throttle.apply(1,
                        1,
                        10,
                        TimeUnit.SECONDS)))
                .uri("http://httpbin.org:80")
            )
            .build();
}
```



## 7. Actuator rest api

`/gateway`的actuator端点允许监视Spring Cloud Gateway应用程序并与之交互。要进行远程访问，必须在应用程序属性中暴露HTTP或JMX 端口。



application.properties

```properties
management.endpoint.gateway.enabled=true # default value
management.endpoints.web.exposure.include=gateway
```



### 7.1 检索所有的全局过滤器

get请求 `/actuator/gateway/globalfilters`

```properties
{
  "org.springframework.cloud.gateway.filter.LoadBalancerClientFilter@77856cc5": 10100,
  "org.springframework.cloud.gateway.filter.RouteToRequestUrlFilter@4f6fd101": 10000,
  "org.springframework.cloud.gateway.filter.NettyWriteResponseFilter@32d22650": -1,
  "org.springframework.cloud.gateway.filter.ForwardRoutingFilter@106459d9": 2147483647,
  "org.springframework.cloud.gateway.filter.NettyRoutingFilter@1fbd5e0": 2147483647,
  "org.springframework.cloud.gateway.filter.ForwardPathFilter@33a71d23": 0,
  "org.springframework.cloud.gateway.filter.AdaptCachedBodyGlobalFilter@135064ea": 2147483637,
  "org.springframework.cloud.gateway.filter.WebsocketRoutingFilter@23c05889": 2147483646
}
```



### 7.2 检索所有应用于路由的过滤器

get请求`/actuator/gateway/routefilters`

```properties
{
    "[SaveSessionGatewayFilterFactory@378f002a configClass = Object]": null,
    "[RemoveResponseHeaderGatewayFilterFactory@6c841199 configClass = AbstractGatewayFilterFactory.NameConfig]": null,
    "[StripPrefixGatewayFilterFactory@1afd72ef configClass = StripPrefixGatewayFilterFactory.Config]": null,
    "[SetResponseHeaderGatewayFilterFactory@3743539f configClass = AbstractNameValueGatewayFilterFactory.NameValueConfig]": null,
    "[RewritePathGatewayFilterFactory@6a818392 configClass = RewritePathGatewayFilterFactory.Config]": null,
    "[ModifyRequestBodyGatewayFilterFactory@43f1bb92 configClass = ModifyRequestBodyGatewayFilterFactory.Config]": null,
    "[AddRequestHeaderGatewayFilterFactory@17273273 configClass = AbstractNameValueGatewayFilterFactory.NameValueConfig]": null,
    "[RewriteResponseHeaderGatewayFilterFactory@d277579 configClass = RewriteResponseHeaderGatewayFilterFactory.Config]": null,
    "[ModifyResponseBodyGatewayFilterFactory@5c5d6175 configClass = ModifyResponseBodyGatewayFilterFactory.Config]": null,
    "[AddRequestParameterGatewayFilterFactory@5f69e2b configClass = AbstractNameValueGatewayFilterFactory.NameValueConfig]": null,
    "[PreserveHostHeaderGatewayFilterFactory@3b27b497 configClass = Object]": null,
    "[AddResponseHeaderGatewayFilterFactory@984169e configClass = AbstractNameValueGatewayFilterFactory.NameValueConfig]": null,
    "[RemoveRequestHeaderGatewayFilterFactory@3c74aa0d configClass = AbstractGatewayFilterFactory.NameConfig]": null,
    "[SecureHeadersGatewayFilterFactory@1de9b505 configClass = Object]": null,
    "[PrefixPathGatewayFilterFactory@7544ac86 configClass = PrefixPathGatewayFilterFactory.Config]": null,
    "[RequestHeaderToRequestUriGatewayFilterFactory@2cc75074 configClass = AbstractGatewayFilterFactory.NameConfig]": null,
    "[RetryGatewayFilterFactory@489091bd configClass = RetryGatewayFilterFactory.RetryConfig]": null,
    "[RequestSizeGatewayFilterFactory@445bb139 configClass = RequestSizeGatewayFilterFactory.RequestSizeConfig]": null,
    "[DedupeResponseHeaderGatewayFilterFactory@6d6bbd35 configClass = DedupeResponseHeaderGatewayFilterFactory.Config]": null,
    "[SetPathGatewayFilterFactory@512d6e60 configClass = SetPathGatewayFilterFactory.Config]": null,
    "[RedirectToGatewayFilterFactory@b1534d3 configClass = RedirectToGatewayFilterFactory.Config]": null,
    "[SetRequestHeaderGatewayFilterFactory@7b122839 configClass = AbstractNameValueGatewayFilterFactory.NameValueConfig]": null,
    "[SetStatusGatewayFilterFactory@5db6b845 configClass = SetStatusGatewayFilterFactory.Config]": null
}
```



### 7.3 清理路由缓存

POST请求`/actuator/gateway/refresh`

该请求将返回一个没有body的200状态码



### 7.4 检索路由

发送GET请求`/actuator/gateway/routes`

```json
[{
  "route_id": "first_route",
  "route_object": {
    "predicate": "org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory$$Lambda$432/1736826640@1e9d7e7d",
    "filters": [
      "OrderedGatewayFilter{delegate=org.springframework.cloud.gateway.filter.factory.PreserveHostHeaderGatewayFilterFactory$$Lambda$436/674480275@6631ef72, order=0}"
    ]
  },
  "order": 0
},
{
  "route_id": "second_route",
  "route_object": {
    "predicate": "org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory$$Lambda$432/1736826640@cd8d298",
    "filters": []
  },
  "order": 0
}]
```



如果需要检索单个路由信息, 在routes后面加上路由id即可



### 7.5 增删路由

要创建一个路由，发送POST请求 `/gateway/routes/{id_route_to_create}`，参数为JSON结构，具体参数数据结构参考上面章节。

要删除一个路由，发送 `DELETE`请求 `/gateway/routes/{id_route_to_delete}`。



## 8. 简单示例

### 8.1 predicates配置文件示例

```yaml
spring:
  application:
    name: spring-cloud-gateway
  cloud:
    gateway:
      routes:
        - id: hello_route
          uri: http://localhost:8888
          predicates:
            - Path=/hello
```



### 8.2 predicates代码配置示例

```java
@Configuration
public class RouteLocatorConfig {
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes().route("hello_route",r->r.path("/hello").uri("http://localhost:8888")).build();
    }
}
```



### 8.3 gatewayfilter配置文件示例

```yaml
spring:
  application:
    name: spring-cloud-gateway
  cloud:
    gateway:
      routes:
        - id: hello_route
          uri: http://localhost:8888
          predicates:
            - Path=/hello
          filters:
            - AddResponseHeader=name, hello
```



### 8.4 gatewayfilter代码配置示例



```java
@Configuration
public class GatewayFilterConfig {
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes().route("hello_route", r -> r.path("/hello").filters(f->f.addResponseHeader("name","hello")).uri("http://localhost:8888")).build();
    }
}
```





