
# NPM
[[前端基础知识#NPM]]


# 模块化开发

## ES5模块化

> 每个文件就是一个模块，有自己作用域。在一个文件里定义的变量、函数、都是私有的，对其他文件不可见。

```js
//声明一个求和函数
function sum(a,b){
    return parseInt(a) + parseInt(b)
}


//声明一个求差函数
function sub(a,b){
    return parseInt(a) - parseInt(b)
}

//导出函数
module.exports={
    sum,
    sub
}
```

```js
//导入func.js模块
var func = require("./func")

//调用func.js中定义的函数
console.log(func.sum(6,1));
console.log(func.sub(10,5));
```



## ES6模块化规范

> ECMAScript 6

**注意,ES6的模块化无法在Node.js中执行，需要用Babel编辑成ES5后再执行。**

```js
//使用export声明一个求乘积的函数并导出
export function mul(a,b){
    return parseInt(a) * parseInt(b);
}

//声明一个求商的函数并导出
export function div(a,b){
    return parseInt(a) / parseInt(b);
}
```

```js
//导入函数
import { mul,div } from "./func";

console.log(mul(2,2));
console.log(div(10,5));
//此时是不能直接运行的,需要安装Babel转码器插件
//SyntaxError: Cannot use import statement outside a module
```

### 安装Babal插件

```shell
npm install -g babel-cli
#查看是否安装成功
babel --version
```

```shell
# --out-dir 或 -d 参数指定输出目录
babel src -d out
```



