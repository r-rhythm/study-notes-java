# 事务回顾
事务四大特征:
  1. 原子性
  2. 一致性
  3. 隔离性
  4. 持久性

事务三种行为:
1. 开启事务: mybatis -> 默认开启事务
2. 提交事务: mybatis -> `sqlSession.commit()`
3. 回滚事务: mybatis -> `sqlSession.rollback() `


# Spring 声明式事务管理

## 开启步骤
1. 编写配置文件
2. 装配事务管理器

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">        
	<property name="dataSource" ref="dataSource"/>
</bean>
```

3. 开启事务注解支持

```xml
  <tx:annotation-driven></tx:annotation-driven>
  
  <!--http://www.springframework.org/schema/tx
 http://www.springframework.org/schema/tx/spring-tx.xsd" -->
```

4. 在需要添加事务的方法上加入@Transactional 注解

## Spring 中声明式事务管理的属性

@Transaction 的书写位置:
  * 类上面表示当前类中所有方法都使用事务管理
  * 方法上面表示当前方法使用事务管理

@Transaction 注解属性:
  * Propagation 事务传播行为
  * Isolation [[事务 Transaction#事务的隔离级别]]
  * Timeout 事务超时, 自动回滚时间
  * Readonly 事务只读
  * rollbackFor & noRollbackFor **注意: SQL 异常或 IO 异常等默认都不会回滚, 想要回滚需要设置 rollbackFor = Exception. Class 表示所有异常都回滚**

# 事务传播行为 Propagation
在 Spring 的声明式事务管理中, 如 `method1()` 执行后调用了 `method2()`, 此时必须设置 `method2()` 这个事务的传播行为
1. PROPAGATION_REQUIRED (*默认*) 若当前没有事务, 就新建一个事务, 如果已经在一个事务中, 加入到这个事务
2. PROPAGATION_REQUIRED_NEW 将当前事务挂起, 创建新的事务执行~~新建一个事务, 如果当前已经存在事务, 将这个事务挂起使用新的事务~~
3. PROPAGATION_SUPPORTS 支持当前事务, 如果当前没有事务就以非事务的方式执行
