# 管理员系统-预约统计
- 前端使用 ECharts 实现可视化数据图表展示
- 后端编写xml语句,查询 reserveDate 的数据与 count 每日总条数
*补充了[[MyBatis-Plus#绑定异常]]的解决方法*
- 编写接口,返回 ECharts 需要的 reserveDate 和 count 数据

# 代码提交git
[[Git]]

# 下单并发处理

*处理并发问题就是解决超卖问题*
关联[[Redis#Redis 解决超卖问题]]

解决方案
- 只有热门医院号才有紧张状态
- 只有放号的一刻才有高并发问题
- 使用 [[Sentinel]] 根据热门医院(热点值)进行流量控制
	- 根据医院的编号,进行流量控制

使用 Sentinel 解决超卖问题
1. 在 pom 中引入 sentinel 依赖
2. 在服务的配置文件中添加 sentinel 的地址信息
3. 在 sentinel 控制台点击热点规则 -> 新增热点规则或使用 Java 代码实现
4. 使用 JEMTER 做并发测试

# 项目部署
1. 将本地代码进行打包
	1. 基于 maven 实现,部署前先检查目录结构,启动的服务模块不能与其他非启动模块混在一起
	2. 在引入其他模块时,不能直接引入父工程,必须引入具体的模块
	3. 在微服务模块的 pom 文件中添加 build 配置,**注意不能放到父工程里**
	4. 将服务的 nacos 地址设置一下
2. 将 jar 包制作为 [[Docker]] 镜像
	1. 创建 Dockerfile 文件,准备 Dockerfile 和 jar 包
	2. 上传两个文件到 linux 系统中
	3. 使用 docker build 命令制作镜像
3. 把镜像上传至镜像服务器
	1. 使用阿里云的*容器镜像服务 ACR*
	2. 创建名称空间 & 镜像仓库
	3. 按照阿里云的提示将镜像推送至远程库
4. 从镜像服务器下载镜像,通过容器启动

# 项目总结
核心功能
1. 医院接口调用平台数据接口
2. 用户登录流程
3. 预约挂号流程

后端技术
- springboot
- springcloud
- redis
- mybatis-plus
- mongoDB
- rabbitMQ
- docker
- 微信相关
	- 微信登录
	- 微信支付
	- 微信退款
- 阿里云
	- 短信服务
	- OSS 存储服务
	- ACR 镜像容器
- easyExcel
- springSchedule + cron
- JWT

项目中遇到的问题
- 跨域问题
- 远程调用超时
- maven 文件的加载约定
- 单点登录的实现(token)
- cookie 跨域传递
- 浏览器预请求(*OPTIONS*)问题

其他补充
- redis 设置微信支付二维码的有效时间(*key : orderId,value : resultMap *)
- 通过用户查询未支付的订单,若超过一定的次数可以对账户做一定限制

前端技术
- Vue + Element_ui
- npm
- node.js
- nuxt