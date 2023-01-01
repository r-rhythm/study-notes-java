## 内部Bean

> 为bean装配一个对象类型的属性值

* `<bean>`

* 在一个bean中完整的定义另外一个bean,称为内部bean

* 内部bean无法在ioc中直接获取,所以一般内部bean一般不会设置id属性

* ```xml
  <!--测试内部bean-->
      <bean id="empLan" class="com.atguigu.ssm.pojo.Employee">
          <constructor-arg name="id" value="2"/>
          <constructor-arg name="name" value="LanJ"/>
          <property name="dept">
              <bean class="com.atguigu.ssm.pojo.Dept">
                  <property name="deptId" value="102"/>
                  <property name="deptName" value="财务部"/>
              </bean>
          </property>
      </bean>
  ```

  



## 集合属性

### List

* 为集合类型的属性注入数值

* ```xml
  <!--测试内部List-->
  <bean id="dept1" class="com.atguigu.ssm.pojo.Dept">
          <property name="deptId" value="101"/>
          <property name="deptName" value="研发"></property>
          <property name="employeeList">
              <list>
                  <ref bean="empLiu"/>
                  <ref bean="empWang"/>
              </list>
          </property>
      </bean>
  ```

  

* ```xml
  <!--测试List-->
      <util:list id="employeeList">
          <ref bean="empLiu"/>
          <ref bean="empWang"/>
      </util:list>
  
      <bean id="dept1" class="com.atguigu.ssm.pojo.Dept">
          <property name="deptId" value="101"/>
          <property name="deptName" value="研发"></property>
          <property name="employeeList" ref="employeeList"/>
      </bean>
  ```

  

### Map

* 为Map类型的属性注入数值

* ```xml
  <!--测试内部map-->
  <bean id="dept2" class="com.atguigu.ssm.pojo.Dept">
          <property name="deptName" value="公关"/>
          <property name="deptId" value="102"/>
          <property name="employeeMap">
              <map>
                  <entry>
                      <key>
                          <value>1</value>
                      </key>
                      <ref bean="empLiu"></ref>
                  </entry>
                  <entry>
                      <key>
                          <value>2</value>
                      </key>
                      <ref bean="empLan"></ref>
                  </entry>
              </map>
          </property>
      </bean>
  ```

  

* ```xml
   <!--测试Map -->
      <util:map id="employeeMap">
          <entry>
              <key>
                  <value>1</value>
              </key>
              <ref bean="empLiu"/>
          </entry>
    
          <entry value-ref="empWang" key="2"/>
      </util:map>
    
      <bean id="dept2" class="com.atguigu.ssm.pojo.Dept">
          <property name="deptName" value="公关"/>
          <property name="deptId" value="102"/>
          <property name="employeeMap" ref="employeeMap"/>
      </bean>
  ```

  

## 管理第三方bean[重要]

### 管理druid数据源步骤

* 导入相关jar包

  * ```xml
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
    ```

    

* 编写properties配置文件,定义属性

  * ```properties
    #key=value
    db.driverClassName=com.mysql.cj.jdbc.Driver
    db.url=jdbc:mysql://localhost:3306/mybatis
    db.username=root
    db.password=root
    ```

    

* 在Spring配置文件中装配`DruidDataSource`

  * 引入外部属性文件

    * location:定义外部属性文件位置
    * classpath:只加载当前模块路径下的资源
    * classpath*:加载同一项目所有模块路径下的资源

  * 装配`DruidDataSource`管理数据源

    * ```xml
      <context:property-placeholder location="druid.properties"/>
      
          <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
              <property name="username" value="${db.username}"/>
              <property name="password" value="${db.password}"/>
              <property name="url" value="${db.url}"/>
              <property name="driverClassName" value="${db.driverClassName}"/>
          </bean>
      ```

  * 测试

    * ```java
      @Test
          public void test01() throws SQLException {
          
              ApplicationContext ioc = new ClassPathXmlApplicationContext("applicationContext_druid.xml");
          
              DruidDataSource dataSource = ioc.getBean("dataSource", DruidDataSource.class);
          
              DruidPooledConnection connection = dataSource.getConnection();
          
              System.out.println("connection = " + connection);
          
          }
      ```

      



## Spring工厂bean[FactoryBean]

> Spring有两种对象,一种普通对象,一种工厂对象
>
> Factory返回的类型不是它本身,而是getObject()方法返回的对象

### 为什么需要FactoryBean

* 使用了Spring框架后,所有对象均交给Spring架构管理,如果项目需要程序员参与到对象的创建的过程中时,此时使用FactoryBean
* FactoryBean的作用就是参与对象的创建过程中

### 使用FactoryBean

* 创建一个实现类实现FactoryBean接口

  * 重写FactoryBean接口的三个方法

    * getObject()参与对象创建的方法

    * getObjectType()设置创建对象的Class类型

    * isSingleTon()设置被创建的对象是否为单例

    * ```java
      public class MyFactory implements FactoryBean<Dept> {
          
          /**
           * 通过此方法创建对象
           *
           * @return
           * @throws Exception
           */
          @Override
          public Dept getObject() throws Exception {
              return new Dept();
          }
          
          /**
           * 创建对象的类型
           *
           * @return
           */
          @Override
          public Class<?> getObjectType() {
              return Dept.class;
          }
          
          /**
           * 创建的对象是否为单例
           *
           * @return
           */
          @Override
          public boolean isSingleton() {
              return false;
          }
      }
      ```

      

* 在Spring配置文件中,装配工厂bean

  * ```xml
    <bean id="myFactory" class="com.atguigu.ssm.factory.MyFactory"/>
    ```

    

* FactoryBean返回类型较为特殊,不是自己本身的类型,而是getObject()返回的类型

  * ```java
    @Override
        public Dept getObject() throws Exception {
            return new Dept();
        }
    
    //****
     Dept myFactory = ioc.getBean("myFactory", Dept.class);
    ```

    



## Bean的作用域[重要]

### bean中的四个作用域

* [[request]] 请求域
  * 当前请求有效,离开当前请求失效
  * URl不变即为当前请求
* session 会话域
  * 浏览器不更换&不关闭即为当前会话
  * ...
* singleton 单例[默认值]
  * 当前项目中有且只有一个对象
  * 创建对象的时机
    * 创建容器对象时创建目标对象
* prototype 多例
  * 当前项目中有多个对象
  * 创建对象时机
    * 每次调用getBean()时,创建对象

### 设置bean作用域

* 在bean标签中添加scope属性即可

  * 单例

    * ```xml
       <!-- 测试单例-->
          <bean id="dept1" class="com.atguigu.ssm.pojo.Dept" scope="singleton">
              <property name="deptId" value="101"/>
              <property name="deptName" value="研发"/>
          </bean>
      ```

    

    * ```java
      public void test01(){
          
              ApplicationContext ioc = new ClassPathXmlApplicationContext("applicationContext_scope.xml");
          
              Dept dept1 = ioc.getBean("dept1", Dept.class);
              Dept dept11 = ioc.getBean("dept1", Dept.class);
          
              System.out.println(dept1 == dept11); //true
          }
      ```

  * 多例

    * ```xml
      <!-- 测试多例-->
          <bean id="dept1" class="com.atguigu.ssm.pojo.Dept" scope="prototype">
              <property name="deptId" value="101"/>
              <property name="deptName" value="研发"/>
          </bean>
      ```

    * ```java
      @Test
          public void test02(){
          
              ApplicationContext ioc = new ClassPathXmlApplicationContext("applicationContext_scope.xml");
          
              Dept dept1 = ioc.getBean("dept1", Dept.class);
              Dept dept11 = ioc.getBean("dept1", Dept.class);
          
              System.out.println(dept1 == dept11); //false
          }
      ```



## Spring中Bean的生命周期

### 默认情况

* 通过构造器或工厂创建bean的实例
* 为bean的属性设置值和对其他betan的引用
* 调用bean的初始化方法
* bean正常使用
* 当容器关闭时,调用bean的销毁方法
  * `ApplicationContext`没有close(),他的子接口有

### 添加后置处理器后

* 后置处理器
  * 后置处理器允许在调用初始化方法前后对bean进行额外的处理
  * 它会对IOC容器内所有bean实例逐一处理,其典型的应用时检查bean属性的正确性
* 实现后置处理器的方式
  * 实现接口`BeanPostProcessor`,并重写他的两个方法



## Spring中自动装配[ 重要 ]

* 根据bean标签的autowire属性指定的装配规则,不需要明确指定数值,Spring自动将匹配的值注入
* autowire只能指定非字面值

### 自动装配的规则

* bean标签中添加autowire属性
* 装配规则
  * byName 按照名称自动装配
    * 自动将IOC容器中bean的id与类中的属性名匹配
    * 如匹配成功 [ beanId名与类中属性名相等 ] 则自动装配成功
  * byType 按照类型自动装配
    * 自动将IOC容器中的class类中属性的类型进行匹配
      * 如class与属性的类型**唯一兼容,**则装配成功 [ 只能匹配一个相同的类 ]
      * 如不**唯一兼容**则装配失败

### 自动装配的总结

* 最终建议使用装配规则使用注解自动装配



## Spring中常用注解[ 非常重要 ]

> 约束 > [ 注解 ] > 配置 > 代码
>
> 使用注解可以简化配置文件

### 使用注解装配bean对象[ 4个 ]

1. @Component
   * 将普通组件装配到IOC容器中
2. @Repository
   * 将持久化层组件装配到IOC容器中
3. @Service
   * 将业务逻辑层组件装配到IOC容器中
4. @Controller
   * 将表示层/控制层组件装配到IOC容器中

#### 使用注解的步骤

* 导入jar包

* 在spring配置文件中,开启组件扫描

  * ```xml
     <!--开启组件扫描-->    
    <context:component-scan base-package="com.atguigu.annotation"/>
    ```

    

#### 总结

* Spring本身无法区分上述四个注解,他们的本质都是@Component,拆分为四个注解的原因是增加代码的可读性
* 类目首字母小写默认作为该bean的id,也可以使用value属性设置bean的id

### 使用注解装配bean对象中属性[ 3个 ]

* @Autowired
  * 使用它实现对象中属性的自动装配
  * 装配原理
    * 支持set() / 构造器 / [[第十六章 反射|反射]] 注入
  * 装配规则[ byName / byType ]
    * **先按照byType匹配,再按照byName匹配**
    * 先按照bean的class与类中的属性类型匹配
      * 匹配1个即装配成功
      * 匹配多个再按照byName匹配[ bean的id与类中属性名称匹配 ]
        * 匹配1个即装配成功
      * 匹配0个
  * @Autowire属性
    * required 设置被当前注解标识的属性是否必须自动装配
      * true[默认值]表示必须装配,如果未装配成功,则报错
      * false表示不必须装配,若未装配成功,默认值为null

