# JS 的组成部分
- ECMAScript: ECMA 组织制定的一种规范
- BOM(Browse Object Model):浏览器对象模型
- DOM(Document Object Model):文档对象模型
# DOM
- 使用 dom 把 HTML 的标签在内存中分配为一个树形结构
- dom 中有元素/属性/值对象

# Js 关键字
- export : Js 默认函数都是私有的,外部无法调用,想要外部调用需要使用这个关键字标记
# Js 中的 this 关键字
- 在函数内部时,this 指向触发当前函数的对象
- 在函数外部时,this 指向 Window 对象
- Js 中的 this 关键字是**可变的**

# Js 中实现倒计时的方法
定时器
- setInterval("js 代码",执行时间间隔 ms) 方法,在固定的时间间隔中重复执行 js 代码
- clearInterval() 方法,清除定时器效果
- 用一个变量接收 setInterval(),在 clearInterval()形参中传入这个变量即可

# Dom 引入外部 Js 文件的方法
 1. 创建一个 script 标签(*元素*) `document.createElement("script")`
 2. 设置这个标签的两个属性 `script.type='type/javascript',script.src = 'http://res.wx.qq.com/connect/zh_CN/htmledition/js/wxLogin.js'`
 3. 将这个自建的标签追加到 body 标签内 `document.body.appendChild(script)`