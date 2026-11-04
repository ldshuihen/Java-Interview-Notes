# `window` 对象命令

## 一、窗口操作

### 1. **打开新窗口**

```javascript
// 打开新标签页
window.open('https://example.com', '_blank');

// 打开新窗口（指定尺寸）
window.open('https://example.com', 'myWindow', 
  'width=800,height=600,resizable=yes,scrollbars=yes');

// 打开空白页
const newWindow = window.open();
newWindow.document.write('<h1>Hello World</h1>');
```

### 2. **关闭窗口**

```javascript
// 关闭当前窗口（只能关闭由脚本打开的窗口）
window.close();

// 关闭指定窗口
const newWindow = window.open('https://example.com');
newWindow.close();
```

### 3. **窗口焦点**

```javascript
// 将焦点移到当前窗口
window.focus();

// 将焦点移出当前窗口
window.blur();
```

### 4. **窗口调整**

```javascript
// 调整窗口大小
window.resizeTo(1024, 768);

// 相对调整窗口大小
window.resizeBy(100, 100);

// 移动窗口位置
window.moveTo(100, 100);

// 相对移动窗口
window.moveBy(50, 50);
```

⚠️ **注意：** 现代浏览器出于安全考虑，限制了这些方法的使用。

---

## 二、位置和导航

### 1. **`window.location`** - 当前位置信息

```javascript
// 获取完整 URL
console.log(window.location.href);
// 输出: "https://example.com:8080/path?query=value#hash"

// 获取协议
console.log(window.location.protocol);  // "https:"

// 获取主机名
console.log(window.location.hostname);  // "example.com"

// 获取端口
console.log(window.location.port);      // "8080"

// 获取路径
console.log(window.location.pathname);  // "/path"

// 获取查询参数
console.log(window.location.search);    // "?query=value"

// 获取锚点
console.log(window.location.hash);      // "#hash"

// 获取 origin（协议 + 主机 + 端口）
console.log(window.location.origin);    // "https://example.com:8080"
```

### 2. **页面跳转**

```javascript
// 跳转到新页面（保留历史记录）
window.location.href = 'https://example.com';
window.location = 'https://example.com';

// 跳转到新页面（不保留历史记录）
window.location.replace('https://example.com');

// 刷新当前页面
window.location.reload();           // 可能从缓存加载
window.location.reload(true);       // 强制从服务器重新加载

// 跳转到锚点
window.location.hash = '#section1';
```

### 3. **`window.history`** - 历史记录

```javascript
// 后退一页
window.history.back();

// 前进一页
window.history.forward();

// 跳转指定步数（负数后退，正数前进）
window.history.go(-1);  // 后退一页
window.history.go(1);   // 前进一页
window.history.go(-2);  // 后退两页

// 添加历史记录（不刷新页面）
window.history.pushState({id: 1}, 'title', '/page1');

// 替换当前历史记录
window.history.replaceState({id: 2}, 'title', '/page2');

// 监听历史记录变化
window.addEventListener('popstate', (event) => {
  console.log('历史记录变化:', event.state);
});
```

---

## 三、对话框

### 1. **`window.alert()`** - 提示框

```javascript
// 显示警告框
window.alert('这是一个提示！');
alert('简写也可以');

// 阻塞代码执行，直到用户点击确定
console.log('前');
alert('提示信息');
console.log('后'); // 用户点击确定后才执行
```

### 2. **`window.confirm()`** - 确认框

```javascript
// 显示确认框
const result = window.confirm('确定要删除吗？');

if (result) {
  console.log('用户点击了确定');
  // 执行删除操作
} else {
  console.log('用户点击了取消');
  // 取消操作
}
```

### 3. **`window.prompt()`** - 输入框

```javascript
// 显示输入框
const name = window.prompt('请输入您的姓名：', '默认值');

if (name !== null) {
  console.log('用户输入：', name);
} else {
  console.log('用户点击了取消');
}
```

---

## 四、定时器

### 1. **`window.setTimeout()`** - 延迟执行

```javascript
// 延迟 1 秒执行
const timerId = window.setTimeout(() => {
  console.log('1 秒后执行');
}, 1000);

// 传递参数
setTimeout((name, age) => {
  console.log(`${name} ${age}岁`);
}, 1000, '张三', 25);

// 清除定时器
clearTimeout(timerId);
```

### 2. **`window.setInterval()`** - 定时重复执行

```javascript
// 每 2 秒执行一次
const intervalId = window.setInterval(() => {
  console.log('每 2 秒执行一次');
}, 2000);

// 停止定时器
clearInterval(intervalId);

// 实际示例：倒计时
let count = 10;
const timer = setInterval(() => {
  count--;
  console.log(`倒计时: ${count}`);
  if (count <= 0) {
    clearInterval(timer);
    console.log('倒计时结束！');
  }
}, 1000);
```

---

## 五、存储

### 1. **`window.localStorage`** - 持久化存储

```javascript
// 存储数据
localStorage.setItem('username', '张三');
localStorage.setItem('age', '25');
localStorage.setItem('user', JSON.stringify({ name: '张三', age: 25 }));

// 读取数据
const username = localStorage.getItem('username');
const user = JSON.parse(localStorage.getItem('user'));

// 删除指定数据
localStorage.removeItem('username');

// 清空所有数据
localStorage.clear();

// 获取存储项数量
console.log(localStorage.length);

// 遍历所有数据
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  const value = localStorage.getItem(key);
  console.log(`${key}: ${value}`);
}
```

### 2. **`window.sessionStorage`** - 会话存储

```javascript
// 使用方式与 localStorage 相同
sessionStorage.setItem('token', 'abc123');
const token = sessionStorage.getItem('token');
sessionStorage.removeItem('token');
sessionStorage.clear();

// 区别：关闭浏览器后数据清空
```

### 3. **监听存储变化**

```javascript
// 监听其他标签页的存储变化
window.addEventListener('storage', (event) => {
  console.log('存储变化:', event.key, event.newValue);
});
```

---

## 六、事件监听

### 1. **`window.addEventListener()`** - 添加事件监听

```javascript
// 监听窗口大小变化
window.addEventListener('resize', () => {
  console.log('窗口大小：', window.innerWidth, window.innerHeight);
});

// 监听页面滚动
window.addEventListener('scroll', () => {
  console.log('滚动位置：', window.scrollY);
});

// 监听页面加载完成
window.addEventListener('load', () => {
  console.log('页面加载完成');
});

// 监听页面即将关闭
window.addEventListener('beforeunload', (event) => {
  event.preventDefault();
  event.returnValue = '';
  // 会弹出确认框
});

// 监听键盘事件
window.addEventListener('keydown', (event) => {
  console.log('按下按键:', event.key);
  if (event.key === 'Escape') {
    console.log('按下了 ESC 键');
  }
});

// 移除事件监听
const handleResize = () => console.log('resize');
window.addEventListener('resize', handleResize);
window.removeEventListener('resize', handleResize);
```

### 2. **常用事件类型**

```javascript
// 鼠标事件
window.addEventListener('click', () => {});
window.addEventListener('dblclick', () => {});
window.addEventListener('mousedown', () => {});
window.addEventListener('mouseup', () => {});
window.addEventListener('mousemove', () => {});

// 键盘事件
window.addEventListener('keydown', () => {});
window.addEventListener('keyup', () => {});
window.addEventListener('keypress', () => {});

// 窗口事件
window.addEventListener('resize', () => {});
window.addEventListener('scroll', () => {});
window.addEventListener('load', () => {});
window.addEventListener('unload', () => {});
window.addEventListener('focus', () => {});
window.addEventListener('blur', () => {});

// 剪贴板事件
window.addEventListener('copy', () => {});
window.addEventListener('paste', () => {});
window.addEventListener('cut', () => {});
```

---

## 七、屏幕信息

### 1. **`window.screen`** - 屏幕信息

```javascript
// 屏幕宽度
console.log(screen.width);          // 1920

// 屏幕高度
console.log(screen.height);         // 1080

// 可用宽度（减去任务栏）
console.log(screen.availWidth);     // 1920

// 可用高度
console.log(screen.availHeight);    // 1040

// 颜色深度
console.log(screen.colorDepth);     // 24

// 像素深度
console.log(screen.pixelDepth);     // 24
```

### 2. **`window.innerWidth/Height`** - 视口尺寸

```javascript
// 浏览器视口宽度（不包括滚动条、工具栏）
console.log(window.innerWidth);

// 浏览器视口高度
console.log(window.innerHeight);

// 整个浏览器窗口宽度（包括工具栏、滚动条）
console.log(window.outerWidth);

// 整个浏览器窗口高度
console.log(window.outerHeight);
```

---

## 八、滚动控制

### 1. **滚动到指定位置**

```javascript
// 滚动到指定坐标
window.scrollTo(0, 500);  // x: 0, y: 500

// 平滑滚动
window.scrollTo({
  top: 500,
  left: 0,
  behavior: 'smooth'
});

// 滚动到顶部
window.scrollTo(0, 0);

// 相对滚动
window.scrollBy(0, 100);  // 向下滚动 100px
window.scrollBy(0, -100); // 向上滚动 100px
```

### 2. **获取滚动位置**

```javascript
// 垂直滚动距离
console.log(window.scrollY);  // 或 window.pageYOffset

// 水平滚动距离
console.log(window.scrollX);  // 或 window.pageXOffset
```

---

## 九、剪贴板操作

### 1. **现代 API（推荐）**

```javascript
// 复制文本
navigator.clipboard.writeText('要复制的文本')
  .then(() => console.log('复制成功'))
  .catch(err => console.error('复制失败:', err));

// 粘贴文本
navigator.clipboard.readText()
  .then(text => console.log('粘贴内容:', text))
  .catch(err => console.error('粘贴失败:', err));
```

### 2. **传统 API**

```javascript
// 复制
document.execCommand('copy');

// 剪切
document.execCommand('cut');

// 粘贴
document.execCommand('paste');
```

⚠️ **注意：** `execCommand` 已被废弃，推荐使用 Clipboard API。

---

## 十、打印

### 1. **`window.print()`** - 打印页面

```javascript
// 打开打印对话框
window.print();

// 打印前准备
window.addEventListener('beforeprint', () => {
  console.log('准备打印');
});

// 打印后清理
window.addEventListener('afterprint', () => {
  console.log('打印完成');
});
```

---

## 十一、性能与请求动画帧

### 1. **`window.requestAnimationFrame()`** - 动画优化

```javascript
// 创建流畅动画
function animate() {
  // 更新动画状态
  element.style.transform = `translateX(${x}px)`;
  
  // 请求下一帧
  requestAnimationFrame(animate);
}

// 开始动画
const animationId = requestAnimationFrame(animate);

// 取消动画
cancelAnimationFrame(animationId);
```

### 2. **性能监控**

```javascript
// 页面加载性能
window.addEventListener('load', () => {
  const timing = performance.timing;
  console.log('页面加载时间:', timing.loadEventEnd - timing.navigationStart);
});

// 测量代码执行时间
performance.mark('start');
// 执行某些操作
performance.mark('end');
performance.measure('myOperation', 'start', 'end');

const measure = performance.getEntriesByName('myOperation')[0];
console.log('执行时间:', measure.duration, 'ms');
```

---

## 十二、你的项目中的实际用法

### 场景 1：监听窗口大小变化

```javascript
// pmsLineBaseDataQuery.vue
mounted() {
  this.mediaScreen();
  window.addEventListener("resize", this.mediaScreen);
},
beforeUnmount() {
  window.removeEventListener("resize", this.mediaScreen);
}
```

### 场景 2：新标签页打开

```javascript
// erpApplicationQuery.vue
window.open(resolved.href, "_blank");
```

### 场景 3：关闭当前窗口

```javascript
// main.js
app.config.globalProperties.closeTipWindow = () => {
  window.opener = null;
  window.open('', '_self');
  window.close();
}
```

---

## 十三、完整对比表

| 类别         | 常用方法                               | 用途          |
| ------------ | -------------------------------------- | ------------- |
| **窗口操作** | `window.open()`, `window.close()`      | 打开/关闭窗口 |
| **位置导航** | `window.location.href`                 | 获取/设置 URL |
| **历史记录** | `window.history.back()`, `pushState()` | 控制历史导航  |
| **对话框**   | `alert()`, `confirm()`, `prompt()`     | 弹出提示框    |
| **定时器**   | `setTimeout()`, `setInterval()`        | 延迟/定时执行 |
| **存储**     | `localStorage`, `sessionStorage`       | 本地数据存储  |
| **事件监听** | `addEventListener()`                   | 监听各类事件  |
| **滚动控制** | `scrollTo()`, `scrollBy()`             | 控制滚动位置  |
| **屏幕信息** | `screen.width`, `innerWidth`           | 获取屏幕尺寸  |
| **打印**     | `window.print()`                       | 打印页面      |

---

## 十四、最佳实践

### ✅ 推荐做法：

```javascript
// 1. 使用现代 API
navigator.clipboard.writeText('text');  // ✅ 现代剪贴板 API
window.scrollTo({ top: 0, behavior: 'smooth' });  // ✅ 平滑滚动

// 2. 及时清理事件监听
mounted() {
  window.addEventListener('resize', this.handleResize);
},
beforeUnmount() {
  window.removeEventListener('resize', this.handleResize);
}

// 3. 使用 localStorage 存储用户偏好
localStorage.setItem('theme', 'dark');
const theme = localStorage.getItem('theme');

// 4. 防抖窗口 resize 事件
let resizeTimer;
window.addEventListener('resize', () => {
  clearTimeout(resizeTimer);
  resizeTimer = setTimeout(() => {
    console.log('窗口大小变化');
  }, 250);
});
```

### ❌ 避免做法：

```javascript
// ❌ 使用已废弃的 API
document.execCommand('copy');

// ❌ 忘记清理事件监听（会导致内存泄漏）
window.addEventListener('resize', () => {});

// ❌ 在循环中使用 setInterval（应使用 requestAnimationFrame）
setInterval(() => {
  // 动画逻辑
}, 16);

// ❌ 滥用 alert（会阻塞线程）
alert('错误信息');  // 应使用自定义弹窗
```

---

## 总结

`window` 对象是浏览器提供的全局对象，包含了大量实用的 API。掌握这些方法可以：

- ✅ 控制窗口和导航
- ✅ 管理本地存储
- ✅ 处理用户交互
- ✅ 优化性能和动画
- ✅ 实现复杂的前端功能

**最常用的：`window.open()`, `window.location`, `localStorage`, `addEventListener()`** 🎯

# Axios 

要对 Axios 进行“完整分类”，我们需要从多个维度来剖析它。Axios 是一个基于 Promise 的 HTTP 网络请求库，其功能丰富，可以从**请求方法、API 使用级别、配置项、拦截器、响应结构、错误处理、高级特性**等几个核心维度进行完整分类。

以下是 Axios 的完整分类指南：

---

### 一、 按请求方法分类

Axios 支持所有的 HTTP 标准请求方法，主要分为两大类：

#### 1. 基础写法
*   **GET**：获取数据（`axios.get(url, config)`）
*   **POST**：提交数据（`axios.post(url, data, config)`）
*   **PUT**：完整更新资源（`axios.put(url, data, config)`）
*   **PATCH**：局部更新资源（`axios.patch(url, data, config)`）
*   **DELETE**：删除资源（`axios.delete(url, config)`）
*   **HEAD**：获取报文首部（`axios.head(url, config)`）
*   **OPTIONS**：询问支持的方法（`axios.options(url, config)`）

#### 2. 别名写法
为了方便，Axios 为上述方法提供了别名，如 `axios.request(config)`，可以直接通过配置对象中的 `method` 字段指定方法。

---

### 二、 按 API 使用级别分类
根据使用场景和封装程度，Axios 的调用方式可分为三类：

#### 1. 全局默认配置
适用于整个应用的基础配置，直接挂载在 `axios` 对象上。
```javascript
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = 'Bearer token';
axios.defaults.timeout = 5000;
```

#### 2. 实例化配置
当项目中需要对接多个不同的后端服务（不同 BaseURL、不同超时时间）时使用。
```javascript
const instance = axios.create({
  baseURL: 'https://another-api.example.com',
  timeout: 2000,
});
// 针对该实例的请求
instance.get('/users');
```

#### 3. 单次请求配置
在发起具体请求时传入的配置，**优先级最高**，会覆盖全局和实例配置。
```javascript
axios.get('/user/12345', {
    timeout: 10000, // 单独覆盖超时时间
    headers: {'X-Custom-Header': 'foobar'}
});
```

---

### 三、 按核心配置项分类
Axios 的 `config` 对象非常庞大，按功能可分为以下几类：

#### 1. 基础请求配置
*   `url`：请求的服务器 URL
*   `method`：创建请求时使用的方法（默认 `get`）
*   `baseURL`：自动加在 `url` 前面
*   `params`：URL 查询参数（与 `url` 拼接，如 `?ID=123`）
*   `data`：作为请求体发送的数据（适用于 POST/PUT/PATCH）

#### 2. 请求头配置
*   `headers`：自定义请求头（如 `Content-Type`, `Authorization`）

#### 3. 超时与跨域配置
*   `timeout`：指定请求超时的毫秒数（0 表示无超时）
*   `withCredentials`：跨域请求时是否需要使用凭证（Cookie）

#### 4. 响应数据格式配置
*   `responseType`：服务器响应的数据类型（`json` [默认], `text`, `blob`, `arraybuffer`, `stream`, `document`）
*   `responseEncoding`：响应的解码方式（默认 `utf8`）

#### 5. 进度与底层控制
*   `onUploadProgress`：上传进度回调
*   `onDownloadProgress`：下载进度回调
*   `maxContentLength`：定义允许的响应内容的最大字节数
*   `maxBodyLength`：定义允许的 HTTP 请求内容的最大字节数

---

### 四、 按拦截器分类
Axios 拦截器分为两种，常用于统一处理 Token 注入、日志记录、错误重试等：

#### 1. 请求拦截器
在请求发送前执行。
```javascript
axios.interceptors.request.use(config => {
    // 在发送请求之前做些什么（如添加 token）
    config.headers.Authorization = `Bearer ${getToken()}`;
    return config;
}, error => {
    // 对请求错误做些什么
    return Promise.reject(error);
});
```

#### 2. 响应拦截器
在收到响应数据后、进入业务代码 `.then` 之前执行。
```javascript
axios.interceptors.response.use(response => {
    // 对响应数据做点什么（如剥离外层 data）
    return response.data;
}, error => {
    // 对响应错误做点什么（如 401 跳转登录页）
    if (error.response.status === 401) {
        router.push('/login');
    }
    return Promise.reject(error);
});
```

*(注：如果使用 `axios.create()` 创建实例，拦截器需挂载在实例上，如 `instance.interceptors...`)*

---

### 五、 按响应数据结构分类
Axios 的 `.then()` 接收到的响应对象包含以下完整字段：

*   `data`：服务器返回的实际数据（已经被解析为 JSON 对象或按 responseType 转换）。
*   `status`：服务器响应的 HTTP 状态码（如 200, 404, 500）。
*   `statusText`：服务器响应的 HTTP 状态信息（如 "OK"）。
*   `headers`：服务器响应的所有头信息。
*   `config`：生成该请求时 Axios 使用的完整配置对象。
*   `request`：生成该请求的底层 XMLHttpRequest 或 Node.js http.ClientRequest 对象。

---

### 六、 按错误处理分类
Axios 在请求非 2xx 状态码或发生网络错误时会进入 `catch`，错误对象主要分为以下几类：

#### 1. 服务端响应错误
请求已到达服务器，服务器返回了状态码，但状态码不在 2xx 范围内。
*   `error.response`：存在此对象，包含 `data`, `status`, `headers`。

#### 2. 无响应错误
请求已发出，但没有收到服务器的响应（如网络断开、跨域被拦截、目标服务器宕机）。
*   `error.request`：存在此对象（XMLHttpRequest 实例）。

#### 3. 请求配置错误
在发送请求前，Axios 内部配置出错（如 URL 格式错误）。
*   `error.message`：包含错误信息。

*验证状态码配置：*
可以通过 `validateStatus` 修改 Axios 判定请求成功的条件：
```javascript
validateStatus: function (status) {
  return status >= 200 && status < 300; // 默认值。可以改成 return status < 500; 
}
```

---

### 七、 按高级特性分类

#### 1. 取消请求
Axios 支持取消正在进行的请求（常用于防抖、组件卸载时取消未完成请求、搜索框切换关键词时取消旧请求）。
*   **旧版 API（AbortController 现代标准推荐）**：
    ```javascript
    const controller = new AbortController();
    axios.get('/user/12345', { signal: controller.signal });
    // 取消请求
    controller.abort();
    ```
*   **废弃的旧 API（CancelToken）**：在新版本中逐渐被弃用，但仍可见。

#### 2. 并发请求
同时发起多个请求，等所有请求都完成后再处理。
*   `axios.all([])`：本质是 `Promise.all`。
*   `axios.spread(callback)`：将多个请求的结果数组展开传给回调函数（新版可用 ES6 解构赋值代替）。
```javascript
axios.all([
  axios.get('/user'),
  axios.get('/orders')
]).then(axios.spread((userRes, ordersRes) => {
  // 两个请求都完成
}));
```

#### 3. 数据转换器
允许在发送前或接收后对数据进行手动转换。
*   `transformRequest`：修改请求数据（只能在 PUT/POST/PATCH 中使用）。
*   `transformResponse`：修改响应数据（在传递给 then/catch 前）。

#### 4. XSRF/CSRF 防护
Axios 内置了跨站请求伪造防护机制。
*   `xsrfCookieName`：用作 xsrf token 值的 cookie 名称（默认 `XSRF-TOKEN`）。
*   `xsrfHeaderName`：携带 xsrf token 值的 HTTP 头名称（默认 `X-XSRF-TOKEN`）。

#### 5. 表单数据提交
支持自动序列化 FormData 对象，常用于文件上传。
```javascript
const formData = new FormData();
formData.append('file', fileObj);
axios.post('/upload', formData, {
  headers: { 'Content-Type': 'multipart/form-data' }
});
```

#### 6. 自动 JSON 处理
Axios 默认会将发送的 JS 对象 `JSON.stringify`，并将收到的 JSON 字符串 `JSON.parse`，无需手动转换。

---

### 总结
Axios 的设计非常模块化和灵活，上述分类涵盖了从**底层 HTTP 协议支持**、到**实例化与配置继承**、再到**生命周期拦截**以及**并发/取消等高级特性**。在实际企业级开发中，通常会使用“**实例化 + 拦截器 + 取消请求 + 统一错误处理**”的组合来封装一个通用的 HTTP 请求服务。

# Axios 

## 一、Axios 简介

**Axios** 是一个基于 Promise 的 HTTP 客户端，用于浏览器和 Node.js。

### 特性：
- ✅ 从浏览器创建 XMLHttpRequests
- ✅ 从 Node.js 创建 HTTP 请求
- ✅ 支持 Promise API
- ✅ 拦截请求和响应
- ✅ 转换请求和响应数据
- ✅ 取消请求
- ✅ 自动转换 JSON 数据
- ✅ 客户端支持防御 XSRF

---

## 二、基本用法

### 1. **安装**

```bash
npm install axios
# 或
yarn add axios
```

### 2. **导入**

```javascript
// ES6 模块
import axios from 'axios';

// CommonJS
const axios = require('axios');
```

### 3. **发送 GET 请求**

```javascript
// 基本 GET 请求
axios.get('/api/users')
  .then(response => {
    console.log(response.data);
  })
  .catch(error => {
    console.error(error);
  });

// 带参数
axios.get('/api/users', {
  params: {
    id: 123,
    name: '张三'
  }
});

// async/await
async function fetchUser() {
  try {
    const response = await axios.get('/api/users/123');
    console.log(response.data);
  } catch (error) {
    console.error(error);
  }
}

// 使用 URL 传递参数
axios.get('/api/users?id=123&name=张三');
```

### 4. **发送 POST 请求**

```javascript
// 基本 POST 请求
axios.post('/api/users', {
  name: '张三',
  age: 25
})
  .then(response => {
    console.log(response.data);
  });

// async/await
async function createUser() {
  try {
    const response = await axios.post('/api/users', {
      name: '张三',
      age: 25
    });
    console.log(response.data);
  } catch (error) {
    console.error(error);
  }
}

// 带配置对象
axios.post('/api/users', 
  { name: '张三' },
  {
    headers: { 'Content-Type': 'application/json' },
    timeout: 5000
  }
);
```

### 5. **其他 HTTP 方法**

```javascript
// PUT - 更新
axios.put('/api/users/123', { name: '李四' });

// PATCH - 部分更新
axios.patch('/api/users/123', { age: 26 });

// DELETE - 删除
axios.delete('/api/users/123');

// HEAD
axios.head('/api/users/123');

// OPTIONS
axios.options('/api/users/123');
```

---

## 三、通用请求方法

### `axios(config)`

```javascript
// 基本用法
axios({
  method: 'get',
  url: '/api/users',
  params: { id: 123 }
});

// POST 请求
axios({
  method: 'post',
  url: '/api/users',
  data: { name: '张三', age: 25 }
});

// 完整配置
axios({
  method: 'post',
  url: '/api/users',
  baseURL: 'https://api.example.com',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123'
  },
  params: { version: 'v1' },
  data: { name: '张三' },
  timeout: 5000,
  responseType: 'json'
});
```

---

## 四、并发请求

### 1. **`axios.all()`**

```javascript
// 同时发送多个请求
axios.all([
  axios.get('/api/users'),
  axios.get('/api/posts')
])
  .then(axios.spread((usersResponse, postsResponse) => {
    console.log('用户:', usersResponse.data);
    console.log('文章:', postsResponse.data);
  }));

// async/await 方式（推荐）
async function fetchData() {
  const [usersResponse, postsResponse] = await Promise.all([
    axios.get('/api/users'),
    axios.get('/api/posts')
  ]);
  
  console.log('用户:', usersResponse.data);
  console.log('文章:', postsResponse.data);
}
```

---

## 五、配置对象

### 完整配置选项：

```javascript
axios({
  // 请求 URL
  url: '/api/users',
  
  // 请求方法
  method: 'get', // get, post, put, delete, patch, head, options
  
  // 基础 URL（会添加到 url 前面）
  baseURL: 'https://api.example.com',
  
  // 请求头
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123',
    'X-Custom-Header': 'value'
  },
  
  // URL 参数（GET 请求）
  params: {
    id: 123,
    name: '张三'
  },
  
  // 参数序列化
  paramsSerializer: function(params) {
    return Qs.stringify(params, { arrayFormat: 'brackets' });
  },
  
  // 请求体（POST/PUT/PATCH）
  data: {
    name: '张三',
    age: 25
  },
  
  // 超时时间（毫秒）
  timeout: 5000,
  
  // 跨域请求是否携带凭证
  withCredentials: false,
  
  // 适配器（自定义请求处理）
  adapter: function(config) {
    /* ... */
  },
  
  // 响应数据类型
  responseType: 'json', // arraybuffer, blob, document, json, text, stream
  
  // 响应编码
  responseEncoding: 'utf8',
  
  // XSRF cookie 名称
  xsrfCookieName: 'XSRF-TOKEN',
  
  // XSRF header 名称
  xsrfHeaderName: 'X-XSRF-TOKEN',
  
  // 上传进度
  onUploadProgress: function(progressEvent) {
    console.log('上传进度:', Math.round(progressEvent.loaded * 100 / progressEvent.total));
  },
  
  // 下载进度
  onDownloadProgress: function(progressEvent) {
    console.log('下载进度:', Math.round(progressEvent.loaded * 100 / progressEvent.total));
  },
  
  // 最大响应大小（字节）
  maxContentLength: 2000,
  
  // 最大重定向次数
  maxRedirects: 5,
  
  // 验证响应状态码
  validateStatus: function(status) {
    return status >= 200 && status < 300; // 默认
  },
  
  // 最大 HTTP 头数量
  maxHeaderCount: 5000,
  
  // 代理配置
  proxy: {
    protocol: 'http',
    host: '127.0.0.1',
    port: 9000,
    auth: {
      username: 'user',
      password: 'pass'
    }
  },
  
  // 取消令牌
  cancelToken: new CancelToken(function(cancel) {})
});
```

---

## 六、响应结构

```javascript
axios.get('/api/users/123')
  .then(response => {
    console.log(response);
    /*
    {
      // 服务器返回的数据
      data: { id: 123, name: '张三' },
      
      // HTTP 状态码
      status: 200,
      
      // HTTP 状态消息
      statusText: 'OK',
      
      // 响应头
      headers: {
        'content-type': 'application/json',
        'cache-control': 'no-cache'
      },
      
      // 请求配置
      config: {
        url: '/api/users/123',
        method: 'get',
        // ...
      },
      
      // 原始请求对象
      request: XMLHttpRequest {}
    }
    */
    
    // 通常只需要 data
    console.log(response.data);
  });
```

---

## 七、错误处理

```javascript
axios.get('/api/users/123')
  .then(response => {
    console.log(response.data);
  })
  .catch(error => {
    if (error.response) {
      // 服务器响应了状态码（2xx 之外）
      console.log('错误状态码:', error.response.status);
      console.log('错误数据:', error.response.data);
      console.log('错误头:', error.response.headers);
      
      // 根据状态码处理
      switch (error.response.status) {
        case 400:
          console.log('请求参数错误');
          break;
        case 401:
          console.log('未授权，请重新登录');
          break;
        case 403:
          console.log('拒绝访问');
          break;
        case 404:
          console.log('请求的资源不存在');
          break;
        case 500:
          console.log('服务器内部错误');
          break;
      }
    } else if (error.request) {
      // 请求已发送但没有收到响应
      console.log('网络错误，请检查网络连接');
    } else {
      // 设置请求时发生错误
      console.log('请求配置错误:', error.message);
    }
    
    // Axios 错误对象
    console.log('完整错误:', error.config);
  });

// async/await 错误处理
async function fetchUser() {
  try {
    const response = await axios.get('/api/users/123');
    console.log(response.data);
  } catch (error) {
    if (error.response) {
      console.error('服务器错误:', error.response.status);
    } else if (error.request) {
      console.error('网络错误');
    } else {
      console.error('其他错误:', error.message);
    }
  }
}
```

---

## 八、拦截器（Interceptors）

### 1. **请求拦截器**

```javascript
// 添加请求拦截器
axios.interceptors.request.use(
  function(config) {
    // 请求发送前做些什么
    console.log('请求拦截器:', config);
    
    // 添加 token
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    
    // 添加时间戳
    config.headers['X-Timestamp'] = Date.now();
    
    return config;
  },
  function(error) {
    // 请求错误做些什么
    console.error('请求拦截器错误:', error);
    return Promise.reject(error);
  }
);

// 示例：添加 loading
axios.interceptors.request.use(
  config => {
    // 显示 loading
    showLoading();
    return config;
  },
  error => {
    hideLoading();
    return Promise.reject(error);
  }
);
```

### 2. **响应拦截器**

```javascript
// 添加响应拦截器
axios.interceptors.response.use(
  function(response) {
    // 对响应数据做些什么
    console.log('响应拦截器:', response);
    
    // 隐藏 loading
    hideLoading();
    
    // 返回响应数据
    return response;
  },
  function(error) {
    // 对响应错误做些什么
    console.error('响应拦截器错误:', error);
    
    hideLoading();
    
    // 统一错误处理
    if (error.response) {
      const status = error.response.status;
      
      if (status === 401) {
        // 未授权，跳转登录
        router.push('/login');
        Message.error('登录已过期，请重新登录');
      } else if (status === 403) {
        Message.error('没有权限访问');
      } else if (status === 500) {
        Message.error('服务器错误');
      }
    }
    
    return Promise.reject(error);
  }
);

// 示例：统一处理业务错误码
axios.interceptors.response.use(
  response => {
    const res = response.data;
    
    // 假设后端返回格式：{ code: 0, message: 'success', data: {} }
    if (res.code !== 0) {
      Message.error(res.message || '请求失败');
      return Promise.reject(new Error(res.message || '请求失败'));
    }
    
    return response;
  },
  error => {
    return Promise.reject(error);
  }
);
```

### 3. **移除拦截器**

```javascript
const myInterceptor = axios.interceptors.request.use(function() {
  /* ... */
});

// 移除拦截器
axios.interceptors.request.eject(myInterceptor);
```

---

## 九、创建 Axios 实例

### 1. **`axios.create()`**

```javascript
// 创建自定义实例
const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 5000,
  headers: {
    'Content-Type': 'application/json'
  }
});

// 使用实例
api.get('/users');  // 等价于 axios.get('https://api.example.com/users')
api.post('/users', { name: '张三' });

// 为不同 API 创建不同实例
const userApi = axios.create({
  baseURL: 'https://api.example.com/users',
  timeout: 5000
});

const orderApi = axios.create({
  baseURL: 'https://api.example.com/orders',
  timeout: 10000
});

userApi.get('/123');
orderApi.get('/456');
```

### 2. **实例拦截器**

```javascript
const api = axios.create({
  baseURL: 'https://api.example.com'
});

// 只为这个实例添加拦截器
api.interceptors.request.use(config => {
  config.headers['X-Custom'] = 'value';
  return config;
});

api.interceptors.response.use(
  response => response,
  error => Promise.reject(error)
);
```

---

## 十、取消请求

### 1. **CancelToken（旧版）**

```javascript
// 创建取消令牌
const CancelToken = axios.CancelToken;
const source = CancelToken.source();

// 发送请求
axios.get('/api/users', {
  cancelToken: source.token
})
  .then(response => {
    console.log(response.data);
  })
  .catch(thrown => {
    if (axios.isCancel(thrown)) {
      console.log('请求已取消:', thrown.message);
    } else {
      console.error('请求错误:', thrown);
    }
  });

// 取消请求
source.cancel('用户取消了请求');

// 或者在创建时指定
axios.get('/api/users', {
  cancelToken: new CancelToken(function executor(c) {
    // executor 函数接收取消函数作为参数
    cancel = c;
  })
});

// 取消
cancel('取消请求');
```

### 2. **AbortController（推荐，新版）**

```javascript
// 创建 AbortController
const controller = new AbortController();

// 发送请求
axios.get('/api/users', {
  signal: controller.signal
})
  .then(response => {
    console.log(response.data);
  })
  .catch(thrown => {
    if (axios.isCancel(thrown)) {
      console.log('请求已取消');
    }
  });

// 取消请求
controller.abort();
```

---

## 十一、你的项目中的实际用法

让我查看你项目的 Axios 封装：


现在让我查看项目中如何使用这个封装的 request：


完美！现在我可以给你完整的项目实际用法了：

---

## 十二、你项目中的 Axios 使用方式

### 1. **项目封装（request.js）**

你的项目已经对 Axios 进行了完整封装：

```javascript
// src/utils/request.js

// 创建 Axios 实例
const service = axios.create({
  baseURL: process.env.VUE_APP_BASE_API,  // 基础 URL
  timeout: 90000  // 请求超时 90 秒
});

// 请求拦截器
service.interceptors.request.use(
  config => {
    // 显示 loading
    if (!config.notLoading) {
      loadingCount.value++;
      loadingInstance.start();
    }
    
    // 添加 token 等认证信息
    const conf = httpHandle.requestInterceptor(config, authorization);
    return { ...config, ...conf };
  },
  error => {
    // 请求错误处理
    return Promise.reject(error);
  }
);

// 响应拦截器
service.interceptors.response.use(
  response => {
    // 隐藏 loading
    if (!response.config.notLoading) {
      loadingCount.value--;
      if (loadingCount.value <= 0) {
        loadingInstance.finish();
      }
    }
    
    // 处理系统异常
    if (response.data.systemException) {
      ElMessage.warning('系统异常，请联系管理员！');
      return null;
    }
    
    return response;
  },
  error => {
    // 错误提示
    ElMessage.error(error.message);
    return Promise.reject(error);
  }
);

// 导出 request 方法（自动添加 language 参数）
export function request(config) {
  // 自动添加 language 参数
  config.params.language = i18n?.global?.locale?.value;
  return service(config);
}

export default service;
```

---

### 2. **API 层定义**

```javascript
// src/api/pmsLineBaseData.js
import request from '@/utils/request';

/**
 * 查询 PMS 线路基础数据列表
 */
export function queryLineBaseDataList(params) {
  return request({
    url: `/sa-rpc/api/v1/pms-line-base-data/queryList`,
    method: 'get',
    params  // GET 请求参数
  });
}

/**
 * 其他示例
 */
export function createLine(data) {
  return request({
    url: `/sa-rpc/api/v1/pms-line-base-data/create`,
    method: 'post',
    data  // POST 请求体
  });
}

export function updateLine(id, data) {
  return request({
    url: `/sa-rpc/api/v1/pms-line-base-data/update/${id}`,
    method: 'put',
    data
  });
}

export function deleteLine(id) {
  return request({
    url: `/sa-rpc/api/v1/pms-line-base-data/delete/${id}`,
    method: 'delete'
  });
}
```

---

### 3. **组件中使用**

```vue
<script>
import { queryLineBaseDataList } from '@/api/pmsLineBaseData';

export default {
  data() {
    return {
      tableData: {
        list: [],
        total: 0
      },
      page: {
        pageNum: 1,
        pageSize: 10
      },
      cLform: {
        contractNo: '',
        supplier: ''
      }
    };
  },
  methods: {
    // 查询数据
    searchTable(page) {
      if (page === 1) {
        this.page.pageNum = 1;
      }
      
      // 合并查询条件和分页参数
      const params = {
        ...this.cLform,
        ...this.page
      };
      
      this.tableLoading = true;
      
      // 调用 API
      queryLineBaseDataList(params)
        .then((res) => {
          if (res.data && res.data.success) {
            this.tableData.list = res.data.result?.result || [];
            this.tableData.total = res.data.result?.count || 0;
          } else {
            this.$message.warning(res.data?.resultMessage || '查询失败');
          }
        })
        .catch((error) => {
          console.error('查询失败：', error);
          this.$message.error('查询失败');
        })
        .finally(() => {
          this.tableLoading = false;
        });
    },
    
    // async/await 方式
    async searchTableAsync() {
      try {
        const params = {
          ...this.cLform,
          ...this.page
        };
        
        const res = await queryLineBaseDataList(params);
        
        if (res.data && res.data.success) {
          this.tableData.list = res.data.result?.result || [];
          this.tableData.total = res.data.result?.count || 0;
        } else {
          this.$message.warning(res.data?.resultMessage || '查询失败');
        }
      } catch (error) {
        console.error('查询失败：', error);
        this.$message.error('查询失败');
      } finally {
        this.tableLoading = false;
      }
    }
  }
};
</script>
```

---

### 4. **实际项目中的用法**

让我查看你项目中的实际调用：


完美！现在让我给你完整的总结：

---

## 十三、项目中的完整流程

### 流程图：

```
组件调用
  ↓
API 层 (pmsLineBaseData.js)
  ↓
request() 函数 (自动添加 language)
  ↓
请求拦截器 (添加 token、显示 loading)
  ↓
发送到服务器
  ↓
响应拦截器 (处理错误、隐藏 loading)
  ↓
返回到组件
```

### 实际代码：

```javascript
// 1. 组件中调用
// pmsLineBaseDataQuery.vue 第 559 行
queryLineBaseDataList({ ...data, ...this.page })
  .then((res) => {
    // 处理成功响应
    if (res.data && res.data.success) {
      this.tableData.list = res.data.result?.result || [];
      this.tableData.total = res.data.result?.count || 0;
    }
  })
  .catch((error) => {
    // 处理错误
    this.$message.error('查询失败');
  })
  .finally(() => {
    // 无论成功失败都执行
    this.tableLoading = false;
  });

// 2. API 层定义
// pmsLineBaseData.js
export function queryLineBaseDataList(params) {
  return request({
    url: `/sa-rpc/api/v1/pms-line-base-data/queryList`,
    method: 'get',
    params  // 自动附加到 URL
  });
}

// 3. request.js 处理
// 自动添加 language 参数
// 请求拦截器：显示 loading、添加 token
// 响应拦截器：隐藏 loading、处理错误
```

---

## 十四、Axios 完整用法对比表

| 用法            | 示例                                      | 说明       |
| --------------- | ----------------------------------------- | ---------- |
| **GET 请求**    | `axios.get(url, { params })`              | 查询数据   |
| **POST 请求**   | `axios.post(url, data)`                   | 创建数据   |
| **PUT 请求**    | `axios.put(url, data)`                    | 完整更新   |
| **PATCH 请求**  | `axios.patch(url, data)`                  | 部分更新   |
| **DELETE 请求** | `axios.delete(url)`                       | 删除数据   |
| **并发请求**    | `Promise.all([axios.get(), axios.get()])` | 同时请求   |
| **拦截器**      | `axios.interceptors.request.use()`        | 统一处理   |
| **实例**        | `axios.create({ baseURL })`               | 自定义配置 |
| **取消请求**    | `controller.abort()`                      | 取消请求   |
| **上传进度**    | `onUploadProgress`                        | 上传进度   |
| **下载进度**    | `onDownloadProgress`                      | 下载进度   |

---

## 十五、最佳实践

### ✅ 推荐做法：

```javascript
// 1. 创建 API 层统一管理
// api/user.js
export function getUserList(params) {
  return request({
    url: '/api/users',
    method: 'get',
    params
  });
}

// 2. 使用 async/await
async function fetchUsers() {
  try {
    const res = await getUserList({ page: 1 });
    this.users = res.data.result;
  } catch (error) {
    this.$message.error('获取用户列表失败');
  }
}

// 3. 统一错误处理（在拦截器中）
axios.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 401) {
      router.push('/login');
    }
    return Promise.reject(error);
  }
);

// 4. 使用实例区分不同 API
const userApi = axios.create({ baseURL: '/api/users' });
const orderApi = axios.create({ baseURL: '/api/orders' });

// 5. 不显示 loading 的请求
request({
  url: '/api/check',
  notLoading: true  // 自定义配置
});
```

### ❌ 避免做法：

```javascript
// ❌ 不要在组件中直接写 URL
axios.get('https://api.example.com/users');

// ✅ 应该在 API 层定义
export function getUsers() {
  return request({ url: '/api/users', method: 'get' });
}

// ❌ 不要重复处理错误
axios.get('/api/users')
  .then(res => {
    if (res.data.code !== 0) {
      this.$message.error(res.data.message);  // 拦截器已处理
    }
  });

// ✅ 依赖拦截器统一处理
axios.get('/api/users')
  .then(res => {
    this.users = res.data.result;
  });
```

---

## 总结

你的项目中 Axios 使用流程：

1. **封装实例**（`request.js`）- 统一配置、拦截器
2. **API 层**（`api/*.js`）- 定义接口方法
3. **组件调用**（`*.vue`）- 使用 Promise 或 async/await

**优势：**
- ✅ 统一管理请求配置
- ✅ 自动处理 loading 状态
- ✅ 自动添加 token 和 language
- ✅ 统一错误处理
- ✅ 代码复用，易于维护

这就是 **Axios 在企业级项目中的标准用法**！🚀

# ES6

**ES6**（全称 ECMAScript 6.0，正式名称为 **ECMAScript 2015**）是 JavaScript 语言的下一代标准，于 2015 年 6 月正式发布。

它的出现是 JavaScript 历史上最重要的里程碑之一。在 ES6 之前，JS 存在很多语法糟粕和局限性（如变量提升、回调地狱、缺乏面向对象的原生支持等）。ES6 不仅引入了大量新语法，让代码更简洁、易读，还弥补了语言在复杂应用开发上的短板，使得 JavaScript 能够胜任大型企业级应用的开发。

以下是 ES6 核心特性的详细介绍，按重要性和使用频率分类：

---

### 一、 变量与作用域

#### 1. `let` 与 `const` (块级作用域)

在 ES6 之前，JS 声明变量只能用 `var`，存在“变量提升”和“全局变量污染”问题。ES6 引入了块级作用域 `{}`。
*   **`let`**：声明变量，具有块级作用域，不存在变量提升（在声明前使用会报错，即“暂时性死区”）。值可修改。
*   **`const`**：声明常量，一旦声明必须初始化，且其绑定的内存地址不能再被修改（如果是对象/数组，内部内容可以修改，但不能重新赋值整个变量）。
```javascript
// var 的问题
if (true) { var a = 1; }
console.log(a); // 1 (泄漏到外部)

// let/const 的解决
if (true) { let b = 2; const c = 3; }
console.log(b); // ReferenceError: b is not defined
```

##### 1. **`let`** - 块级作用域

```javascript
// var 的问题：函数作用域
var a = 1;
if (true) {
  var a = 2;  // 覆盖外面的 a
}
console.log(a);  // 2

// let 的解决方案：块级作用域
let b = 1;
if (true) {
  let b = 2;  // 只在这个块内有效
}
console.log(b);  // 1
```

##### 2. **`const`** - 常量

```javascript
// 基本类型：不可修改
const PI = 3.14159;
PI = 3;  // ❌ 报错

// 引用类型：可以修改属性
const obj = { name: '张三' };
obj.name = '李四';  // ✅ 可以
obj.age = 25;       // ✅ 可以

// 但不能重新赋值
obj = {};  // ❌ 报错

// 数组
const arr = [1, 2, 3];
arr.push(4);  // ✅ 可以
arr = [];     // ❌ 报错
```

##### 3. **对比表**

| 特性         | `var`      | `let`      | `const`    |
| ------------ | ---------- | ---------- | ---------- |
| **作用域**   | 函数作用域 | 块级作用域 | 块级作用域 |
| **变量提升** | ✅          | ❌          | ❌          |
| **重复声明** | ✅          | ❌          | ❌          |
| **可修改**   | ✅          | ✅          | ❌          |
| **推荐度**   | ❌          | ⭐⭐⭐        | ⭐⭐⭐⭐⭐      |

---

### 二、 字符串与解构/展开

#### 1. 模板字符串
使用反引号 `` ` `` 标识，支持多行字符串，并在字符串中直接嵌入变量或表达式（使用 `${}`）。
```javascript
const name = "Axios";
const age = 8;
// 传统拼接: 'My name is ' + name + ', I am ' + age + ' years old.'
const str = `My name is ${name}, 
I am ${age + 1} years old.`; 
```

#### 2. 解构赋值
按照一定模式，从数组和对象中提取值，对变量进行赋值。极大地简化了数据提取的代码。
```javascript
// 对象解构
const user = { name: 'Tom', age: 20, gender: 'male' };
const { name, age } = user; // 提取 name 和 age

// 数组解构
const arr = [1, 2, 3];
const [x, y] = arr; // x=1, y=2

// 函数参数解构（非常常用）
function printUser({ name, age }) {
  console.log(name, age);
}
printUser(user);
```

##### 1. **对象解构**

```javascript
const user = {
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com',
  address: {
    city: '北京',
    district: '朝阳区'
  }
};

// 基本解构
const { name, age } = user;
console.log(name, age);  // '张三' 25

// 重命名
const { name: userName, age: userAge } = user;
console.log(userName, userAge);

// 默认值
const { name, gender = '未知' } = user;
console.log(gender);  // '未知'

// 嵌套解构
const { address: { city, district } } = user;
console.log(city, district);  // '北京' '朝阳区'

// 剩余属性
const { name, ...rest } = user;
console.log(rest);  // { age: 25, email: '...', address: {...} }
```

##### 2. **数组解构**

```javascript
const numbers = [1, 2, 3, 4, 5];

// 基本解构
const [first, second] = numbers;
console.log(first, second);  // 1 2

// 跳过元素
const [first, , third] = numbers;
console.log(first, third);  // 1 3

// 默认值
const [a, b, c, d = 0] = [1, 2, 3];
console.log(d);  // 0

// 剩余元素
const [first, ...rest] = numbers;
console.log(rest);  // [2, 3, 4, 5]

// 交换变量
let x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y);  // 2 1
```

##### 3. **函数参数解构**

```javascript
// 对象参数
function greet({ name, age }) {
  console.log(`你好，${name}，今年${age}岁`);
}

greet({ name: '张三', age: 25 });

// 默认值
function createUser({ name = '匿名用户', age = 0 } = {}) {
  return { name, age };
}

createUser();  // { name: '匿名用户', age: 0 }

// 数组参数
function sum([a, b]) {
  return a + b;
}

sum([1, 2]);  // 3
```

---

#### 3.展开运算符（Spread）

##### 1. **数组展开**

```javascript
// 合并数组
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const merged = [...arr1, ...arr2];  // [1, 2, 3, 4, 5, 6]

// 复制数组
const copy = [...arr1];

// 添加元素
const newArr = [...arr1, 4, 5];  // [1, 2, 3, 4, 5]

// 转换为数组
const nodeList = document.querySelectorAll('div');
const arr = [...nodeList];

// Math 函数
const numbers = [1, 5, 3, 9, 2];
Math.max(...numbers);  // 9
Math.min(...numbers);  // 1
```

##### 2. **对象展开**

```javascript
// 合并对象
const obj1 = { a: 1, b: 2 };
const obj2 = { b: 3, c: 4 };
const merged = { ...obj1, ...obj2 };  // { a: 1, b: 3, c: 4 }

// 复制对象
const copy = { ...obj1 };

// 添加属性
const newObj = { ...obj1, c: 3 };

// 更新属性
const updated = { ...obj1, b: 10 };  // { a: 1, b: 10 }

// 删除属性（结合解构）
const { b, ...rest } = obj1;  // rest = { a: 1 }
```

##### 3. **函数参数**

```javascript
// 剩余参数
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

sum(1, 2, 3, 4);  // 10

// 合并参数
function greet(greeting, ...names) {
  return `${greeting}, ${names.join(' and ')}!`;
}

greet('Hello', '张三', '李四', '王五');
// 'Hello, 张三 and 李四 and 王五!'
```

---



### 三、 函数增强

#### 1. 箭头函数

引入 `=>` 定义函数。它不仅简化了代码，更重要的是**改变了 `this` 的指向**。
*   **特性**：箭头函数没有自己的 `this`，它的 `this` 继承自定义时所在上下文（词法作用域）的 `this`。这也意味着它不能用作构造函数，也不能使用 `call/apply/bind` 改变 `this`。
```javascript
// 传统写法
const add = function(a, b) { return a + b; };
// 箭头函数
const add = (a, b) => a + b;

// 解决回调中 this 丢失的问题
function Timer() {
  this.seconds = 0;
  setInterval(() => {
    this.seconds++; // 此处 this 指向 Timer 实例
  }, 1000);
}
```

##### 1. **基本语法**

```javascript
// 传统函数
function add(a, b) {
  return a + b;
}

// 箭头函数
const add = (a, b) => {
  return a + b;
};

// 简写（单行返回）
const add = (a, b) => a + b;

// 单个参数可省略括号
const double = x => x * 2;

// 无参数
const sayHello = () => console.log('Hello');
```

##### 2. **`this` 指向**

```javascript
// 传统函数：this 指向调用者
const obj = {
  name: '张三',
  sayName: function() {
    console.log(this.name);  // '张三'
  }
};

// 箭头函数：this 指向定义时的上下文
const obj2 = {
  name: '李四',
  sayName: () => {
    console.log(this.name);  // undefined（指向全局）
  }
};

// 实际应用场景
class Counter {
  constructor() {
    this.count = 0;
  }
  
  // ✅ 使用箭头函数保持 this
  start() {
    setInterval(() => {
      this.count++;  // this 指向 Counter 实例
      console.log(this.count);
    }, 1000);
  }
  
  // ❌ 传统函数会丢失 this
  startBad() {
    setInterval(function() {
      this.count++;  // this 指向 window/global
    }, 1000);
  }
}
```

##### 3. **不适用场景**

```javascript
// ❌ 对象方法（需要 this）
const obj = {
  name: '张三',
  sayName: () => {
    console.log(this.name);  // undefined
  }
};

// ✅ 应使用传统方法
const obj = {
  name: '张三',
  sayName() {
    console.log(this.name);  // '张三'
  }
};

// ❌ 构造函数
const Person = () => {};  // 报错
const person = new Person();

// ❌ 原型方法
function Person() {}
Person.prototype.sayName = () => {
  console.log(this.name);  // undefined
};

// ❌ 需要 arguments
const sum = () => {
  console.log(arguments);  // 报错
};
sum(1, 2, 3);
```

#### 2. 参数默认值

直接在函数定义时为参数赋默认值。
```javascript
function greet(name = 'Guest') {
  console.log(`Hello, ${name}`);
}
greet(); // Hello, Guest
```

#### 3. rest 参数
使用 `...变量名` 获取函数的多余参数，返回一个数组，替代了以前笨拙的 `arguments` 对象。
```javascript
function sum(...numbers) {
  return numbers.reduce((prev, cur) => prev + cur, 0);
}
sum(1, 2, 3, 4); // 10
```

---

### 四、 扩展运算符

同样是 `...` 语法，但它用在数组或对象前面，用于将数组或对象“展开”成逗号分隔的序列。在合并数组、拷贝对象时极为常用。
```javascript
const arr1 = [1, 2];
const arr2 = [3, 4];
const mergedArr = [...arr1, ...arr2]; // [1, 2, 3, 4]

const obj1 = { a: 1 };
const obj2 = { b: 2 };
const mergedObj = { ...obj1, ...obj2 }; // { a: 1, b: 2 } (浅拷贝)
```

---

### 五、 面向对象编程

#### 1. `class` 类

ES6 提供了更接近传统面向对象语言（如 Java/C++）的 `class` 语法。本质上还是基于原型的语法糖，但极大地提升了代码的易读性和编写效率。
```javascript
class Animal {
  // 构造函数
  constructor(name) {
    this.name = name;
  }
  // 实例方法
  speak() {
    console.log(`${this.name} makes a sound.`);
  }
  // 静态方法 (挂在类本身上)
  static create(name) {
    return new Animal(name);
  }
}

class Dog extends Animal {
  speak() {
    super.speak(); // super 调用父类方法
    console.log(`${this.name} barks.`);
  }
}

const dog = new Dog('Rex');
dog.speak(); 
```

##### 1. **基本语法**

```javascript
// ES5 构造函数
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.sayName = function() {
  console.log(this.name);
};

// ES6 类
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  sayName() {
    console.log(this.name);
  }
}

// 使用
const person = new Person('张三', 25);
person.sayName();  // '张三'
```

##### 2. **继承**

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    console.log(`${this.name} makes a noise`);
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);  // 调用父类构造函数
    this.breed = breed;
  }
  
  // 重写父类方法
  speak() {
    console.log(`${this.name} barks`);
  }
  
  fetch() {
    console.log(`${this.name} fetches the ball`);
  }
}

const dog = new Dog('旺财', '金毛');
dog.speak();  // '旺财 barks'
dog.fetch();  // '旺财 fetches the ball'
```

##### 3. **静态方法**

```javascript
class MathHelper {
  static add(a, b) {
    return a + b;
  }
  
  static subtract(a, b) {
    return a - b;
  }
}

MathHelper.add(1, 2);  // 3
MathHelper.subtract(5, 3);  // 2
```

##### 4. **Getter 和 Setter**

```javascript
class User {
  constructor(name) {
    this._name = name;
  }
  
  get name() {
    return this._name.toUpperCase();
  }
  
  set name(value) {
    if (value.length < 2) {
      throw new Error('名字太短');
    }
    this._name = value;
  }
}

const user = new User('张三');
console.log(user.name);  // '张三'
user.name = '李';  // 报错
```

---

### 六、 异步编程

#### 1. Promise 对象

传统 JS 处理异步依赖回调函数，多层嵌套会导致“回调地狱”，代码难以维护。Promise 提供了链式调用的解决方案。
*   **三种状态**：`pending`（进行中）、`fulfilled`（已成功）、`rejected`（已失败）。状态一旦改变不可逆。
```javascript
const fetchData = new Promise((resolve, reject) => {
  // 异步操作
  setTimeout(() => {
    if (success) {
      resolve('数据获取成功'); // 将状态改为 fulfilled
    } else {
      reject('网络错误'); // 将状态改为 rejected
    }
  }, 1000);
});

fetchData
  .then(data => console.log(data)) // 处理成功
  .catch(err => console.error(err)) // 处理失败
  .finally(() => console.log('执行完毕')); // 无论成功失败都执行
```
*(注：ES2017 引入的 `async/await` 是基于 Promise 的终极异步解决方案，让异步代码看起来像同步代码，但在严格分类上属于 ES8，常与 ES6 一起被提及。)*

##### 1. **基本用法**

```javascript
// 创建 Promise
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve('成功');
    } else {
      reject('失败');
    }
  }, 1000);
});

// 使用
promise
  .then(result => {
    console.log(result);  // '成功'
  })
  .catch(error => {
    console.error(error);
  })
  .finally(() => {
    console.log('完成');
  });
```

##### 2. **链式调用**

```javascript
function fetchUser(id) {
  return new Promise(resolve => {
    setTimeout(() => resolve({ id, name: '张三' }), 1000);
  });
}

function fetchPosts(userId) {
  return new Promise(resolve => {
    setTimeout(() => resolve([{ id: 1, title: '文章1' }]), 1000);
  });
}

// 链式调用
fetchUser(1)
  .then(user => {
    console.log(user);
    return fetchPosts(user.id);
  })
  .then(posts => {
    console.log(posts);
  })
  .catch(error => {
    console.error(error);
  });
```

##### 3. **async/await**

```javascript
// Promise 方式
fetchUser(1)
  .then(user => fetchPosts(user.id))
  .then(posts => console.log(posts))
  .catch(error => console.error(error));

// async/await 方式（推荐）
async function getUserAndPosts() {
  try {
    const user = await fetchUser(1);
    console.log(user);
    
    const posts = await fetchPosts(user.id);
    console.log(posts);
  } catch (error) {
    console.error(error);
  }
}

getUserAndPosts();
```

##### 4. **并发处理**

```javascript
// Promise.all - 所有都成功才成功
const promise1 = Promise.resolve(1);
const promise2 = Promise.resolve(2);
const promise3 = Promise.resolve(3);

Promise.all([promise1, promise2, promise3])
  .then(values => {
    console.log(values);  // [1, 2, 3]
  });

// Promise.race - 第一个完成的
Promise.race([promise1, promise2, promise3])
  .then(value => {
    console.log(value);  // 1
  });

// Promise.allSettled - 所有都完成（不管成功失败）
Promise.allSettled([
  Promise.resolve(1),
  Promise.reject('error'),
  Promise.resolve(3)
]).then(results => {
  console.log(results);
  // [
  //   { status: 'fulfilled', value: 1 },
  //   { status: 'rejected', reason: 'error' },
  //   { status: 'fulfilled', value: 3 }
  // ]
});

// Promise.any - 第一个成功的
Promise.any([
  Promise.reject('error1'),
  Promise.resolve(2),
  Promise.resolve(3)
]).then(value => {
  console.log(value);  // 2
});
```

---

### 七、 模块化

ES6 引入了原生的模块系统，取代了以前社区的 CommonJS (Node.js) 和 AMD (RequireJS) 规范。
*   **`export`**：用于导出模块的变量、函数或类。
*   **`import`**：用于引入其他模块提供的功能。
```javascript
// math.js
export const PI = 3.14159;
export function add(a, b) { return a + b; }
// 默认导出 (一个模块只能有一个)
export default function multiply(a, b) { return a * b; }

// main.js
import multiply, { PI, add } from './math.js';
console.log(multiply(2, 3), PI, add(1, 2));
```
这是现代前端工程化（Webpack, Vite 等）和现代框架的基石。

#### 1. **导出（Export）**

```javascript
// 命名导出
// utils.js
export const PI = 3.14159;
export function add(a, b) {
  return a + b;
}
export class User {
  constructor(name) {
    this.name = name;
  }
}

// 默认导出
// math.js
export default function multiply(a, b) {
  return a * b;
}

// 混合导出
// api.js
export const API_URL = 'https://api.example.com';
export function fetchData() { }
export default function init() { }
```

#### 2. **导入（Import）**

```javascript
// 命名导入
import { PI, add, User } from './utils.js';

// 重命名导入
import { PI as MY_PI, add as sum } from './utils.js';

// 默认导入
import multiply from './math.js';

// 全部导入
import * as Utils from './utils.js';
Utils.add(1, 2);

// 混合导入
import init, { API_URL, fetchData } from './api.js';

// 动态导入
button.onclick = async () => {
  const module = await import('./module.js');
  module.default();
};
```

---

### 八、 新的数据结构与类型

#### 1. `Set` 和 `Map`
*   **Set**：类似于数组，但成员的值都是唯一的，没有重复的值。常用于数组去重。
*   **Map**：类似于对象，也是键值对的集合。但“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。
```javascript
// 数组去重
const uniqueArr = [...new Set([1, 2, 2, 3, 3])]; // [1, 2, 3]

const map = new Map();
const objKey = { name: 'key' };
map.set(objKey, 'value');
map.get(objKey); // 'value'
```

##### 1. **Map**

```javascript
// 创建 Map
const map = new Map();

// 设置值
map.set('name', '张三');
map.set('age', 25);

// 获取值
console.log(map.get('name'));  // '张三'

// 检查是否存在
console.log(map.has('age'));  // true

// 删除
map.delete('age');

// 大小
console.log(map.size);  // 1

// 遍历
map.forEach((value, key) => {
  console.log(`${key}: ${value}`);
});

// 初始化
const userMap = new Map([
  ['name', '张三'],
  ['age', 25],
  ['email', 'zhangsan@example.com']
]);

// 与普通对象的区别
const obj = {};
obj[1] = 'one';
obj['1'] = 'one';  // 覆盖

const map = new Map();
map.set(1, 'one');
map.set('1', 'string one');  // 不覆盖
```

##### 2. **Set**

```javascript
// 创建 Set
const set = new Set();

// 添加
set.add(1);
set.add(2);
set.add(3);
set.add(1);  // 重复，不会添加

// 检查
console.log(set.has(2));  // true

// 删除
set.delete(2);

// 大小
console.log(set.size);  // 2

// 数组去重
const arr = [1, 2, 2, 3, 3, 4];
const unique = [...new Set(arr)];  // [1, 2, 3, 4]

// 初始化
const set2 = new Set([1, 2, 3, 4, 5]);

// 遍历
set.forEach(value => {
  console.log(value);
});
```

---



#### 2. `Symbol`

ES6 引入的第 7 种基本数据类型。表示独一无二的值，主要用于防止对象属性名冲突（常用于定义对象的内部私有属性或迭代器接口）。
```javascript
const s1 = Symbol('id');
const s2 = Symbol('id');
console.log(s1 === s2); // false

// 创建唯一标识
const sym1 = Symbol();
const sym2 = Symbol('description');

console.log(sym1 === sym2);  // false

// 对象属性
const user = {
  [Symbol('id')]: 1,
  name: '张三'
};

// 防止属性名冲突
const SIZE = Symbol();

class Collection {
  constructor() {
    this[SIZE] = 0;
  }
  
  add() {
    this[SIZE]++;
  }
  
  get size() {
    return this[SIZE];
  }
}
```

---

### 九、 其他常用扩展

*   **数组扩展**：`Array.from()` (将伪数组转为数组)、`Array.of()`、`find()`、`findIndex()`、`includes()`。
*   **对象扩展**：属性简写（变量名和属性名一致时可简写）、`Object.keys()`、`Object.values()`、`Object.assign()` (对象合并，浅拷贝)。
*   **Proxy 和 Reflect**：Proxy 用于拦截并自定义对象的基本操作（如属性查找、赋值等，是 Vue 3 响应式系统的核心基础）。Reflect 提供操作对象的 API。

### 总结
ES6 是现代 JavaScript 的分水岭。如果你今天学习 JS，实际上学的大部分都是 ES6+ 的语法。目前所有的主流浏览器都已完全支持 ES6。我们在日常开发（如写 Vue/React 组件、调用 Axios 等）中使用的几乎全是 ES6 语法，Babel 等工具的作用仅仅是将 ES6+ 代码转换为旧版浏览器兼容的 ES5 代码而已。

## 十二、迭代器与生成器

### 1. **迭代器**

```javascript
const iterable = [1, 2, 3];
const iterator = iterable[Symbol.iterator]();

console.log(iterator.next());  // { value: 1, done: false }
console.log(iterator.next());  // { value: 2, done: false }
console.log(iterator.next());  // { value: 3, done: false }
console.log(iterator.next());  // { value: undefined, done: true }

// for...of
for (const item of iterable) {
  console.log(item);
}
```

### 2. **生成器**

```javascript
function* generator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = generator();
console.log(gen.next());  // { value: 1, done: false }
console.log(gen.next());  // { value: 2, done: false }
console.log(gen.next());  // { value: 3, done: false }
console.log(gen.next());  // { value: undefined, done: true }

// 传值
function* counter() {
  let i = 0;
  while (true) {
    const reset = yield i++;
    if (reset) i = 0;
  }
}

const counterGen = counter();
console.log(counterGen.next());  // { value: 0, done: false }
console.log(counterGen.next());  // { value: 1, done: false }
console.log(counterGen.next(true));  // { value: 0, done: false }
```

---

## 十三、Proxy 和 Reflect

### 1. **Proxy**

```javascript
const user = {
  name: '张三',
  age: 25
};

const proxy = new Proxy(user, {
  get(target, prop) {
    console.log(`读取属性: ${prop}`);
    return target[prop];
  },
  
  set(target, prop, value) {
    console.log(`设置属性: ${prop} = ${value}`);
    target[prop] = value;
    return true;
  }
});

proxy.name;  // 读取属性: name
proxy.name = '李四';  // 设置属性: name = 李四
```

### 2. **Reflect**

```javascript
const obj = { name: '张三' };

// 等同于 obj.name
Reflect.get(obj, 'name');

// 等同于 obj.name = '李四'
Reflect.set(obj, 'name', '李四');

// 检查属性
Reflect.has(obj, 'name');

// 删除属性
Reflect.deleteProperty(obj, 'name');
```

---

## 十四、其他重要特性

### 1. **默认参数**

```javascript
// ES5
function greet(name) {
  name = name || '匿名用户';
  console.log(`你好，${name}`);
}

// ES6
function greet(name = '匿名用户') {
  console.log(`你好，${name}`);
}

greet();  // '你好，匿名用户'
greet('张三');  // '你好，张三'
```

### 2. **剩余参数**

```javascript
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

sum(1, 2, 3, 4);  // 10
```

### 3. **新增方法**

```javascript
// 字符串
'hello'.startsWith('h');  // true
'hello'.endsWith('o');  // true
'hello'.includes('ell');  // true
'hello'.repeat(3);  // 'hellohellohello'

// 数组
[1, 2, 3].find(x => x > 1);  // 2
[1, 2, 3].findIndex(x => x > 1);  // 1
[1, 2, 3].includes(2);  // true
Array.from('hello');  // ['h', 'e', 'l', 'l', 'o']
Array.of(1, 2, 3);  // [1, 2, 3]

// 对象
Object.assign({}, { a: 1 }, { b: 2 });  // { a: 1, b: 2 }
Object.keys({ a: 1, b: 2 });  // ['a', 'b']
Object.values({ a: 1, b: 2 });  // [1, 2]
Object.entries({ a: 1, b: 2 });  // [['a', 1], ['b', 2]]
```

---

## ES6 特性使用对比表

| 特性         | ES5 写法                | ES6 写法                  | 推荐度 |
| ------------ | ----------------------- | ------------------------- | ------ |
| **变量声明** | `var`                   | `let` / `const`           | ⭐⭐⭐⭐⭐  |
| **函数**     | `function()`            | `() =>`                   | ⭐⭐⭐⭐   |
| **字符串**   | `'Hello ' + name`       | `` `Hello ${name}` ``     | ⭐⭐⭐⭐⭐  |
| **解构**     | `var name = obj.name`   | `const { name } = obj`    | ⭐⭐⭐⭐⭐  |
| **展开**     | `arr1.concat(arr2)`     | `[...arr1, ...arr2]`      | ⭐⭐⭐⭐⭐  |
| **类**       | `function Person()`     | `class Person`            | ⭐⭐⭐⭐   |
| **Promise**  | 回调地狱                | `Promise` / `async-await` | ⭐⭐⭐⭐⭐  |
| **模块**     | CommonJS                | `import/export`           | ⭐⭐⭐⭐⭐  |
| **默认参数** | `name = name || '默认'` | `name = '默认'`           | ⭐⭐⭐⭐⭐  |

# AJAX

### AJAX 介绍

**AJAX**（全称：**A**synchronous **J**avaScript **A**nd **X**ML，即“异步的 JavaScript 和 XML”）不是一种单一的技术，而是一种**Web 开发技术组合**。它用于在无需重新加载整个网页的情况下，能够更新部分网页内容。

简单来说：**AJAX 让网页实现了“局部刷新”。**

---

### 一、 为什么需要 AJAX？（背景与意义）

在 AJAX 出现之前，传统的 Web 页面工作方式是这样的：
1. 用户点击按钮或提交表单。
2. 浏览器向服务器发送请求。
3. 服务器处理请求并返回一个**完整的 HTML 页面**。
4. 浏览器丢弃旧页面，重新渲染整个新页面。

**传统方式的痛点：**
*   **用户体验极差**：每次交互都会伴随页面的闪烁和白屏等待。
*   **浪费带宽和资源**：即使页面上只有一小部分数据需要改变，也要重新传输整个页面的 HTML、CSS、JS。

**AJAX 的解决思路：**
通过浏览器内置的 `XMLHttpRequest` (XHR) 对象，在后台悄悄与服务器通信。拿到数据后，使用 JavaScript 动态修改网页的某一部分（通过 DOM 操作）。**整个过程不会打断用户的操作，页面也不会刷新。**
> *经典案例：你在刷微博或朋友圈时，往下拉到底部，新的内容自动加载出来，而页面顶部你刚看过的内容丝毫没有跳动，这就是 AJAX 的功劳。*

---

### 二、 AJAX 的核心组成部分

AJAX 是几种技术的强强联合：

1.  **JavaScript**：主角，负责捕获用户事件、发送请求、接收响应并更新页面。
2.  **XMLHttpRequest (XHR)**：核心对象，负责在客户端和服务器端之间传输数据。
3.  **服务器端技术**：如 Node.js, PHP, Java 等，负责接收 AJAX 请求并返回数据。
4.  **数据格式**：
    *   **过去**：使用 **XML** 格式传输数据（这也是 AJAX 名字中 XML 的由来），但 XML 格式冗长，解析麻烦。
    *   **现在**：绝大多数使用 **JSON** 格式（虽然现在用的是 JSON，但名字依然叫 AJAX，没有改成 AJAJ）。
5.  **CSS 和 HTML**：负责展示数据和页面样式。
6.  **DOM**：Document Object Model，JS 通过 DOM 动态修改页面上的元素。

---

### 三、 AJAX 的工作原理

AJAX 的请求生命周期可以简化为以下几步：

1.  **触发事件**：用户在页面上执行操作（如点击“加载更多”）。
2.  **创建 XHR 对象**：JS 创建一个 `XMLHttpRequest` 实例。
3.  **发送请求**：XHR 对象向服务器发送 HTTP 请求（GET 或 POST）。
4.  **服务器处理**：服务器接收请求，查询数据库或处理逻辑，将结果（通常是 JSON 数据）返回。
5.  **接收响应**：XHR 对象监听服务器的响应状态，一旦数据接收完毕，触发回调函数。
6.  **局部更新**：JS 解析返回的数据，使用 DOM API 更新页面的特定部分。

---

### 四、 原生 AJAX 怎么写？（XMLHttpRequest 示例）

在现代框架出现前，写原生的 AJAX 比较繁琐，体现了“回调地狱”的雏形：

```javascript
// 1. 创建 XHR 对象
const xhr = new XMLHttpRequest();

// 2. 配置请求（方法，URL，是否异步）
xhr.open('GET', 'https://api.example.com/data', true);

// 3. 监听状态变化（处理响应）
xhr.onreadystatechange = function () {
  // readyState === 4 表示请求完成，status === 200 表示成功
  if (xhr.readyState === 4 && xhr.status === 200) {
    // 5. 解析服务器返回的 JSON 数据
    const data = JSON.parse(xhr.responseText);
    
    // 6. 使用 JS/DOM 局部更新页面
    document.getElementById('content').innerText = data.message;
  }
};

// 4. 处理错误
xhr.onerror = function() {
  console.error('请求失败');
};

// 5. 发送请求
xhr.send();
```
*注：这种基于事件回调的写法比较古老，且处理复杂异步逻辑时容易陷入“回调地狱”。*

---

### 五、 AJAX 的优缺点

**优点：**
1.  **无刷新更新数据**：极大提升用户体验，操作连贯。
2.  **异步通信**：不打断用户的操作，请求在后台进行。
3.  **减轻服务器和带宽负担**：只请求需要的数据，无需生成完整页面，按需获取。
4.  **前后端分离的基础**：使得前端只负责页面展示，后端只负责提供数据 API 成为可能。

**缺点：**
1.  **破坏浏览器后退功能**：因为页面没有真正刷新，浏览器的历史记录不会记录局部变化，导致点“后退”按钮时表现异常（现代前端框架通过 History API 解决了此问题）。
2.  **对 SEO 不友好**：搜索引擎爬虫通常不会执行 JS 发送 AJAX 请求，导致爬虫抓不到动态加载的内容（现代有 SSR 服务端渲染技术来解决）。
3.  **存在跨域限制**：受浏览器同源策略限制，AJAX 默认只能向同域名、同端口的服务器发请求（需要后端配置 CORS 或前端使用代理解决）。

---

### 六、 AJAX 的进化与现代替代品

AJAX 是一种**概念/模式**，而实现这种模式的具体技术在不断演进：

1.  **原生时代**：`XMLHttpRequest` (XHR)。写法繁琐，基于回调。
2.  **jQuery 时代**：`$.ajax()`。对 XHR 进行了封装，极大地简化了代码，处理了浏览器兼容性问题。曾统治了一个时代。
3.  **现代原生时代**：`Fetch API`。浏览器原生内置，基于 **Promise** 设计，链式调用，比 XHR 更强大、更优雅，替代了 XHR 的地位。
4.  **第三方库时代**：`Axios`（你在上一个问题中看到的）。在 Fetch 的基础上进一步封装，提供了拦截器、自动 JSON 转换、取消请求等高级功能，是目前最流行的请求库。

**总结：**
AJAX 是 Web 2.0 时代的基石。今天我们虽然很少直接手写 `XMLHttpRequest`，甚至不怎么提 AJAX 这个词，但无论是 Vue、React 中的数据加载，还是 Axios、Fetch 的底层运作，其核心思想依然是 AJAX 所倡导的**“异步获取数据，局部渲染页面”**。

# JS语法

## JavaScript 语法

JavaScript 语法是构建所有 JS 应用的基础。现代 JavaScript（基于 ECMAScript 标准，特别是 ES6+）的语法非常丰富且现代化。

下面按照**从基础到进阶**的逻辑，为您详细梳理 JavaScript 的核心语法体系。

---

### 一、 基本词法规则

1.  **区分大小写**：`var` 和 `Var` 是不同的。
2.  **语句与分号**：JS 语句通常以分号 `;` 结尾。虽然 JavaScript 有**自动分号插入（ASI）**机制，但强烈建议手动加分号，以防代码压缩或解析时出现歧义。
3.  **注释**：
    *   单行注释：`// 这是注释`
    *   多行注释：`/* 这是多行注释 */`
4.  **严格模式**：在文件或函数顶部加上 `"use strict";`，启用更严格的解析和错误处理，现代开发推荐始终使用。

---

### 二、 变量声明

JS 提供了三种声明变量的方式（ES6+ 推荐使用后两种）：

*   `var`：**传统方式**。存在变量提升，允许重复声明，属于函数级作用域。**现代开发不推荐使用**。
*   `let`：**ES6 新增**。不存在变量提升（有暂时性死区 TDZ），不允许同一作用域内重复声明，属于块级作用域。用于声明可变变量。
*   `const`：**ES6 新增**。声明常量。一旦声明必须初始化，且基础数据类型的值不可改变。如果指向对象，对象的属性可以修改，但指针不可变。属于块级作用域。**优先使用 const**。

```javascript
var oldVar = 1;
let mutableVar = 2;
const immutableVar = 3; 
// immutableVar = 4; // 报错：Assignment to constant variable.
```

---

### 三、 数据类型

JS 是弱类型语言，变量没有类型，只有值有类型。分为**基本数据类型**和**引用数据类型**。

#### 1. 基本数据类型
保存在**栈内存**中，按值传递。
*   `Number`：数字（包含整数和浮点数，如 `10`, `3.14`，还有特殊值 `NaN`, `Infinity`）。
*   `String`：字符串。可用单引号 `''`、双引号 `""` 或反引号 `` ``（模板字符串，支持换行和插值 `${}`）。
*   `Boolean`：布尔值，只有 `true` 和 `false`。
*   `Null`：空值，表示“什么都没有”。（注意：`typeof null` 返回 `"object"`，这是历史遗留 Bug）。
*   `Undefined`：未定义，表示声明了但未赋值。
*   `Symbol`：**ES6 新增**，表示独一无二的值，常用于对象私有属性。
*   `BigInt`：**ES2020 新增**，用于表示任意精度的整数（如 `9007199254740991n`）。

#### 2. 引用数据类型
保存在**堆内存**中，按引用（内存地址）传递。统称为 `Object` 类型。
*   `Object`：普通对象 `{ key: value }`
*   `Array`：数组 `[1, 2, 3]`
*   `Function`：函数
*   `Date`、`RegExp`、`Map`、`Set` 等。

#### 3. 类型检测
*   `typeof x`：判断基本类型（`typeof "a" === "string"`）。
*   `x instanceof Y`：判断引用类型（`[] instanceof Array === true`）。
*   `Array.isArray(x)`：最准确的数组判断。
*   `Object.prototype.toString.call(x)`：最准确的类型判断方法。

---

### 四、 运算符

除了常规的算术（`+ - * / %`）和比较（`> < >= <=`）运算符外，需特别注意：

1.  **赋值**：`=`，`+=`，`-=` 等。
2.  **比较**：
    *   `==`（宽松相等，会隐式转换类型，**不推荐**）
    *   `===`（严格相等，不转换类型，**推荐**）
3.  **逻辑运算符**：`&&`（与）、`||`（或）、`!`（非）。具有**短路特性**，常用于设置默认值。
4.  **ES6+ 新增运算符**：
    *   **指数运算符**：`**`（如 `2 ** 3 === 8`）。
    *   **可选链操作符**：`?.`（安全访问深层属性，如 `user?.address?.city`，若 user 为 null/undefined 则不报错返回 undefined）。
    *   **空值合并运算符**：`??`（仅当左侧为 `null` 或 `undefined` 时返回右侧值，如 `a ?? "default"`。比 `||` 更严谨，因为 `0` 或 `""` 不会被忽略）。
    *   **扩展/展开运算符**：`...`（用于数组或对象的浅拷贝与合并）。

---

### 五、 控制流（条件与循环）

#### 1. 条件语句
*   `if...else if...else`
*   `switch...case`（注意配合 `break` 使用，支持穿透）。
*   **三元运算符**：`condition ? trueExpr : falseExpr`。

#### 2. 循环语句
*   `for`：标准循环 `for(let i=0; i<10; i++)`。
*   `while` / `do...while`：条件循环。
*   **`for...of`**：**ES6 新增**，用于遍历**可迭代对象**（Array, String, Map, Set），直接获取值。
*   **`for...in`**：用于遍历**对象的可枚举属性**（不建议遍历数组，因为会遍历原型链且顺序不保证）。
*   `break`（跳出循环）、`continue`（跳过本次循环）。

---

### 六、 函数

JS 中函数是“一等公民”，可以赋值给变量、作为参数传递、作为返回值返回。

#### 1. 函数声明
```javascript
function add(a, b) {
  return a + b;
}
```

#### 2. 函数表达式
```javascript
const add = function(a, b) {
  return a + b;
};
```

#### 3. 箭头函数 (ES6)
前面已经详细介绍过，语法极简，没有自己的 `this`。
```javascript
const add = (a, b) => a + b;
```

#### 4. 参数特性
*   **默认参数**：`function greet(name = "Guest") {}`
*   **剩余参数**：`function sum(...nums) {}`（替代以前的 `arguments` 对象）。

---

### 七、 对象与数组 (核心数据结构)

#### 1. 对象 语法
*   **字面量**：`const obj = { name: "Tom", age: 18 };`
*   **属性访问**：`obj.name` 或 `obj["name"]`。
*   **解构赋值 (ES6)**：`const { name, age } = obj;`
*   **属性简写**：如果键名和变量名相同，可简写 `const obj = { name, age };`
*   **方法简写**：`const obj = { sayHi() { console.log("hi"); } };`
*   **展开语法**：`const newObj = { ...obj, city: "NY" };`（浅拷贝与合并）。

#### 2. 数组 语法

*   **字面量**：`const arr = [1, 2, 3];`
*   **解构赋值**：`const [a, b] = arr;`
*   **常用方法**：
    *   增删：`push()`, `pop()`, `shift()`, `unshift()`, `splice()`。
    *   遍历：`forEach()`。
    *   **高阶函数（核心）**：`map()`（映射）、`filter()`（过滤）、`reduce()`（汇总）、`find()`（查找）、`some()`/`every()`（判断）。
    *   展开：`const newArr = [...arr, 4];`

---

### 八、 面向对象编程 (OOP)

JS 基于**原型**实现面向对象，ES6 引入了 `class` 关键字使其更接近传统 OOP 语言，但其底层本质仍是原型链。

#### 1. Class 语法
```javascript
class Person {
  // 构造函数
  constructor(name, age) {
    this.name = name; // 实例属性
    this.age = age;
  }

  // 实例方法
  sayHi() {
    console.log(`Hi, I'm ${this.name}`);
  }

  // 静态方法（属于类本身，不属于实例）
  static create(name) {
    return new Person(name, 18);
  }
}

const p = new Person("Alice", 20);
p.sayHi();
```

#### 2. 继承
使用 `extends` 关键字继承，子类构造函数中必须调用 `super()`。
```javascript
class Student extends Person {
  constructor(name, age, grade) {
    super(name, age); // 调用父类构造函数
    this.grade = grade;
  }
  
  // 重写父类方法
  sayHi() {
    super.sayHi(); // 调用父类方法
    console.log(`I'm in grade ${this.grade}`);
  }
}
```

---

### 九、 异步编程（JavaScript 的灵魂）

JS 是单线程的，依靠**事件循环**处理异步任务。

#### 1. 回调函数
最原始的方式，容易引发“回调地狱”。
```javascript
setTimeout(() => { console.log("1秒后执行"); }, 1000);
```

#### 2. Promise (ES6)
解决回调地狱，提供链式调用。状态：`pending` -> `fulfilled` / `rejected`。
```javascript
const promise = new Promise((resolve, reject) => {
  // 异步操作
  if (success) resolve("成功数据");
  else reject("失败原因");
});

promise
  .then(data => console.log(data))
  .catch(err => console.error(err))
  .finally(() => console.log("执行完毕"));
```

#### 3. Async / Await (ES2017)
Promise 的语法糖，让异步代码看起来像同步代码，极其强大。
```javascript
async function fetchData() {
  try {
    // await 会暂停函数执行，直到 Promise 决议
    const response = await fetch('https://api.example.com/data');
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error("请求出错:", error);
  }
}
```

---

### 十、 模块化

现代 JS 采用 ES6 模块化标准，取代了以前的 CommonJS (require) 等。

*   **导出**：
    *   命名导出：`export const a = 1; export function b() {}`
    *   默认导出（每个文件只能有一个）：`export default class {}`
*   **导入**：
    *   导入命名导出：`import { a, b } from './module.js';`
    *   导入默认导出：`import MyDefault from './module.js';`
    *   全部导入：`import * as myModule from './module.js';`

*(注意：在 Node.js 环境的 package.json 中需配置 `"type": "module"` 才能直接使用 ES Modules)*

---

### 十一、 错误处理

使用 `try...catch...finally` 捕获同步错误或 `async/await` 中的异步错误。
可以手动抛出错误：`throw new Error("自定义错误信息")`。

### 总结

JS 语法的特点是**极其灵活**，同一件事通常有多种写法（如函数声明、表达式、箭头函数）。掌握 ES6+ 语法（let/const, 箭头函数, 解构, Promise/Async, Class, 模块化）是现代前端开发的基础，因为主流框架全都基于这些现代语法构建。

## `=`/`==` /`===` 的区别

在 JavaScript 中，`=`、`==` 和 `===` 是三个完全不同性质的运算符。简单来说：一个是用来**赋值**的，另外两个是用来**比较**的。

下面是它们的详细区别：

---

### 1. `=` （赋值运算符）
*   **作用**：用于把右边的值赋给左边的变量。它**不是**用来比较相等的。
*   **返回值**：赋值操作本身会返回被赋的那个值。

**示例：**
```javascript
let a = 10;      // 把 10 赋值给变量 a
let b = a;       // 把变量 a 的值赋值给变量 b，此时 b 也为 10

// 连续赋值
let x, y;
x = y = 5;       // 先把 5 赋值给 y，然后 y 的值（5）再赋值给 x
```

> **⚠️ 初学者常见错误**：
> 在 `if` 语句中不小心把 `==` 写成了 `=`，会导致严重的逻辑错误。
> ```javascript
> let num = 1;
> if (num = 2) {  // 这里是赋值！num 变成了 2，且 2 是真值，所以条件永远成立
>  console.log("执行了"); // 一定会执行
> }
> ```

---

### 2. `==` （宽松相等运算符 / Loose Equality）
*   **作用**：用于比较两个值是否相等。
*   **核心特点**：**会进行类型转换（隐式转换）**。如果两个操作数的类型不同，JavaScript 会在比较之前尝试将它们转换为相同的类型（通常转换为数字），然后再比较。
*   **别名**：常被称为“双等号”。

**转换规则示例：**
```javascript
1 == 1            // true (类型相同，值相同)
1 == '1'          // true (字符串 '1' 被转换为数字 1，然后比较)
0 == false        // true (布尔值 false 被转换为数字 0，然后比较)
0 == ''           // true (空字符串被转换为数字 0，然后比较)
null == undefined // true (这是一个特殊规则，null 和 undefined 宽松相等)

// 也有不相等的时候
NaN == NaN        // false (NaN 和任何值都不相等，包括它自己)
null == 0         // false (null 在 == 时只会被转换为 undefined，不会转为 0)
```

> **缺点**：因为类型转换规则非常复杂且反直觉，容易引发难以排查的 Bug。

---

### 3. `===` （严格相等运算符 / Strict Equality）
*   **作用**：用于比较两个值是否严格相等。
*   **核心特点**：**不会进行类型转换**。它要求两个操作数不仅**值要相同**，**数据类型也必须完全相同**，才会返回 `true`。
*   **别名**：常被称为“三等号”。

**示例：**
```javascript
1 === 1            // true (类型相同，值相同)
1 === '1'          // false (类型不同，一个是 Number，一个是 String)
0 === false        // false (类型不同，一个是 Number，一个是 Boolean)
null === undefined // false (类型不同，一个是 Null，一个是 Undefined)
NaN === NaN        // false (NaN 与任何值都不等)

// 引用类型比较（比较的是内存地址）
[] === []          // false (虽然都是空数组，但它们在内存中是两个不同的对象)
{} === {}          // false (同理)
let a = {}, b = a;
a === b            // true (a 和 b 指向内存中的同一个对象)
```

---

### 总结与最佳实践

| 运算符 | 名称       | 作用                     | 是否进行类型转换 | 使用场景                                 |
| :----- | :--------- | :----------------------- | :--------------- | :--------------------------------------- |
| `=`    | 赋值运算符 | 给变量赋值               | 无               | 声明变量或修改变量值时使用               |
| `==`   | 宽松相等   | 比较值是否相等           | **是**           | **不推荐使用**（除非你明确需要类型转换） |
| `===`  | 严格相等   | 比较值和类型是否完全相同 | **否**           | **强烈推荐**，在所有比较场景都应使用     |

#### 💡 面试/开发铁律：
在绝大多数现代 JavaScript 项目（以及 ESLint 规范）中，**永远使用 `===`，禁用 `==`**。
使用 `===` 可以让你的代码意图更明确，避免因为 JavaScript 离奇的隐式类型转换而导致未知的 Bug。

如果确实需要判断 `null` 或 `undefined`，可以使用 `===` 分别判断：
```javascript
// 不推荐
if (value == null) { ... } // 实际上等于 value === null || value === undefined

// 推荐写法，意图清晰
if (value === null || value === undefined) { ... }
```

# JS 函数

## 一、函数定义方式

### 1. **函数声明（Function Declaration）**

```javascript
// 基本语法
function greet(name) {
  return `Hello, ${name}!`;
}

// 调用
const message = greet('张三');
console.log(message);  // 'Hello, 张三!'

// 特点：函数提升（hoisting）
sayHello();  // ✅ 可以在声明前调用

function sayHello() {
  console.log('Hello!');
}
```

### 2. **函数表达式（Function Expression）**

```javascript
// 基本语法
const greet = function(name) {
  return `Hello, ${name}!`;
};

// 调用
const message = greet('张三');
console.log(message);

// 特点：不会提升
// sayHello();  // ❌ 报错：Cannot access 'sayHello' before initialization

const sayHello = function() {
  console.log('Hello!');
};

// 命名函数表达式
const factorial = function fact(n) {
  if (n <= 1) return 1;
  return n * fact(n - 1);
};

// fact(5);  // ❌ 外部无法访问 fact
factorial(5);  // ✅ 120
```

### 3. **箭头函数（Arrow Function）**

```javascript
// ES6 引入
const greet = (name) => {
  return `Hello, ${name}!`;
};

// 简写（单参数可省略括号）
const double = x => x * 2;

// 简写（单行返回可省略 return）
const add = (a, b) => a + b;

// 无参数
const sayHello = () => console.log('Hello!');

// 返回对象（需要括号）
const createUser = (name, age) => ({ name, age });

// 特点：没有自己的 this
```

箭头函数是 ES6（ECMAScript 2015）引入的一种全新的语法，它提供了一种更简洁的方式来编写函数表达式。除了语法上的简化，它在 `this` 的绑定机制上也有根本性的改变。

下面从**基本语法**、**核心特性**、**实际应用**和**不适用场景**四个维度进行详细介绍。

---

#### 一、 基本语法

箭头函数使用 `=>`（胖箭头）来定义。

##### 1. 标准写法

```javascript
// 普通函数
const add = function(a, b) {
  return a + b;
};

// 箭头函数
const add = (a, b) => {
  return a + b;
};
```

##### 2. 语法简写

*   **单个参数**：如果只有一个参数，可以省略括号 `()`。
*   **无参数**：如果没有参数，必须使用空括号 `()`。
*   **单行函数体**：如果函数体只有一句返回语句，可以省略大括号 `{}` 和 `return` 关键字（隐式返回）。

```javascript
// 单个参数 + 单行返回
const square = x => x * x; 

// 无参数 + 单行返回
const sayHello = () => "Hello!";

// 多个参数 + 单行返回
const add = (a, b) => a + b;

// 返回对象字面量（注意：必须用括号包裹，否则大括号会被解析为函数体）
const createUser = (name, age) => ({ name: name, age: age });
```

---

#### 二、 核心特性（与普通函数的区别）

箭头函数不仅仅是语法糖，它有几个非常关键的内部行为差异。

##### 1. 没有自己的 `this`（词法 `this`）

这是箭头函数最重要的特性。普通函数的 `this` 是在**被调用时**动态决定的（取决于调用方式）。而箭头函数**没有自己的 `this`**，它的 `this` 是在**定义时**继承自外层执行上下文（即“词法作用域”）的 `this`。

```javascript
function Timer() {
  this.s1 = 0;
  this.s2 = 0;

  // 普通函数：this 指向调用它的对象（window/setInterval的上下文），导致 this.s1 变成 NaN 或全局变量
  setInterval(function() {
    this.s1++; // 这里的 this 不指向 Timer 实例
    console.log("普通函数 this.s1:", this.s1);
  }, 1000);

  // 箭头函数：this 继承自外层的 Timer 函数，指向 Timer 实例
  setInterval(() => {
    this.s2++; // 这里的 this 正确指向 Timer 实例
    console.log("箭头函数 this.s2:", this.s2);
  }, 1000);
}

const timer = new Timer();
```

##### 2. 没有 `arguments` 对象

箭头函数内部不存在 `arguments` 对象。如果你在箭头函数中访问 `arguments`，它会获取到外层普通函数的 `arguments`。如果需要获取不定参数，请使用 **剩余参数（...rest）**。

```javascript
const showArgs = () => {
  console.log(arguments); // 报错或获取到外层的 arguments
};

// 正确做法：使用剩余参数
const showRest = (...args) => {
  console.log(args); // [1, 2, 3]
};
showRest(1, 2, 3);
```

##### 3. 不能作为构造函数（不能使用 `new`）

箭头函数没有 `[[Construct]]` 内部方法，因此不能使用 `new` 关键字调用，否则会报错。它也没有 `prototype` 属性。

```javascript
const Person = (name) => {
  this.name = name;
};

const p = new Person("Alice"); // TypeError: Person is not a constructor
```

##### 4. 不能用作生成器

箭头函数不能使用 `yield` 关键字，因此不能作为 Generator 函数。

##### 5. `call`、`apply`、`bind` 无法改变其 `this`

由于箭头函数没有自己的 `this`，所以使用 `call()`、`apply()` 或 `bind()` 方法去修改箭头函数的 `this` 是无效的。它们只能传递参数，无法改变 `this` 指向。

```javascript
const arrowFunc = () => {
  console.log(this);
};

const obj = { name: "Object" };

// this 依然是定义时的外层 this，不会被改为 obj
arrowFunc.call(obj); 
```

---

#### 三、 最佳应用场景

##### 1. 回调函数（尤其是数组方法）

箭头函数让 `map`、`filter`、`find` 、`reduce` 等数组方法的代码变得非常简洁。

下面通过具体的对比示例，展示箭头函数如何让 `map`、`filter`、`reduce` 和 `find` 等数组方法的代码变得极其简洁。

假设我们有一个用户列表数据：
```javascript
const users = [
  { id: 1, name: 'Alice', age: 25, active: true },
  { id: 2, name: 'Bob', age: 17, active: false },
  { id: 3, name: 'Charlie', age: 30, active: true },
  { id: 4, name: 'David', age: 16, active: true }
];
```

---

###### 1. `map` (映射：对每个元素进行处理，返回新数组)

**需求**：提取所有用户的名字，生成一个名字数组。

```javascript
// 👎 普通函数写法：显得臃肿
const names1 = users.map(function(user) {
  return user.name;
});

// 👍 箭头函数写法：单参数省略括号，单行省略 return 和大括号
const names2 = users.map(user => user.name);

console.log(names2); // ['Alice', 'Bob', 'Charlie', 'David']
```

---

###### 2. `filter` (过滤：保留满足条件的元素，返回新数组)

**需求**：筛选出所有已成年（age >= 18）的用户。

```javascript
// 👎 普通函数写法
const adults1 = users.filter(function(user) {
  return user.age >= 18;
});

// 👍 箭头函数写法
const adults2 = users.filter(user => user.age >= 18);

console.log(adults2.length); // 2 (Alice 和 Charlie)
```

---

###### 3. `find` (查找：返回第一个满足条件的元素)

**需求**：查找名字为 'Charlie' 的用户信息。

```javascript
// 👎 普通函数写法
const charlie1 = users.find(function(user) {
  return user.name === 'Charlie';
});

// 👍 箭头函数写法
const charlie2 = users.find(user => user.name === 'Charlie');

console.log(charlie2); // { id: 3, name: 'Charlie', age: 30, active: true }
```

---

###### 4. `reduce` (汇总：将数组元素汇总为一个单一值)

**需求**：计算所有用户的年龄总和。
*注意：`reduce` 的回调函数接收两个参数：累计器和当前元素。*

```javascript
// 👎 普通函数写法
const totalAge1 = users.reduce(function(sum, user) {
  return sum + user.age;
}, 0); // 0 是 sum 的初始值

// 👍 箭头函数写法：多参数必须带括号 (sum, user)
const totalAge2 = users.reduce((sum, user) => sum + user.age, 0);

console.log(totalAge2); // 88 (25 + 17 + 30 + 16)
```

---

###### 🌟 终极组合：链式调用

在实际开发中，经常需要组合使用这些方法。箭头函数的优势在链式调用时会成倍放大。

**需求**：找出所有**活跃的成年用户**，并**提取他们的名字**，最后**拼接成一个逗号分隔的字符串**。

```javascript
// 👎 如果用普通函数，嵌套和 return 会让人眼花缭乱
const result1 = users
  .filter(function(user) {
    return user.active && user.age >= 18;
  })
  .map(function(user) {
    return user.name;
  })
  .join(', ');

// 👍 使用箭头函数，代码就像一行话一样清晰流畅
const result2 = users
  .filter(user => user.active && user.age >= 18)
  .map(user => user.name)
  .join(', ');

console.log(result2); // "Alice, Charlie"
```

**总结**：在数组方法中，回调函数通常不需要自己的 `this`（不需要操作外部对象），且逻辑往往比较简短。这正是箭头函数最完美的应用场景。



##### 2. 需要保留外层 `this` 的场景（如事件监听、定时器）

在 Vue/React 等现代框架中，或者原生 DOM 事件绑定中，箭头函数可以避免使用 `const that = this;` 或 `.bind(this)` 的 hack 写法。
```javascript
class Counter {
  constructor() {
    this.count = 0;
  }
  
  start() {
    // 箭头函数保证 this 指向 Counter 实例
    setInterval(() => {
      this.count++;
      console.log(this.count);
    }, 1000);
  }
}
```

##### 3. Promise 链式调用

```javascript
fetchData()
  .then(data => process(data))
  .then(result => display(result))
  .catch(err => console.error(err));
```

---

#### 四、 不适用场景

##### 1. 对象的方法

如果对象的方法内部需要引用对象自身的属性，不要使用箭头函数，因为它的 `this` 会指向对象所在的外层作用域（通常是 `window` 或 `undefined`），而不是对象本身。
```javascript
const obj = {
  name: "Alice",
  // 错误：箭头函数的 this 指向外层，不指向 obj
  sayHi: () => {
    console.log(this.name); // undefined
  },
  // 正确：普通函数，this 指向 obj
  sayHello() {
    console.log(this.name); // "Alice"
  }
};
```
*注：在 Vue 2 中定义 methods 时也是同理，不能使用箭头函数。*

##### 2. 原型链上的方法

同上，原型方法如果用箭头函数，内部的 `this` 无法指向实例对象。
```javascript
function Person(name) {
  this.name = name;
}

// 错误做法
Person.prototype.sayName = () => {
  console.log(this.name); // this 不指向实例
};

// 正确做法
Person.prototype.sayName = function() {
  console.log(this.name);
};
```

##### 3. DOM 事件监听器

当你需要通过 `this` 来获取触发事件的 DOM 元素时，不能用箭头函数。
```javascript
const button = document.querySelector('button');

// 错误：this 指向外层 window，不指向 button
button.addEventListener('click', () => {
  this.classList.toggle('active'); // 报错
});

// 正确：this 指向 button
button.addEventListener('click', function() {
  this.classList.toggle('active');
});
```

#### 总结

*   **本质**：更简洁的函数表达式，没有自己的 `this`、`arguments`、`prototype`，不能 `new`。
*   **用在哪**：短小的回调函数、需要继承外层 `this` 的地方、函数式编程数组操作。
*   **不用在哪**：对象方法、原型方法、需要自身 `this` 的 DOM 事件、构造函数。



### 4. **对比表**

| 特性          | 函数声明          | 函数表达式              | 箭头函数           |
| ------------- | ----------------- | ----------------------- | ------------------ |
| **提升**      | ✅ 是              | ❌ 否                    | ❌ 否               |
| **this 绑定** | ✅ 动态            | ✅ 动态                  | ❌ 词法             |
| **arguments** | ✅ 有              | ✅ 有                    | ❌ 无               |
| **new 调用**  | ✅ 可以            | ✅ 可以                  | ❌ 不可以           |
| **语法**      | `function name()` | `const fn = function()` | `const fn = () =>` |

---

## 二、函数参数

### 1. **基本参数**

```javascript
// 单个参数
function square(x) {
  return x * x;
}

// 多个参数
function add(a, b) {
  return a + b;
}

// 调用
square(5);  // 25
add(1, 2);  // 3
```

### 2. **默认参数**

```javascript
// ES5 方式
function greet(name) {
  name = name || '匿名用户';
  console.log(`Hello, ${name}`);
}

// ES6 默认参数
function greet(name = '匿名用户') {
  console.log(`Hello, ${name}`);
}

greet();  // 'Hello, 匿名用户'
greet('张三');  // 'Hello, 张三'

// 多个默认参数
function createUser(name = '张三', age = 25, gender = '男') {
  return { name, age, gender };
}

createUser();  // { name: '张三', age: 25, gender: '男' }
createUser('李四');  // { name: '李四', age: 25, gender: '男' }
createUser('李四', 30);  // { name: '李四', age: 30, gender: '男' }

// 默认参数表达式
function fetchData(url, options = getDefaultOptions()) {
  // ...
}

// 默认参数可以使用前面的参数
function multiply(a, b = a * 2) {
  return a * b;
}

multiply(5);  // 50
```

### 3. **剩余参数（Rest Parameters）**

```javascript
// 收集剩余参数为数组
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

sum(1, 2, 3, 4, 5);  // 15

// 结合普通参数
function greet(greeting, ...names) {
  return `${greeting}, ${names.join(' and ')}!`;
}

greet('Hello', '张三', '李四', '王五');
// 'Hello, 张三 and 李四 and 王五!'

// 箭头函数
const sum = (...numbers) => numbers.reduce((total, num) => total + num, 0);
```

### 4. **解构参数**

```javascript
// 对象解构
function createUser({ name, age, gender = '男' }) {
  return { name, age, gender };
}

createUser({ name: '张三', age: 25 });
// { name: '张三', age: 25, gender: '男' }

// 默认值
function createUser({ name = '张三', age = 0 } = {}) {
  return { name, age };
}

createUser();  // { name: '张三', age: 0 }

// 数组解构
function getFirstAndSecond([first, second]) {
  return { first, second };
}

getFirstAndSecond([1, 2, 3, 4]);  // { first: 1, second: 2 }
```

---

## 三、this 指向

### 1. **全局环境**

```javascript
// 浏览器环境
console.log(this === window);  // true

// Node.js 环境
console.log(this === global);  // false
console.log(this === globalThis);  // true
```

### 2. **对象方法**

```javascript
const user = {
  name: '张三',
  sayName: function() {
    console.log(this.name);  // '张三'
  }
};

user.sayName();  // this 指向 user

// 箭头函数不行
const user2 = {
  name: '李四',
  sayName: () => {
    console.log(this.name);  // undefined（this 指向全局）
  }
};

user2.sayName();
```

### 3. **构造函数**

```javascript
function Person(name) {
  this.name = name;
  console.log(this);  // 指向新创建的实例
}

const person = new Person('张三');
console.log(person.name);  // '张三'
```

### 4. **事件处理**

```javascript
// 传统函数
button.addEventListener('click', function() {
  console.log(this);  // 指向 button 元素
});

// 箭头函数
button.addEventListener('click', () => {
  console.log(this);  // 指向定义时的上下文
});
```

### 5. **改变 this 指向**

```javascript
// call - 立即调用
function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const user = { name: '张三' };

greet.call(user, 'Hello', '!');  // 'Hello, 张三!'
greet.call(user, 'Hi', '.');     // 'Hi, 张三.'

// apply - 参数为数组
greet.apply(user, ['Hello', '!']);

// bind - 返回新函数，不立即调用
const boundGreet = greet.bind(user);
boundGreet('Hello', '!');  // 'Hello, 张三!'

// 实际应用：保存 this
const obj = {
  name: '张三',
  init: function() {
    const self = this;  // 保存 this
    
    setTimeout(function() {
      console.log(self.name);  // '张三'
    }, 1000);
    
    // 或使用 bind
    setTimeout(function() {
      console.log(this.name);  // '张三'
    }.bind(this), 1000);
    
    // 或使用箭头函数（推荐）
    setTimeout(() => {
      console.log(this.name);  // '张三'
    }, 1000);
  }
};
```

---

## 四、闭包（Closure）

### 1. **基本概念**

```javascript
function outer() {
  const outerVar = '我是外部变量';
  
  function inner() {
    console.log(outerVar);  // 可以访问外部函数的变量
  }
  
  return inner;
}

const closure = outer();
closure();  // '我是外部变量'
```

### 2. **实际应用**

```javascript
// 1. 数据封装
function createCounter() {
  let count = 0;
  
  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count
  };
}

const counter = createCounter();
counter.increment();  // 1
counter.increment();  // 2
counter.decrement();  // 1
counter.getCount();   // 1

// 2. 函数工厂
function multiplyBy(factor) {
  return function(number) {
    return number * factor;
  };
}

const double = multiplyBy(2);
const triple = multiplyBy(3);

double(5);  // 10
triple(5);  // 15

// 3. 防抖函数
function debounce(fn, delay) {
  let timer = null;
  
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}

// 使用
const handleSearch = debounce((keyword) => {
  console.log('搜索:', keyword);
}, 300);

input.addEventListener('input', (e) => {
  handleSearch(e.target.value);
});

// 4. 节流函数
function throttle(fn, delay) {
  let lastTime = 0;
  
  return function(...args) {
    const now = Date.now();
    if (now - lastTime >= delay) {
      fn.apply(this, args);
      lastTime = now;
    }
  };
}

// 使用
const handleScroll = throttle(() => {
  console.log('滚动:', window.scrollY);
}, 1000);

window.addEventListener('scroll', handleScroll);
```

### 3. **经典问题**

```javascript
// ❌ 错误：使用 var
for (var i = 1; i <= 3; i++) {
  setTimeout(function() {
    console.log(i);  // 4, 4, 4
  }, 1000);
}

// ✅ 解决 1：使用 let
for (let i = 1; i <= 3; i++) {
  setTimeout(function() {
    console.log(i);  // 1, 2, 3
  }, 1000);
}

// ✅ 解决 2：使用闭包
for (var i = 1; i <= 3; i++) {
  (function(j) {
    setTimeout(function() {
      console.log(j);  // 1, 2, 3
    }, 1000);
  })(i);
}
```

---

## 五、高阶函数

### 1. **定义**

```javascript
// 函数作为参数
function forEach(arr, fn) {
  for (let i = 0; i < arr.length; i++) {
    fn(arr[i], i, arr);
  }
}

forEach([1, 2, 3], (item) => console.log(item));

// 函数作为返回值
function once(fn) {
  let called = false;
  
  return function(...args) {
    if (!called) {
      called = true;
      return fn.apply(this, args);
    }
  };
}

const init = once(() => {
  console.log('初始化');
});

init();  // '初始化'
init();  // 无输出
```

### 2. **数组方法（高阶函数）**

```javascript
const numbers = [1, 2, 3, 4, 5];

// map - 映射
const doubled = numbers.map(x => x * 2);
// [2, 4, 6, 8, 10]

// filter - 过滤
const evens = numbers.filter(x => x % 2 === 0);
// [2, 4]

// reduce - 归并
const sum = numbers.reduce((total, num) => total + num, 0);
// 15

// forEach - 遍历
numbers.forEach(x => console.log(x));

// some - 是否有满足条件的
const hasEven = numbers.some(x => x % 2 === 0);  // true

// every - 是否都满足条件
const allPositive = numbers.every(x => x > 0);  // true

// find - 查找第一个
const first = numbers.find(x => x > 3);  // 4

// findIndex - 查找索引
const index = numbers.findIndex(x => x > 3);  // 3

// sort - 排序
const sorted = [...numbers].sort((a, b) => b - a);
// [5, 4, 3, 2, 1]

// 链式调用
const result = numbers
  .filter(x => x % 2 !== 0)      // [1, 3, 5]
  .map(x => x * 2)               // [2, 6, 10]
  .reduce((total, num) => total + num, 0);  // 18
```

### 3. **函数组合**

```javascript
// compose - 从右到左
function compose(...fns) {
  return function(x) {
    return fns.reduceRight((acc, fn) => fn(acc), x);
  };
}

// pipe - 从左到右
function pipe(...fns) {
  return function(x) {
    return fns.reduce((acc, fn) => fn(acc), x);
  };
}

// 使用
const double = x => x * 2;
const addTen = x => x + 10;
const toString = x => `Result: ${x}`;

const transform = pipe(double, addTen, toString);
transform(5);  // 'Result: 20'
```

---

## 六、递归函数

### 1. **基本递归**

```javascript
// 阶乘
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}

factorial(5);  // 120

// 斐波那契数列
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

fibonacci(10);  // 55
```

### 2. **尾递归优化**

```javascript
// 普通递归（会栈溢出）
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}

// 尾递归
function factorial(n, acc = 1) {
  if (n <= 1) return acc;
  return factorial(n - 1, n * acc);
}

factorial(5);  // 120

// 斐波那契尾递归
function fibonacci(n, a = 0, b = 1) {
  if (n === 0) return a;
  if (n === 1) return b;
  return fibonacci(n - 1, b, a + b);
}

fibonacci(10);  // 55
```

### 3. **实际应用**

```javascript
// 遍历树形结构
function findNode(tree, id) {
  if (tree.id === id) return tree;
  
  if (tree.children) {
    for (const child of tree.children) {
      const found = findNode(child, id);
      if (found) return found;
    }
  }
  
  return null;
}

// 深拷贝
function deepClone(obj, map = new WeakMap()) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (map.has(obj)) return map.get(obj);
  
  const clone = Array.isArray(obj) ? [] : {};
  map.set(obj, clone);
  
  for (const key in obj) {
    clone[key] = deepClone(obj[key], map);
  }
  
  return clone;
}
```

---

## 七、Generator 函数

### 1. **基本用法**

```javascript
// 定义
function* generator() {
  yield 1;
  yield 2;
  yield 3;
}

// 使用
const gen = generator();

console.log(gen.next());  // { value: 1, done: false }
console.log(gen.next());  // { value: 2, done: false }
console.log(gen.next());  // { value: 3, done: false }
console.log(gen.next());  // { value: undefined, done: true }

// for...of
for (const value of generator()) {
  console.log(value);  // 1, 2, 3
}
```

### 2. **传值**

```javascript
function* counter() {
  let i = 0;
  while (true) {
    const reset = yield i++;
    if (reset) i = 0;
  }
}

const gen = counter();
console.log(gen.next());       // { value: 0, done: false }
console.log(gen.next());       // { value: 1, done: false }
console.log(gen.next(true));   // { value: 0, done: false }
```

### 3. **实际应用**

```javascript
// 异步流程控制
function* asyncFlow() {
  const user = yield fetchUser(1);
  const posts = yield fetchPosts(user.id);
  return posts;
}

// 自动执行器
function run(generatorFn) {
  const gen = generatorFn();
  
  function next(value) {
    const result = gen.next(value);
    
    if (result.done) return result.value;
    
    Promise.resolve(result.value).then(next);
  }
  
  next();
}

run(asyncFlow);
```

---

## 八、Async/Await 函数

### 1. **基本用法**

```javascript
// 定义 async 函数
async function fetchData() {
  return '数据';
}

// 返回 Promise
fetchData().then(data => console.log(data));  // '数据'

// await 必须在 async 函数中使用
async function getData() {
  const response = await fetch('/api/data');
  const data = await response.json();
  return data;
}
```

### 2. **错误处理**

```javascript
// try...catch
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('请求失败:', error.message);
    throw error;
  }
}

// 多个异步操作
async function getAllData() {
  try {
    const [users, posts, comments] = await Promise.all([
      fetch('/api/users').then(r => r.json()),
      fetch('/api/posts').then(r => r.json()),
      fetch('/api/comments').then(r => r.json())
    ]);
    
    return { users, posts, comments };
  } catch (error) {
    console.error('获取数据失败:', error);
  }
}
```

### 3. **你的项目中的用法**

```javascript
// pmsLineBaseDataQuery.vue
async searchTableAsync() {
  try {
    const params = {
      ...this.cLform,
      ...this.page
    };
    
    const res = await queryLineBaseDataList(params);
    
    if (res.data && res.data.success) {
      this.tableData.list = res.data.result?.result || [];
      this.tableData.total = res.data.result?.count || 0;
    } else {
      this.$message.warning(res.data?.resultMessage || '查询失败');
    }
  } catch (error) {
    console.error('查询失败：', error);
    this.$message.error('查询失败');
  } finally {
    this.tableLoading = false;
  }
}
```

---

## 九、函数属性与方法

### 1. **name 属性**

```javascript
function greet() {}
console.log(greet.name);  // 'greet'

const anonymous = function() {};
console.log(anonymous.name);  // 'anonymous'

const arrow = () => {};
console.log(arrow.name);  // 'arrow'
```

### 2. **length 属性**

```javascript
function add(a, b, c) {}
console.log(add.length);  // 3

const sum = (a, b) => a + b;
console.log(sum.length);  // 2
```

### 3. **call / apply / bind**

```javascript
function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const user = { name: '张三' };

// call - 立即调用，参数列表
greet.call(user, 'Hello', '!');

// apply - 立即调用，参数数组
greet.apply(user, ['Hello', '!']);

// bind - 返回新函数
const boundGreet = greet.bind(user);
boundGreet('Hello', '!');
```

---

## 十、IIFE（立即执行函数）

```javascript
// 基本语法
(function() {
  console.log('立即执行');
})();

// 箭头函数
(() => {
  console.log('立即执行');
})();

// 传参
(function(name) {
  console.log(`Hello, ${name}`);
})('张三');

// 用途：创建私有作用域
const counter = (function() {
  let count = 0;
  
  return {
    increment: () => ++count,
    getCount: () => count
  };
})();

counter.increment();  // 1
counter.getCount();   // 1
```

---

## 十一、柯里化（Currying）

```javascript
// 基本柯里化
function add(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}

add(1)(2)(3);  // 6

// 自动柯里化
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function(...moreArgs) {
      return curried.apply(this, args.concat(moreArgs));
    };
  };
}

// 使用
function add(a, b, c) {
  return a + b + c;
}

const curriedAdd = curry(add);
curriedAdd(1)(2)(3);  // 6
curriedAdd(1, 2)(3);  // 6
curriedAdd(1)(2, 3);  // 6

// 实际应用
function format(template, ...values) {
  return template.replace(/\{(\d+)\}/g, (match, index) => values[index]);
}

const formatWithTemplate = curry(format);
const greet = formatWithTemplate('Hello, {0}! You are {1} years old.');

greet('张三', 25);  // 'Hello, 张三! You are 25 years old.'
```

---

## 十二、函数最佳实践

### ✅ 推荐做法：

```javascript
// 1. 使用箭头函数（不需要 this 时）
const add = (a, b) => a + b;

// 2. 使用默认参数
function greet(name = '匿名用户') {
  console.log(`Hello, ${name}`);
}

// 3. 使用剩余参数
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

// 4. 使用解构参数
function createUser({ name, age }) {
  return { name, age };
}

// 5. 纯函数（相同输入 = 相同输出）
function calculatePrice(price, tax) {
  return price * (1 + tax);
}

// 6. 函数单一职责
function validateEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

function sendEmail(email, content) {
  // 发送邮件
}

// 7. async/await 处理异步
async function fetchData() {
  try {
    const res = await axios.get('/api/data');
    return res.data;
  } catch (error) {
    console.error(error);
    throw error;
  }
}
```

### ❌ 避免做法：

```javascript
// ❌ 不要修改参数
function badFn(obj) {
  obj.name = 'modified';  // 副作用
  return obj;
}

// ✅ 应该返回新对象
function goodFn(obj) {
  return { ...obj, name: 'modified' };
}

// ❌ 不要使用 arguments
function sum() {
  return Array.from(arguments).reduce((a, b) => a + b, 0);
}

// ✅ 使用剩余参数
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

// ❌ 不要创建不必要的闭包
for (var i = 0; i < 5; i++) {
  (function(j) {
    setTimeout(() => console.log(j), 1000);
  })(i);
}

// ✅ 使用 let
for (let i = 0; i < 5; i++) {
  setTimeout(() => console.log(i), 1000);
}
```

---

## 总结

### JavaScript 函数核心要点：

```javascript
// 1. 定义方式：函数声明、函数表达式、箭头函数
// 2. 参数：默认参数、剩余参数、解构参数
// 3. this：动态绑定 vs 词法绑定
// 4. 闭包：访问外部作用域
// 5. 高阶函数：函数作为参数或返回值
// 6. 递归：函数调用自身
// 7. Generator：可暂停和恢复
// 8. Async/Await：异步编程
// 9. 柯里化：参数分步传递
// 10. 纯函数：无副作用
```

**函数是 JavaScript 的一等公民！** 🚀





# JS 分类

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