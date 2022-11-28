# 1 SpringCloud
## 1.1 一些组件  
- 服务治理 Eureka
- 服务通信 Ribbon
- 服务通信 Feign
- 服务网关 zuul
- 服务容错 Hystrix
- 服务配置 Config
- 服务监控 Actuator
- 服务跟踪 Zipkin

## 1.2 服务治理  
### 1.2.1 概念  
服务治理的核心有三部分组成：服务提供者，服务消费者，注册中心。  
在分布式系统架构中，每个服务在启动时，将自己的信息存储在注册中心，叫做服务注册。  
服务消费者从注册中心获取服务提供者的网络信息，通过该信息调用服务，叫做服务发现。  
SpringCloud的服务治理使用Eureka来实现，Eureka是Netflix开源的机遇REST的服务治理解决方案，SpringCloud集成了Eureka，提供服务注册和服务发现的功能，可以和基于SpringBoot搭建的微服务应用轻松完成整合，开箱即用，Srping Cloud Eureka。

### 1.2.2 Srping Cloud Eureka  
- Eureka Server 注册中心
- Eureka Client 所有要进行注册的微服务通过Eureka Client连接到Eureka Server，完成注册