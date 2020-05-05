# springcloud config

## 1. 简介

Spring Cloud Config是Spring Cloud团队创建的一个全新项目，用来为分布式系统中的基础设施和微服务应用提供集中化的外部配置支持，它分为服务端与客户端两个部分。其中服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置仓库并为客户端提供获取配置信息、加密/解密信息等访问接口；而客户端则是微服务架构中的各个微服务应用或基础设施，它们通过指定的配置中心来管理应用资源与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息。Spring Cloud Config实现了对服务端和客户端中环境变量和属性配置的抽象映射，所以它除了适用于Spring构建的应用程序之外，也可以在任何其他语言运行的应用程序中使用。由于Spring Cloud Config实现的配置中心默认采用Git来存储配置信息，所以使用Spring Cloud Config构建的配置服务器，天然就支持对微服务应用配置信息的版本管理，并且可以通过Git客户端工具来方便的管理和访问配置内容。当然它也提供了对其他存储方式的支持，比如：SVN仓库、本地化文件系统。



## 2. springcloud config的使用



### 2.1 新建git仓库

- 在gitlab中新建一个名为springcloud_config的仓库

- 在仓库中添加配置文件config-test.yml



config-test.yml

```yaml
person:
    name: JackDick
info:
    profile: default
```



### 2.2 构建配置中心



首先创建config-server工程, 作为配置中心服务, 添加maven依赖

```xml
<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-config-server</artifactId>
	</dependency>
```



配置文件中指定git地址

```yaml
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: http://192.168.60.133:8899/zhongpc/springcloud_config.git
          username: zhongpc
          password: Sendi123!
server:
  port: 1234
```



编写入口类并启动

```java
package com.nacht;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

/**
 * @author Nacht
 * Created on 2019/9/18
 */
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class,args);
    }
}

```



### 2.3 客户端指定配置中心并读取配置文件

首先创建一个工程 config-client, 添加maven依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
```



配置文件(bootstrap.yml)中指定config-server地址

```yaml
spring:
  application:
    name: config-test
  cloud:
    config:
      uri: http://localhost:1234/
      profile: default  #指定为配置文件中info.profile的值
      label: master
server:
  port: 1235
```



编写入口类

```java
package com.nacht;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @author Nacht
 * Created on 2019/9/18
 */
@SpringBootApplication
public class ConfigClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClientApplication.class,args);
    }
}

```



controller中获取配置在配置中心中的变量值

```java
package com.nacht.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author Nacht
 * Created on 2019/9/18
 */
@RestController
public class HelloController {
    @Value("${person.name}")
    String haha;
    @RequestMapping("/hello")
    public String hello(){
        return haha;
    }
}

```



### 2.4 请求客户端/hello获取变量值

GET http://localhost:1235/hello

返回结果为JackDick(git中配置文件的值)



### 2.5 动态刷新配置

如果要实现客户端配置的动态刷新

首先, 客户端需要引入maven依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



配置文件中添加下面的配置

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```



Controller(获取配置文件变量值的类)添加注解@RefreshScope



在配置更新后, 手动通过POST方式调用刷新接口

POST http://localhost:1235/actuator/refresh

调用接口后配置即会被动态刷新

















































