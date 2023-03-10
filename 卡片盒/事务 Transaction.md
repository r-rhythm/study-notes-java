# 事务
> [!info] 什么是事务
> 事务就是一件事的完整业务逻辑,如: a 为 b 转账,a 账号减少金额与 b 账号添加金额是一个完整的业务逻辑,缺一不可.
> 事务时数据库操作的基本单元,指的是逻辑上一组操作,要么全部成功,要么全部失败

# 事务的特性
## 事务四大特性 ACID
- 原子性(*Atomicity*): 一个事务中的所有操作,要么全部完成,要么全部失败,不会结束在中间的某一个环节.事务在执行过程中发生错误会被回滚到事务开始前的状态,就像这个事务从来没有执行过一样
- 一致性(*Consisistency*): 在一个或多个事务执行后,数据从一个正确的状态更新到另一个正确的状态,不会出现矛盾 
- 隔离性(*Isolation*): 事务的隔离性是指多个用户并发访问数据库时，一个用户的事务不能被其它用户的事务所干扰，多个并发事务之间数据要相互隔离
- 持久性(*Durability*): 持久性是指一个事务一旦被提交，它对数据库中数据的修改就是永久性的，即使系统故障也不会丢失。

## 并发访问问题
如果不考虑隔离性，事务存在 3 中并发访问问题。

1. 脏读：一个事务读到了另一个事务未提交的数据.**不允许发生**
2. 不可重复读：一个事务读到了另一个事务已经提交(update)的数据。引发另一个事务，在事务中的多次查询结果不一致。
3. 虚读 /幻读：是指在一个事务内,两次读取之间有另一个事务进行了写入操作，使得第二次读取看到了第一次读取未看到的数据，这就是所谓的“幻觉”

# 事务的隔离级别
- 数据库规范规定了 4 种隔离级别，分别用于描述两个事务并发的所有情况。

1. **read uncommitted**[1] 读未提交，一个事务读到另一个事务没有提交的数据。
   a)存在：3 个问题（脏读、不可重复读、虚读）。
   b)解决：0 个问题

2. **read committed[2]** 读已提交，一个事务读到另一个事务已经提交的数据。(*Oracle 默认的级别*)
   a)存在：2 个问题（不可重复读、虚读）。
   b)解决：1 个问题（脏读）

3. **repeatable read[4]**:可重复读，在一个事务中读到的数据始终保持一致，无论另一个事务是否提交。(*MySQL 默认的隔离级别*)

   a)存在：1 个问题（虚读）。

   b)解决：2 个问题（脏读、不可重复读）

   4.**serializable 串行化[8]**，同时只能执行一个事务，相当于事务中的单线程。(*他可以解决所有读的问题,但是效率很低*)

    a)存在：0 个问题。

    b)解决：3 个问题（脏读、不可重复读、虚读）

- 安全和性能对比
  - 安全性：`serializable > repeatable read > read committed > read uncommitted`
  - 性能 ： `serializable < repeatable read < read committed < read uncommitted`
- 常见数据库的默认隔离级别：
  - MySql：`repeatable read`
  - Oracle：`read committed`


# 本地事务
基于单个服务,操作单一数据库资源访问的事务称为本地事务