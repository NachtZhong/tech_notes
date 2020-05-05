Springclout hystrix



## 1. 简介

在微服务架构中，我们将系统拆分成了一个个的服务单元，各单元应用间通过服务注册与订阅的方式互相依赖。由于每个单元都在不同的进程中运行，依赖通过远程调用的方式执行，这样就有可能因为网络原因或是依赖服务自身问题出现调用故障或延迟，而这些问题会直接导致调用方的对外服务也出现延迟，若此时调用方的请求不断增加，最后就会出现因等待出现故障的依赖方响应而形成任务积压，线程资源无法释放，最终导致自身服务的瘫痪，进一步甚至出现故障的蔓延最终导致整个系统的瘫痪。如果这样的架构存在如此严重的隐患，那么相较传统架构就更加的不稳定。为了解决这样的问题，因此产生了断路器等一系列的服务保护机制。

在Spring Cloud Hystrix中实现了线程隔离、断路器等一系列的服务保护功能。它也是基于Netflix的开源框架 Hystrix实现的，该框架目标在于通过控制那些访问远程系统、服务和第三方库的节点，从而对延迟和故障提供更强大的容错能力。Hystrix具备了服务降级、服务熔断、线程隔离、请求缓存、请求合并以及服务监控等强大功能。

- 服务熔断: 服务端进行, 一般是某个服务故障或者异常引起, 当某个异常条件触发, 直接熔断整个服务, 而不是一直等待服务超时
- 服务降级: 一般从整体负荷考虑, 当某个服务熔断之后, 服务器将不再被调用, 此时客户端可以准备一个本地的fallback回调, 返回一个缺省值

举个例子, 小明打电话, 每次打不通就换备用手机来打=> 服务降级  , 第一次打不通, 就暂时停用主力手机, 采用备用手机=> 服务熔断

服务熔断一般是由于某个服务故障引起, 而服务降级一般是从整体符合考虑

熔断是一个框架级的处理, 每个微服务都需要, 而降级一般需要对业务有层级之分(高级服务负荷大/出现故障时启用低级服务作为备用逻辑, 个人理解)



## 2. hystrix的使用

### 2.1 引入maven依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```



### 2.2 在应用主类中使用`@EnableCircuitBreaker`或`@EnableHystrix`注解开启Hystrix的使用

```java
package com.nacht;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.SpringCloudApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

/**
 * @author Nacht
 * Created on 2019/9/9
 */
@SpringCloudApplication
public class EurekaRibbonHystrixConsumer {
    public static void main(String[] args) {
        SpringApplication.run(EurekaRibbonHystrixConsumer.class,args);
    }
}
```



注意@SpringCloudApplication注解中包含了@EnableCircuitBreaker注解

```java
/**
 * @author Spencer Gibb
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker
public @interface SpringCloudApplication {

}
```



### 2.3 修改controller, 增加降级/熔断逻辑

```java
package com.nacht.controller;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.loadbalancer.LoadBalancerClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

/**
 * @author Nacht
 * Created on 2019/9/9
 */
@RestController
public class ConsumerController {

    @Autowired
    RestTemplate restTemplate ;
    @RequestMapping("/hello")
    @HystrixCommand(fallbackMethod = "fallback")
    public String getProducerString(){
        String url = "http://EUREKA-PRODUCER/hello";
        return restTemplate.getForObject(url, String.class);
    }
    private String fallback(){
        return "fuck you !";
    }
}
```



### 2.4 修改http://EUREKA-PRODUCER/hello下游服务的提供逻辑

在响应请求之前, 加上Thread.sleep(5000L)的逻辑, 让返回相应延迟5s



### 2.5 尝试调用



启动服务和下游服务, 用postman进行请求, 由于一开始调用的下游服务只是延长了响应时间, 所以只是触发了服务降级逻辑, 每一次的调用都会尝试请求下游服务, 在hystrix指定的超时时间内没收到回应之后调入fallback函数返回, (小明每次打电话先用主机, 失败超时后再用备机), 此时实验观测结果为每次调用1s后返回fallback内容



停止下游服务进程后, 由于hystrix判断下游服务产生故障, 触发熔断器, 此时再进行调用不会再去请求下游服务, 而是直接返回fallback内容(小明用主机呼叫失败之后只使用备机, 不再使用主机), 此时实验观测结果为每次调用5ms后返回fallback内容



### 3. 熔断逻辑的解释

这是官方的解释, 比我拙劣的表达好多了

当我们把服务提供者`EUREKA-PRODUCER`中加入了模拟的时间延迟之后，在服务消费端的服务降级逻辑因为hystrix命令调用依赖服务超时，触发了降级逻辑，但是即使这样，受限于Hystrix超时时间的问题，我们的调用依然很有可能产生堆积。

这个时候断路器就会发挥作用，那么断路器是在什么情况下开始起作用呢？这里涉及到断路器的三个重要参数：快照时间窗、请求总数下限、错误百分比下限。这个参数的作用分别是：

- 快照时间窗：断路器确定是否打开需要统计一些请求和错误数据，而统计的时间范围就是快照时间窗，默认为最近的10秒。
- 请求总数下限：在快照时间窗内，必须满足请求总数下限才有资格根据熔断。默认为20，意味着在10秒内，如果该hystrix命令的调用此时不足20次，即时所有的请求都超时或其他原因失败，断路器都不会打开。
- 错误百分比下限：当请求总数在快照时间窗内超过了下限，比如发生了30次调用，如果在这30次调用中，有16次发生了超时异常，也就是超过50%的错误百分比，在默认设定50%下限情况下，这时候就会将断路器打开。

那么当断路器打开之后会发生什么呢？我们先来说说断路器未打开之前，对于之前那个示例的情况就是每个请求都会在当hystrix超时之后返回`fallback`，每个请求时间延迟就是近似hystrix的超时时间，如果设置为5秒，那么每个请求就都要延迟5秒才会返回。当熔断器在10秒内发现请求总数超过20，并且错误百分比超过50%，这个时候熔断器打开。打开之后，再有请求调用的时候，将不会调用主逻辑，而是直接调用降级逻辑，这个时候就不会等待5秒之后才返回fallback。通过断路器，实现了自动地发现错误并将降级逻辑切换为主逻辑，减少响应延迟的效果。

在断路器打开之后，处理逻辑并没有结束，我们的降级逻辑已经被成了主逻辑，那么原来的主逻辑要如何恢复呢？对于这一问题，hystrix也为我们实现了自动恢复功能。当断路器打开，对主逻辑进行熔断之后，hystrix会启动一个休眠时间窗，在这个时间窗内，降级逻辑是临时的成为主逻辑，当休眠时间窗到期，断路器将进入半开状态，释放一次请求到原来的主逻辑上，如果此次请求正常返回，那么断路器将继续闭合，主逻辑恢复，如果这次请求依然有问题，断路器继续进入打开状态，休眠时间窗重新计时。

通过上面的一系列机制，hystrix的断路器实现了对依赖资源故障的端口、对降级策略的自动切换以及对主逻辑的自动恢复机制。这使得我们的微服务在依赖外部服务或资源的时候得到了非常好的保护，同时对于一些具备降级逻辑的业务需求可以实现自动化的切换与恢复，相比于设置开关由监控和运维来进行切换的传统实现方式显得更为智能和高效。