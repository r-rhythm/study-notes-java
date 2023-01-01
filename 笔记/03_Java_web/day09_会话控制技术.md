# Cookie
## 简介
> 他是服务器存放在浏览器的一小份数据, 之后浏览器每次访问服务器都会携带这份数据, 以便于服务器区分不同的浏览器
## Cookie 的工作原理
1. 浏览器端向服务器端发送请求
2. 服务器端创建 Cookie 对象并存储用户信息并将它发送给浏览器端
3. 浏览器端之后再次发送请求, 携带 Cookie 对象
4. 服务器端就能通过 Cookie 来区分浏览器端

## Cookie 基本使用
创建 Cookie
   1. `new Cookie("key","value")`
   2. 响应给浏览器端 `response.addCookie(cookie)`

获取 Cookie
   1. `request.getCookie();`

修改 Cookie
   4. 直接创建一个同名的 Cookie 进行覆盖

## Cookie 的字符集问题
Cookie 的 name 不支持中文, value 支持, 如需使用中文需要设置字符集, 不推荐使用中文

## Cookie 持久化

Cookie 是会话级别, 即浏览器不重启便一直有效

想要重启后依然生效, 就是 Cookie 的持久化
* `setMaxAge(ss)`
  * `ss>0 设置Cookie的持久时间为ss秒`
  * `ss=0 设置Cookie立即失效`
  * `ss<0 设置Cookie恢复会话级别`

## Cookie 的有效路径

它的默认有效路径在上下文
* `setPath()设置Cookie有效路径` ,  它不能设置多个路径, 后面的会把前面的覆盖
* `setDomain()跨域`

## Cookie 应用示例
需求: 使用 Cookie 实现 7 天免输入在登录页面
  1. 获取用户名及密码
  2. 将用户名及密码存储在 Cookie, 持久化 7 天
  3. 在登录页面获取 Cookie 中的数据, 实现免输入的效果

## Cookie 不足
* value 是 string 类型, 存储数据不够灵活
* Cookie 存储在浏览器端, 相对不安全
* 当 Cookie 数量过多, 会浪费数据流量
* 浏览器厂商都会对 Cookie 有限制, 一般数量在 200~500, 大小在 2~5 kb



# Session
## 简介
Session 是四个域对象之一
Session 也是两个会话对象之一 (cookie, session)

## Session 的工作原理
1. 浏览器向服务器发出第一次请求
2. 服务器创建 Session 对象, 同时创建一个特殊的 Cookie 对象 `(name=JSESSIONID,VALUE=SessionId)`
3. 浏览器以后再次发送请求, 携带 Cookie 对象
4. 服务器通过 Cookie 的 value 所携带的 SessionId 找到 Session 对象, 从而分别不同的浏览器

浏览器向服务器发送请求, 但不是第一次请求, 原则上不新建 Session 对象, 获取之前已经创建好的 Session 对象直接使用
是否能获取到 Session 对象, 由两个因素决定
1. 是否携带了特殊的 cookie
- 存在, 则继续判断 Session 是否存在, (Session 默认最大空闲时间是 30 分钟), 如果不存在重新创建 Session
- 不存在则直接创建 Session 对象

**session 默认会话级别的原因是 Cookie 默认是会话级别的** ~~???~~

## 持久化 Session
1. 持久化 Cookie
2. 持久化 Session,设置 Session 最大空闲时间
     * 全局设置, 在 web. Xml 中设置
     * 局部设置 `session.setMaxInactiveInterval(ss)

## Session 四种失效情况
* 强制失效
  * `session.invalidate();`

## Session 钝化与活化
 * Session 域对象浏览器不关闭不更换即有效, 即它的有效性与服务器无关
 * Sessoin 是由服务器创建的
钝化就是关闭服务器前将 Session 系列化后存入硬盘

**Sessin 中的对象如需一同序列化, 则这个对象必须实现序列化接口 Serializable**


# Cookie 和 session 有什么不同?
1. 存储位置的不同, cookie 存储在客户端中. Session 存储在服务器中
2. Cookie 只能存储 string 类型的对象, session 可以存储任意类型的对象
3. Session 比 cookie 更具有安全性
4. Session 存储在服务器端, 过多的 session 会降低服务器性能