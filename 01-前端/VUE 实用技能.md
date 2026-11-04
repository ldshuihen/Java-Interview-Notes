# VUE3 问题

## Loading 效果

在 Vue 3 项目中，Loading 效果通常分为**全局 Loading** 和 **局部 Loading**。

*   **全局 Loading**：通常用于路由跳转、全局数据初始化、或者多个并发请求时的统一等待状态。它会覆盖整个屏幕，阻止用户操作。
*   **局部 Loading**：通常用于某个组件内部的数据加载（如表格刷新、按钮提交、卡片内容加载），只覆盖特定区域，不影响页面其他部分的交互。

下面分别介绍这两种 Loading 的实现方案。

---

### 一、 全局 Loading 实现

全局 Loading 通常需要一个**全局状态管理**来控制其显隐。可以使用 Pinia 或 Vue3 的响应式 API。

#### 1. 创建全局状态
这里我们不用 Pinia，直接用一个简单的响应式模块来管理，方便任何地方调用。

```typescript
// src/utils/loading.ts
import { ref } from 'vue'

// 全局 loading 状态
const isLoading = ref(false)
// loading 文案
const loadingText = ref('加载中...')

// 用来处理并发请求的计数器（防止多个请求同时发起和结束导致提前关闭 loading）
let requestCount = 0

export function showGlobalLoading(text = '加载中...') {
  requestCount++
  if (requestCount === 1) {
    isLoading.value = true
    loadingText.value = text
  }
}

export function hideGlobalLoading() {
  if (requestCount <= 0) return
  requestCount--
  if (requestCount === 0) {
    isLoading.value = false
  }
}

export function useGlobalLoading() {
  return {
    isLoading,
    loadingText,
    showGlobalLoading,
    hideGlobalLoading
  }
}
```

#### 2. 创建全局 Loading 组件
使用 `<Teleport>` 将组件传送到 `body`，确保它永远在最上层。

```vue
<!-- src/components/GlobalLoading.vue -->
<template>
  <Teleport to="body">
    <transition name="fade">
      <div v-if="isLoading" class="global-loading-mask">
        <div class="global-loading-spinner">
          <div class="loader"></div>
          <p>{{ loadingText }}</p>
        </div>
      </div>
    </transition>
  </Teleport>
</template>

<script setup lang="ts">
import { useGlobalLoading } from '@/utils/loading'

const { isLoading, loadingText } = useGlobalLoading()
</script>

<style scoped>
.global-loading-mask {
  position: fixed;
  inset: 0;
  background: rgba(255, 255, 255, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}
.loader {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #42b883;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
  margin-bottom: 10px;
}
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
</style>
```

#### 3. 在 App.vue 中挂载
```vue
<template>
  <div>
    <router-view />
    <!-- 全局挂载 -->
    <GlobalLoading />
  </div>
</template>

<script setup>
import GlobalLoading from '@/components/GlobalLoading.vue'
</script>
```

#### 4. 结合 Axios 拦截器使用
最常见的场景是在请求拦截器中自动触发：
```typescript
// src/utils/request.ts
import axios from 'axios'
import { showGlobalLoading, hideGlobalLoading } from './loading'

const service = axios.create()

// 请求拦截器
service.interceptors.request.use(config => {
  // 可以根据某个 header 判断是否需要全局 loading，例如 config.headers.noGlobalLoading
  showGlobalLoading()
  return config
})

// 响应拦截器
service.interceptors.response.use(
  response => {
    hideGlobalLoading()
    return response.data
  },
  error => {
    hideGlobalLoading()
    return Promise.reject(error)
  }
)
```

---

### 二、 局部 Loading 实现

局部 Loading 最优雅的方式是使用 **自定义指令**。这样可以在任何组件上通过 `v-loading="true"` 来实现，就像 Element Plus 那样。

#### 1. 创建局部 Loading 指令
```typescript
// src/directives/loading.ts
import type { Directive, DirectiveBinding } from 'vue'

// 创建一个局部的 loading DOM 元素
function createLoadingMask(text: string) {
  const mask = document.createElement('div')
  mask.className = 'local-loading-mask'
  mask.style.position = 'absolute'
  mask.style.zIndex = '999'
  mask.style.backgroundColor = 'rgba(255, 255, 255, 0.9)'
  mask.style.display = 'flex'
  mask.style.justifyContent = 'center'
  mask.style.alignItems = 'center'
  mask.style.flexDirection = 'column'
  mask.style.inset = '0'
  
  const spinner = document.createElement('div')
  spinner.className = 'local-loading-spinner'
  spinner.style.border = '3px solid #f3f3f3'
  spinner.style.borderTop = '3px solid #42b883'
  spinner.style.borderRadius = '50%'
  spinner.style.width = '30px'
  spinner.style.height = '30px'
  spinner.style.animation = 'spin 1s linear infinite'
  
  const textNode = document.createElement('span')
  textNode.textContent = text
  textNode.style.fontSize = '12px'
  textNode.style.color = '#42b883'
  textNode.style.marginTop = '8px'

  mask.appendChild(spinner)
  mask.appendChild(textNode)
  return mask
}

export const loadingDirective: Directive = {
  mounted(el: HTMLElement, binding: DirectiveBinding) {
    // 确保目标元素有定位属性
    const position = window.getComputedStyle(el).position
    if (position === 'static' || !position) {
      el.style.position = 'relative'
    }

    // 获取自定义文案，默认为空
    const text = binding.arg || ''
    const mask = createLoadingMask(text)
    el.__loadingMask = mask

    if (binding.value) {
      el.appendChild(mask)
    }
  },
  updated(el: HTMLElement, binding: DirectiveBinding) {
    if (binding.oldValue !== binding.value) {
      const mask = el.__loadingMask
      if (binding.value) {
        if (mask && !el.contains(mask)) {
          el.appendChild(mask)
        }
      } else {
        if (mask && el.contains(mask)) {
          el.removeChild(mask)
        }
      }
    }
  },
  unmounted(el: HTMLElement) {
    const mask = el.__loadingMask
    if (mask && el.contains(mask)) {
      el.removeChild(mask)
    }
    delete el.__loadingMask
  }
}

// 扩展 HTMLElement 类型 (TS 支持)
declare module 'vue' {
  interface HTMLElement {
    __loadingMask?: HTMLElement
  }
}
```

#### 2. 全局注册指令
在 `main.ts` 中注册：
```typescript
import { createApp } from 'vue'
import App from './App.vue'
import { loadingDirective } from './directives/loading'

const app = createApp(App)
app.directive('loading', loadingDirective)
app.mount('#app')
```

#### 3. 在组件中使用
```vue
<template>
  <div class="card" v-loading="isLoading" v-loading="'数据获取中'">
    <h3>用户信息</h3>
    <p>姓名：{{ userInfo.name }}</p>
    <p>年龄：{{ userInfo.age }}</p>
    <button @click="fetchData">刷新数据</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const isLoading = ref(false)
const userInfo = ref({ name: '张三', age: 25 })

const fetchData = async () => {
  isLoading.value = true
  // 模拟 API 请求
  await new Promise(resolve => setTimeout(resolve, 1500))
  userInfo.value = { name: '李四', age: 30 }
  isLoading.value = false
}
</script>

<style scoped>
.card {
  width: 300px;
  height: 200px;
  border: 1px solid #ccc;
  padding: 20px;
  /* 必须有非 static 定位，指令内部已经做了兼容，但最好手动写上 */
  position: relative; 
}
</style>
```

---

### 总结与对比

| 特性         | 全局 Loading                                    | 局部 Loading                                        |
| :----------- | :---------------------------------------------- | :-------------------------------------------------- |
| **作用范围** | 整个浏览器视口                                  | 当前指令绑定的 DOM 元素内部                         |
| **适用场景** | 路由跳转、首屏加载、全局通用的核心 API 并发请求 | 表格数据刷新、抽屉/弹窗内的数据加载、按钮防重复提交 |
| **实现推荐** | Teleport + Pinia/响应式单例 + Axios 拦截器      | Vue 自定义指令 `v-loading`                          |
| **用户交互** | 通常阻止整个页面的鼠标和键盘交互                | 只阻止局部区域的交互，页面其他部分可正常操作        |

**实战建议**：
现代后台系统中，并不是所有的请求都需要全屏遮罩。在 Axios 拦截器中，可以增加一个判断逻辑，例如：对于 `GET` 请求且带有特定 header（如 `X-Loading: false`）的接口，不触发全局 Loading，而是交给组件内部自己去管理局部 Loading，这样用户体验会更好。

# VUE3文件结构



### Vue3 + Vite 项目目录

Vite 创建的 Vue3 项目

这是目前 **Vue3 官方首选** 的项目结构，最常用。

```
your-vue3-project/
├── .vscode/            # VSCode 编辑器配置（自动生成）
├── node_modules/       # 项目依赖包（npm install 后生成）
├── public/             # 静态资源（不会被打包，直接复制到 dist）
│   └── favicon.ico     # 网站图标
├── src/                # 【核心源码目录】99% 工作都在这里
│   ├── assets/         # 静态资源：图片、样式、字体（会被打包）
│   ├── components/     # 公共组件（全局复用的小组件）
│   ├── views/          # 页面组件（路由对应的大页面）
│   ├── router/         # 路由配置（vue‑router）
│   │   └── index.js
│   ├── store/          # 状态管理（pinia/vuex）
│   │   └── index.js
│   ├── api/            # 接口请求（axios 封装 + 接口地址）
│   ├── utils/          # 工具函数（时间格式化、防抖等）
│   ├── hooks/          # Vue3 组合式函数（自定义 hooks）
│   ├── layouts/        # 布局组件（可选：头部、侧边栏、底部）
│   ├── directives/     # 自定义指令（可选）
│   ├── App.vue         # 根组件（所有页面的入口）
│   └── main.js         # 项目入口文件（创建 Vue 实例）
├── .env.development    # 开发环境变量
├── .env.production     # 生产环境变量
├── .gitignore          # Git 忽略文件
├── index.html          # 页面入口 HTML（Vite 独有）
├── package.json        # 项目配置 + 依赖 + 脚本命令
├── vite.config.js      # Vite 配置文件（代理、打包、路径）
└── README.md           # 项目说明文档
```

#### 根目录

##### 1. `.vscode/`

VSCode 的工作区配置文件夹，**不提交git**。
里面可以放：

- 代码格式化配置 `.prettierrc`、ESLint配置
- settings.json：项目专属编辑器设置，比如自动格式化、文件排除> 
> 作用：团队所有人打开项目，编辑器行为保持一致。

##### 2. `node_modules/`

执行 `npm install` 自动生成。
存放项目所有第三方依赖：vue、vue‑router、pinia、axios 等。

- 体积巨大，**必须写进.gitignore，不要提交到仓库**
- 删除后执行 `npm install` 可完整恢复。

##### 3. `public/` 公共静态资源

> 
> ✅**不会经过Vite打包编译**，构建时原封不动复制到dist输出目录。
> 直接通过根路径访问 `/favicon.ico`。
> 适合放：网站图标、大文件、第三方CDN拷贝的js文件。
> 区分 `src/assets`：public资源不做压缩、不做hash；assets会被vite处理、压缩、重命名hash。

- `favicon.ico`：浏览器标签页小图标。

##### 4. `.env.development` / `.env.production` 环境变量文件

- `.env.development`：`npm run dev` 开发环境读取
- `.env.production`：`npm run build` 打包生产环境读取
定义环境变量，例如接口基础地址：

```
VITE_API_BASE_URL = '/api'
```

代码中通过 `import.meta.env.VITE_API_BASE_URL` 获取。

> 
> Vite规定自定义变量必须前缀 `VITE_`。

##### 5. `.gitignore`

Git忽略配置文件。写在这里的文件/文件夹，git不会追踪，不会提交到远程仓库。
一般忽略：`node_modules`、dist、.vscode、本地环境文件。

##### 6. `index.html`

👉 **Vite项目真正的入口HTML文件（非常重要）**
Vite不像webpack把html生成在内部，而是把index.html放在项目根。
里面包含挂载节点 `<div id="app"></div>`，引入 `src/main.js`。
可以修改title、引入外部js、cdn。

##### 7. `package.json`

项目的“说明书”。

1. `scripts`：脚本命令，`npm run dev` / `npm run build` 在这里配置
2. `dependencies`：业务运行依赖（vue、router、pinia，打包会打进产物）
3. `devDependencies`：开发依赖（vite、eslint、prettier，仅开发阶段用）
4. name、version：项目名称版本。

##### 8. `vite.config.js`

Vite的配置文件，项目构建、开发服务器核心配置。
常用配置：

- 开发服务器 proxy 代理（解决跨域）
- 路径别名 `@` 映射 src
- 打包配置、输出目录、插件引入。

##### 9. `README.md`

项目文档，markdown格式。写项目介绍、启动步骤、依赖安装、打包命令。团队协作看这个文档。

---

#### src 目录：业务代码全部在这里

##### src/App.vue

**根组件，整个Vue应用的顶层组件。**
所有路由页面、全局组件都挂载在它下面。一般放 `<router-view />` + 全局公共布局。

##### src/main.js

👉 **Vue应用JS入口文件**
创建vue应用实例，挂载到index.html的`#app`节点。
在这里注册：路由、pinia、全局组件、全局指令。

```
import { createApp } from 'vue'
import App from './App.vue'
const app = createApp(App)
app.use(router)
app.use(store)
app.mount('#app')
```

##### src/assets

**会被Vite编译打包的静态资源**
存放：图片、css、scss、字体文件。
import导入使用，打包后会压缩、加上哈希文件名。

```
import logo from '@/assets/logo.png'
```

> 
> 和public区别：
> assets：import引入，参与打包，适合业务中引用的小图片样式；
> public：直接url访问，不编译，适合静态大资源。

##### src/components

**公共可复用小组件**，非路由页面。
例如：按钮、弹窗、分页、搜索框、表单组件。

> 
> 规范：组件大写命名，多个页面重复使用就抽到此目录。

##### src/views

**路由页面组件！**
每一个文件对应一个路由页面，访问路由渲染views下面的组件。
例：`Home.vue`、`Login.vue`、`User/List.vue`。

> 
> 区分 components vs views：
> views：完整页面，被router引用；
> components：页面里面拆分出来的小组件。

##### src/router/index.js

vue‑router路由配置。
定义路由规则：path、component、name、meta、嵌套路由、路由守卫。
main.js中 `app.use(router)` 注册。

##### src/store

Pinia状态管理仓库，全局共享状态。
按模块拆分store，例如 user.js、cart.js。
存放全局数据：登录用户信息、全局配置。

##### src/api

接口层，axios统一管理。

1. 封装axios实例（baseURL、拦截器请求/响应）
2. 统一管理所有后端接口请求函数
页面只需要 import 接口函数调用，不直接写axios。
解耦，后端接口改地址，只改api层，不用改每一个页面。

##### src/utils

工具函数库，纯JS，不依赖Vue。
时间格式化、防抖节流、密码校验、金额处理、本地存储封装。
多个地方复用的通用方法抽到此。

##### src/hooks

⭐Vue3自定义组合式hooks，`script setup`专用。
把组件中重复逻辑抽离出来，**复用响应式逻辑**。
例如：`useUserInfo`、`useTable`、`usePagination`。

> 
> 和utils区分：
> utils：普通工具函数，无响应式；
> hooks：内部使用ref/reactive/computed，带Vue响应式。

##### src/layouts 【可选】

布局组件目录。
后台管理系统常用：整体布局，包含侧边栏、头部导航、主内容区`<router-view>`。
路由页面嵌套在layout里面。

##### src/directives 【可选】

存放自定义指令，统一管理全部自定义指令，在main.js全局注册。
v‑permission权限、v‑copy复制、v‑drag拖拽等。

#### 面试简短背诵版

1. `public`静态资源不打包；`assets`资源参与vite编译打包。
2. `components`是公共小组件；`views`是路由页面。
3. `utils`普通工具函数；`hooks`是带响应式的组合式复用逻辑。
4. `api`统一封装axios接口；`store`存放pinia全局状态。
5. `index.html`是Vite真正入口，main.js创建Vue实例挂载。
6. `vite.config.js`做代理、别名、打包配置。



### package.json 

#### 一、是什么

`package.json` 是 Node/npm 项目**根目录的项目描述文件**，JSON格式，用来描述项目元信息、声明依赖、自定义脚本、配置项目行为。
> 生成命令：
> ```bash
> npm init       # 交互式填写
> npm init -y    # 一键生成默认package.json
> ```
> ⚠️ JSON标准**不支持注释**，写注释会报错。

#### 二、核心字段（按面试优先级排序）

```json
{
  "name": "vue-personal-web",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite --mode development",
    "build": "vite build",
    "test": "vite build --mode test",
    "prod": "vite build --mode production",
    "preview": "vite preview"
  },
  "dependencies": {
    "vue": "^3.4.21",
    "axios": "^1.6.0"
  },
  "devDependencies": {
    "vite": "^5.2.0",
    "@vitejs/plugin-vue": "^5.0.0"
  },
  "engines": {
    "node": ">=18.0.0"
  }
}
```

##### 1. name：项目名称

- 小写，可用短横线 `-`，不能有空格；
- **如果不发布到npm公共仓库，名称随便写**；发布npm包则不能重名。

##### 2. version：版本号（语义化版本 SemVer）

格式：`主版本.次版本.补丁版本` → `X.Y.Z`
- X 主版本：破坏性不兼容改动
- Y 次版本：新增功能，向下兼容
- Z 补丁：bug修复，无新功能

版本符号：
- `^1.2.3`：主版本固定，可升级次版本、补丁 `>=1.2.3 <2.0.0`（npm默认）
- `~1.2.3`：次版本固定，仅升级补丁 `>=1.2.3 <1.3.0`
- `1.2.3`：锁定精确版本，不会自动升级

##### 3. private

```json
"private": true
```
业务前端项目**建议开启**，禁止`npm publish`误上传到npm公共仓库。Vite脚手架默认自带。

##### 4. type 模块类型

- `"type":"module"`：ES Module，`import / export`（Vite项目默认）
- 默认不写= `commonjs`，用`require / module.exports`

##### 5. scripts（重中之重，你前面的dev/build/prod都在这里）

脚本名称**完全自定义，不是npm内置命令**。
`npm run xxx` 会执行后面的shell命令，优先读取项目内`node_modules/.bin`下的工具。
```json
"scripts": {
  "dev": "vite --mode development",
  "build": "vite build",
  "prod": "vite build --mode production"
}
```
执行：`npm run dev`、`npm run build`

##### 6. dependencies 生产依赖

> 项目**线上运行时必须**的包，打包后代码会引用。
> 安装命令：`npm install xxx`
> 示例：vue、vue-router、axios

##### 7. devDependencies 开发依赖

> **仅本地开发、打包构建阶段使用，线上运行不需要**。打包后不会打进dist。
> 安装命令：`npm install xxx -D`
> 示例：vite、eslint、typescript

> ✅面试核心考点：
> dependencies：运行依赖，线上运行需要；
> devDependencies：开发构建依赖，打包产物不需要。

##### 8. engines

限制node/npm版本，防止环境不一致报错
```json
"engines": {
  "node": ">=18.0.0"
}
```

##### 其他常用字段（了解即可）

- `description`：项目描述
- `author`：作者
- `license`：开源协议（MIT最常用）
- `keywords`：数组，关键词，发布npm包搜索用
- `repository`：git仓库地址
- `browserslist`：控制浏览器兼容，postcss/autoprefixer使用

#### 三、配套文件

1. `package-lock.json`：npm自动生成锁文件，锁定所有依赖（含间接依赖）精确版本，**必须提交git**。
2. `node_modules`：存放下载的依赖包，**加入.gitignore，不要提交**。

#### 四、常用命令

```bash
# 读取package.json安装所有依赖
npm install
# 安装生产依赖，写入dependencies
npm install axios
# 安装开发依赖，写入devDependencies
npm install vite -D
# 执行脚本
npm run dev
npm run build
# 卸载包
npm uninstall axios
```

#### 五、面试简答题（直接背）

##### Q1：dependencies 和 devDependencies 的区别？

> dependencies 是项目运行时依赖，上线后代码执行需要；
> devDependencies 只在本地开发、构建打包时使用，打包后的dist不会包含这些包，线上不需要。

##### Q2：scripts原理是什么？

> scripts是自定义脚本，npm run xxx会在项目node_modules/.bin目录查找命令并执行，脚本名没有内置含义，可以自定义。

##### Q3：package.json和package-lock.json区别

> package.json声明依赖**版本范围**，存在版本浮动；
> package-lock.json锁定所有直接、间接依赖**精确版本**，保证不同环境安装依赖完全一致。

#### 六、踩坑总结

1. package.json不能写注释；
2. 不要手动修改package-lock.json；
3. 业务前端项目务必设置`private:true`；
4. 不要混用npm/yarn/pnpm，锁文件会冲突。

如果你想，我可以把：
`package.json` + `package-lock.json` + npm install / dev / build 合并成一页面试速记文档。

### vite.config.js 

（Vite前端工程，面试高频）

> 核心：`vite.config.js` 是 Vite 的**配置文件**，项目根目录。Vite 执行 `vite`（dev）、`vite build`、`vite preview` 时自动读取。
> 后缀可选：`.js` / `.ts`；Vite 原生支持 ES Module。
> 生成方式：Vite脚手架创建项目自动生成。

> 导入：`defineConfig` 是 Vite 提供的工具函数，**提供TS类型提示，只是语法糖，不改变配置本身**。

#### 完整模板（Vue3 + Vite，可直接复制）

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path' // node内置path模块

// [https://vitejs.dev/config/](https://vitejs.dev/config/)
export default defineConfig({
  // 1. 项目根目录
  root: '.',
  // 2. 打包输出目录
  build: {
    outDir: 'dist',
    assetsDir: 'assets',
    sourcemap: false,
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true, // 打包删除console
      }
    }
  },
  // 3. 开发服务器配置（npm run dev）
  server: {
    host: '0.0.0.0', // 允许局域网访问
    port: 5173,
    open: true, // 启动自动打开浏览器
    cors: true,
    proxy: {
      // 后端接口代理，解决跨域
      '/api': {
        target: '[http://127.0.0.1:8080](http://127.0.0.1:8080)',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  },
  // 4. 预览打包产物 npm run preview
  preview: {
    host: '0.0.0.0',
    port: 4173
  },
  // 5. 插件（核心！Vite基于插件体系）
  plugins: [vue()],
  // 6. 路径别名 import导入简写
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src')
    }
  },
  // 7. 环境变量相关
  envDir: '.',
  envPrefix: 'VITE_',
  // 8. 全局css预处理器（scss less）
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/assets/style/vars.scss";`
      }
    }
  },
  // 9. 静态资源处理
  assetsInclude: [/\.md$/],
  // 10. 依赖预构建（Vite核心特性）
  optimizeDeps: {
    include: ['axios']
  }
})
```

---

#### 逐个字段详细拆解（按面试优先级排序）

##### 1. defineConfig

```js
import { defineConfig } from 'vite'
export default defineConfig({ ... })
```
- 作用：**类型提示**，TS/编辑器自动补全配置项；
- 本质：只是一个返回配置对象的函数，**完全可以直接导出对象**：`export default { ... }`，效果一样。

##### 2. plugins 插件数组【最高频】

Vite 是插件驱动框架，插件用来扩展能力
- `@vitejs/plugin-vue`：解析`.vue`单文件组件（Vue项目必备）
- `@vitejs/plugin-react`：react项目用
- 常用第三方插件：
  - `vite-plugin-vue-setup-extend`：setup语法糖写name
  - `vite-plugin-pages`：自动路由
  - `vite-plugin-compression`：gzip压缩打包产物

> 面试一句话：Vite插件可以在构建、dev服务、模块转换各个阶段介入代码处理。

##### 3. server 开发服务器配置（`npm run dev`生效）

```js
server:{
  host: '0.0.0.0', // 监听所有网卡，局域网其他设备访问
  port: 5173,      // 默认端口
  open:true,       // 启动自动打开浏览器
  cors:true,       // 开启跨域
  proxy:{}         // 接口代理（前端解决开发环境跨域）
}
```
###### proxy 代理原理（面试重点）

前端浏览器请求 `/api/xxx` → Vite开发服务器转发到 `target` 后端地址。
> ⚠️ **仅开发环境生效！打包build之后proxy失效！**
> 打包后上线，跨域要在Nginx配置，vite代理只在dev阶段有用。

##### 4. build 打包配置（`npm run build` 执行打包）

```js
build:{
  outDir: 'dist',       // 打包输出文件夹，默认dist
  assetsDir: 'assets',  // js/css/img静态资源存放子目录
  sourcemap: false,     // 是否生成源码映射，生产环境关闭，防止源码泄露
  minify:'terser',      // 压缩工具，esbuild更快；terser支持删除console
  rollupOptions:{       // vite底层打包基于rollup！rollup的配置在这里
    // 多入口、分包、自定义chunk名称
    output:{
      chunkFileNames: 'assets/js/[name]-[hash].js',
      assetFileNames: 'assets/[ext]/[name]-[hash].[ext]'
    }
  }
}
```
- `drop_console:true`：打包自动删除所有console.log
- 底层：Vite开发阶段用esbuild做预构建；**生产打包底层是Rollup**（面试高频考点）

##### 5. resolve 路径别名

```js
resolve:{
  alias:{
    '@': resolve(__dirname, 'src')
  }
}
```
作用：`import Hello from '@/components/Hello.vue'`
等价于 `import Hello from 'src/components/Hello.vue'`
> 注意：TS项目需要在tsconfig.json同步配置paths，否则ts报错。

##### 6. css 全局样式、预处理器配置

```js
css:{
  preprocessorOptions:{
    scss:{
      additionalData: `@import "@/assets/vars.scss";`
    }
  }
}
```
`additionalData`：**全局注入scss变量、mixin**，不用每个vue文件手动import。
支持 scss / less / stylus。

##### 7. optimizeDeps【Vite核心原理，面试重点】依赖预构建

```js
optimizeDeps:{
  include:['axios','element-plus']
}
```
> Vite开发阶段：原生ES Module，浏览器按需请求文件。
> 但是很多第三方包是CommonJS，并且大量小文件。
> **依赖预构建：Vite启动dev时，用esbuild把第三方依赖提前打包成单个模块，转换成ESM，减少浏览器请求数量。**
- include：强制预构建某些包
- exclude：跳过预构建

> 面试一句话：
> Vite开发环境，源码不打包，直接交给浏览器ESM加载；第三方依赖通过esbuild预构建。生产打包用Rollup。

##### 8. preview 预览服务（npm run preview）

打包完成dist后，启动静态服务器预览dist，**仅本地测试，不能公网生产使用**。

##### 9. envPrefix 环境变量前缀

```js
envPrefix:"VITE_"
```
Vite环境变量默认必须以`VITE_`开头，代码中用`import.meta.env.VITE_XXX`读取。
可以修改前缀，但不推荐。

---

#### Vite 两个阶段核心原理（面试必背，和vite.config强相关）

1. **开发环境（npm run dev）**
- 不打包源码；利用浏览器原生ESM；
- 使用esbuild做**依赖预构建**，处理第三方包；
- vite.config → server、plugins、optimizeDeps生效；build配置**不生效**。

2. **生产打包（npm run build）**
- 底层调用Rollup，打包全部代码，输出dist；
- build、plugins、rollupOptions生效；server代理完全失效。

---

#### 高频面试题

##### Q1：Vite开发环境和生产打包底层分别用什么？

> 开发环境：esbuild做依赖预构建，源码直接浏览器ESM加载，不打包源码；
> 生产打包：底层 Rollup。

##### Q2：vite的proxy代理什么时候生效？打包后还生效吗？

> 只在`npm run dev`开发服务器生效；执行build打包后，proxy配置直接失效。上线跨域需要Nginx反向代理。

##### Q3：optimizeDeps是干什么的？

> 依赖预构建。Vite启动dev时，使用esbuild将第三方依赖提前打包、转换为ESM，解决CommonJS包兼容问题，减少浏览器大量文件请求，提升开发速度。

##### Q4：defineConfig作用？

> 提供配置的类型提示，方便编辑器自动补全，不是必须的语法糖。

##### Q5：alias路径别名原理？

> resolve.alias 在导入模块时做路径替换，简化import写法。TS项目需要同步在tsconfig.json配置paths。

#### 常见踩坑

1. proxy 打包后失效，很多新手踩坑；
2. 环境变量必须VITE_前缀，否则前端读取不到；
3. `__dirname` 在vite.config.ts中使用需要额外配置；
4. 打包后静态资源404：修改`base`配置，`base:'./'`相对路径；
```js
// 部署在子目录时使用
base: './'
```

---

#### 精简背诵版（面试口述）

> vite.config.js是Vite的配置文件，执行dev/build/preview自动加载。
> plugins是插件系统扩展能力；server配置开发服务器、接口代理（仅dev有效）；build配置生产打包，底层Rollup；resolve配置路径别名；optimizeDeps是依赖预构建，esbuild提前处理第三方依赖。
> 开发环境使用浏览器原生ESM，esbuild预构建依赖；生产打包使用Rollup。

如果你需要，我可以把：
package.json + package-lock.json + npm命令 + vite.config.js 整套前端工程面试汇总整理成一页。





### `vite.config.js`

`vite.config.js` 是 **Vite 项目的核心配置文件**，相当于 Vue CLI 的 `vue.config.js`。

它的作用：**控制项目怎么启动、怎么打包、怎么代理接口、路径别名、插件**等。

---

#### 一、最简基础版（刚创建项目时）

```js
// 导入配置方法
import { defineConfig } from 'vite'
// 导入 Vue 插件
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  // 插件
  plugins: [vue()],
})
```

---

#### 二、企业级完整版（最常用）

这是你**实际开发一定会用到**的配置，直接复制就能用：

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path' // 路径模块

export default defineConfig({
  // 1. 插件
  plugins: [vue()],

  // 2. 路径别名（让 @ 直接指向 src 文件夹）
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  },

  // 3. 开发服务器配置
  server: {
    port: 3000, // 启动端口号
    open: true, // 自动打开浏览器
    cors: true, // 允许跨域
    
    // 接口代理（解决开发环境跨域！）
    proxy: {
      '/api': {
        target: 'http://localhost:8080', // 后端接口地址
        changeOrigin: true, // 开启跨域
        rewrite: (path) => path.replace(/^\/api/, '') // 重写路径
      }
    }
  },

  // 4. 打包配置
  build: {
    outDir: 'dist', // 打包输出文件夹
    assetsDir: 'assets', // 静态资源目录
    minify: 'esbuild' // 压缩代码
  }
})
```

---

#### 三、4 大核心配置（必须懂）

##### 1. `plugins` 插件

用来扩展 Vite 功能，比如支持 Vue、JSX、自动导入等。

```js
plugins: [vue()]
```

---

##### 2. `resolve.alias` 路径别名

**让 `@` 代表 `src` 目录**，告别繁琐的 `../../`。

配置后可以这样写：

```js
// 原来
import Hello from '../../components/Hello.vue'

// 现在（更简洁）
import Hello from '@/components/Hello.vue'
```

---

##### 3. `server` 开发服务器（最重要）

- `port: 3000`：设置启动端口
- `open: true`：自动打开浏览器
- **`proxy`：接口代理（解决跨域）**

前端请求 `/api/user` → 自动转发到后端 `http://localhost:8080/user`

---

##### 4. `build` 打包配置

控制 `npm run build` 怎么打包：

- `outDir: 'dist'`：打包后生成的文件夹名
- `assetsDir`：图片、样式等资源目录

---

#### 四、和 .env / package.json 的关系

1. **读取 .env 环境变量**
   Vite 会自动根据 `npm run dev`/`build` 加载 `.env.development` 或 `.env.production`。

2. **运行命令**
   `package.json` 的 `scripts` 会调用这个配置文件：

   ```json
   "dev": "vite"
   ```

---

#### 五、常见使用场景

##### 场景1：修改启动端口

```js
server: {
  port: 8080
}
```

##### 场景2：配置接口代理（解决跨域）

```js
server: {
  proxy: {
    '/api': {
      target: 'https://api.xxx.com',
      changeOrigin: true
    }
  }
}
server: {
    host: '0.0.0.0',
        port: 5173,
            // 配置代理，解决开发环境跨域问题
            proxy: {
                '/api': {
                    target: 'http://localhost:8080', // Spring Boot 后端服务地址
                        changeOrigin: true,
                            rewrite: (path) => path.replace(/^\/api/, ''),
                },
                },
        },
```

###### src/utils/env.ts

```typescript
export interface AppEnv {
  title: string
  apiBaseUrl: string
  apiTimeout: number
}

export const appEnv: AppEnv = {
  title: import.meta.env.VITE_APP_TITLE || 'Vue3 Demo',
  apiBaseUrl:
    import.meta.env.VITE_API_BASE_URL || '/api',
  apiTimeout: Number(import.meta.env.VITE_API_TIMEOUT || 10000),
}

```



###### **现在的工作流程**：

1. 前端发起 /api/auth/login 请求。
2. 由于 baseURL 是相对路径 /api，Vite 的代理会成功拦截此请求。
3. Vite 将请求转发到 http://localhost:8080/auth/login。
4. 后端 Spring Boot 应用正确接收并处理请求。

请您**重启 Vite 开发服务器**以使代理配置生效，然后重新尝试登录。

###### 为什么 Vite 代理会失效？

Vite 的 server.proxy 只对 相对路径 的请求生效。当您在 axios.create 中提供了一个完整的 baseURL (如 https://... ) 时，Axios 会认为这是一个绝对路径请求，从而 完全绕过 Vite 的代理机制，直接访问该 baseURL 。

###### Vite 代理已处理跨域 :

- 在 vite.config.ts 中，我们配置的 proxy 已经完美地解决了 开发环境 下的跨域问题。
- 它的工作原理是：浏览器发出的请求实际上是给了 Vite 开发服务器（例如 http://localhost:5173 ），这是一个同源请求。然后 Vite 服务器再将这个请求转发给后端（ http://localhost:8080 ），服务器之间的通信不受浏览器同源策略的限制。

##### 场景3：让 @ 指向 src

```js
resolve: {
  alias: {
    '@': path.resolve(__dirname, 'src')
  }
}
```

---

#### 总结

1. **`vite.config.js` = Vite 项目总配置**；
2. 4 大核心：**插件、路径别名、开发服务器、打包**；
3. **路径别名 `@`** 和 **接口代理 `proxy`** 是日常最常用；
4. 修改配置后 **必须重启服务** 才生效。

需要我给你一份**带自动导入、ESLint、压缩优化**的超级完整版配置吗？

### vite.config.ts 

（TS版本Vite配置，面试重点）

> `vite.config.ts` = **TypeScript版本的Vite配置文件**，放在项目根目录；
> 执行 `vite / npm run dev`、`vite build / npm run build`、`vite preview` 时，Vite自动读取。
> ✅ 配置**所有字段和 vite.config.js 完全一样，只是文件是TS，自带强类型校验**，IDE自动提示、类型报错。
> Vite内部会自动用esbuild编译这个ts配置文件，不需要你手动tsc编译。

> 区别一句话：
> - `vite.config.js`：JS版，弱类型，适合JS项目
> - `vite.config.ts`：TS版，**强类型、自动类型校验**，Vue3+TS项目脚手架默认生成

#### 优先级

**Vite 项目里，.ts 和 .js 配置文件只能用一个，不能同时生效！**

1. **`vite.config.ts` = TypeScript 版本配置**
2. **`vite.config.js` = JavaScript 版本配置**

##### **规则：Vite 只会自动读取其中一个！**

优先级：
**`.ts` > `.js`**
只要项目里同时存在 **`vite.config.ts`**，Vite **就会忽略 `.js` 文件**。

#### 完整可直接复制模板（Vue3 + TS + Vite）

```ts
import { defineConfig, loadEnv } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'

// defineConfig支持函数写法，可以拿到 command、mode，读取环境变量做动态配置
export default defineConfig(({ command, mode }) => {
  // loadEnv：读取.env环境变量文件，mode就是development/test/production
  const env = loadEnv(mode, process.cwd(), 'VITE_')
  // command: 'serve'开发环境(dev) | 'build'打包环境
  const isBuild = command === 'build'

  return {
    // 基础公共路径，部署到子目录用 base: './'
    base: '/',

    // 插件系统
    plugins: [vue()],

    // 模块解析，路径别名
    resolve: {
      alias: {
        // @ 代表 src目录
        '@': path.resolve(__dirname, 'src')
      },
      // 导入时可省略的后缀
      extensions: ['.mjs', '.js', '.ts', '.jsx', '.tsx', '.json', '.vue']
    },

    // 开发服务器：仅npm run dev生效，打包build失效
    server: {
      host: '0.0.0.0', // 允许局域网访问
      port: 5173,
      open: true,
      cors: true,
      // 接口代理，解决开发环境跨域
      proxy: {
        '/api': {
          target: '[http://127.0.0.1:8080](http://127.0.0.1:8080)',
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/api/, '')
        }
      }
    },

    // 打包配置 npm run build
    build: {
      outDir: 'dist',
      assetsDir: 'assets',
      sourcemap: false, // 生产关闭，防止源码泄露
      minify: 'terser',
      terserOptions: {
        compress: {
          drop_console: true, // 删除console.log
          drop_debugger: true
        }
      },
      // 底层rollup打包配置
      rollupOptions: {
        output: {
          // 分包，静态资源哈希命名，缓存优化
          chunkFileNames: 'assets/js/[name]-[hash].js',
          assetFileNames: 'assets/[ext]/[name]-[hash].[ext]'
        }
      }
    },

    // 打包产物预览：npm run preview
    preview: {
      host: '0.0.0.0',
      port: 4173
    },

    // css预处理器 scss/less全局注入变量
    css: {
      preprocessorOptions: {
        scss: {
          additionalData: `@import "@/assets/style/vars.scss";`
        }
      }
    },

    // 依赖预构建（Vite核心）
    optimizeDeps: {
      include: ['axios', 'element-plus']
    },

    // 环境变量前缀
    envPrefix: 'VITE_'
  }
})
```

#### 核心关键字段讲解（TS特有点标记）

##### 1. defineConfig

```ts
import { defineConfig } from 'vite'
```
- TS场景下，**自动推导UserConfig类型**，写配置时IDE实时提示、类型校验，写错配置直接标红。
- 也可以手动导入类型：
```ts
import type { UserConfig } from 'vite'
export default {
  //配置
} satisfies UserConfig
```
> `satisfies` TS4.9+语法，同样做类型约束。

##### 2. defineConfig 函数形式（TS项目高频）

```ts
export default defineConfig(({ command, mode }) => {
  // 可以根据mode开发/测试/生产，动态返回不同配置
  return {}
})
```
- `mode`：`development` / `test` / `production`，对应 `--mode`
- `command`：`serve`（dev开发） / `build`（打包）
- 配合`loadEnv`读取.env文件，**动态切换配置**，非常适合多环境项目(dev/test/prod)

##### 3. loadEnv

```ts
const env = loadEnv(mode, process.cwd(), 'VITE_')
```
> Vite内置API，**在vite.config.ts内部读取环境变量**
> 三个参数：mode、项目根目录、变量前缀。

##### 4. resolve.alias 路径别名

> ⚠️ TS项目踩坑重点：
> 在`vite.config.ts`配置`@ -> src`之后，**必须同步在tsconfig.json中配置paths**，否则TS编译报错找不到模块：
```json
// tsconfig.json
"compilerOptions": {
  "paths": {
    "@/*": ["src/*"]
  }
}
```

##### 5. plugins、server、build、css、optimizeDeps

**配置含义和vite.config.js完全一模一样！**
> 记住：**配置规则没有任何区别，只是文件后缀ts，带类型**

###### vite.config.ts 与 vite.config.js对比

| 项目 | vite.config.js | vite.config.ts |
|---|---|---|
| 文件类型 | JavaScript | TypeScript |
| 类型校验 | 弱类型，无校验 | ✅强类型，IDE提示、报错 |
| 适用项目 | JS项目 | Vue3+TS / React+TS |
| 配置字段 | 完全相同 | 完全相同 |
| 加载方式 | Vite直接读取 | Vite自动用esbuild转译ts再加载 |

#### 高频面试题

##### Q1：vite.config.ts 和 vite.config.js 的区别？

> 配置选项完全一致。`.ts`是TypeScript版本，自带Vite类型定义，IDE有智能提示、类型检查；Vite内部会自动编译ts配置文件，不需要手动tsc。一般TS项目使用vite.config.ts，JS项目用js版本。

##### Q2：defineConfig 的作用？

> 提供配置的类型推导，TS项目下自动识别Vite的UserConfig类型，编辑器自动补全，配置写错会有类型提示，是语法糖，不改变运行逻辑。

##### Q3：defineConfig写成函数形式有什么好处？

> 可以拿到`mode`和`command`，配合`loadEnv`读取环境变量，**动态返回不同环境的Vite配置**，开发、测试、生产打包用不同配置。

##### Q4：vite.config.ts里面配置alias，TS报错找不到@/xxx？

> vite.config.ts只是Vite构建层面的别名；TS的类型检查需要在tsconfig.json的compilerOptions.paths同步配置别名映射。

#### 常见坑

1. `__dirname` 在ts环境正常使用，需要安装`@types/node`，否则报类型缺失
```bash
npm install @types/node -D
```
2. 不要把业务代码写在vite.config.ts，它只是**构建阶段执行**，打包后不会进入前端产物
3. server.proxy仅dev生效，打包build之后失效，上线跨域Nginx处理
4. 环境变量必须`VITE_`前缀，否则前端代码`import.meta.env`读取不到

#### 注意

##### 什么时候需要重启 Vite 服务

✅ 需要重启 dev 服务：

1. 修改 vite.config.js/vite.config.ts
2. 修改 .env 环境变量文件
3. 修改 optimizeDeps 配置
4. 修改自定义 Vite 插件
5. 新增 / 删除 package.json 依赖，执行 npm install 之后

✅ 不需要重启，HMR 自动生效：

1. src 目录下的 vue、ts、js、css、scss 业务代码

##### 文件路径 / 文件名大小写问题

 Linux 区分大小写，Windows 不区分。本地写`Home.vue`，代码里 import `home.vue`，本地没问题；放到 Linux 服务器或者本地 dev 环境，修改文件后 HMR 失效，需要重启。

##### 环境变量

- Vite 环境变量**必须 VITE_开头**，否则 `import.meta.env` 读取不到；
- 环境变量**打包阶段静态替换**，不是运行时读取；
- 区分：`.env` 所有环境生效；`.env.development` dev；`.env.production` build。

##### 静态资源

- 放在 `public` 文件夹：**直接原样复制到 dist 根目录，不会经过 Vite 编译**，引用直接写 `/xxx.png`；
- 放在 `src/assets`：会被 Vite 处理、哈希命名、压缩，用 import 导入。

#### 面试精简背诵版

> vite.config.ts是Vite的TypeScript版本配置文件，配置项和js版本完全一致。
> 支持defineConfig做类型校验，支持函数式写法，可获取mode、command，配合loadEnv读取环境变量，实现多环境动态配置。
> TS项目配置alias后，需要同步修改tsconfig.json的paths，解决TS模块找不到的问题。
> Vite底层自动编译这个ts配置文件，不需要手动tsc。

---

如果你想，我可以把前面整套：
package.json + package-lock.json + npm命令 + vite.config.js + vite.config.ts 合并成一份**前端工程化面试一页纸**。

#### .env 文件

完整详解（Vite 环境变量，面试高频）

> `.env` 是**环境变量配置文件**，放在项目根目录，用来存放不同环境的全局变量。
> Vite 会根据当前启动的 `mode`，自动加载对应的 env 文件；**只在项目构建阶段读取**。

#### 一、Vite 识别的 env 文件优先级（由低到高，后面覆盖前面）

```
.env                # 所有环境都加载（公共默认）
.env.local          # 所有环境，本地私有，git一般忽略
.env.development    # npm run dev，开发环境 mode=development
.env.development.local
.env.test           # 测试环境 mode=test
.env.production     # npm run build，生产打包 mode=production
.env.production.local
```
优先级：**`.env.[mode].local` > `.env.[mode]` > `.env`**

> `.local` 结尾的文件，**建议写入 .gitignore**，存放本地开发私有配置（比如本地数据库、密钥，不要提交到代码仓库）

#### 二、编写规则

1. 格式：`KEY=VALUE`，等号两边**不要加空格**
2. 默认只有以 `VITE_` 开头的变量，前端业务代码 `import.meta.env` 才能读取！
3. 变量值默认是字符串，不需要引号；如果值包含空格，才需要引号包裹
4. 注释用 `#` 放在行首
```env
# .env.development
# 开发环境
VITE_API_BASE_URL=[http://127.0.0.1:8080](http://127.0.0.1:8080)
VITE_APP_TITLE=个人网站开发版
# 普通变量，不带VITE_，前端代码读不到
SECRET_KEY=123456
```

#### 三、代码中读取

```ts
// src 业务代码中
console.log(import.meta.env.VITE_API_BASE_URL)
console.log(import.meta.env.MODE)        // 当前环境：development / production
console.log(import.meta.env.DEV)         // 开发环境 true
console.log(import.meta.env.PROD)        // 生产打包 true
```

> ⚠️ 重点：**环境变量是打包阶段静态替换，不是浏览器运行时读取文件！**
> 打包后，变量直接被替换成字符串硬编码在js里，**不能在页面运行时修改**。

#### 四、vite.config.ts 里面读取 env

需要用Vite内置的 `loadEnv`
```ts
import { defineConfig, loadEnv } from 'vite'
export default defineConfig(({mode})=>{
  const env = loadEnv(mode, process.cwd(), 'VITE_')
  console.log(env.VITE_API_BASE_URL)
  return {
    server:{
      proxy:{
        '/api':{
          target: env.VITE_API_BASE_URL
        }
      }
    }
  }
})
```

#### 五、启动命令，指定 mode

```bash
# 默认 mode=development
npm run dev

# 指定mode为test，加载 .env.test
npm run dev -- --mode test

# npm run build 默认 mode=production，加载 .env.production
npm run build

# 打包测试环境
npm run build -- --mode test
```

#### 六、重要特性&坑点（你前面关心的：修改env需要重启）

✅ **修改.env文件，HMR不会生效，必须重启 npm run dev**
> Vite只在启动服务的一瞬间读取一次env文件；运行过程中修改env，不会自动重新加载，需要重启dev服务。

✅ 区分：
- 前端浏览器拿到的变量：**必须VITE_前缀**
- vite.config.ts 中 loadEnv 读取，可以自定义前缀

❌ 不要在 `.env` 放**密钥、数据库密码**！
打包之后所有VITE_开头的变量都会暴露在前端js代码里，任何人打开浏览器F12就能看到。
> 后端密钥、数据库账号密码，**放在服务器环境变量或者后端配置，绝对不能写在前端.env**

#### 七、.gitignore 推荐配置

```gitignore
# 本地私有环境变量，不上传
.env.local
.env.*.local
```
公共的 `.env` `.env.development` `.env.production` 提交git，团队共用。

#### 八、面试题

##### Q1：Vite中.env文件什么时候加载？修改env文件需要重启吗？

> Vite在启动dev或者打包build的时候一次性读取环境变量。修改.env文件，HMR不会更新，**必须重启服务**。

##### Q2：为什么变量要加 VITE_ 前缀？

> Vite默认只暴露VITE_开头的变量给前端代码，防止把系统、其他敏感环境变量泄露到前端。可以通过vite.config的`envPrefix`修改前缀，但不推荐。

##### Q3：import.meta.env 是运行时读取文件吗？

> 不是，是**打包阶段静态字符串替换**。打包完成后，变量直接写死在js中，浏览器无法动态读取.env文件。

##### Q4：.env 和 .env.development 的区别？

> .env 所有环境都会加载；.env.development只有mode=development（npm run dev）加载。同名变量，后面加载的会覆盖前面。

#### 背诵精简版

> .env是Vite的环境变量文件，区分多环境。env文件按mode自动加载，优先级local最高。
> 只有VITE_前缀的变量，前端代码可以通过`import.meta.env`读取；变量打包时静态替换，不是运行时读取。
> 修改env文件不会触发HMR，**需要重启vite服务**。敏感密钥不能放在前端env，会暴露。

要不要把`.env`、vite.config、HMR、打包部署相关的面试题合并成一页？







### Proxy

​          
在本项目中，“Proxy”的使用体现在两个截然不同的层面：**底层响应式原理**和**网络请求代理**。

#### 1. 底层：Vue 3 响应式系统 (核心)

虽然你没有在业务代码中手动写下 `new Proxy()`，但项目中的 **`ref` 和 `reactive` 底层全部是基于 JavaScript `Proxy` 实现的**。

- **工作原理**: 当你在 [project.ts](file:///C:/Users/24141/Desktop/trae_cn/vue3-demo/src/store/project.ts) 中定义 `const visitCount = ref(1)` 时，Vue 会创建一个 Proxy 对象。当你读取或修改 `visitCount.value` 时，这个 Proxy 会拦截（Intercept）你的操作，从而实现依赖收集和视图自动更新。
- **与 Vue 2 的区别**: 这是 Vue 3 相比 Vue 2 最重大的性能提升。Vue 2 使用 `Object.defineProperty`，而 Vue 3 的 `Proxy` 性能更好，且能完美监听数组下标变化和对象属性的新增/删除。

#### 2. 配置层：Vite 开发服务器代理 (目前未配置)

在前端开发中，通常会在 [vite.config.ts](file:///C:/Users/24141/Desktop/trae_cn/vue3-demo/vite.config.ts) 的 `server.proxy` 选项中配置代理，以解决开发环境下的 **跨域 (CORS)** 问题。

目前项目配置如下：

```typescript
// vite.config.ts
server: {
  host: '0.0.0.0',
  port: 5173,
  // 暂时没有配置 proxy
},
```

---

#### 💡 增加 Proxy 使用建议

如果你需要对接后端接口，建议在 `vite.config.ts` 中增加代理配置。

**修改方案**：
在 [vite.config.ts](file:///C:/Users/24141/Desktop/trae_cn/vue3-demo/vite.config.ts) 中增加如下配置，将所有 `/api` 开头的请求代理到测试服务器：

```typescript
// vite.config.ts
server: {
  host: '0.0.0.0',
  port: 5173,
  proxy: {
    '/api': {
      target: 'http://api.example.com', // 后端接口域名
      changeOrigin: true,               // 允许跨域
      rewrite: (path) => path.replace(/^\/api/, ''), // 去掉路径中的 /api 前缀
    },
  },
},
```

#### 总结

- **已经在使用**: 作为 Vue 3 项目，**响应式数据（Ref/Computed）** 每一秒都在通过底层 `Proxy` 运行。
- **推荐使用**: 在处理真实后端接口跨域时，应在 `vite.config.ts` 中手动配置 **开发服务器 Proxy**。

### Hook

---

#### 一、一句话定义（背这个）

**Vue3 Hook 就是把组件里的复用逻辑（数据 + 方法）抽出来的函数。**
本质：**逻辑复用工具**，让代码更简洁、更好复用。

---

#### 二、超级通俗比喻

- 一个组件 = 一间屋子
- 数据、方法 = 屋里的家具、电器
- **Hook = 可移动的模块化家具套装**
  抽到哪里都能用，不用每个屋子都重新写一遍！

---

#### 三、Hook 长什么样？（标准格式）

```ts
// 命名必须以 use 开头！！！（规范）
function useXxx() {
  // 数据
  // 方法
  return { 数据, 方法 };
}
```

---

#### 四、Vue3 自带的官方 Hook（最常用）

这些是 Vue 自带的，必须记住：

- `ref()` —— 定义基础类型响应式数据
- `reactive()` —— 定义对象类型响应式数据
- `computed()` —— 计算属性
- `watch()` —— 监听
- `watchEffect()` —— 自动依赖监听
- `onMounted()` —— 生命周期

---

#### 五、自定义 Hook（业务用得最多）

比如你要**复用“获取待办事项”的逻辑**：

##### 1. 建一个 Hook 文件 `useTodos.ts`

```ts
import { ref, onMounted } from 'vue';
import axios from 'axios';

// 自定义 Hook
export function useTodos() {
  const todos = ref([]);

  // 获取数据
  const fetchTodos = async () => {
    const res = await axios.get('/api/todos');
    todos.value = res.data.data;
  };

  // 生命周期自动加载
  onMounted(() => fetchTodos());

  // 返回数据和方法
  return { todos, fetchTodos };
}
```

##### 2. 在组件里直接用

```vue
<script setup>
// 直接引入使用
import { useTodos } from '@/hooks/useTodos';

const { todos, fetchTodos } = useTodos();
</script>
```

**任何组件想用这个逻辑，直接引入就行！**

---

#### 六、Hook 的好处（面试必说）

1. **逻辑复用**（不用复制粘贴）
2. **代码更清晰**（分离业务逻辑）
3. **便于维护**（改一处，全生效）
4. **符合 Vue3 组合式API 思想**

---

#### 七、面试标准答案（可直接背）

**Vue3 Hook 是基于组合式API的逻辑复用函数，以 use 开头，将组件中的响应式数据、业务方法、生命周期逻辑抽离封装，实现逻辑复用，让代码更简洁、易维护，是 Vue3 开发的核心规范。**

---

#### 八、超级记忆口诀

**Hook 就是抽逻辑，
数据方法放一起。
组件想用直接引，
Vue3 开发最给力！**

---

如果你需要，我可以给你写
**「项目中常用的 5 个业务 Hook（登录、列表、分页、点赞、提交）」**
直接放进你的 Vue3 项目！



### **index.html **

#### 一、一句话核心作用（背这个）

**index.html 是 Vite 项目的【入口页面】，也是整个 Vue 应用的“载体”。**
它相当于 **Vue 应用的外壳**，Vue 实例会挂载到这个页面里。

---

#### 二、最通俗的比喻

- **index.html = 房子的地基 + 框架**
- **Vue 应用 = 房子里的装修和家具**

没有 index.html，Vue 就没地方渲染。

---

#### 三、index.html 到底做了哪 3 件事？

##### 1. **提供挂载点**

里面一定有这行：

```html
<div id="app"></div>
```

Vue 就是把界面渲染到这里。

##### 2. **引入入口 JS**

Vite 通过这行找到 Vue：

```html
<script type="module" src="/src/main.js"></script>
```

→ 告诉 Vite：从 main.js 启动项目。

##### 3. **基础页面结构**

配置标题、图标、字符编码等。

---

#### 四、和 Vue2 的最大区别（面试加分）

- **Vue2（webpack）**：public/index.html
- **Vue3（Vite）**：**根目录 index.html**

因为 **Vite 以 HTML 作为入口**，而不是 JS。

---

#### 五、标准 index.html 长这样

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <link rel="icon" href="/favicon.ico">
  <title>我的Vue3项目</title>
</head>
<body>
  <!-- Vue 挂载点 -->
  <div id="app"></div>

  <!-- 入口JS -->
  <script type="module" src="/src/main.js"></script>
</body>
</html>
```

---

#### 六、总结

**index.html 是 Vite 项目的入口页面，
提供 Vue 挂载容器 #app，
并加载 main.js 启动整个应用。**

index.html 是 Vite 构建工具的**项目入口文件**，负责提供 Vue 应用的挂载容器、配置页面元信息，并引入项目的核心入口脚本 main.js，从而启动整个 Vue3 应用，是前端项目的基础承载页面。

### **main.js / main.ts **

---

#### 一、核心作用

**main.js / main.ts 是整个 Vue3 项目的【入口脚本】，
负责：创建 Vue 实例 → 挂载插件 → 渲染 App.vue → 启动项目。**

它是**所有代码的起点**。

---

#### 二、main.js 和 main.ts 有什么区别？

- **main.js**：JavaScript 版本入口
- **main.ts**：**TypeScript 版本入口**（你项目用这个）

**功能完全一样，只是语言不同！**
你的项目是 **Vue3 + TS** → 用 **main.ts**

---

#### 三、它到底做了哪 4 件事？（标准流程）

```ts
// 1. 引入 Vue 核心
import { createApp } from 'vue'

// 2. 引入 根组件
import App from './App.vue'

// 3. 创建 Vue 应用实例
const app = createApp(App)

// 4. 挂载到 index.html 的 #app 上
app.mount('#app')
```

---

#### 四、完整作用

1. **创建 Vue 应用实例**
2. **加载根组件 App.vue**
3. **注册全局插件**（路由、Pinia、ElementPlus 等）
4. **将应用挂载到 index.html 的 #app 节点**
5. **启动整个前端项目**

---

#### 五、最通俗比喻

- **index.html = 舞台**
- **#app = 表演区域**
- **main.ts = 导演**
- **App.vue = 节目单**

**main.ts 喊“开始”，整个项目才跑起来。**

---

#### 六、标准描述

**main.ts 是 Vue3 项目的脚本入口文件，负责创建应用实例、注册全局组件与插件、挂载根组件 App.vue，并将实例渲染到 index.html 的挂载点中，是启动整个前端应用的核心入口。**

✅ **index.html = 外壳**
✅ **main.ts = 入口启动**
✅ **App.vue = 根组件**

### `App.vue`

在 Vue 2 和 Vue 3 中，`App.vue` 的核心作用是高度一致的：**它是整个单页面应用（SPA）的根组件**。你可以把它理解为一个“外壳”或“主容器”，所有的其他页面和组件都会被嵌套在这个组件内部。

虽然在 Vue 2 和 Vue 3 中 `App.vue` 的职责基本相同，但由于底层架构和 API 的变化，它在写法和一些细节上存在差异。

以下是 `App.vue` 在 Vue 2/3 中的详细作用及对比：

---

#### 一、 `App.vue` 的共同核心作用（Vue 2 & Vue 3 通用）

1. **应用的基础布局结构**
   `App.vue` 通常包含了整个应用不变的基础结构。例如：导航栏、侧边栏、页脚等。无论页面怎么切换，这些基础结构始终存在。
2. **路由出口**
   在使用 Vue Router 时，`App.vue` 必须包含 `<router-view>` 标签。它是路由变化的占位符，当 URL 改变时，对应的新页面组件就会渲染到 `<router-view>` 的位置。
3. **全局状态与生命周期管理**
   作为最外层的组件，`App.vue` 适合做一些全局性的初始化操作。例如：在 `created` 或 `mounted` 中获取用户信息、初始化全局配置、监听窗口大小变化等。
4. **全局样式注入**
   在 `App.vue` 的 `<style>` 标签中编写的样式（特别是未加 `scoped` 的样式）会作为全局样式影响到整个应用。通常用于重置浏览器默认样式（CSS Reset）或定义全局变量。

---

#### 二、 Vue 2 中的 `App.vue` 特点

在 Vue 2 中，`App.vue` 严格遵循 Options API（选项式 API），并且**模板必须有一个唯一的根标签**。

**Vue 2 示例：**

```vue
<template>
  <!-- Vue 2 必须有且仅有一个根元素，通常用 div 包裹 -->
  <div id="app">
    <nav>全局导航栏</nav>
    <!-- 路由出口 -->
    <router-view />
    <footer>全局页脚</footer>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      globalData: ''
    }
  },
  created() {
    console.log('App 实例已创建，开始初始化全局数据...');
  },
  methods: {
    initGlobalData() { /* ... */ }
  }
}
</script>

<style>
/* 全局样式，没有 scoped */
body { margin: 0; }
#app { font-family: Avenir, Helvetica, Arial, sans-serif; }
</style>
```

---

#### 三、 Vue 3 中的 `App.vue` 特点

Vue 3 引入了 Composition API（组合式 API），并且**支持了 Fragment（片段）**，即模板不再强制要求单一的根标签。此外，Vue 3 推荐使用 `<script setup>` 语法糖，让代码更简洁。

**Vue 3 示例：**

```vue
<template>
  <!-- Vue 3 支持多根节点 -->
  <nav>全局导航栏</nav>
  <!-- 路由出口 -->
  <router-view />
  <footer>全局页脚</footer>
</template>

<script setup>
// Vue 3 使用 Composition API 和 <script setup>
import { onMounted } from 'vue';

onMounted(() => {
  console.log('App 组件已挂载，开始初始化全局数据...');
});

// 组件名通常通过单独的插件或宏定义（如 vite-plugin-vue-setup-extend）
// 或不需要显式定义 name
</script>

<style>
/* 全局样式 */
body { margin: 0; }
</style>
```

---

#### 四、 Vue 2 与 Vue 3 中 `App.vue` 的核心差异对比

| 特性                  | Vue 2                                                        | Vue 3                                                        |
| :-------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **模板根节点**        | 必须有且仅有一个根标签（如 `<div id="app">`）                | 支持多根节点，无需无意义的包裹标签                           |
| **脚本写法**          | Options API (`export default { data, methods, created }`)    | 推荐 `<script setup>` + Composition API (`onMounted`, `ref` 等) |
| **组件 `name` 属性**  | 通常需要写 `name: 'App'`                                     | 在 `<script setup>` 中通常不需要显式写 `name`（除非用于 `<keep-alive>` 缓存等特定场景需额外配置） |
| **全局 API 挂载方式** | 在 `main.js` 中通过 `new Vue({...})` 实例化，`App` 作为一个配置对象传入 | 在 `main.js` 中通过 `createApp(App)` 创建应用实例，更加模块化 |
| **全局上下文**        | 所有 Vue 实例共享相同的全局上下文（如 `Vue.prototype`）      | `App.vue` 对应独立的应用实例，全局属性通过 `app.config.globalProperties` 挂载 |

#### 总结

无论在 Vue 2 还是 Vue 3，`App.vue` 都是应用的**顶层容器**，负责统筹全局布局、路由分发和全局初始化。从 Vue 2 迁移到 Vue 3 时，`App.vue` 的主要变化在于**去除了单根节点的限制**以及**采用了更灵活的 `<script setup>` 组合式 API 写法**。

### Vue2/3 Router

在 Vue 2 和 Vue 3 中，Vue Router 的核心作用是完全一致的：**它是 Vue.js 的官方路由管理器，用于构建单页面应用（SPA）。**

它的根本作用是实现**“视图与 URL 的同步”**——当用户的 URL 改变时，页面不需要重新刷新加载，而是由 Vue Router 在前端拦截 URL 变化，并映射到对应的 Vue 组件进行局部渲染。

虽然核心作用一致，但由于 Vue 2 对应 Vue Router 3.x，Vue 3 对应 Vue Router 4.x，两者在 API 设计、初始化方式和底层实现上有显著差异。

---

#### 一、 Vue Router 的共同核心作用（Vue 2 & Vue 3 通用）

1. **动态路由匹配**
   可以将不同模式的 URL 映射到同一个组件，并通过参数传递数据。例如 `/user/:id`，无论是 `/user/1` 还是 `/user/2`，都会渲染 `User` 组件，并传入对应的 `id`。
2. **嵌套路由（子路由）**
   允许在组件内部继续放置 `<router-view>`，实现页面层级嵌套。例如一个后台管理系统，侧边栏不变，右侧内容区根据 `/settings/profile` 或 `/settings/security` 渲染不同的子组件。
3. **编程式导航**
   除了使用 `<router-link>` 声明式跳转外，还可以在 JavaScript 逻辑中通过代码控制跳转（如 `router.push()`、`router.replace()`），常用于登录跳转、表单提交后跳转等。
4. **导航守卫**
   提供全局的、路由独享的、或组件内的钩子函数（如 `beforeEach`、`beforeRouteEnter`）。主要用于**权限控制**（未登录拦截）、**数据预载**、**修改页面标题**等。
5. **路由模式切换**
   支持两种路由模式：
   * **Hash 模式**：URL 带有 `#`（如 `http://www.abc.com/#/home`），兼容性极好，不需要服务器配置。
   * **History 模式**：URL 干净无 `#`（如 `http://www.abc.com/home`），利用 HTML5 History API，但需要后端配置支持（Nginx 等）以防止刷新 404。
6. **路由懒加载**
   结合 Webpack/Vite 的动态导入语法（`() => import('./views/Home.vue')`），实现按需加载组件。这能将应用打包成多个小代码块，首次加载时只加载核心代码，极大提升首屏渲染速度。

---

#### 二、 Vue 2 中的 Vue Router (v3) 特点与用法

在 Vue 2 中，Vue Router 通过 `new VueRouter()` 实例化，并作为插件注入到 Vue 根实例中。在组件内部，通过 `this.$router` 和 `this.$route` 访问路由。

**初始化 (`router/index.js`)：**

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from '../views/Home.vue'

Vue.use(VueRouter) // 必须显式调用 Vue.use()

const routes = [
  { path: '/', component: Home }
]

// 创建实例
const router = new VueRouter({
  mode: 'history', // 默认为 hash
  routes
})

export default router
```

**挂载 (`main.js`)：**

```javascript
import Vue from 'vue'
import App from './App.vue'
import router from './router'

new Vue({
  router, // 注入路由
  render: h => h(App)
}).$mount('#app')
```

**组件内访问：**

```javascript
export default {
  created() {
    // 获取路由参数
    console.log(this.$route.params.id)
    // 编程式导航
    this.$router.push('/about')
  }
}
```

---

#### 三、 Vue 3 中的 Vue Router (v4) 特点与用法

Vue 3 引入了 `createRouter` 和 `createWebHistory` 等组合式 API，不再使用 `new` 关键字。组件内推荐使用 `useRouter` 和 `useRoute` 钩子函数获取路由对象。

**初始化 (`router/index.js`)：**

```javascript
import { createRouter, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'

const routes = [
  { path: '/', component: Home }
]

// 创建实例
const router = createRouter({
  // Vue 3 中模式配置方式变了
  history: createWebHistory(), // 对应 Vue2 的 mode: 'history'
  // history: createWebHashHistory(), // 对应 hash 模式
  routes
})

export default router
```

*注：在 Vue Router 4 中，不再需要 `Vue.use(VueRouter)`，而是直接在创建 app 实例时通过 `app.use(router)` 注入。*

**挂载 (`main.js`)：**

```javascript
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

const app = createApp(App)
app.use(router) // 使用插件
app.mount('#app')
```

**组件内访问 (Composition API)：**

```vue
<script setup>
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

// 获取路由参数
console.log(route.params.id)
// 编程式导航
router.push('/about')
</script>
```

---

#### 四、 Vue 2 (v3) 与 Vue 3 (v4) 核心差异对比

| 特性             | Vue 2 (Vue Router 3)                             | Vue 3 (Vue Router 4)                                         |
| :--------------- | :----------------------------------------------- | :----------------------------------------------------------- |
| **创建实例**     | `new VueRouter({...})`                           | `createRouter({...})`                                        |
| **路由模式配置** | `mode: 'history'` 或 `mode: 'hash'`              | `history: createWebHistory()` 或 `createWebHashHistory()`    |
| **插件安装**     | 在 router 文件中 `Vue.use(VueRouter)`            | 在 main 文件中 `app.use(router)`，更模块化                   |
| **组件内访问**   | Options API: `this.$router`, `this.$route`       | Composition API: `useRouter()`, `useRoute()` (更符合 Vue3 理念) |
| **动态路由删除** | 无法直接动态删除已添加的路由                     | 提供了 `router.removeRoute()` 方法，方便动态权限路由管理     |
| **通配符匹配**   | 支持使用 `*` 作为通配符捕获 404 (如 `path: '*'`) | **废弃了 `*`**，必须使用正则表达式定义 404 (如 `path: '/:pathMatch(.*)*'`) |

#### 总结

Vue Router 是 Vue 生态中连接“页面”与“URL”的桥梁。无论在 Vue 2 还是 Vue 3 中，它都负责管控制页面跳转逻辑、权限拦截和组件按需加载。从 Vue 2 升级到 Vue 3，Router 最大的变化是**全面拥抱了函数式编程（`createXxx`）**，废弃了 `new` 实例化和 `this` 指向的依赖，使得代码在 Tree-shaking 和类型推断（TypeScript 支持）方面表现得更优秀。

### Vue2/3 `store`

在 Vue 2 和 Vue 3 中，`store`（状态管理库，最典型的是 Vuex 或 Pinia）的核心作用是高度一致的：**它是应用的全局状态（数据）管理中心**。

在复杂的单页面应用（SPA）中，组件层级往往很深，如果仅靠组件的 `props` 和 `emit` 来传递数据，会遇到“Props 逐级透传”和“兄弟组件通信困难”的问题。`store` 的出现就是为了将这些**需要在多个组件间共享的状态**抽取出来，进行统一管理。

虽然在 Vue 2 和 Vue 3 中 `store` 解决的问题相同，但由于 Vue 框架底层 API 的变化，以及官方推荐的状态管理库发生了变迁（Vue 2 时代主要是 **Vuex 3**，Vue 3 时代官方推荐 **Pinia**），其使用方式和架构设计有了显著的差异。

---

#### 一、 `store` 的共同核心作用（Vue 2 & Vue 3 通用）

1. **集中式状态管理**
   将全局的数据（如用户登录信息、购物车数据、全局主题设置、权限数据等）集中存放在一个“仓库”中。任何组件都可以直接访问这些状态，而不需要通过组件树一层层传递。
2. **状态的可预测性（单向数据流）**
   状态管理的核心原则是：**组件不能直接修改仓库里的状态**。必须通过提交一个动作来描述“我要做什么”，然后由仓库去执行具体的修改。这保证了状态的每一次变更都是可追踪、可调试的，避免了数据被任意篡改导致的混乱。
3. **响应式更新**
   当 `store` 中的状态发生改变时，所有依赖该状态的组件都会自动且高效地重新渲染。
4. **开发者工具集成**
   官方的状态管理库都能完美集成到 Vue DevTools 中，开发者可以清晰地看到状态的快照、变更历史，甚至实现“时间旅行”（回退状态）。

---

#### 二、 Vue 2 中的 `store` (通常指 Vuex 3)

在 Vue 2 中，Vuex 是绝对的标准。Vuex 的设计相对严格，将仓库划分为 `state`、`getters`、`mutations`、`actions` 四个核心模块。

**特点：**

* 通过 `new Vuex.Store()` 创建实例。
* 通过 `Vue.use(Vuex)` 安装插件。
* 组件内通过 `this.$store` 访问。
* **Mutation 必须是同步的**，异步操作必须放在 Action 中。

**初始化 (`store/index.js`)：**

```javascript
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++ // 同步修改
    }
  },
  actions: {
    asyncIncrement({ commit }) {
      setTimeout(() => {
        commit('increment') // 异步操作后提交 mutation
      }, 1000)
    }
  },
  getters: {
    doubleCount(state) {
      return state.count * 2
    }
  }
})
```

**组件内访问：**

```javascript
export default {
  computed: {
    count() { return this.$store.state.count },
    doubleCount() { return this.$store.getters.doubleCount }
  },
  methods: {
    add() { this.$store.commit('increment') },
    asyncAdd() { this.$store.dispatch('asyncIncrement') }
  }
}
```

---

#### 三、 Vue 3 中的 `store` (官方推荐 Pinia / Vuex 4)

Vue 3 完全支持 Vuex 4（与 Vuex 3 API 基本一致，只是适配了 Vue 3），**但 Vue 官方现在强烈推荐使用 Pinia 作为新的状态管理库**。Pinia 被认为是 Vuex 5 的精神续作，专门为 Vue 3 的 Composition API 设计。

**Pinia 相比 Vuex 的颠覆性改变：**

1. **去掉了 Mutations**：可以直接在 Actions 中修改 State（无论是同步还是异步），极大简化了代码。
2. **扁平化架构**：不再有单一巨大的 store 根对象，而是拆分成多个独立的 store（模块化自带）。
3. **完美的 TypeScript 支持**：不再需要写复杂的类型推断字典。
4. **极其轻量**：体积只有约 1KB。

**初始化 (`stores/counter.js` - Pinia 写法)：**

```javascript
import { defineStore } from 'pinia'

// 定义一个独立的 store 模块
export const useCounterStore = defineStore('counter', {
  state: () => ({
    count: 0
  }),
  getters: {
    doubleCount: (state) => state.count * 2,
  },
  actions: {
    increment() {
      this.count++ // 直接修改 state，无需 mutation
    },
    asyncIncrement() {
      setTimeout(() => {
        this.increment() // 异步操作也直接在 action 里写
      }, 1000)
    }
  }
})
```

**组件内访问 (Composition API)：**

```vue
<script setup>
import { useCounterStore } from '@/stores/counter'

// 获取 store 实例
const counterStore = useCounterStore()

// 直接在模板中使用 counterStore.count, counterStore.doubleCount
// 调用方法：counterStore.increment()
</script>
```

---

#### 四、 Vue 2 (Vuex) 与 Vue 3 (Pinia) 核心差异对比

| 特性                | Vue 2 (Vuex 3)                                   | Vue 3 (Pinia 官方推荐)                                  |
| :------------------ | :----------------------------------------------- | :------------------------------------------------------ |
| **实例化方式**      | `new Vuex.Store({...})`                          | `defineStore('name', {...})`                            |
| **模块化**          | 需要在根 store 中通过 `modules` 嵌套配置，较繁琐 | 天然模块化，每个 `defineStore` 就是一个独立模块，扁平化 |
| **Mutations**       | **必须有**，且只能执行同步代码修改状态           | **废弃了**，直接在 actions 中修改 state                 |
| **组件内获取**      | `this.$store.state.xxx` / `mapState` 辅助函数    | `const store = useXxxStore()`，更符合 Composition API   |
| **TypeScript 支持** | 较弱，自动提示往往需要大量额外的类型声明代码     | 极佳，开箱即用的完整类型推导                            |
| **体积与概念**      | 概念多，体积较大                                 | 概念少，极其轻量                                        |

#### 总结

在 Vue 2/3 中，`store` 都是用来解决**跨组件复杂状态共享**问题的。如果说 `Router` 管理的是页面的映射，那么 `store` 管理的就是数据的流动。

从 Vue 2 到 Vue 3，状态管理的演变趋势是**“做减法”**。Vue 2 时代的 Vuex 强调严苛的单向数据流规范（引入 Mutations 层）；而到了 Vue 3 时代，官方用 Pinia 替代了 Vuex，去掉了冗余的 Mutations，拥抱了函数式编程，使得状态管理代码变得更直观、更易维护，并且获得了完美的 TS 支持。

### `VUE2/3 components`

在 Vue 2 和 Vue 3 中，`components`（组件）是整个 Vue 框架的**核心基石**。它的作用可以一句话概括：**将复杂的页面拆分成多个独立、可复用、可维护的小积木（区块）。**

需要明确的是，在 Vue 开发中，“components”通常包含两层含义：

1. **目录层面**：项目中的 `src/components` 文件夹，用于存放全局通用组件或 UI 组件（如按钮、弹窗、导航栏）。
2. **代码层面**：Vue 组件配置对象中的 `components` 选项，用于**局部注册**子组件。

无论是在 Vue 2 还是 Vue 3，组件的核心思想（高内聚、低耦合、数据驱动视图）是不变的，但在**注册方式、内部写法以及通信机制**上随着版本升级有了显著优化。

---

#### 一、 `components` 的共同核心作用（Vue 2 & Vue 3 通用）

1. **代码复用**
   将重复使用的 UI 和逻辑封装成一个组件（例如：`<MyButton>`、`<UserCard>`）。在多处需要使用时，只需引入标签即可，避免了代码的复制粘贴。
2. **逻辑隔离与模块化**
   每个组件拥有自己独立的状态、模板、样式和生命周期。组件内部的逻辑修改不会影响其他组件，极大地降低了大型项目的维护复杂度。
3. **简化页面结构**
   如果把一个复杂的首页全写在一个文件里，可能有几千行代码。通过组件化，主页面文件只需要负责拼接各个组件（如 `<Header />`、`<Sidebar />`、`<Footer />`），结构一目了然。
4. **隔离作用域**
   组件的 `data` 必须是一个函数返回对象（保证每个组件实例有独立的数据状态）；样式加上 `scoped` 后，各组件的 CSS 互不干扰。

---

#### 二、 Vue 2 中的 `components` 特点与用法

在 Vue 2 中，组件通常是一个包含 `template`、`data`、`methods` 等选项的普通 JavaScript 对象。组件通信主要依赖 `props`（父传子）、`$emit`（子传父）和 Event Bus（事件总线）。

**1. 局部注册（作为选项）：**

```javascript
import ChildComponent from './ChildComponent.vue'

export default {
  name: 'ParentPage',
  // 在这里注册子组件
  components: {
    ChildComponent // 键值一样可简写
  },
  data() {
    return { msg: 'Hello' }
  }
}
```

**2. 全局注册（在 main.js 中）：**

```javascript
import Vue from 'vue'
import MyButton from './components/MyButton.vue'

// 全局注册后，任何地方都可以直接使用 <MyButton />，无需引入
Vue.component('MyButton', MyButton)
```

---

#### 三、 Vue 3 中的 `components` 特点与用法

Vue 3 引入了 Composition API 和 `<script setup>` 语法糖，使得组件的写法发生了革命性的变化。**在 `<script setup>` 中，组件的局部注册变成了“零配置”的自动注册**。

**1. 局部注册（`<script setup>` 语法糖）：**
在 Vue 3 中，只要在 `<script setup>` 标签内 `import` 了一个组件，它就会**自动可用**于模板中，不再需要写 `components: { ... }` 选项。

```vue
<template>
  <!-- 直接使用，无需显式注册 -->
  <ChildComponent />
</template>

<script setup>
import ChildComponent from './ChildComponent.vue'
// 不需要写 components 选项！
</script>
```

**2. 全局注册（在 main.js 中）：**
Vue 3 不再使用 `Vue.component`，因为不再有全局的 `Vue` 构造函数。取而代之的是通过 `app` 实例注册。

```javascript
import { createApp } from 'vue'
import App from './App.vue'
import MyButton from './components/MyButton.vue'

const app = createApp(App)
// 通过 app 实例全局注册
app.component('MyButton', MyButton)
app.mount('#app')
```

**3. 异步组件（Vue 3 优化）**
在 Vue 3 中，定义异步组件（用于路由懒加载或组件按需加载）必须使用 `defineAsyncComponent` 函数，比 Vue 2 更加明确。

```javascript
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() => import('./AsyncComp.vue'))
```

---

#### 四、 Vue 2 与 Vue 3 中 `components` 的核心差异对比

| 特性                 | Vue 2                                            | Vue 3 (`<script setup>`)                                     |
| :------------------- | :----------------------------------------------- | :----------------------------------------------------------- |
| **局部注册方式**     | 必须在 `components: { }` 选项中手动声明          | **自动注册**，只要 `import` 引入即可在模板使用               |
| **多根节点**         | **不支持**，模板必须有唯一根标签包裹             | **支持**，可以在 `template` 中写多个平级的根标签             |
| **组件内部逻辑组织** | Options API（`data`, `methods`, `watch` 分散写） | Composition API，相关逻辑可聚合在一起，复用性更高            |
| **全局注册 API**     | `Vue.component('name', comp)`                    | `app.component('name', comp)`                                |
| **组件间通信**       | `$emit`、`$on`、`EventBus`（中央事件总线）       | 废弃了 `$on`/`$off`，EventBus 被淘汰。推荐使用 `mitt` 库或 `provide/inject`、`Pinia` |

#### 总结

`components` 是构建 Vue 应用的砖块。它的作用是**拆分和复用**。

从 Vue 2 到 Vue 3，组件的“作用”没有变，但“使用体验”大幅提升。Vue 3 的 `<script setup>` 彻底干掉了繁琐的 `components` 注册选项，实现了“引入即用”，同时支持了多根节点，并抛弃了容易造成混乱的 EventBus 通信模式，让组件的开发变得更加轻量、高效和类型安全。

### api

在前端开发语境下，提到 `api`，通常包含两层含义。结合你前面关于 `App.vue`、`Router`、`Store`、`Components` 的提问，这里主要指的是**项目目录中的 `src/api` 文件夹（接口管理模块）**。同时，由于 Vue 2 和 Vue 3 框架本身的演进，也会涉及**Vue 框架的 API（选项式 vs 组合式）**的变化。

我们分这两层来详细解答：

---

#### 第一层：项目架构中的 `src/api`（接口管理模块）

在 Vue 2 和 Vue 3 的项目中，`src/api` 的核心作用完全一致：**作为前端应用与后端服务器进行数据交互的统一门面。**

它将散落在各个组件内部的 `axios` 或 `fetch` 请求抽离出来，集中管理。

##### 共同核心作用（Vue 2 & Vue 3 通用）

1. **统一管理请求地址**：将后端的所有接口 URL 集中存放在 `api` 目录下，按业务模块划分（如 `user.js`、`order.js`）。避免接口地址在组件中硬编码，降低维护成本。
2. **封装请求逻辑**：在 `api` 文件中封装具体的请求方法。组件不再关心请求是怎么发的，只需要调用对应的方法并获取 Promise 返回的数据。
3. **配合全局拦截器**：通常与 `src/utils/request.js`（封装的 axios 实例）配合，在拦截器中统一处理 Token 注入、loading 显示、错误提示（如 401 跳转登录），而 `api` 文件只负责定义具体的业务接口。

##### 代码对比示例

**1. 封装文件 (`src/api/user.js`)** —— Vue 2 和 Vue 3 写法基本一致

```javascript
import request from '@/utils/request'

// 获取用户信息
export function getUserInfo(params) {
  return request({
    url: '/user/info',
    method: 'get',
    params
  })
}
```

**2. 组件中调用** —— 这里体现了 Vue 2 和 Vue 3 的差异

**Vue 2 组件中调用：**

```javascript
import { getUserInfo } from '@/api/user'

export default {
  data() {
    return { userInfo: {} }
  },
  created() {
    this.fetchUserInfo()
  },
  methods: {
    async fetchUserInfo() {
      try {
        const res = await getUserInfo({ id: 123 })
        this.userInfo = res.data
      } catch (error) {
        console.error(error)
      }
    }
  }
}
```

**Vue 3 组件中调用 (`<script setup>`)：**

```vue
<script setup>
import { ref, onMounted } from 'vue'
import { getUserInfo } from '@/api/user'

const userInfo = ref({})

const fetchUserInfo = async () => {
  try {
    const res = await getUserInfo({ id: 123 })
    userInfo.value = res.data
  } catch (error) {
    console.error(error)
  }
}

onMounted(() => {
  fetchUserInfo()
})
</script>
```

*优势：Vue 3 中调用 api 更加直观，不再需要挂载到 `methods` 上，逻辑组织更加内聚。*

---

#### 第二层：Vue 框架的 API（Options API vs Composition API）

Vue 2 到 Vue 3 最大的变化就是框架底层 API 的范式转变。

##### 1. Vue 2 的 Options API（选项式 API）

在 Vue 2 中，一个组件的代码结构由一系列固定的选项组成：`data`、`methods`、`computed`、`watch`、生命周期钩子。

* **作用**：规范了组件的编写结构，让初学者能快速按图索骥。
* **缺点**：当组件变得复杂时，同一个业务逻辑的代码会被拆散到 `data`、`methods`、`watch` 等不同选项中，导致代码来回跳转，难以维护。且难以进行有效的 TypeScript 类型推导。

##### 2. Vue 3 的 Composition API（组合式 API）

Vue 3 引入了 `setup` 函数（及 `<script setup>` 语法糖）和一系列以 `on`、`use` 开头的函数（如 `ref`、`reactive`、`onMounted`）。

* **作用**：允许将同一个业务逻辑关注点的代码（状态、方法、计算属性、监听器）聚合在一起写。
* **优势**：
  * **逻辑复用更优雅**：通过自定义 Hook（如 `useMousePosition`）取代了 Vue 2 容易导致命名冲突的 Mixins。
  * **更好的 TS 支持**：因为是纯 JavaScript 函数，天然具备完美的类型推导。

---

#### 总结对比表

| 维度                   | Vue 2                                   | Vue 3                                                |
| :--------------------- | :-------------------------------------- | :--------------------------------------------------- |
| **`src/api` 定位**     | 统一管理后端接口                        | 统一管理后端接口（无变化）                           |
| **`src/api` 调用方式** | 在 `methods` 中定义方法异步调用         | 在 `setup` 中直接定义函数并在 `onMounted` 等时机调用 |
| **Vue 框架 API 范式**  | Options API（选项式：data/methods分离） | Composition API（组合式：逻辑聚合）                  |
| **逻辑复用方案**       | Mixins（容易污染、命名冲突）            | 自定义 Hooks（`useXxx`，清晰隔离）                   |

**总而言之**：
架构层面的 `api` 作用是作为前后端数据交互的**统一代理层**，Vue 2/3 都不可或缺；而框架层面的 API，Vue 3 用组合式取代了选项式，是为了解决大型组件中**逻辑碎片化**的问题，让代码更像高内聚的函数式编程。

### `layouts`

在前端项目架构中，`layouts`（布局）通常不是一个内置的 Vue 核心概念（像 `Router` 或 `Store` 那样），而是一种**项目设计模式**。它通常以 `src/layouts` 文件夹的形式存在于项目中。

如果说 `App.vue` 是整个应用的“总外壳”，那么 `layouts` 就是**不同业务场景的“分外壳”**。它的核心作用是**提取页面级公共结构，实现多套页面骨架的复用**。

在 Vue 2 和 Vue 3 中，`layouts` 的设计思想完全一致，但由于 API 的不同，其实现方式和灵活度有显著差异。

---

#### 一、 `layouts` 的共同核心作用（Vue 2 & Vue 3 通用）

1. **页面级结构复用**
   在中后台管理系统中，大多数页面都有“顶部导航栏 + 左侧菜单栏 + 内容区”。如果不使用 `layouts`，每个页面（如用户管理、订单管理）都要复制粘贴一遍这几十行骨架代码。通过 `layouts`，只需写一次 `BasicLayout.vue`，所有常规页面套用即可。
2. **多套布局隔离**
   一个项目往往需要多种布局：
   * `BasicLayout.vue`：带导航栏和侧边栏的常规后台页面。
   * `UserLayout.vue`：仅居中显示一个卡片，用于登录/注册页面。
   * `BlankLayout.vue`：空白页面，用于打印页或全屏数据大屏。
3. **配合路由实现按需渲染**
   通过 Vue Router 的机制，在路由配置中声明某个页面使用哪种布局，将页面组件作为内容注入到布局的“插槽”中。

---

#### 二、 Vue 2 中的 `layouts` 实现方式

在 Vue 2 中，由于 Options API 和 Router 3 的特性，通常有两种主流的实现方式：

##### 方式一：嵌套路由（最常见）

将布局组件作为父路由，具体业务页面作为子路由。布局组件内部用 `<router-view>` 充当内容出口。

**路由配置 (`router.js`)：**

```javascript
const routes = [
  {
    path: '/',
    component: () => import('@/layouts/BasicLayout.vue'), // 使用基础布局
    children: [
      { path: 'dashboard', component: () => import('@/views/Dashboard.vue') },
      { path: 'user', component: () => import('@/views/UserList.vue') }
    ]
  },
  {
    path: '/login',
    component: () => import('@/layouts/BlankLayout.vue'), // 使用空白布局
    children: [
      { path: '', component: () => import('@/views/Login.vue') }
    ]
  }
]
```

**布局组件 (`BasicLayout.vue`)：**

```vue
<template>
  <div class="layout-container">
    <Header />
    <Sidebar />
    <main class="content">
      <!-- 子路由渲染在这里 -->
      <router-view />
    </main>
  </div>
</template>
```

##### 方式二：动态组件 + 路由 meta（进阶）

在 `App.vue` 中读取当前路由的 `meta` 信息，动态切换布局组件，并通过 Vue 2 的 `<router-view>` 深度嵌套或 `keep-alive` 配合。

---

#### 三、 Vue 3 中的 `layouts` 实现方式

Vue 3 引入了 `<script setup>` 和 Vue Router 4 的新特性（如 `<router-view>` 的 `v-slot`），使得布局的实现变得**极其优雅和灵活**。我们不再强制需要嵌套路由，而是可以在 `App.vue` 中直接接管渲染过程。

**路由配置 (`router/index.js`) - 利用 meta 标记布局：**

```javascript
const routes = [
  {
    path: '/dashboard',
    component: () => import('@/views/Dashboard.vue'),
    meta: { layout: 'BasicLayout' } // 声明使用的布局
  },
  {
    path: '/login',
    component: () => import('@/views/Login.vue'),
    meta: { layout: 'BlankLayout' } // 声明使用的布局
  }
]
```

**根组件 (`App.vue`) - 动态渲染布局：**
在 Vue 3 中，利用 `v-slot` 获取路由组件，并将其作为插槽内容传递给动态布局组件。

```vue
<template>
  <!-- 根据路由 meta 信息，动态解析为对应的布局组件 -->
  <component :is="layoutComponent">
    <!-- 将渲染好的业务页面组件，作为默认插槽传给布局组件 -->
    <router-view v-slot="{ Component }">
      <component :is="Component" />
    </router-view>
  </component>
</template>

<script setup>
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import BasicLayout from '@/layouts/BasicLayout.vue'
import BlankLayout from '@/layouts/BlankLayout.vue'

const route = useRoute()

// 预设布局映射
const layouts = {
  BasicLayout,
  BlankLayout
}

// 根据当前路由的 meta.layout 决定使用哪个布局，默认用 BasicLayout
const layoutComponent = computed(() => {
  return layouts[route.meta.layout] || BasicLayout
})
</script>
```

**布局组件 (`BasicLayout.vue`)：**

```vue
<template>
  <div class="layout-container">
    <Header />
    <Sidebar />
    <main class="content">
      <!-- 接收 App.vue 传进来的业务页面 -->
      <slot />
    </main>
  </div>
</template>
```

---

#### 四、 Vue 2 与 Vue 3 中 `layouts` 核心差异对比

| 特性                 | Vue 2 (Router 3)                                   | Vue 3 (Router 4)                                             |
| :------------------- | :------------------------------------------------- | :----------------------------------------------------------- |
| **主流实现机制**     | 依赖**嵌套路由** (`children`)，布局作为父组件      | 依赖**动态组件** (`<component :is>`) 与 **`v-slot`** 插槽传递 |
| **路由配置复杂度**   | 较高，需要多层级嵌套结构，不同布局要分开配置父路由 | 较低，路由扁平化，仅靠 `meta: { layout: 'Xxx' }` 标记即可    |
| **布局组件内容出口** | `<router-view />`                                  | `<slot />` (纯粹组件化，不再绑定路由概念)                    |
| **灵活度**           | 布局与路由强耦合                                   | 布局与路由解耦，任何组件都可以被任意布局包裹                 |
| **过渡动画支持**     | 直接在 `<router-view>` 外包 `<transition>`         | 通过 `v-slot` 拿到组件后，可自由控制 `<transition>` 和 `<keep-alive>` 的层级 |

#### 总结

`layouts` 的作用是管理页面的“骨架”，避免在每个业务页面中重复编写导航栏、侧边栏等公共 UI。

从 Vue 2 到 Vue 3，布局方案从“嵌套路由驱动”进化为了“动态组件与插槽驱动”。Vue 3 的方案让路由表变得更加干净扁平，布局组件也回归了纯粹的 UI 组件本质（只负责接收插槽内容），体现了 Vue 3 组合式 API 带来的极高灵活性。

### views

在前端 Vue 项目架构中，`views`（有时也被命名为 `pages`）的作用在 Vue 2 和 Vue 3 中是完全一致的：**它是页面级组件的容器，代表了应用中一个个独立的“页面”。**

如果打个比方：`components` 是搭建房屋用的砖头、门窗；`layouts` 是房屋的框架结构（承重墙）；而 `views` 就是最终装修好的一个个独立房间（如客厅、卧室、厨房）。

它在项目架构中扮演着承上启下的角色，是**路由直接映射的目标**。

---

#### 一、 `views` 的共同核心作用（Vue 2 & Vue 3 通用）

1. **路由的直接映射对象**
   在 `router/index.js` 中配置路由时，`component` 属性指向的通常是 `views` 目录下的文件。例如，访问 `/user/list` 时，路由就会把 `src/views/user/UserList.vue` 渲染到 `<router-view>` 中。
2. **页面级数据编排**
   `views` 通常不负责具体的 UI 细节（比如按钮长什么样），而是负责**数据组装和分发**。它会在内部调用 `api` 获取后端数据，从 `store` 获取全局状态，然后将这些数据作为 `props` 传递给子组件（`components`）去渲染。
3. **组装业务模块**
   一个 `view` 文件通常会将多个通用的基础组件拼装成一个完整的业务页面。比如一个“订单详情页”，会组合 `OrderHeader`、`OrderTable`、`OrderAction` 等多个组件，并协调它们之间的交互。
4. **配合 Layouts 渲染**
   `views` 中的页面最终会被注入到 `layouts` 定义好的骨架中（如填充到侧边栏右侧的内容区）。

---

#### 二、 `views` 与 `components` 的本质区别

在面试或项目规范中，经常会被问到为什么要分 `views` 和 `components`：

| 维度         | `src/views` (页面组件)                           | `src/components` (基础/通用组件)                         |
| :----------- | :----------------------------------------------- | :------------------------------------------------------- |
| **粒度**     | 宏观，对应一个完整的路由页面                     | 微观，对应页面内的某个 UI 区块                           |
| **路由关联** | **强关联**，每个 view 基本都有一个专属路由       | **无关联**，路由变化不会直接触发组件                     |
| **数据来源** | 主动出击：调用 API、读取 Store                   | 被动接收：通过 `props` 接收数据，通过 `$emit` 抛出事件   |
| **复用性**   | 极低，通常只在一个路由下使用                     | 极高，可以在不同页面甚至不同项目中复用                   |
| **命名规范** | 通常以业务模块或路由层级命名 (如 `UserList.vue`) | 通常以前缀描述功能 (如 `BaseButton.vue`, `UserCard.vue`) |

---

#### 三、 Vue 2 与 Vue 3 中 `views` 的写法对比

虽然 `views` 的架构作用不变，但由于 Vue 2 和 Vue 3 的 API 范式不同，页面组件的内部代码组织方式差异巨大。

##### Vue 2 中的 View (Options API)

在 Vue 2 中，页面逻辑被拆散在 `data`、`methods`、`created` 等不同选项中。如果页面复杂，相关逻辑会显得很分散。

```vue
<!-- src/views/user/UserList.vue -->
<template>
  <div class="page-user-list"> <!-- Vue 2 必须单根节点 -->
    <UserTable :data="userList" @delete="handleDelete" />
  </div>
</template>

<script>
import { getUserList, deleteUser } from '@/api/user'
import UserTable from '@/components/UserTable.vue'

export default {
  name: 'UserList',
  components: { UserTable },
  data() {
    return {
      userList: []
    }
  },
  created() {
    this.fetchData()
  },
  methods: {
    async fetchData() {
      const res = await getUserList()
      this.userList = res.data
    },
    async handleDelete(id) {
      await deleteUser(id)
      this.fetchData()
    }
  }
}
</script>
```

##### Vue 3 中的 View (Composition API + `<script setup>`)

在 Vue 3 中，页面逻辑可以按“功能”聚合在一起，且不再需要手动注册组件，代码更加紧凑、直观。也可以直接使用 `useRoute` 获取路由参数。

```vue
<!-- src/views/user/UserList.vue -->
<template>
  <!-- Vue 3 支持多根节点 -->
  <div class="page-header">用户管理</div>
  <UserTable :data="userList" @delete="handleDelete" />
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import { getUserList, deleteUser } from '@/api/user'
import UserTable from '@/components/UserTable.vue' // 自动注册，无需 components 选项

const route = useRoute()
const userList = ref([])

// 数据获取逻辑聚合
const fetchData = async () => {
  const res = await getUserList({ pageId: route.query.page })
  userList.value = res.data
}

// 删除逻辑聚合
const handleDelete = async (id) => {
  await deleteUser(id)
  fetchData()
}

onMounted(() => {
  fetchData()
})
</script>
```

#### 总结

`views` 是 Vue 项目中最接近用户所见所操作的“物理页面”的一层。它的作用是**接收路由调度、拉取业务数据、编排通用组件**，最终呈现出完整的界面。

无论是 Vue 2 还是 Vue 3，`views` 在项目目录结构中的定位和职责划分都是不变的。变化的主要是其内部的代码组织方式：Vue 3 的 `<script setup>` 让页面级组件的代码变得更加内聚、简洁，彻底摆脱了选项式 API 中逻辑分散的问题。

### utils

在前端 Vue 项目架构中，`utils`（通常对应 `src/utils` 目录）的作用在 Vue 2 和 Vue 3 中是**完全一致且与框架版本无关的**。

它的核心作用是：**存放与业务无关的、通用的、纯粹的 JavaScript/TypeScript 工具函数库。**

你可以把 `utils` 理解为项目中的**“瑞士军刀”或“工具箱”**。它里面的函数不关心 Vue 的组件、路由或状态管理，只负责处理最基础的数据逻辑。

---

#### 一、 `utils` 的共同核心作用（Vue 2 & Vue 3 通用）

1. **代码复用（DRY 原则）**
   将在多个组件或页面中反复使用的纯逻辑代码抽取出来。例如：格式化日期、深拷贝对象、生成随机 UUID 等。避免在不同文件中复制粘贴相同的逻辑。
2. **保持业务代码简洁**
   让 `views` 和 `components` 中的代码更专注于业务和 UI 交互，将底层的杂活、脏活交给 `utils` 处理。
3. **纯函数与无副作用**
   `utils` 中的函数理想情况下应该是“纯函数”——输入相同的参数，永远返回相同的结果，且不修改外部状态。这使得它们极易进行单元测试。

---

#### 二、 `utils` 目录通常包含哪些内容？

在实际项目中，`utils` 通常会按功能拆分成多个文件：

*   **`format.js` (格式化处理)**：时间格式化、金额千分位转换、文件大小格式化。
*   **`validate.js` (校验工具)**：手机号校验、邮箱校验、身份证校验、URL 合法性校验。
*   **`storage.js` (本地存储)**：对 `localStorage`、`sessionStorage` 的封装，加入 JSON 序列化/反序列化或过期时间处理。
*   **`auth.js` (权限/令牌)**：获取、设置、删除 Token（通常结合 `storage.js`）。
*   **`dom.js` (DOM 操作)**：节流、防抖函数，滚动条位置计算，元素 class 操作（虽然 Vue 减少了直接 DOM 操作，但偶尔仍需要）。
*   **`request.js` (axios 封装)**：这通常也放在 `utils` 中，用于创建 axios 实例，配置拦截器（处理 Token、统一错误处理）。

---

#### 三、 Vue 2 与 Vue 3 中的差异：函数的定义无差异，调用方式有差异

由于 `utils` 存放的是纯 JS 代码，**函数本身的写法在 Vue 2 和 Vue 3 中没有任何区别**。差异主要体现在**组件中如何调用这些工具函数**。

##### 1. 工具函数的写法（Vue 2/3 通用）

```javascript
// src/utils/format.js

// 格式化日期
export const formatDate = (date, fmt = 'YYYY-MM-DD') => {
  // 纯 JS 逻辑处理
  // ...
  return formattedDate
}

// 金额格式化
export const formatMoney = (num) => {
  return Number(num).toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ',')
}
```

##### 2. 组件中的调用方式对比

**Vue 2 中的调用 (Options API)：**
在 Vue 2 中，通常在 `methods` 或 `filters` 中引入并使用工具函数。有时为了方便，会将其挂载到 Vue 原型上（`Vue.prototype.$formatDate`），但这容易导致命名污染。

```javascript
import { formatDate } from '@/utils/format'

export default {
  data() {
    return { createTime: new Date() }
  },
  computed: {
    // 在计算属性中调用工具函数
    displayTime() {
      return formatDate(this.createTime)
    }
  }
}
```

**Vue 3 中的调用 (Composition API + `<script setup>`)：**
在 Vue 3 中，直接在 `<script setup>` 顶部引入并在任意位置使用即可。不再需要挂载到全局原型上，符合模块化开发的理念，且 Tree-shaking 更加友好。

```vue
<template>
  <div>{{ displayTime }}</div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { formatDate } from '@/utils/format' // 直接引入工具函数

const createTime = ref(new Date())

// 在计算属性中直接调用
const displayTime = computed(() => {
  return formatDate(createTime.value)
})
</script>
```

---

#### 四、 `utils` 与 `api`、`store` 的边界划分

在项目开发中，开发者有时会混淆 `utils`、`api` 和 `store` 的职责，以下是明确的边界：

| 目录        | 核心职责                                                     | 依赖关系                                         | 示例                                |
| :---------- | :----------------------------------------------------------- | :----------------------------------------------- | :---------------------------------- |
| **`utils`** | 处理**纯前端基础逻辑**，不涉及任何后端接口，不涉及 Vue 响应式。 | 纯 JS，零依赖（或仅依赖第三方库如 dayjs/lodash） | `formatDate`、`debounce`、`isEmail` |
| **`api`**   | 负责与**后端服务器通信**，定义所有的 HTTP 请求地址和请求方法。 | 依赖 `utils/request.js` (axios封装)              | `getUserList()`、`login()`          |
| **`store`** | 负责管理**跨组件共享的响应式状态**。                         | 依赖 Vue 框架，可能调用 `api`                    | `userStore.token`、`cartStore.list` |

#### 总结

在 Vue 2 和 Vue 3 项目中，`utils` 都是不可或缺的基础设施层。它的作用是**沉淀通用的、与业务解耦的纯逻辑工具**。

无论你使用的是 Vue 2 的 Options API 还是 Vue 3 的 Composition API，`utils` 目录的代码几乎可以做到 100% 移植复用。变化仅仅在于组件内部引入和使用这些工具函数的代码组织形式变得更加直观和模块化了。





### npm run dev / test / prod 

> 核心前提（面试必背）
> 1. **`dev`、`test`、`prod` 不是 npm 内置命令**，全部是 `package.json` → `scripts` 自定义脚本名称，名字完全可以自定义。
> 2. npm run 的本质：`npm run xxx` 会读取 scripts 字段后面的命令字符串，在当前项目目录下执行 shell 指令。
> 3. **区分两个概念**：
>    - **脚本名(dev/test/prod)**：人为约定的别名；
>    - **mode(环境模式 development / test / production)**：Vite/Webpack 识别的环境标识，控制打包策略、加载对应环境变量。

> 补充：`build` 是**打包动作**，不是环境。`vite build` 默认是 production 模式，但你可以手动指定 `--mode test` 打出测试包。

#### 一、基础示例 package.json

```json
{
  "scripts": {
    "dev": "vite --mode development",
    "test": "vite build --mode test",
    "prod": "vite build --mode production",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

---

#### 二、逐个详解

##### 1. npm run dev（开发环境 development）

**作用：启动本地开发服务器，用于开发、写代码、前后端联调**
底层命令：`vite` / `webpack serve`（vue-cli）
```json
"dev": "vite --mode development"
```
###### ✅ 核心特性

1. **启动内存级开发服务**：代码不会输出到磁盘dist目录，全部在内存运行；访问 `localhost:5173`（Vite）或 `8080`（VueCLI）
2. **HMR 热模块更新**：修改组件/JS/CSS，浏览器局部更新，**不需要整页刷新**，开发效率极高
3. **不做代码压缩、Tree-Shaking 不完整**，保留完整 sourcemap，方便浏览器断点调试
4. **自动加载环境文件 `.env.development`**，读取开发环境变量（开发后端接口地址、调试开关）
5. 保留所有 console、debug 代码，不做代码混淆

###### ✅ 使用场景

本地开发、写页面、联调后端开发环境接口。
> ⚠️ 重要坑：**dev 的内置开发服务器绝对不能直接公网上线**。
> 性能差、内存占用高、没有安全优化，只适合本地开发。

> VueCLI老项目别名：`npm run serve`，等价于 dev。

---

##### 2. npm run test（测试环境 test / UAT）

> 这个脚本有**两种完全不同的业务含义**，看scripts后面的命令，是前端面试高频考点。

###### 场景A：打包【UAT测试环境包】（微服务前后端分离项目最常用）

```json
"test": "vite build --mode test"
```
- 执行**打包构建**（本质也是build动作），输出dist静态文件夹
- 加载环境文件：`.env.test`，读取测试环境接口地址（UAT测试后端，独立测试数据库，和生产隔离）
- 打包策略：**开启代码压缩、Tree-Shaking**，和生产打包规则几乎一致；
- 差异：一般会保留监控埋点、错误日志，不会关闭调试日志，方便测试人员定位bug；
- 产物dist上传到测试服务器，Nginx托管，测试人员做功能测试、回归测试、验收测试。

###### 场景B：单元测试脚本（Jest / Vitest）

```json
"test": "vitest run"
```
> 和页面打包完全无关！
> 不编译页面，只执行JS/TS单元测试用例，校验工具函数、组件逻辑正确性。

> 判断技巧：scripts后面是 `vite build` 就是打包测试包；是 `vitest/jest` 就是单元测试。

---

#### 3. npm run prod（生产环境 production）

> prod 是**生产环境打包脚本**，专门用于生成上线包。
```json
"prod": "vite build --mode production"
```
##### ✅ 核心特性

1. 执行打包构建，输出dist静态资源目录；
2. 加载环境文件：`.env.production`，读取**正式生产后端接口**，访问真实业务数据库；
3. 完整生产优化：
   - JS/CSS压缩、代码混淆
   - Tree-Shaking 删除未使用代码
   - 图片静态资源压缩
   - 默认移除 `console.log`（可配置）
   - 默认不生成sourcemap（线上防止源码泄露，可按需开启）
4. 产物dist交给Nginx部署，对外提供公网访问，给真实用户使用。

> 补充：少数项目里 `"prod": "vite preview"`，只是本地静态预览dist，**不能当做线上服务**。

#### 三、Vite 环境变量加载规则（重点）

Vite 根据 `--mode` 参数加载环境文件，**优先级由低到高，后面覆盖前面**
1. `.env`                全局公共变量，所有环境都会加载
2. `.env.{mode}`         对应环境文件：`.env.development` / `.env.test` / `.env.production`
3. `.env.{mode}.local`   本地私有环境文件，一般加入.gitignore，只在本机生效，不提交到代码仓库

> 环境变量读取（Vite）
```js
import.meta.env.MODE          // 获取当前模式：development / test / production
import.meta.env.VITE_API_URL  // 自定义环境变量，Vite要求必须VITE_开头
```
> VueCLI(Webpack)：变量前缀 `VUE_APP_`，用 `process.env` 获取。

#### 四、横向对比汇总表

| 脚本 | 底层动作 | 是否生成dist | 读取环境文件 | 主要用途 | 代码压缩 |
|---|---|---|---|---|---|
| npm run dev | 启动内存开发服务 | ❌ 无dist | .env.development | 本地开发、联调 | ❌ 不压缩，保留调试代码 |
| npm run test(打包场景) | vite build打包 | ✅ dist | .env.test | UAT测试，测试人员验收 | ✅ 压缩，保留埋点日志 |
| npm run prod | vite build打包 | ✅ dist | .env.production | 正式线上部署 | ✅ 完整压缩、清除调试代码 |

#### 五、高频面试考点&易错点

1. ❌ 误区：dev/test/prod是npm自带命令
✅ 答案：不是，全部是scripts自定义脚本，名称可以随便改，比如online、uat。

2. ❌ 误区：build就是生产打包
✅ 答案：build只是打包动作，`vite build --mode test` 打出的就是测试包；build本身不代表生产环境。

3. ❌ 误区：NODE_ENV 和 mode 完全一样
✅ 答案：
- `mode`：Vite用来加载 `.env.xxx` 文件；
- `NODE_ENV`：控制打包优化策略（production开启压缩）。
Vite打包时，只要mode=production，自动把NODE_ENV置为production。

4. ❌ 误区：打包后的dist可以直接双击打开index.html
✅ 答案：大部分情况不行，路由history模式需要Nginx配置，直接打开会404。

5. 环境变量必须加前缀，否则前端无法读取：Vite是`VITE_`，VueCLI是`VUE_APP_`。

#### 六、面试精简口述版（直接背）

> dev、test、prod是package.json scripts中自定义脚本，非npm原生命令。
> npm run dev：启动本地开发服务，内存运行，开启HMR热更新，用于本地开发调试，读取development环境变量，不压缩代码。
> npm run test分两种：一种是打包测试UAT环境包，用于测试人员验收；另一种是vitest/jest执行单元测试。
> npm run prod：以production模式打包，生成dist静态资源，代码压缩混淆，读取生产环境变量，用于Nginx部署上线。
> build只是打包动作，build可以打包测试环境，也可以打包生产环境，不能等同于prod。

#### 七、完整工程模板（可直接复制使用）

##### package.json

```json
{
  "name": "demo",
  "scripts": {
    "dev": "vite --mode development",
    "test": "vite build --mode test",
    "prod": "vite build --mode production",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```
##### 环境文件（4个，放在项目根目录）

`.env`
```env
VITE_APP_TITLE=前端项目
```
`.env.development`

```env
VITE_API_URL=[http://127.0.0.1:8080](http://127.0.0.1:8080)
```
`.env.test`
```env
VITE_API_URL=shturl.cc/MQZ83dAdLsCP
```
`.env.production`
```env
VITE_API_URL=shturl.cc/CzJRt8go
```

如果你需要，我可以继续：
1. 讲解history路由Nginx配套配置（打包后404问题）
2. 区分 `NODE_ENV` 和 `import.meta.env` 的面试题
3. 跨环境打包时的常见问题（环境变量不生效、打包体积过大）

需要哪个？



# 绝对/相对路径

## 一、基本概念

### 1. **相对路径（Relative Path）**

相对于当前文件位置的路径。

```javascript
// 当前文件：src/views/user/UserList.vue

// 同目录
import Dialog from './Dialog.vue'           // ./ 表示当前目录

// 上级目录
import API from '../../api/user.js'         // ../ 表示上级目录

// 上上级目录
import utils from '../../../utils/index.js' // ../../ 表示上上级目录
```

### 2. **绝对路径（Absolute Path）**

从项目根目录或别名开始的完整路径。

```javascript
// 使用 @ 别名（推荐）
import API from '@/api/user.js'
import utils from '@/utils/index.js'
import Dialog from '@/views/user/Dialog.vue'

// 使用完整绝对路径（不推荐，太冗长）
import API from 'C:/project/src/api/user.js'  // ❌ 不要用
```

---

## 二、Vue 3 中的路径配置

### 1. **Vite 项目配置（vite.config.js）**

```javascript
// 1. 导入 vite 内置配置函数
import { defineConfig } from 'vite'
// 2. 导入 Node.js 内置路径处理模块 path
import { resolve } from 'path'

// 3. 导出 Vite 全局配置对象
export default defineConfig({
  // 解析相关配置
  resolve: {
    // 路径别名配置数组/对象
    alias: {
      // 简写 @ 等价于项目根目录下 src 文件夹
      '@': resolve(__dirname, 'src'),
      // @components 直接指向 src/components
      '@components': resolve(__dirname, 'src/components'),
      // 工具函数目录
      '@utils': resolve(__dirname, 'src/utils'),
      // 接口请求目录
      '@api': resolve(__dirname, 'src/api'),
      // 静态资源（图片、字体、样式）
      '@assets': resolve(__dirname, 'src/assets'),
      // 页面视图目录（Vue页面/React页面）
      '@views': resolve(__dirname, 'src/views'),
    }
  }
})
```

#### `resolve`（来自 Node.js path 模块）

作用：**拼接、规范化绝对文件路径**

语法：`resolve(基础路径, 拼接路径)`

#### `__dirname`

Node.js 全局变量：**当前 vite.config 文件所在的文件夹绝对路径**（项目根目录）。

#### alias 路径别名（最核心）

alias = 路径映射：把长的相对路径，替换成简短符号，解决**层级嵌套过深、相对路径书写麻烦**的痛点

### 2. **Vue CLI 项目配置（vue.config.js）**

```javascript
const path = require('path')

module.exports = {
  chainWebpack: (config) => {
    config.resolve.alias
      .set('@', path.resolve(__dirname, 'src'))
      .set('@components', path.resolve(__dirname, 'src/components'))
      .set('@utils', path.resolve(__dirname, 'src/utils'))
      .set('@api', path.resolve(__dirname, 'src/api'))
  }
}
```

### 3. **TypeScript 支持（tsconfig.json）**

如果使用 TypeScript，需要同步配置：

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"],
      "@api/*": ["src/api/*"]
    }
  }
}
```

---

## 三、实际使用场景对比

### 场景 1：深层目录导入

**项目结构：**

```
src/
├── api/
│   └── user.js
├── utils/
│   └── index.js
├── components/
│   └── Header.vue
└── views/
    └── admin/
        └── settings/
            └── Profile.vue  ← 当前文件
```

**❌ 使用相对路径：**

```javascript
// Profile.vue（在 views/admin/settings/ 下）
import { getUser } from '../../../../api/user.js'
import { formatDate } from '../../../../utils/index.js'
import Header from '../../../../components/Header.vue'
```

**✅ 使用绝对路径（别名）：**

```javascript
// Profile.vue
import { getUser } from '@/api/user.js'
import { formatDate } from '@/utils/index.js'
import Header from '@/components/Header.vue'
```

---

### 场景 2：文件移动时

**假设把 `Profile.vue` 从 `views/admin/settings/` 移到 `views/user/`**

**❌ 相对路径需要全部修改：**

```javascript
// 修改前（在 views/admin/settings/）
import { getUser } from '../../../../api/user.js'

// 修改后（在 views/user/）
import { getUser } from '../../../api/user.js'  // 需要改
```

**✅ 绝对路径不需要修改：**

```javascript
// 无论文件在哪，都不用改
import { getUser } from '@/api/user.js'  // 永远不变
```

---

## 四、路径语法详解

### 1. **相对路径符号**

| 符号     | 含义       | 示例                          |
| -------- | ---------- | ----------------------------- |
| `./`     | 当前目录   | `import A from './A.vue'`     |
| `../`    | 上级目录   | `import B from '../B.vue'`    |
| `../../` | 上上级目录 | `import C from '../../C.vue'` |

### 2. **绝对路径符号**

| 符号 | 含义                     | 示例                                 |
| ---- | ------------------------ | ------------------------------------ |
| `@/` | src 目录（别名）         | `import A from '@/components/A.vue'` |
| `/`  | 项目根目录               | 不常用，通常用别名                   |
| `~`  | node_modules（部分工具） | `import 'lodash'`                    |

---

## 五、不同类型文件的导入

### 1. **导入 Vue 组件**

```javascript
// 相对路径
import MyComponent from './MyComponent.vue'
import Parent from '../Parent.vue'

// 绝对路径（推荐）
import MyComponent from '@/components/MyComponent.vue'
import Header from '@/components/layout/Header.vue'
```

### 2. **导入 JavaScript/TypeScript 文件**

```javascript
// 相对路径
import { formatDate } from '../../utils/date.js'
import API from '../api/user.ts'

// 绝对路径（推荐）
import { formatDate } from '@/utils/date.js'
import API from '@/api/user.ts'
```

### 3. **导入 CSS/SCSS 文件**

```vue
<style scoped>
/* 相对路径 */
@import './styles/custom.css';
@import '../theme/variables.scss';

/* 绝对路径（推荐） */
@import '@/assets/styles/global.css';
@import '@/assets/theme/variables.scss';
</style>
```

### 4. **导入静态资源（图片等）**

```vue
<template>
  <!-- 相对路径 -->
  <img src="./assets/logo.png" />
  
  <!-- 绝对路径（需要配置） -->
  <img src="@/assets/logo.png" />
  
  <!-- public 目录（绝对路径，不需要 @） -->
  <img src="/logo.png" />
</template>
```

---

## 六、API 请求中的路径

### 1. **Axios 配置**

```javascript
// src/api/request.js
import axios from 'axios'

const request = axios.create({
  baseURL: '/api',  // 相对路径，会拼接当前域名
  timeout: 10000
})

export default request
```

### 2. **接口调用**

```javascript
// src/api/user.js
import request from '@/api/request'  // 绝对路径

// ✅ 使用相对路径（推荐，自动拼接 baseURL）
export function getUser(id) {
  return request.get(`/user/${id}`)  // 实际请求：/api/user/1
}

// ❌ 不要写完整 URL（除非调用外部接口）
export function getUser(id) {
  return axios.get('http://localhost:8080/api/user/' + id)
}
```

---

## 七、路由中的路径

### 1. **路由配置**

```javascript
// src/router/index.js
import { createRouter } from 'vue-router'

const routes = [
  {
    path: '/user',           // 路由路径（绝对路径，以 / 开头）
    component: () => import('@/views/user/UserList.vue')  // 组件导入用 @
  },
  {
    path: 'profile',         // 相对路径（不以 / 开头，相对于父路由）
    component: () => import('./Profile.vue')
  }
]
```

### 2. **路由跳转**

```javascript
// ✅ 绝对路径跳转
this.$router.push('/user/list')
this.$router.push({ path: '/user/profile' })

// ✅ 命名路由（推荐，更安全）
this.$router.push({ name: 'UserProfile', params: { id: 123 } })

// ✅ 相对路径跳转（相对于当前路由）
this.$router.push('profile')        // 当前 /user → /user/profile
this.$router.push('../settings')    // 当前 /user/profile → /settings
```

---

## 八、最佳实践

### ✅ 推荐做法：

```javascript
// 1. 使用 @ 别名导入 src 下的文件
import API from '@/api/user'
import utils from '@/utils/date'
import Header from '@/components/Header.vue'

// 2. 同目录或相邻目录用相对路径
import Dialog from './Dialog.vue'
import Parent from '../Parent.vue'

// 3. 路由配置用绝对路径
this.$router.push('/user/list')

// 4. API 请求用相对路径
axios.get('/api/user/1')
```

### ❌ 不推荐做法：

```javascript
// 1. 使用过多的 ../
import API from '../../../../api/user'  // ❌ 难以阅读

// 2. 使用完整文件系统路径
import API from 'C:/project/src/api/user'  // ❌ 无法移植

// 3. 混用风格不统一
import A from './A'
import B from '@/components/B'  // 同层却用不同方式
```

---

## 九、常见问题

### Q1: 什么时候用 `@/`，什么时候用 `./`？

**规则：**

- 导入 `src/` 下的文件 → 用 `@/`
- 导入同目录或相邻目录 → 用 `./` 或 `../`

```javascript
// ✅ 推荐
import API from '@/api/user'              // src/api
import Dialog from './Dialog.vue'         // 同目录
import Parent from '../Parent.vue'        // 上级目录
```

### Q2: 为什么图片路径有时不用 `@/`？

```vue
<template>
  <!-- src/assets 中的图片，用 @/ -->
  <img src="@/assets/logo.png" />
  
  <!-- public 目录的图片，直接用 / -->
  <img src="/logo.png" />
</template>
```

**原因：**

- `src/assets/` → 会被 Webpack/Vite 处理，需要 `@/`
- `public/` → 直接复制到构建目录，用 `/` 访问

### Q3: TypeScript 中路径提示不生效？

**解决：** 配置 `tsconfig.json` 和 `vite.config.js`/`vue.config.js`

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

---

## 十、总结对比表

| 特性         | 相对路径 `./` `../` | 绝对路径 `@/` |
| ------------ | ------------------- | ------------- |
| **可读性**   | 深层目录混乱        | 清晰直观      |
| **文件移动** | 需要修改路径        | 不需要修改    |
| **配置**     | 无需配置            | 需要配置别名  |
| **适用场景** | 同目录/相邻目录     | 跨目录导入    |
| **团队协作** | 容易出错            | 统一规范      |
| **推荐度**   | ⭐⭐⭐                 | ⭐⭐⭐⭐⭐         |



# `this.$`(vue2为主)

在 Vue 2/3 中，**`this.$` 前缀**代表 Vue 实例的**内置全局属性和方法**，以及**插件注入的 API**。

---

## 一、Vue 内置的 `$` 属性

### 1. **`this.$data`** - 组件数据对象

指向组件 `data()` 返回的对象。

```javascript
export default {
  data() {
    return {
      message: 'Hello',
      count: 0,
      user: { name: '张三', age: 25 }
    };
  },
  mounted() {
    console.log(this.$data);
    // 输出: { message: 'Hello', count: 0, user: { name: '张三', age: 25 } }
    
    console.log(this.$data.message);  // 'Hello'
    
    // 等价于
    console.log(this.message);  // 'Hello'
  }
}
```

**使用场景：**

```javascript
// 1. 重置数据到初始状态
resetData() {
  Object.assign(this.$data, this.$options.data.call(this));
}

// 2. 克隆整个数据对象
const dataClone = JSON.parse(JSON.stringify(this.$data));

// 3. 遍历所有数据
Object.keys(this.$data).forEach(key => {
  console.log(`${key}: ${this.$data[key]}`);
});
```

---

### 2. **`this.$props`** - 组件 Props

包含父组件传递给当前组件的所有 props。

```javascript
export default {
  props: {
    title: { type: String, required: true },
    count: { type: Number, default: 0 },
    visible: Boolean
  },
  mounted() {
    console.log(this.$props);
    // 输出: { title: '标题', count: 10, visible: true }
    
    console.log(this.$props.title);  // '标题'
    
    // 等价于
    console.log(this.title);  // '标题'
  }
}
```

**使用场景：**

```javascript
// 1. 传递所有 props 给子组件
<child-component v-bind="$props" />

// 2. 监听所有 props 变化
watch: {
  $props: {
    handler(newProps) {
      console.log('Props 变化:', newProps);
    },
    deep: true
  }
}

// 3. 过滤特定 props
const stringProps = Object.entries(this.$props)
  .filter(([key, value]) => typeof value === 'string');
```

---

### 3. **`this.$el`** - 组件根 DOM 元素

指向组件模板的根 DOM 元素。

```vue
<template>
  <div class="container">
    <h1>标题</h1>
    <p>内容</p>
  </div>
</template>

<script>
export default {
  mounted() {
    console.log(this.$el);
    // 输出: <div class="container">...</div>
    
    // 操作 DOM
    this.$el.style.backgroundColor = '#f0f0f0';
    
    // 查找子元素
    const h1 = this.$el.querySelector('h1');
    console.log(h1.textContent);  // '标题'
    
    // 获取所有子元素
    const children = this.$el.querySelectorAll('p');
    console.log(children.length);  // 1
  }
}
</script>
```

**使用场景：**

```javascript
// 1. 获取元素尺寸
const rect = this.$el.getBoundingClientRect();
console.log('宽度:', rect.width, '高度:', rect.height);

// 2. 添加事件监听
this.$el.addEventListener('click', this.handleClick);

// 3. 滚动到元素位置
this.$el.scrollIntoView({ behavior: 'smooth' });

// 4. 获取父元素
const parent = this.$el.parentElement;
```

---

### 4. **`this.$options`** - 组件选项

包含组件初始化时的所有选项（data、methods、props、自定义选项等）。

```javascript
export default {
  name: 'MyComponent',
  props: { title: String },
  data() {
    return { message: 'Hello' };
  },
  methods: {
    handleClick() {}
  },
  customOption: 'custom value',
  
  mounted() {
    console.log(this.$options.name);         // 'MyComponent'
    console.log(this.$options.customOption); // 'custom value'
    console.log(this.$options.props);        // { title: { type: [Function: String] } }
    console.log(this.$options.methods);      // { handleClick: [Function] }
    
    // 获取 data 初始值
    const initialData = this.$options.data.call(this);
    console.log(initialData);  // { message: 'Hello' }
  }
}
```

**使用场景：**

```javascript
// 1. 重置数据到初始状态
resetData() {
  const initialData = this.$options.data.call(this);
  Object.assign(this.$data, initialData);
}

// 2. 获取组件名称
console.log(this.$options.name);

// 3. 访问自定义选项
const customValue = this.$options.customOption;
```

---

### 5. **`this.$refs`** - 模板引用

访问模板中带有 `ref` 属性的 DOM 元素或子组件。

```vue
<template>
  <div>
    <!-- DOM 元素引用 -->
    <input ref="myInput" type="text" />
    
    <!-- 组件引用 -->
    <child-component ref="childComp" />
    
    <!-- 多个元素引用（v-for） -->
    <div v-for="item in list" :key="item.id" ref="itemList">
      {{ item.name }}
    </div>
    
    <!-- Element Plus 组件 -->
    <el-form ref="formRef">
      <el-form-item prop="name">
        <el-input v-model="form.name" />
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      list: [{ id: 1, name: 'A' }, { id: 2, name: 'B' }],
      form: { name: '' }
    };
  },
  methods: {
    // 1. 聚焦输入框
    focusInput() {
      this.$refs.myInput.focus();
      this.$refs.myInput.value = 'Hello';
    },
    
    // 2. 调用子组件方法
    callChildMethod() {
      this.$refs.childComp.childMethod();
    },
    
    // 3. 表单验证
    validateForm() {
      this.$refs.formRef.validate((valid) => {
        if (valid) {
          console.log('验证通过');
        }
      });
    },
    
    // 4. 重置表单
    resetForm() {
      this.$refs.formRef.resetFields();
    },
    
    // 5. 访问 v-for 中的多个元素
    getItemList() {
      console.log(this.$refs.itemList);
      // 输出: [div, div] (DOM 元素数组)
    }
  }
}
</script>
```

**⚠️ 注意事项：**

```javascript
// ❌ 错误：在 created 中访问（DOM 还没渲染）
created() {
  console.log(this.$refs.myInput);  // undefined
}

// ✅ 正确：在 mounted 或之后访问
mounted() {
  console.log(this.$refs.myInput);  // <input> 元素
}

// ✅ 使用 $nextTick 确保 DOM 已更新
this.message = 'new';
this.$nextTick(() => {
  console.log(this.$refs.myInput);  // 可以访问
});
```

---

### 6. **`this.$attrs`** - 未注册的属性

包含父组件传递但未被 props 接收的属性。

```vue
<!-- 父组件 -->
<child-component 
  id="myId" 
  class="myClass" 
  data-test="test"
  :title="'标题'"  <!-- 被 props 接收 -->
/>

<!-- 子组件 -->
<script>
export default {
  props: {
    title: String  // 只注册了 title
  },
  mounted() {
    console.log(this.$attrs);
    // 输出: { id: 'myId', class: 'myClass', 'data-test': 'test' }
  }
}
</script>
```

**使用场景：**

```vue
<!-- 传递所有 attrs 给子元素 -->
<template>
  <div v-bind="$attrs">
    <slot />
  </div>
</template>

<script>
export default {
  inheritAttrs: false  // 阻止默认继承行为
}
</script>
```

---

### 7. **`this.$slots`** - 插槽内容

访问父组件传递的插槽内容。

```vue
<template>
  <div>
    <!-- 默认插槽 -->
    <slot />
    
    <!-- 命名插槽 -->
    <slot name="header" />
    <slot name="footer" />
  </div>
</template>

<script>
export default {
  mounted() {
    console.log(this.$slots.default);  // 默认插槽内容
    console.log(this.$slots.header);   // header 插槽内容
    console.log(this.$slots.footer);   // footer 插槽内容
  }
}
</script>
```

---

## 二、Vue 内置的 `$` 方法

### 1. **`this.$emit(eventName, [...args])`** - 触发自定义事件

向父组件发送事件。

```vue
<!-- 子组件 -->
<script>
export default {
  methods: {
    // 1. 简单事件
    handleClose() {
      this.$emit('close');
    },
    
    // 2. 传递数据
    handleUpdate() {
      this.$emit('update', { id: 1, name: '张三' });
    },
    
    // 3. 多个参数
    handleConfirm() {
      this.$emit('confirm', 'success', { data: 'value' });
    },
    
    // 4. update:xxx 模式（用于 v-model）
    handleInput(value) {
      this.$emit('update:modelValue', value);
    }
  }
}
</script>

<!-- 父组件使用 -->
<child-component 
  @close="handleClose"
  @update="handleUpdate"
  @confirm="handleConfirm"
/>
```

**Vue 3 推荐写法（defineEmits）：**

```vue
<script setup>
const emit = defineEmits(['close', 'update', 'confirm']);

const handleClose = () => {
  emit('close');
};
</script>
```

---

### 2. **`this.$on(event, callback)`** - 监听事件（Vue 2）

⚠️ **注意：** Vue 3 已移除 `$on`、`$off`、`$once`，建议使用 mitt 或 mitt 库。

```javascript
// Vue 2
export default {
  mounted() {
    // 监听事件
    this.$on('custom-event', (data) => {
      console.log('收到事件:', data);
    });
    
    // 只监听一次
    this.$once('once-event', () => {
      console.log('只触发一次');
    });
  },
  beforeDestroy() {
    // 移除监听
    this.$off('custom-event');
  }
}
```

---

### 3. **`this.$watch(expOrFn, callback, [options])`** - 监听数据变化

```javascript
export default {
  data() {
    return {
      message: 'Hello',
      user: { name: '张三', age: 25 },
      list: [1, 2, 3]
    };
  },
  mounted() {
    // 1. 监听单个属性
    this.$watch('message', (newVal, oldVal) => {
      console.log('message 变化:', oldVal, '->', newVal);
    });
    
    // 2. 深度监听对象
    this.$watch('user', (newVal) => {
      console.log('user 变化:', newVal);
    }, { deep: true });
    
    // 3. 立即执行（immediate）
    this.$watch('message', (newVal) => {
      console.log('立即执行:', newVal);
    }, { immediate: true });
    
    // 4. 监听函数返回值
    this.$watch(
      () => this.user.name + this.user.age,
      (newVal) => {
        console.log('name 或 age 变化:', newVal);
      }
    );
    
    // 5. 停止监听
    const unwatch = this.$watch('message', (newVal) => {
      console.log(newVal);
    });
    
    // 停止监听
    unwatch();
  }
}
```

**在 Options API 中使用 watch 选项：**

```javascript
export default {
  watch: {
    // 简单监听
    message(newVal, oldVal) {
      console.log('变化:', oldVal, '->', newVal);
    },
    
    // 深度监听
    user: {
      handler(newVal) {
        console.log('user 变化:', newVal);
      },
      deep: true,
      immediate: true
    },
    
    // 监听路由
    '$route.query'(newVal) {
      console.log('路由参数变化:', newVal);
    }
  }
}
```

---

### 4. **`this.$nextTick(callback)`** - DOM 更新后执行

在下次 DOM 更新循环结束后执行回调。

```vue
<template>
  <div>
    <p ref="message">{{ message }}</p>
    <button @click="updateMessage">更新</button>
  </div>
</template>

<script>
export default {
  data() {
    return { message: 'Hello' };
  },
  methods: {
    async updateMessage() {
      this.message = 'World';
      
      // ❌ 错误：DOM 还没更新
      console.log(this.$refs.message.textContent);  // 'Hello'
      
      // ✅ 正确：使用 $nextTick
      this.$nextTick(() => {
        console.log(this.$refs.message.textContent);  // 'World'
      });
      
      // ✅ 使用 async/await
      await this.$nextTick();
      console.log(this.$refs.message.textContent);  // 'World'
    }
  }
}
</script>
```

**使用场景：**

```javascript
// 1. 数据更新后操作 DOM
this.list.push(newItem);
this.$nextTick(() => {
  this.$refs.list.scrollTop = this.$refs.list.scrollHeight;
});

// 2. 表单验证
this.form.name = 'new name';
this.$nextTick(() => {
  this.$refs.form.validate();
});

// 3. 获取更新后的元素尺寸
this.visible = true;
this.$nextTick(() => {
  const rect = this.$refs.dialog.getBoundingClientRect();
  console.log(rect.width, rect.height);
});
```

---

### 5. **`this.$forceUpdate()`** - 强制重新渲染

强制组件重新渲染（通常不需要使用）。

```javascript
export default {
  data() {
    return {
      obj: { name: '张三' }
    };
  },
  methods: {
    updateName() {
      // ⚠️ Vue 无法检测到的变化
      this.obj.age = 25;  // 添加新属性
      
      // 强制更新
      this.$forceUpdate();
    }
  }
}
```

**✅ 更好的做法：**

```javascript
// 使用 Vue.set 或 this.$set
this.$set(this.obj, 'age', 25);

// 或使用展开运算符
this.obj = { ...this.obj, age: 25 };
```

---

### 6. **`this.$destroy()`** - 销毁组件（Vue 2）

⚠️ **注意：** Vue 3 已移除，不推荐手动调用。

```javascript
// Vue 2
this.$destroy();  // 销毁当前组件实例
```

---

## 三、Vue Router 注入的 `$` 属性

### 1. **`this.$router`** - 路由器实例

```javascript
export default {
  methods: {
    // 1. 字符串路径
    this.$router.push('/home');
    
    // 2. 对象路径
    this.$router.push({ path: '/home' });
    
    // 3. 命名路由
    this.$router.push({ name: 'user', params: { id: 123 } });
    
    // 4. 带查询参数
    this.$router.push({ path: '/user', query: { id: 123 } });
    
    // 5. 替换当前页（不留历史）
    this.$router.replace('/login');
    
    // 6. 前进/后退
    this.$router.go(-1);   // 后退
    this.$router.go(1);    // 前进
    this.$router.back();   // 后退
    this.$router.forward(); // 前进
    
    // 7. 解析路由（生成 URL）
    const resolved = this.$router.resolve({
      path: '/user',
      query: { id: 123 }
    });
    console.log(resolved.href);  // '/user?id=123'
    window.open(resolved.href, '_blank');
  }
}
```

---

### 2. **`this.$route`** - 当前路由信息

```javascript
export default {
  mounted() {
    console.log(this.$route);
    // 输出:
    // {
    //   path: '/user/123',
    //   name: 'userDetail',
    //   params: { id: '123' },
    //   query: { tab: 'info' },
    //   hash: '#section1',
    //   fullPath: '/user/123?tab=info#section1',
    //   matched: [...],  // 路由记录
    //   meta: { requiresAuth: true }
    // }
    
    // 常用属性
    console.log(this.$route.path);      // '/user/123'
    console.log(this.$route.name);      // 'userDetail'
    console.log(this.$route.params);    // { id: '123' }
    console.log(this.$route.query);     // { tab: 'info' }
    console.log(this.$route.hash);      // '#section1'
    console.log(this.$route.fullPath);  // '/user/123?tab=info#section1'
    console.log(this.$route.meta);      // { requiresAuth: true }
  },
  watch: {
    // 监听路由变化
    $route(to, from) {
      console.log('从', from.path, '跳转到', to.path);
    }
  }
}
```

---

## 四、Vuex/Pinia 注入的 `$` 属性

### 1. **`this.$store`** - Vuex Store（Vue 2）

```javascript
export default {
  computed: {
    // 访问 state
    count() {
      return this.$store.state.count;
    },
    
    // 使用 getters
    doubleCount() {
      return this.$store.getters.doubleCount;
    }
  },
  methods: {
    // 提交 mutation
    increment() {
      this.$store.commit('increment');
      this.$store.commit('increment', 10);  // 传递参数
    },
    
    // 分发 action
    async fetchData() {
      await this.$store.dispatch('fetchUser', { id: 123 });
    }
  }
}
```

### 2. **Vue 3 + Pinia**

```javascript
import { useUserStore } from '@/store/user';

export default {
  setup() {
    const userStore = useUserStore();
    
    console.log(userStore.name);       // state
    userStore.setName('张三');          // action
    console.log(userStore.fullName);   // getter
    
    return { userStore };
  }
}
```

---

## 五、Element Plus 注入的 `$` 方法

### 1. **`this.$message`** - 消息提示

```javascript
// 基本用法
this.$message('普通消息');
this.$message.success('成功');
this.$message.error('错误');
this.$message.warning('警告');
this.$message.info('信息');

// 高级配置
this.$message({
  message: '这是一条消息',
  type: 'success',      // success | warning | info | error
  duration: 3000,       // 显示时间（毫秒），0 表示不自动关闭
  showClose: true,      // 显示关闭按钮
  center: true,         // 居中显示
  customClass: 'my-class',  // 自定义类名
  onClose: () => {
    console.log('关闭时回调');
  }
});
```

### 2. **`this.$alert`** - 警告框

```javascript
this.$alert('这是一段内容', '标题', {
  confirmButtonText: '确定',
  callback: (action) => {
    console.log('用户点击了确定');
  }
});

// Promise 方式
this.$alert('内容', '标题')
  .then(() => {
    console.log('用户点击了确定');
  })
  .catch(() => {
    console.log('用户关闭了弹窗');
  });
```

### 3. **`this.$confirm`** - 确认框

```javascript
this.$confirm('确定要删除吗？', '提示', {
  confirmButtonText: '确定',
  cancelButtonText: '取消',
  type: 'warning',
  center: true
}).then(() => {
  // 用户点击确定
  this.deleteItem();
}).catch(() => {
  // 用户点击取消
  console.log('已取消');
});
```

### 4. **`this.$prompt`** - 输入框

```javascript
this.$prompt('请输入您的姓名', '提示', {
  confirmButtonText: '确定',
  cancelButtonText: '取消',
  inputPattern: /^[\u4e00-\u9fa5]{2,4}$/,
  inputErrorMessage: '姓名格式不正确'
}).then(({ value }) => {
  console.log('用户输入:', value);
}).catch(() => {
  console.log('已取消');
});
```

### 5. **`this.$notify`** - 通知

```javascript
this.$notify({
  title: '通知',
  message: '这是一条通知消息',
  type: 'success',  // success | warning | info | error
  duration: 4500,
  position: 'top-right',  // top-right | top-left | bottom-right | bottom-left
  offset: 50,
  showClose: true,
  onClick: () => {
    console.log('点击了通知');
  }
});
```

### 6. **`this.$loading`** - 加载动画

```javascript
// 全屏加载
const loading = this.$loading({
  lock: true,
  text: '加载中...',
  spinner: 'el-icon-loading',
  background: 'rgba(0, 0, 0, 0.7)'
});

// 关闭加载
loading.close();

// 局部加载
const loading = this.$loading({ target: this.$refs.myElement });
loading.close();
```

### 7. **`this.$msgbox`** - 消息框（底层 API）

```javascript
this.$msgbox({
  title: '消息',
  message: '这是一条消息',
  showCancelButton: true,
  confirmButtonText: '确定',
  cancelButtonText: '取消',
  beforeClose: (action, instance, done) => {
    if (action === 'confirm') {
      // 异步操作
      setTimeout(() => {
        done();  // 关闭弹窗
      }, 1000);
    } else {
      done();
    }
  }
}).then(() => {
  console.log('确定');
}).catch(() => {
  console.log('取消');
});
```

---

## 六、自定义全局属性

### 在 `main.js` 中注入：

```javascript
// main.js
import { createApp } from 'vue';
import App from './App.vue';

const app = createApp(App);

// 注入全局属性
app.config.globalProperties.$http = axios;
app.config.globalProperties.$utils = {
  formatDate(date) {
    return new Date(date).toLocaleDateString();
  },
  formatAmount(amount) {
    return '¥' + Number(amount).toFixed(2);
  }
};

app.config.globalProperties.$CMIModal = modal;
app.config.globalProperties.getEnumText = getEnumText;
app.config.globalProperties.closeTipWindow = () => {
  window.opener = null;
  window.open('', '_self');
  window.close();
};

app.mount('#app');
```

### 在组件中使用：

```javascript
export default {
  methods: {
    fetchData() {
      // 使用全局 $http
      this.$http.get('/api/data').then(res => {
        console.log(res.data);
      });
    },
    
    formatDate() {
      // 使用全局工具方法
      const date = this.$utils.formatDate(new Date());
      console.log(date);
    },
    
    closeModal() {
      // 使用自定义全局方法
      this.closeTipWindow();
    }
  }
}
```

---

## 七、完整对比表

| 属性/方法               | 来源         | 用途         | 示例                       |
| ----------------------- | ------------ | ------------ | -------------------------- |
| **`this.$data`**        | Vue 内置     | 组件数据对象 | `this.$data.message`       |
| **`this.$props`**       | Vue 内置     | 组件 props   | `this.$props.title`        |
| **`this.$el`**          | Vue 内置     | 根 DOM 元素  | `this.$el.querySelector()` |
| **`this.$options`**     | Vue 内置     | 组件选项     | `this.$options.name`       |
| **`this.$refs`**        | Vue 内置     | 模板引用     | `this.$refs.myInput`       |
| **`this.$attrs`**       | Vue 内置     | 未注册属性   | `this.$attrs.id`           |
| **`this.$slots`**       | Vue 内置     | 插槽内容     | `this.$slots.default`      |
| **`this.$emit`**        | Vue 内置     | 触发事件     | `this.$emit('update')`     |
| **`this.$watch`**       | Vue 内置     | 监听数据     | `this.$watch('msg', cb)`   |
| **`this.$nextTick`**    | Vue 内置     | DOM 更新后   | `this.$nextTick(cb)`       |
| **`this.$forceUpdate`** | Vue 内置     | 强制渲染     | `this.$forceUpdate()`      |
| **`this.$router`**      | Vue Router   | 路由器       | `this.$router.push()`      |
| **`this.$route`**       | Vue Router   | 当前路由     | `this.$route.path`         |
| **`this.$store`**       | Vuex         | 状态管理     | `this.$store.state`        |
| **`this.$message`**     | Element Plus | 消息提示     | `this.$message.success()`  |
| **`this.$alert`**       | Element Plus | 警告框       | `this.$alert()`            |
| **`this.$confirm`**     | Element Plus | 确认框       | `this.$confirm()`          |
| **`this.$notify`**      | Element Plus | 通知         | `this.$notify()`           |
| **`this.$loading`**     | Element Plus | 加载动画     | `this.$loading()`          |
| **`this.$http`**        | 自定义       | HTTP 请求    | `this.$http.get()`         |

---

## 八、最佳实践

### ✅ 推荐做法：

```javascript
// 1. 使用 $nextTick 操作 DOM
this.message = 'new';
this.$nextTick(() => {
  this.$refs.input.focus();
});

// 2. 使用 $refs 调用子组件方法
this.$refs.childComponent.refresh();

// 3. 使用 $emit 与父组件通信
this.$emit('update:modelValue', newValue);

// 4. 使用 $route 获取路由参数
const id = this.$route.params.id;

// 5. 使用 $router 编程式导航
this.$router.push({ name: 'user', params: { id: 123 } });

// 6. 使用 $message 提示用户
this.$message.success('操作成功');
```

### ❌ 避免做法：

```javascript
// ❌ 在 created 中访问 $refs（DOM 未渲染）
created() {
  this.$refs.input.focus();  // undefined
}

// ❌ 过度使用 $forceUpdate
this.$forceUpdate();  // 应该修复响应式问题

// ❌ 直接修改 $data
this.$data.message = 'new';  // 应该用 this.message = 'new'

// ❌ 忘记清理事件监听
mounted() {
  window.addEventListener('resize', this.handleResize);
}
// 应该在 beforeUnmount 中移除
```

---

## 总结

**`this.$` 是 Vue 实例的全局 API 入口**，包含：

- 🔧 **Vue 内置**: `$data`, `$refs`, `$emit`, `$router`, `$route` 等
- 🎨 **UI 框架**: `$message`, `$confirm`, `$notify` (Element Plus)
- 📦 **状态管理**: `$store` (Vuex)
- 🔌 **自定义**: 通过 `app.config.globalProperties` 注入

## Vue2/3 this.$

简单直接地回答：**`this.$` 主要用于 Vue 2，在 Vue 3 中虽然保留了支持，但官方推荐的做法是不再使用它。**

准确地说，`this.$` 的命运与 Vue 从 **Options API（选项式 API）** 向 **Composition API（组合式 API）** 的过渡密切相关。

以下是详细的解析：

---

### 一、 Vue 2 时代：`this.$` 的绝对主场

在 Vue 2 中，所有组件逻辑都写在 `export default { ... }` 中，`this` 指向当前组件的实例。Vue 官方和生态插件（如 Vue Router、Vuex）约定俗成地将全局属性和内置方法挂载在 `this.$` 下，主要是为了**与开发者自定义的 `data`、`methods` 区分开来**。

在 Vue 2 中，你每天都会大量写这样的代码：

```javascript
// Vue 2 中的常见 this.$ 用法
export default {
  methods: {
    doSomething() {
      // 1. 路由跳转
      this.$router.push('/home');
      // 2. 获取路由参数
      const id = this.$route.params.id;
      // 3. 访问 Vuex 状态管理
      this.$store.dispatch('user/login');
      // 4. 触发自定义事件
      this.$emit('update', newValue);
      // 5. 访问挂载在 Vue 原型上的全局变量 (如 Main.js 中设置的 Vue.prototype.$http = axios)
      this.$http.get('/api/data');
      // 6. 强制更新 / 下一次 DOM 更新后执行
      this.$forceUpdate();
      this.$nextTick(() => { /* ... */ });
    }
  }
}
```

---

### 二、 Vue 3 时代：`this.$` 逐渐被废弃

Vue 3 引入了 Composition API 和 `<script setup>` 语法糖。在新的架构下，**组件实例不再像以前那样作为单一的对象上下文存在，`this` 的概念被极大地弱化了**。

#### 1. 在 `<script setup>` 中（现代 Vue 3 写法）：**完全不用 `this.$`**

在 `<script setup>` 中，`this` 是 `undefined`。你不再需要通过 `this.$` 去获取路由或状态，而是通过**组合式函数**按需引入。

```vue
<!-- Vue 3 现代写法：告别 this.$ -->
<script setup>
import { useRouter, useRoute } from 'vue-router';
import { useCounterStore } from '@/stores/counter';
import { nextTick, getCurrentInstance } from 'vue';

const router = useRouter();  // 替代 this.$router
const route = useRoute();    // 替代 this.$route
const counterStore = useCounterStore(); // 替代 this.$store

// 触发事件 (defineEmits 是编译宏，不需要引入)
const emit = defineEmits(['update']);
emit('update', 'newVal');    // 替代 this.$emit

// NextTick
nextTick(() => { /* ... */ }); // 替代 this.$nextTick
</script>
```

#### 2. 在 Vue 3 的 Options API 中（向后兼容写法）：**依然可以用 `this.$`**

如果你在 Vue 3 项目中依然坚持使用 Vue 2 的 `export default { data(), methods: {} }` 写法，`this.$` 仍然是生效的，这主要是为了保证 Vue 2 项目能平滑升级到 Vue 3。

```javascript
// Vue 3 中使用 Options API (不推荐用于新组件，但兼容旧代码)
import { defineComponent } from 'vue';

export default defineComponent({
  methods: {
    doSomething() {
      // 在 Vue3 的 Options API 中依然可以使用
      this.$router.push('/home'); 
    }
  }
})
```

#### 3. 全局属性的特殊情况

在 Vue 2 中，我们习惯把 Axios 等工具挂载到 `Vue.prototype.$http` 上，然后在组件里用 `this.$http` 调用。
在 Vue 3 中，如果你非要这么做，需要使用 `app.config.globalProperties.$http`。在组件的 Options API 中依然可以用 `this.$http`，但**官方极其不推荐**，而是建议使用 `provide/inject` 或直接在组件中 `import axios`。

### 总结对比表

| 功能需求          | Vue 2 写法 (`this.$`)   | Vue 3 推荐写法 (`<script setup>`)            |
| :---------------- | :---------------------- | :------------------------------------------- |
| **路由跳转**      | `this.$router.push()`   | `const router = useRouter(); router.push()`  |
| **获取路由参数**  | `this.$route.params.id` | `const route = useRoute(); route.params.id`  |
| **状态管理**      | `this.$store.state.xxx` | `const store = useXxxStore(); store.xxx`     |
| **派发事件**      | `this.$emit('xxx')`     | `const emit = defineEmits(); emit('xxx')`    |
| **DOM 更新回调**  | `this.$nextTick()`      | `import { nextTick } from 'vue'; nextTick()` |
| **读取 DOM 元素** | `this.$refs.inputRef`   | `const inputRef = ref(null); inputRef.value` |

**结论：** 
如果你在维护 **Vue 2** 项目，`this.$` 是你的日常工具。
如果你在开发新的 **Vue 3** 项目，请彻底忘记 `this.$`，拥抱 `useXxx` 的组合式函数写法，这样代码会更清晰、更易于类型推断和 Tree-shaking。