# 数据字典导入
1. 点击导入按钮,弹出选择框,在框内选择文件
2. 调用接口,使用[[EasyExcel]]读取内容加入到数据库
	1. 通过MultipartFile对象可以获得上传的文件
	2. 创建service实现方法
	3. 创建监听器,将他交由spring管理
	4. 在service注入
# 使用[[Redis]]作为缓存
1. 添加 Redis 信息到配置文件
2. 在公共的工具模块下面创建 Redis 配置类,在这个配置类上方使用 @EnableCaching 表示开启缓存支持
3. 使用 [[SpringCache]] 对应的注解对方法进行标记即可
# MongoDb 内容
[[MongoDB]]
