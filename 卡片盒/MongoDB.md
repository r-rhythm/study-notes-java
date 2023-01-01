# 概述
- 和 [[Redis#Redis 概述]] 一样, MongoDB 也是 [[NoSQL]] 非关系型数据库
- 使用 C++ 编写, 它是一个==分布式==文件存储的数据库系统
## 特点
- 他的数据以 key-value 的格式存储, 类似 [[Json]] , 称为 BSON
- 以分布式的方式存储数据, 原理类似 [[ElasticSearch_核心概念#分片 Shards ※]]
- 支持丰富的查询表达式 (*单表*)
- 不支持 Join 多表连接查询
- MongoDB 支持索引

## MongoDB 概念
1. 数据库 
2. 集合 (表)
3. 文档 (记录)

| SQL 术语/概念 | MongoDB 术语/概念 | 解释/说明                           |
| ------------ | ---------------- | ----------------------------------- |
| database     | database         | 数据库                              |
| table        | collection       | 数据库表/集合                       |
| row          | document         | 数据记录行/文档                     |
| column       | field            | 数据字段/域                         |
| index        | index            | 索引                                |
| table joins  |                  | 表连接, MongoDB 不支持                |
| primary key  | primary key      | 主键, MongoDB 自动将_id 字段设置为主键 |   

## 数据类型
- Date: 日期 + 时间
- Object ID: 每次添加内容都会自动生成主键值, 主键名称固定为_id, 类型就是 Object ID
## MongoDB 与 MySQL 的不同
- ~~不需要提前创建数据库/集合, 它会根据添加的数据自动创建.**自动设置存入的数据类型**~~
- 它会根据添加的数据自动创建数据库&集合并自动设置字段的数据类型

# 为什么使用 MongoDB
1. 可以较为容易实现动态的表结构字段
2. 读写速度更快, 可承载的并发量更高
3. 在需求不明确 (表结构不确定) 的情况下, 使用 MongoDB 开发的维护成本最低

# MongoDB 的一些缺点
1. 很难胜任及其复杂的业务环境 (多表查询)
2. 占用空间大, 靠大量冗余数据来保证可用性
3. 单机 MongoDB 可靠性差, 且不支持事务, 只能靠集群的方式来保证可靠性

# 安装 MongoDB
基于 [[Docker]] 安装
1. 拉取镜像, 不建议使用最新的 `latest` 使用 `docker pull mongo:4.4.8`
# [[SpringData]] 整合 MongoDB
1. 引入 `spring-boot-starter-data-mongodb` 依赖
2. 在配置文件中添加 MongoDB `spring.data.mongodb.uri=mongodb://192.168.6.200:27017/test`
## MongoTemplate
- 实现 CURD 操作
- **聚合操作及其方便**
### 实现步骤
1. 添加一个对应的实体类,@Document ("集合名") 指定这个实体类与哪个集合建立映射,@Id 指定接收主键 id 的属性
2. 直接注入 MongoTemplate 实例即可
### 特性
- 使用实体类添加数据后会自动回填生成的主键值
- 实体类中没有赋值的属性不会添加到 MongoDB 中
查询
- 简单条件查询, 使用 Query 对象封装条件, 在 Query 中使用 Criteria 链式编程来封装条件
- 模糊查询, MongoTemplate 没有提供封装的模糊查询方式, 只能自己使用正则表达式 (regex) 来自己构建查询条件. 条件查询一般使用 MongoRepository
- 分页查询, `query.skip(0).limit(2)` 
- ==聚合查询==, aggregate (aggregation 封装聚合条件的对象, inputType MongoDB 表对应实体类 clazz, outputType 查询后的返回类型 clazz, 可用于 input 一样)
## MongoRepository
- 实现 CURD 操作
- 查询操作及其方便, 它支持通过 [[SpringData#SpringData 定义方法的规范]] 定义查询方法, 可用自动实现查询操作
实现步骤
1. 创建一个接口, 继承 MogoRepository\<entity, 主键类型>
2. 注入 Repository 实例即可
特性
- 在 Repository 中, 添加和修改都是调用 save () 方法实现, 它底层通过判断 id 来决定是修改还是添加
- FindById () 方法得到的不是结果对象,**需要在使用 get () 方法得到所需要的对象
查询
- 条件查询, Example 对象通过封装实体类对象指定查询条件, 所以需要创建一个实体类对象, 使用 Example. Of (entity) 来封装查询条件
- 模糊查询, 在封装条件的 Example 类中添加一个匹配器 ExampleMatcher 即可 `Example.of(entity,matcher)`
- 排序条件, `Sort.by(Sort.Direction.DESC,"排序字段")`
- 分页查询, `Pageable p = PageRequest.of(page,size,sort)` page 表示==当前页 (*0 为第一页*)==, size 表示每页记录数
最终优化, 使用 [[SpringData]] 操作 MongoDB, 只需要按照它的规范创建方法, SpringData 就会自动为我们实现查询的方法

## 注解
- @Transient 被该注解标注的，将不会被录入到数据库中。只作为普通的 javaBean 属性
# MongoDB 集群
## 主从模式
1. 写操作会在主机 (*Primary*) 操作, 在主机添加后再把数据复制到从机 (*Secondary*)
2. 可以通过从机和主机查询
## 仲裁模式
类似于 [[Redis#哨兵模式 Sentinel]]
1. 存在仲裁节点 (*Arbiter*), 它只负责仲裁, 不存储数据
2. 当主机挂掉后, 由仲裁节点