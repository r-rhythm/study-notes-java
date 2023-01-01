## Filter&Listener

### Filter(过滤器)

> * Servlet处理请求,响应请求
> * Filter过滤请求,过滤响应

* `doFilter()过滤请求与响应`
* `FilterChain()封装Filter链信息`(多个Filter)
  * `chain.doFilter()放行请求`

**过滤器在web.xml注册时url要设置为需要过滤的Servlet,一般过滤器url不会设置自己**

#### 单个过滤器工作原理

* 浏览器向服务器发送请求
* 先执行过滤器中放行前的代码
* 放行请求,交由Servlet处理
* 执行响应过滤代码
* 响应

#### 多个过滤器的工作原理

> 过滤器的执行顺序由注册时**(filter-mapping)**的顺序决定

* 浏览器向服务器发送请求
* 执行Filter1的放行前代码
* 执行Filter2的放行前代码
* 执行Servlet
* 执行Filter2的放行后代码
* 执行Filter1的放行后代码
* 响应

#### 生命周期

* `Filter()`
  * 在启动服务器时调用构造器创建Filter对象
  * 在整个生命周期中执行一次
* `init()`
  * 同上
* `doFilter()`
  * 请求一次执行一次
* `destroy()`
  * 关闭服务器时执行

#### Filter中URL匹配规则

* 精确匹配

  * > 同时有URL匹配和名称匹配的,会先执行URL匹配再执行名称匹配

  * URL匹配

  * 名称匹配

* 模糊匹配

  * > 匹配的URL中包含符号[*]的规则成为模糊匹配

  * 前置模糊

  * 后置模糊

#### Filter中HttpFilter

#### 基于注解创建Filter



## Listener(监听器)

* 监听对象
  * `HttpServletRequest`,`HttpSession`,`ServletContext`等
* 监听这些对象的创建及销毁时机,对象中的属性变化等情况



## 总结服务器端三大组件

#### 三大组件的加载顺序

> Listener  ->  Filter -> Servlet

### 共性

* 都需要实现某个接口
* 都需要注册后才能使用

