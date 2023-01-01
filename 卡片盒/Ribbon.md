> [!info] 简介
> * Ribbon是由Netflix发布的开源项目
> * 它提供客户端的负载均衡(*Load Balance*)和服务调用
> * 轮询原理,第几次请求%集群总服务器数量*

> [!tip] [[Nginx]]与Ribbon的区别
> 1. Nginx负责服务端的负载均衡,Ribbon是本地负载均衡客户端
> 2. Nginx与Ribbon工作机制不同,Ribbon不直接转发请求,而是从服务中心读取集群配置信息后**缓存到本地**后再转发请求

实现负载均衡
	1. 注册集群
	2. 在配置类中添加@LoadBalance注解开启负载均衡
	3. 在调用端远程调用时直接调用服务名称

# IRule
> [!note] Ribbon核心对象IRule
> 使用这个核心对象来配置负载均衡的策略
> 1. 创建一个Rule规则配置类
> 2. 装配一个返回IRule对象的方法
> 3. 在启动类添加@RibbonClient(name="服务名",configuration=配置类.class)

> [!warning] 
> 这个配置类**必须放到启动类所在包及其子包外面**

