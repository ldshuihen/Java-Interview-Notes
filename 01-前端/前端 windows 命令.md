# 前端 window 对象完整命令（API）分类详解
`window` 是浏览器全局顶层对象，所有全局变量、全局函数都挂载在它上面；JS 在浏览器环境运行的入口载体，包含**窗口控制、BOM 导航、DOM 基础、存储、定时器、网络、设备、弹窗、全局属性**等上千个 API，下面按功能大类划分，附常用方法/属性、作用、使用场景。

## 一、弹窗交互类（最基础交互命令）
页面原生提示弹窗，同步阻塞代码执行
1. `window.alert(msg)`：警告弹窗，仅展示文本，确定关闭
2. `window.confirm(msg)`：确认弹窗，返回布尔值（确定=true / 取消=false）
3. `window.prompt(text, 默认值)`：输入弹窗，接收用户输入文本，取消返回 `null`

示例：
```js
const res = confirm('确定退出页面？')
```

## 二、定时器 & 异步延时调度（任务调度）
浏览器宏任务队列核心 API，控制代码延迟/循环执行
### 1. 延时单次执行
- `setTimeout(回调函数, 毫秒数, 参数)`：延迟指定时间执行一次
- `clearTimeout(定时器ID)`：取消延时任务

### 2. 循环重复执行
- `setInterval(回调, 间隔毫秒)`：每隔固定时间循环执行
- `clearInterval(定时器ID)`：关闭循环定时器

### 3. 帧动画专用（渲染时序）
- `requestAnimationFrame(cb)`：跟随浏览器屏幕刷新帧率执行（16ms/帧），做动画最优
- `cancelAnimationFrame(id)`：取消帧动画

## 三、浏览器窗口尺寸、位置控制（窗口可视区域）

获取/修改浏览器窗口大小、屏幕坐标

**读取属性**

- `window.innerWidth / innerHeight`：可视视口宽高（不含浏览器工具栏、滚动条）⭐最常用
- `window.outerWidth / outerHeight`：整个浏览器窗口（包含标题栏、边框）总尺寸
- `window.screenX / screenY`：浏览器窗口距离电脑屏幕左上角坐标

**窗口操作方法**

- `window.open(url, 窗口名, 窗口配置)`：新开弹窗/新标签页
- `window.close()`：关闭当前窗口（仅脚本打开的窗口可关闭）
- `window.moveTo(x,y)`：移动浏览器窗口位置
- `window.resizeTo(w,h)`：修改窗口大小

现代浏览器中这部分 API 分为两类：
1. **只读属性**：获取窗口、视口、屏幕大小、窗口坐标（日常开发最常用）
2. **操作方法**：打开新窗口、移动、缩放、关闭窗口（受浏览器安全策略严格限制）

### 一、窗口尺寸四大属性对比

#### 1. window.innerWidth / window.innerHeight

✅ **最常用：浏览器可视区域（视口 Viewport）宽高**
含义：
页面能展示内容的可视区域大小
包含：页面滚动条
不包含：浏览器标题栏、地址栏、书签栏、底部状态栏、窗口边框

特点：
- 窗口缩放时实时变化
- 移动端适配、响应式布局判断核心依据
- 永远不带浏览器外壳区域

示例：
```js
console.log('可视宽度：', window.innerWidth);
console.log('可视高度：', window.innerHeight);
```

#### 2. window.outerWidth / window.outerHeight

整个**完整浏览器窗口**总宽高
包含：标题栏、地址栏、边框、状态栏、整个浏览器外壳 + 内部视口区域
> 通俗理解：拖动浏览器窗口边角整体拉大缩小，这个值才会改变

使用场景：极少用，基本前端业务开发不会接触。

#### 两者直观区别举例

电脑上浏览器默认打开：
- innerWidth：网页内容区域宽度
- outerWidth = innerWidth + 左右窗口边框宽度

#### 补充两组滚动偏移尺寸（易混淆）

不属于窗口大小，但经常搭配使用：
- `document.documentElement.clientWidth`：视口宽（和 innerWidth 几乎一致，无滚动条时完全相等）
- `window.scrollX / window.scrollY`：页面横向、纵向滚动出去的距离

### 二、窗口屏幕坐标属性：screenX、screenY

window.screenX ｜ window.screenLeft（别名，兼容IE）

浏览器窗口**左上角距离电脑物理屏幕左上角的水平像素距离**

window.screenY ｜ window.screenTop（别名）

浏览器窗口**左上角距离电脑屏幕左上角的垂直像素距离**

适用场景：
1. 桌面端打开弹窗，想要把新窗口居中摆放
2. 多显示器分屏场景判断窗口在哪块屏幕

举例：
把浏览器往屏幕右边拖动一段距离，`screenX` 数值会变大。

> 移动端无意义，移动端浏览器铺满屏幕，坐标固定为 0。

### 三、窗口操作方法（open / close / moveTo / resizeTo）

⚠️ 重要前置规则：
现代浏览器为了防止恶意广告弹窗骚扰用户，**大幅度限制了窗口移动、缩放功能**
1. 只有 JS 通过 `window.open()` 新建出来的弹出窗口，才允许移动、调整大小
2. 用户手动打开的主页面，脚本无法移动、缩放浏览器窗口（调用无效）
3. 弹窗会被浏览器弹窗拦截器拦截，必须是**用户点击事件内触发 open()** 才不会被拦截

#### 1. window.open() 打开新窗口/弹窗

语法：
```js
window.open(URL, name, features, replace)
```
参数逐项详解：
1. **URL**：要打开的网页地址，为空则打开空白页面
2. **name**：窗口名称
   - `_blank`：默认，新标签页打开
   - `_self`：当前页面跳转
   - 自定义字符串：创建独立弹窗窗口，可后续用名字操控
3. **features**：弹窗配置字符串，控制宽高、菜单栏、地址栏等，用逗号分隔
4. replace：布尔，是否替换当前历史记录（很少用）

##### 常用配置示例

```js
// 打开固定尺寸弹窗，无菜单栏、地址栏
const newWin = window.open(
  'https://www.baidu.com',
  'popWindow',
  'width=600,height=450,top=100,left=200,menubar=no,toolbar=no,status=no,scrollbars=yes'
);
```
features 常用配置项：
| 配置           | 作用                   |
| -------------- | ---------------------- |
| width/height   | 弹窗宽高               |
| left/top       | 弹窗距离屏幕左上角坐标 |
| menubar=no     | 隐藏菜单栏             |
| toolbar=no     | 隐藏前进后退工具栏     |
| location=no    | 隐藏地址栏             |
| status=no      | 隐藏底部状态栏         |
| scrollbars=yes | 内容超出时显示滚动条   |

返回值：
返回新窗口的 window 对象 `newWin`，可以操控这个弹窗。

#### 2. window.close() 关闭窗口

规则：
1. **只能关闭 JS 自己用 open() 打开的窗口**
   ```js
   // 关闭刚刚创建的弹窗
   newWin.close()
   ```
2. 用户手动打开的主页，执行 `window.close()` 无效，浏览器会拒绝关闭
3. 部分浏览器关闭前会弹出确认提示

#### 3. window.moveTo(x, y)：移动窗口位置

作用：将窗口左上角整体移动到屏幕坐标 (x, y) 处
```js
// 把弹窗移动到屏幕左上角
newWin.moveTo(0, 0);
```
限制：
- 仅脚本创建的弹窗可用
- 不能把窗口移出屏幕可视区域（浏览器限制）

配套：`moveBy(dx, dy)`
相对当前位置偏移移动：横向+dx，纵向+dy
```js
newWin.moveBy(50, 30); // 向右50px，向下30px
```

#### 4. window.resizeTo(width, height)：修改窗口整体尺寸

设置浏览器窗口完整外框大小（对应 outerWidth/outerHeight）
```js
newWin.resizeTo(800, 600);
```

配套：`resizeBy(w, h)`
在现有尺寸基础上增减宽高
```js
newWin.resizeBy(100, 50); // 宽+100，高+50
```

### 四、经典实战案例：打开居中弹窗

需求：点击按钮，弹出窗口，自动在屏幕正中间显示
```html
<button id="btn">打开居中弹窗</button>
<script>
  const btn = document.getElementById('btn');
  btn.onclick = function () {
    // 弹窗尺寸
    const w = 600;
    const h = 450;
    // 计算居中坐标
    const left = window.screenX + (window.innerWidth - w) / 2;
    const top = window.screenY + (window.innerHeight - h) / 2;

    const pop = window.open(
      '',
      'centerWin',
      `width=${w},height=${h},left=${left},top=${top}`
    );
    pop.document.write('<h3>居中弹窗内容</h3>');
  }
</script>
```

### 五、各浏览器兼容总结

1. **移动端**
innerWidth 视口尺寸常用；moveTo、resizeTo、open 弹窗缩放全部失效，移动端不支持窗口分离；
2. **PC端主流浏览器（Chrome/Edge/Firefox）**
- 只读尺寸属性全部兼容
- 移动、缩放窗口仅允许脚本新建弹窗使用
3. 老旧IE：有 screenLeft / screenTop 别名兼容差异，现在基本无需考虑

### 六、开发使用场景汇总

1. innerWidth：响应式布局、判断设备大屏/平板/手机
2. screenX/screenY：弹窗居中布局计算
3. window.open：后台系统弹出详情窗口、打印预览弹窗
4. moveTo/resizeTo：桌面端后台管理系统老旧弹窗场景，新项目基本很少使用





### window.open('_blank') 场景：接口请求拿到跳转地址后新标签页打开

#### 核心痛点
不能直接提前写死链接，需要先调用后端接口，拿到返回的URL地址，再新开空白标签页跳转；
这里有一个**浏览器弹窗拦截关键问题**：
浏览器规定：**非用户主动点击触发的异步回调里调用 window.open()，会被浏览器弹窗拦截**

> 举例：点击按钮 → 发ajax请求(异步) → 请求成功回调里执行 open，大概率被拦截。

#### 两种标准解决方案

##### 方案1：预先打开空白窗口，请求完成后再赋值地址（最稳妥、推荐）

流程：
1. 用户点击瞬间，立刻用 `window.open('_blank')` 新建空白标签（此时是用户同步行为，不会被拦截）
2. 保存新窗口实例
3. 发起后端接口请求
4. 接口拿到真实跳转链接后，给空白窗口赋值 `newWin.location.href = 目标url`
5. 兜底：接口失败时关闭空白窗口

完整示例代码（fetch/axios两种写法都附上）
```html
<button id="jumpBtn">获取链接并新标签页打开</button>

<script>
const jumpBtn = document.getElementById('jumpBtn');
jumpBtn.addEventListener('click', async () => {
  // 1. 同步先行打开空白新标签，规避拦截
  const newTab = window.open('', '_blank');

  try {
    // 2. 调用后端API获取跳转地址
    const res = await fetch('/api/getTargetUrl');
    const data = await res.json();
    // 后端返回的目标链接
    const targetUrl = data.url;

    // 3. 给新开的空白标签赋值地址，完成跳转
    newTab.location.href = targetUrl;
  } catch (err) {
    // 接口报错：关闭空白页面，避免空白标签残留
    newTab.close();
    alert('链接获取失败，请重试');
    console.error(err);
  }
});
</script>
```

###### Axios 版本写法

```js
jumpBtn.addEventListener('click', async () => {
  const newTab = window.open('', '_blank');
  axios.get('/api/getTargetUrl')
    .then(res => {
      newTab.location.href = res.data.url;
    })
    .catch(() => {
      newTab.close();
      alert('获取地址失败');
    })
})
```

---

##### 方案2：异步拿到地址后，由用户二次确认跳转（备选）

如果不想先开空白页，异步拿到URL后，弹出确认框，用户点击确认再新开标签，依靠第二次用户交互绕过拦截：
```js
jumpBtn.onclick = async () => {
  // 异步请求地址
  const res = await fetch('/api/getTargetUrl');
  const url = (await res.json()).url;

  // 用户手动确认，属于用户行为，不会拦截
  if(confirm('即将跳转新页面，确认打开？')){
    window.open(url, '_blank');
  }
}
```

###### 1. window.open 参数说明

```js
// 第二个参数固定 _blank = 始终在全新独立标签页打开
window.open(跳转地址, '_blank')
```
- `_blank`：每次调用都生成新标签
- 填自定义名称字符串：多次调用会复用同一个标签页

###### 2. 边界场景处理

1. **接口返回外链（http/https第三方地址）**
完全兼容，直接赋值即可；
2. **需要携带token、登录鉴权**
- 同源地址：cookie自动携带，无需处理
- 跨域第三方页面：后端生成带临时签名/令牌的一次性跳转链接返回前端
3. **新窗口加载等待效果优化**
可以先给空白窗口写入加载提示文字：
```js
newTab.document.body.innerHTML = '<div>链接加载中，请稍候...</div>';
```

###### 3. 为什么直接在接口回调里 open 会被拦截？

浏览器安全策略：阻止网站偷偷批量弹出广告窗口，
只有**鼠标点击、键盘回车这类用户主动同步交互**内的 `window.open` 才放行；
异步网络请求回调属于宏任务，脱离了用户即时交互上下文，被判定为恶意弹窗。

---

#### 简写极简封装函数

```js
// 封装：请求接口获取url，新标签页打开
async function openNewTabByApi(apiUrl) {
  const win = window.open('', '_blank');
  try {
    const { url } = await (await fetch(apiUrl)).json();
    win.location = url;
  } catch {
    win.close();
  }
}
```

## 四、滚动相关 API（页面滚动控制）

控制页面滚动位置、监听滚动行为
### 属性
- `window.scrollX / pageXOffset`：页面水平滚动距离（别名，完全等价）
- `window.scrollY / pageYOffset`：页面垂直滚动距离

### 方法
1. `window.scroll(x, y)`：滚动到指定坐标
2. `window.scrollTo()`：同上，标准写法
3. `window.scrollBy(dx, dy)`：相对当前位置滚动一段距离
4. 平滑滚动配置：
```js
window.scrollTo({ top: 500, behavior: 'smooth' }) // 平滑滚动
```



## 五、浏览器历史记录（History 路由）

控制浏览器前进、后退、路由跳转，SPA 前端路由底层依赖
挂载于 `window.history`
1. `history.back()`：后退一页 ←
2. `history.forward()`：前进一页 →
3. `history.go(n)`：n正数前进、负数后退，0刷新页面
4. `history.pushState(数据,标题,路径)`：新增历史记录，不刷新页面（Vue/React路由核心）
5. `history.replaceState()`：替换当前历史记录，不新增栈

监听路由变化：`popstate` 事件（点击浏览器前进后退触发）

## 六、地址栏 URL 信息（Location 地址对象）
`window.location`：读写浏览器地址栏完整链接，跳转页面核心
### 常用属性（拆解URL各部分）
```
https://www.xxx.com/home?id=1#anchor
```
- `href`：完整网址，赋值即可页面跳转 `location.href = '/login'`
- `protocol`：协议 https:/http:/file:
- `hostname`：域名
- `pathname`：路径 /home
- `search`：查询参数 ?id=1
- `hash`：哈希值 #anchor（锚点）

### 方法
- `location.assign(url)`：跳转页面（新增历史记录，可回退）
- `location.replace(url)`：跳转并替换当前历史，无法回退
- `location.reload()`：刷新当前页面，参数true=强制清缓存刷新

## 七、本地存储（客户端持久化数据）
浏览器本地存储，数据永久/会话级保存在客户端，都挂载window下
1. **sessionStorage**：`window.sessionStorage`
   - 会话存储：关闭标签页数据自动清空，同源标签不共享
   - 方法：`setItem/getItem/removeItem/clear/key/length`

2. **localStorage**：`window.localStorage`
   - 永久本地存储：手动删除/清浏览器缓存才消失，同源所有页面共享
   - 容量约5MB，仅存字符串

3. 补充：Cookie 属于 document，不属于window顶层

## 八、屏幕信息 Screen（设备硬件屏幕）
`window.screen`：获取电脑/手机屏幕硬件参数
- `screen.width / height`：设备屏幕物理总宽高
- `screen.availWidth / availHeight`：可用屏幕尺寸（扣除任务栏）
- `screen.colorDepth`：屏幕色深

## 九、导航器信息 Navigator（浏览器/设备环境检测）
`window.navigator`：读取浏览器、操作系统、设备、网络环境信息，常用于适配、UA判断
### 常用属性
- `navigator.userAgent`：UA字符串，判断PC/手机、浏览器型号
- `navigator.platform`：操作系统平台（Windows/Mac/Linux/Android/iOS）
- `navigator.language`：系统默认语言
- `navigator.onLine`：布尔值，检测设备是否联网

### 实用能力API
- `navigator.clipboard`：剪贴板读写（复制粘贴）
- `navigator.geolocation`：获取用户地理位置（需授权）
- `navigator.mediaDevices`：调用摄像头、麦克风（音视频采集）

## 十、事件监听（全局窗口事件）
window 全局绑定页面生命周期、系统行为事件
### 常用事件
1. 页面加载
- `load`：页面所有资源（图片、样式）全部加载完成触发
- `DOMContentLoaded`：DOM树构建完毕就触发（比load快，常用）

2. 窗口状态
- `resize`：浏览器窗口大小改变时触发（适配布局必备）
- `scroll`：页面滚动持续触发
- `blur`：窗口失去焦点（切后台）
- `focus`：窗口重新获取焦点

3. 页面关闭/离开
- `beforeunload`：关闭/刷新页面前触发，可弹出确认挽留用户
- `unload`：页面即将卸载销毁

## 十一、全局错误捕获 & 调试
1. `window.onerror`：全局JS运行错误捕获，收集前端报错日志
2. `window.onunhandledrejection`：捕获Promise未处理的异步错误
3. `window.console`：控制台打印日志（log/warn/error/table等）

## 十二、Fetch/网络请求（现代异步请求）
全局网络请求API，替代XMLHttpRequest
- `window.fetch(url)`：发起GET/POST网络请求，基于Promise
配套：`Request`、`Response`、`Headers` 均为window全局构造函数

## 十三、Web Worker 多线程
JS默认单线程，Worker开辟后台子线程，不阻塞页面渲染
- `window.Worker()`：创建后台工作线程，计算密集型任务使用

## 十四、全局顶层属性（内置全局变量）
挂载window上的原生全局常量/对象
- `window.document`：DOM文档对象（页面HTML根节点）
- `window.Math`：数学计算工具（sin、pow、random随机数等）
- `window.Date`：日期时间构造函数
- `window.JSON`：JSON序列化/解析（JSON.stringify / parse）
- `window.Array / Object / String`：所有基础构造函数
- `window.isNaN() / parseInt() / parseFloat()`：全局类型转换方法

## 十五、废弃/不推荐旧API
- `window.showModalDialog()`：模态弹窗，已废弃
- `window.execScript()`：IE专属脚本执行，淘汰
- 部分move/resize弹窗控制：现代浏览器出于安全限制，大多已禁用

## 整体层级总结
```
window 顶层全局对象
├─ 弹窗：alert/confirm/prompt
├─ 定时器：setTimeout/setInterval/raf
├─ 窗口尺寸滚动：innerWidth、scroll、scrollTo
├─ 路由体系：history + location
├─ 本地存储：localStorage、sessionStorage
├─ 设备信息：navigator、screen
├─ 全局事件：resize、scroll、load、error
├─ 网络：fetch、XMLHttpRequest
├─ 内置全局工具：Math、Date、JSON、console
└─ 多线程：Web Worker
```
