# 简介
> [!info] Gateway 简介
> 1. 充当客户端与服务端的中间过滤器(链)角色,它的底层设计就是多个过滤器链
> 3. 之前使用的[[Nginx]]起到的就是网关的作用
> 4. Gateway 基于 Spring5.x + SpringBoot2.x 和 Project Reactor 等技术

# 网关的作用
 1. 限流保护
 2. 熔断降级
 3. 请求转发(过滤)
 4. 统一鉴权
 5. 反向代理
 6. 负载均衡
 7. 解决[[跨域]]
 8. ....

# 网关三个核心概念
1. 路由(*Route*):路由允许我们通过不同的 URL 访问不同的内容,它是构建网关的基本模块，由 ID，目标 URI，一系列的断言和过滤器组成
2. 断言(*Predicate*):可以**匹配 HTTP 请求中的所有内容**（例如请求头或请求参数），如果请求与断言相匹配则进行路由
3. 过滤(*filter*):使用过滤器，可以在请求被路由前或者之后对请求进行修改。

## 配置动态路由
1. 创建一个网关服务
2. 引入依赖
3. 为网关配置路由及规则并在服务中心注册
4. 在配置文件中开启 discovery 并配置路由规则 url: lb[^lb]://服务名 配置动态路由,实现负载均衡

## 断言规则
- \- Before 格林威治时间:表示在这个时间之前能访问
- \- After 表示这个时间之后才能访问
- \- Path 匹配路径
- \- between 时间区间
- \- method 指定的访问方式
- \- ...

## Filter
自定义全局 GlobalFilter 实现鉴权或判断等等...
- 实现 GlobalFilter(*过滤逻辑*)和 Ordered(*过滤器优先级,0最高*)接口

[^lb]: Load Balance 负载均衡

# 网关执行流程 
![[Gateway-流程图.png]]
1. 客户端向网关发出请求,网关在 Handler Mapping 中找到与请求匹配的路由
2. 查找都匹配的路由后,将其发送到 Web Handler 
3. Web Handler 通过指定过滤链将请求过滤后发送给服务进行处理返回

# 过滤器配置
## 配置全局过滤器解决[[跨域]]
```java
@Configuration
public class CorsConfig {
    @Bean
    public CorsWebFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        // 允许的请求方式
        config.addAllowedMethod("*");
        // 允许的请求来源
        config.addAllowedOrigin("*");
        // 允许携带的头信息
        config.addAllowedHeader("*");
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource(new PathPatternParser());
        source.registerCorsConfiguration("/**", config);
        return new CorsWebFilter(source);
    }
}
```
还可以在 yml 配置文件中进行配置