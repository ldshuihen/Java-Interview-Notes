HTML（**HTML**）学习如果按结构系统化分类，会更容易理解。一般可以从 **6 个层次**来学习：基础结构 → 标签 → 表单 → 媒体 → 语义化 → HTML5 API。

下面给你一套 **面试级 + 完整学习分类体系**。

------

# HTML

## 一、HTML 基础结构

HTML 文档的基本骨架。

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>My Page</title>
</head>
<body>

Hello World

</body>
</html>
```

核心结构：

| 标签              | 作用       |
| ----------------- | ---------- |
| `<!DOCTYPE html>` | 声明 HTML5 |
| `<html>`          | 根标签     |
| `<head>`          | 页面元数据 |
| `<title>`         | 页面标题   |
| `<meta>`          | 页面信息   |
| `<body>`          | 页面内容   |

------

## 二、HTML 内容标签（最基础）

用于展示文本内容。

### 1 标题

```html
<h1>标题1</h1>
<h2>标题2</h2>
<h3>标题3</h3>
```

------

### 2 段落

```html
<p>这是一个段落</p>
```

------

### 3 文本格式

```html
<b>加粗</b>
<i>斜体</i>
<u>下划线</u>
<strong>强调</strong>
<em>强调</em>
```

------

### 4 换行

```html
<br>
<hr>
```

------

## 三、HTML 结构布局标签

用于页面结构。

### 1 div

```html
<div>容器</div>
```

最常见布局元素。

------

### 2 span

```html
<span>行内元素</span>
```

用于包裹文本。

------

### 3 列表

#### 无序列表

```html
<ul>
  <li>苹果</li>
  <li>香蕉</li>
</ul>
```

#### 有序列表

```html
<ol>
  <li>第一步</li>
  <li>第二步</li>
</ol>
```

------

## 四、HTML 超链接与资源

### 1 超链接

```html
<a href="https://google.com">访问</a>
```

------

### 2 图片

```html
<img src="img.jpg" alt="图片">
```

------

### 3 下载

```html
<a href="file.zip" download>下载</a>
```

------

## 五、HTML 表格

```html
<table border="1">
<tr>
    <th>姓名</th>
    <th>年龄</th>
</tr>
<tr>
    <td>张三</td>
    <td>18</td>
</tr>
</table>
```

核心标签：

| 标签  | 作用   |
| ----- | ------ |
| table | 表格   |
| tr    | 行     |
| td    | 单元格 |
| th    | 表头   |

------

## 六、HTML 表单（前端交互核心）

HTML 表单用于提交数据到后端（如 **Spring Boot**）。

### 1 form

```html
<form action="/login" method="post">
</form>
```

------

### 2 input

```html
<input type="text">
<input type="password">
<input type="checkbox">
<input type="radio">
<input type="file">
```

------

### 3 下拉框

```html
<select>
<option>中国</option>
<option>美国</option>
</select>
```

------

### 4 文本域

```html
<textarea></textarea>
```

------

## 七、HTML5 语义化标签

HTML5 增加了结构语义。

| 标签    | 含义   |
| ------- | ------ |
| header  | 页头   |
| nav     | 导航   |
| main    | 主内容 |
| section | 内容区 |
| article | 文章   |
| aside   | 侧边栏 |
| footer  | 页脚   |

示例：

```html
<header>网站头</header>
<nav>导航</nav>
<main>
   <article>文章</article>
</main>
<footer>底部</footer>
```

好处：

- SEO友好
- 结构清晰
- 可读性高

------

## 八、HTML5 多媒体

### 视频

```html
<video controls>
 <source src="a.mp4">
</video>
```

------

### 音频

```html
<audio controls>
 <source src="a.mp3">
</audio>
```

------

## 九、HTML5 图形

### Canvas

```html
<canvas id="c"></canvas>
```

配合 **JavaScript** 绘图。

------

### SVG

```html
<svg width="100" height="100">
  <circle cx="50" cy="50" r="40"/>
</svg>
```

------

## 十、HTML5 Web API

HTML5 与 JS 结合能力。

| API            | 作用     |
| -------------- | -------- |
| LocalStorage   | 本地存储 |
| SessionStorage | 会话存储 |
| Geolocation    | 地理位置 |
| WebSocket      | 实时通信 |
| Drag&Drop      | 拖拽     |
| History API    | 路由控制 |

------

## 十一、HTML 在前端体系中的位置

完整前端结构：

```
前端技术体系
│
├── HTML
│   ├── 页面结构
│   ├── 标签
│   ├── 表单
│   └── 语义化
│
├── CSS
│   ├── 样式
│   ├── 布局
│   └── 动画
│
├── JavaScript
│   ├── DOM操作
│   ├── 事件
│   ├── AJAX
│   └── 网络请求
│
└── 框架
    ├── :contentReference[oaicite:3]{index=3}
    ├── :contentReference[oaicite:4]{index=4}
    └── :contentReference[oaicite:5]{index=5}
```

------

## 十二、HTML 学习路线（推荐）

**第一阶段**

```
HTML结构
文本标签
列表
图片
链接
```

**第二阶段**

```
表格
表单
HTML5语义化
```

**第三阶段**

```
音视频
Canvas
HTML5 API
```

**第四阶段**

```
DOM
JavaScript
Vue
```

------

如果你愿意，我可以继续给你整理一份 **《HTML 全部标签分类总表（面试级）》**，把 **100+ HTML 标签一次性系统分类出来**，这在学习 **Vue 3** 或前端面试时非常有用。

# HTML应用场景

HTML 的学习如果只看标签会比较抽象。**最好的方式是按“应用场景”分类**，每个场景给出一段 **典型代码**。这样在学习 **HTML** 时就会非常清晰。

下面给出 **10类常见 HTML 应用场景 + 典型代码**（接近真实项目）。

------

## 一、页面结构布局（最常见）

用于构建网页整体结构。

典型场景：

- 官网首页
- 后台管理系统
- 博客页面

```html
<header>
  <h1>我的网站</h1>
</header>

<nav>
  <a href="#">首页</a>
  <a href="#">文章</a>
  <a href="#">关于</a>
</nav>

<main>
  <section>
    <h2>最新文章</h2>
    <p>这里是文章摘要</p>
  </section>
</main>

<footer>
  <p>© 2026 MySite</p>
</footer>
```

特点：

- 语义化结构
- SEO友好
- 适合与 **JavaScript** 和 **CSS** 配合

------

## 二、导航菜单

用于网站导航。

场景：

- 网站顶部导航
- 左侧菜单

```html
<nav>
  <ul>
    <li><a href="/">首页</a></li>
    <li><a href="/blog">博客</a></li>
    <li><a href="/about">关于</a></li>
  </ul>
</nav>
```

------

## 三、文章展示（内容网站）

场景：

- 博客
- 新闻网站
- 文档系统

```html
<article>
  <h1>HTML学习指南</h1>

  <p>HTML 是构建网页的基础语言。</p>

  <section>
    <h2>第一章</h2>
    <p>HTML 基本结构...</p>
  </section>

</article>
```

------

## 四、图片展示（图库）

场景：

- 产品展示
- 相册
- 电商商品

```html
<div class="gallery">

  <img src="img1.jpg" alt="图片1">
  <img src="img2.jpg" alt="图片2">
  <img src="img3.jpg" alt="图片3">

</div>
```

常配合 **Vue.js** 做动态渲染。

------

## 五、超链接跳转

场景：

- 页面跳转
- 外部网站
- 下载

```html
<a href="https://google.com">访问Google</a>

<a href="/docs/tutorial.html">查看教程</a>

<a href="file.zip" download>下载文件</a>
```

------

## 六、数据表格展示

场景：

- 管理系统列表
- 成绩表
- 数据统计

```html
<table border="1">

<tr>
  <th>姓名</th>
  <th>年龄</th>
</tr>

<tr>
  <td>张三</td>
  <td>20</td>
</tr>

<tr>
  <td>李四</td>
  <td>22</td>
</tr>

</table>
```

在后台系统中通常由 **Vue 3** 动态渲染。

------

## 七、用户登录注册（表单）

场景：

- 登录
- 注册
- 信息提交

```html
<form action="/login" method="post">

  <label>用户名</label>
  <input type="text" name="username">

  <label>密码</label>
  <input type="password" name="password">

  <button type="submit">登录</button>

</form>
```

数据通常提交到 **Spring Boot** 后端。

------

## 八、搜索框

场景：

- 网站搜索
- 商品搜索

```html
<form action="/search">

<input type="text" name="keyword" placeholder="搜索文章">

<button>搜索</button>

</form>
```

------

## 九、视频播放

场景：

- 在线教育
- 视频网站
- 产品介绍

```html
<video controls width="600">

  <source src="video.mp4" type="video/mp4">

</video>
```

------

## 十、音频播放

场景：

- 音乐网站
- 播客

```html
<audio controls>

  <source src="music.mp3">

</audio>
```

------

## 十一、图形绘制（Canvas）

场景：

- 图表
- 游戏
- 动画

```html
<canvas id="myCanvas" width="200" height="200"></canvas>

<script>

const canvas = document.getElementById("myCanvas")
const ctx = canvas.getContext("2d")

ctx.fillStyle = "red"
ctx.fillRect(20,20,150,100)

</script>
```

------

## 十二、拖拽交互

场景：

- 文件上传
- 拖拽排序

```html
<div draggable="true">
拖动我
</div>
```

------

## 十三、HTML在真实项目中的位置

完整结构：

```
前端系统
│
├── HTML
│   ├── 页面结构
│   ├── 表单
│   ├── 媒体
│   └── 内容
│
├── CSS
│   ├── 样式
│   ├── 布局
│
├── JavaScript
│   ├── DOM操作
│   ├── 事件
│   ├── Ajax
│
└── 前端框架
     └── Vue
```

------

## 面试级总结

HTML应用场景可以总结为 **10类**：

| 分类     | 场景     |
| -------- | -------- |
| 页面结构 | 网站布局 |
| 导航     | 菜单     |
| 文章     | 内容展示 |
| 图片     | 图库     |
| 链接     | 页面跳转 |
| 表格     | 数据展示 |
| 表单     | 用户输入 |
| 搜索     | 查询     |
| 视频音频 | 多媒体   |
| Canvas   | 图形     |

------

如果你愿意，我可以再给你整理一份 **“HTML 30个真实网页组件代码（项目级）”**，比如：

- 登录页
- 商品卡片
- 评论区
- 后台管理表格
- 文件上传
- 图片轮播
- 分页组件

这套代码 **基本覆盖 80% 网站 HTML 场景**，对学习 **Vue 3** 和前端面试非常有帮助。

