# 模块构建

1. service-item-client,创建需要对外暴露模块的远程调用接口
2. web-util,web 工程通用的工具模块
3. web,web 父工程
4. web-all,将页面数据添加至 resources 中

# 商品详情页显示

在 web-all 中使用远程调用,调用 service-item-client 的接口将数据提供给 Themeleaf(thymeleaf 直接使用 key-value 的方式取值)

# 商品详情页优化

详情页是用户高频访问的区域,所以必须尽量提高响应速度与并发处理
优化思路

1. 优化 MySQL 自身查询性能 [[day02_MySQL高级-索引|MySQL 索引]]
2. 使用 [[Redis]] 作为缓存来优化响应,尽量减少 MySQL IO 操作

# 分布式锁

讲解了[[分布式锁]]相关概念及实现
