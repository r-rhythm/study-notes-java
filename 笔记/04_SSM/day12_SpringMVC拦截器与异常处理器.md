## SpringMVC中文件上传与下载

### 文件上传

* 引入jar包

  * ```xml
    <!-- commons-fileupload -->
            <dependency>
                <groupId>commons-fileupload</groupId>
                <artifactId>commons-fileupload</artifactId>
                <version>1.4</version>
            </dependency>
    ```

* 装配multipartResolver

  * ```xml
    <!--    装配multiPartResolver
    		id:必须是multipartResolver
    -->
    
        <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
            <property name="defaultEncoding" value="UTF-8"></property>
            <property name="maxUploadSize" value="1024000"></property>
        </bean>
    ```

* 测试

  * ```html
    <form th:action="@{/testFileUpload}" method="post" enctype="multipart/form-data">
        <!--
        文件上传的表单的要求：
            1.请求方式为POST
            2.form表单的enctype的属性值必须指定为multipart/form-data
            3.上传文件的表单项的type属性值为file
        -->
        描述：<input type="text" name="description"><br>
        文件：<input type="file" name="file"><br>
        <input type="submit" value="上传">
    </form>
    ```

  * ```java
    /**
         * 上传文件
         *
         * @param description
         * @param file
         * @param request
         * @return
         * @throws IOException
         */
        @PostMapping("/testFileUpload")
        public String testFileUpload(String description,
                                     MultipartFile file,
                                     HttpServletRequest request) throws IOException {
            
            //获取上传的文件【使用真实路径】
            String realPath = request.getServletContext().getRealPath("/WEB-INF/fileupload");
            
            //获得上传文件的名称
            String filename = file.getOriginalFilename();
            
            //处理文件名,设置唯一性以实现同名文件上传
            String uuid = UUID.randomUUID().toString();
            
            //上传文件
            file.transferTo(new File(realPath + File.separator + uuid + filename));
            
            return "success";
        }
    ```

* 具体实现

  * 应当允许同名文件上传[默认不支持]

    * UUID:是一个32位十六进制的全球唯一随机数

    * ```java
       //处理文件名,设置唯一性以实现同名文件上传
              String uuid = UUID.randomUUID().toString();
              
              //上传文件
              file.transferTo(new File(realPath + File.separator + uuid + filename));
      ```

      

  * 上传的文件应当设置大小上限

    * ```xml
      <property name="maxUploadSize" value="1024000"></property>
      ```



### 文件下载

* 实现步骤

  * 下载文件字节

  * 设置HttpHeader

  * 设置状态码

  * 创建ResponseEntity

  * ```java
    /**
         * 测试文件下载
         *
         * @param fileName
         * @param session
         * @return
         * @throws Exception
         */
        @RequestMapping("/downLoad")
        public ResponseEntity<byte[]> doFileDown(String fileName,
                                                 HttpSession session) throws Exception {
            
            ResponseEntity<byte[]> responseEntity = null;
    
    //        获取需要下载文件的真实路径
            String realPath = session.getServletContext().getRealPath("/WEB-INF/filedownload/" + fileName);
    
    //        1.创建输入流[字节]
            FileInputStream is = new FileInputStream(realPath);
            byte[] bs = new byte[is.available()];
    //        下载字节文件
            is.read(bs);
    
    //        2.设置HttpHeader
            HttpHeaders headers = new HttpHeaders();
    //        设置当前文件为附件,通知浏览器下载资源,不用打开
            headers.add("Context-Disposition", "attachment;filename=" + fileName);
    
    //        处理中文文件名问题
            headers.setContentDispositionFormData("attachment", new String(fileName.getBytes("utf-8"), "ISO-8859-1"));
    
    //        创建ResponseEntity
            responseEntity = new ResponseEntity<>(bs, headers, HttpStatus.OK);
            
            return responseEntity;
        }
    ```

    

## SpringMVC[[拦截器]][重点]

> Filter => Servlet => **Interceptor** => Controller

### [[拦截器]]与过滤器的区别

* 过滤器属于web三大组件之一,拦截器属于框架[SpringMVC,Mybatis(PageInterceptor分页)...]
* 过滤器适用于过滤Servlet,拦截器用于拦截Controller
* 过滤器有两个触发点[执行时机],拦截器存在三个触发点



### 拦截器概述

* 实现拦截器的两种方式

  * 实现`HandlerInterceptor`接口
  * ~~继承`HandlerInterceptorAdapter`类~~

* 实现步骤

  * 实现或~~继承~~

  * 重写三个方法

    * ```Java
      @Component
      public class MyInterceptor1 implements HandlerInterceptor {
          
          
          
          /**
           * 在Controller前执行
           *
           * @param request
           * @param response
           * @param handler
           * @return
           * @throws Exception
           */
          @Override
          public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
              System.out.println("MyInterceptor1 --> preHandle1111()!!");
              return true; //如果为false,DispatcherServlet默认这个拦截器已经做出响应,不再处理它了
          }
          
          /**
           * Controller之后执行
           *
           * @param request
           * @param response
           * @param handler
           * @param modelAndView
           * @throws Exception
           */
          @Override
          public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
              
              System.out.println("MyInterceptor1 --> postHandle222()!!");
              
              
          }
          
          
          /**
           * 视图渲染之后执行
           *
           * @param request
           * @param response
           * @param handler
           * @param ex
           * @throws Exception
           */
          @Override
          public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
              
              System.out.println("MyInterceptor1 --> afterCompletion3333()!!");
              
              
          }
      ```

      

  * springmvc中装配拦截器

    * ```xml
      <mvc:interceptors>
      
              <mvc:interceptor>
                  <mvc:mapping path="/testInterceptor"/>
      <!--            <ref bean="MyInterceptor1"></ref>-->
                  <bean class="com.atguigu.interceptor.MyInterceptor1"></bean>
      
              </mvc:interceptor>
              
          </mvc:interceptors>
      ```



### 拦截器配置

* 全局配置:为所有URL装配拦截器
* 局部配置:为指定的URL装配拦截器



### 拦截器的工作原理

* 单个拦截器
  * 浏览器向服务器发送请求
  * URL匹配
    * 执行拦截器的第一个方法preHandle()
    * 执行Controller相应方法,处理请求做出响应
    * 执行拦截器的第二个方法postHandle()
    * ViewResolver&View执行,视图渲染
    * 执行拦截器的第三个方法afterCompletion()
    * 响应
* 多个拦截器
  * 浏览器向服务器发送请求
  * URL匹配
    * 执行**拦截器1**的第一个方法preHandle()
    * 执行**拦截器2**的第一个方法preHandle()
    * 执行Controller相应方法,处理请求做出响应
    * 执行**拦截器2**的第二个方法postHandle()
    * 执行**拦截器1**的第二个方法postHandle()
    * ViewResolver&View执行,视图渲染
    * 执行**拦截器2**的第三个方法afterCompletion()
    * 执行**拦截器1**的第三个方法afterCompletion()
    * 响应

### preHandle的返回值问题

* 第一个拦截器返回值为false时
  * 只执行第一个拦截器的preHandle()方法,程序终止
* 不是第一个拦截器preHandle()返回false时
  * 执行当前拦截器及之前拦截器的preHandle()
  * 再执行之前拦截器的afterCompletion()

## 异常处理器

### SpringMVC异常处理

* SpringMVC提供`HandlerExceptionResolver接口`作为异常处理器
* 常用的异常处理器
  * `DefaultHandlerExceptionResolver`默认异常处理器
  * `SimpleMappingExceptionResolver`自定义异常处理器

### 默认异常处理器

* `DefaultHandlerExceptionResolver`
* 处理十五种异常,默认**自动开启**
* 即使出现异常,依然会返回ModelAndView对象

### 自定义异常处理器

* `SimpleMappingExceptionResolver`

* 装配`SimpleMappingExceptionResolver`

  * ```xml
    <!-- SimpleMappingExceptionResolver-->
        <bean id="simpleMappingExceptionResolver" class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
            <property name="exceptionMappings">
                <props>
                    <!--  如果发生空指针异常,跳转到这个映射页面-->
                    <prop key="java.lang.NullPointerException">error/nullpointer_exception</prop>
                </props>
            </property>
        </bean>
    ```

    

### 总结

* 如程序中出现异常,则自动优先使用`DefaultHandlerExceptionResolver`
* 如默认异常处理器未处理该异常,再使用`SimpleMappingExceptionResolver`处理