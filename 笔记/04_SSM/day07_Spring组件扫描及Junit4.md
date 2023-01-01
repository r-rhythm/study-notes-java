## 组件扫描详解

### 排除扫描

* type属性

  * assignable 排除指定的对象(全类名)

    * ```xml
      <!--从注解扫描中排除指定的类-->
          <context:component-scan base-package="com.atguigu.annotation">
              <context:exclude-filter type="assignable" expression="com.atguigu.annotation.service.impl.EmployeeServiceImpl"/>
          </context:component-scan>
      ```

      

  * annotation 排除指定的注解[注解全类名]

    * ```xml
      <!--从扫描中排除指定的注解-->
          <context:component-scan base-package="com.atguigu.annotation">
              <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service"/>
          </context:component-scan>
      ```

      

### 包含扫描

* 注意,包含扫描必须关闭父标签默认设置,将其扫描的其他包改为false

  * ```xml
    <!--设置只扫描指定的类-->
        <context:component-scan base-package="com.atguigu.annotation" use-default-filters="false">
            <context:include-filter type="assignable" expression="com.atguigu.annotation.mapper.impl.EmployeeDaoImpl"/>
        </context:component-scan>
    
    ```

  * ```xml
    <!--设置只扫描指定的类-->
        <context:component-scan base-package="com.atguigu.annotation" use-default-filters="false">
    <!--        <context:include-filter type="assignable" expression="com.atguigu.annotation.mapper.impl.EmployeeDaoImpl"/>-->
            <context:include-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
        </context:component-scan>
    ```

    

## 完全注解开发

* 创建配置类,代替xml配置文件
  * @Configuration 定义当前类为配置类
  * @ComponentScan 设置开启组件扫描
    * basePackages 扫描当前包及其子包
* 容器对象使用 AnnotationConfigApplicationContext(配置类.class)

## Spring集成(整合)Junit4

* 添加spring-test-5.3.1.jar

  * ```xml
    <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-test</artifactId>
                <version>5.3.1</version>
            </dependency>
    ```

    

* 在Junit测试类上添加两个注解即可

  * ```java
    //使用配置类
    //@ContextConfiguration(classes = Config.class)
    
    //配置文件前缀需要添加classpath
    @ContextConfiguration(locations = "classpath:applicationContext_annotation.xml")
    @RunWith(SpringJUnit4ClassRunner.class)
    public class SpringJunitTest {
    
        @Autowired
        EmployeeService employeeService;
        
        @Test
        public void test01(){
        
            employeeService.getAllEmp();
            
        }
    }
    ```
    
    

* 优势

  * 简化了创建容器对象
  * 简化了从容器对象中获取目标对象