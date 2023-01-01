# 概述

与Apache poi 的不同

| EasyExcel               | Apache poi                             |
| ----------------------- | -------------------------------------- |
| 一行行的加载到内存/解析 | 一次性把所有的文件加载到内存解析与处理 |                            |

[参阅官方文档](https://easyexcel.opensource.alibaba.com/docs/current/)
# 使用步骤

## 写操作
1. 引入官方推荐的版本依赖 
2. 创建一个实体类,属性与表头名称对应
3. 可以使用 @ExcelProperty("表头名称") 注解映射表头名称
4. 直接调用 EasyExcel 中的静态方法 *write/read* 即可
## 读操作
1. 设置 @ExcelProperty(value="表头名称",index=?) index 表示表格中的第几列
2. 创建一个监听器类,继承 AnalysisEventListener\<entity>,重写 invoke() 和 doAfterAllAnalysed() 方法
3. invoke() 方法会从 excel 表格一行一行的读取,将每行的数据封装到实体类对象中,**它从第二行开始读取,因为它认为第一行是表头**,使用 invokeHeadMap() 方法读取表头
4. doAfterAllAnalysed() 读取完后执行的方法
通过批量添加优化效率
- 每一行数据都提交一次数据库,频繁 I/O 大幅降低效率

# 注意

## 监听器管理问题 
#problem
- 监听器问题:如果监听器是交由 [[Spring]] 管理的,监听器对象会变为单例,那么在并发读取操作时会回调同一个对象,就无法区分是哪个文件读取到的数据,为了避免这个问题,**监听器不要交给 Spring 管理** 
- 分析:监听器不交给 Spring 管理,他就无法通过自动注入来使用 Mapper 插入数据,为了解决这个问题,使用参数传递的方式(*思想*)来将 Mapper 传递给监听器使用
	1. 给监听器添加 Service 属性并提供有参构造
	2. 在调用 read() 方法时传入 new Lisener(Service) 即可解决这些问题

# 关于 Excel 的知识点补充
- 每个 Excel 文件是一个 workbook
- 每个 workbook 里有多个 sheet
- sheet 包含行/列和表头