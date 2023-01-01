# Ajax
Asynchronous JavaScript And XML
使用 Js 发送异步请求,以 XML| **Json** 方式回显数据
## 同步请求
- 超链接
- 表单
- Js
- ...
> 同步请求不足
> * 当响应之前,整个页面时无法操作的
> * 即使要刷新的局部的某个数据,也必须刷新整个页面,这降低了服务器的运行速度

## 异步请求(局部刷新)
- 使用 Ajax 发送同步和异步请求,它可以在不发生页面跳转的情况下刷新数据
- 实际研发中,一般结合同步与异步共用

## Ajax 缺陷
- 从运营角度考虑,使用 Ajax 不利于网站 SEO(在搜索引擎中关键字匹配不会匹配到 Ajax 渲染数据,网站排名靠后)

# Axios
 它是发送 Ajax 请求的框架
## Axios 中两种参数
1. params(Query String Paramters 查询字符串参数),支持 Get & Post 提交请求方式
2. data(请求体参数),支持 Post 提交方式,它传递过来就是 Json 格式
## Axios 中的 this 关键字
- 在函数内部时,this 指向 Window 对象
- 在函数外部时,this 指向 Vue 对象
- 与 [[Vue#Vue 中的 this 关键字]] 相反
