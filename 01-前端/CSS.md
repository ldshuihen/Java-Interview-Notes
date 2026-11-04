# CSS 

## 一、CSS 基础概念

### 1. **什么是 CSS？**

**CSS（Cascading Style Sheets，层叠样式表）** 用于控制网页的外观和布局。

```css
/* 基本语法 */
选择器 {
  属性: 值;
}

/* 示例 */
h1 {
  color: blue;
  font-size: 24px;
}
```

### 2. **CSS 的三种引入方式**

#### ① 内联样式（Inline Style）
```html
<h1 style="color: blue; font-size: 24px;">标题</h1>
```
- ⚠️ 不推荐，难以维护

#### ② 内部样式表（Internal Style Sheet）
```html
<head>
  <style>
    h1 {
      color: blue;
      font-size: 24px;
    }
  </style>
</head>
```
- 适合单个页面

#### ③ 外部样式表（External Style Sheet）✅ 推荐
```html
<head>
  <link rel="stylesheet" href="styles.css">
</head>
```

```css
/* styles.css */
h1 {
  color: blue;
  font-size: 24px;
}
```
- ✅ 可复用，易维护

---

## 二、CSS 选择器

### 1. **基础选择器**

```css
/* 元素选择器 */
div { color: red; }
p { margin: 10px; }

/* 类选择器（最常用） */
.container { max-width: 1200px; }
.btn-primary { background: blue; }

/* ID 选择器（唯一性） */
#header { height: 60px; }
#main-content { padding: 20px; }

/* 通配符选择器 */
* { margin: 0; padding: 0; }
```

### 2. **组合选择器**

```css
/* 后代选择器 */
.parent .child { color: red; }

/* 子元素选择器 */
.parent > .child { color: blue; }

/* 相邻兄弟选择器 */
h1 + p { font-size: 18px; }

/* 通用兄弟选择器 */
h1 ~ p { margin-top: 10px; }
```

### 3. **属性选择器**

```css
/* 具有某属性 */
input[required] { border: 2px solid red; }

/* 属性值等于 */
input[type="text"] { padding: 8px; }

/* 属性值包含 */
[class*="btn-"] { cursor: pointer; }

/* 属性值以...开头 */
[class^="icon-"] { display: inline-block; }
```

### 4. **伪类选择器**

```css
/* 状态伪类 */
a:hover { color: red; }          /* 鼠标悬停 */
a:active { color: green; }       /* 点击时 */
input:focus { border-color: blue; } /* 聚焦时 */
button:disabled { opacity: 0.5; }   /* 禁用时 */

/* 结构伪类 */
li:first-child { font-weight: bold; }  /* 第一个子元素 */
li:last-child { margin-bottom: 0; }    /* 最后一个子元素 */
li:nth-child(2n) { background: #f0f0f0; } /* 偶数项 */
li:nth-child(odd) { background: #fff; }   /* 奇数项 */

/* 否定伪类 */
input:not([type="submit"]) { width: 100%; }
```

### 5. **伪元素选择器**

```css
/* 在元素前插入内容 */
h1::before {
  content: "★";
  color: gold;
}

/* 在元素后插入内容 */
.required::after {
  content: "*";
  color: red;
}

/* 首行样式 */
p::first-line {
  font-weight: bold;
}

/* 首字母样式 */
p::first-letter {
  font-size: 2em;
  color: blue;
}

/* 选中文字样式 */
::selection {
  background: yellow;
  color: black;
}
```

---

## 三、盒模型（Box Model）

### 1. **盒模型组成**

```
┌─────────────────────────────┐
│         margin              │
│  ┌───────────────────────┐  │
│  │       border          │  │
│  │  ┌─────────────────┐  │  │
│  │  │    padding      │  │  │
│  │  │  ┌───────────┐  │  │  │
│  │  │  │  content  │  │  │  │
│  │  │  │  (内容)    │  │  │  │
│  │  │  └───────────┘  │  │  │
│  │  └─────────────────┘  │  │
│  └───────────────────────┘  │
└─────────────────────────────┘
```

### 2. **两种盒模型**

#### ① 标准盒模型（默认）
```css
box-sizing: content-box;
```
```
总宽度 = width + padding + border + margin
```

#### ② IE 盒模型（推荐）✅
```css
box-sizing: border-box;
```
```
总宽度 = width（包含 padding 和 border）
```

### 3. **实际应用**

```css
/* 全局重置（推荐） */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* 示例 */
.card {
  width: 300px;
  padding: 20px;
  border: 2px solid #ddd;
  /* 标准盒模型总宽度: 300 + 20*2 + 2*2 = 344px */
  /* border-box 总宽度: 300px ✅ */
}
```

---

## 四、布局系统

### 1. **普通文档流（Normal Flow）**

```html
<!-- 块级元素：独占一行 -->
<div>块1</div>
<div>块2</div>

<!-- 行内元素：同行显示 -->
<span>行内1</span>
<span>行内2</span>
```

### 2. **Flexbox 弹性布局** ⭐⭐⭐⭐⭐

```css
/* 容器 */
.container {
  display: flex;
  flex-direction: row;           /* 主轴方向: row | column */
  justify-content: center;       /* 主轴对齐: flex-start | center | flex-end | space-between | space-around */
  align-items: center;           /* 交叉轴对齐: flex-start | center | flex-end | stretch */
  flex-wrap: wrap;               /* 换行: nowrap | wrap */
  gap: 16px;                     /* 间距 */
}

/* 子元素 */
.item {
  flex: 1;                       /* 等分 */
  flex-grow: 1;                  /* 放大比例 */
  flex-shrink: 0;                /* 缩小比例 */
  flex-basis: 200px;             /* 基础大小 */
  align-self: flex-end;          /* 单独对齐 */
  order: 1;                      /* 排列顺序 */
}
```

**常用场景：**

```css
/* 水平垂直居中 */
.center {
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 两端对齐 */
.spaceBetween {
  display: flex;
  justify-content: space-between;
}

/* 圣杯布局 */
.layout {
  display: flex;
}
.sidebar { width: 200px; }
.main { flex: 1; }
```

### 3. **Grid 网格布局** ⭐⭐⭐⭐

```css
/* 容器 */
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);     /* 3列等宽 */
  grid-template-columns: 200px 1fr 200px;    /* 固定+自适应+固定 */
  grid-template-rows: auto 1fr auto;         /* 行高 */
  gap: 20px;                                 /* 间距 */
}

/* 子元素 */
.grid-item {
  grid-column: 1 / 3;    /* 跨列 */
  grid-row: 1 / 2;       /* 跨行 */
}
```

**常用场景：**

```css
/* 响应式卡片布局 */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
}

/* 复杂布局 */
.dashboard {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 200px 1fr;
  grid-template-rows: 60px 1fr 40px;
}
```

### 4. **定位布局（Position）**

```css
/* static（默认） */
.static { position: static; }

/* relative（相对定位） */
.relative {
  position: relative;
  top: 10px;      /* 相对于自身 */
  left: 20px;
}

/* absolute（绝对定位） */
.absolute {
  position: absolute;
  top: 0;
  right: 0;       /* 相对于最近的定位父元素 */
}

/* fixed（固定定位） */
.fixed {
  position: fixed;
  bottom: 20px;
  right: 20px;    /* 相对于视口 */
}

/* sticky（粘性定位） */
.sticky {
  position: sticky;
  top: 0;         /* 滚动到顶部时固定 */
}
```

---

## 五、常用样式属性

### 1. **文本样式**

```css
.text {
  color: #333;
  font-size: 16px;
  font-weight: bold;           /* normal | bold | 100-900 */
  font-family: Arial, sans-serif;
  line-height: 1.5;            /* 行高 */
  letter-spacing: 1px;         /* 字间距 */
  text-align: center;          /* left | center | right | justify */
  text-decoration: none;       /* underline | line-through | none */
  text-transform: uppercase;   /* uppercase | lowercase | capitalize */
  white-space: nowrap;         /* 不换行 */
  overflow: hidden;            /* 隐藏溢出 */
  text-overflow: ellipsis;     /* 省略号 */
}
```

### 2. **背景样式**

```css
.background {
  background-color: #f0f0f0;
  background-image: url('bg.jpg');
  background-repeat: no-repeat;     /* repeat | no-repeat | repeat-x | repeat-y */
  background-position: center;      /* center | top | bottom | left | right */
  background-size: cover;           /* cover | contain | auto */
  background-attachment: fixed;     /* 固定背景 */
  
  /* 简写 */
  background: #f0f0f0 url('bg.jpg') no-repeat center/cover;
}
```

### 3. **边框样式**

```css
.border {
  border: 1px solid #ddd;           /* 宽度 样式 颜色 */
  border-radius: 8px;               /* 圆角 */
  border-radius: 50%;               /* 圆形 */
  box-shadow: 0 2px 8px rgba(0,0,0,0.1); /* 阴影 */
}
```

### 4. **过渡与动画**

```css
/* 过渡效果 */
.button {
  background: blue;
  transition: all 0.3s ease;        /* 属性 时长 速度曲线 延迟 */
}
.button:hover {
  background: darkblue;
}

/* 关键帧动画 */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.fade-in {
  animation: fadeIn 0.5s ease-in-out;
}

/* 复杂动画 */
@keyframes slideIn {
  0% {
    transform: translateX(-100%);
    opacity: 0;
  }
  100% {
    transform: translateX(0);
    opacity: 1;
  }
}
```

### 5. **变换（Transform）**

```css
.transform {
  transform: translate(50px, 100px);   /* 位移 */
  transform: rotate(45deg);            /* 旋转 */
  transform: scale(1.5);               /* 缩放 */
  transform: skew(10deg, 5deg);        /* 倾斜 */
  
  /* 组合变换 */
  transform: translateX(-50%) translateY(-50%) scale(1.2);
}
```

---

## 六、响应式设计

### 1. **媒体查询（Media Queries）**

```css
/* 移动端优先 */
.container {
  width: 100%;
  padding: 0 16px;
}

/* 平板 */
@media (min-width: 768px) {
  .container {
    max-width: 750px;
    margin: 0 auto;
  }
}

/* 桌面端 */
@media (min-width: 1024px) {
  .container {
    max-width: 1200px;
  }
}

/* 大屏 */
@media (min-width: 1440px) {
  .container {
    max-width: 1400px;
  }
}
```

### 2. **响应式单位**

```css
.responsive {
  width: 50%;              /* 相对于父元素 */
  font-size: 16px;         /* 固定像素 */
  font-size: 1rem;         /* 相对于根元素 */
  font-size: 1em;          /* 相对于父元素 */
  width: 50vw;             /* 相对于视口宽度 */
  height: 100vh;           /* 相对于视口高度 */
}
```

---

## 七、CSS 预处理器

### 1. **SCSS/SASS**

```scss
// 变量
$primary-color: #3498db;
$font-size-base: 16px;

// 嵌套
.container {
  max-width: 1200px;
  margin: 0 auto;
  
  .header {
    height: 60px;
    
    .logo {
      font-size: $font-size-base;
      color: $primary-color;
      
      &:hover {
        opacity: 0.8;
      }
    }
  }
}

// 混合（Mixin）
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

.center-box {
  @include flex-center;
}

// 继承
%btn-base {
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
}

.btn-primary {
  @extend %btn-base;
  background: blue;
  color: white;
}
```



### 2. **Less**

```less
@primary-color: #3498db;

.container {
  width: 100%;
  
  .header {
    background: @primary-color;
  }
}
```

---

## 八、CSS 模块化与工程化

### 1. **CSS Modules**

```vue
<!-- Vue 单文件组件 -->
<template>
  <div :class="$style.container">
    <h1 :class="$style.title">标题</h1>
  </div>
</template>

<style module>
.container {
  max-width: 1200px;
}

.title {
  color: blue;
}
</style>
```

### 2. **Scoped CSS（Vue 推荐）**

```vue
<template>
  <div class="container">
    <h1 class="title">标题</h1>
  </div>
</template>

<style scoped>
/* 只作用于当前组件 */
.container {
  max-width: 1200px;
}

.title {
  color: blue;
}
</style>
```

### 3. **BEM 命名规范**

```css
/* Block（块） */
.card { }

/* Element（元素） */
.card__title { }
.card__content { }
.card__footer { }

/* Modifier（修饰符） */
.card--primary { }
.card--large { }
.card__title--bold { }
```

---

## 九、CSS 层叠与优先级

### 1. **优先级计算**

```
内联样式 (1000) > ID 选择器 (100) > 类/属性/伪类 (10) > 元素/伪元素 (1)
```

```css
/* 优先级: 1 */
div { }

/* 优先级: 10 */
.container { }
[type="text"] { }
:hover { }

/* 优先级: 11 */
.container div { }       /* 10 + 1 */

/* 优先级: 100 */
#header { }

/* 优先级: 1000 */
<div style="color: red;">  /* 内联样式 */
```

### 2. **!important（慎用）**

```css
.title {
  color: red !important;  /* 最高优先级 */
}
```

⚠️ 不推荐过度使用，会破坏层叠规则

---

## 十、现代 CSS 特性

### 1. **CSS 变量（自定义属性）**

```css
/* 定义全局变量 */
:root {
  --primary-color: #3498db;
  --font-size-base: 16px;
  --spacing: 16px;
}

/* 使用变量 */
.button {
  background: var(--primary-color);
  font-size: var(--font-size-base);
  padding: var(--spacing);
}

/* 局部变量 */
.card {
  --card-bg: #fff;
  background: var(--card-bg);
}
```

### 2. **CSS Grid + Flexbox 组合**

```css
.layout {
  display: grid;
  grid-template-columns: 200px 1fr;
  
  .sidebar {
    display: flex;
    flex-direction: column;
  }
  
  .main {
    display: flex;
    flex-direction: column;
  }
}
```

### 3. **Container Queries（容器查询）**

```css
/* 根据容器宽度而非视口响应 */
@container (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 200px 1fr;
  }
}
```

---

## 十一、CSS 调试技巧

### 1. **通用调试样式**

```css
/* 显示所有元素边框 */
* {
  outline: 1px solid rgba(255, 0, 0, 0.3);
}

/* 查看溢出 */
.overflow-debug {
  overflow: auto;
  background: rgba(0, 255, 0, 0.1);
}
```

### 2. **浏览器 DevTools**

- **Elements** - 查看和修改样式
- **Computed** - 查看计算后的样式
- **Layout** - 查看盒模型
- **Console** - 查看错误

---

## 十二、最佳实践

### ✅ 推荐做法：

```css
/* 1. 全局重置 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* 2. 使用 CSS 变量 */
:root {
  --primary-color: #3498db;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
}

/* 3. 移动端优先的响应式 */
.container {
  width: 100%;
}

@media (min-width: 768px) {
  .container { max-width: 750px; }
}

@media (min-width: 1024px) {
  .container { max-width: 1200px; }
}

/* 4. 使用 Flexbox/Grid 布局 */
.layout {
  display: flex;
  gap: var(--spacing-md);
}

/* 5. 语义化类名 */
.user-card { }
.user-card__avatar { }
.user-card--active { }
```

### ❌ 避免做法：

```css
/* ❌ 滥用 !important */
.title { color: red !important; }

/* ❌ 选择器嵌套过深 */
.parent .child .grandchild .item { }

/* ❌ 使用内联样式 */
<div style="color: red; margin: 10px;">

/* ❌ 硬编码颜色和尺寸 */
.button { background: #3498db; }
/* 应使用变量 */
.button { background: var(--primary-color); }
```

---

## 十三、你的项目中的 CSS

### Vue 项目中的样式：

```vue
<!-- pmsLineBaseDataQuery.vue -->
<style scoped lang="scss">
/* scoped 限制样式只作用于当前组件 */
/* lang="scss" 使用 SCSS 预处理器 */

.enquiries-form {
  :deep .el-form-item__label {
    text-align: right;  /* 右对齐 label */
  }
}

.select-style {
  width: 100%;
}
</style>
```

### 全局样式注入：

```javascript
// main.js
import '@/style/cmi.scss'  // 全局样式，所有组件可用
```

---

## 总结

| 特性         | 说明                          | 重要度 |
| ------------ | ----------------------------- | ------ |
| **选择器**   | 类选择器最常用                | ⭐⭐⭐⭐⭐  |
| **盒模型**   | 使用 `box-sizing: border-box` | ⭐⭐⭐⭐⭐  |
| **Flexbox**  | 一维布局首选                  | ⭐⭐⭐⭐⭐  |
| **Grid**     | 二维布局首选                  | ⭐⭐⭐⭐   |
| **响应式**   | 媒体查询 + 相对单位           | ⭐⭐⭐⭐⭐  |
| **CSS 变量** | 主题化和维护                  | ⭐⭐⭐⭐   |
| **预处理器** | SCSS/Less 提升开发效率        | ⭐⭐⭐⭐   |
| **模块化**   | Scoped CSS / CSS Modules      | ⭐⭐⭐⭐⭐  |

# CSS 规则

## 一、CSS 规则的基本结构

### 1. **CSS 规则集（Rule Set）**

```css
/* 选择器 { 属性: 值; } */
选择器 {
  属性1: 值1;
  属性2: 值2;
  属性3: 值3;
}
```

**示例：**

```css
/* 完整的 CSS 规则 */
.container {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  background-color: #ffffff;
  border-radius: 8px;
}
```

---

### 2. **组成部分**

```css
/* 选择器 */      /* 声明块 */
   h1         { color: blue; font-size: 24px; }
   ↑                ↑              ↑
 选择器            属性            值
```

**详细说明：**

| 部分       | 说明                   | 示例                  |
| ---------- | ---------------------- | --------------------- |
| **选择器** | 选择要应用样式的元素   | `h1`, `.class`, `#id` |
| **声明块** | 用 `{}` 包裹的所有声明 | `{ color: blue; }`    |
| **声明**   | 一个属性-值对          | `color: blue;`        |
| **属性**   | 要设置的 CSS 属性      | `color`, `font-size`  |
| **值**     | 属性的具体值           | `blue`, `24px`        |

---

## 二、选择器类型

### 1. **基础选择器**

```css
/* 1. 元素选择器（标签选择器） */
div { }
p { }
h1 { }
button { }

/* 2. 类选择器 */
.container { }
.btn { }
.btn-primary { }

/* 3. ID 选择器 */
#header { }
#main-content { }

/* 4. 通配符选择器 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* 5. 属性选择器 */
input[type="text"] { }
a[href^="https"] { }      /* 以 https 开头 */
a[href$=".pdf"] { }       /* 以 .pdf 结尾 */
a[href*="example"] { }    /* 包含 example */
[class~="btn"] { }        /* 包含 btn 单词 */
[lang|="zh"] { }          /* 以 zh 开头 */
```

---

### 2. **组合选择器**

```css
/* 1. 后代选择器（空格） */
.container p { }           /* .container 内所有 p */
div span { }               /* div 内所有 span */

/* 2. 子选择器（>） */
.container > p { }         /* .container 的直接子元素 p */
ul > li { }                /* ul 的直接子元素 li */

/* 3. 相邻兄弟选择器（+） */
h1 + p { }                 /* h1 后面的第一个 p */
input + label { }          /* input 后面的 label */

/* 4. 通用兄弟选择器（~） */
h1 ~ p { }                 /* h1 后面所有同级的 p */
input:checked ~ label { }  /* 选中的 input 后面的 label */

/* 5. 群组选择器（,） */
h1, h2, h3 { }             /* h1、h2、h3 共用样式 */
.btn, .button { }          /* .btn 和 .button 共用样式 */
```

---

### 3. **伪类选择器**

```css
/* 用户交互伪类 */
a:hover { }                /* 鼠标悬停 */
a:active { }               /* 激活状态（点击时） */
a:visited { }              /* 已访问链接 */
a:link { }                 /* 未访问链接 */
input:focus { }            /* 获得焦点 */
button:disabled { }        /* 禁用状态 */

/* 结构伪类 */
li:first-child { }         /* 第一个子元素 */
li:last-child { }          /* 最后一个子元素 */
li:nth-child(2) { }        /* 第 2 个子元素 */
li:nth-child(odd) { }      /* 奇数个子元素 */
li:nth-child(even) { }     /* 偶数个子元素 */
li:nth-child(3n) { }       /* 每 3 个 */
li:nth-child(3n+1) { }     /* 第 1, 4, 7... 个 */
li:nth-last-child(2) { }   /* 倒数第 2 个 */
p:first-of-type { }        /* 第一个 p 元素 */
p:last-of-type { }         /* 最后一个 p 元素 */
p:nth-of-type(2) { }       /* 第 2 个 p 元素 */
p:only-child { }           /* 唯一的子元素 */
p:only-of-type { }         /* 唯一类型的子元素 */

/* 表单伪类 */
input:checked { }          /* 选中的复选框/单选框 */
input:disabled { }         /* 禁用状态 */
input:enabled { }          /* 启用状态 */
input:required { }         /* 必填字段 */
input:optional { }         /* 可选字段 */
input:valid { }            /* 验证通过 */
input:invalid { }          /* 验证失败 */
input:read-only { }        /* 只读 */
input:read-write { }       /* 可读写 */

/* 其他伪类 */
:not(.btn) { }             /* 不包含 .btn 的元素 */
:empty { }                 /* 空元素 */
:root { }                  /* 根元素（html） */
```

---

### 4. **伪元素选择器**

```css
/* 伪元素（双冒号） */
p::before {                /* 在 p 前插入内容 */
  content: "前缀: ";
}

p::after {                 /* 在 p 后插入内容 */
  content: " [结束]";
}

p::first-line {            /* p 的第一行 */
  font-weight: bold;
}

p::first-letter {          /* p 的第一个字母 */
  font-size: 2em;
  color: red;
}

::selection {              /* 选中的文本 */
  background: yellow;
  color: black;
}

::placeholder {            /* 输入框占位符 */
  color: #999;
}

::marker {                 /* 列表标记 */
  color: blue;
}
```

---

## 三、声明规则

### 1. **属性声明**

```css
.selector {
  /* 基本语法：属性: 值; */
  color: red;
  font-size: 16px;
  margin: 10px;
  padding: 20px;
  
  /* 多个值 */
  margin: 10px 20px 10px 20px;  /* 上 右 下 左 */
  margin: 10px 20px;            /* 上下 左右 */
  margin: 10px;                 /* 所有方向 */
  
  /* 简写属性 */
  background: #fff url('bg.png') no-repeat center;
  font: bold 16px/1.5 Arial, sans-serif;
  border: 1px solid #ddd;
}
```

---

### 2. **值类型**

```css
/* 1. 关键字值 */
display: block;
position: relative;
text-align: center;

/* 2. 长度值 */
width: 100px;          /* 绝对单位 */
width: 10rem;          /* 相对单位（根元素） */
width: 10em;           /* 相对单位（父元素） */
width: 50%;            /* 百分比 */
width: 10vw;           /* 视口宽度 */
width: 10vh;           /* 视口高度 */

/* 3. 颜色值 */
color: red;            /* 关键字 */
color: #ff0000;        /* 十六进制 */
color: #f00;           /* 简写十六进制 */
color: rgb(255, 0, 0); /* RGB */
color: rgba(255, 0, 0, 0.5); /* RGBA */
color: hsl(0, 100%, 50%);    /* HSL */
color: hsla(0, 100%, 50%, 0.5); /* HSLA */

/* 4. URL 值 */
background-image: url('image.png');

/* 5. 计算值 */
width: calc(100% - 20px);
width: calc(50vw - 100px);

/* 6. 变量值 */
width: var(--container-width);
color: var(--primary-color);
```

---

### 3. **重要声明（!important）**

```css
/* 优先级最高，覆盖所有其他规则 */
.text {
  color: blue !important;
}

/* ⚠️ 慎用！会破坏层叠规则 */
```

---

## 四、CSS 层叠与优先级

### 1. **优先级计算（Specificity）**

```css
/* 优先级从低到高 */

/* 1. 元素选择器 (0,0,0,1) */
div { }

/* 2. 类选择器 (0,0,1,0) */
.container { }

/* 3. 属性选择器 (0,0,1,0) */
[type="text"] { }

/* 4. 伪类 (0,0,1,0) */
:hover { }

/* 5. ID 选择器 (0,1,0,0) */
#header { }

/* 6. 内联样式 (1,0,0,0) */
<div style="color: red;">

/* 7. !important (最高优先级) */
color: red !important;
```

**优先级计算规则：**

```
(内联样式, ID, 类/属性/伪类, 元素/伪元素)

示例：
#nav .menu li a:hover  →  (0, 1, 2, 2)
.container p           →  (0, 0, 1, 1)
div.wrapper            →  (0, 0, 1, 1)
```

---

### 2. **层叠规则**

```css
/* 当优先级相同时，后面的规则覆盖前面的规则 */
p {
  color: blue;    /* 被覆盖 */
}

p {
  color: red;     /* 生效 */
}

/* 来源优先级（从低到高）： */
/* 1. 浏览器默认样式 */
/* 2. 用户自定义样式 */
/* 3. 作者样式表（你的 CSS） */
/* 4. 作者 !important */
/* 5. 用户 !important */
```

### CSS 多 class 的生效规则

在你的代码中：

```vue
<el-button type="" plain class="btn btnRest" @click="resetForm">
  {{ $t("button.reset") }}
</el-button>
```

**`btn` 和 `btnRest` 两个 class 都会生效！** 它们不会互相覆盖，而是**合并应用**。

---

#### 一、CSS 层叠规则详解

##### 1. **多个 class 同时生效**

```html
class="btn btnRest"
```

**CSS 定义：**

```scss
// cmi.scss 第 70-72 行
.btn {
  min-width: 80px;
}

// 假设的 btnRest 样式（项目中未定义）
.btnRest {
  background-color: #f5f5f5;
  color: #666;
}
```

**最终生效的样式：**

```css
/* 两个 class 的样式会合并 */
element {
  min-width: 80px;        /* 来自 .btn */
  background-color: #f5f5f5;  /* 来自 .btnRest */
  color: #666;            /* 来自 .btnRest */
}
```

---

##### 2. **CSS 层叠规则（Cascading）**

当多个规则应用于同一元素时，浏览器按照以下规则决定哪个样式生效：

###### **规则 1：优先级（Specificity）**

```css
/* 优先级从低到高 */
div { }              /* 元素选择器 (0,0,0,1) */
.btn { }             /* 类选择器 (0,0,1,0) */
#header { }          /* ID 选择器 (0,1,0,0) */
style="..."          /* 内联样式 (1,0,0,0) */
!important           /* 最高优先级 */
```

**在你的项目中：**

```scss
.btn {
  min-width: 80px;   /* 优先级: (0,0,1,0) */
}

.btnRest {
  min-width: 100px;  /* 优先级: (0,0,1,0) - 相同 */
}
```

⚠️ **当优先级相同时，应用后面的规则！**

---

###### **规则 2：源代码顺序（Source Order）**

```css
/* 如果 .btn 和 .btnRest 设置了相同属性 */

/* 情况 1：.btn 在后面 */
.btnRest {
  min-width: 100px;
}
.btn {
  min-width: 80px;  /* ✅ 生效（在后面） */
}

/* 情况 2：.btnRest 在后面 */
.btn {
  min-width: 80px;
}
.btnRest {
  min-width: 100px;  /* ✅ 生效（在后面） */
}
```

---

###### **规则 3：!important 最高优先级**

```css
.btn {
  min-width: 80px !important;  /* ✅ 总是生效 */
}

.btnRest {
  min-width: 100px;  /* 被覆盖 */
}
```

---

#### 三、示例演示

##### 示例 1：两个 class 都定义不同属性

```scss
.btn {
  min-width: 80px;
  padding: 8px 16px;
}

.btnRest {
  background-color: #f5f5f5;
  color: #666;
}
```

```html
<button class="btn btnRest">重置</button>
```

**✅ 最终样式：**

```css
button {
  min-width: 80px;        /* 来自 .btn */
  padding: 8px 16px;      /* 来自 .btn */
  background-color: #f5f5f5;  /* 来自 .btnRest */
  color: #666;            /* 来自 .btnRest */
}
```

---

##### 示例 2：两个 class 定义相同属性

```scss
.btn {
  min-width: 80px;
  background: blue;
}

.btnRest {
  min-width: 100px;
  background: gray;
}
```

```html
<button class="btn btnRest">重置</button>
```

**✅ 取决于 CSS 文件中的顺序：**

```css
/* 如果 cmi.scss 中 .btn 在 .btnRest 后面 */
button {
  min-width: 80px;   /* .btn 生效 */
  background: blue;  /* .btn 生效 */
}

/* 如果 .btnRest 在 .btn 后面 */
button {
  min-width: 100px;  /* .btnRest 生效 */
  background: gray;  /* .btnRest 生效 */
}
```

---

##### 示例 3：使用 !important

```scss
.btn {
  min-width: 80px !important;  /* 最高优先级 */
}

.btnRest {
  min-width: 100px;
}
```

```html
<button class="btn btnRest">重置</button>
```

**✅ 最终样式：**

```css
button {
  min-width: 80px !important;  /* 总是生效 */
}
```

---

#### 四、CSS 层叠规则总结

##### 决策流程图：

```
多个规则应用到同一元素
         ↓
   有 !important 吗？
    ↙️          ↘️
  是              否
   ↓              ↓
!important 生效   比较优先级
                     ↓
              优先级高的生效
                     ↓
            优先级相同？
             ↙️          ↘️
           是              否
            ↓              ↓
        源代码后面的    优先级高的生效
        规则生效
```

---

#### 优先级对比表：

| 选择器         | 示例                    | 优先级值  | 说明         |
| -------------- | ----------------------- | --------- | ------------ |
| **!important** | `color: red !important` | ∞         | 最高优先级   |
| **内联样式**   | `style="color: red"`    | (1,0,0,0) | 写在 HTML 中 |
| **ID 选择器**  | `#header`               | (0,1,0,0) | 唯一标识     |
| **类选择器**   | `.btn`, `.btnRest`      | (0,0,1,0) | 常用         |
| **属性选择器** | `[type="text"]`         | (0,0,1,0) | 按属性选择   |
| **伪类**       | `:hover`, `:focus`      | (0,0,1,0) | 状态         |
| **元素选择器** | `div`, `button`         | (0,0,0,1) | 标签         |
| **伪元素**     | `::before`              | (0,0,0,1) | 伪元素       |
| **通配符**     | `*`                     | (0,0,0,0) | 最低         |

---

#### 符合的 CSS 规则：

1. **✅ 层叠规则（Cascading）**
   - 多个 class 的样式会合并应用
   - 不冲突的属性都会生效

2. **✅ 优先级规则（Specificity）**
   - `.btn` 和 `.btnRest` 优先级相同（都是类选择器）
   - 如果有冲突，源代码后面的生效

3. **✅ 继承规则（Inheritance）**
   - 某些属性会继承父元素（如 `color`, `font-size`）
   - 某些属性不会继承（如 `margin`, `padding`）

4. **✅ 源代码顺序（Source Order）**
   - 优先级相同时，后面的覆盖前面的

---

#### 最佳实践：

```scss
// ✅ 推荐：通用样式 + 特定样式
.btn {
  min-width: 80px;  // 所有按钮共享
}

.btnRest {
  // 未来可以在这里添加重置按钮的特定样式
  background: #f5f5f5;
}

.btnSearch {
  // 搜索按钮的特定样式
  background: #409eff;
  color: white;
}
```

这就是 **CSS 的层叠（Cascading）特性**，多个规则可以和谐共存，共同决定元素的最终样式！🎨

---

## 五、CSS 规则类型

### 1. **@规则（At-Rules）**

```css
/* 1. @import - 导入样式表 */
@import url('reset.css');
@import 'variables.css';

/* 2. @media - 媒体查询 */
@media screen and (max-width: 768px) {
  .container {
    width: 100%;
  }
}

@media print {
  body {
    background: white;
  }
}

@media (prefers-color-scheme: dark) {
  body {
    background: #333;
    color: #fff;
  }
}

/* 3. @keyframes - 动画关键帧 */
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes slideIn {
  0% {
    transform: translateX(-100%);
  }
  50% {
    transform: translateX(10%);
  }
  100% {
    transform: translateX(0);
  }
}

/* 4. @font-face - 自定义字体 */
@font-face {
  font-family: 'MyFont';
  src: url('myfont.woff2') format('woff2'),
       url('myfont.woff') format('woff');
  font-weight: normal;
  font-style: normal;
}

/* 5. @supports - 特性查询 */
@supports (display: grid) {
  .container {
    display: grid;
  }
}

@supports not (display: grid) {
  .container {
    display: flex;
  }
}

/* 6. @layer - 层叠层（CSS3） */
@layer base {
  * { margin: 0; }
}

@layer components {
  .btn { padding: 10px; }
}

@layer utilities {
  .text-center { text-align: center; }
}

/* 7. @charset - 字符编码 */
@charset "UTF-8";

/* 8. @namespace - 命名空间 */
@namespace url(http://www.w3.org/1999/xhtml);

/* 9. @page - 页面设置（打印） */
@page {
  size: A4;
  margin: 2cm;
}

@page :first {
  margin-top: 5cm;
}
```

---

### 2. **嵌套规则（CSS 预处理器/原生 CSS）**

#### SCSS/Less 嵌套：

```scss
.container {
  max-width: 1200px;
  
  .header {
    background: white;
    
    h1 {
      font-size: 24px;
      
      &:hover {  // & 指代父选择器
        color: blue;
      }
    }
    
    &--primary {  // BEM 命名
      background: blue;
    }
    
    &__title {
      font-weight: bold;
    }
  }
}

/* 编译后的 CSS */
.container { max-width: 1200px; }
.container .header { background: white; }
.container .header h1 { font-size: 24px; }
.container .header h1:hover { color: blue; }
.container .header--primary { background: blue; }
.container .header__title { font-weight: bold; }
```

#### Vue 中的深度选择器：

```vue
<style scoped>
/* 穿透 scoped */
.container :deep(.el-button) {
  border-radius: 8px;
}

/* 兼容写法 */
.container ::v-deep .el-button {
  border-radius: 8px;
}
</style>
```

---

## 六、CSS 声明块规则

### 1. **声明分隔符**

```css
/* 分号分隔每个声明 */
.selector {
  property1: value1;  /* ✅ 推荐：每个声明后加分号 */
  property2: value2;
  property3: value3;  /* 最后一个也建议加分号 */
}

/* ❌ 错误：缺少分号 */
.selector {
  property1: value1
  property2: value2  /* 会报错 */
}
```

---

### 2. **注释规则**

```css
/* 单行注释 */
.selector {
  color: red;
}

/* 
 * 多行注释
 * 这是详细说明
 */
.selector {
  color: blue;
}

/* ⚠️ CSS 不支持 // 注释 */
// .selector { }  /* 错误！ */

/* ✅ 在 SCSS/Less 中可以使用 // */
.selector {
  color: red;  // 这是 SCSS 注释
}
```

---

### 3. **供应商前缀**

```css
/* 为了兼容旧浏览器 */
.selector {
  -webkit-transform: rotate(45deg);  /* Chrome, Safari, 旧 Edge */
     -moz-transform: rotate(45deg);  /* 旧 Firefox */
      -ms-transform: rotate(45deg);  /* IE 9 */
          transform: rotate(45deg);  /* 标准语法（最后写） */
  
  -webkit-border-radius: 8px;
          border-radius: 8px;
  
  -webkit-box-shadow: 0 2px 4px rgba(0,0,0,0.1);
          box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* ✅ 使用 Autoprefixer 自动添加前缀 */
```

---

## 七、CSS 变量（自定义属性）

### 1. **定义变量**

```css
/* 全局变量（:root） */
:root {
  --primary-color: #3498db;
  --secondary-color: #2ecc71;
  --font-size-base: 16px;
  --spacing: 16px;
  --border-radius: 8px;
  --shadow: 0 2px 8px rgba(0,0,0,0.1);
}

/* 局部变量 */
.container {
  --container-bg: #ffffff;
  --container-padding: 20px;
}
```

### 2. **使用变量**

```css
.button {
  background-color: var(--primary-color);
  font-size: var(--font-size-base);
  padding: var(--spacing);
  border-radius: var(--border-radius);
  box-shadow: var(--shadow);
}

/* 带默认值 */
.text {
  color: var(--text-color, #333);  /* 如果变量不存在，使用 #333 */
}
```

### 3. **JavaScript 操作变量**

```javascript
// 获取变量值
const primaryColor = getComputedStyle(document.documentElement)
  .getPropertyValue('--primary-color');

// 设置变量值
document.documentElement.style.setProperty('--primary-color', '#e74c3c');

// 删除变量
document.documentElement.style.removeProperty('--primary-color');
```

---

## 八、CSS 规则的最佳实践

### ✅ 推荐做法：

```css
/* 1. 使用语义化命名 */
.user-card { }
.article-header { }
.navigation-menu { }

/* 2. 遵循 BEM 命名规范 */
.block { }                    /* 块 */
.block__element { }           /* 元素 */
.block--modifier { }          /* 修饰符 */

/* 3. 使用 CSS 变量 */
:root {
  --primary: #3498db;
  --spacing: 16px;
}

.button {
  background: var(--primary);
  padding: var(--spacing);
}

/* 4. 简写属性 */
margin: 10px 20px;
background: #fff url('bg.png') no-repeat center;

/* 5. 合理的优先级 */
/* 优先使用类选择器，避免使用 ID 和 !important */
.card { }
.card--primary { }

/* 6. 移动端优先 */
.container {
  width: 100%;
}

@media (min-width: 768px) {
  .container {
    max-width: 720px;
  }
}

@media (min-width: 1200px) {
  .container {
    max-width: 1140px;
  }
}
```

---

### ❌ 避免做法：

```css
/* ❌ 过度使用 ID 选择器 */
#header #nav #menu { }

/* ❌ 嵌套过深 */
.container .wrapper .content .text p span { }

/* ❌ 滥用 !important */
.text {
  color: red !important;
  font-size: 16px !important;
}

/* ❌ 硬编码颜色和尺寸 */
.button {
  background: #3498db;  /* 应使用变量 */
  padding: 12px 24px;   /* 应使用变量 */
}

/* ❌ 内联样式（难以维护） */
<div style="color: red; font-size: 16px;">

/* ❌ 使用已废弃的属性 */
@import url('style.css');  /* 影响性能，应使用构建工具合并 */
```

---

## 九、你的项目中的实际规则

### 1. **全局样式（cmi.scss）**

```scss
// c:/Users/24141/Desktop/china-mobile/sa-rpc-web/src/style/cmi.scss

.cmi-wrapper {
  .btn {
    min-width: 80px;  // ✅ 通用按钮样式
  }
  
  .btnRight {
    text-align: right;
    margin: 0 16px 16px 0;
  }
  
  .btnArrow {
    width: 32px;
    height: 32px;
  }
  
  // ✅ 使用嵌套
  .enquiries-form {
    margin-top: 10px;
    width: 100%;
  }
}
```

### 2. **组件样式（带 scoped）**

```vue
<style scoped lang="scss">
// 组件专属样式
.enquiries-form {
  // ✅ 使用深度选择器
  :deep .el-form-item__label {
    text-align: right;
  }
}

.select-style {
  width: 100%;
}
</style>
```

---

## 十、CSS 规则优先级表

| 选择器类型     | 示例                     | 优先级    |
| -------------- | ------------------------ | --------- |
| **内联样式**   | `style="color: red"`     | (1,0,0,0) |
| **ID 选择器**  | `#header`                | (0,1,0,0) |
| **类选择器**   | `.container`             | (0,0,1,0) |
| **属性选择器** | `[type="text"]`          | (0,0,1,0) |
| **伪类**       | `:hover`, `:nth-child()` | (0,0,1,0) |
| **元素选择器** | `div`, `p`               | (0,0,0,1) |
| **伪元素**     | `::before`, `::after`    | (0,0,0,1) |
| **通配符**     | `*`                      | (0,0,0,0) |
| **!important** | `color: red !important`  | 最高      |

---

## 十一、总结

### CSS 规则核心要点：

```css
/* 1. 基本结构 */
选择器 {
  属性: 值;
}

/* 2. 选择器优先级 */
!important > 内联 > ID > 类/属性/伪类 > 元素/伪元素

/* 3. 层叠规则 */
/* 后面的覆盖前面的（优先级相同时） */

/* 4. 最佳实践 */
/* - 使用类选择器 */
/* - 避免嵌套过深 */
/* - 使用 CSS 变量 */
/* - 移动端优先 */
/* - 语义化命名 */
```

**在你的 Vue 项目中：**
- ✅ 使用 `scoped` 隔离组件样式
- ✅ 使用 SCSS 嵌套提高可读性
- ✅ 使用 `:deep` 穿透子组件样式
- ✅ 全局样式统一管理（`cmi.scss`、`index.scss`）

掌握这些 CSS 规则，可以写出更高效、更易维护的样式代码！🎨





# SCSS/SASS 语法

## 一、SCSS 与 SASS 的区别

| 特性           | SASS                     | SCSS                          |
| -------------- | ------------------------ | ----------------------------- |
| **文件扩展名** | `.sass`                  | `.scss`                       |
| **语法风格**   | 缩进式（无花括号和分号） | 类 CSS 语法（有花括号和分号） |
| **兼容性**     | 需要学习新语法           | 任何 CSS 都是合法的 SCSS      |
| **流行度**     | 较低                     | ⭐⭐⭐⭐⭐ 主流选择                |

### SASS 语法示例：
```sass
$primary-color: #3498db
.container
  max-width: 1200px
  .header
    background: $primary-color
```

### SCSS 语法示例：
```scss
$primary-color: #3498db;
.container {
  max-width: 1200px;
  .header {
    background: $primary-color;
  }
}
```

**推荐使用 SCSS，因为它与 CSS 完全兼容！**

---

## 二、变量（Variables）

### 1. **基本用法**

```scss
// 定义变量
$primary-color: #3498db;
$font-size-base: 16px;
$spacing: 16px;
$font-family: 'Helvetica Neue', Arial, sans-serif;
$border-radius: 4px;

// 使用变量
.button {
  background-color: $primary-color;
  font-size: $font-size-base;
  padding: $spacing;
  border-radius: $border-radius;
  font-family: $font-family;
}
```

### 2. **变量作用域**

```scss
// 全局变量
$global-color: blue;

.container {
  // 局部变量
  $local-color: red;
  color: $local-color; // red
}

.header {
  color: $global-color; // blue
  // color: $local-color; // ❌ 错误：局部变量不可访问
}
```

### 3. **变量默认值**

```scss
$primary-color: blue;
$primary-color: red !default; // 如果已定义则不覆盖

.button {
  background: $primary-color; // blue
}
```

### 4. **变量插值**

```scss
$side: top;
$radius: 10px;

.button {
  border-#{$side}-radius: $radius; // border-top-radius: 10px;
  
  // 用于类名
  .icon-#{$side} {
    margin-#{$side}: 5px;
  }
}
```

### 5. **数据类型**

```scss
// 数字
$width: 100px;
$opacity: 0.8;
$line-height: 1.5;

// 字符串
$name: "button";
$color-name: primary;

// 颜色
$red: #ff0000;
$blue: rgb(0, 0, 255);
$green: rgba(0, 255, 0, 0.5);

// 布尔值
$is-active: true;
$is-disabled: false;

// 列表
$colors: red, green, blue;
$sizes: 10px 20px 30px;

// Map（键值对）
$theme: (
  primary: #3498db,
  success: #2ecc71,
  warning: #f39c12,
  danger: #e74c3c
);

// null
$spacing: null;
```

---

## 三、嵌套（Nesting）

### 1. **选择器嵌套**

```scss
// SCSS
.navbar {
  background: #333;
  padding: 10px;
  
  .nav-item {
    display: inline-block;
    padding: 8px 16px;
    
    a {
      color: white;
      text-decoration: none;
      
      &:hover {
        color: #3498db;
      }
    }
  }
}

// 编译后的 CSS
.navbar {
  background: #333;
  padding: 10px;
}
.navbar .nav-item {
  display: inline-block;
  padding: 8px 16px;
}
.navbar .nav-item a {
  color: white;
  text-decoration: none;
}
.navbar .nav-item a:hover {
  color: #3498db;
}
```

### 2. **父选择器引用 `&`**

```scss
// 伪类
.button {
  background: blue;
  
  &:hover {
    background: darkblue;
  }
  
  &:active {
    background: navy;
  }
  
  &:disabled {
    opacity: 0.5;
  }
}

// BEM 命名
.card {
  background: white;
  
  &__header {
    padding: 16px;
  }
  
  &__content {
    padding: 20px;
  }
  
  &__footer {
    padding: 16px;
  }
  
  &--primary {
    background: blue;
  }
  
  &--large {
    font-size: 18px;
  }
}

// 编译后
.card { background: white; }
.card__header { padding: 16px; }
.card__content { padding: 20px; }
.card__footer { padding: 16px; }
.card--primary { background: blue; }
.card--large { font-size: 18px; }

// 组合使用
.link {
  color: blue;
  
  &:hover,
  &:focus {
    color: darkblue;
  }
  
  .icon & {
    font-size: 20px; // .icon .link
  }
}
```

### 3. **属性嵌套**

```scss
// SCSS
.card {
  border: {
    style: solid;
    width: 1px;
    color: #ddd;
    radius: 8px;
  }
  
  margin: {
    top: 10px;
    bottom: 20px;
  }
  
  font: {
    size: 16px;
    weight: bold;
    family: Arial, sans-serif;
  }
}

// 编译后的 CSS
.card {
  border-style: solid;
  border-width: 1px;
  border-color: #ddd;
  border-radius: 8px;
  margin-top: 10px;
  margin-bottom: 20px;
  font-size: 16px;
  font-weight: bold;
  font-family: Arial, sans-serif;
}
```

### 4. **@at-root（跳出嵌套）**

```scss
// 跳出到根级别
.parent {
  color: blue;
  
  @at-root .child {
    color: red;
  }
  
  @at-root {
    .sibling1 { color: green; }
    .sibling2 { color: yellow; }
  }
}

// 编译后
.parent { color: blue; }
.child { color: red; }
.sibling1 { color: green; }
.sibling2 { color: yellow; }

// 与媒体查询结合
.component {
  background: white;
  
  @media (max-width: 768px) {
    @at-root .mobile-#{$selector} {
      padding: 10px;
    }
  }
}
```

---

## 四、混合（Mixins）

### 1. **基本用法**

```scss
// 定义混合
@mixin border-radius {
  -webkit-border-radius: 4px;
  -moz-border-radius: 4px;
  border-radius: 4px;
}

// 使用混合
.button {
  @include border-radius;
  padding: 10px 20px;
}

.card {
  @include border-radius;
  background: white;
}
```

### 2. **带参数的混合**

```scss
// 单个参数
@mixin flex-center($direction: row) {
  display: flex;
  flex-direction: $direction;
  justify-content: center;
  align-items: center;
}

.container {
  @include flex-center; // 使用默认值
}

.column-container {
  @include flex-center(column); // 传参
}

// 多个参数
@mixin size($width, $height: $width) {
  width: $width;
  height: $height;
}

.square {
  @include size(100px); // width: 100px; height: 100px;
}

.rectangle {
  @include size(200px, 100px); // width: 200px; height: 100px;
}

// 复杂混合
@mixin button-variant($bg-color, $text-color: white) {
  background-color: $bg-color;
  color: $text-color;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  
  &:hover {
    background-color: darken($bg-color, 10%);
  }
  
  &:active {
    background-color: darken($bg-color, 20%);
  }
}

.btn-primary {
  @include button-variant(#3498db);
}

.btn-danger {
  @include button-variant(#e74c3c);
}
```

### 3. **可变参数（...）**

```scss
// 接受任意数量的参数
@mixin box-shadow($shadows...) {
  -webkit-box-shadow: $shadows;
  box-shadow: $shadows;
}

.card {
  @include box-shadow(0 2px 4px rgba(0,0,0,0.1));
}

.modal {
  @include box-shadow(
    0 4px 8px rgba(0,0,0,0.2),
    0 8px 16px rgba(0,0,0,0.1)
  );
}

// 解包列表
$shadows: 0 2px 4px rgba(0,0,0,0.1), 0 4px 8px rgba(0,0,0,0.1);
.element {
  @include box-shadow($shadows...);
}
```

### 4. **内容块（@content）**

```scss
// 传递内容块
@mixin media-mobile {
  @media (max-width: 768px) {
    @content;
  }
}

@mixin media-desktop {
  @media (min-width: 1024px) {
    @content;
  }
}

.container {
  width: 100%;
  padding: 10px;
  
  @include media-mobile {
    padding: 5px;
  }
  
  @include media-desktop {
    max-width: 1200px;
    margin: 0 auto;
  }
}

// 主题切换
@mixin theme($theme-colors) {
  @each $name, $color in $theme-colors {
    &.theme-#{$name} {
      @content;
      background-color: $color;
    }
  }
}

.card {
  @include theme((
    light: #fff,
    dark: #333
  )) {
    border-radius: 8px;
  }
}
```

### 5. **常用实用混合**

```scss
// 1. 文本省略
@mixin text-ellipsis {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

// 2. 清除浮动
@mixin clearfix {
  &::after {
    content: "";
    display: table;
    clear: both;
  }
}

// 3. 绝对居中
@mixin absolute-center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

// 4. 隐藏元素
@mixin hide-text {
  text-indent: -9999px;
  overflow: hidden;
}

// 5. 视网膜屏幕图片
@mixin image-2x($image, $width, $height) {
  background-image: url($image);
  background-size: $width $height;
  
  @media (-webkit-min-device-pixel-ratio: 2),
         (min-resolution: 192dpi) {
    background-image: url("#{$image}@2x.png");
  }
}

// 使用
.title {
  @include text-ellipsis;
  max-width: 200px;
}

.clearfix {
  @include clearfix;
}
```

---

## 五、继承（@extend）

### 1. **基本用法**

```scss
// 定义占位符（不会编译到 CSS）
%button-base {
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

// 继承
.btn-primary {
  @extend %button-base;
  background-color: #3498db;
  color: white;
}

.btn-danger {
  @extend %button-base;
  background-color: #e74c3c;
  color: white;
}

// 编译后
.btn-primary, .btn-danger {
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.btn-primary {
  background-color: #3498db;
  color: white;
}

.btn-danger {
  background-color: #e74c3c;
  color: white;
}
```

### 2. **继承 vs 混合**

```scss
// 使用 @extend（生成更少的 CSS）
%alert-base {
  padding: 10px;
  border-radius: 4px;
  margin-bottom: 10px;
}

.alert-success {
  @extend %alert-base;
  background: #d4edda;
}

.alert-danger {
  @extend %alert-base;
  background: #f8d7da;
}

// 使用 @mixin（生成更多 CSS，但支持参数）
@mixin alert-base {
  padding: 10px;
  border-radius: 4px;
  margin-bottom: 10px;
}

.alert-success {
  @include alert-base;
  background: #d4edda;
}

.alert-danger {
  @include alert-base;
  background: #f8d7da;
}
```

**选择建议：**
- 相同样式，不需要参数 → 使用 `@extend`
- 需要参数或内容块 → 使用 `@mixin`

---

## 六、导入与模块化（@import / @use）

### 1. **传统 @import（已弃用）**

```scss
// 导入 SCSS 文件（不会发起 HTTP 请求）
@import 'variables';
@import 'mixins';
@import 'base/reset';
@import 'components/buttons';

// 编译时会合并到一个 CSS 文件
```

### 2. **现代 @use（推荐）**

```scss
// 导入并使用命名空间
@use 'variables' as vars;
@use 'mixins';

.button {
  background: vars.$primary-color;
  @include mixins.button-style;
}

// 自定义命名空间
@use 'variables' as *; // 不带命名空间

.card {
  background: $primary-color; // 直接使用
}
```

### 3. **@forward（转发）**

```scss
// _theme.scss
@forward 'variables';
@forward 'mixins';

// main.scss
@use 'theme'; // 同时导入 variables 和 mixins
```

### 4. **文件组织结构**

```
src/
├── styles/
│   ├── _variables.scss      // 变量
│   ├── _mixins.scss         // 混合
│   ├── _functions.scss      // 函数
│   ├── _reset.scss          // 重置样式
│   ├── _base.scss           // 基础样式
│   ├── components/
│   │   ├── _buttons.scss
│   │   ├── _cards.scss
│   │   └── _forms.scss
│   ├── layouts/
│   │   ├── _header.scss
│   │   ├── _footer.scss
│   │   └── _sidebar.scss
│   └── main.scss            // 主入口
```

```scss
// main.scss
@use 'variables' as *;
@use 'mixins' as *;
@use 'reset';
@use 'base';
@use 'components/buttons';
@use 'components/cards';
@use 'layouts/header';
@use 'layouts/footer';
```

---

## 七、函数（Functions）

### 1. **内置函数**

```scss
// 颜色函数
$color: #3498db;

.lighter { color: lighten($color, 20%); }      // 变浅
.darker { color: darken($color, 20%); }        // 变深
.saturated { color: saturate($color, 30%); }   // 增加饱和度
.desaturated { color: desaturate($color, 30%); } // 减少饱和度
.transparent { color: rgba($color, 0.5); }      // 调整透明度

// 数字函数
$number: 10.5;

.ceiling { value: ceil($number); }     // 11
.floor { value: floor($number); }      // 10
.round { value: round($number); }      // 11
.abs { value: abs(-10); }              // 10
.min { value: min(10px, 20px); }       // 10px
.max { value: max(10px, 20px); }       // 20px
.percentage { value: percentage(0.5); } // 50%

// 字符串函数
$string: "Hello World";

.to-upper { value: to-upper-case($string); } // "HELLO WORLD"
.to-lower { value: to-lower-case($string); } // "hello world"
.str-length { value: str-length($string); }  // 11
.str-index { value: str-index($string, "W"); } // 7

// 列表函数
$list: 10px, 20px, 30px;

.length { value: length($list); }       // 3
.nth { value: nth($list, 2); }          // 20px
.join { value: join($list, 40px); }     // 10px, 20px, 30px, 40px
.append { value: append($list, 40px); } // 10px, 20px, 30px, 40px

// Map 函数
$map: (primary: #3498db, success: #2ecc71);

.map-get { value: map-get($map, primary); }     // #3498db
.map-keys { value: map-keys($map); }            // primary, success
.map-values { value: map-values($map); }        // #3498db, #2ecc71
.map-merge { value: map-merge($map, (warning: #f39c12)); }

// 类型检查函数
@if type-of($value) == number { ... }
@if type-of($value) == color { ... }
@if type-of($value) == string { ... }
@if unitless($value) { ... }  // 无单位
```

### 2. **自定义函数**

```scss
// 1. 计算 REM
@function px-to-rem($px) {
  @return $px / 16px * 1rem;
}

.element {
  font-size: px-to-rem(16px); // 1rem
  padding: px-to-rem(32px);   // 2rem
}

// 2. 网格计算
@function grid-width($n) {
  @return $n * 60px + ($n - 1) * 20px;
}

.col-1 { width: grid-width(1); } // 60px
.col-2 { width: grid-width(2); } // 140px
.col-3 { width: grid-width(3); } // 220px

// 3. 颜色主题
@function theme-color($key) {
  $colors: (
    primary: #3498db,
    success: #2ecc71,
    warning: #f39c12,
    danger: #e74c3c
  );
  @return map-get($colors, $key);
}

.button {
  background: theme-color('primary');
}

.alert {
  background: theme-color('danger');
}

// 4. 数学计算
@function golden-ratio($value, $steps: 1) {
  $golden: 1.618;
  @return $value * pow($golden, $steps);
}

.small { font-size: golden-ratio(12px, -1); }
.base { font-size: golden-ratio(12px, 0); }
.large { font-size: golden-ratio(12px, 1); }
```

---

## 八、控制指令

### 1. **条件判断（@if / @else）**

```scss
@mixin button-size($size) {
  @if $size == small {
    padding: 5px 10px;
    font-size: 12px;
  } @else if $size == medium {
    padding: 10px 20px;
    font-size: 14px;
  } @else if $size == large {
    padding: 15px 30px;
    font-size: 18px;
  } @else {
    @error "Unknown size: #{$size}";
  }
}

.btn-small { @include button-size(small); }
.btn-medium { @include button-size(medium); }

// 类型检查
@mixin border($value) {
  @if type-of($value) == number {
    border: $value solid black;
  } @else if type-of($value) == color {
    border: 1px solid $value;
  } @else {
    @error "Invalid value: #{$value}";
  }
}
```

### 2. **循环（@for）**

```scss
// 包含终点值
@for $i from 1 through 3 {
  .col-#{$i} {
    width: percentage($i / 3);
  }
}
// 编译: .col-1 { width: 33.33333%; }
//      .col-2 { width: 66.66667%; }
//      .col-3 { width: 100%; }

// 不包含终点值
@for $i from 1 to 4 {
  .margin-#{$i} {
    margin: #{$i * 10}px;
  }
}
// 编译: .margin-1 { margin: 10px; }
//      .margin-2 { margin: 20px; }
//      .margin-3 { margin: 30px; }

// 网格系统
$grid-columns: 12;
@for $i from 1 through $grid-columns {
  .col-#{$i} {
    width: percentage($i / $grid-columns);
  }
}
```

### 3. **遍历列表（@each）**

```scss
// 简单列表
$colors: red, green, blue;

@each $color in $colors {
  .text-#{$color} {
    color: $color;
  }
}
// 编译: .text-red { color: red; }
//      .text-green { color: green; }
//      .text-blue { color: blue; }

// Map 遍历
$social-colors: (
  facebook: #3b5998,
  twitter: #1da1f2,
  instagram: #e1306c,
  youtube: #ff0000
);

@each $name, $color in $social-colors {
  .btn-#{$name} {
    background-color: $color;
    
    &:hover {
      background-color: darken($color, 10%);
    }
  }
}

// 多变量列表
$icons: (success, check, green),
        (error, cross, red),
        (warning, exclamation, orange);

@each $name, $icon, $color in $icons {
  .icon-#{$name} {
    &::before { content: $icon; }
    color: $color;
  }
}
```

### 4. **while 循环**

```scss
$i: 6;
@while $i > 0 {
  .item-#{$i} { width: 10px * $i; }
  $i: $i - 2;
}
// 编译: .item-6 { width: 60px; }
//      .item-4 { width: 40px; }
//      .item-2 { width: 20px; }
```

---

## 九、运算符

### 1. **数学运算**

```scss
$base-width: 100px;

.container {
  width: $base-width + 50px;      // 150px
  height: 200px - 50px;           // 150px
  padding: 10px * 2;              // 20px
  margin: 100px / 2;              // 50px
  
  // 需要括号的情况
  font-size: (14px / 16px) * 1rem; // 0.875rem
}

// 兼容性：相同单位可以运算
.element {
  width: 100px + 2em;  // 100px + 2em（不同单位保留）
  width: 100px + 50px; // 150px（相同单位计算）
}
```

### 2. **关系运算符**

```scss
@mixin responsive-text($size) {
  @if $size >= 18px {
    font-weight: bold;
  } @else {
    font-weight: normal;
  }
  font-size: $size;
}
```

### 3. **逻辑运算符**

```scss
@mixin button-state($is-active, $is-disabled) {
  @if $is-active and not $is-disabled {
    background: green;
  } @else if $is-disabled {
    background: gray;
  } @else {
    background: blue;
  }
}
```

### 4. **颜色运算**

```scss
$color: #010203;

.element {
  color: $color + #040506;  // #050709（逐通道相加）
  color: $color * 2;        // #020406（逐通道相乘）
}
```

---

## 十、插值（Interpolation）

### 1. **变量插值**

```scss
$name: button;
$side: top;
$property: margin;

// 类名
.#{$name} {
  padding: 10px;
}

// 属性名
.element {
  #{$property}-#{$side}: 10px; // margin-top: 10px;
}

// 属性值
.element {
  background: url("/images/#{$name}.png");
}
```

### 2. **动态生成样式**

```scss
$breakpoints: (
  sm: 576px,
  md: 768px,
  lg: 992px,
  xl: 1200px
);

@each $name, $width in $breakpoints {
  @media (min-width: $width) {
    .hidden-#{$name} {
      display: none;
    }
    
    .visible-#{$name} {
      display: block;
    }
  }
}
```

---

## 十一、注释

### 1. **多行注释（保留到编译后）**

```scss
/*
 * 这是多行注释
 * 会保留到编译后的 CSS 中
 */
$primary-color: #3498db;
```

### 2. **单行注释（压缩时删除）**

```scss
// 这是单行注释
// 压缩模式下会被删除
$secondary-color: #2ecc71;
```

### 3. **重要注释（永远保留）**

```scss
/*!
 * 重要注释
 * 即使在压缩模式下也会保留
 * @version 1.0.0
 */
```

---

## 十二、你的项目中的 SCSS 示例

### 查看项目中的实际使用：


你的项目使用了 **SCSS** 和 **深度选择器**：

```scss
<style scoped lang="scss">
// lang="scss" 启用 SCSS 语法
// scoped 限制样式只作用于当前组件

.enquiries-form {
  // 嵌套语法
  :deep .el-form-item__label {
    // 深度选择器：穿透 scoped 限制
    text-align: right;
  }
}

.select-style {
  width: 100%;
}
</style>
```

### 深度选择器说明：

```scss
// Vue 2 (旧语法)
::v-deep .el-form-item__label { }

// Vue 3 (推荐语法)
:deep(.el-form-item__label) { }

// 你的项目使用的语法（兼容）
:deep .el-form-item__label { }
```

---

## 十三、最佳实践

### ✅ 推荐做法：

```scss
// 1. 使用变量统一管理
:root {
  --primary-color: #3498db;
}

$primary-color: #3498db;

// 2. 使用混合复用代码
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

// 3. 使用占位符继承
%card-base {
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

// 4. 限制嵌套深度（最多3-4层）
.card {
  .header {
    .title {
      // ✅ 最多3层
    }
  }
}

// 5. 使用模块化导入
@use 'variables' as vars;
@use 'mixins' as mixins;

// 6. 使用函数处理单位
@function px-to-rem($px) {
  @return $px / 16px * 1rem;
}
```

### ❌ 避免做法：

```scss
// ❌ 嵌套过深（选择器地狱）
.page {
  .container {
    .row {
      .col {
        .card {
          .header {
            .title { } // 太深了！
          }
        }
      }
    }
  }
}

// ❌ 过度使用 @extend
.button { padding: 10px; }
.btn-primary { @extend .button; }
.btn-danger { @extend .button; }
// 会生成: .button, .btn-primary, .btn-danger { }

// ❌ 硬编码颜色和尺寸
.button { background: #3498db; } // 应使用变量

// ❌ 使用 @import（已弃用）
@import 'variables'; // 应使用 @use
```

---

## 十四、编译配置

### Vue CLI 项目（vue.config.js）：

```javascript
module.exports = {
  css: {
    loaderOptions: {
      sass: {
        // 全局注入变量（无需每个文件都导入）
        additionalData: `
          @use "@/styles/variables.scss" as *;
          @use "@/styles/mixins.scss" as *;
        `
      }
    }
  }
}
```

### Vite 项目（vite.config.js）：

```javascript
export default {
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `
          @use "@/styles/variables.scss" as *;
          @use "@/styles/mixins.scss" as *;
        `
      }
    }
  }
}
```

---

## 十五、总结对比表

| 特性       | CSS                   | SCSS              | 优势         |
| ---------- | --------------------- | ----------------- | ------------ |
| **变量**   | ❌ 无（只有 CSS 变量） | ✅ `$variable`     | 统一管理     |
| **嵌套**   | ❌ 不支持              | ✅ 支持            | 代码组织更好 |
| **混合**   | ❌ 不支持              | ✅ `@mixin`        | 代码复用     |
| **继承**   | ❌ 不支持              | ✅ `@extend`       | 减少重复     |
| **函数**   | ❌ 有限                | ✅ 强大            | 灵活计算     |
| **循环**   | ❌ 不支持              | ✅ `@for/@each`    | 动态生成     |
| **条件**   | ❌ 不支持              | ✅ `@if/@else`     | 逻辑控制     |
| **模块化** | ⚠️ 有限                | ✅ `@use/@forward` | 更好组织     |

---

## 学习资源

- 📚 **官方文档**: https://sass-lang.com/documentation
- 🎮 **在线练习**: https://www.sassmeister.com/
- 🛠️ **编译工具**: `sass`, `node-sass`, `Dart Sass`

**SCSS 是现代前端开发的必备技能，掌握它能大幅提升 CSS 开发效率！** 🚀

# Less 预处理器

## 一、Less 概述

**Less（Leaner Style Sheets）** 是 CSS 预处理器，扩展了 CSS 的功能。

### Less vs SCSS 对比：

| 特性           | Less         | SCSS/SASS         |
| -------------- | ------------ | ----------------- |
| **文件扩展名** | `.less`      | `.scss` / `.sass` |
| **语法**       | 类 CSS       | 类 CSS / 缩进式   |
| **变量符号**   | `@variable`  | `$variable`       |
| **运行环境**   | JavaScript   | Ruby / Dart       |
| **浏览器支持** | 可客户端编译 | 需预编译          |
| **流行度**     | ⭐⭐⭐          | ⭐⭐⭐⭐⭐             |

### 为什么选择 Less？

- ✅ 语法简单，学习成本低
- ✅ 可以直接在浏览器中运行（不推荐生产环境）
- ✅ Ant Design 使用 Less
- ✅ 与 CSS 兼容

---

## 二、变量（Variables）

### 1. **基本用法**

```less
// 定义变量（使用 @）
@primary-color: #3498db;
@font-size-base: 16px;
@spacing: 16px;
@border-radius: 4px;

// 使用变量
.button {
  background-color: @primary-color;
  font-size: @font-size-base;
  padding: @spacing;
  border-radius: @border-radius;
}
```

### 2. **延迟加载（Lazy Loading）**

```less
@var: red;

.class {
  @var: white;
  color: @var; // white（使用最近作用域的变量）
}

.class2 {
  color: @var; // red
}
```

### 3. **变量插值**

```less
@name: button;
@side: top;

.@{name} {
  border-@{side}-radius: 4px;
  margin-@{side}: 10px;
}

// 编译后
.button {
  border-top-radius: 4px;
  margin-top: 10px;
}
```

### 4. **变量作为属性名**

```less
@property: background;
@value: blue;

.element {
  @{property}-color: @value;
}

// 编译后
.element {
  background-color: blue;
}
```

### 5. **变量作为选择器**

```less
@selector: card;

.@{selector} {
  padding: 20px;
  
  &__@{selector}-header {
    font-size: 18px;
  }
}

// 编译后
.card {
  padding: 20px;
}
.card__card-header {
  font-size: 18px;
}
```

---

## 三、混合（Mixins）

### 1. **基本用法**

```less
// 定义混合（类似类选择器）
.border-radius {
  -webkit-border-radius: 4px;
  -moz-border-radius: 4px;
  border-radius: 4px;
}

// 使用混合
.button {
  .border-radius(); // 括号可省略
  padding: 10px 20px;
}

.card {
  .border-radius;
  background: white;
}
```

### 2. **带参数的混合**

```less
// 单个参数
.flex-center(@direction: row) {
  display: flex;
  flex-direction: @direction;
  justify-content: center;
  align-items: center;
}

.container {
  .flex-center(); // 使用默认值
}

.column-container {
  .flex-center(column); // 传参
}

// 多个参数
.size(@width; @height: @width) {
  width: @width;
  height: @height;
}

.square {
  .size(100px); // width: 100px; height: 100px;
}

.rectangle {
  .size(200px; 100px); // width: 200px; height: 100px;
}

// 命名参数
.mixin(@color: black; @margin: 10px; @padding: 20px) {
  color: @color;
  margin: @margin;
  padding: @padding;
}

.element {
  .mixin(@margin: 20px; @color: red); // 命名参数，顺序无关
}
```

### 3. **可变参数（...）**

```less
// 接受任意数量的参数
.box-shadow(@shadows...) {
  -webkit-box-shadow: @shadows;
  box-shadow: @shadows;
}

.card {
  .box-shadow(0 2px 4px rgba(0,0,0,0.1));
}

.modal {
  .box-shadow(
    0 4px 8px rgba(0,0,0,0.2),
    0 8px 16px rgba(0,0,0,0.1)
  );
}

// @rest 变量获取所有参数
.mixin(...) { }           // 接受 0 或多个参数
.mixin() { }              // 接受 0 个参数
.mixin(@a) { }            // 接受 1 个参数
.mixin(@a; @b) { }        // 接受 2 个参数
```

### 4. **模式匹配**

```less
// 根据参数数量匹配
.mixin(@a) {
  color: @a;
}

.mixin(@a; @b) {
  color: fade(@a; 50%);
  background: @b;
}

.element1 {
  .mixin(red); // 匹配第一个
}

.element2 {
  .mixin(red; blue); // 匹配第二个
}

// 根据值匹配
.mixin(dark; @color) {
  color: darken(@color; 20%);
}

.mixin(light; @color) {
  color: lighten(@color; 20%);
}

.mixin(@_; @color) { // 匹配任何值
  display: block;
}

.element1 {
  .mixin(dark; red); // 匹配第一个
}

.element2 {
  .mixin(light; red); // 匹配第二个
}
```

### 5. **守卫（Guards）**

```less
// 条件判断
.mixin(@a) when (lightness(@a) >= 50%) {
  background-color: black;
}

.mixin(@a) when (lightness(@a) < 50%) {
  background-color: white;
}

.mixin(@a) {
  color: @a;
}

.element1 {
  .mixin(#ddd); // lightness >= 50%, background: black
}

.element2 {
  .mixin(#333); // lightness < 50%, background: white
}

// 比较运算符
.mixin(@a) when (@a > 10) { }
.mixin(@a) when (@a < 10) { }
.mixin(@a) when (@a >= 10) { }
.mixin(@a) when (@a <= 10) { }
.mixin(@a) when (@a = 10) { }
.mixin(@a) when (@a /= 10) { } // 不等于

// 类型检查
.mixin(@a) when (isnumber(@a)) { }
.mixin(@a) when (isstring(@a)) { }
.mixin(@a) when (iscolor(@a)) { }
.mixin(@a) when (iskeyword(@a)) { }
.mixin(@a) when (isurl(@a)) { }

// 单位检查
.mixin(@a) when (ispixel(@a)) { }
.mixin(@a) when (ispercentage(@a)) { }
.mixin(@a) when (isem(@a)) { }

// 逻辑运算
.mixin(@a) when (@a > 10) and (@a < 20) { }
.mixin(@a) when (@a < 10) or (@a > 20) { }
.mixin(@a) when not (@a > 10) { }
```

### 6. **@rules 传递**

```less
// 传递整个规则集
.desktop-and-old-ie(@rules) {
  @media screen and (min-width: 1200px) {
    @rules();
  }
  
  html.lt-ie9 & {
    @rules();
  }
}

.header {
  background-color: blue;
  
  .desktop-and-old-ie({
    background-color: red;
  });
}
```

---

## 四、嵌套（Nesting）

### 1. **选择器嵌套**

```less
// Less
.navbar {
  background: #333;
  padding: 10px;
  
  .nav-item {
    display: inline-block;
    padding: 8px 16px;
    
    a {
      color: white;
      text-decoration: none;
      
      &:hover {
        color: #3498db;
      }
    }
  }
}

// 编译后的 CSS
.navbar {
  background: #333;
  padding: 10px;
}
.navbar .nav-item {
  display: inline-block;
  padding: 8px 16px;
}
.navbar .nav-item a {
  color: white;
  text-decoration: none;
}
.navbar .nav-item a:hover {
  color: #3498db;
}
```

### 2. **父选择器引用 `&`**

```less
// 伪类
.button {
  background: blue;
  
  &:hover {
    background: darkblue;
  }
  
  &:active {
    background: navy;
  }
}

// BEM 命名
.card {
  background: white;
  
  &__header {
    padding: 16px;
  }
  
  &__content {
    padding: 20px;
  }
  
  &--primary {
    background: blue;
  }
}

// 编译后
.card { background: white; }
.card__header { padding: 16px; }
.card__content { padding: 20px; }
.card--primary { background: blue; }
```

### 3. **属性嵌套**

```less
// Less
.card {
  border: {
    style: solid;
    width: 1px;
    color: #ddd;
    radius: 8px;
  }
  
  margin: {
    top: 10px;
    bottom: 20px;
  }
}

// 编译后的 CSS
.card {
  border-style: solid;
  border-width: 1px;
  border-color: #ddd;
  border-radius: 8px;
  margin-top: 10px;
  margin-bottom: 20px;
}
```

### 4. **@at-root（跳出嵌套）**

```less
.parent {
  color: blue;
  
  @at-root .child {
    color: red;
  }
}

// 编译后
.parent { color: blue; }
.child { color: red; }
```

---

## 五、运算（Operations）

### 1. **数学运算**

```less
@base-width: 100px;

.container {
  width: @base-width + 50px;      // 150px
  height: 200px - 50px;           // 150px
  padding: 10px * 2;              // 20px
  margin: 100px / 2;              // 50px
}

// 括号改变优先级
.element {
  width: (@base-width + 50px) * 2; // 300px
}
```

### 2. **颜色运算**

```less
@color: #010203;

.element {
  color: @color + #040506;  // #050709
  color: @color * 2;        // #020406
}
```

### 3. **变量运算**

```less
@margin: 10px;

.element {
  margin: @margin * 2;    // 20px
  padding: @margin + 5px; // 15px
}
```

---

## 六、函数（Functions）

### 1. **颜色函数**

```less
@color: #3498db;

.lighter { color: lighten(@color, 20%); }      // 变浅
.darker { color: darken(@color, 20%); }        // 变深
.saturate { color: saturate(@color, 30%); }    // 增加饱和度
.desaturate { color: desaturate(@color, 30%); } // 减少饱和度
.fade { color: fade(@color, 50%); }            // 调整透明度
.fadein { color: fadein(@color, 10%); }        // 增加不透明度
.fadeout { color: fadeout(@color, 10%); }      // 减少不透明度
.spin { color: spin(@color, 10); }             // 旋转色相
.mix { color: mix(@color, #fff, 50%); }        // 混合颜色

// 提取颜色通道
.red { color: red(@color); }     // R
.green { color: green(@color); } // G
.blue { color: blue(@color); }   // B
.alpha { color: alpha(@color); } // A
```

### 2. **数学函数**

```less
@number: 10.5;

.ceiling { value: ceil(@number); }      // 11
.floor { value: floor(@number); }       // 10
.percentage { value: percentage(0.5); } // 50%
.round { value: round(@number); }       // 11
.sqrt { value: sqrt(25); }              // 5
.abs { value: abs(-10); }               // 10
.sin { value: sin(1); }
.cos { value: cos(1); }
.tan { value: tan(1); }
.pow { value: pow(2; 3); }              // 8
```

### 3. **类型检查函数**

```less
.mixin(@a) when (isnumber(@a)) {
  width: @a;
}

.mixin(@a) when (iscolor(@a)) {
  color: @a;
}

.mixin(@a) when (isstring(@a)) {
  content: @a;
}

.mixin(@a) when (ispixel(@a)) {
  padding: @a;
}

.mixin(@a) when (ispercentage(@a)) {
  width: @a;
}
```

### 4. **字符串函数**

```less
@string: "Hello World";

.to-upper { value: to-upper-case(@string); }
.to-lower { value: to-lower-case(@string); }
.escape { value: escape(@string); }
.e { value: e(@string); } // 去掉引号
```

### 5. **列表函数**

```less
@list: 10px 20px 30px;

.length { value: length(@list); }      // 3
.extract { value: extract(@list; 2); } // 20px
```

---

## 七、命名空间与作用域

### 1. **命名空间**

```less
// 定义命名空间
#bundle() {
  .button {
    padding: 10px 20px;
    border-radius: 4px;
  }
  
  .tab {
    display: inline-block;
  }
}

// 使用命名空间
.header {
  #bundle > .button;
}

.footer {
  #bundle > .tab;
}
```

### 2. **作用域**

```less
@var: red;

.class {
  @var: white;
  color: @var; // white（局部作用域优先）
}

.class2 {
  color: @var; // red（全局作用域）
}
```

---

## 八、导入（Import）

### 1. **基本导入**

```less
// 导入 Less 文件
@import "variables.less";
@import "mixins.less";
@import "base/reset.less";

// 导入 CSS 文件（不会处理，直接输出）
@import "library.css";

// 条件导入
@import (optional) "theme.less"; // 文件不存在时不报错
@import (multiple) "utils.less"; // 允许多次导入
```

### 2. **导入选项**

```less
// reference：导入但不输出
@import (reference) "variables.less";

// inline：内联输出，不处理
@import (inline) "library.css";

// less：强制作为 Less 处理
@import (less) "library.css";

// css：强制作为 CSS 输出
@import (css) "library.less";

// once：只导入一次（默认）
@import (once) "utils.less";

// multiple：允许多次导入
@import (multiple) "utils.less";
```

### 3. **文件组织结构**

```
src/
├── styles/
│   ├── variables.less      // 变量
│   ├── mixins.less         // 混合
│   ├── reset.less          // 重置样式
│   ├── base.less           // 基础样式
│   ├── components/
│   │   ├── buttons.less
│   │   ├── cards.less
│   │   └── forms.less
│   ├── layouts/
│   │   ├── header.less
│   │   ├── footer.less
│   │   └── sidebar.less
│   └── main.less           // 主入口
```

```less
// main.less
@import 'variables.less';
@import 'mixins.less';
@import 'reset.less';
@import 'base.less';
@import 'components/buttons.less';
@import 'components/cards.less';
@import 'layouts/header.less';
@import 'layouts/footer.less';
```

---

## 九、转义（Escaping）

### 1. **~"value" 语法**

```less
.element {
  filter: ~"ms:alwaysHasItsOwnSyntax.For.Stuff()";
  width: ~"calc(300px - 20px)";
}

// 编译后
.element {
  filter: ms:alwaysHasItsOwnSyntax.For.Stuff();
  width: calc(300px - 20px);
}
```

### 2. **~`expression` 语法（JavaScript 表达式）**

```less
@var: ~`"hello".toUpperCase() + "!"`;

.element {
  content: @var; // "HELLO!"
}
```

---

## 十、合并属性

### 1. **逗号合并（+）**

```less
.mixin() {
  box-shadow+: 0 2px 4px rgba(0,0,0,0.1);
}

.element {
  .mixin();
  box-shadow+: 0 4px 8px rgba(0,0,0,0.2);
}

// 编译后
.element {
  box-shadow: 0 2px 4px rgba(0,0,0,0.1), 0 4px 8px rgba(0,0,0,0.2);
}
```

### 2. **空格合并（+_）**

```less
.mixin() {
  transform+_: scale(1.2);
}

.element {
  .mixin();
  transform+_: rotate(45deg);
}

// 编译后
.element {
  transform: scale(1.2) rotate(45deg);
}
```

---

## 十一、父选择器高级用法

### 1. **改变连接符**

```less
.button {
  &-primary {
    background: blue;
  }
  
  &-danger {
    background: red;
  }
}

// 编译后
.button-primary { background: blue; }
.button-danger { background: red; }
```

### 2. **多个 &**

```less
.link {
  &,
  &:hover {
    color: blue;
  }
}

// 编译后
.link, .link:hover {
  color: blue;
}
```

### 3. **条件父选择器**

```less
.element {
  .parent & {
    color: red;
  }
}

// 编译后
.parent .element {
  color: red;
}
```

---

## 十二、实用混合示例

### 1. **文本省略**

```less
.text-ellipsis() {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.title {
  .text-ellipsis();
  max-width: 200px;
}
```

### 2. **清除浮动**

```less
.clearfix() {
  &::after {
    content: "";
    display: table;
    clear: both;
  }
}

.container {
  .clearfix();
}
```

### 3. **绝对居中**

```less
.absolute-center() {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.modal {
  .absolute-center();
}
```

### 4. **响应式断点**

```less
@breakpoint-sm: 576px;
@breakpoint-md: 768px;
@breakpoint-lg: 992px;
@breakpoint-xl: 1200px;

.media-mobile(@rules) {
  @media (max-width: @breakpoint-md) {
    @rules();
  }
}

.media-desktop(@rules) {
  @media (min-width: @breakpoint-lg) {
    @rules();
  }
}

.container {
  width: 100%;
  
  .media-mobile({
    padding: 10px;
  });
  
  .media-desktop({
    max-width: 1200px;
    margin: 0 auto;
  });
}
```

---

## 十三、你的项目中使用 Less

### 查看项目中是否有 Less 使用：


你的项目当前没有使用 `.less` 文件，使用的是 **SCSS**（`.scss`）。

### 如果需要使用 Less，配置方式：

#### Vue CLI 项目：

```javascript
// vue.config.js
module.exports = {
  css: {
    loaderOptions: {
      less: {
        lessOptions: {
          javascriptEnabled: true,
          modifyVars: {
            'primary-color': '#3498db',
          }
        }
      }
    }
  }
}
```

#### 组件中使用：

```vue
<style scoped lang="less">
@primary-color: #3498db;

.container {
  background: @primary-color;
  
  .header {
    padding: 20px;
  }
}
</style>
```

---

## 十四、Less vs SCSS 对比总结

| 特性         | Less           | SCSS                  |
| ------------ | -------------- | --------------------- |
| **变量符号** | `@variable`    | `$variable`           |
| **混合定义** | `.mixin() { }` | `@mixin mixin() { }`  |
| **混合调用** | `.mixin()`     | `@include mixin()`    |
| **继承**     | `:extend()`    | `@extend`             |
| **守卫**     | `when ()`      | `@if`                 |
| **导入**     | `@import`      | `@use` / `@import`    |
| **循环**     | ❌ 不支持       | ✅ `@for/@each`        |
| **Map**      | ❌ 不支持       | ✅ 支持                |
| **生态系统** | Ant Design     | Bootstrap, Vue, React |

---

## 十五、最佳实践

### ✅ 推荐做法：

```less
// 1. 使用变量统一管理
@primary-color: #3498db;
@spacing: 16px;

// 2. 使用混合复用代码
.flex-center(@direction: row) {
  display: flex;
  flex-direction: @direction;
  justify-content: center;
  align-items: center;
}

// 3. 限制嵌套深度（最多3-4层）
.card {
  .header {
    .title {
      // ✅ 最多3层
    }
  }
}

// 4. 使用命名空间组织
#theme() {
  .colors() {
    @primary: #3498db;
  }
}

// 5. 使用守卫实现条件样式
.mixin(@color) when (lightness(@color) > 50%) {
  background-color: black;
}
```

### ❌ 避免做法：

```less
// ❌ 嵌套过深
.page {
  .container {
    .row {
      .col {
        .card {
          .header {
            .title { } // 太深了！
          }
        }
      }
    }
  }
}

// ❌ 硬编码颜色和尺寸
.button { background: #3498db; } // 应使用变量

// ❌ 过度使用 JavaScript 表达式
@var: ~`Math.random()`; // 不可预测
```

---

## 十六、安装与编译

### 1. **安装 Less**

```bash
npm install less --save-dev
npm install less-loader --save-dev  # Webpack 项目
```

### 2. **命令行编译**

```bash
# 单个文件
lessc styles.less styles.css

# 压缩输出
lessc styles.less styles.css --compress

# 监听模式
lessc styles.less styles.css --watch
```

### 3. **Webpack 配置**

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.less$/,
        use: [
          'style-loader',
          'css-loader',
          'less-loader'
        ]
      }
    ]
  }
}
```

---

## 总结

| 项目需求            | 推荐选择                 |
| ------------------- | ------------------------ |
| **新项目**          | SCSS（更强大，生态更好） |
| **使用 Ant Design** | Less（组件库使用 Less）  |
| **需要循环/Map**    | SCSS                     |
| **简单项目**        | Less（语法更简单）       |
| **团队协作**        | SCSS（更流行）           |

**你的项目使用 SCSS，这是更主流的选择！** ✅

如果需要迁移到 Less，主要改动：
- `$variable` → `@variable`
- `@mixin` → `.mixin()`
- `@include` → `.mixin()`
- `@for/@each` → 需要使用递归混合替代

# Note

## CSS 多 class 的原因

在 Vue/HTML 中，一个元素可以拥有**多个 class**，这是非常常见的做法。让我解释为什么这里使用了 `btn` 和 `btnRest` 两个 class：

```vue
<el-button type="" plain class="btn btnRest" @click="resetForm">
  {{ $t("button.reset") }}
</el-button>
```

---

### 一、多 class 的作用

#### 1. **样式复用（BEM/模块化思想）**

```html
<!-- btn: 通用按钮样式 -->
<!-- btnRest: 重置按钮的特定样式 -->
class="btn btnRest"
```

**通用样式作用：**

- ✅ 统一所有按钮的最小宽度
- ✅ 保持按钮尺寸一致性
- ✅ 复用通用样式，避免重复代码

**CSS 定义可能如下：**

```scss
// 通用按钮样式（所有按钮共享）
.btn {
  padding: 8px 16px;
  border-radius: 4px;
  font-size: 14px;
  cursor: pointer;
  border: 1px solid #ddd;
  transition: all 0.3s;
}

// 重置按钮特定样式
.btnRest {
  background-color: #f5f5f5;
  color: #666;
  margin-right: 10px;
  
  &:hover {
    background-color: #e8e8e8;
  }
}

// 搜索按钮特定样式
.btnSearch {
  background-color: #409eff;
  color: white;
  
  &:hover {
    background-color: #66b1ff;
  }
}
```



---

### 二、CSS 多 class 的常见模式

#### 模式 1：基础类 + 变体类

```html
<!-- 基础样式 -->
<button class="btn">按钮</button>

<!-- 基础 + 尺寸变体 -->
<button class="btn btn-sm">小按钮</button>
<button class="btn btn-lg">大按钮</button>

<!-- 基础 + 颜色变体 -->
<button class="btn btn-primary">主要</button>
<button class="btn btn-danger">危险</button>
<button class="btn btn-success">成功</button>
```

```scss
// 基础样式
.btn {
  padding: 8px 16px;
  border-radius: 4px;
  font-size: 14px;
}

// 尺寸变体
.btn-sm {
  padding: 4px 8px;
  font-size: 12px;
}

.btn-lg {
  padding: 12px 24px;
  font-size: 16px;
}

// 颜色变体
.btn-primary {
  background: #409eff;
  color: white;
}

.btn-danger {
  background: #f56c6c;
  color: white;
}
```

---

#### 模式 2：组件类 + 功能类

```html
<!-- 你的项目中的用法 -->
<button class="btn btnRest">重置</button>
<button class="btn btnSearch">搜索</button>

<!-- 其他示例 -->
<div class="card card-highlight">
<div class="modal modal-large">
<div class="table table-striped">
```

---

#### 模式 3：布局类 + 样式类

```html
<!-- 布局 + 样式 -->
<div class="flex justify-center items-center">
<div class="grid grid-cols-3 gap-4">

<!-- 间距 + 颜色 -->
<div class="mt-4 p-4 bg-blue-500 text-white">
```

---

### 三、多 class 的优势

#### ✅ 优势：

```html
<!-- 1. 样式复用 -->
<button class="btn btn-primary btn-lg">大按钮</button>
<button class="btn btn-primary btn-sm">小按钮</button>
<!-- 共享 btn 和 btn-primary，只有尺寸不同 -->

<!-- 2. 语义清晰 -->
<button class="btn btnRest">重置</button>
<!-- 一看就知道是重置按钮 -->

<!-- 3. 易于维护 -->
<!-- 修改 .btn 会影响所有按钮 -->
<!-- 修改 .btnRest 只影响重置按钮 -->

<!-- 4. 减少代码重复 -->
<!-- 不需要为每个按钮写完整样式 -->
```

#### ❌ 避免：

```html
<!-- ❌ 不要过度使用 -->
<button class="btn btn-primary btn-large btn-rounded btn-shadow btn-hover">

<!-- ✅ 保持简洁 -->
<button class="btn btn-primary btn-lg">
```

---

### 四、总结

```vue
class="btn btnRest"
```

**含义：**
- **`btn`**: 通用按钮样式（`min-width: 80px`），所有按钮共享
- **`btnRest`**: 重置按钮的标识（语义化命名），便于区分和操作

**设计思想：**
- 🎯 **DRY 原则**（Don't Repeat Yourself）：通用样式提取到 `.btn`
- 🏷️ **语义化命名**：`.btnRest` 清晰表达按钮用途
- 🔧 **易于扩展**：未来可以为 `.btnRest` 添加特定样式
- 🧪 **便于测试**：可以通过 class 名称定位特定按钮

这是 **CSS 模块化** 和 **BEM 命名规范** 的常见实践！✅

## 全局/局部导入

### 一、全局导入 vs 局部导入

#### 1. **全局导入（Global Import）**

样式在所有组件中都可用，无需在每个组件中单独导入。

#### 2. **局部导入（Local Import）**

样式只在当前组件中可用，需要在每个使用的组件中导入。

---

### 二、全局导入方式

#### 方式 1：在 `main.js` 中导入 ✅ 推荐

```javascript
// src/main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

// 全局样式导入
import '@/styles/global.scss'           // 全局样式
import '@/styles/variables.scss'        // 全局变量
import '@/styles/mixins.scss'           // 全局混合
import 'element-plus/dist/index.css'    // Element Plus 样式

const app = createApp(App)
app.use(router)
app.mount('#app')
```

**优点：**

- ✅ 一次导入，全局可用
- ✅ 所有组件都可以使用
- ✅ 便于统一管理

**缺点：**

- ⚠️ 增加打包体积
- ⚠️ 可能产生命名冲突

---

#### 方式 2：在 `vue.config.js` 中自动注入（仅 SCSS/Less 变量和混合）

```javascript
// vue.config.js
module.exports = {
  css: {
    loaderOptions: {
      scss: {
        // 自动注入到每个 .vue 文件的 <style lang="scss"> 中
        additionalData: `
          @use "@/styles/variables.scss" as *;
          @use "@/styles/mixins.scss" as *;
        `
      },
      // Less 配置
      less: {
        lessOptions: {
          javascriptEnabled: true
        },
        additionalData: `
          @import "@/styles/variables.less";
          @import "@/styles/mixins.less";
        `
      }
    }
  }
}
```

**使用示例：**

```vue
<template>
  <div class="container">
    <h1 class="title">标题</h1>
  </div>
</template>

<!-- 无需导入，直接使用变量和混合 -->
<style scoped lang="scss">
.container {
  max-width: $max-width;  // ✅ 直接使用变量，无需 @use
  
  .title {
    @include flex-center; // ✅ 直接使用混合，无需 @use
    color: $primary-color;
  }
}
</style>
```

**优点：**

- ✅ 变量和混合全局可用
- ✅ 不会增加 CSS 体积（只有变量和混合，没有实际样式）
- ✅ 避免重复导入

**缺点：**

- ⚠️ 只能注入变量和混合，不能注入实际样式
- ⚠️ 配置稍复杂

---

#### 方式 3：在 `App.vue` 中导入

```vue
<!-- src/App.vue -->
<template>
  <router-view />
</template>

<style lang="scss">
/* 全局样式 */
@import '@/styles/global.scss';
@import '@/styles/reset.scss';
@import '@/styles/typography.scss';

/* 全局重置 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}
</style>
```

---

### 三、局部导入方式

#### 方式 1：在组件的 `<style>` 中导入

```vue
<template>
  <div class="my-component">
    <h1>局部样式组件</h1>
  </div>
</template>

<script>
export default {
  name: 'MyComponent'
}
</script>

<!-- 局部样式（不带 scoped） -->
<style lang="scss">
@import '@/styles/special.scss'; // 只在当前组件生效

.my-component {
  padding: 20px;
}
</style>
```

#### 方式 2：带 `scoped` 的局部样式 ✅ 推荐

```vue
<template>
  <div class="my-component">
    <h1 class="title">标题</h1>
  </div>
</template>

<!-- scoped 限制样式只作用于当前组件 -->
<style scoped lang="scss">
@import '@/styles/utils.scss'; // 局部导入

.my-component {
  .title {
    color: $primary-color; // 需要额外导入变量
    font-size: 18px;
  }
}
</style>
```

#### 方式 3：模块化样式（CSS Modules）

```vue
<template>
  <div :class="$style.container">
    <h1 :class="$style.title">标题</h1>
  </div>
</template>

<style module lang="scss">
.container {
  padding: 20px;
}

.title {
  color: blue;
}
</style>
```

---

### 四、完整对比表

| 特性         | 全局导入              | 局部导入            |
| ------------ | --------------------- | ------------------- |
| **作用范围** | 所有组件              | 当前组件            |
| **导入位置** | `main.js` / `App.vue` | 组件 `<style>` 标签 |
| **打包体积** | 较大（全部打包）      | 较小（按需打包）    |
| **命名冲突** | 可能冲突              | 不会冲突（scoped）  |
| **维护性**   | 统一管理              | 分散管理            |
| **适用场景** | 全局样式、主题、重置  | 组件专属样式        |

---

### 五、实际项目中的使用策略

#### 📁 推荐的文件组织结构：

```
src/
├── styles/
│   ├── _variables.scss      # 全局变量（通过 vue.config.js 注入）
│   ├── _mixins.scss         # 全局混合（通过 vue.config.js 注入）
│   ├── index.scss           # 全局样式（在 main.js 导入）
│   ├── cmi.scss             # CMI 主题样式（在 main.js 导入）
│   ├── reset.scss           # 重置样式
│   └── components/
│       ├── _buttons.scss    # 按钮组件样式
│       └── _cards.scss      # 卡片组件样式
├── views/
│   └── paymentManage/
│       ├── pmsLineBaseDataQuery.vue  # 使用 scoped 局部样式
│       └── erpApplicationQuery.vue   # 使用 scoped 局部样式
└── main.js                  # 全局导入入口
```

---

### 六、不同场景的使用方式

#### 场景 1：全局变量和混合

**配置（vue.config.js）：**

```javascript
module.exports = {
  css: {
    loaderOptions: {
      scss: {
        additionalData: `
          @use "@/styles/variables.scss" as *;
          @use "@/styles/mixins.scss" as *;
        `
      }
    }
  }
}
```

**使用（任意组件）：**

```vue
<style scoped lang="scss">
// ✅ 无需导入，直接使用
.container {
  max-width: $max-width;
  
  @include flex-center;
  
  .title {
    color: $primary-color;
  }
}
</style>
```

---

#### 场景 2：全局样式（重置、主题）

**在 main.js 中导入：**

```javascript
// src/main.js
import '@/styles/index.scss'    // 全局样式
import '@/styles/cmi.scss'      // 主题样式
```

**效果：**

```vue
<!-- 任意组件，无需导入 -->
<template>
  <div class="component-table">  <!-- ✅ 全局样式自动生效 -->
    <el-table :data="tableData">
      <!-- ... -->
    </el-table>
  </div>
</template>

<style scoped lang="scss">
/* 只需写组件专属样式 */
.my-custom-style {
  padding: 10px;
}
</style>
```

---

#### 场景 3：组件专属样式

```vue
<template>
  <div class="pms-line-query">
    <el-form :model="form">
      <!-- ... -->
    </el-form>
  </div>
</template>

<style scoped lang="scss">
// 只在当前组件生效
.pms-line-query {
  .enquiries-form {
    :deep .el-form-item__label {
      text-align: right;
    }
  }
  
  .select-style {
    width: 100%;
  }
}
</style>
```

---

#### 场景 4：共享组件样式

**创建共享样式文件：**

```scss
// src/styles/components/_buttons.scss
.shared-button {
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  
  &--primary {
    background: $primary-color;
    color: white;
  }
  
  &--danger {
    background: $danger-color;
    color: white;
  }
}
```

**在需要的组件中导入：**

```vue
<template>
  <button class="shared-button shared-button--primary">
    主要按钮
  </button>
</template>

<style scoped lang="scss">
@import '@/styles/components/buttons.scss';

// 组件专属样式
.custom-button {
  margin-top: 10px;
}
</style>
```

---

### 七、`scoped` 样式穿透

#### 问题：scoped 无法影响子组件样式

```vue
<template>
  <el-form>
    <el-form-item label="名称">
      <el-input v-model="name" />
    </el-form-item>
  </el-form>
</template>

<!-- ❌ 无法生效 -->
<style scoped>
.el-form-item__label {
  text-align: right;
}
</style>
```

#### 解决方案：深度选择器

```vue
<!-- Vue 3 推荐语法 -->
<style scoped>
:deep(.el-form-item__label) {
  text-align: right;
}
</style>

<!-- 兼容语法（你的项目使用） -->
<style scoped>
:deep .el-form-item__label {
  text-align: right;
}
</style>

<!-- Vue 2 旧语法 -->
<style scoped>
::v-deep .el-form-item__label {
  text-align: right;
}
</style>
```




---

## 十、最佳实践建议

### 📋 推荐的导入策略：

```javascript
// 1. main.js - 全局样式
import '@/styles/index.scss'       // 全局基础样式
import '@/styles/reset.scss'       // 重置样式
import 'element-plus/dist/index.css'

// 2. vue.config.js - 自动注入变量和混合
module.exports = {
  css: {
    loaderOptions: {
      scss: {
        additionalData: `
          @use "@/styles/variables.scss" as *;
          @use "@/styles/mixins.scss" as *;
        `
      }
    }
  }
}

// 3. 组件 - scoped 局部样式
<style scoped lang="scss">
// ✅ 变量和混合自动可用（无需导入）
// ✅ 只需写组件专属样式
.container {
  max-width: $max-width;
  
  @include flex-center;
}
</style>
```

---

## 十一、完整使用示例

### 示例 1：全局样式 + 局部样式

```vue
<template>
  <div class="pms-line-query">
    <!-- 使用全局样式 -->
    <div class="component-table">  <!-- ✅ 来自 index.scss -->
      <el-table :data="tableData">
        <!-- ... -->
      </el-table>
    </div>
    
    <!-- 使用局部样式 -->
    <div class="custom-section">
      <h3>自定义区域</h3>
    </div>
  </div>
</template>

<style scoped lang="scss">
// 组件专属样式
.pms-line-query {
  .custom-section {
    padding: 20px;
    background: #f5f5f5;
    
    h3 {
      color: $primary-color; // 需要配置 additionalData 才能直接用
      font-size: 18px;
    }
  }
}
</style>
```

---

### 示例 2：样式穿透（修改 Element Plus 样式）

```vue
<template>
  <el-form :model="form" class="custom-form">
    <el-form-item label="名称">
      <el-input v-model="form.name" />
    </el-form-item>
  </el-form>
</template>

<style scoped lang="scss">
// ✅ 穿透 scoped，修改子组件样式
.custom-form {
  :deep(.el-form-item__label) {
    text-align: right;
    font-weight: bold;
  }
  
  :deep(.el-input__inner) {
    border-radius: 8px;
  }
}
</style>
```

---

## 十二、总结对比表

| 导入方式               | 配置位置              | 作用范围 | 适用内容             | 推荐度 |
| ---------------------- | --------------------- | -------- | -------------------- | ------ |
| **main.js 导入**       | `main.js`             | 全局     | 重置、主题、通用样式 | ⭐⭐⭐⭐⭐  |
| **vue.config.js 注入** | `vue.config.js`       | 全局     | 变量、混合           | ⭐⭐⭐⭐⭐  |
| **组件 scoped**        | 组件 `<style>`        | 当前组件 | 组件专属样式         | ⭐⭐⭐⭐⭐  |
| **组件非 scoped**      | 组件 `<style>`        | 全局     | 共享组件样式         | ⭐⭐⭐    |
| **CSS Modules**        | 组件 `<style module>` | 当前组件 | 需要隔离的样式       | ⭐⭐⭐⭐   |

---

