# 微服务
[[SpringCloud]]

# 分布式
[[分布式]]

# SpringCloud
[[卡片盒/SpringCloud]]

# RestTemplate-远程调用
1. 创建一个RestTemplate配置类
2. 创建返回RestTemplate的方法,并装配到spring中
3. 通过RestTemplate远程调用

# Eureka
[[Eureka]]

# Ribbon
[[Ribbon]]

# OpenFeign
[[OpenFeign]]

# Hystrix
常用于服务降级
[[Hystrix]]


---


# Gateway 网关
1. SpringCloud最早使用的是Netflix的Zuul,随后更换为自研的Gateway
[[Gateway]]

# Sleuth分布式链路追踪器
[[Sleuth]]
## 使用
1. 运行zipkin的jar包启动zipkin
2. 引入jar包,配置yaml配置zipkin&sleuth(*spring.application同级别*)
3. 写一个测试测试方法

# SpringCloudAlibaba
1. Nacos:注册中心和配置中心
2. Sentinal:降级 熔断 限流

#  Nacos
[[Nacos]]

[^lb]: Load Balance 负载均衡

--- 


# Sentinel
## 控制台安装与使用
[[Sentinel#控制台]]

## 流控规则(掌握)
[[Sentinel#流控规则]]

## 降级规则(掌握)
[[Sentinel#降级规则]]

## 热点key限流(理解)
[[Sentinel#热点key限流]]

## 系统规则
[[Sentinel#系统规则 面试常问]]

## @SentinelResource
[[Sentinel#SentinelResource]]

## 规则持久化(基于[[Nacos]]配置中心实现)
[[Sentinel#规则持久化]]


---

# 总结
[[Eureka]]
* Eureka注册中心服务搭建
* 在注册中心如何注册服务

[[Ribbon]]
* Ribbon和Nginx的区别

[[OpenFeign]]
* 如何实现远程调用(重点掌握)
* 设置超时时间

[[Hystrix]]
* 降级/熔断/限流的概念
* 降级的用法
* 熔断的机制

[[Gateway]]
1. 三个核心概念
1. 负载均衡
2. 断言
3. 自定义网关过滤器

[[Sleuth]]
1. 如何表示调用关系

[[Nacos]]
1. 掌握注册中心
2. 读取配置中心的数据
3. 集群搭建

[[Sentinel]]
- Sentinel 降级/熔断/限流