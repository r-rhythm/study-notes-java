# 简介
> [!question] 什么是 SpringCloud
> 1. SpringCloud 并非一种技术,它提供了**一系列的组件**
> 2. 开发人员使用 SpringCloud 提供的这些组件实现微服务架构
> 3. SpringCloud 很多原生组件大部分已经不再维护,实际使用中一般使用阿里巴巴提供的组件 SpringCloud Alibaba

# 微服务
>[!info] 微服务
>* 微服务是一种**架构风格**
>* 他主张把一个项目按照功能等方式拆分为多个不同的**独立模块**
>* 独立部署,模块之间使用 HTTP 等进行通信
>* 每个服务可以采用不同的技术,数据库等等...
>* 它将一个复杂的单一系统细化为各个完全独立的服务
>* 每个模块都可以脱离其他模块独立运行,独立部署,互不影响

# SpringCloud 和 [[SpringBoot]] 的关系
1. SpringCloud 实现微服务架构必须依赖 SpringBoot
2. SpringBoot 则可以独立使用
3. **SpringBoot 与 SpringCloud 有严格的版本对应关系**

# SpringCloud 中的核心组件
- [[Eureka]] 注册中心
- [[Hystrix]] 服务降级
- [[Ribbon]] 负载均衡
---
- [[Nacos]] 注册 & 配置中心
- [[OpenFeign]] 远程调用
- [[Gateway]] 网关
- [[Sentinel]] 服务降级 / 熔断 / 限流
