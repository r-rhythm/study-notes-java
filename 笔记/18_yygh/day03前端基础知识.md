# 搭建环境
1. 解压项目文件到工作区,下载相关依赖 npm install

## vue-element-template 目录结构
[[前端基础知识#vue-element-template 目录结构]]

##  医院设置管理前端
1. 前端框架默认请求mock模拟服务器,把他修改为请求我们的服务器,修改.env.development![[Pasted image 20221019102223.png]]
2. 在本地服务创建两个接口(和mock一致,可以使用mock登录一次在控制台查看Network查看需要返回的数据),这一步需要**注意响应的状态码**
3. 修改前端 api / user.js 为接口的路径,修改请求的服务器后会产生跨域问题
4. 把 store/modules/user.js 的 logout 函数调用的方法注释掉![[Pasted image 20221019105637.png]]
# ![[跨域]]

# 配置 Nginx 反向代理
- 在搭建了分布式服务后,每个服务都对应了不同的路径.而前端的 VUE_APP_BASE_API 只合适配置一个路径,这就导致了无法访问不同的访问
- 通过配置 [[Nginx]] 反向代理来解决这一问题
安装 Windows 版本的 Nginx
- 前端的 VUE_APP_BASE_API 配置为 Nginx 的地址
- 在 Nginx 中配置文件中配置代理转发

# 前端
1. 添加路由(*参考其他路由进行修改即可*)
2. 创建路由对应跳转的页面 component:()=> import('@/xxxx/xxx'),在views文件夹中创建对应的页面
3. 在api文件夹 创建js文件,定义要调用的接口信息
```js
import request from '@/utils/request'

const api_name = '/hosp/hospital-set'

export default {  // export表示能从外部调用,使用 default 就不用每个方法都 export function

    getPageQuery(current,limit,seachObj){  // 方法的参数参考后端,接口中有几个参数就传几个参数
        return request({
            url: `${api_name}/findPageQuery/${current}/${limit}`, // 注意是飘号不是单引
            method: 'post',
            // 如果没有RequestBody params:searchObj
            // 如果有RequestBody data:searchObj 表示将他转换为 json 传递
            data:seachObj
        })
    }
}
```
4. 在list页面中调用接口获得数据并显示
# 页面结构
```js
// 1.引入定义接口的 js 文件
import hospset_api from '@/api/yygh/hospset'

export default {
    // 2.编写vue代码
    data(){ // 定义初始值
        return{
            list:[]
        }
    },

    created(){ // 在页面渲染前执行的钩子函数
    // 进入页面就调用方法得到数据
        this.fetchData()
    },

    methods:{
        fetchData(){
        }
    }
}
```