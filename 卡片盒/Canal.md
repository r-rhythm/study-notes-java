# 介绍
阿里巴巴旗下的 MySQL 数据库 binlog 增量订阅 & 消费组件, 主要用于解决 MySQL 与 Redis 间数据同步的问题
[alibaba/canal: 阿里巴巴 MySQL binlog 增量订阅&消费组件 (github.com)](https://github.com/alibaba/canal)

# 使用
[参阅: 超详细canal入门，看这篇就够了 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/177001630)

# 实现原理
Canal 将自己伪装为一个 slave 节点, 利用 MySQL 的主从同步原理监听主机发送的 binlog 文件, 然后解析这个 binlog 文件, 再将修改的消息发送到存储目的地的
