## JdbcTemplate

### 简介

* JdbcTemplate是Spring提供的轻量级持久化层框架
  * 它类似于Mybatis

### 使用JdbcTemplate

* 导入jar包

  * ```xml
    <!--spring-orm-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-orm</artifactId>
                <version>5.3.1</version>
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
    ```

    

* 编写配置文件

  * ```xml
    <context:property-placeholder location="classpath:druid.properties"/>
    
        <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
            <property name="driverClassName" value="${db.driverClassName}"/>
            <property name="url" value="${db.url}"/>
            <property name="username" value="${db.username}"/>
            <property name="password" value="${db.password}"/>
        </bean>
    
        <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
            <property name="dataSource" ref="dataSource"/>
        </bean>
    ```

    

* 使用核心类库

  * JdbcTemplate

    * 

    * ```java
      @Test
          public void test01(){
          
              ApplicationContext ioc = new ClassPathXmlApplicationContext("spring_Template.xml");
          
              JdbcTemplate jdbcTemplate = ioc.getBean("jdbcTemplate", JdbcTemplate.class);
          
      //        System.out.println("jdbcTemplate = " + jdbcTemplate);
          
              String sql = "insert into tbl_dept(dept_name,dept_location) values (?,?)";
              
              jdbcTemplate.update(sql,"人事","2-123");
              
          }
      ```

### 常用的API

* jdbcTemplate.update(String sql,Object...)通用的增删改方法

* batchUpdate(String sql,List<Object[]>)批处理增删改方法

* queryForObject(String sql,Class clazz,Object...)查询单个值通用方法

* queryForObject(String sql,rowMapper<T> rm,Object...)查询单个对象

* query(String sql,RowMapper<T>,Object...)查询多个对象通用方法

  * ```java
    @Repository
    public class EmployeeDaoImpl implements EmployeeDao {
        
        @Autowired
        private JdbcTemplate jdbcTemplate;
        
        @Override
        public List<Employee> selectAllEmployee() {
            
            String sql = "select id,last_name,email,salary from tbl_employee";
        
            //rowMapper指的是行的映射,将结果集的字段直接映射到行中.它相当于mybatis中的映射文件
            RowMapper<Employee> rowMapper = new BeanPropertyRowMapper<>(Employee.class);
        
            return jdbcTemplate.query(sql, rowMapper);
        }
    }
    ```

    

* JdbcTemplate是自动提交事务的

## Spring声明式事务管理

### 回顾事务

* 事务四大特征
  * 原子性
  * 一致性
  * 隔离性
  * 持久性
* 事务三种行为
  * 开启事务
    * mybatis -> 默认开启事务
  * 提交事务
    * mybatis -> sqlSession.commit()
  * 回滚事务
    * mybatis -> sqlSession.rollback()

### Spring中支持的两种事务管理

* 编程式事务管理(传统风格)
  * 获取数据库连接Connection对象
  * 取消事务的自动提交
  * 执行操作[ **核心业务** ]
  * 正常完成操作时手动提交事务
  * 执行失败时回滚事务
  * 关闭相关资源
    * 事务管理代码与核心业务代码相耦合,导致事务管理代码比较分散,混乱
    * 解决方案,先将事务管理代码[ **非核心业务** ]横向提取,再动态织入
* 声明式事务管理
  * **使用AOP思想去管理事务**
  * 先将事务管理代码[ **非核心业务** ]横向提取,再动态织入



### 声明式事务管理使用步骤

* 导入AspectJ坐标支持(使用注解可省略)

* 编写配置文件

  * 装配事务管理器

    * ```xml
      <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
              <property name="dataSource" ref="dataSource"/>
          </bean>
      ```

      

  * 开启事务注解支持

    * ```xml
      <tx:annotation-driven></tx:annotation-driven>
      
      <!--http://www.springframework.org/schema/tx
             http://www.springframework.org/schema/tx/spring-tx.xsd" -->
      ```

      

* 在需要添加事务的方法上加入@Transactional注解



### Spring中声明式事务管理的属性

* @Transaction的书写位置

  * 类上面 表示当前类中所有方法都使用事务管理
  * 方法上面 表示当前方法使用事务管理

* @Transaction属性

  * Propagation 事务传播行为
  * Isolation 事务隔离级别
  * Timeout 事务超时,自动回滚时间
  * Readonly 事务只读
  * rollbackFor & noRollbackFor 

* **Propagation [[事务 Transaction#事务的传播行为]]**

  * > 如method1()执行后调用了method2(),此时必须设置method2()这个事务的传播行为

  * **REQUIRED** 如果之前有事务,就在之前的事务内运行,如果没有则开启新的事务并在自己的事务内运行

  * **REQUIRES_NEW** 当前的方法必须启动新事物,并在自己的事务内运行,如果有事务正在运行,应该将他挂起

  * 实例 结账时，账号余额支持买多少本书，就允许用户购买,直到账户余额不够支付下一本

    * ```java
      @Transactional
          public void checkOut(String username, List<String> isbns) {
      
              for (String isbn : isbns) {
                  bookShopService.purchase(username,isbn);
              }
          }
      ```

      

    * ```java
      //使用REQUIRED则按照执行此方法前的事务运行,如果此方法在同个事务中多次调用一次出错会全部回滚
      //@Transactional(propagation = Propagation.REQUIRED)
      @Transactional(propagation =  Propagation.REQUIRES_NEW)
      public void purchase(String username, String isbn) {
          //查询book价格
          Integer price = bookShopDao.findBookPriceByIsbn(isbn);
          //修改库存
          bookShopDao.updateBookStock(isbn);
          //修改余额
          bookShopDao.updateUserAccount(username, price);
      }
      ```

* Isolation 设置事务隔离级别

  * 隔离级别
    * read uncommitted 读未提交[1]
    * **read committed** 读已提交[2]
    * **repeatable read** 可重复读[4]
    * serializable 串行化[8]

* Timeout 事务超时

  * 超时未提交事务,默认回滚

* ReadOnly 事务只读

  * 只有查询时使用事务只读,可以提高一点性能

* rollbackFor | noRollbackFor

  * rollbackFor 设置回滚的异常 **注意:SQL异常或IO异常等默认都不会回滚,想要回滚需要设置 rollbackFor = Exception.class 表示所有异常都回滚**
  * noRollbackFor 设置不回滚异常

### 基于XML配置Spring声明式事务管理

## Spring5新特性