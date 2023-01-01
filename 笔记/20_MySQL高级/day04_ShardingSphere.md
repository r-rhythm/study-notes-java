# 读写分离
## 简介
用户的请求量随着业务的增长不断增大,采用读写分离架构将不同的请求分配给不同的服务器进行处理,尽可能的提升系统的可用性
## 基本实现
- 类似于 [[Redis#Redis 主从复制※]]
- 主机只负责写,从机只负责读
- **通过 SQL 的语义分析,决定走主机还是从机**
## 可能遇到的问题
- 数据的一致性问题
- [[Nacos#CAP 定理|CAP 定理]]是任何分布式的基本定理
- 使用数据的最终一致性[^1]来解决数据一致性问题

# 数据分片架构
## 垂直分片
- 专库专用,单一职责原则
- 在微服务架构下,不同的服务使用不同的数据库
- 垂直拆分之后，表中的数据量依然超过单节点所能承载的阈值，则需要水平分片来进一步处理
# ShardingSphere 概述
- 可以从代码层面(中间件)或应用层面来实现读写分离与数据分片架构
- 可以将任意数据库转换为分布式数据库,对原有的数据库进行增强
- [文档](https://shardingsphere.apache.org/legacy_zh.html)
## ShardingSphere-JDBC
### 概述
- 在应用层面实现读写分离/分库分表
- 轻量级的 Java 框架,增强版的 JDBC 驱动
- 支持任何基于 JDBC 的 ORM 框架
- 支持任意第三方数据库连接池
### SQL 执行流程
1. SQL 解析
2. SQL 优化
3. SQL 路由匹配
4. SQL 改写
## ShardingSphere-Proxy
- 数据库中间件,相同的还有 Alibaba 的 [[MyCAT]]
- 向应用程序完全透明,可以直接把他当作 MySQL 使用
# MySQl 主从同步
## 同步原理
- slave 会从 master 读取 binlog(binary log) 来进行数据同步(最终一致性)
- binlog 可以存储实际数据,又可以存储指令,它的思想类似于 [[Redis#Redis 持久化策略]] 中的 RDB 和 COF 混合
## 具体步骤
1. master 在执行更改数据的时候会将执行语句记录到 binlog 中
2. 当 slave 节点执行 start slave 命令指定它的主机后,slave 节点会创建一个 I/O 线程来连接 master,并请求 master 中的 binlog
3. slave 连接上 master 之后,master 会创建一个 log dump 线程,它会读取 binlog 并发送给 slave,在读取的过程中它会对 binlog 加锁
4. I/O 线程接收到到 log dump 线程发过来的更新之后,会将数据保存到 relay log 中继日志中
5. slave 的 SQL 线程读取中继日期,解析执行相同的操作,以达到主从操作一致,保证最终数据一致

#  MySQL 一主多从搭建
## binlog 格式说明
- binlog_format=STATEMENT,日志记录写指令,性能高但是使用系统类的函数可能导致主从数据不同步问题,例如 now() 函数
- binlog_format=ROW(默认),日志记录写后的数据,性能较差,但不存在数据不同步问题
- binlog_format=MIXED,以上两种格式混合使用,系统自动选择

# ShardingSphere-JDBC
## 读写分离

## 垂直分片
### 概念
- 逻辑表

## 水平分片
- 有多个库和多个表的情况下,就不能使用表的主键自增,否则主键的唯一性就无法保证了

逻辑表与实际表建立映射

# ShardingSphere-Proxy


---
[^1]: 系统中的所有数据副本经过一定时间后，最终能够达到一致的状态。