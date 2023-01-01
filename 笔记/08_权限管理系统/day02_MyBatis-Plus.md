## MyBatis-Plus

![[MyBatis-Plus]]


### 依赖

* ```xml
  <dependency>
      <groupId>com.baomidou</groupId>
      <artifactId>mybatis-plus-boot-starter</artifactId>
      <version>3.4.1</version>
  </dependency>
  ```

### 配置文件

* application.yaml

  ```yaml
    spring:
      application:
        name: service-system
      profiles:
        active: dev
    ```

* application-dev.yaml

  * ```yaml
    server:
      port: 8800
    #配置在控制台输出MyBatisPlus执行的Sql语句
    mybatis-plus:
      configuration:
        log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    spring:
      datasource:
        driver-class-name: com.mysql.cj.jdbc.Driver
        username: root
        password: root
        url: jdbc:mysql:///guigu-auth
    ```



### 启动类

创建一个启动类`ServiceAuthApplicaiton`

使用`@MapperScan`注解扫描Mapper

 ```java
  @SpringBootApplication //标注这个类为SpringBoot启动类
  @MapperScan(basePackages = "com.atguigu.system.mapper") //这里到包名不到类名
  public class ServiceAuthApplication {
  
  }
  ```



### Mapper类

在service-system下创建`SysRoleMapper`接口继承BaseMapper,这是Mybatis-Plus提供的默认Mapper接口。

指定泛型为`此工程内的SysRole`

 ```java
  import com.atguigu.model.system.SysRole;
  import com.baomidou.mybatisplus.core.mapper.BaseMapper;
  
   * @author: Wei
   * @date: 2022/9/16 9:26
   */
  public interface SysRoleMapper extends BaseMapper<SysRole> {
  
  }
```


### 测试类

新建测试类进行测试 **测试类的包名要与main的相同**

* `selectList` 查询所有 形参`query Wrapper`为查询条件,没有查询条件时传入null

 ```java
  package com.atguigu.system.test; //这个包的结构要与main相同
  
  @SpringBootTest
  public class RoleMapperTest {
      
      
      @Resource //用这个注解可以解决自动装配报错问题
      private SysRoleMapper sysRoleMapper;
      
      
      /**
       * 测试查询所有
       */
      @Test
      public void testGetAll(){
      
          List<SysRole> sysRoles = sysRoleMapper.selectList(null); //条件为空
          /*
          *   mybatis-plus发送的sql
          *   SELECT id,role_name,role_code,description,create_time,update_time,is_deleted FROM sys_role WHERE 		*	is_deleted=0
          * 
          * */
          
          for (SysRole sysRole : sysRoles) {
              System.out.println("sysRole = " + sysRole);
          }
      
      }
      
      /**
       * 测试添加
       */
      @Test
      public void testSave(){
       
          //INSERT INTO sys_role ( role_name, role_code, description ) VALUES ( ?, ?, ? )
          sysRoleMapper.insert(new SysRole("工程测试","101","负责测试"));
          
      }
      
  }
  ```

  **注意：**

  IDEA在sysRoleMapper处报错，因为找不到注入的对象，因为类是动态创建的，但是程序可以正确的执行。

  为了避免报错，可以在 mapper 层 的接口上添加 @Repository 或直接使用 @Resource 代替 @Autowired。



## CRUD测试

### 测试添加 insert

```java
/**
     * 测试添加
     */
    @Test
    public void testSave(){
        
        /*
        * INSERT INTO sys_role ( role_name, role_code, description ) VALUES ( ?, ?, ? )
        * 
        * */
        sysRoleMapper.insert(new SysRole("角色管理员","role","负责角色管理"));
        
    }
```



### 测试更新 update

```java
/**
     * 测试更新
     */
    @Test
    public void testUpdate(){
    
        SysRole sysRole = new SysRole();
        sysRole.setId(12L);
        sysRole.setRoleName("编剧");
        
        /*
        * UPDATE sys_role SET role_name=? WHERE id=? AND is_deleted=0
        * */
        sysRoleMapper.updateById(sysRole);
    
    }
```



### 批量查询及批量删除

* 批量查询

  * 创建一个List

  * `mapper.selectBatchIds(List list)`

  * 测试

    ```java
    /**
         * 测试批量查询
         */
        @Test
        public void testBatchSave(){
        
            ArrayList<Long> ids = new ArrayList<>();
            ids.add(1L);
            ids.add(2L);
            ids.add(8L);
        
            /*
            * SELECT 
            *   id,role_name,role_code,description,create_time,update_time,is_deleted 
            * FROM 
            *   sys_role 
            * WHERE 
            *   id IN ( ? , ? , ? ) AND is_deleted=0
            * */
            List<SysRole> sysRoles = sysRoleMapper.selectBatchIds(ids);
            
            for (SysRole sysRole : sysRoles) {
                System.out.println("sysRole = " + sysRole);
            }
            
        }
    ```

    

* 批量删除 **项目的删除都是逻辑删除**

  * `mapper.deleteBatchIds(List list)`

[[MyBatis-Plus]]



[[MyBatis-Plus]]



## Controller层

### 定义统一返回结果对象

> 项目中我们会将响应封装成json返回，一般我们会将所有接口的数据格式统一， 使前端(iOS Android, Web)对数据的操作更一致、轻松。
>
> 一般情况下，统一返回数据格式没有固定的格式，只要能描述清楚返回的数据状态以及要返回的具体数据就可以。但是一般会包含状态码、返回消息、数据这几部分内容

* 此项目中统一返回自定的 Result



### Knife4j

[文档地址]: https://doc.xiaominfo.com/

> knife4j是为Java MVC框架集成Swagger生成Api文档的增强解决方案。
>
> 用于生成API文档及**调试后台所写的方法**

#### Swagger介绍

> 前后端分离开发模式中，api文档是最好的沟通方式。
>
> Swagger 是一个规范和完整的框架，用于生成、描述、调用和可视化 [[RESTful]] 风格的 Web 服务。
>
> 1、及时性 (接口变更后，能够及时准确地通知相关前后端开发人员)
>
> 2、规范性 (并且保证接口的规范性，如接口的地址，请求方式，参数及响应格式和错误信息)
>
> 3、一致性 (接口信息一致，不会出现因开发人员拿到的文档版本不一致，而出现分歧)
>
> 4、可测性 (直接在接口文档上进行测试，以方便理解业务)

#### 集成Knife4j

* 依赖

  ```xml
  <dependency>
      <groupId>com.github.xiaoymin</groupId>
      <artifactId>knife4j-spring-boot-starter</artifactId>
    	<!--在父工程中已经统一指定版本-->  
  </dependency>
  ```

* 添加knife4j配置类

  * 在service-util创建knife4j配置类`Knife4jConfig`,引入knife4j依赖

  * 使用`@SpringBootConfiguration @EnableSwagger2mvc`两个注解声明他为配置类 \<**注意,他要写在启动类相同的包下,否则无法扫描到**>

  * 代码

    ```java
    @Configuration
    @EnableSwagger2WebMvc
    public class Knife4jConfig {
        
        @Bean
        public Docket adminApiConfig(){
            List<Parameter> pars = new ArrayList<>();
            ParameterBuilder tokenPar = new ParameterBuilder();
            tokenPar.name("token")
                    .description("用户token")
                    .defaultValue("")
                    .modelRef(new ModelRef("string"))
                    .parameterType("header")
                    .required(false)
                    .build();
            pars.add(tokenPar.build());
            //添加head参数end
            
            Docket adminApi = new Docket(DocumentationType.SWAGGER_2)
                    .groupName("adminApi")
                    .apiInfo(adminApiInfo())
                    .select()
                    //只显示admin路径下的页面
                    .apis(RequestHandlerSelectors.basePackage("com.atguigu"))
                    .paths(PathSelectors.regex("/admin/.*"))
                    .build()
                    .globalOperationParameters(pars);
            return adminApi;
        }
        
        private ApiInfo adminApiInfo(){
            
            return new ApiInfoBuilder()
                    .title("后台管理系统-API文档")
                    .description("本文档描述了后台管理系统微服务接口定义")
                    .version("1.0")
                    .contact(new Contact("qy", "http://atguigu.com", "493290402@qq.com"))
                    .build();
        }
        
    }
    ```



### Controller层添加注解

* @Api(tags = "角色管理") 在API中描述这个控制层的作用

* @ApiOperation("方法描述")在API内描述这个方法

  ```java
  @Api(tags = "角色管理") //在API中描述这个控制层的作用
  @RestController
  @RequestMapping("/admin/system/sysRole")
  public class SysRoleController {
      
      @Resource
      private SysRoleService sysRoleService;
      
      /**
       * 测试添加
       *
       * @return
       */
      @ApiOperation("添加信息")//在API内描述这个方法
      @PostMapping("/save")
      public Result saveSysRole(@RequestBody SysRole sysRole) {
          
          sysRoleService.save(sysRole);
          
          return Result.ok();
      }
  }
  ```

  [knife4j地址]: http://localhost:8800/doc.html

  

![[MyBatis-Plus#MyBatis-Plus分页]]



## 创建异常处理器

* service-util下system下创建handler包

* 该包下创建一个全局的异常处理器`GlobalExceptionHandler` 

* 使用**@ControllerAdvice**注解声明当前类为全局的异常处理器类

* 使用@ResponseBody注解

* 创建具体处理异常的方法,返回Result

* 使用@ExceptionHandler(异常类型.Clazz)标识此方法可以处理那种类型的异常

```java
  /**
   * 全局异常处理类
   *
   * @author: Wei
   * @date: 2022/9/16 23:37
   */
  
  @ControllerAdvice
  public class GlobalExceptionHandler {
      
      
      @ExceptionHandler(Exception.class)
      @ResponseBody
      public Result error(Exception e) {
          
          e.printStackTrace();
          return Result.fail(e.getMessage());
          
      }
      
  }
  ```



