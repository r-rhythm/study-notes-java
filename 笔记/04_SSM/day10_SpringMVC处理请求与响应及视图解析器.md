## SpringMVC 处理请求数据

### SpringMVC 默认处理数据的方式

* 默认情况下 SpringMVC 可以将参数名与方法中的形参名一致的参数自动入参

* ```xml
  <a th:href="@{/requestParamsDefault(stuName='liuwei',stuId=3)}">测试RequestParamsDefault</a> 
  ```

* ```java
  /**
       * 测试默认入参[参数名与形参名一致,自动入参]
       *
       * @param stuName
       * @param stuId
       * @return
       */
      @RequestMapping("/requestParamsDefault")
      public String requestParamsDefault(String stuName, Integer stuId) {
          
          System.out.println("stuName = " + stuName);
          System.out.println("stuId = " + stuId);
          
          return SUCCESS;
          
      }
  ```

  

###  @RequetParam 注解

* 当请求参数名称与形参名称不一致时, 可以使用@RequestParam 注解自动入参

* name 定义指定的请求参数名称, 绑定到形参中

* value name 的别名

* required 设置这个请求参数是否必须入参, 设置为 false 若为入参则默认值为 null

* defaultValue 设置默认值, 当入参为 null 时将 defaultValue 入参

* ```xml
  <a th:href="@{/requestParams(stuName='Lanj',stuId=1)}">测试@RequestParam入参</a>
  
  ```

* ```java
  /**
       * 测试@RequestParam入参
       *
       * @param name
       * @param id
       * @return
       */
      @RequestMapping("/requestParams")
      public String requestParams(@RequestParam("stuName") String name,
                                  @RequestParam("stuId") Integer id) {
          
          System.out.println("name = " + name);
          System.out.println("id = " + id);
          return SUCCESS;
      }
  ```

  

###  SpringMVC-POJO 入参

* 当请求参数名与 POJO 属性名一致时, 可直接入参

* ```html
  <h3>测试POJO入参</h3>
  <form th:action="@{/saveEmployee}">
      员工姓名:<input type="text" name="lastName"><br>
      员工邮箱:<input type="text" name="email">
      部门名称:<input type="text" name="dept.deptName"><br>
      <input type="submit" value="提交">
  </form>
  ```

* ```java
  /**
       * 测试POJO入参
       *
       * @param employee
       * @return
       */
      @RequestMapping("/saveEmployee")
      public String saveEmployee(Employee employee) {
          System.out.println("employee = " + employee);
          return SUCCESS;
      }
  ```

  



###  SpringMVC 处理请求头

* @RequestHeader

  * 获取请求头信息

  * 属性与上面的各类一致 `name\value...`

  * ```html
    <h3>测试请求头</h3>
    <a th:href="@{/testRequestHeader}">测试请求头参数</a>
    ```

  * ```java
    /**
         * 测试请求头参数入参
         *
         * @param ref
         * @param ua
         * @return
         */
        @RequestMapping("/testRequestHeader")
        public String testRequestHeader(@RequestHeader(value = "Referer", required = false) String ref,
                                        @RequestHeader("User-Agent") String ua) {
            
            System.out.println("ref = " + ref);
            System.out.println("ua = " + ua);
            return SUCCESS;
            
        }
    ```

    

###  SpringMVC 处理 Cookie 信息

* @CookieValue

  * 通过 Cookie 的 name 获取 cookie 的 value

  * ```java
    /**
         * 测试获取cookie的value
         *
         * @param un
         * @return
         */
        @RequestMapping("/getCookie")
        public String getCookie(@CookieValue("username") String un) {
            
            System.out.println("un = " + un);
            
            return SUCCESS;
        }
    ```

    

### SpringMVC使用ServletAPI

* SpringMVC中使用原生ServletAPI对象,直接将对象入参即可

* 原生常用对象

  * HttpServletRequest
  * HttpServletResponse
  * HttpSession
  * ...

* ```java
  /**
       * 测试原生Servlet对象
       *
       * @param response
       * @return
       */
      @RequestMapping("/crateCookie")
      public String crateCookie(HttpServletResponse response) {
          
          Cookie cookie = new Cookie("username", "admin");
          
          response.addCookie(cookie);
          return SUCCESS;
      }
  ```



##  处理响应数据

### SpringMVC 处理响应数据的两种方式

* 使用 ModelAndView 对象作为**方法返回值**类型, 处理响应数据

  * ```java
    /**
         * 测试ModelAndView
         *
         * @return
         */
        @RequestMapping("/testModelAndView")
        public ModelAndView testModelAndView() {
            
            ModelAndView mv = new ModelAndView();
            
            //设置模型数据[默认request域]
            mv.addObject("username", "liuwei");
            
            //设置视图名称[默认转发]
            mv.setViewName("response_success");
            return mv;
        }
    ```

    

* 使用Map,Model,ModelMap作为**方法入参**,处理响应数据

  * ```java
    /**
         * 测试ModelMap入参处理响应数据
         * 
         * public class ModelMap extends LinkedHashMap<String, Object>
         *
         * @param modelMap
         * @return
         */
        @RequestMapping("/testModelParam")
        public String testModelParam(ModelMap modelMap) {
            
            modelMap.put("stuName", "admin");
            return "response_success";
        }
    ```

    

###  源码解析 ModelAndView

* 存储数据模型[Model]及视图对象[View]

### SpringMVC 中使用域对象

* 默认使用 request 域

* 使用其他域

  * seesion

    * 原生 session
    * @SessionAttributes

  * application

  * ```java
    /**
         * 测试将数据共享到session域中,application同理
         * ServletContext application
         *
         * @param session
         * @return
         */
        @RequestMapping("/testSession")
        public String testSession(HttpSession session/*,ServletContext application*/) {
            
            session.setAttribute("username", "liuweiiiiii");
            
            return "response_success";
        }
    ```

    



## SpringMVC请求域响应乱码问题

> 回顾
>
> ​	三行代码解决请求响应乱码问题
>
> * request.setCharacterEncoding("UTF-8")
> * response.setContentType("text/html;charset=UTF-8")

### SpringMVC中请求域响应乱码默认情况

* 请求乱码

* 响应不乱码,SpringMVC底层处理响应乱码问题

### SpringMVC中处理请求乱码问题

* CharacterEncodingFilter

  * 注册该过滤器
  * 为encoding设置初始化参数`encoding=UTF-8`
  * 为forceRequestEncoding设置初始化参数`true`

* ```xml
  <filter>
          <filter-name>CharacterEncodingFilter</filter-name>
          <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  
          <init-param>
              <param-name>encoding</param-name>
              <param-value>UTF-8</param-value>
          </init-param>
          <init-param>
              <param-name>forceRequestEncoding</param-name>
              <param-value>true</param-value>
          </init-param>
      </filter>
  
      <filter-mapping>
          <filter-name>CharacterEncodingFilter</filter-name>
          <url-pattern>/*</url-pattern>
      </filter-mapping>
  ```

  

### CharacterEnodingFilter 源码解析





## SpringMVC 视图及视图解析器

### SpringMVC 工作原理 2

* DispartcherServlet 第 1061 行, 调用 Controller 中的相应方法
* Controller 中无论返回 String 还是 ModelAndView, 最终都会封装为 ModelAndView

### ViewResolver 对象

* 视图解析器对象
* 从 ModelAndView 中将 View 解析出来

### View 对象

* 视图对象
* 视图渲染
  * 将数据共享到域中[request]
  * 跳转路径[转发]



## SpringMVC 中项目实战应用

### 视图控制器

* `<mvc:view-controller>`

  * 将 URL 与逻辑视图名映射

* 加上这个标签之后会导致其他@Controller 的 URL 失效

* 使用 `<mvc:annotation-driven>` 解决其他 URL 失效的问题
```xml
  <!-- 装配视图控制器-->
      <mvc:view-controller path="/" view-name="index"></mvc:view-controller>
      <mvc:view-controller path="/toRequestPage" view-name="doRequestPage"></mvc:view-controller>
      <mvc:view-controller path="/toResponsePage" view-name="doResponsePage"></mvc:view-controller>
  
  
      <!--    
          装配annotation-driven
          作用1:解决[装配view-controller导致其他Controller中URL失效]问题
          作用2:解决[装配default-servlet-handler导致Controller无法正常使用]问题
       -->
      <mvc:annotation-driven></mvc:annotation-driven>
```

  

### SpringMVC 重定向
语法 `return "redirect:/视图名.html"`

```xml
    <!--    解决静态资源加载问题-->
        <mvc:default-servlet-handler></mvc:default-servlet-handler>
    
    
        <!--
            装配annotation-driven
            作用1:解决[装配view-controller导致其他Controller中URL失效]问题
            作用2:解决[装配default-servlet-handler导致Controller无法正常使用]问题
         -->
        <mvc:annotation-driven></mvc:annotation-driven>
    
```

```java
    /**
         * 测试重定向
         *
         * @return
         */
        @RequestMapping("/testSendRedirect")
        public String testSendRedirect() {
            
            System.out.println(" >>> 重定向");
            
            return "redirect:/redirect_success.html";
        }
```


* 源码解析

### SpringMVC 中处理静态资源

* 在 SpringMVC 中默认无法加载静态资源

原因
* Tomcat 服务器默认加载静态资源的组件: DefaultServlet
* SpringMVC 中注册 DispatherServlet 的 URL 将 DefaultServlet 的覆盖掉, 所以导致 DefaultServlet 失效, 无法加载静态资源

* 解决 SpringMVC 中静态加载问题

  * `<mvc:default-servlet-handler>`

  * 注意, 装配这个标签也会导致@Controller 无法正常使用, 也需要配置 `<mvc:annotation-driven>`

```xml
    <!--    解决静态资源加载问题-->
        <mvc:default-servlet-handler></mvc:default-servlet-handler>
    
    
        <!--
            装配annotation-driven
            作用1:解决[装配view-controller导致其他Controller中URL失效]问题
            作用2:解决[装配default-servlet-handler导致Controller无法正常使用]问题
         -->
        <mvc:annotation-driven></mvc:annotation-driven>
```
