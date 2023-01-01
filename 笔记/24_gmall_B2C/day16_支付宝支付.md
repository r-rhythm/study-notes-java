# 简介
支付宝采用公私钥来保证信息传递的安全性
[开发文档](https://opendocs.alipay.com/open/02e7gq?ref=api&scene=20)
![[支付宝-支付时序图.png]]

# 搭建 service-payment 支付模块
1. 引入 alipay 依赖 [依赖地址](https://search.maven.org/artifact/com.alipay.sdk/alipay-sdk-java)
2. 实例化 alipay 客户端对象 [jdk 示例代码](https://opendocs.alipay.com/open/02np94)
3. 调用 alipay 接口


# 异步通知
对于支付成功后的扣减库存等操作, 使用异步调用的方式, 将他们隐藏在服务器内部, 就不会因为用户的操作而打断程序进程
1. 在请求时在 request 对象设置回调地址 `notify_url`
2. 支付成功后, 支付宝会已 Post 请求的方式向这个回调地址发起请求并携带数据
3. [异步通知说明](https://opendocs.alipay.com/open/204/105301?ref=api)

支付宝在通过异步通知的方式调用我们的接口并返回数据后, 必须按照以下的方式进行数据校验:
1. 验证通知校验中的 `out_trade_id` 是否为本系统产生的交易号
2. 判断 `total_amount` 是否确实为该订单的实际金额 (即商家创建订单时的金额)
3. 验证 `app_id` 是否为商家本身
4. 如果有任何一个验证不通过, 则为异常通知, 忽略

同步回调与异步回调的区别:
1. 同步回调是支付成功后我们在本地发起的
2. 异步回调则是支付成功后支付宝主动发起的, 本地测试需要使用内网穿透暴露端口
3. 使用异步回调来进行商品库存的扣减等操作, 可靠性更高. 用户的操作不会打断内部程序的运行

异步调用后处理流程
1. 使用支付宝提供的 api 进行签名验证 `AlipaySignature.rsaChechV1()`
2. 验签成功后, 按照异步通知中校验参数的步骤对支付结果中的数据进行二次验证
# 内网穿透


# 退款处理