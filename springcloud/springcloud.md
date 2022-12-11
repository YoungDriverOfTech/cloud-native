# 1. Eureka
## 1.1 项目创建  
### 1.1.1 创建maven项目，修改pom文件如下所示  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.scprac</groupId>
    <artifactId>scpractice</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>eurekaserver</module>
    </modules>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <!--声明为父项目-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.7.RELEASE</version>
    </parent>

    <!--构建web环境-->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--解决JDK9以上没有JAXB api的问题-->
        <dependency>
            <groupId>javax.xml.bind</groupId>
            <artifactId>jaxb-api</artifactId>
            <version>2.3.0</version>
        </dependency>

        <dependency>
            <groupId>com.sun.xml.bind</groupId>
            <artifactId>jaxb-impl</artifactId>
            <version>2.3.0</version>
        </dependency>

        <dependency>
            <groupId>com.sun.xml.bind</groupId>
            <artifactId>jaxb-core</artifactId>
            <version>2.3.0</version>
        </dependency>

        <dependency>
            <groupId>javax.activation</groupId>
            <artifactId>activation</artifactId>
            <version>1.1.1</version>
        </dependency>
    </dependencies>

    <!--构建springCloud-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.SR2</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 1.1.2 创建子模块 eurekaserver  

要让这个模块成为一个注册中心。其配置如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>scpractice</artifactId>
        <groupId>com.scprac</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>eurekaserver</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <!--引入eureka server的相关依赖-->
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
    </dependencies>

</project>
```

- 创建配置文件 

eureka项目中创建application.yml添加eurekaserver相关的配置  

```yml
server:
  port: 8761
eureka:
  client:
    register-with-eureka: false # 是否把本服务也注册尽eureka
    fetch-registry: false # 是否同步其他服务的数据
    service-url:
      defaultZone: http://localhost:8761/eureka/ # 访问eureka的url
```

- 创建启动类   

创建包和启动类在eureka微服务下面  

***@SpringBootApplication*** 注解生命该类是springboot服务的入口
***@EnableEurekaServer*** 声明该类是Eureka Server的微服务，提供服务注册和服务发现的功能，即注册中心

```java
package com.scp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

### 1.1.3 Eureka Client  
- 创建module，pom文件如下  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>scpractice</artifactId>
        <groupId>com.scprac</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>eurekaclient</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```

- 配置文件  
添加eureka client的相关配置  

```yml
server:
  port: 8010
spring:
  application:
    name: provider
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```
`spring.application.name`: 当前服务注册在eureka server上的名称
`eureka.client.service-url.defaultZone`: 注册中心的访问地址
`eureka.instance.preder-ip-address`: 是否将当前服务的IP注册到eureka server

- 创建启动类  
```java
@SpringBootApplication
public class ProvideApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProvideApplication.class, args);
    }
}
```

### 1.1.4 RestTamplate 的使用  
服务之间的相互调用是通过这个服务实现的。    

- 什么是restTemplate？  

是Spring框架停工的基于rest的服务组件，对http请求和响应进行了封装。提供了很多访问rest服务的方法。可以简单代码的开发和配置。

- 怎么使用？  

创建maven模块，pom文件引入响应的依赖. 因为父工程已经引入了boot，所以这个模块什么也不引入也行。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>scpractice</artifactId>
        <groupId>com.scprac</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>resttemplate</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

</project>
```

### 1.1.5 服务消费者  

- 创建自模块，pom如下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>scpractice</artifactId>
        <groupId>com.scprac</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>consumer</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
    </dependencies>

</project>
```

- 配置文件  
```yaml
server:
  port: 8020
spring:
  application:
    name: consumer
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
  instance:
    prefer-ip-address: true
```

- 创建启动类  
```java
package com.scp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}

```

# 2. 服务网关  

 完成一个业务时，客户端需要调用多个微服务。那就需要记住多个不同微服务的地址（URL），不便管理，且每个微服务都要进行安全验证，而且存在跨域的问题。  
 可以在客户端和微服务之间建立一个网关（就类似于nginx），客户端只需要和网关打交道就行了。  

 springcloud集成了zuul组件，通过它实现了服务网关  

 ## 2.1 什么事zuul  
 是netflix提供的一个开源api网关服务器，是客户端和后端所有请求的中间层，对外开发一个api，将所有的请求导入统一的入口，屏蔽了服务端的具体实现逻辑，zuul可以实现反向代理的功能。在网关内部实现动态路由，身份认证，ip过滤，数据监控等功能。  

 ## 2.2 实现  
 - 创建工程  
 ```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>scpractice</artifactId>
        <groupId>com.scprac</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>zuul</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!--    需要注册为微服务    -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>

        <!--    zuul的依赖    -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
    </dependencies>

</project>
 ```

 - 配置文件  
 
 只要访问请求的url中含有/p/, 那么就可以直接访问到provider
 ```yml
server:
  port: 8030
spring:
  application:
    name: gateway
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
# 配置服务和网关的映射
zuul:
  routes:
    provider: /p/**

 ```

 - 创建启动类  

`@EnableZuulProxy`: 包含了`@EnableZuulServer`设置该类是网关的启动类
`@EnableAutoConfiguration`: 可以帮助springboot应用将所有服务条件的`@Configuration`加载到当前springboot创建并且使用的ioc容器中。
 ```java
package com.scp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@EnableZuulProxy
@EnableAutoConfiguration
public class ZuulApplication {

    public static void main(String[] args) {
        SpringApplication.run(ZuulApplication.class, args);
    }
}

 ```

通过网关访问服务  
> http://localhost:8030/p/student/findAll


- Zuul还自带了负载均衡  

修改provider，使其提供两个实例。 启动多个provider，可以在eureka里面看到一共几个实例。


# 3. Ribbon 负载均衡  

## 3.1 什么是Ribbon  

springcloud Ribbon是负载均衡的解决方案，由netflix发布的负载均衡器，spring cloud ribbon是基于netflix ribbon来实现的，是一个用于对http请求进行控制的负载均衡客户端。  

在注册中心对ribbon进行注册之后，ribbon可以基于负载均衡算法，如：轮询，随机，加权轮询，加权随机。帮助服务消费者自动的调用接口。 开发者也可以根据具体需求自定义ribbon的负载均衡算法。  
实际开发中，springcloud ribbon需要结合spring cloud eureka来使用。Eureka server提供所有可以调用的服务提供者的列表，ribbon基于算法从这些服务提供者中，选择要调用的具体实例。  

## 3.2 实现  
- 创建module，添加pom依赖  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>scpractice</artifactId>
        <groupId>com.scprac</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>ribbon</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```

- 配置文件  
```yml
server:
  port: 8040
spring:
  application:
    name: ribbon
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
  instance:
    prefer-ip-address: true
```

- 启动类  
```java
package com.scp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class RibbonApplication {
    public static void main(String[] args) {
        SpringApplication.run(RibbonApplication.class, args);
    }
    
    @Bean
    @LoadBalanced // 这个就是负载均衡器
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

# 4. Feign  
## 4.1 什么是feign  

和ribbon一样，也是由netflix提供的，是一个声明式的，模版化的web service客户端，简化了开发者编写web服务客户端的操作，开发者可以通过简单的借口和注解来调用http api。 Spring cloud feign 整合了ribbon和hystrix，具有可插拔，基于注解，负载均衡，服务熔断等一系列的便捷功能。  

相比较于ribbon + resttamplate的方式，feign可以大大简化代码的开发，feign支持多种注解，包括自身的注解，JAX-RS注解，Spring MVC注解， springcloud对feign进行了优化，整合了ribbon和eureka，从而让feign的使用更加的方便。

## 4.2 ribbon和feign的区别  

- ribbon 是一个通用的http客户端工具，feign是基于ribbon来实现的

## 4.3 feign的特点  
- feign是一个声明式的web service客户端
- 支持feign注解，spring mvc注解，jax-rs注解
- feign基于ribbon来实现的，使用起来更加简单
- feign集成了hystrix，具备服务熔断的功能。

## 4.4 实现  
- 创建module  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>scpractice</artifactId>
        <groupId>com.scprac</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>feign</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```

- 配置文件  
```yml
server:
  port: 8050
spring:
  application:
    name: feign
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```

- 启动类  

```java
package com.scp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableFeignClients
public class FeignApplication {
    public static void main(String[] args) {
        SpringApplication.run(FeignApplication.class, args);
    }
}

```

- 创建声明式接口

```java
package com.scp.feign;

import com.scp.entity.Student;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

import java.util.Collection;

// 访问注册中心中注册的provider服务
@FeignClient(value = "provider")
public interface FeignProviderClient {
    
    // 访问微服务里面的方法，不用写实现
    @GetMapping("/student/findAll")
    public Collection<Student> findAll();

    // 访问微服务里面的方法，不用写实现
    @GetMapping("/student/index")
    public String index();
}

```

- feign的controller
```java
package com.scp.controller;

import com.scp.entity.Student;
import com.scp.feign.FeignProviderClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Collection;

@RestController
@RequestMapping("/feign")
public class FeignHandler {

    @Autowired
    private FeignProviderClient feignProviderClient;

    @GetMapping("/findAll")
    public Collection<Student> findAll() {
        return feignProviderClient.findAll();
    }

    @GetMapping("/index")
    public String index() {
        return feignProviderClient.index();
    }
}

```

## 4.5 Feign的熔断  

如果provider down调了，不能返回white page。 那么feign就要有熔断机制，给客户好的感受，不能让客户看到white page

- 配置文件里面的配置

```yml
server:
  port: 8050
spring:
  application:
    name: feign
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
# -----熔断机制------
feign:
  hystrix:
    enabled: true
# -----熔断机制------
```

- 创建FeignProviderClient的实现类   

这个类来处理容错的处理逻辑，通过@Component注解将Feign Error实例注入IOC中。  
```java
package com.scp.feign.impl;

import com.scp.entity.Student;
import com.scp.feign.FeignProviderClient;
import org.springframework.stereotype.Component;

import java.util.Collection;

@Component
public class FeignError implements FeignProviderClient {

    // 访问这些方法的时候如果出错，就返回这个类里面的内容
    @Override
    public Collection<Student> findAll() {
        return null;
    }

    @Override
    public String index() {
        return "服务器正在维护";
    }
}

```

- 在FeignProviderClient定义处通过@FeignClient的fallback属性设置映射  

```java
@FeignClient(value = "provider", fallback = FeignError.class) // fallback 来配置
public interface FeignProviderClient {

    // 访问微服务里面的方法，不用写实现
    @GetMapping("/student/findAll")
    public Collection<Student> findAll();

    // 访问微服务里面的方法，不用写实现
    @GetMapping("/student/index")
    public String index();
}
```

# 5. Hystrix 容错机制  

## 5.1 特点

在不改变各个微服务调用关系的前提下，针对错误情况进行预先处理。
- 设计原则  
    - 服务隔离机制
    - 服务降级机制
    - 熔断机制
    - 提供实时的监控和报警功能
    - 提供实时配置修改功能

hystrix 数据监控需要结合spring boot actuator组件来使用，actuator提供了对服务的健康监控，数据统计功能。可以通过hystrix-stream节点获取监控的请求数据，同时提供了可视化的监控假面。

## 5.2 实现 

- 创建自模块  
```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>scpractice</artifactId>
        <groupId>com.scprac</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>hystrix</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
            <version>2.0.7.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
    </dependencies>

</project>
```

- 创建配置文件  

```yml
server:
  port: 8060
spring:
  application:
    name: hystrix
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
feign:
  hystrix:
    enabled: true
management:
  endpoints:
    web:
      exposure:
        include: 'hystrix.stream'
```

- 启动类  
```java
package com.scp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableFeignClients
@EnableCircuitBreaker
@EnableHystrixDashboard
public class HystrixApplication {
    public static void main(String[] args) {
        SpringApplication.run(HystrixApplication.class, args);
    }
}
```
`@EnableCircuitBreaker` 声明启用数据监控
`@EnableHystrixDashboard` 声明启用可视化的数据

- controller
```java
package com.scp.controller;

import com.scp.entity.Student;
import com.scp.feign.FeignProviderClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Collection;

@RestController
@RequestMapping("/hystrix")
public class HystrixHandler {

    @Autowired
    private FeignProviderClient feignProviderClient;

    @GetMapping("/findAll")
    public Collection<Student> findAll() {
        return feignProviderClient.findAll();
    }

    @GetMapping("/index")
    public String index() {
        return feignProviderClient.index();
    }
}

```

- feign
```java
package com.scp.feign;

import com.scp.entity.Student;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

import java.util.Collection;

// 访问注册中心中注册的provider服务
@FeignClient(value = "provider")
public interface FeignProviderClient {

    // 访问微服务里面的方法，不用写实现
    @GetMapping("/student/findAll")
    public Collection<Student> findAll();

    // 访问微服务里面的方法，不用写实现
    @GetMapping("/student/index")
    public String index();
}

```

> 使用如下url来监视api的运行情况
>> http://localhost:8060/actuator/hystrix.stream  

> 使用如下url来监视api的运行情况(可视化)
>> http://localhost:8060/hystrix
>> 输入要监控的地址节点，即上面block的url，名字随便取就能看数据监控了

# 6. Spring Cloud本地配置中心  
Spring Cloud Config, 通过服务端可以为多个客户端提供配置服务。Spring Cloud Config可以将配置文件存放在本地，也可以将配置文件存在远程的Git仓库。创建Config Server，通过它管理所有的配置文件。  

## 6.1 本地文件配置系统  

- 创建config server模块  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>scpractice</artifactId>
        <groupId>com.scprac</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>nativeconfigserver</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```

- 配置文件

```yml
server:
  port: 8762
spring:
  application:
    name: nativeconfigserver
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: classpath:/shared
# profiles.active 配置文件的获取方式，native表示从本地去去
# cloud.config.server.native.search-locations 本地配置文件存放的路径
# classpath 指的就是resources，那么在resources文件夹下面创建shared文件夹，然后把配置文件放入shared里面，就能取到配置
```

- shared  
在resources文件夹下面创建shared文件夹，并且创建configclient-dev.yml

```yaml
server:
  port: 8070
foo: foo version 1
```

- 创建启动类  
```java
package com.scp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class NativeConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(NativeConfigServerApplication.class, args);
    }
}

```

`@EnableConfigServer` 声明配置中心。

## 6.2 创建客户端   

读取本地配置中心的配置文件  

- 创建模块

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>scpractice</artifactId>
        <groupId>com.scprac</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>nativeconfigclient</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
    </dependencies>

</project>
```

- 配置文件  
bootstrap.yml 通过这个配置文件来读取本地配置中心的相关信息，这个名字是固定的。  

```yaml
spring:
  application:
    name: configclient
  profiles:
    active: dev
  cloud:
    config:
      uri: http://localhost:8762
      fail-fast: true
# 查找的方法是把name和active用-连接起来，组成一个文件名字，然后去配置中心找。那么组成的名字就是 【configclient-dev】
# cloud.config.url 去哪里找配置文件，配置本地 Config Server的访问路径
# fail-fast 设置客户端优先判断Config Server是否获取正常
```

- 启动类  

```java
package com.scp.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/native")
public class NativeConfigHandler {

    @Value("${server.port}")
    private String port;

    @Value("${foo}")
    private String foo;

    @RequestMapping("/index")
    public String index() {
        return this.port + "-" + this.foo;
    }
}

```

# 7. Spring Cloud远程配置中心  

## 7.1 创建配置文件，以及上传github
在根目录下面创建一个文件夹（注意不是module），然后添加配置如下，再上传到github  
```yml
server:
  port: 8070
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka
spring:
  application:
    name: configclient
```

## 7.2 搭建Config Server

- 新建module  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>scpractice</artifactId>
        <groupId>com.scprac</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>configserver</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
    </dependencies>

</project>
```

- 配置文件  
```yml
server:
  port: 8888
spring:
  application:
    name: configserver
  cloud:
    config:
      server:
        git:
          uri: https://github.com/YoungDriverOfTech/cloud-native-config-client.git
          searchPaths: config
          username: # username
          password: # password
      label: master
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

- 启动类  
```java
package com.scp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}

```

## 7.3 创建Config Client  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>scpractice</artifactId>
        <groupId>com.scprac</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>configclient</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
    </dependencies>

</project>
```

- 配置文件  
创建的文件应该是bootstrap.yml, 因为要去读远程仓库的配置文件   
```yml
server:
  port: 8888
spring:
  application:
    name: configserver
  cloud:
    config:
      server:
        git:
          uri: https://github.com/YoungDriverOfTech/cloud-native-config-client.git
          searchPaths: config
          username: # username
          password: # password
      label: master
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

- 创建启动类  
```java
package com.scp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ConfigClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClientApplication.class, args);
    }
}

```

- Handler 

```java
package com.scp.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/hello")
public class HelloHandler {

    @Value("${server.port}")
    private String port;

    @RequestMapping("/index")
    public String index() {
        return this.port;
    }
}
```

# 8. 服务跟踪  
Spring Cloud Zipkin是可以采集并且更总分布式系统中请求数据的组件。让开发者可以更加直观的监控到请求在各个微服务所耗费的时间等。包含Zipkin Server和Zipkin Client。  

## 8.1 Zipkin服务端搭建  
- 创建模块 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>scpractice</artifactId>
        <groupId>com.scprac</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>zipkin</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-server</artifactId>
            <version>2.9.4</version>
        </dependency>

        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-autoconfigure-ui</artifactId>
            <version>2.9.4</version>
        </dependency>
    </dependencies>
</project>
```

- 配置文件  
```yml
server:
  port: 9090
```

- 启动类  
```java
package com.scp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import zipkin.server.internal.EnableZipkinServer;

@SpringBootApplication
@EnableZipkinServer
public class ZipkinApplication {

    public static void main(String[] args) {
        SpringApplication.run(ZipkinApplication.class, args);
    }
}

```
`@EnableZipkinServer` 声明启动zipkin server

## 8.2 Zipkin客户端搭建  

- 创建module  
