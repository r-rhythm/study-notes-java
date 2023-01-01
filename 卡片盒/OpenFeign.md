> [!info] 简介
> * OpenFeign 以更简单的方式实现了[[Ribbon]]+RestTemplate 相同的效果(*远程调用+负载均衡*)
> * 它支持 SpringMVC 中的所有注解

# OpenFeign 使用
1. 引入 OpenFeign 依赖
2. 在**调用端启动类添加@EnableFeignClients**(basePackages={""}) 要确保要调用的接口扫描的包和启动类在同以包下
3. 创建一个接口,并**在接口上方添加@FeignClient**(value="被调用的服务名称",fallback = 服务降级的类.clss)注解;==开启服务降级配置后,必须创建一个实现类来为这些接口实现降级方法==
4. 在这个接口中定义被调用的方法(复制即可)
5. **注意:方法内如果有注解,定义远程调用的方法时必须为注解添加参数**
6. 在 controller 注入这个接口,调用方法即可
> [!warning] 注意
> * 如果调用端与被调用端的包名不同,则需要添加@EnableFeignClients(basepackage="被调用方法包")
> * 远程调用的默认调用时间为 1s,如果超过一秒被调用端还未返回,则直接超时报错
>     * 在调用端配置 ribbon 客户端超时时间(*OpenFeign 默认支持 Ribbon*)

# 远程调用超时问题
- 远程调用默认的超时时间为 1 秒,可以在配置文件中配置远程调用的超时时间
- 超时时间设置建议配置在生产端

#  OpenFeign 日志打印功能
 *日志能显示当前服务运行状态的信息*
 1. 创建配置类
 2. 装配一个返回 Logger.Level 的 bean 方法
 3. 在 yaml 配置文件内添加 logging.level 配置,这个配置为远程调用的接口名: 日志级别

![[SpringBoot#^c9aa67]]

# Feign 调用头信息丢失问题
在使用 feign 进行远程调用的时候, 会产生一个新的代理对象来完成此次调用, 就会丢失掉原来的请求头信息

解决方案:
在调用方自定义一个拦截器实现 `RequestInterceptor` 来进行手动处理, 在每次调用前都先检查以下头文件, 将请求头中的用户信息手动的添加到 header 中, 再进行其他微服务的调用即可

代码示例:
```java
package com.atguigu.gmall.common.interceptor;  
  
import feign.RequestInterceptor;  
import feign.RequestTemplate;  
import org.springframework.stereotype.Component;  
import org.springframework.web.context.request.RequestContextHolder;  
import org.springframework.web.context.request.ServletRequestAttributes;  
  
import javax.servlet.http.HttpServletRequest;  
  
@Component  
public class FeignInterceptor implements RequestInterceptor {  
    @Override  
    public void apply(RequestTemplate requestTemplate){    
            ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();  
            HttpServletRequest request = attributes.getRequest();  
            //  添加header 数据  
            requestTemplate.header("userTempId", request.getHeader("userTempId"));  
            requestTemplate.header("userId", request.getHeader("userId"));  
    }  
}
```
