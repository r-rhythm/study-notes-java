## SSM整合

### SSM整合思路

> Spring + SpringMVC + MyBatis

* Spring+SpringMVC
  * Spring与SpringMVC冲突问题
    * 组件扫描
      * SpringMVC开启组件扫描,只扫描Controller层
      * Spring开启组件扫描,排除Controller层
    * 容器对象管理问题
      * SpringMVC容器对象交给**DispatcherServlet**管理
      * Spring容器对象从程序员管理,升级为**监听器管理**[**ContextLoaderListener**]
* Spring+MyBatis
  * Spring与MyBatis冲突问题
    * 关于外部属性文件,数据源,事务的相关管理问题,交给**Spring管理**
  * Spring管理MyBatis核心对象
    * SqlSessionFactory
    * XxxMapper代理对象



### Spring整合SpringMVC

* 创建web工程

* pom.xml导入依赖

  * ```xml
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

* 配置文件

  * web.xml

    * 上下文初始化参数

      * contextConfigLocation:设置spring配置文件路径

      * ```xml
         <!--    设置ContextConfigLocation
                    spring.xml路径
            -->
        
        <context-param>
                <param-name>contextConfigLocation</param-name>
                <param-value>classpath:spring.xml</param-value>
            </context-param>
        ```

        

    * 一个监听器

      * ContextLoaderListener:管理spring容器对象

      * ```xml
        <!--    注册ContextLoaderListener  
                    管理spring容器对象
            -->
            <listener>
                <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
            </listener>
        ```

        

    * 两个过滤器

      * CharacterEncodingFilter:处理POST请求乱码

        * ```xml
          <!--    注册CharacterEncodingFilter
          		处理POST请求乱码
          -->
              <filter>
                  <filter-name>CharacterEncodingFilter</filter-name>
                  <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
                  <init-param>
                      <param-name>encoding</param-name>
                      <param-value>utf-8</param-value>
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

          

      * HiddenHttpMethodFilter:支持PUT&DELETE请求方式

        * ```xml
          <!--    注册HiddenHttpMethodFilter
          		实现REST风格请求
          -->
          
              <filter>
                  <filter-name>HiddenHttpMethodFilter</filter-name>
                  <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
              </filter>
              <filter-mapping>
                  <filter-name>HiddenHttpMethodFilter</filter-name>
                  <url-pattern>/*</url-pattern>
              </filter-mapping>
          ```

          

    * 一个Servlet

      * DispatcherServlet:前端控制器

        * ```xml
          <!--    注册DispatcherServlet
          		前端控制器
          -->
              <servlet>
                  <servlet-name>DispatcherServlet</servlet-name>
                  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
                  <init-param>
                      <param-name>contextConfigLocation</param-name>
                      <param-value>classpath:springmvc.xml</param-value>
                  </init-param>
                  <load-on-startup>1</load-on-startup>
              </servlet>
              <servlet-mapping>
                  <servlet-name>DispatcherServlet</servlet-name>
                  <url-pattern>/</url-pattern>
              </servlet-mapping>
          ```

          

  * springmvc.xml

    * 开启组件扫描,只扫描Controller层

      * ```xml
        <!--    开启组件扫描,只扫描controller
        		注意要关闭上层父包的扫描
        -->
            <context:component-scan base-package="com.atguigu.ssm" use-default-filters="false">
                <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
            </context:component-scan>
        ```

        

    * 装配视图解析器:ThymeleafViewResolver

      * ```xml
        <!--    装配视图解析器-->
            <bean class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
                <property name="characterEncoding" value="UTF-8"></property>
                <property name="templateEngine">
                    <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
                        <property name="templateResolver">
                            <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
                                <property name="characterEncoding" value="UTF-8"></property>
                                <property name="prefix" value="/WEB-INF/pages/"></property>
                                <property name="suffix" value=".html"></property>
                            </bean>
                        </property>
                    </bean>
                </property>
            </bean>
        ```

        

    * 装配视图控制器

      * ```xml
        <!--    装配视图控制器-->
            <mvc:view-controller path="/" view-name="index"></mvc:view-controller>
        ```

        

    * 解决静态资源加载问题

      * ```xml
        <!--    解决静态资源加载问题-->
            <mvc:default-servlet-handler></mvc:default-servlet-handler>
        ```

        

    * 装配Annotation-driven

      * ```xml
        <!--    解决后续问题-->
            <mvc:annotation-driven></mvc:annotation-driven>
        ```

        

  * spring.xml

    * 开启组件扫描,排除Controller层

      * ```xml
        <!--    开启组件扫描,排除controller层-->
            <context:component-scan base-package="com.atguigu.ssm">
                <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
            </context:component-scan>
        ```

        

### Spring整合MyBatis

* 开启组件扫描

* 加载外部属性文件

  * ```xml
    <!--    加载外部文件-->
        <context:property-placeholder location="classpath:db.properties"></context:property-placeholder>
    ```

* 装配DruidDataSource

  * ```xml
    <!--    装配DruidDataSource-->
        <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
            <property name="driverClassName" value="${db.driverClassName}"></property>
            <property name="url" value="${db.url}"></property>
            <property name="username" value="${db.username}"></property>
            <property name="password" value="${db.username}"></property>
        </bean>
    ```

* 装配事务管理器

  * ```xml
    <!--    装配事务管理器-->
        <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource"></property>
        </bean>
    ```

* 开启事务注解支持

  * ```xml
    <!--    开启事务注解支持-->
        <tx:annotation-driven></tx:annotation-driven>
    ```

* 装配SqlSessionFactoryBean(管理SqlSessionFactory)

  * ```xml
    <!--    装配SqlSessionBeanFactory(管理SqlSessionFactory)-->
        <bean class="org.mybatis.spring.SqlSessionFactoryBean">
            <!--        设置数据源-->
            <property name="dataSource" ref="dataSource"></property>
            <!--        设置别名-->
            <property name="typeAliasesPackage" value="com.atguigu.ssm.pojo"></property>
            <!--        加载所有映射文件-->
            <property name="mapperLocations" value="classpath:/mappers/*.xml"></property>
    
    
            <!--        设置mybatis-config中的settings-->
            <property name="configuration">
                <bean class="org.apache.ibatis.session.Configuration">
                    <!--                设置驼峰式自动映射-->
                    <property name="mapUnderscoreToCamelCase" value="true"></property>
                    <!--                开启懒加载-->
                    <property name="lazyLoadingEnabled" value="true"></property>
                    <!--                开启二级缓存-->
                    <property name="cacheEnabled" value="true"></property>
                </bean>
            </property>
    
            <property name="plugins">
                <array>
                    <bean class="com.github.pagehelper.PageInterceptor">
                        <property name="properties">
                            <props>
                                <!-- 设置 reasonable 为 true 表示将页码进行合理化修正。页码的有效范围：1~总页数-->
                                <prop key="reasonable">true</prop>
    
                                <!-- 数据库方言：同样都是 SQL 语句，拿到不同数据库中，在语法上会有差异-->
                                <!-- 默认情况下，按照 MySQL 作为数据库方言来运行-->
                                <prop key="helperDialect">mysql</prop>
                            </props>
                        </property>
                    </bean>
                </array>
            </property>
    
        </bean>
    ```

* 装配MapperScannerConfigurer(管理代理对象)

  * ```xml
    <!--    装配MapperScannerConfigurer-->
        <mybatis-spring:scan base-package="com.atguigu.ssm.mapper"></mybatis-spring:scan>
    ```



### SSM总结

* 导入jar包依赖

  * ```xml
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
    
            <!--mybatis-->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>3.5.7</version>
            </dependency>
    
            <!-- pageHelper 分页插件-->
            <dependency>
                <groupId>com.github.pagehelper</groupId>
                <artifactId>pagehelper</artifactId>
                <version>5.1.8</version>
            </dependency>
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-classic</artifactId>
                <version>1.2.3</version>
            </dependency>
    
            <!--导入druid的jar包-->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.1.10</version>
            </dependency>
    
            <!--导入mysql的jar包-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>8.0.26</version>
            </dependency>
            <!--spring-orm-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-orm</artifactId>
                <version>5.3.1</version>
            </dependency>
            <!--spring-aspects-->
            <!--        <dependency>-->
            <!--            <groupId>org.springframework</groupId>-->
            <!--            <artifactId>spring-aspects</artifactId>-->
            <!--            <version>5.3.1</version>-->
            <!--        </dependency>-->
    
            <!--mybatis-spring整合包-->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis-spring</artifactId>
                <version>2.0.6</version>
            </dependency>
    
    
            <!--lombok 插件-->
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>1.18.12</version>
                <scope>provided</scope>
            </dependency>
    
            <!-- Junit5测试框架-->
            <dependency>
                <groupId>org.junit.jupiter</groupId>
                <artifactId>junit-jupiter-api</artifactId>
                <version>5.7.0</version>
                <scope>test</scope>
            </dependency>
            <!-- Spring5 test-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-test</artifactId>
                <version>5.3.1</version>
            </dependency>
    
        </dependencies>
    ```

* 编写配置文件

  * db.properties

    * ```properties
      #key=value
      db.driverClassName=com.mysql.cj.jdbc.Driver
      db.url=jdbc:mysql://localhost:3306/mybatis
      db.username=root
      db.password=root
      ```

  * logback.xml

    * ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <configuration debug="true">
          <!-- 指定日志输出的位置 -->
          <appender name="STDOUT"
                    class="ch.qos.logback.core.ConsoleAppender">
              <encoder>
                  <!-- 日志输出的格式 -->
                  <!-- 按照顺序分别是：时间、日志级别、线程名称、打印日志的类、日志主体内容、换行 -->
                  <pattern>[%d{HH:mm:ss.SSS}] [%-5level] [%thread] [%logger] [%msg]%n</pattern>
              </encoder>
          </appender>
          <!-- 设置全局日志级别。日志级别按顺序分别是：DEBUG、INFO、WARN、ERROR -->
          <!-- 指定任何一个日志级别都只打印当前级别和后面级别的日志。 -->
          <root level="INFO">
              <!-- 指定打印日志的appender，这里通过“STDOUT”引用了前面配置的appender -->
              <appender-ref ref="STDOUT" />
          </root>
          <!-- 根据特殊需求指定局部日志级别 -->
          <logger name="org.springframework.web.servlet.DispatcherServlet" level="DEBUG" />
          <!--包级别的日志设置 -->
          <logger name="com.atguigu.controller" level="DEBUG"></logger>
          <logger name="com.atguigu.serivce" level="DEBUG"></logger>
          <logger name="com.atguigu.mapper" level="DEBUG"></logger>
      
      </configuration>
      ```

  * mybatis-config.xml[省略]

    * 使用Spring整合之后,可以完全省略掉mybatis的配置文件,若一部分使用spring配置,一部分使用mybatis配置文件配置,可能导致出错

  * web.xml

    * ```xml
      <!--    设置ContextConfigLocation
                  spring.xml路径
          -->
          <context-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>classpath:spring.xml</param-value>
          </context-param>
      
          <!--    注册ContextLoaderListener
                  管理spring容器对象
          -->
          <listener>
              <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
          </listener>
      
      
          <!--    注册CharacterEncodingFilter-->
          <filter>
              <filter-name>CharacterEncodingFilter</filter-name>
              <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
              <init-param>
                  <param-name>encoding</param-name>
                  <param-value>utf-8</param-value>
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
      
      
          <!--    注册HiddenHttpMethodFilter-->
      
          <filter>
              <filter-name>HiddenHttpMethodFilter</filter-name>
              <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
          </filter>
          <filter-mapping>
              <filter-name>HiddenHttpMethodFilter</filter-name>
              <url-pattern>/*</url-pattern>
          </filter-mapping>
      
          <!--    注册DispatcherServlet-->
          <servlet>
              <servlet-name>DispatcherServlet</servlet-name>
              <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
              <init-param>
                  <param-name>contextConfigLocation</param-name>
                  <param-value>classpath:springmvc.xml</param-value>
              </init-param>
              <load-on-startup>1</load-on-startup>
          </servlet>
          <servlet-mapping>
              <servlet-name>DispatcherServlet</servlet-name>
              <url-pattern>/</url-pattern>
          </servlet-mapping>
      ```

  * spring.xml

    * ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
             xmlns:mybatis-spring="http://mybatis.org/schema/mybatis-spring"
             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring.xsd">
      
          <!--    开启组件扫描,排除controller层-->
          <context:component-scan base-package="com.atguigu.ssm">
              <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
          </context:component-scan>
      
      
          <!--    加载外部文件-->
          <context:property-placeholder location="classpath:db.properties"></context:property-placeholder>
      
          <!--    装配DruidDataSource-->
          <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
              <property name="driverClassName" value="${db.driverClassName}"></property>
              <property name="url" value="${db.url}"></property>
              <property name="username" value="${db.username}"></property>
              <property name="password" value="${db.username}"></property>
          </bean>
      
          <!--    装配事务管理器-->
          <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
              <property name="dataSource" ref="dataSource"></property>
          </bean>
      
          <!--    开启事务注解支持-->
          <tx:annotation-driven></tx:annotation-driven>
      
          <!--    装配SqlSessionBeanFactory(管理SqlSessionFactory)-->
          <bean class="org.mybatis.spring.SqlSessionFactoryBean">
              <!--        设置数据源-->
              <property name="dataSource" ref="dataSource"></property>
              <!--        设置别名-->
              <property name="typeAliasesPackage" value="com.atguigu.ssm.pojo"></property>
              <!--        加载所有映射文件-->
              <property name="mapperLocations" value="classpath:/mappers/*.xml"></property>
      
      
              <!--        设置mybatis-config中的settings-->
              <property name="configuration">
                  <bean class="org.apache.ibatis.session.Configuration">
                      <!--                设置驼峰式自动映射-->
                      <property name="mapUnderscoreToCamelCase" value="true"></property>
                      <!--                开启懒加载-->
                      <property name="lazyLoadingEnabled" value="true"></property>
                      <!--                开启二级缓存-->
                      <property name="cacheEnabled" value="true"></property>
                  </bean>
              </property>
      
              <property name="plugins">
                  <array>
                      <bean class="com.github.pagehelper.PageInterceptor">
                          <property name="properties">
                              <props>
                                  <!-- 设置 reasonable 为 true 表示将页码进行合理化修正。页码的有效范围：1~总页数-->
                                  <prop key="reasonable">true</prop>
      
                                  <!-- 数据库方言：同样都是 SQL 语句，拿到不同数据库中，在语法上会有差异-->
                                  <!-- 默认情况下，按照 MySQL 作为数据库方言来运行-->
                                  <prop key="helperDialect">mysql</prop>
                              </props>
                          </property>
                      </bean>
                  </array>
              </property>
      
          </bean>
      
          <!--    装配MapperScannerConfigurer-->
          <mybatis-spring:scan base-package="com.atguigu.ssm.mapper"></mybatis-spring:scan>
      
      </beans>
      ```

  * springmvc.xml

    * ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:context="http://www.springframework.org/schema/context"
             xmlns:mvc="http://www.springframework.org/schema/mvc"
             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">
      
          <!--    开启组件扫描,只扫描controller-->
          <context:component-scan base-package="com.atguigu.ssm" use-default-filters="false">
              <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
          </context:component-scan>
      
          <!--    装配视图解析器-->
          <bean class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
              <property name="characterEncoding" value="UTF-8"></property>
              <property name="templateEngine">
                  <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
                      <property name="templateResolver">
                          <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
                              <property name="characterEncoding" value="UTF-8"></property>
                              <property name="prefix" value="/WEB-INF/pages/"></property>
                              <property name="suffix" value=".html"></property>
                          </bean>
                      </property>
                  </bean>
              </property>
          </bean>
      
          <!--    装配视图控制器-->
          <mvc:view-controller path="/" view-name="index"></mvc:view-controller>
      
          <!--    解决静态资源加载问题-->
          <mvc:default-servlet-handler></mvc:default-servlet-handler>
      
          <!--    解决后续问题-->
          <mvc:annotation-driven></mvc:annotation-driven>
      
      </beans>
      ```

* 测试

  * index.html

    * ```html
      <!DOCTYPE html>
      <html lang="en" xmlns:th="http://www.thymeleaf.org">
      <head>
          <meta charset="UTF-8">
          <title>首页</title>
      </head>
      <body>
      <div align="center">
          <h2>首页</h2>
          <a th:href="@{emps/1}">管理员工信息</a>
      </div>
      </body>
      </html>
      ```

  * controller

    * ```java
      @Controller
      public class EmployeeController {
          
          
          @Autowired
          private EmployeeService employeeService;
          
          /**
           * 测试SSM整合
           *
           * @param pageNum
           * @param map
           * @param request
           * @return
           */
          @GetMapping("emps/{pageNum}")
          public String getAllEmp(@PathVariable("pageNum") Integer pageNum,
                                  Map<String, Object> map,
                                  HttpServletRequest request) {
              
              //查询前分页
              PageHelper.startPage(pageNum, 10);
              
              //获取所有员工
              List<Employee> allEmp = employeeService.getAllEmp();
              
              //封装到PageInfo
              PageInfo<Employee> pageInfo = new PageInfo<>(allEmp, 5);
              String pageRs = PageUtils.getPageInfo(pageInfo, request);
              
              //将pageRs共享到域中
              map.put("pageRs", pageRs);
              
              //共享到域中
              map.put("empList", pageInfo.getList());
              
              return "emps/list";
          }
          
      }
      ```

  * list.html

    * ```html
      <!DOCTYPE html>
      <html lang="en" xmlns:th="http://www.thymeleaf.org">
      <head>
          <meta charset="UTF-8">
          <title>员工信息</title>
      </head>
      <body>
      <div align="center">
          <h2>员工信息</h2>
          <table border="1" width="650px" height="450px">
              <tr>
                  <th>员工编号</th>
                  <th>员工姓名</th>
                  <th>员工邮箱</th>
                  <th>员工薪资</th>
              </tr>
              <tr align="center" th:each="emp : ${empList}">
                  <td th:text="${emp.id}"></td>
                  <td th:text="${emp.lastName}"></td>
                  <td th:text="${emp.email}"></td>
                  <td th:text="${emp.salary}"></td>
              </tr>
          </table>
      </div>
      </body>
      </html>
      ```