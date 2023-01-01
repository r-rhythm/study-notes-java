# [[Redis#Redis 概述]]

# [[Redis#安装Redis]]

# [[Redis#Redis Key]]

# [[Redis#Redis 中的五种基本类型]]

# [[Redis#Redis 配置文件介绍]]

# [[Redis#在 IDEA 中集成 Redis]]


# [[Redis#Redis 事务 Transaction]]

# [[Redis#Redis 中事务锁]]


# 并发模拟

* `yum -y install httpd-tools` 安装Apache ab模拟测试
* 新建一个模拟表单提交参数以&符号结尾;存放当前目录。`vim postfile  ` 内容`prodid=0101&`
* `ab -n 2000 -c 200 -k -p ~/postfile -T application/x-www-form-urlencoded http://主机Ip地址:8080/表单提交到的项目地址`

# [[Redis#Redis 持久化策略]]


# [[Redis#Redis 主从复制※]]


### 哨兵模式

> 能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库

1. 在自定义的/root/myredis目录下新建sentinel.conf文件
2. 配置哨兵,填写内容
   1. `sentinel monitor mymaster 127.0.0.1 6379 1`
   2. 其中mymaster为监控对象起的服务器名称， 1 为至少有多少个哨兵同意迁移的数量。 
3. 启动哨兵
   1. `redis-sentinel /root/myredis/sentinel.conf`



# [[Redis#Redis集群搭建※]]