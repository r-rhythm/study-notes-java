## 初识SpringMVC

### 概述

* SpringMVC是Spring为控制层提供的一个子框架
* SpringMVC可以使用注解标识普通的POJO成为请求处理器,无需实现任何接口
  * @Controller

### 工作原理简介

* 浏览器向服务器发送请求
* 通过DispatcherServlet将请求委托给Controller
  * DispatcherServlet 前端控制器
  * Controller 请求处理器
* Controller调用Service层,Service层调用Dao层
  * 返回ModelAndView对象
  * ModelAndView对象中存储模型数据和视图对象
* 在DispatcherServlet中,将ModelAndView中的View解析出来
* 响应(跳转路径)



## SpringMVC-HelloWorld

* 创建Web工程[Maven]

* 导入相关jar包及设置打包方式`war`

  * ```xml
    <packaging>war</packaging>
    <dependencies>
            <!--spring-webmvc-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-webmvc</artifactId>
                <version>5.3.1</version>
            </dependency>
    
            <!-- 导入thymeleaf与spring5的整合包 -->
            <dependency>
                <groupId>org.thymeleaf</groupId>
                <artifactId>thymeleaf-spring5</artifactId>
                <version>3.0.12.RELEASE</version>
            </dependency>
    
            <!--servlet-api-->
            <dependency>
                <groupId>javax.servlet</groupId>
                <artifactId>javax.servlet-api</artifactId>
                <version>4.0.1</version>
                <scope>provided</scope>
            </dependency>
        </dependencies>
    ```

    

* 编写配置文件

  * web.xml中注册DispatcherServlet

    * 注册DispatcherServlet的URL,建议设置为 / 或 /*

    * 设置DispatcherServlet优先级`load-on-startup`

    * 设置初始化参数(springmvc配置文件路径)

    * ```xml
      <!-- 注册dispatcherServlet-->
          <servlet>
              <servlet-name>DispatcherServlet</servlet-name>
              <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      
          <!-- 设置springmvc配置文件路径
                  管理springmvc容器对象
          -->
              <init-param>
                  <param-name>contextConfigLocation</param-name>
                  <param-value>classpath:springmvc.xml</param-value>
              </init-param>
      
          <!--设置启动优先级,在开启服务器时创建DispatherServlet-->
              <load-on-startup>1</load-on-startup>
          </servlet>
      
          <!-- 注册DispatcherServlet的URL-->
          <servlet-mapping>
              <servlet-name>DispatcherServlet</servlet-name>
      
          <!--  [推荐使用]或/*-->
              <url-pattern>/</url-pattern>
          </servlet-mapping>
      
          <!--
              /与/*区别
                  /*:匹配当前上下文下的所有路径
                  /:匹配当前上下文下的所有路径[排除jsp]
          -->
      ```

      

  * springmvc.xml

    * 开启组件扫描

    * 装配视图解析器[ThymeleafViewResolver]

    * ```xml
      <!-- 开启组件扫描-->
          <context:component-scan base-package="com.atguigu"/>
      
          <!-- 装配视图解析器[ThymeleafViewResolver]-->
          <bean class="org.thymeleaf.spring5.view.ThymeleafViewResolver" id="viewResolver">
              <!--配置字符集属性-->
              <property name="characterEncoding" value="UTF-8"></property>
              <!--配置模板引擎属性-->
              <property name="templateEngine">
                  <!--配置内部bean-->
                  <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
                      <!--配置模块的解析器属性-->
                      <property name="templateResolver">
                          <!--配置内部bean-->
                          <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
                              <!--配置前缀-->
                              <property name="prefix" value="/WEB-INF/pages/"></property>
                              <!--配置后缀-->
                              <property name="suffix" value=".html"></property>
                              <!--配置字符集-->
                              <property name="characterEncoding" value="UTF-8"></property>
                          </bean>
                      </property>
                  </bean>
              </property>
          </bean>
      ```

      

* 编写Controller层

  * 使用@Controller标示请求处理器类

  * 使用RequestMapping标识请求映射器[方法或类]

  * ```java
    @Controller //将这个pojo标识为一个控制器
    public class ToIndexController {
        
        /**
         * 视图前缀 /WEB-INF/pages/
         * <p>
         * 逻辑视图名称 index
         * <p>
         * 视图后缀 .html
         * <p>
         * 物理视图 /WEB-INF/pages/  index  .html
         *
         * @return
         */
        @RequestMapping("/")
        //用来映射请求,通过它来指定控制器可以处理哪些URL请求
        public String toIndex() {
            //逻辑视图名称
            return "index";
        }
    }
    ```

* 常见问题

  * ClassNotFound 原因
    * jar包未导入成功
    * 部署的war包错误
  * 404
    * 查看所有控制台是否报错,根据错误更改
    * 控制台未发现异常,查看对应视图路径下页面是否存在



## @RequestMapping注解 [ 重要 ]

* @RequestMapping书写位置
  * 类上面 为当前类映射URL
  * 方法上面 为当前方法映射URL
    * 通过URL访问当前方法

### @RequestMapping中属性

* path

  * String[] 
  * 为指定的类或方法定义多个映射的URI(通过不同的URI访问同一个方法)

* value

  * String[]

  * 与path的属性一致,是path的别名

  * ```java
    @RequestMapping({"/value","/path"})
        public String testPath(){
            System.out.println("Hello!World...");
            return "HelloWorld";
        }
    ```

    ```html
    <!--测试path&value-->
    <a href="hello/value">测试path&value</a>
    ```

    

* method

  * RequestMehod[]

  * 为指定的方法或类定义请求方式

    * ```java
      //测试提交方法
          @RequestMapping(value = "/get",method = RequestMethod.GET)
          public String testMethod(){
              System.out.println("Hello!World [GET]...");
              return "HelloWorld";
          }
      ```

      

    * 默认支持RequestMethod内的所有请求方式

    * 如请求方式不支持,报错405

      * ```html
        <!--测试post提交-->
        <form action="hello/get" method="post">
            <input type="submit" value="测试POST">
        </form>
        ```

        

  * 扩展简写语法

    * @GetMapping

    * @PostMapping

      * ```java
        //测试提交方法POST
            @PostMapping("/post")
            public String testMethodPost(){
                System.out.println("Hello!World [POST]...");
                return "HelloWorld";
            }
        ```

        

    * @PutMapping

    * ...

* params

  * String[]

  * 为指定类或方法定义请求参数

    * ```Java
      //测试params
          @RequestMapping(value = "/params",params = "userName")
          public String testParams(){
              System.out.println("Hello!World [Params]...");
              return "HelloWorld";
          }
      ```

      

    * 如为方法定义了指定请求参数,而请求中未携带参数,会报错

      * ```html
        <a href="hello/params?userName=liuwei">测试params</a>
        ```

        

* headers

  * String[]

  * 为指定方法或类定义请求头

    * ```java
      //测试Header
          @RequestMapping(value = "/header",headers = "User-Agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.0.0 Safari/537.36")
          public String testHeader(){
              System.out.println("Hello!World [Header]...");
              return "HelloWorld";
          }
      ```

      

  * 如未携带请求头则报错

### @RequestMapping支持Ant风格
* Ant风格资源地址支持三种匹配符
  * ? 匹配一个任意字符
  * \* 匹配任意数量任意字符
  * \** 匹配多层路径下,任意数量任意字符



## @PathVariable[重点]

### 概述

* 可以获取URL中占位符的参数

  * @RequestMapping注解支持占位符,使用{}

  * ```java
    @DeleteMapping("/student/{stuId}")
        public String delStu(@PathVariable("stuId") Integer stuId) {
            System.out.println("删除成功..." + stuId);
            return SUCCESS;
        }
    ```

  * ```html
    <form th:action="@{/student/123}" method="post">
        <input type="hidden" name="_method" value="DELETE">
        <input type="submit" value="测试DELETE删除">
    </form>
    ```

    

* 属性

  * value 设置占位符名称
  * name 与value作用一致,是value的别名
  * required 标识当前被@PathVariable标识的的参数必须入参[默认值true]



## [[RESTful]]风格的CRUD[重点]

### [[RESTful]]风格概述

### REST与传统风格对比

* REST风格优势
  * 提高网站搜索排名
  * 便于第三方平台对接

### HiddenHttpMethoedFilter过滤器

* 可以将POST请求转换为PUT&DELETE

#### 使用步骤

* 注册HiddenMehodFilter过滤器[web.xml]

  * ```xml
    <!-- 注册HiddenHttpMethodFilter-->
        <filter>
            <filter-name>HiddenHttpMethodFilter</filter-name>
            <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
        </filter>
        <filter-mapping>
            <filter-name>HiddenHttpMethodFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>
    ```

    

* 表单提交

  * 提交方式必须是POST
  * 提交参数必须包含:_method参数
    * 参数数值 PUT&DELETE[不区分大小写]

### RESTful风格CRUD[简易版]

* 增加[POST]

* 删除[DELETE]

* 修改[PUT]

* 查询[GET]

* ```html
  <body>
  <h2>测试REST</h2>
  
  <form th:action="@{/student}" method="post">
      <input type="submit" value="测试添加[POST]">
  </form>
  
  
  <form th:action="@{/student/123}" method="post">
      <input type="hidden" name="_method" value="delete">
      <input type="submit" value="测试通过id删除[DELETE]">
  </form>
  
  <!--<a th:href="@{/student}">测试查询</a>-->
  
  <form th:action="@{/student/456}">
      <input type="submit" value="测试通过id查询[GET]">
  </form>
  
  <form th:action="@{/student}" method="post">
      <input type="hidden" name="_method" value="PUT">
      <input type="submit" value="测试修改[PUT]">
  </form>
  
  </body>
  ```

* ```java
  @Controller
  public class RestController {
      
      private final String SUCCESS = "success";
      
      /**
       * 测试添加
       *
       * @return
       */
      @PostMapping("/student")
      public String addStu() {
          System.out.println("添加成功...");
          return SUCCESS;
      }
      
      
      /**
       * 测试通过id删除
       *
       * @param stuId
       * @return
       */
      @DeleteMapping("/student/{stuId}")
      public String delStu(@PathVariable("stuId") Integer stuId) {
          System.out.println("删除成功...id=" + stuId);
          return SUCCESS;
      }
      
      
      /**
       * 测试修改
       *
       * @return
       */
      @PutMapping("/student")
      public String updateStu() {
          System.out.println("修改成功...");
          return SUCCESS;
      }
      
      
      /**
       * 测试查询
       *
       * @return
       */
      @GetMapping("/student/{stuId}")
      public String getStu(@PathVariable("stuId") Integer stuId) {
          System.out.println("查询" + stuId + "成功...");
          return SUCCESS;
      }
  
  }
  ```

  