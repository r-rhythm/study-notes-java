> [!note] 注册中心
> Eureka由服务(*server*)和客户端(*client*)两部分组成
> 	客户端在Eureka注册服务后,默认30秒发送一次心跳,如果多个周期未收到心跳(默认90秒),会将其从注册中心清除*
> 	[[Nacos#CAP 定理]] AP

> [!tip] 细节补充
> * Eureka由Netflix开发
> * Eureka已经不再维护,只保留原有功能

# Eureka-注册中心

使用注册中心的步骤
1. 创建一个服务作为注册中心
2. 添加依赖
3. 配置eureka ^ad5757
4. 在启动类使用@EnableEurakaServer注解
5. 在需要注册的客户端添加依赖及配置文件,使用@EnableEurekaClient注解

> [!note] 注册中心工作流程
> 1. Service Provider在注册中心注册自己的IP,端口号,接口名称,参数等...
> 2.  Service Consumer通过注册中心获取到Service Provider的消息再进行远程调用


---
1. [[SpringCloud 组件#微服务|什么是微服务]]
2. [[SpringCloud 组件#^a01a10|分布式]]
3. [[SpringCloud 组件#简介|springcloud是什么]]


