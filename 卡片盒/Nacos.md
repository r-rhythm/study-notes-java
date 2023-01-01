# 简介
> [!info]
> * 由 Alibaba 提供的注册中心&配置中心 cloud 组件
> * Nacos = [[Eureka]] + Config + Bus
> * 替代[[Eureka]]做服务注册中心
> * 替代 Config 做配置中心

# 安装
1. 从官网下载压缩包
2. 解压后通过 startup.cmd 运行
3. 注意,如果启动失败则可能是以集权模式启动的,如果需要使用单机模式启动,则通过命令行 -m standalone 启动
4. 启动成功后访问 http://localhost:8848/nacos 查看

# 作为注册中心使用
创建服务在 Nacos 中进行注册
1. 父工程引入 alibaba-cloud 依赖,子模块引入 alibaba-nacos
2. 只要引入了 nacos 依赖,服务启动时就会自动在 nacos 中进行注册
3. 添加 nacos 服务地址到配置文件中
4. 在启动类上使用@EnableDiscoveryClient 注解 ,在 Nacos 中进行注册即可,无需为服务中心单独建立服务

# CAP 定理
> [!info] CAP 定理
> 指在一个系统中,三者不可兼得
>
>* Consistency(一致性)
>     * 分布式系统中同一个  ==事务==  前后都要保持一致,失败或成功都要保持一致
>     * 假如用户的请求在服务链中某个服务执行失败,则整个事务都失败
>
>* Availability(可用性)
>    * 在集群中即使一部分节点 down 机,仍然可以提供服务
>
>* Parition tolerance(分区容错性)
>    * 各个服务中,事件相互独立,它允许各个服务对同一事件做不同处理,与 Consistency(一致性)互斥
>    * 假如用户购买商品,支付服务运转正常,而库存管理模块调用失败,也允许此次事件成功
>    * 即使某以服务网络出现了分区(无法与其他服务联通),其他服务仍然可以正常运转"履行职责"

# 做为配置中心使用
## 简介
在微服务中,根据运行环境的不同,会使用不同的配置作为程序的配置文件,例如数据库地址/redis 地址等,这些文件如果存储在服务模块中,会导致服务间的配置文件杂乱无章,无法统一管理,**将配置文件统一迁移至 nacos 中进行统一管理,使其更方便管理与操作.**

## nacos 管理配置文件
1. 重新安装 nacos,将它的数据原改为 MySQL,将未来添加的配置内容持久化到数据库中,避免丢失
	1. 在数据库中添加一个 nacos 库
	2. 使用 SQL 脚本在这个库中创建 nacos 持久化配置所需要的表
3. 在模块中引入 nacos config 依赖
4. 在模块中创建 [[SpringBoot#加载顺序|bootstrap.yaml]] 引导文件,在 bootstrap 文件中配置注册中心的地址
5. 将重复的配置,如 [[Redis]],[[RabbitMQ]] 等提取为通用配置等等操作,使配置更易于管理

Nacos 提供了一系列的配置存储机制,可以直接在配置中心添加和管理配置,
在实际开发中某些数据并不适合存入数据库中,以便获取
3. 创建 application.yaml 文件
4. Data id 的格式固定为 `${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}`
5. 通过 SpringCould 原生注解@RefreshScope 实现配置自动更新

## 配置中心分类
- 每个 namespace 包含多个 group
- 每个 group 包含多个 data id

# 搭建 Nacos 集群
1. 上传 nacos 并解压
2. 重命名 nacos 后复制三个
3. 修改他们的配置文件内的端口号
4. 使用 mysql 数据库替代 Nacos 自带的 derby-data 数据库
```properties
#*************** Config Module Related Configurations ***************#  
### If use MySQL as datasource:
spring.datasource.platform=mysql  
  
### Count of DB:  
db.num=1  
  
### Connect URL of DB:  
db.url.0=jdbc:mysql://192.168.1.112/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC  
db.user.0=root  
db.password.0=root

```
5. 修改/conf/cluster 文件,添加集群的 ip 及端口号
6. 修改 nacos/bin/startup.sh 中的内存大小,512m 足够
7. 启动 Nacos

配置[[Nginx]]负载均衡
