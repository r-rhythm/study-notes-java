# [[卡片盒/Spring]]



## HelloSpring

> Spring是一个IOC(DI)和AOP容器的开源框架

* 导入jar包

  * ```xml
    <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>5.3.1</version>
            </dependency>
    ```

    

* 编写spring.xml配置文件

  * ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    ```

    

* 使用核心类库[ApplicationContex]获取IOC容器中对象

  * ```xml
    <!--    使用Spring的IOC容器帮我们创建对象并管理对象-->
        <bean id="empLiu" class="com.atguigu.pojo.Employee">
            <property name="lastName" value="liuwei"/>
            <property name="email" value="liuw-hk@outlook.com"/>
            <property name="salary" value="12345.6"/>
        </bean>
    ```

* 测试HelloWorld

  * ```Java
    //        创建IOC容器对象
            ApplicationContext ioc = new ClassPathXmlApplicationContext("applicationContext.xml");
    
    Employee empLiu = ioc.getBean("empLiu", Employee.class);
            System.out.println("empLiu = " + empLiu);
    ```

    

## Spring特性

* 非侵入式
  * 基于Spring开发的应用中的对象可以不依赖于Spring的API
* 容器
  * Spring容器可以管理所有对象
* 组件化
  * Spring实现了用简单的组件配置组合成一个复杂的应用
* 一站式
  * Spring可以整合一系列优秀的开源框架和第三方类库
* IOC--Inversion of Control
  * 控制反转
  * 将对象的控制权从程序员手里反转给Spring
* DI--Dependency Injection
  * 依赖注入
  * Spring可以管理对象与对象之间的依赖关系
* AOP--Aspect Oriented Programming
  * 面向切面编程
  * 在不修改源代码的基础上进行功能的扩展

## Spring中获取getBean的三种方式

### getBean("id")

* 通过bean的id从容器中获取对象
  * 获取对象的类型是Object,需要强转

### getBean(Class clazz)

* 通过Class方式获取对象
  * 当容器中存在多个相同Class时,会报出异常`NoUniqueBeanDefinitionException`

### getBean("id",Class clazz)

* 通过id与Class定位获取对象

  * ```java
    Employee empLiu = ioc.getBean("empLiu", Employee.class);
            System.out.println("empLiu = " + empLiu);
    ```



## Spring底层IOC实现

### BeanFactory

* Spring底层就是对象工厂[BeanFactory]
* 结构图
  * BeanFactory,底层IOC实现
    * ApplicationContext
      * ConfigurableApplicationContext,包含了refresh()和close()的扩展方法
        * ClassPathXmlApplicationContext,基于类路径加载xml
        * FileSystemXmlApplicationContext,基于系统文件加载xml

### 源码解析BeanFactory

* Spring默认在创建IOC容器对象之后帮我们创建装配的对象

## 依赖注入的三种方式

### Spring中的setXX()注入

* `<property>`

* 底层是使用setXX()为属性赋值

  * ```xml
    <bean id="empLiu" class="com.atguigu.pojo.Employee">
            <property name="lastName" value="liuwei"/>
            <property name="email" value="liuw-hk@outlook.com"/>
            <property name="salary" value="12345.6"/>
        </bean>
    ```

    

### Spring中的构造注入

* `<constructor-arg>`

* 通过有参构造器为属性赋值

  * ```xml
    <!--    通过构造器注入对象属性-->
        <bean id="jiaJ" class="com.atguigu.pojo.Employee">
            <constructor-arg name="id" value="4"/>
            <constructor-arg name="lastName" value="jiaJ"/>
            <constructor-arg name="email" value="jj@123.com"/>
            <constructor-arg name="salary" value="150000"/>
        </bean>
    ```

    

### Spring中的 [ p 名称空间 ] 注入

* ```xml
        xmlns:p="http://www.springframework.org/schema/p"
  
  <bean
        p:
        ></bean>
  ```

  

* 它通过setXX()为属性赋值

  * ```xml
    <bean id="zeWen_Liu" class="com.atguigu.pojo.Employee"
         p:id="5" p:lastName="刘泽文" p:email="zewen@123.com" p:salary="12345.7"
         ></bean>
    ```

    

## Spring依赖注入数值问题

### 字面量数值

* 基本数据类型及其包装类,String
* 语法区分,使用value属性或子标签注入的数值为字面量数值

### XML中的CDATA区

* `<![CDATA[ ]]`

* 可以在XML中书写特殊字符,如`<>&`等

* ```xml
  <bean id="lanJ" class="com.atguigu.pojo.Employee">
          <property name="lastName">
              <value><![CDATA[<--LanJ&-->]]></value>
          </property>
          <property name="email" value="lj@123.com"/>
          <property name="salary" value="12345.678"/>
      </bean>
  ```

  

### 引入外部已声明的bean

* `<bean ref="beanId">`

* 将外面已经申明的bean注入到属性中

  * ```xml
     <!--外部声明的bean-->
    <bean id="deptInfo" class="com.atguigu.pojo.Dept"
            p:deptId="1" p:deptName="财务部门" p:deptLocation="1-45"
        ></bean>
    
        <bean id="jiaJ" class="com.atguigu.pojo.Employee" >
            <constructor-arg name="id" value="4"/>
            <constructor-arg name="lastName" value="jiaJ"/>
            <constructor-arg name="email" value="jj@123.com"/>
            <constructor-arg name="salary" value="150000"/>
            <!--引入外部声明的bean-->
            <constructor-arg name="dept" ref="deptInfo"/>
        </bean>
    ```

    

* 外部已声明bean的级联属性赋值问题

  * 级联更改引入的bean的属性值会直接影响到外部bean的属性值,因为他们是指向同一个引用对象[String赋值问题]