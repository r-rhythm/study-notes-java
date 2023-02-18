# [[SpringBoot]]
# HelloWorld
## 手动
- **添加父工程坐标**
   ```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
    </parent>
    
    ```

- 添加 web 启动器,maven 工程放在 project 里面
   ```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    
    ```

- 创建启动类
  - `XxxApplication`
  - 在类上面使用 `@SpringBootApplication` 注解声明当前项目为 Spring Boot 的应用程序
- 在 main 方法内调用 `SpringApplication.run(Clazz,args)`
- 注意,启动类会自动扫描当下包及其子包,**其他层需要放到这个包下**
- `@RestController = @ResponseBody + @Controller`

## 自动
- 选择 Spring initialization 工程,进入初始化界面
- 选择要添加的启动器创建工程即可

## SpringBoot 配置文件
- SpringBoot 支持两种配置文件
  - `application*.properties`
  - `application*.yaml`
- properties 与 yaml 可以同时存在
- 配置文件必须以 application 开头

## 何为 yaml
- 数据结构用树形结构呈现，通过缩进来表示层级
- 连续的项目通过减号 ” - ” 来表示
- 键值结构里面的 key/value 对用冒号 ": "来分隔。
- YAML 配置文件的扩展名是 yaml 或 yml
- yaml 与 properties 配置文件除了展示形式不相同以外，其它功能和作用都是一样的

## [[SpringBoot#SpringBoot 常用注解]]
## 自定义启动器

### 自定义返回单个数据源启动器

- 创建项目 `spring-boot-jdbc-starter` 引入依赖

   ```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
    </parent>
    
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>
    
    <dependencies>
        <!--引入spring‐boot‐starter；所有starter的基本配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
    </dependency>
    
        <!--自动配置连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.12</version>
    </dependency>
    
        <dependency>
                <groupId>c3p0</groupId>
                <artifactId>c3p0</artifactId>
                <version>0.9.1.2</version>
    </dependency>
    
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
    ```

- 创建属性配置类

   ```java
    @Component
    @ConfigurationProperties(prefix = "spring.jdbc.datasource")
    public class DataSourceProperties {
        
        private String driverClassName ;
        private String url;
        private String username;
        private String password;
        // 生成set get toString方法
    ```

- 创建自动配置类

   ```java
    @SpringBootConfiguration
    @EnableConfigurationProperties(DataSourceProperties.class)
    public class DataSourceAutoConfiguration {
        
        @Autowired
        private DataSourceProperties dataSourceProperties ;
        
        @Bean
        public DataSource createDataSource(){
            DruidDataSource dataSource = new DruidDataSource();
            dataSource.setDriverClassName(dataSourceProperties.getDriverClassName());
            dataSource.setUrl(dataSourceProperties.getUrl());
            dataSource.setUsername(dataSourceProperties.getUsername());
            dataSource.setPassword(dataSourceProperties.getPassword());
            return dataSource;
        }
    }
    ```

- 编写自动配置属性文件
  - > 这个属性文件创建之后,spring boot 会在运行启动类的时候加载它
  - 在 `resource` 文件夹下面新建 `META-INF/spring.factories`
```properties
    org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.atguigu.autoconfig.DataSourceAutoConfiguration
```

- 做完了之后注意要执行 install , 安装项目

### 使用自定义加载器

- 在其他项目中引入依赖

 ```xml
    <!--  引入自定义加载器-->
            <dependency>
                <groupId>com.atguigu</groupId>
                <artifactId>spring-boot-jdbc-starter</artifactId>
                <version>1.0-SNAPSHOT</version>
            </dependency>
    ```

* **配置连接池信息**

 ```yaml
    spring:
      jdbc:
        datasource:
          driverClassName: com.mysql.jdbc.Driver
          url: jdbc:mysql:///springboot_dev
          username: root
          password: root
    ```

- 测试

 ```java
    @RestController
    public class TestController {

        //这里装配的是该工程所依赖的我自己定义的加载器,用于获取数据库连接
        @Autowired
        private DataSourceProperties dataSourceProperties;

        @GetMapping("/hello")
        public String hello(){

            System.out.println("dataSourceProperties.getDriverClassName() = " + dataSourceProperties.getDriverClassName());

            return "hello SpringBoot Auto";
        }

    }
    ```

### 返回多个数据源

* `@ConditionalOnProperty(value =)`

## SpringBoot常用启动器

### SpringBoot整合MVC

* 添加依赖

```xml
  <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```

- 创建启动类 Application

   ```java
    @SpringBootApplication
    public class Day01SpringbootMvcApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(Day01SpringbootMvcApplication.class, args);
        }
    
    }
    ```

- 创建 POJO

   ```java
    public class User {
        private String username ;
        private String password ;
        private Integer age ;
        private String sex ;
        //get/set,toString...
    }
    ```

- 创建 Controller

   ```java
    @Controller
    @RequestMapping(path = "/user")
    public class UserController {
    
        @RequestMapping(path = "/findAll")
        @ResponseBody
        public List<User> findAll(){
            //查询所有
            List<User> users = new ArrayList<User>();
    
            User user1 = new User();
            user1.setUsername("白落提");
            user1.setPassword("123456");
            user1.setAge(18);
            user1.setSex("男");
    
            User user2 = new User();
            user2.setUsername("卡拉肖克*琳");
            user2.setPassword("654321");
            user2.setAge(18);
            user2.setSex("女");
    
            User user3 = new User();
            user3.setUsername("海问香");
            user3.setPassword("666666");
            user3.setAge(19);
            user3.setSex("女");
    
            users.add(user1);
            users.add(user2);
            users.add(user3);
    
            return users ;
        }
    }
    
    ```



#### 静态资源目录

> 在 springboot 中有一个叫做 ResourceProperties 的类，里面就定义了静态资源的默认查找路径
>
> 我们只要静态资源放在这些目录中任何一个，[[SpringMVC]]都会帮我们处理。

- classpath:/META-INF/resources/
- classpath:/resources/
- classpath:/static/
- classpath:/public

#### 自定义拦截器

- 创建一个拦截器

   ```java
    @Component
    public class MyInterceptor implements HandlerInterceptor {
        
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        
            System.out.println("preHandle 执行了!!");
            
            return true;
        }
        
        @Override
        public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        
            System.out.println("postHandle 执行了!!");
            
        }
        
        @Override
        public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        
            System.out.println("afterCompletion 执行了!!");
            
        }
    }
    ```

- 创建一个拦截器**配置类**

  - 实现 `WebMvcConfigurer` 接口

  - 重写他的 `addInterceptors` 方法

    - `registry.addInterceptor().addPathPatterns("需要拦截的 URL").excludePathPatterns("不拦截的 URL 路径")`

   ```java
    //将这个类标注为SpringBoot配置类
    @SpringBootConfiguration
    public class MyInterceptorConfig implements WebMvcConfigurer {
        
        @Autowired //装配自定义的拦截器
        private MyInterceptor myInterceptor;
        
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            
            //注册拦截器
            registry.addInterceptor(myInterceptor)
                    .addPathPatterns("/user/getAll") //配置需要拦截的URL路径
                    .excludePathPatterns("/*.html"); //配置不拦截的URL
            
        }
    }
    ```

**以前的配置文件在 SpringBoot 中使用配置类来配置,要注意的是,配置类需要使用@SpringBootConfiguration 注解来标注**

## 综合案例

### 环境搭建

- 数据库

   ```sql
    create database springboot character set utf8 ;
    
    use springboot ; 
    
    CREATE TABLE `tb_user` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      `name` varchar(20) NOT NULL,
      `gender` varchar(5) DEFAULT NULL,
      `age` int(11) DEFAULT NULL,
      `address` varchar(32) DEFAULT NULL,
      `qq` varchar(20) DEFAULT NULL,
      `email` varchar(50) DEFAULT NULL,
      `username` varchar(20) NOT NULL,
      `phone` varchar(11) DEFAULT NULL,
      PRIMARY KEY (`id`),
      UNIQUE KEY `user_username_uindex` (`username`)
    ) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8;
    
    INSERT INTO `tb_user` VALUES (1,'黄蓉','女',38,'桃花岛','212223390222','huangrong222@qq.com','huangrong','15600003333'),(2,'黄老邪','男',58,'湖北省武汉市','212223390','huanglaoxie@qq.com','huanglaoxie','15872320405'),(3,'小龙女','男',18,'湖北省荆门市','212223390','xiaolongnv@qq.com','xiaolongnv','15600004444'),(7,'杨过','男',30,'扬州','212223390','yangguo@qq.com','yangguo','15600005555');
    
    ```

- 导入依赖

   ```xml
    <dependencies>
            <!--单元测试启动器-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
            </dependency>
    
            <!--通用mapper启动器依赖-->
            <dependency>
                <groupId>tk.mybatis</groupId>
                <artifactId>mapper-spring-boot-starter</artifactId>
                <version>2.1.5</version>
            </dependency>
            <!--JDBC启动器依赖-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-jdbc</artifactId>
            </dependency>
            <!--mysql驱动-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>8.0.26</version>
            <!--druid启动器依赖-->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid-spring-boot-starter</artifactId>
                <version>1.1.10</version>
            </dependency>
            <!--web启动器依赖-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
    
            <!--spring boot actuator依赖-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-actuator</artifactId>
            </dependency>
    
            <!--编码工具包-->
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
            </dependency>
    
            <!--热部署-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-devtools</artifactId>
                <scope>runtime</scope>
                <optional>true</optional>
            </dependency>
    
            <!-- 配置使用redis启动器 -->
    <!--        <dependency>-->
    <!--            <groupId>org.springframework.boot</groupId>-->
    <!--            <artifactId>spring-boot-starter-data-redis</artifactId>-->
    <!--        </dependency>-->
    
        </dependencies>
    
        <build>
            <plugins>
                <!--spring boot maven插件 , 可以将项目运行依赖的jar包打到我们的项目中-->
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>
    ```

- 编写 application.yaml

   ```yaml
    spring: 
      datasource: #配置数据源的前缀 spring.datasource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql:///springboot
        username: root
        password: root
        type: com.alibaba.druid.pool.DruidDataSource
    mybatis: 
      type-aliases-package: com.atguigu.pojo #配置pojo别名
    ```

- pojo

 ```java
    
    import javax.persistence.Entity;
    import javax.persistence.Id;
    import javax.persistence.Table;
    import java.io.Serializable;
    
    @Entity //指定一个实体类
    @Table(name = "tb_user") //指定数据库中表的名称
    public class User implements Serializable {
        
        @Id //指定主键
        private Integer id;
        private String name;
        private String gender;
        private Integer age;
        private String address;
        private String qq;
        private String email;
        private String username;
        private String phone;
    //...get/set&toString
    }
    ```

- mapper 层
  - 继承 `tk.mybatis.mapper.common.Mapper` 包下的 `Mapper` 它是通用 Mapper 接口,其他接口继承该接口即可
```java
    import com.atguigu.pojo.User;
    import tk.mybatis.mapper.common.Mapper;
    
    /**
     * @author: Wei
     * @date: 2022/9/13 19:58
     */
    public interface UserMapper extends Mapper<User> {
    
    }
    ```

- 测试 Mapper

   ```java
    @SpringBootTest(classes = Application.class)
    public class TestSpringBoot {
        
        @Autowired
        private UserMapper userMapper;
        
        
        @Test
        public void test01(){
        
            List<User> users = userMapper.selectAll();
            for (User user : users) {
                System.out.println("user = " + user);
            }
        
        }
    }
    ```

- service 层

   ```java
    @Service
    public class UserServiceImpl implements UserService {
        
        @Resource
        private UserMapper userMapper;
        
        
        @Override
        public List<User> findAll() {
            return userMapper.selectAll();
        }
        
        @Override
        public User getById(Integer id) {
            return userMapper.selectByPrimaryKey(id);
        }
        
        @Override
        public void saveUser(User user) {
            userMapper.insert(user);
        }
        
        @Override
        public void updateUser(User user) {
            userMapper.updateByPrimaryKey(user);
        }
        
        @Override
        public void deleteById(Integer id) {
            userMapper.deleteByPrimaryKey(id);
        }
    }
    ```

- controller 层
```java
    @RestController
    @RequestMapping("/user")
    public class UserController {
    
        @Autowired
        private UserService userService;
    
        @GetMapping("/getAll")
        public List<User> getAll(){
        
            List<User> all = userService.findAll();
        
            return all; //直接返回Json字符串
        
        }
        
        @GetMapping("/getAll2")
        public Result getAll2(){
            
            List<User> all = userService.findAll();
            
            return Result.ok(all); //响应给ajax
        }
    }
    
```