> [!info] Seata
> - 阿里开源的一款微服务架构下的分布式事务解决方案,高性能,高可用
> - 支持 AT / TCC / SAGA / XA 多种模式

Seata 的设计思路就是将一个分布式事务视为一个全局事务,下面挂了若干个分支事务

## AT 模式
- Transaction Corrdinator(TC) 事务协调器,负责维护全局事务的提交或回滚
- Transaction Manager(TM) 事务控制权,服务开启一个全局事务,并最终发起提交/回滚的决议
- Resource Manager(RM) 资源管理器,负责本地事务的注册/提交/回滚,提交本地事务的状态
- XID 全局事务的唯一ID

![[分布式事务-Seata.png]]
*TM是一个分布式事务的发起者和终结者，TC负责维护分布式事务的运行状态，而RM则负责本地事务的运行。*

执行流程
1. TM 向 TC 申请开启一个全局事务,全局事务创建超过并返回一个全局唯一的 XID
2. XID 在调用链的上下文中传播
3. RM 向 TC 注册分支事务,

## Seata 整合
1. 启动[[Nacos]]
2. 在 nacos 的配置中心添加 seataServer.properties,内容从课件中复制
3. 创建一个数据库 seata142  添加三张表 mysql_global_table
4. 新建 dataId 为 common.yml 的配置文件,在 api 模块的 bootstrap.yml 中中引入这个配置
5. 启动 seataServer 服务,修改它的注册中心和配置中心为 Nacos,配置中心的 dataID 要和上面添加的名称一致
6. 在程序入口添加服务添加注解@GlobalTransactional 开启全局事务即可

## Seata 原理
- 在执行 SQL 前存储一份快照(before image),执行 SQL 后也会存储一份快照,如果要回滚直接通过逆向 SQL 回到之前的状态

---
