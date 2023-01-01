# 基本构造
- 行
- 头
- 体
# 常见的请求头/响应头
请求头
- User-Agent 返回访问终端(*浏览器类型*)
- language 语言环境
响应头
- Content-disposition 表示响应的数据要以下载的方式打开
# 常见状态码
- 304 读取缓存中的数据
- 403 无权限
# 常见请求方式
- Get
- Post
- Put
- Delete
- OPTIONS
# 下载
1. 下载实质上就是服务器响应数据给浏览器,所以会用到 response 响应信息
2. 必须设置响应头信息为 Content-disposition 例如  `setHeader("Content-disposition","attachment;fileName"+fileName+"文件类型后缀"`

# 常用补充
- 当外部传入的数据在系统中并没有对应的实体类无法使用@RequestBody 获取数据时,可以使用 request 可以获得所有的请求信息
- 使用 `request.getParameterMap()` 方法可以获得所有的请求参数,并封装到集合内.他的返回结果 `Map<String, String[]> parameterMap` String\[] 数组的意义是为了接收同一个 key 有多个 value 的情况,例如前端的复选框等

# 常见问题
## OPTIONS 预请求问题
- 某些浏览器在发送请求前会先发送一次 OPTIONS 请求(*预检请求*),它会先检查一些接口是否能够联通
- 此时如果接口[[Cookie#Cookie 如何跨域传递|使用请求头传递数据]]时,不要在接口内判空抛异常等,不然第一次的 OPTIONS 请求不携带数据就会让这个浏览器预检失败,导致浏览器将第二次的正常请求短路掉,不再请求这个接口
