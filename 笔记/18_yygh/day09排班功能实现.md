# 医院排班功能实现
## 按照大小科室树形显示
DepartmentVo属性:
- depcode 科室编号
- depname 科室名称
- List\<DepartmentVo> children 下级节点
> **按照大科室进行分组**
1. 查询当前医院下的所有科室
2. 将查询出来的结果使用stream流的方式,按照各个大科室进行分组(*按照bigcode分组*)
3. 分组后会得到一个Map集合,这个Map的key是大科室的bigcode,value是这个大科室下的所有小科室
4. 通过entrySet的方式遍历这个map,分别对大小科室进行封装,,最后统一封装至大科室,这个map没遍历一次,就将一个大科室的所有数据封装完毕,将他存入返回结果的List中
## 显示该科室可以预约的日期和号数
基于 [[MongoDB#MongoTemplate]] 聚合查询实现功能
通过医院与科室编码,查询出当前科室的所有排班信息, 并按照日期分组,分别统计出总排班号数和剩余可用号数
点击某个科室显示这个科室的所有预约的日期(管理员可以查看所有日期的信息)
1. 根据医院编号 + 科室查询
2. 根据 workdate 进行分组
3. 统计医生/号/可用号
# 显示日期医生排班数据
- 通过排班日期查询数据时,MongoDB中的日期格式是Date,需要使用joda-time工具类将String类型转换为Date类型才能查询出数据
# NUXT 搭建项目用户系统前端环境
- 基于 Vue.js 的轻量级服务端渲染技术框架
- 能够更好的SEO