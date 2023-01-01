# 概述
基于 AMQP 协议, 使用 erlang 语言开发, 稳定性更好并发能力更强
支持跨平台多种语言使用
[[消息队列 Message Queue#常见的MQ产品]]
# RabbitMQ 的基本组成
- Broker[^Broker] 消息队列服务器实体
- Vhost 虚拟主机, 一个 broker 中可以有多个虚拟主机, 用作不同用户的权限分离
- Channel 消息通道
- Exchange[^Exchange] 信息交换机, 它指定了消息按照什么规则, 路由到哪个队列
- Queue[^Queue] 消息队列载体, 每个消息都会分发到一个或多个队列
- Binding 将队列与交换机进行绑定
- RoutingKey 路由关键字, 交换机通过路由 key 指定的规则进行消息转发
- Produce 生产者, 生产信息. 生产者不关注队列, 只关注交换机和路由 key 
- Consumer 消费者, 消费消息. 消费者不关注交换机, 只关注队列

# RabbitMQ 的五种工作模式
> [!note] 简单模式
> P (produce 生产者) -> 队列 -> C (consumer 消费者)
>  * 一个生产者一个消费者
>  * 生产者发送消息到队列, 消费者从队列获取信息
>  * 使用底层默认的交换机 Defult Exchange

>[!note] 工作模式 Work queues
一个生产者发送消息到消息队列, 多个消费者从消费队列中获取
> * 生产者发送消息到队列, 多个消费者从队列中获取信息
> * 使用底层默认的交换机 Defult Exchange
> * 多个消费者是竞争关系

> [!note] **发布/订阅模式**(Publish/Subscribe)
> 一个生产者 -> 一个交换机 -> 多个队列 -> 多个消费者
> * 生产者发送消息到交换机, 交换机进行消息转发, 将消息分发给不同的队列
> * 多个消费者在不同的队列获取消息
> * 使用广播型交换机, fanout

> [!note] **路由模式**(Routing)
> 通过路由 key 进行匹配, 在生成到队列前进行动态的匹配, 决定发送到哪个队列中
> * 交换机和队列通过路由 key 进行绑定
> * 生产者发送消息到交换机, 交换机通过路由 key 进行匹配转发消息到路由 key 所匹配的队列中
> * 使用 direct 定向类型的交换机, 或又称为==定向投递==

> [!note] **通配符模式**(Topic)
> 与路由模式相同, 但是通过使用通配符的方式以匹配到更多的内容
>
> 通配符:
> * \* 只匹配一个词, 例如 info.\* 可以匹配 info. Insert/info. Xxx
> * \# 匹配多个词如 info.# 可以匹配 info. Abc. Insert ...

# 交换机类型
- Fanout[^fanout]: 广播, 将信息分发给所有绑定到此交换机的队列
- Direct: 定向, 把信息分发给符合指定的路由 key 的队列
- Topic: 通配符, 交换机通过通配符匹配到指定的队列, 并将消息转发给它
*消费端底层就是一个监听器, 实时监控消息队列内是否有消息, 如果没有就一直等待*
*交换机不负责存储数据, 只负责转发消息*



[^Broker]: 消息服务器，作为 server (服务器) 提供消息核心服务
[^Exchange]: 消息交换机, 它指定消息按什么规则, 路由到哪个队列。
[^Queue]: 队列 PTP 模式下，特定生产者向特定 queue 发送消息，消费者订阅特定的 queue 完成指定消息的接收
[^fanout]: 扇出, 表示一个模块调用多个模块. 和它相对的还有扇入*fanin*


# 消息的可靠性投递 (消息不丢失)

## 本地消息表
使用本地消息表实现[[消息队列 Message Queue#消息百分百成功投递]]

## Confirm 生产者发送至交换机消息确认
生产者会在接收到确认收到消息的 ack 消息前一直发送, 指定确认发送的消息被接收
1. 消息持久化, 开启磁盘持久化配置, 在消息持久化后向生产者发送 ack 确认信息, 即使在服务宕机重启后, 也会重新向生产者发送 ack 确认信息
2. 手动消费确认, 有时信息被正确的投递到消费方, 但是消费方处理业务失败, 便会产生不一致问题. 要解决这个问题, 就协议引入消费方确认, 即只有成功消费后才返回 ack 消息, 否则告知 mq nack

## Return 交换机发送至队列失败退回 (事务表)
交换机发送消息给队列, 如果转发的过程中失败了, 交换机返回投递失败信息 returnCallback
*代码示例参见下方的消息发送失败的重试机制*


## 消费者端保证成功接收

* 消费端使用 ack (*Acknowledge*) 属性确定是否真正接收到消息
	* 自动确认 `acknowledge=none`
	* 手动确认 `acknowledge=manual`
大部分接收失败都是由于网络闪断导致的

## 消息发送失败的重试机制
1. 实现 `RabbitTemplate.ConfirmCallback` 接口中的 confirm 方法, 判断 ack 是否为 true
2. 如果它为 false 则代表消息没有达到交换机
3. 自定义一个类, 继承 `CorrelationData` 类, 它是一个 mq 消息所使用到的交换机路由 key 等详细信息的封装实体
4. 自定一个消息发送重试方法 retrySendMessage, 将 mq 的 correlationData 对象传入, 并定义重新发送的业务逻辑
5.  调用 `retrySendMessage(correlationData)` 方法来重新尝试发送消息

代码示例:
```java
package com.atguigu.gmall.common.config;  
  
import com.alibaba.fastjson.JSON;  
import com.atguigu.gmall.model.GmallCorrelationData;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.amqp.core.Message;  
import org.springframework.amqp.rabbit.connection.CorrelationData;  
import org.springframework.amqp.rabbit.core.RabbitTemplate;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.data.redis.core.RedisTemplate;  
import org.springframework.stereotype.Component;  
  
import java.util.concurrent.TimeUnit;  
  
/**  
 * 保证消息的可靠性传递  
 *  
 * @author: liu-wēi  
 * @date: 26 Dec 2022,19:35  
 */
@Component  
@Slf4j  
public class MqProducerAckConfig implements RabbitTemplate.ConfirmCallback, RabbitTemplate.ReturnCallback {  
      
    @Autowired  
    private RedisTemplate redisTemplate;  
      
    @Autowired  
    private RabbitTemplate rabbitTemplate;  
      
    /**  
     * Confirmation callback.     * 生产者发送消息到交换机确认  
     *  
     * @param correlationData correlation data for the callback.  
     * @param ack             true for ack, false for nack  
     * @param cause           An optional cause, for nack, when available, otherwise null.  
     */    
    @Override  
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {  
        if (ack) {  
            log.info("信息发送成功,数据: " + JSON.toJSONString(correlationData));  
        } else {  
            log.error("消息发送失败,数据: " + JSON.toJSONString(correlationData));  
              
            // 重新尝试发送消息  
            this.retrySendMessage(correlationData, 3);  
        }  
          
    }  
      
    /**  
     * 重新发送消息的方法  
     *  
     * @param correlationData 将生产者与消息关联起来的基类  
     */  
    private void retrySendMessage(CorrelationData correlationData, int retryTimes) {  
        // 将关联消息类转换为GmallCorrelationData  
        GmallCorrelationData gmallCorrelationData = (GmallCorrelationData) correlationData;  
          
        // 获取重试次数  
        int retryCount = gmallCorrelationData.getRetryCount();  
          
        // 判断重试次数  
        if (retryCount == retryTimes) {  
            // 重试次数达到上限  
            log.error("重试次数已经达到上限,发送消息失败: " + JSON.toJSONString(gmallCorrelationData));  
        } else {  
            // 更新重试次数  
            retryCount += 1;  
            gmallCorrelationData.setRetryCount(retryCount);  
              
            // 将 gmallCorrelationData 对象存入缓存  
            redisTemplate.opsForValue().set(  
                    gmallCorrelationData.getId(),  
                    JSON.toJSONString(gmallCorrelationData),  
                    10, TimeUnit.MINUTES);  
              
            // 判断是否为延迟消息  
            if (gmallCorrelationData.isDelay()) {  
                // 延迟消息重试  
                rabbitTemplate.convertAndSend(  
                        gmallCorrelationData.getExchange(),  
                        gmallCorrelationData.getRoutingKey(),  
                        gmallCorrelationData.getMessage(),  
                        message -> {  
                            message.getMessageProperties().setDelay(gmallCorrelationData.getDelayTime() * 1000);  
                            return message;  
                        }, gmallCorrelationData);  
                  
            } else {  
                // 发送消息  
                rabbitTemplate.convertAndSend(  
                        gmallCorrelationData.getExchange(),  
                        gmallCorrelationData.getRoutingKey(),  
                        gmallCorrelationData.getMessage(),  
                        gmallCorrelationData  
                );  
            }  
        }  
          
    }  
      
    /**  
     * Returned message callback.     * 交换机发送消息到指定队列失败时回调此方法  
     *  
     * @param message    the returned message.  
     * @param replyCode  the reply code.  
     * @param replyText  the reply text.  
     * @param exchange   the exchange.  
     * @param routingKey the routing key.  
     */    
    @Override  
    public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {  
        // 根据固定的key来获取数据对象id  
        String correlationId = (String) message.getMessageProperties().getHeaders().get("spring_returned_message_correlation");  
          
        // 根据这个key在redis中重新获取到数据对象的json数据  
        String strJson = (String) redisTemplate.opsForValue().get(correlationId);  
          
        // 将json转换为数据对象  
        GmallCorrelationData gmallCorrelationData = JSON.parseObject(strJson, GmallCorrelationData.class);  
          
        // 调用重试方法,重新发送消息  
        this.retrySendMessage(gmallCorrelationData, 3);  
    }  
}
```


# 消费端限流
*流量削峰*
* 设置 refetch 属性
* `<rabbit:listener-container refetch="1">` 表示每次从队列中取出 1 个消息进行消费

**什么场景会用到?**
1. 在并发量瞬间增大时, 使用消息队列对特定的操作做流量限流
2. 如在数据库中对写入操作做限流, 保证查询的及时性


# TTL
设置 [[TTL]] 的两种方式
1. 对整个队列设置
2. 设置单个消息的过期时间 (在发送消息时设置)
3. **如果队列与单个消息同时设置了过期时间, 则以最短的时间为准**


# 延迟队列

*RabbitMQ 中**没有提供**延迟队列功能, 但可以使用 TTL+死信队列组合方法可以实现延迟队列的效果*

使用场景
* 下单后, 30 分钟未支付订单取消, 回滚库存等...

实现延迟队列有两种方式
1. 基于死信队列
2. 使用延迟插件

## 基于死信队列实现延迟队列
1. 向一个不会被消费的队列中发送消息, 该队列设置 [[TTL]] 超时时间
2. 当消息达到过期时间后, 消息称为死信, 自动通过死信交换机将信息转发到死信队列
3. 监听死信队列, 当从队列中接收到消息后对该信息进行消费... 即通过死信队列+ [[TTL]] 实现延迟队列

## 使用 x-delay-message 插件实现延迟队列
插件内部就是一个小型数据库, 发送消息通过交换机将信息转发到数据库中, 到达设定时间后在把信息放到队列中进行转发
[下载地址](https://www.rabbitmq.com/community-plugins.html)
1. 下载插件后将插件复制一份到 rabbitMQ 中的 plugins 目录中 `docker cp rabbitmq_delayed_message_exchange-3.9.0. Ez rabbitmq:/plugins`
2. 进入容器内部 plugins 目录查看是否复制成功 `docker exec -it rabbitmq /bin/bash` `cd plugins` 
3. 在 plugins 目录下执行命令启用插件 `rabbitmq-plugins enable rabbitmq_delayed_message_exchange`
4. 重启 RabbitMQ 容器
5. 在 Java 程序中绑定交换机及队列, 发送消息时指定延迟时间即可实现延迟队列

Java 代码中配置:
```java
package com.atguigu.gmall.mq.config;  
  
import org.springframework.amqp.core.Binding;  
import org.springframework.amqp.core.BindingBuilder;  
import org.springframework.amqp.core.CustomExchange;  
import org.springframework.amqp.core.Queue;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
  
import java.util.HashMap;  
import java.util.Map;  
  
  
@Configuration  
public class DelayedMqConfig {  
      
    public static final String EXCHANGE_DELAY = "exchange.delay";  
    public static final String ROUTING_DELAY = "routing.delay";  
    public static final String QUEUE_DELAY_1 = "queue.delay.1";  
      
    /**  
     * 队列  
     *  
     * @return  
     */  
    @Bean  
    public Queue delayedQueue() {  
          
        return new Queue(QUEUE_DELAY_1, true, false, false);  
    }  
      
    /**  
     * 交换机  
     *  
     * @return  
     */  
    @Bean  
    public CustomExchange customExchange() {  
        Map<String, Object> arguments = new HashMap<>();  
        arguments.put("x-delayed-type", "direct");  
        return new CustomExchange(EXCHANGE_DELAY, "x-delayed-message", true, false, arguments);  
    }  
      
    /**  
     * 绑定关系  
     *  
     * @return  
     */  
    @Bean  
    public Binding delayBinding() {  
          
        return BindingBuilder.bind(delayedQueue()).to(customExchange()).with(ROUTING_DELAY).noargs();  
    }  
      
}
```

## 死信队列和延迟插件的区别
- 时序问题: 死信队列要求队列里面的所有消息过期时间一致, 因为队列是先进先出的; 也就是说, 假设第一个消息没有到达过期时间, 即使第二个消息已经达到过期时间也是无法进行消费的, 它必须等待第一个消息出列后才可以被消费; 如果死信队列中的过期时间不一致, 则使用它实现的延迟队列是无法保证可靠性的

# Springboot 整合 RabbitMQ
* Spring 中封装了 @RabbitListener 注解用来监听 rabbitMQ 的消息
* 生产者使用 Springboot 封装的 RabbitTemplate 对象调用 convertAndSend (exchange, routingKey, message) 方法, 向指定的消息队列中发送消息
* 消费者端创建监听器监听消息队列

```java
import com.atguigu.utils.MqConst;  
import com.rabbitmq.client.Channel;  
import org.springframework.amqp.core.Message;  
import org.springframework.amqp.rabbit.annotation.Exchange;  
import org.springframework.amqp.rabbit.annotation.Queue;  
import org.springframework.amqp.rabbit.annotation.QueueBinding;  
import org.springframework.amqp.rabbit.annotation.RabbitListener;  
import org.springframework.stereotype.Component;  
  
/**  
 * @author: Wei  
 * @date: 2022/10/4,19:58  
 * @description:  
 */  
  
@Component  
public class MsgListener {  
	  
	/**  
	 * 将这个监听器与一个队列进行绑定,并设置这个队列的路由/交换机  
	 *  
	 * @param dataString 接收到的消息  
	 * @param message    接收到的消息的具体属性  
	 * @param channel    信道的消息  
	 */  
	@RabbitListener(  
			bindings = @QueueBinding(  
					value = @Queue(value = MqConst.QUEUE_NAME, durable = "true"),  
					exchange = @Exchange(value = MqConst.EXCHANGE_DIRECT),  
					key = {MqConst.ROUTING_KEY}  
			))  
	public void patientTip(String dataString, Message message, Channel channel) {  
		System.out.println(dataString);  
//        System.out.println("message = " + message);  
//        System.out.println("channel = " + channel);  
// 在开启了手动确认后,接收端必须使用 channel.BasicAck(deliveryTag,false)来确认消息已经收到.第一个参数是消息的唯一标识,第二个参数意为是否批量答复
	}  
	  
}
```

[^1]: channel. BasicNack (deliveryTag, true, false) 一个参数表示收到的标签, 第二个表示可以接收所有的信息, 第三表示重回队列,


# 安装 RabbitMQ
1. RabbitMQ 是 Erlang 编写的, 安装必须有 Erlang 语言环境
2. 安装 rpm 包
    1. **rpm -ivh erlang-21.3.8.9-1. El7. X86_64. Rpm**
    2. **rpm -ivh socat-1.7.3.2-1. El6. Lux. X86_64. Rpm**
    3. **rpm -ivh** **rabbitmq-server-3.8.1-1. El7. Noarch. Rpm**
3. 启用管理插件
    1. Rabbitmq-plugins enable rabbitmq_management
4. 修改主机名
    1. /etc/hostname
    2. /etc/hosts 里面前两个地址后面添加上修改后的主机名
5. 启动 RabbitMQ 服务 `systemctl start rabbitmq-server`
6. 访问 http://192.168.6.200:15672/ 测试
    1. 默认的用户名和密码 guest 只支持本地的访问, 不支持远程访问
    2. 创建新的用户才能进行远程访问
    3. 在进行任何操作前, 需要先为这个账户分配权限

# RabbitMQ 集群搭建
* 使用 HAProxy, 实现[[负载均衡]]的效果*
## 单机多实例部署
1. 确定当前 RabbitMQ 服务正常使用
2. 停止 rabbitMQ 服务
3. 创建多个节点指定不同的端口号
```shell
	RABBITMQ_NODE_PORT=5672 RABBITMQ_NODENAME=rabbit1 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15672}]"  rabbitmq-server -detached
```
3. 选定一个主节点
```shell
	rabbitmqctl -n rabbit1 stop_app
	rabbitmqctl -n rabbit1 reset
	rabbitmqctl -n rabbit1 start_app
```
4. 重置子节点并将子节点加入主节点的集群
	```shell
	rabbitmqctl -n rabbit2 stop_app
	rabbitmqctl -n rabbit2 reset
	rabbitmqctl -n rabbit2 join_cluster rabbit1
```

 >[!warning] 单机的用户无法在集群登录, 需要重新创建一个用户

## 集群的配置
默认的集群只复制交换机和队列, 但不复制队列中的内容

想要在某台服务器宕机后也能正常工作, 就需要为这个集群配置**镜像队列**

在管理后台定制 Policies, 使用 ha-mode, 设置值为 all, 表示复制所有队列内的消息


## 集群的负载均衡 HA-proxy
[[负载均衡#策略]]

> [!tip] HA-proxy
> 类似于 Nginx, HAProxy 提供高可用性、负载均衡以及基于 TCP 和 HTTP 应用的代理，支持虚拟主机。HAProxy 实现了一种事件驱动、单一进程模型，此模型支持非常大的并发连接数。

1. 下载 HAProxy
2. 配置 HAProxy

# 补充
- 它发送的数据默认都是字符串格式, 需要通过配置消息转换器将他发送的消息转换为 [[Json]] 格式进行发送 #待完善/不清晰 