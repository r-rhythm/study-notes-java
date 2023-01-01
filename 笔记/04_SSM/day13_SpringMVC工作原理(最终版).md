## SpringMVC工作原理

### 扩展三个对象

> HandlerMapping -> HandlerExecutionChain -> HandlerAdapter

* HandlerAdapter
  * 请求处理器适配器对象
  * 作用:触发请求处理器中的相应方法,,处理请求,做出响应
* HandlerExecutionChain
  * 请求处理器执行链对象,有请求处理器和当前请求处理器对应的所有拦截器组成
  * 作用:通过HandlerExecutionChain获取HandlerAdapter
* HandlerMapping
  * 定义所有请求和请求处理器映射关系
  * 作用:通过它可以获取HandlerExecutionChain对象



### 工作原理[终极]-URL不存在

* 浏览器向服务器发送请求
* 执行DispatcherServlet,并判断当前URL是否在@RequestMapping中映射
  * 映射[URL存在]
  * 未映射[**URL不存在**]
    * 判断是否配置了`default-servlet-handler`
      * 配置了,则提示静态资源未找到
      * 未配置:RequestMapping未找到



### 工作原理[终极]-URL存在

* 浏览器向服务器发送请求
* 执行DispatcherServlet,并判断当前URL是否在@RequestMapping中映射
  * 映射[URL存在]
* 执行DispatcherServlet中的doDispatch方法
  * 初始化扩展的三个对象`HandlerMapping/HandlerExecutionChain /HandlerAdapter `
* 执行HandlerExecutionChain对象中的applyPreHandle(),触发拦截器的第一个方法
* 执行`ha.handle()`触发Controller中的相应方法
* 判断Controller中是否存在异常
  * 有异常:执行异常处理器,并返回ModelAndView
    * 先执行默认异常处理器
    * 默认异常处理器无法处理,执行SimpleMappingExceptionResolver
  * ​    无异常
    * 执行`applyPostHandle()`执行拦截器的第二个方法
* 执行`processDIspatcherResault()`进入ViewResolver&View的处理阶段
  * ViewResolver将View从ModelAndView中解析出来
  * View进行视图渲染阶段
    * SpringMVC默认解决了响应乱码`ThymeleafView`中
    * 将数据共享到request域中`WebEncoding.... 783行`
* 执行afterCompletion