# JavaScript（JS）分类

如果系统学习或用于面试，一般可以按 **语言特性、运行环境、功能领域、技术栈** 等多种方式分类。下面给你一个 **比较完整的 JavaScript 分类体系**。

------

## 一、按 JavaScript 语言核心能力分类（最基础）

这是 **JS语言本身的组成部分**。

```
JavaScript
│
├── 基础语法
├── 数据类型
├── 运算符
├── 控制结构
├── 函数
├── 对象
├── 原型与继承
└── ES6+ 新特性
```

### 1 基础语法

主要是 JS 的基本结构。

- 变量声明
  - `var`
  - `let`
  - `const`
- 注释
- 分号规则
- 作用域

示例：

```javascript
let name = "Tom"
const age = 20
```

------

### 2 数据类型

JS 数据类型分为：

**基本类型**

```
String
Number
Boolean
Undefined
Null
Symbol
BigInt
```

**引用类型**

```
Object
Array
Function
Date
RegExp
Map
Set
```

------

### 3 运算符

```
算术运算符
比较运算符
逻辑运算符
赋值运算符
位运算符
三元运算符
```

例如：

```javascript
let result = a > b ? a : b
```

------

### 4 控制结构

```
if
switch
for
while
do...while
break
continue
```

------

### 5 函数

函数是 JS 的核心。

```
普通函数
匿名函数
箭头函数
回调函数
高阶函数
闭包
```

例子：

```javascript
const add = (a,b)=>{
    return a+b
}
```

------

### 6 对象

JS 是 **基于对象的语言**。

对象相关：

```
对象创建
this
构造函数
class
```

示例：

```javascript
class User{
    constructor(name){
        this.name = name
    }
}
```

------

### 7 原型与继承（JS核心）

```
prototype
__proto__
原型链
继承
```

------

### 8 ES6+ 新特性

现代 JS 基本都用 ES6+。

```
let / const
箭头函数
模板字符串
解构赋值
Promise
async/await
Module模块
class
```

------

## 二、按 JavaScript 运行环境分类

JS 可以运行在多个环境。

```
JavaScript
│
├── 浏览器端 JS
├── Node.js
├── 移动端 JS
└── 桌面端 JS
```

### 1 浏览器端 JavaScript

最常见。

主要操作：

```
DOM
BOM
事件
Ajax
Fetch
```

例如：

```javascript
document.getElementById("btn")
```

------

### 2 Node.js

服务器端 JS。

常见用途：

```
Web服务器
API服务
工具脚本
构建工具
```

常见框架：

```
Express
Koa
NestJS
```

------

### 3 移动端 JS

用 JS 写 APP。

技术：

```
React Native
Weex
Ionic
```

------

### 4 桌面端 JS

用 JS 写桌面软件。

技术：

```
Electron
NW.js
```

例如：

```
VSCode
```

就是 Electron 写的。

------

## 三、按前端开发功能分类

前端开发中 JS 通常按 **功能模块**分类。

```
JavaScript
│
├── DOM操作
├── 事件处理
├── 网络请求
├── 数据处理
├── 浏览器API
└── 动画与交互
```

------

### 1 DOM 操作

控制页面元素。

```
获取元素
修改内容
修改样式
创建元素
删除元素
```

例如：

```javascript
document.querySelector(".box").style.color="red"
```

------

### 2 事件处理

用户交互。

```
click
mouseover
keydown
submit
scroll
```

------

### 3 网络请求

JS 调用后端接口。

技术：

```
Ajax
Fetch
Axios
WebSocket
```

例如：

```javascript
axios.get("/api/user")
```

------

### 4 数据处理

JS 非常擅长数据处理。

```
数组操作
JSON处理
Map / Reduce
Filter
```

例如：

```javascript
arr.filter(item=>item.age>18)
```

------

### 5 浏览器 API

浏览器提供大量 API。

```
localStorage
sessionStorage
History API
Geolocation
Canvas
WebRTC
```

------

## 四、按前端技术生态分类（现代开发）

现代 JS 技术栈。

```
JavaScript生态
│
├── 框架
│   ├── Vue
│   ├── React
│   └── Angular
│
├── 状态管理
│   ├── Vuex
│   ├── Pinia
│   └── Redux
│
├── 路由
│   ├── Vue Router
│   └── React Router
│
├── HTTP库
│   ├── Axios
│   └── Fetch
│
├── 构建工具
│   ├── Vite
│   ├── Webpack
│   └── Rollup
│
└── 运行环境
    └── Node.js
```

------

## 五、JavaScript 总体知识体系（最完整结构）

给你一张 **完整结构图**：

```
JavaScript
│
├── 语言基础
│   ├── 数据类型
│   ├── 函数
│   ├── 对象
│   ├── 原型
│   └── ES6+
│
├── 浏览器编程
│   ├── DOM
│   ├── BOM
│   ├── 事件
│   └── Ajax
│
├── 异步编程
│   ├── Callback
│   ├── Promise
│   ├── async/await
│   └── EventLoop
│
├── 模块化
│   ├── CommonJS
│   ├── ES Module
│   └── AMD
│
├── 网络通信
│   ├── Ajax
│   ├── Fetch
│   ├── Axios
│   └── WebSocket
│
└── JS生态
    ├── Node.js
    ├── Vue
    ├── React
    └── 构建工具
```

------

💡 **如果是面试准备，我可以再给你一套：**

**《JavaScript 面试知识体系图（超完整）》**

会包含：

```
JS核心原理
原型链
this
闭包
作用域
事件循环
Promise
浏览器渲染
模块化
```

很多大厂面试都围绕这几个点。

# JS

## **JavaScript 语言基础**

可以理解为：**不依赖浏览器、不依赖框架，仅属于 JavaScript 语言本身的核心知识体系**。
在学习或面试中，通常可以分为 **10 大基础模块**。

------

### 一、JavaScript 语言基础整体结构

```text
JavaScript语言基础
│
├── 1 变量与作用域
├── 2 数据类型
├── 3 运算符
├── 4 控制流程
├── 5 函数
├── 6 对象
├── 7 原型与继承
├── 8 this 关键字
├── 9 异步编程基础
└── 10 ES6+ 新特性
```

------

### 二、变量与作用域

变量是存储数据的容器。

#### 变量声明

```javascript
var a = 10
let b = 20
const c = 30
```

区别：

| 关键字 | 特点                     |
| ------ | ------------------------ |
| var    | 函数作用域，存在变量提升 |
| let    | 块级作用域               |
| const  | 常量，不可重新赋值       |

------

#### 作用域

作用域决定变量可访问范围。

分类：

```text
全局作用域
函数作用域
块级作用域
```

示例

```javascript
function test(){
    let a = 10
}
```

------

### 三、数据类型

JavaScript 数据类型分为 **两大类**。

#### 1 基本数据类型（Primitive）

```text
String
Number
Boolean
Undefined
Null
Symbol
BigInt
```

示例：

```javascript
let name = "Tom"
let age = 18
let flag = true
```

------

#### 2 引用数据类型（Reference）

```text
Object
Array
Function
Date
RegExp
Map
Set
```

示例

```javascript
let user = {
    name:"Tom",
    age:18
}
```

------

### 四、运算符

JS 常见运算符。

#### 1 算术运算符

```text
+
-
*
/
%
++
--
```

------

#### 2 比较运算符

```text
>
<
>=
<=
==
===
!=
```

重点：

```text
==   类型转换比较
===  严格比较
```

------

#### 3 逻辑运算符

```text
&&
||
!
```

------

#### 4 赋值运算符

```text
=
+=
-=
*=
/=
```

------

### 五、控制流程

程序执行的控制结构。

#### 1 条件语句

```javascript
if (age > 18) {
    console.log("成年人")
}
```

结构：

```text
if
if...else
if...else if
switch
```

------

#### 2 循环语句

```text
for
while
do...while
for...in
for...of
```

示例：

```javascript
for(let i=0;i<5;i++){
    console.log(i)
}
```

------

### 六、函数

函数是 JS 最重要的部分。

#### 1 普通函数

```javascript
function add(a,b){
    return a+b
}
```

------

#### 2 函数表达式

```javascript
const add = function(a,b){
    return a+b
}
```

------

#### 3 箭头函数

```javascript
const add = (a,b)=>a+b
```

------

#### 4 高阶函数

函数作为参数或返回值。

```javascript
arr.map(item => item*2)
```

------

#### 5 闭包

函数访问外部变量。

```javascript
function outer(){
    let a = 10
    return function(){
        console.log(a)
    }
}
```

------

### 七、对象

JavaScript 是 **基于对象的语言**。

#### 创建对象

```javascript
let user = {
    name:"Tom",
    age:20
}
```

访问属性：

```javascript
user.name
user["age"]
```

------

#### 构造函数

```javascript
function User(name){
    this.name = name
}

let u = new User("Tom")
```

------

#### class 语法

ES6 新写法：

```javascript
class User{
    constructor(name){
        this.name = name
    }
}
```

------

### 八、原型与继承

JS 通过 **原型链** 实现继承。

核心概念：

```text
prototype
__proto__
constructor
原型链
```

示例：

```javascript
function Animal(){}
Animal.prototype.eat = function(){
    console.log("eat")
}
```

------

### 九、this 关键字

`this` 指向 **当前调用对象**。

不同情况：

| 场景     | this指向        |
| -------- | --------------- |
| 普通函数 | window / global |
| 对象方法 | 对象本身        |
| 构造函数 | 实例对象        |
| 箭头函数 | 外层作用域      |

示例：

```javascript
const user = {
    name:"Tom",
    say(){
        console.log(this.name)
    }
}
```

------

### 十、异步编程基础

JS 是 **单线程 + 异步模型**。

常见方式：

```text
回调函数 (callback)
Promise
async / await
```

------

#### Promise

```javascript
new Promise((resolve,reject)=>{
    resolve("success")
})
```

------

#### async / await

```javascript
async function test(){
    const res = await fetch("/api")
}
```

------

### 十一、ES6+ 新特性

现代 JS 的重要部分。

主要包括：

```text
let / const
箭头函数
模板字符串
解构赋值
展开运算符
Promise
class
模块化(import/export)
Symbol
Map / Set
```

示例：

```javascript
const user = {name:"Tom",age:20}

const {name,age} = user
```

------

### 十二、JavaScript语言基础完整知识树

```text
JavaScript语言基础
│
├── 变量
│   ├── var
│   ├── let
│   └── const
│
├── 数据类型
│   ├── 基本类型
│   └── 引用类型
│
├── 运算符
│
├── 控制流程
│   ├── 条件
│   └── 循环
│
├── 函数
│   ├── 普通函数
│   ├── 箭头函数
│   └── 闭包
│
├── 对象
│
├── 原型链
│
├── this
│
├── 异步编程
│   ├── Promise
│   └── async/await
│
└── ES6+
```

------

如果你是 **Vue3 或前端面试准备**，我可以再给你整理一份：

**《JavaScript 面试核心原理 20 讲》**

包括：

```text
作用域链
闭包
原型链
this指向
事件循环(Event Loop)
Promise原理
模块化
浏览器渲染
```

很多 **大厂前端面试基本都围绕这些核心原理。**

**JavaScript 浏览器编程**指的是：
**JavaScript 在浏览器环境中操作页面、处理用户交互、与服务器通信等能力。**

## JS 浏览器编程

浏览器提供了大量 **Web API**，JS 可以调用这些 API 实现页面功能。

------

### 一、JavaScript 浏览器编程整体分类

浏览器端 JS 一般可以分为 **6 大类**：

```text
JavaScript浏览器编程
│
├── 1 DOM编程（操作页面）
├── 2 BOM编程（操作浏览器）
├── 3 事件编程（用户交互）
├── 4 网络通信（HTTP请求）
├── 5 浏览器存储（数据存储）
└── 6 浏览器高级API
```

------

### 二、DOM 编程（Document Object Model）

**DOM 编程 = 操作网页内容**

DOM 是浏览器把 HTML 解析成的 **树形结构对象模型**。

HTML：

```html
<div id="app">
  <p>Hello</p>
</div>
```

DOM结构：

```text
document
   │
   └── div
        │
        └── p
```

------

### DOM常见操作分类

```text
DOM编程
│
├── 获取元素
├── 修改元素
├── 操作样式
├── 创建元素
└── 删除元素
```

------

#### 1 获取元素

```javascript
document.getElementById("app")

document.querySelector(".box")

document.querySelectorAll("p")
```

------

#### 2 修改元素内容

```javascript
element.innerText = "Hello JS"

element.innerHTML = "<b>Hello</b>"
```

------

#### 3 修改样式

```javascript
element.style.color = "red"

element.classList.add("active")
```

------

#### 4 创建元素

```javascript
const div = document.createElement("div")
document.body.appendChild(div)
```

------

#### 5 删除元素

```javascript
element.remove()
```

------

### 三、BOM 编程（Browser Object Model）

BOM 用于 **操作浏览器本身**。

核心对象：

```text
window
│
├── location
├── history
├── navigator
└── screen
```

------

#### 1 window

浏览器全局对象。

```javascript
window.alert("hello")
```

------

#### 2 location（地址栏）

```javascript
location.href = "https://google.com"

location.reload()
```

------

#### 3 history（浏览历史）

```javascript
history.back()

history.forward()
```

------

#### 4 navigator（浏览器信息）

```javascript
navigator.userAgent
```

------

### 四、事件编程（Event）

浏览器是 **事件驱动模型**。

常见事件：

```text
鼠标事件
键盘事件
表单事件
页面事件
```

------

#### 1 鼠标事件

```text
click
dblclick
mouseover
mouseout
mousedown
mouseup
```

示例：

```javascript
button.onclick = function(){
    alert("clicked")
}
```

------

#### 2 键盘事件

```text
keydown
keyup
keypress
```

------

#### 3 表单事件

```text
submit
change
input
focus
blur
```

------

#### 4 页面事件

```text
load
resize
scroll
```

------

### 五、网络通信（Ajax / HTTP）

浏览器 JS 可以向服务器发送请求。

主要方式：

```text
Ajax
Fetch
Axios
WebSocket
```

------

#### 1 Ajax（传统）

```javascript
const xhr = new XMLHttpRequest()

xhr.open("GET","/api/user")

xhr.send()
```

------

#### 2 Fetch（现代浏览器）

```javascript
fetch("/api/user")
  .then(res=>res.json())
```

------

#### 3 Axios（最常用）

```javascript
axios.get("/api/user")
```

------

#### 4 WebSocket（实时通信）

```javascript
const ws = new WebSocket("ws://localhost:8080")
```

常用于：

```text
聊天
实时数据
游戏
```

------

### 六、浏览器存储

浏览器可以存储数据。

分类：

```text
浏览器存储
│
├── Cookie
├── localStorage
├── sessionStorage
└── IndexedDB
```

------

#### 1 localStorage

长期存储。

```javascript
localStorage.setItem("name","Tom")

localStorage.getItem("name")
```

------

#### 2 sessionStorage

会话存储。

```javascript
sessionStorage.setItem("token","123")
```

------

#### 3 Cookie

服务器与浏览器之间传递数据。

```javascript
document.cookie = "user=Tom"
```

------

### 七、浏览器高级 API

现代浏览器提供很多高级能力。

```text
浏览器高级API
│
├── Canvas（绘图）
├── WebGL（3D图形）
├── WebRTC（视频通信）
├── Geolocation（定位）
├── File API（文件）
├── Drag & Drop（拖拽）
└── Notification（通知）
```

------

#### Canvas 示例

```javascript
const canvas = document.getElementById("c")
const ctx = canvas.getContext("2d")

ctx.fillRect(10,10,100,100)
```

------

### 八、JavaScript 浏览器编程完整知识结构

```text
JavaScript浏览器编程
│
├── DOM编程
│   ├── 获取元素
│   ├── 修改内容
│   ├── 修改样式
│   └── 操作节点
│
├── BOM编程
│   ├── window
│   ├── location
│   ├── history
│   └── navigator
│
├── 事件系统
│   ├── 鼠标事件
│   ├── 键盘事件
│   ├── 表单事件
│   └── 页面事件
│
├── 网络通信
│   ├── Ajax
│   ├── Fetch
│   ├── Axios
│   └── WebSocket
│
├── 浏览器存储
│   ├── Cookie
│   ├── localStorage
│   ├── sessionStorage
│   └── IndexedDB
│
└── 浏览器高级API
    ├── Canvas
    ├── WebRTC
    ├── Geolocation
    └── File API
```

------

💡 **如果你在学习 Vue3 前端开发，我可以再给你一张：**

**《Vue3 浏览器编程关系图》**

把这些全部串起来：

```text
HTML
CSS
JavaScript
DOM
BOM
事件
Axios
Vue3
Pinia
Vue Router
```

很多人看完那一张图，**整个前端体系会一下子清晰起来。**

## **JavaScript 异步编程**

是 JS 中非常重要的一部分，因为 **JavaScript 是单线程语言**，为了不阻塞页面执行，需要通过 **异步机制**处理耗时操作（如网络请求、定时任务、文件读取等）。

下面从 **原理 → 分类 → 使用方式 → 关系结构** 系统介绍。

------

### 一、为什么需要异步编程

JavaScript 是 **单线程执行模型**。

如果所有任务都同步执行：

```text
任务1 → 任务2 → 任务3 → 任务4
```

如果任务 2 是网络请求，需要等待 3 秒：

```text
任务1 → 等待3秒 → 任务3
```

页面就会 **卡住**。

因此 JavaScript 引入 **异步机制**：

```text
任务1 → 任务2(后台执行) → 任务3 → 任务4
```

等任务2完成再回调处理。

------

### 二、JavaScript 异步编程整体分类

JS 异步编程可以分为 **4 大类**：

```text
JavaScript异步编程
│
├── 1 回调函数（Callback）
├── 2 Promise
├── 3 async / await
└── 4 事件循环（Event Loop）
```

其中：

```text
Callback → 最早方式
Promise → 解决回调地狱
async/await → Promise语法糖
Event Loop → 异步执行机制
```

------

### 三、回调函数（Callback）

最早的异步方式。

**回调函数：任务完成后调用的函数。**

示例：

```javascript
setTimeout(function(){
    console.log("3秒后执行")
},3000)
```

执行流程：

```text
主线程
   │
setTimeout
   │
定时器线程
   │
时间到
   │
回调函数执行
```

------

#### 回调地狱（Callback Hell）

多层回调会变得很难维护。

```javascript
getUser(function(user){
    getOrder(user,function(order){
        getProduct(order,function(product){
            console.log(product)
        })
    })
})
```

结构：

```text
回调
 └─回调
    └─回调
       └─回调
```

问题：

- 代码难读
- 难维护
- 错误处理复杂

------

### 四、Promise

Promise 是 **ES6 引入的异步解决方案**。

作用：

```text
解决回调地狱
```

------

#### Promise 三种状态

```text
pending   （进行中）
fulfilled （成功）
rejected  （失败）
```

------

#### Promise 基本结构

```javascript
const promise = new Promise((resolve,reject)=>{
    
    if(true){
        resolve("成功")
    }else{
        reject("失败")
    }

})
```

------

#### Promise 使用

```javascript
promise
  .then(res=>{
      console.log(res)
  })
  .catch(err=>{
      console.log(err)
  })
```

------

#### Promise 链式调用

```javascript
getUser()
   .then(user=>getOrder(user))
   .then(order=>getProduct(order))
   .then(product=>console.log(product))
```

优点：

```text
代码更清晰
避免回调地狱
统一错误处理
```

------

### 五、async / await

`async / await` 是 **Promise 的语法糖**。

让异步代码看起来像 **同步代码**。

------

#### async 函数

```javascript
async function test(){
    return "hello"
}
```

本质：

```text
async函数返回Promise
```

------

#### await 关键字

`await` 用于等待 Promise。

示例：

```javascript
async function getData(){

    const user = await getUser()
    const order = await getOrder(user)

    console.log(order)

}
```

代码结构：

```text
同步写法
异步执行
```

优点：

```text
代码更简洁
逻辑更清晰
更易维护
```

------

### 六、事件循环（Event Loop）

JS 异步任务真正的执行机制。

JS 运行模型：

```text
调用栈
任务队列
事件循环
```

结构：

```text
浏览器
│
├── Call Stack（调用栈）
├── Web APIs
└── Task Queue（任务队列）
```

执行流程：

```text
1 JS执行同步代码
2 遇到异步任务
3 交给浏览器API
4 完成后进入任务队列
5 Event Loop把任务放回调用栈
```

------

### 七、宏任务与微任务

事件循环中任务分两类：

```text
宏任务（Macro Task）
微任务（Micro Task）
```

------

#### 宏任务

```text
setTimeout
setInterval
I/O
UI渲染
```

------

#### 微任务

```text
Promise.then
MutationObserver
queueMicrotask
```

执行顺序：

```text
同步代码
↓
微任务
↓
宏任务
```

示例：

```javascript
console.log(1)

setTimeout(()=>{
    console.log(2)
})

Promise.resolve().then(()=>{
    console.log(3)
})

console.log(4)
```

输出：

```text
1
4
3
2
```

------

### 八、JS 异步编程完整结构

```text
JavaScript异步编程
│
├── 回调函数
│   └── Callback
│
├── Promise
│   ├── then
│   ├── catch
│   └── finally
│
├── async / await
│
├── 事件循环
│   ├── Call Stack
│   ├── Task Queue
│   └── Web APIs
│
└── 任务类型
    ├── 宏任务
    │   ├── setTimeout
    │   └── setInterval
    │
    └── 微任务
        └── Promise.then
```

------

### 九、前端开发中常见异步场景

```text
HTTP请求
Axios
Fetch
定时任务
动画
文件读取
数据库操作
WebSocket
```

例如：

```javascript
axios.get("/api/user")
```

本质：

```text
Axios
  ↓
Promise
  ↓
async / await
```

------

✅ **一句话总结**

```text
Callback → Promise → async/await
        ↓
     EventLoop
```

------

如果你准备 **Vue3 / 前端面试**，我还可以给你一张非常重要的图：

**《JavaScript Event Loop 完整执行流程图（面试级）》**

能彻底理解：

```
宏任务
微任务
Promise
setTimeout
DOM渲染
```

很多大厂前端面试 **几乎必问这个模型**。

## **JavaScript 模块化（Module System）** 

是为了解决 **代码组织、复用、依赖管理、避免全局污染** 等问题而提出的一种编程方式。

简单理解：

```text
模块 = 一个独立的JS文件
```

每个模块：

- 有自己的变量作用域
- 可以导出功能
- 可以导入其他模块

------

### 一、为什么需要模块化

早期 JavaScript 没有模块系统，所有变量都是全局的。

例如：

```javascript
var user = {}
var order = {}
var product = {}
```

问题：

```text
1 全局变量污染
2 代码难维护
3 依赖关系混乱
4 文件加载顺序复杂
```

于是出现了 **模块化方案**。

------

### 二、JavaScript 模块化整体分类

JS 模块化发展经历了几个阶段：

```text
JavaScript模块化
│
├── 1 IIFE模块化
├── 2 CommonJS
├── 3 AMD
├── 4 CMD
└── 5 ES Module（ESM）
```

发展顺序：

```text
IIFE → CommonJS → AMD/CMD → ES Module
```

------

### 三、IIFE 模块化（最早）

IIFE = **立即执行函数**

通过函数作用域实现模块隔离。

示例：

```javascript
var userModule = (function(){

    var name = "Tom"

    function getName(){
        return name
    }

    return {
        getName
    }

})()

console.log(userModule.getName())
```

特点：

```text
优点
1 防止变量污染
2 结构简单

缺点
1 无依赖管理
2 不支持模块加载
```

------

### 四、CommonJS（Node.js 模块）

CommonJS 是 **Node.js 的模块规范**。

核心思想：

```text
同步加载模块
```

------

#### 导出模块

```javascript
// user.js

const name = "Tom"

function getName(){
    return name
}

module.exports = {
    getName
}
```

------

#### 引入模块

```javascript
// app.js

const user = require("./user")

console.log(user.getName())
```

特点：

```text
1 同步加载
2 适合服务器环境
3 Node.js默认模块系统
```

缺点：

```text
浏览器不适合
```

------

### 五、AMD（Asynchronous Module Definition）

AMD 是 **浏览器端模块规范**。

核心思想：

```text
异步加载模块
```

代表库：

```text
RequireJS
```

------

示例：

```javascript
define(["user"],function(user){

    function test(){
        console.log(user.getName())
    }

    return {
        test
    }

})
```

特点：

```text
优点
1 异步加载
2 浏览器友好

缺点
1 写法复杂
```

------

### 六、CMD（Common Module Definition）

CMD 是 **SeaJS 提出的模块规范**。

特点：

```text
依赖就近加载
```

示例：

```javascript
define(function(require,exports,module){

    const user = require("user")

    function test(){
        console.log(user.getName())
    }

    exports.test = test

})
```

特点：

```text
1 按需加载
2 写法更接近CommonJS
```

------

### 七、ES Module（ESM）

ES Module 是 **ES6 官方模块化方案**，现在是主流。

核心关键字：

```text
export
import
```

------

#### 导出模块

```javascript
export const name = "Tom"

export function getName(){
    return name
}
```

------

#### 导入模块

```javascript
import {name,getName} from "./user.js"
```

------

#### 默认导出

```javascript
export default function(){
    console.log("hello")
}
```

导入：

```javascript
import test from "./user.js"
```

------

特点：

```text
1 官方标准
2 支持静态分析
3 支持 Tree Shaking
4 浏览器和Node都支持
```

------

### 八、模块化加载对比

| 模块系统  | 运行环境    | 加载方式 |
| --------- | ----------- | -------- |
| IIFE      | 浏览器      | 手动     |
| CommonJS  | Node.js     | 同步     |
| AMD       | 浏览器      | 异步     |
| CMD       | 浏览器      | 按需     |
| ES Module | 浏览器+Node | 静态     |

------

### 九、现代前端模块化

现代前端开发基本使用：

```text
ES Module
```

结合构建工具：

```text
Vite
Webpack
Rollup
```

示例：

```javascript
import axios from "axios"
import { createApp } from "vue"
```

------

### 十、JavaScript 模块化完整结构

```text
JavaScript模块化
│
├── 早期方案
│   └── IIFE
│
├── Node模块
│   └── CommonJS
│
├── 浏览器模块
│   ├── AMD
│   └── CMD
│
└── 现代标准
    └── ES Module
        ├── export
        └── import
```

------

### 十一、模块化与 Vue3 的关系

在 Vue3 项目中：

```text
Vue组件
│
├── import vue
├── import axios
├── import router
└── import store
```

例如：

```javascript
import { createApp } from "vue"
import App from "./App.vue"
import router from "./router"
```

本质：

```text
Vue3 = 基于 ES Module
```

------

✅ **一句话总结**

```text
JS模块化发展：

IIFE
  ↓
CommonJS (Node)
  ↓
AMD / CMD (浏览器)
  ↓
ES Module (标准)
```

------

如果你是在 **准备前端面试或学习 Vue3**，我可以再给你一张非常重要的图：

**《JavaScript 模块化 + 构建工具（Webpack / Vite）完整关系图》**

可以把下面这些一次性讲清：

```text
ES Module
CommonJS
Vite
Webpack
Babel
Tree Shaking
打包原理
```

很多人看完那张图 **前端工程化就彻底理解了**。

## js 网络通信分类介绍

**JavaScript 网络通信**指的是：
**浏览器中的 JavaScript 与服务器之间进行数据交互的技术。**

简单理解：

```text
浏览器(JS)  ←→  服务器(API)
```

JS 通过网络通信技术向服务器 **发送请求、接收数据、更新页面**。

------

### 一、JavaScript 网络通信整体分类

JavaScript 网络通信一般分为 **5 大类技术**：

```text
JavaScript网络通信
│
├── 1 Ajax（传统异步请求）
├── 2 Fetch API（现代原生请求）
├── 3 Axios（第三方HTTP库）
├── 4 WebSocket（实时通信）
└── 5 SSE（Server-Sent Events）
```

从发展历史看：

```text
Ajax → Fetch → Axios
            ↓
        WebSocket
```

------

### 二、Ajax（Asynchronous JavaScript and XML）

Ajax 是 **最早的浏览器异步请求技术**。

核心对象：

```text
XMLHttpRequest
```

#### Ajax 工作流程

```text
浏览器
   │
JavaScript
   │
XMLHttpRequest
   │
HTTP请求
   │
服务器
   │
返回数据
   │
更新页面
```

------

#### Ajax 示例

```javascript
const xhr = new XMLHttpRequest()

xhr.open("GET","/api/user")

xhr.onreadystatechange = function(){
    if(xhr.readyState === 4){
        console.log(xhr.responseText)
    }
}

xhr.send()
```

特点：

优点：

- 支持异步
- 不刷新页面更新数据

缺点：

- 代码复杂
- 可读性差

------

### 三、Fetch API（现代浏览器请求）

Fetch 是浏览器提供的 **现代 HTTP 请求 API**。

特点：

```text
Promise 风格
更简洁
原生支持
```

------

#### Fetch 示例

```javascript
fetch("/api/user")
   .then(res => res.json())
   .then(data => {
       console.log(data)
   })
```

也可以配合 `async / await`：

```javascript
async function getUser(){

    const res = await fetch("/api/user")
    const data = await res.json()

    console.log(data)

}
```

------

### 四、Axios（最常用）

Axios 是一个 **基于 Promise 的 HTTP 请求库**。

在 Vue / React 项目中使用最多。

特点：

```text
Promise封装
自动JSON解析
请求拦截器
响应拦截器
请求取消
```

------

#### Axios 示例

安装：

```bash
npm install axios
```

请求：

```javascript
import axios from "axios"

axios.get("/api/user")
     .then(res=>{
         console.log(res.data)
     })
```

POST 请求：

```javascript
axios.post("/api/user",{
    name:"Tom"
})
```

------

#### Axios 优势

| 功能     | 说明         |
| -------- | ------------ |
| 拦截器   | 统一处理请求 |
| 自动JSON | 自动解析数据 |
| 跨浏览器 | 兼容性好     |
| 简洁API  | 易使用       |

------

### 五、WebSocket（实时通信）

WebSocket 是 **全双工通信协议**。

HTTP 是：

```text
请求 → 响应
```

WebSocket 是：

```text
客户端 ←→ 服务器
实时通信
```

------

#### WebSocket 连接

```javascript
const ws = new WebSocket("ws://localhost:8080")
```

监听消息：

```javascript
ws.onmessage = function(event){
    console.log(event.data)
}
```

发送消息：

```javascript
ws.send("hello")
```

------

#### WebSocket 使用场景

```text
聊天系统
实时行情
在线游戏
实时通知
```

------

### 六、SSE（Server-Sent Events）

SSE 是 **服务器向客户端推送数据**的一种方式。

特点：

```text
单向通信
服务器 → 浏览器
```

------

#### SSE 示例

```javascript
const eventSource = new EventSource("/events")

eventSource.onmessage = function(event){
    console.log(event.data)
}
```

------

### 七、网络通信方式对比

| 技术      | 通信方式   | 特点          |
| --------- | ---------- | ------------- |
| Ajax      | 请求响应   | 传统方式      |
| Fetch     | 请求响应   | 现代浏览器API |
| Axios     | 请求响应   | 最常用库      |
| WebSocket | 双向通信   | 实时通信      |
| SSE       | 服务器推送 | 单向推送      |

------

### 八、JavaScript 网络通信完整结构

```text
JavaScript网络通信
│
├── HTTP请求
│   ├── Ajax
│   ├── Fetch
│   └── Axios
│
├── 实时通信
│   └── WebSocket
│
└── 服务器推送
    └── SSE
```

------

### 九、Vue3 项目中的网络通信

在 Vue3 项目中一般使用：

```text
Vue3
│
├── Axios
│   ├── 请求接口
│   ├── 拦截器
│   └── Token认证
│
└── WebSocket
    └── 实时消息
```

例如：

```javascript
import axios from "axios"

axios.get("/api/question")
```

------

### 十、前端网络通信完整体系

```text
JavaScript网络通信
│
├── 请求技术
│   ├── Ajax
│   ├── Fetch
│   └── Axios
│
├── 实时通信
│   └── WebSocket
│
├── 推送技术
│   └── SSE
│
└── 底层协议
    ├── HTTP
    ├── HTTPS
    └── WS
```

------

✅ **一句话总结**

```text
JS网络通信：

HTTP请求
  ├─ Ajax
  ├─ Fetch
  └─ Axios

实时通信
  └─ WebSocket

服务器推送
  └─ SSE
```

------

如果你正在学习 **Vue3 或前端面试**，我可以给你一张非常清晰的：

**《浏览器 → Axios → API → 后端服务 完整通信架构图》**

能把下面这些全部串起来：

```text
浏览器
Axios
HTTP
Nginx
SpringBoot
数据库
JWT认证
```

## JS 生态分类介绍

**JavaScript 生态（JavaScript Ecosystem）**指的是围绕 **JavaScript 语言形成的一整套技术体系**，包括前端框架、后端运行环境、工具链、构建工具、测试工具等。

简单理解：

```text
JavaScript生态 = JavaScript + 框架 + 工具 + 平台
```

------

### 一、JavaScript 生态整体结构

JS 生态可以分为 **7 大类**：

```text
JavaScript生态
│
├── 1 运行环境
├── 2 前端框架
├── 3 后端框架
├── 4 包管理工具
├── 5 构建工具
├── 6 状态管理
├── 7 测试工具
```

------

### 二、运行环境（Runtime）

运行环境指 **JavaScript 可以运行的平台**。

```text
JavaScript运行环境
│
├── 浏览器
└── Node.js
```

#### 1 浏览器

浏览器提供：

```text
DOM
BOM
Web API
```

例如：

```javascript
document.getElementById("app")
```

------

#### 2 Node.js

服务器端 JavaScript 运行环境。

```text
Node.js
│
├── 文件系统
├── HTTP服务器
├── 网络通信
└── 模块系统
```

示例：

```javascript
const http = require("http")
```

------

### 三、前端框架

用于 **构建用户界面（UI）**。

```text
前端框架
│
├── Vue
├── React
└── Angular
```

#### Vue

特点：

```text
响应式
组件化
易学习
```

示例：

```javascript
createApp(App).mount("#app")
```

------

#### React

特点：

```text
组件化
虚拟DOM
函数式编程
```

------

#### Angular

特点：

```text
完整框架
TypeScript
企业级
```

------

### 四、后端框架

Node.js 生态中的后端框架。

```text
Node后端框架
│
├── Express
├── Koa
└── NestJS
```

------

#### Express

最流行 Node 框架。

```javascript
const express = require("express")

const app = express()
```

------

#### Koa

由 Express 团队开发。

特点：

```text
中间件
async/await
更轻量
```

------

#### NestJS

现代 Node 企业框架。

特点：

```text
TypeScript
依赖注入
模块化
```

------

### 五、包管理工具

用于 **管理 JavaScript 依赖包**。

```text
包管理工具
│
├── npm
├── yarn
└── pnpm
```

------

#### npm

Node 默认包管理器。

```bash
npm install axios
```

------

#### yarn

Facebook 推出的包管理器。

```bash
yarn add vue
```

------

#### pnpm

新一代高效包管理工具。

特点：

```text
节省磁盘
安装速度快
```

------

### 六、构建工具（Build Tools）

用于 **打包、编译、优化前端代码**。

```text
构建工具
│
├── Webpack
├── Vite
├── Rollup
└── Parcel
```

------

#### Webpack

最经典的前端构建工具。

功能：

```text
模块打包
代码分割
资源管理
```

------

#### Vite

现代前端开发工具。

特点：

```text
极速启动
按需编译
ES Module
```

------

#### Rollup

适合构建 **库项目**。

------

### 七、状态管理

用于管理 **应用状态数据**。

```text
状态管理
│
├── Vuex
├── Pinia
└── Redux
```

------

#### Vuex

Vue2 常用状态管理。

------

#### Pinia

Vue3 推荐状态管理。

```javascript
defineStore("user",{
   state:()=>({name:"Tom"})
})
```

------

#### Redux

React 生态常用。

------

### 八、测试工具

用于 **自动化测试 JavaScript 代码**。

```text
测试工具
│
├── Jest
├── Mocha
└── Cypress
```

------

#### Jest

最流行 JS 测试框架。

------

#### Mocha

经典测试框架。

------

#### Cypress

前端 E2E 测试工具。

------

### 九、JavaScript 生态完整结构

```text
JavaScript生态
│
├── 运行环境
│   ├── 浏览器
│   └── Node.js
│
├── 前端框架
│   ├── Vue
│   ├── React
│   └── Angular
│
├── 后端框架
│   ├── Express
│   ├── Koa
│   └── NestJS
│
├── 包管理
│   ├── npm
│   ├── yarn
│   └── pnpm
│
├── 构建工具
│   ├── Webpack
│   ├── Vite
│   └── Rollup
│
├── 状态管理
│   ├── Vuex
│   ├── Pinia
│   └── Redux
│
└── 测试工具
    ├── Jest
    ├── Mocha
    └── Cypress
```

------

### 十、现代前端开发技术栈

如果使用 **Vue3**，常见技术栈是：

```text
Vue3技术栈
│
├── Vue3
├── Vue Router
├── Pinia
├── Axios
├── Vite
└── npm
```

------

✅ **一句话总结**

```text
JavaScript生态 =

运行环境
+ 前端框架
+ Node后端
+ 包管理
+ 构建工具
+ 状态管理
+ 测试工具
```

------

如果你正在 **系统学习前端（尤其 Vue3）**，我可以再给你一张非常清晰的：

**《JavaScript 前端完整技术生态图（从浏览器到工程化）》**

里面会把这些一次性串起来：

```text
HTML
CSS
JavaScript
ES6
DOM
Vue3
Axios
Router
Pinia
Vite
Node.js
Nginx
```

很多人看完这张图 **前端体系会彻底打通**。