> [!info] 工作流程
> 1. DispatchServlet 捕获到客户端发送的请求后,查询请求的 URI 查找对应的映射是否存在,如果存在,返回一个 HandlerExecutionChain 执行链对象.这个执行链对象封装了 HandlerAdapter 和[多个] HandlerInterceptor...
> 2. 依次执行拦截器(*Interceptor*)中的 preHandle() 方法,拦截器返回 true 继续执行,如果某个拦截器返回 false ,则直接执行拦截器的 afterCompletion() 方法(*倒序*)
> 3. 通过 HandlerAdapter 对象触发对应的 Controller 中的方法,如果 Controller 中有异常,执行异常处理器,无异常执行拦截器的 postHandle() 方法并返回 ModelAndView 对象
> 4. 执行 processDispatcherResault() 方法进入视图解析阶段,视图解析器将视图对象从 ModelAndView 中解析出来并渲染
> 5. 将 Model 数据共享到 request 域中
> 6. 执行拦截器的 afterCompletion() 方法

# 常用注解
- @RequestBody 获取 Json 格式的数据,并将他封装到对象中

# 特殊对象
- MultipartFile 对象可以直接获取上传的文件,**要注意的是,这个对象的名称要与前端的 `<input type='file' name='file'/>` 文件上传框的 name 相同**