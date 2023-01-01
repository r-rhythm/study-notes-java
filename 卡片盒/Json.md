# Json 简介
 JavaScript Object Notation
 Json 可以在不同语言,不同生态间进行数据交互
# Json 转换对象
- 使用 FastJson 工具,可以完成集合与对象间的转换
# Json 数据格式
1. 对象格式:{"name":"lucy":"20"}
2. 数组格式:["a","b"] ,Java中的List集合转换为Json就会变为数组
3. 两种格式混用
```json
{
	"name":"lucy",
	"loves":[
		"吃饭",
		"睡觉"
	]
}
```