# 插值表达式
```js
<!-- id标识vue作用的范围 -->
<div id="app">
    <!-- {{}} 插值表达式，绑定vue中的data数据 -->
    {{ message }}
</div>

// 引入vue包
<script src="vue.min.js"></script>

<script>
    // 创建一个vue对象
    new Vue({
        el: '#app', // 绑定vue作用的范围
        data: { // 定义页面中显示的模型数据
            message: 'Hello Vue!'
        }
    })
</script>
```
# 指令
- v-bind 单项绑定
- v-model 双向绑定
```js
// 单向绑定
<body>
    <div id="app">
        <!-- 在标签属性内获取绑定的data值 -->
        <input type="text" v-bind:value="message"></input>
    </div>
    <script src="vue.min.js"></script>
    <script>
        new Vue({
            el: '#app',
            data: {
                message:"hello,vue"
            }
        })
    </script>
</body>

// 双向绑定
<body>
    <div id="app">
        <div id="app">
            1、{{info}}
            <br/>
            2、<input type="text" :value="info"/>
            <br/>
            3、<input type="text" v-model="info"/>
        </div>
    </div>
    <script src="vue.min.js"></script>
    <script>
        new Vue({
            el: '#app',
            data: {
                info:'JavaScript魔法世界'
            }
        })
    </script>
</body>
```
- v-on 绑定事件
```js
<body>
    <div id="app">
        <input type="button" value="按钮一个" @click="f1"></input>
        <!-- <input type="button" value="按钮一个" v-on:click="f1"></input> -->
    </div>
    <script src="vue.min.js"></script>
    <script>
        new Vue({
            el: '#app',
            data: {},
            methods:{
                f1(){
                    alert("hello")
                }
            }
        })
    </script>
</body>
```
- v-if 判断
- v-for 循环

# Vue 高级
## 组件 Components
- 使用 Vue 组件可以自定义标签,自定义标签的名称与内容
- 扩展 HTML 元素,封装可重用的代码
```js
<body>
    <div id="app">
        <Navbar></Navbar>
    </div>
    <script src="vue.min.js"></script>
    <script>
        new Vue({
            el: '#app',
            data: {}
            components:{
                'Navbar':{
                    template:'<ul><li>首页</li><li>测试</li></ul>'
                }
            }
        })
    </script>
</body>
```
## 生命周期
- created() 在页面数据渲染之前执行
- mounted() 在页面数据渲染之后执行

# Vue 中的 this 关键字
- 在函数内部时,this 指向 **Vue 对象**
    var vm = new Vue({});
    var this = vm;
- 在函数外部时,this 指向 **Window 对象**

# Vue路由
- 路由允许我们通过不同的 URL 访问不同的内容

# 实现页面跳转
1. 在函数中直接调用路由跳转 this.$router.push(path:{'目标路径'})
2. 配置隐藏路由
	1. 定义路由,设置 `hidden: true` 属性
	2. 使用 `<router-link :to="'target/path/携带参数'" ></router-link>` 标签跳转
# 获取页面跳转时携带的参数
- 如果是通过路径 / 传输的参数,在目标路径通过 `this.$route.params.id` 方法可以获得请求 URL 中携带的参数,注意这里是 **route** 不是 router
- 如果是通过 ? 和 & 拼接 URL 所传递的参数,则使用 `this.$route.query.参数名` 来获取到 URL 中拼接的参数