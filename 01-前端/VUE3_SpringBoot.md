# 问题点

### SpringBoot+Vue3 跨域：前端proxy vs 后端配置

直接给你**开发/生产双场景结论**，这是现在企业主流做法：

#### 一、核心结论（最标准方案）

1. **开发环境（本地跑）**：**必须用 Vue3 的 proxy 代理解决** ✅
2. **生产环境（部署上线）**：**前端不再用 proxy，统一在后端 SpringBoot 解决跨域** ✅

---

#### 二、为什么要这么做？

##### 1. 开发环境：Vue3 proxy 是最优解

- Vue 的 `vite.config.js` / `vue.config.js` 配置 **devServer 代理**
- 原理：**前端发起请求 → 本地Node服务器转发 → 后端接口**
- 浏览器看不到跨域，完全规避同源策略
- 配置简单、无侵入、不影响后端代码

##### 2. 生产环境：绝对不能用前端 proxy

- 打包后（`npm run build`）**proxy 会失效**
- 生产是纯静态文件（Nginx/CDN托管），没有Node服务器
- 必须让**后端开放跨域权限**，前端直接请求后端

---

#### 三、具体配置代码（直接复制用）

##### ① Vue3 开发环境 proxy 配置（Vite）

`vite.config.js`
```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  server: {
    // 本地代理，只在开发环境生效
    proxy: {
      '/api': {
        target: 'http://localhost:8080', // 你的SpringBoot地址
        changeOrigin: true, // 开启跨域
        rewrite: (path) => path.replace(/^\/api/, '') // 路径重写
      }
    }
  }
})
```

##### ② SpringBoot 后端跨域配置（生产/通用）

新建一个配置类，**全局生效**，一劳永逸：
```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**") // 所有接口
                .allowedOriginPatterns("*") // 允许所有域名
                .allowedMethods("GET", "POST", "PUT", "DELETE") // 允许请求方式
                .allowCredentials(true) // 允许携带cookie
                .maxAge(3600); // 预检有效期
    }
}
```

---

#### 四、补充：企业级备选方案

如果项目部署用 **Nginx**，也可以在 **Nginx 配置跨域/反向代理**，这也是生产常用方案，后端就不用写跨域配置了。

---

#### 总结

1. **开发**：Vue3 proxy（首选，简单无侵入）
2. **生产**：SpringBoot 全局跨域配置（标准方案）
3. 千万不要：开发环境用后端跨域（不规范）、生产用前端 proxy（无效）

我可以给你一套**开箱即用的完整请求封装**（axios+proxy+后端跨域），直接复制到项目就能跑，需要吗？

### 开发环境不用 proxy，只靠后端跨域配置，也能正常跑项目

这是**完全合法、能跑通**的方案，只是**不推荐、不主流**而已。

我给你把**原理、能不能用、优缺点**讲得明明白白：

---

#### 一、开发环境：后端配置跨域 + 前端不写 proxy → 能跑通 ✅

只要你后端加了 `CorsConfig` 跨域配置：
- 前端**直接写后端真实地址**（比如 `http://localhost:8080`）
- 前端**不配置 proxy**
- **浏览器不会报跨域**，请求能正常发送、正常接收

这是**技术上完全可行**的。

---

#### 二、那为什么大家都推荐「开发用 proxy」？

虽然**能跑**，但**后端解决跨域有 3 个大坑**：

##### 1. 开发/生产环境代码不一致，容易上线出问题

- 开发：前端请求 `http://localhost:8080/api/xxx`
- 生产：前端请求 `/api/xxx`
你必须**手动改地址**，一不小心就上线报错。

##### 2. 携带 Cookie/Token 时容易出问题

后端跨域在**本地开发**时，经常因为**端口不同、域名不同**导致：
- 跨域配置不生效
- 登录态丢失
- 预检请求（OPTIONS）被拦截
proxy 能完美避开这些问题。

##### 3. 多人协作时不方便

后端一旦改端口、改地址，**所有前端人员都要跟着改代码**，非常麻烦。

---

#### 三、两种方案对比（最直观）

| 方案           | 开发环境能否用 |                       优点 | 缺点                      |     推荐度 |
| -------------- | -------------- | -------------------------: | ------------------------- | ---------: |
| **Vue3 proxy** | ✅ 能           | 无跨域、统一路径、开发舒适 | 需简单配置                | ⭐⭐⭐⭐⭐ 首选 |
| **后端跨域**   | ✅ 能           |               不用配置前端 | 路径混乱、易出bug、不规范 |   ⭐ 不推荐 |

---

#### 四、如果你非要在开发环境「不用 proxy、只用后端跨域」

##### 前端代码这样写（直接请求后端地址）

```js
// axios 基础路径 直接写后端完整地址
axios.defaults.baseURL = 'http://localhost:8080';
```

##### 后端保持跨域配置

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOriginPatterns("*")
                .allowedMethods("*")
                .allowCredentials(true);
    }
}
```
✅ **这样就能跑，没有任何问题**。

---

#### 最终一句话总结

**开发环境：不用 proxy，只靠后端跨域 → 能跑、能用、合法，但不推荐，容易埋坑。**
**企业标准：开发用 proxy，生产用后端跨域 → 最稳、最舒服。**

你现在是遇到了 **proxy 配置报错**，还是单纯想确认能不能这么用？我可以直接帮你搞定。

# 在 **Spring Boot + Vue3 前后端分离项目面试**

中，面试官问到 **Vue3 与 Spring Boot 对接**时，问题一般集中在 **通信机制、接口设计、跨域、安全、状态管理、部署** 六大类。

我给你整理一份 **“面试高频问题全景图（偏实战）”**，基本覆盖 80% 面试。

------

## 一、前后端通信基础（必问）

### 1 前端如何调用后端接口？

典型问题：

- Vue3 如何调用 SpringBoot 接口？
- 你们项目用什么请求库？

回答核心：

- 使用 **Axios**
- 封装 request 工具
- baseURL 指向 SpringBoot 服务

示例：

```js
import axios from "axios"

const request = axios.create({
  baseURL: "http://localhost:8080/api",
  timeout: 5000
})

export default request
```

调用：

```js
request.get("/user/list")
```

面试可能追问：

- 为什么封装 axios？
- axios 拦截器作用？

------

### **问题：Vue3 如何调用 SpringBoot 接口？**

**回答要点：**

1. **使用 HTTP 客户端库**

   - 最常用的是 **Axios**，也可以使用原生 Fetch。
   - Axios 支持请求拦截、响应拦截、请求超时等功能，适合企业项目。

2. **封装请求工具**

   - 封装统一的请求工具可以：
     - 设置 **baseURL**，统一指向后端 SpringBoot 服务；
     - 配置 **超时时间**、**请求头**；
     - 添加 **请求和响应拦截器**；
     - 统一处理错误信息或接口返回格式。

3. **基础示例**

   ```js
   import axios from "axios"
   
   const request = axios.create({
     baseURL: "http://localhost:8080/api", // 后端接口基础路径
     timeout: 5000                        // 请求超时
   })
   
   export default request
   ```

4. **调用接口**

   ```js
   // 获取用户列表
   request.get("/user/list").then(res => {
       console.log(res)
   })
   ```

------

### **面试可能追问 & 回答示例**

1. **为什么封装 axios？**
   - 统一管理接口请求，避免重复代码。
   - 可以集中处理请求头（如 token）、错误信息、统一返回格式。
   - 后期修改 baseURL 或增加全局拦截器只需要改封装文件。
2. **axios 拦截器作用？**
   - **请求拦截器**：
     - 可以在请求发送前加 token、设置 header、显示 loading。
   - **响应拦截器**：
     - 可以统一处理响应数据或错误，例如 token 过期跳转登录、接口异常提示等。

**示例：**

```js
// 请求拦截器
request.interceptors.request.use(config => {
  const token = localStorage.getItem('token')
  if(token){
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// 响应拦截器
request.interceptors.response.use(
  res => res.data, // 直接返回 data
  err => {
    if(err.response.status === 401){
      // token 过期，跳转登录
      router.push("/login")
    }
    return Promise.reject(err)
  }
)
```

------

💡 **面试加分点**：

- 可以提到 **前后端分离** 的架构思路：Vue3 完全作为前端 SPA，SpringBoot 提供 RESTful API。
- 可以结合 **统一响应结构** 讲解拦截器的好处，例如所有接口返回 `{code, message, data}` 格式，在拦截器统一处理。

------

如果你需要，我可以帮你把 **“前端调用后端接口 + axios 封装 + 拦截器 + token +跨域”** 整合成一份 **完整面试标准回答模板**，保证面试讲起来一口气就说清楚。

好的，我帮你把 **Vue3 前端调用 Spring Boot 接口的核心流程**整合成一份 **完整面试标准回答模板**，覆盖 **axios封装、请求/响应拦截器、token认证、跨域处理、统一返回**，一口气就能讲清楚面试官想考的点。

------

### **Vue3 + Spring Boot 前后端对接 — 面试标准回答模板**

------

#### **1. 前端如何调用后端接口**

- Vue3 前端通过 **HTTP 客户端库**（通常是 **Axios**）调用 Spring Boot 提供的 RESTful API。
- 请求方式包括：`GET`、`POST`、`PUT`、`DELETE`。
- 使用 **封装的请求工具**，可以统一管理 baseURL、请求头、拦截器和错误处理。

```js
// src/utils/request.js
import axios from "axios";

// 创建 axios 实例
const request = axios.create({
  baseURL: "http://localhost:8080/api", // Spring Boot 服务地址
  timeout: 5000
});

export default request;
```

**调用接口示例：**

```js
import request from "@/utils/request";

// 获取用户列表
request.get("/user/list").then(res => {
  console.log(res);
});

// 登录
request.post("/user/login", { username: "admin", password: "123" })
  .then(res => console.log(res));
```

------

#### **2. 为什么要封装 Axios**

- **统一管理**：baseURL、timeout、请求头统一配置。
- **复用性高**：全局修改只需在封装文件改。
- **方便扩展**：可以集中添加拦截器、处理 token、处理错误。
- **便于维护**：接口格式统一，项目大时更易管理。

------

#### **3. Axios 拦截器作用**

##### **请求拦截器**

- 在请求发送前处理：
  - 添加 token
  - 设置统一请求头
  - 显示 loading 等

##### **响应拦截器**

- 接收响应后处理：
  - 统一返回 data
  - token 过期处理（401 跳转登录）
  - 错误提示

```js
// src/utils/request.js
request.interceptors.request.use(config => {
  const token = localStorage.getItem("token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

request.interceptors.response.use(
  res => res.data, // 直接返回 data
  err => {
    if (err.response.status === 401) {
      // token 过期，跳转登录
      router.push("/login");
    } else {
      alert(err.response.data.message || "接口异常");
    }
    return Promise.reject(err);
  }
);
```

------

#### **4. Token 登录流程**

1. 用户输入用户名/密码，调用登录接口。
2. Spring Boot 验证成功后返回 **token**（通常是 JWT）。
3. 前端存储 token（localStorage/sessionStorage）。
4. 后续接口请求在 **请求头** 中携带 token（`Authorization: Bearer token`）。
5. 后端验证 token，确保用户身份。

------

#### **5. 跨域问题及解决方案**

- **为什么跨域**：

  - 前端端口（5173）与后端端口（8080）不同，浏览器同源策略阻止请求。

- **解决方案**：

  1. **后端 CORS 配置**

     ```java
     @CrossOrigin(origins = "*")
     @RestController
     public class UserController {}
     ```

  2. **全局配置**

     ```java
     @Configuration
     public class CorsConfig implements WebMvcConfigurer {
         @Override
         public void addCorsMappings(CorsRegistry registry){
             registry.addMapping("/**")
                     .allowedOrigins("*")
                     .allowedMethods("*");
         }
     }
     ```

  3. **开发环境代理（Vite）**

     ```js
     // vite.config.js
     export default {
       server: {
         proxy: {
           '/api': 'http://localhost:8080'
         }
       }
     };
     ```

------

#### **6. 统一接口返回格式**

- 后端统一返回格式：

```json
{
  "code": 200,
  "message": "success",
  "data": {}
}
```

- 前端拦截器统一处理：

```js
request.interceptors.response.use(res => {
  const { code, message, data } = res;
  if(code === 200){
    return data;
  } else {
    alert(message);
    return Promise.reject(message);
  }
});
```

- 便于项目维护、错误统一处理、提升代码可读性。

------

#### **7. 面试加分点**

- 可以提到 **前后端分离架构**：Vue3 SPA 前端 + Spring Boot 提供 REST API。
- 可以强调安全性：
  - token 过期处理
  - refresh token
  - 跨域防护
- 可以提到 **可扩展性和可维护性**：
  - Axios 封装 + 拦截器
  - 统一接口格式

------

这份模板可以让你在面试中 **一口气完整回答前端如何调用后端接口**，覆盖 **封装、拦截器、token、跨域、统一返回**，基本可以应对 90% 的面试问题。

------

如果你愿意，我可以帮你画一张 **“Vue3 前端调用 SpringBoot 接口流程图”**，把 **请求 → 拦截器 → token → 跨域 → 响应返回** 全部流程可视化，面试讲起来更直观。

你希望我帮你画吗？

------

## 二、接口设计问题（高频）

### 2 Vue3 如何传递参数给 SpringBoot？

常见方式：

| 方式     | Vue3      | SpringBoot      |
| -------- | --------- | --------------- |
| URL参数  | params    | @RequestParam   |
| JSON数据 | data      | @RequestBody    |
| Path参数 | /user/1   | @PathVariable   |
| 表单     | form-data | @ModelAttribute |

示例

Vue3：

```js
request.post("/user/login", {
  username: "admin",
  password: "123"
})
```

SpringBoot：

```java
@PostMapping("/login")
public Result login(@RequestBody LoginDTO dto){
}
```

面试可能问：

- `@RequestBody` 和 `@RequestParam` 区别
- GET 能不能用 RequestBody？

你整理的这部分是 **前端与 Spring Boot 传参方式的核心内容**，在面试中非常高频，我帮你梳理成 **面试标准回答模板**，可以直接用来讲解，同时包含 **常见追问及回答要点**。



------

### **1. 常见参数传递方式**

| 方式          | Vue3 发送方式 | Spring Boot 接收方式                 | 适用场景                   |
| ------------- | ------------- | ------------------------------------ | -------------------------- |
| **URL 参数**  | `params`      | `@RequestParam`                      | GET 请求查询参数、简单过滤 |
| **JSON 数据** | `data`        | `@RequestBody`                       | POST/PUT 传递复杂对象      |
| **Path 参数** | `/user/1`     | `@PathVariable`                      | RESTful 风格 URL 中的标识  |
| **表单**      | `form-data`   | `@ModelAttribute` 或 `@RequestParam` | 文件上传或传统表单提交     |

------

### **2. 示例：登录接口**

##### **Vue3 端**

```js
request.post("/user/login", {
  username: "admin",
  password: "123"
});
```

##### **Spring Boot 端**

```java
@PostMapping("/login")
public Result login(@RequestBody LoginDTO dto) {
    // 验证用户名密码
    return Result.success();
}
```

------

### **3. 面试可能追问**

#### **(1) `@RequestBody` 和 `@RequestParam` 区别**

| 项目            | @RequestParam            | @RequestBody                          |
| --------------- | ------------------------ | ------------------------------------- |
| 数据来源        | URL 参数或表单数据       | HTTP 请求体 JSON 数据                 |
| 支持方法        | GET/POST/PUT/DELETE 通用 | POST/PUT（理论上 GET 也可，但不推荐） |
| 适用场景        | 简单参数、单值           | 对象、复杂数据结构                    |
| Spring 自动绑定 | 自动转换基本类型         | 自动序列化 JSON 到对象                |

**举例**：

```java
@GetMapping("/user")
public String getUser(@RequestParam String username) { ... }

@PostMapping("/user")
public String createUser(@RequestBody UserDTO userDTO) { ... }
```

------

#### **(2) GET 能不能用 `@RequestBody`？**

- **技术上可以**：Spring Boot 从 2.x 版本开始允许 GET 请求带 body。
- **不推荐**：
  - HTTP 标准中 GET 请求的 body 不具备语义，容易导致兼容性问题。
  - GET 请求最好通过 **URL 参数 (`@RequestParam`) 或 PathVariable** 传递。

**推荐实践**：

- GET → `@RequestParam` / `@PathVariable`
- POST → `@RequestBody`

------

### **4. 面试加分点**

1. **RESTful 风格**：
   - GET `/users` → 查询列表
   - GET `/users/{id}` → 查询单个用户
   - POST `/users` → 创建用户
   - PUT `/users/{id}` → 更新用户
   - DELETE `/users/{id}` → 删除用户
2. **复杂对象参数**：
   - POST 请求传递复杂对象时，必须使用 `@RequestBody`。
   - 前端使用 JSON 对象传递，后端用 DTO 接收。
3. **前端封装技巧**：
   - Axios 可以封装 `get`、`post` 等方法，自动处理参数序列化。
   - 对 GET 请求参数，可以自动拼接到 URL 上。

------

💡 **面试示例回答**：

> “前端 Vue3 可以通过 URL 参数、JSON 请求体、Path 参数或表单数据传递给 Spring Boot。
> URL 参数用 `@RequestParam` 接收，Path 参数用 `@PathVariable`，JSON 数据用 `@RequestBody`。
> GET 请求通常不推荐用 `@RequestBody`，POST/PUT 传对象用 `@RequestBody`。封装 Axios 可以统一处理这些请求方式。”

------

如果你愿意，我可以帮你把 **整个 Vue3 ↔ Spring Boot 参数传递 + axios 封装 + token + 跨域 +统一返回** 做成 **一张完整流程图**，面试讲起来更直观，几句话就能把整个流程说清楚。

你希望我帮你画吗？



------

## 三、跨域问题（100%会问）

### 3 Vue 调用 SpringBoot 为什么会跨域？

因为：

- 前端端口：5173
- 后端端口：8080
- **协议 / 域名 / 端口 不同**

解决方式：

SpringBoot：

```java
@CrossOrigin
@RestController
public class UserController{
}
```

或全局配置：

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    public void addCorsMappings(CorsRegistry registry){
        registry.addMapping("/**")
        .allowedOrigins("*")
        .allowedMethods("*");
    }
}
```

面试可能继续问：

- 跨域本质是什么？
- 浏览器为什么限制？
- 什么是 **预检请求 OPTIONS**？



你整理的这一部分是 **前后端分离项目面试必问的跨域问题**。我帮你把它梳理成一份 **面试标准回答模板**，覆盖 **本质、原因、解决方案、预检请求**，方便直接在面试中讲。

------

### **1. 为什么会跨域？**

- **同源策略（Same-Origin Policy）** 是浏览器的一种安全机制，限制不同源的网页互相访问资源。
- **不同源定义**：协议、域名或端口任意一个不同就算不同源。
- **示例**：
  - 前端：`http://localhost:5173`
  - 后端：`http://localhost:8080`
  - 端口不同 → 浏览器认为是跨域请求 → 请求被拦截

------

### **2. 解决方案**

#### **(1) 后端 CORS 配置**

- **方法 1：在 Controller 上加 `@CrossOrigin` 注解**

```java
@CrossOrigin
@RestController
public class UserController {
}
```

- **方法 2：全局配置**

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")          // 所有接口
                .allowedOrigins("*")        // 允许所有源
                .allowedMethods("*")        // 允许 GET/POST/PUT/DELETE 等
                .allowCredentials(true)     // 允许携带 Cookie
                .maxAge(3600);              // 预检请求缓存时间
    }
}
```

------

#### **(2) 前端开发环境代理（Vite）**

- 避免开发环境跨域请求，可以通过 **代理** 将 `/api` 转发到 SpringBoot。

```js
// vite.config.js
export default {
  server: {
    proxy: {
      '/api': 'http://localhost:8080'
    }
  }
}
```

- 调用接口时：

```js
request.get("/api/user/list")
```

------

### **3. 面试可能追问**

#### **(1) 跨域本质是什么？**

- 跨域本质是 **浏览器的安全策略限制**，防止恶意网站获取敏感数据。
- 跨域请求如果没有服务器允许，浏览器会阻止请求返回数据，但**请求仍可能到达服务器**。

------

#### **(2) 浏览器为什么限制？**

- 防止 **CSRF 攻击** 和 **数据泄露**。
- 保护用户隐私和账户安全。
- 仅浏览器端生效，**后端本身不受限制**。

------

#### **(3) 什么是预检请求 OPTIONS？**

- 对于 **非简单请求**（例如：使用 `PUT/DELETE`、请求头自定义、Content-Type 不是 `application/x-www-form-urlencoded` 等），浏览器会先发送 **OPTIONS 请求**。
- **作用**：
  - 检查服务器是否允许实际请求。
  - 确认方法和头信息是否安全。
- 服务器返回允许信息后，浏览器才会发起实际请求。

------

### **4. 面试加分点**

- 可以提到 **前后端分离架构下，跨域处理的三种方式**：
  1. 后端 CORS 配置
  2. 前端代理
  3. Nginx 反向代理（生产环境常用）
- 可以结合 token / Cookie 讲 **带凭证跨域**：
  - `allowCredentials(true)` + 前端 `withCredentials: true`
- 预检请求可以通过减少自定义头和简单请求优化，降低 OPTIONS 请求次数。

------

💡 **面试示例回答**：

> “跨域是浏览器同源策略的限制，当前端端口和后端端口不同，浏览器会阻止前端读取后端响应。
> 解决方法有两种：一是在 SpringBoot 后端开启 CORS（`@CrossOrigin` 或全局配置），二是在开发环境用 Vite 代理。
> 对于非简单请求，浏览器会先发 OPTIONS 预检请求，确认服务器允许后才会发送实际请求。”

------

如果你愿意，我可以帮你把 **前端调用接口 → 跨域 → token → axios 拦截器 → 响应返回** 做成 **一张完整流程图**，让整个 Vue3 ↔ Spring Boot 对接流程可视化，面试讲起来更直观。

你希望我帮你画吗？

------

## 四、登录认证（重点）

### 4 Vue3 如何保存登录状态？

常见：

| 技术         | 作用     |
| ------------ | -------- |
| token        | 登录凭证 |
| localStorage | 持久化   |
| pinia        | 全局状态 |

登录流程：

```
Vue登录
   │
   │POST /login
   ▼
SpringBoot
   │
生成token
   ▼
返回 token
   ▼
Vue 存 localStorage
```

请求时：

```
Authorization: Bearer token
```

axios拦截器：

```js
request.interceptors.request.use(config=>{
   const token = localStorage.getItem("token")
   if(token){
      config.headers.Authorization = token
   }
   return config
})
```

SpringBoot：

常见认证：

- **Spring Security**
- **Sa-Token**
- **Apache Shiro**





------

### **1. 登录流程（前后端分离）**

1. **前端 Vue3 登录**：用户输入用户名/密码 → 调用 `/login` 接口。
2. **后端 SpringBoot 验证**：
   - 验证账号密码。
   - 成功生成 **token**（如 JWT 或自定义 token）。
3. **返回 token 给前端**。
4. **前端保存 token**：
   - 本地存储 `localStorage` 或 `sessionStorage`。
   - 也可以使用全局状态管理（如 Pinia、Vuex）保存 token。
5. **后续请求携带 token**：
   - HTTP 请求头：`Authorization: Bearer <token>`。
   - 后端验证 token，确认用户身份。

------

#### **流程示意**

```
Vue 前端登录
   │
   │ POST /login
   ▼
SpringBoot 后端
   │
生成 token
   ▼
返回 token
   ▼
Vue 保存 token（localStorage / pinia）
```

------

### **2. 前端 axios 拦截器**

- **请求拦截器**：在每次请求发送前自动携带 token。
- **响应拦截器**：统一处理 token 过期、跳转登录等。

```js
import request from "@/utils/request";

request.interceptors.request.use(config => {
  const token = localStorage.getItem("token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

request.interceptors.response.use(
  res => res.data,
  err => {
    if (err.response.status === 401) {
      // token 过期或未登录，跳转登录
      router.push("/login");
    }
    return Promise.reject(err);
  }
);
```

------

### **3. token 保存方式**

| 技术                              | 作用                                       |
| --------------------------------- | ------------------------------------------ |
| **token**                         | 登录凭证，后续请求携带，后端验证           |
| **localStorage / sessionStorage** | 持久化存储 token，页面刷新仍可保持登录状态 |
| **Pinia / Vuex**                  | 全局状态管理，方便组件间共享 token         |

> 💡 注意：token 存储在 localStorage 存在 XSS 风险，可结合 HttpOnly Cookie 提升安全性。

------

### **4. 后端常用认证方式**

1. **Spring Security**
   - 企业级标准认证框架
   - 支持 JWT、session、权限控制
2. **Sa-Token**
   - 简单易用，轻量级 token 框架
   - 支持多端登录、权限、过期处理
3. **Apache Shiro**
   - 灵活的权限与认证框架
   - 支持 session、权限注解

------

### **5. 面试可能追问**

1. **token 为什么不用 session？**
   - 无状态、分布式支持好、跨域友好。
2. **token 过期如何处理？**
   - 前端 401 处理 + refresh token 机制。
3. **前端刷新页面是否丢失登录状态？**
   - 通过 localStorage / sessionStorage + pinia 可保持状态。
4. **如何防止 XSS 攻击盗用 token？**
   - 尽量使用 HttpOnly Cookie 存储敏感 token。
   - 对前端输入数据进行严格校验。

------

💡 **面试示例回答**：

> “前端 Vue3 用户登录后调用 Spring Boot /login 接口，后端生成 token 返回前端。
> 前端将 token 存在 localStorage，并通过 axios 请求拦截器在每次请求头携带 `Authorization: Bearer token`。
> 后端可用 Spring Security、Sa-Token 或 Shiro 验证 token，实现用户身份认证和权限控制。”

------

如果你愿意，我可以帮你把 **“跨域 + token 登录 + axios 拦截器 + 参数传递 +统一返回”** 做成一张 **完整 Vue3 ↔ SpringBoot 接口调用流程图**，面试讲起来就一目了然。

你希望我画吗？

------

## 五、统一响应结构（常问）

### 5 为什么要统一返回结构？

SpringBoot：

```json
{
  "code":200,
  "message":"success",
  "data":{}
}
```

Vue3：

```js
request.interceptors.response.use(res=>{
   if(res.data.code !== 200){
       alert(res.data.message)
   }
   return res.data
})
```

面试官想考察：

- **前后端规范设计能力**

你整理的这一部分就是 **前后端接口规范化设计的核心点**，我帮你把它整理成一份 **面试标准回答模板**，讲起来既完整又有逻辑性。

------

### **1. 为什么要统一返回结构？**

- **可维护性**：项目大时，统一格式便于前端统一处理响应和错误。
- **可读性**：所有接口都有一致的字段，便于理解和调试。
- **错误处理统一化**：前端可以统一处理状态码、提示信息，而不需要每个接口单独判断。
- **前后端规范化**：体现团队开发规范和工程化水平。

------

### **2. 常见统一返回格式**

**SpringBoot 后端示例**：

```json
{
  "code": 200,          // 状态码，200 成功，其他代表错误
  "message": "success",  // 提示信息
  "data": {}            // 数据
}
```

> 💡 `code` 用于判断成功或失败，`message` 用于提示，`data` 承载实际业务数据。

------

### **3. 前端处理方式**

**Vue3 + axios 拦截器统一处理**：

```js
request.interceptors.response.use(res => {
  const { code, message, data } = res.data;

  if(code !== 200){
    alert(message);        // 全局统一提示
    return Promise.reject(message);
  }

  return data;             // 直接返回业务数据
});
```

**效果**：

- 前端业务逻辑只需关心 `data`，不用重复判断 `code`。
- 错误信息集中处理，减少冗余代码。

------

### **4. 面试官可能追问**

1. **为什么不直接返回数据？**
   - 接口返回状态不统一，前端处理逻辑复杂。
   - 不利于统一错误提示和日志记录。
2. **如何设计 code 值？**
   - 200 → 成功
   - 4xx → 客户端错误（参数错误、未登录等）
   - 5xx → 服务端错误
   - 具体可根据团队规范自定义业务码
3. **message 作用？**
   - 提供可读性提示，方便前端展示给用户或调试。
   - 与 code 搭配，可快速判断接口状态。
4. **data 可以为空吗？**
   - 可以，统一结构的关键是格式一致，data 为空也返回空对象 `{}` 或空数组 `[]`。

------

💡 **面试示例回答**：

> “统一接口返回结构可以提高项目的可维护性和前后端规范性。
> 后端统一返回 `{code, message, data}`，前端通过 axios 响应拦截器统一处理。
> 这样前端业务逻辑只需关注 `data`，错误信息也能统一提示，减少重复代码和错误处理成本。”

------

我可以帮你把 **前面整理的“参数传递 + 跨域 + token登录 + axios拦截器 + 统一返回结构”** 全部整合成 **一张完整流程图**，面试讲起来几句话就能把整个 Vue3 ↔ SpringBoot 调用流程说清楚，非常直观。

你希望我帮你画吗？



------

## 六、接口异常处理

### 6 Vue 如何处理接口异常？

axios：

```js
request.interceptors.response.use(
   res => res,
   error => {
      if(error.response.status === 401){
         router.push("/login")
      }
   }
)
```

SpringBoot：

```java
@RestControllerAdvice
public class GlobalExceptionHandler{
}
```

你整理的这一部分是 **接口异常处理**，面试中也很常问，尤其是 **全局异常处理和前端统一处理**。我帮你梳理成 **面试标准回答模板**，逻辑整，讲起来条理清晰。

------

### **1. 前端如何处理接口异常**

#### **使用 Axios 响应拦截器统一处理**

```js
request.interceptors.response.use(
  res => res, // 正常响应直接返回
  error => {
    const { status } = error.response;

    switch(status){
      case 401:
        // 未登录或 token 过期，跳转登录
        router.push("/login");
        break;
      case 403:
        alert("没有权限访问该资源");
        break;
      case 500:
        alert("服务器内部错误");
        break;
      default:
        alert(error.response.data?.message || "请求出错");
    }

    return Promise.reject(error);
  }
);
```

**优点：**

- 全局统一处理接口异常，避免在每个组件重复写 try/catch。
- 可结合 token 过期统一跳转登录。
- 错误信息可集中弹窗提示或日志记录。

------

### **2. 后端如何处理接口异常**

#### **Spring Boot 全局异常处理**

- 使用 `@RestControllerAdvice` + `@ExceptionHandler` 对异常统一处理：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UnauthorizedException.class)
    public Result handleUnauthorized(UnauthorizedException e){
        return Result.error(401, "未授权访问");
    }

    @ExceptionHandler(Exception.class)
    public Result handleException(Exception e){
        return Result.error(500, "服务器异常");
    }
}
```

**优点：**

- 后端接口异常统一返回格式。
- 前端可直接通过 `code` 或 `status` 处理错误。
- 避免每个 Controller 都写 try/catch。

------

### **3. 面试可能追问**

1. **前端为什么要统一处理异常？**
   - 避免重复代码，提高可维护性。
   - 可以统一处理 token 过期、跳转登录、提示信息。
2. **后端为什么要统一异常处理？**
   - 保持接口返回一致性（统一返回 `{code, message, data}`）。
   - 易于前端统一捕获和处理错误。
   - 避免泄露堆栈信息，提高安全性。
3. **401 和 403 区别？**
   - **401**：未登录或 token 过期 → 需要认证
   - **403**：权限不足 → 已登录，但无权限访问

------

💡 **面试示例回答**：

> “前端通过 axios 响应拦截器统一处理接口异常，如 token 过期跳转登录、403 提示权限不足、500 弹窗提示服务器错误。
> 后端使用 `@RestControllerAdvice` 全局异常处理，将异常统一封装为 `{code, message, data}` 返回给前端。
> 这样前后端都统一处理异常，减少重复代码，保证接口返回规范。”

------

我可以把你整理的 **前六大模块（参数传递 + 跨域 + token登录 + axios封装 +统一返回 +异常处理）** 全部整合成 **一张完整 Vue3 ↔ SpringBoot 接口调用流程图**，面试讲起来几句话就能把整个流程说清楚，非常直观。

你希望我画吗？

------

## 七、文件上传

### 7 Vue 如何上传文件？

Vue3：

```js
const form = new FormData()
form.append("file", file)

request.post("/upload", form)
```

SpringBoot：

```java
@PostMapping("/upload")
public String upload(MultipartFile file){
}
```

你整理的这一部分是 **前后端文件上传流程**，也是面试常问点之一。我帮你整理成 **面试标准回答模板**，讲起来逻辑清晰，面试官能快速理解你的掌握程度。



------

### **1. 前端上传文件**

#### **步骤：**

1. 创建 `FormData` 对象。
2. 调用接口 `POST /upload`。
3. 文件附加到 FormData。

```js
const form = new FormData();
form.append("file", file); // file 是 input 文件对象

request.post("/upload", form, {
  headers: {
    "Content-Type": "multipart/form-data"
  }
}).then(res => {
  console.log("上传成功", res);
}).catch(err => {
  console.error("上传失败", err);
});
```

------

#### **注意点：**

- 必须设置 `Content-Type` 为 `multipart/form-data`。
- 可以同时上传多个文件：

```js
files.forEach(file => form.append("files", file));
```

------

### **2. 后端接收文件（SpringBoot）**

```java
@PostMapping("/upload")
public Result upload(@RequestParam("file") MultipartFile file) {
    if(file.isEmpty()){
        return Result.error("文件为空");
    }

    // 保存文件，例如：
    String fileName = file.getOriginalFilename();
    String path = "/upload/" + fileName;
    file.transferTo(new File(path));

    return Result.success(path);
}
```

**要点：**

- 使用 `MultipartFile` 接收上传的文件。
- 可以结合统一返回结构 `{code, message, data}`。
- 可配置文件大小限制和存储路径（application.properties）。

------

### **3. 面试可能追问**

1. **如何上传多个文件？**

   - 前端 FormData 多次 append 或使用数组
   - 后端 `MultipartFile[] files` 接收

2. **如何防止文件覆盖？**

   - 文件重命名（加时间戳、UUID）
   - 保存到不同目录

3. **文件大小限制怎么处理？**

   - SpringBoot 配置：

     ```properties
     spring.servlet.multipart.max-file-size=10MB
     spring.servlet.multipart.max-request-size=20MB
     ```

4. **前端如何显示上传进度？**

   - Axios 支持 `onUploadProgress` 回调：

     ```js
     request.post("/upload", form, {
       onUploadProgress: progressEvent => {
         let percent = Math.round((progressEvent.loaded * 100) / progressEvent.total);
         console.log(`上传进度：${percent}%`);
       }
     })
     ```

------

💡 **面试示例回答**：

> “前端 Vue3 使用 FormData 包装文件，通过 axios POST 上传，Content-Type 为 `multipart/form-data`。
> 后端 Spring Boot 用 `@RequestParam MultipartFile file` 接收文件，保存后返回统一响应结构。
> 可以同时上传多个文件，并通过 UUID 重命名防止覆盖，也可通过 axios `onUploadProgress` 显示上传进度。”

------

如果你愿意，我可以帮你把 **前七大模块（参数传递、跨域、token登录、axios封装、统一返回、异常处理、文件上传）** 整合成 **一张完整流程图**，让 Vue3 ↔ SpringBoot 调用流程可视化，面试讲起来几句话就能把整个流程说清楚。

你希望我画吗？

------

## 八、接口防刷 / 限流（进阶）

可能问：

- 接口被恶意调用怎么办？

SpringBoot：

- **Alibaba Sentinel**
- Redis 限流
- token验证

你整理的这一部分是 **接口防刷/限流（进阶面试）**，我帮你整理成 **面试标准回答模板**，逻辑清晰，讲起来条理分明。



------

### **1. 面试问题**

- “接口被恶意调用怎么办？”
- “如何保证接口在高并发下不被刷爆？”

------

### **2. 常见解决方案**

| 方案                    | 原理                                      | 适用场景                     |
| ----------------------- | ----------------------------------------- | ---------------------------- |
| **Alibaba Sentinel**    | 流量控制、熔断降级                        | 高并发接口保护，复杂限流规则 |
| **Redis 限流**          | 计数器 + TTL 实现滑动窗口 / 漏桶 / 令牌桶 | 简单接口防刷，秒杀抢购等场景 |
| **Token 验证 / 防重放** | 每次请求生成唯一 token，重复请求拒绝      | 防止重复提交，保证幂等性     |

------

### **3. 示例：Redis 限流**

```java
public boolean isAllowed(String userId) {
    String key = "req:" + userId;
    long count = redisTemplate.opsForValue().increment(key, 1);
    if(count == 1){
        redisTemplate.expire(key, 1, TimeUnit.SECONDS);
    }
    return count <= 5; // 每秒最多5次
}
```

- 限制每秒每用户最多访问 5 次。
- 超出限制返回 429 或自定义提示。

------

### **4. Sentinel 限流示例**

```java
@GetMapping("/api/data")
@SentinelResource(value = "getData", blockHandler = "handleBlock")
public Result getData() {
    return Result.success("接口返回数据");
}

public Result handleBlock(BlockException ex) {
    return Result.error(429, "请求过于频繁，请稍后再试");
}
```

- Sentinel 支持：
  - 流控（QPS/线程数）
  - 熔断降级
  - 系统自适应保护
  - 热点参数限流

------

### **5. 面试加分点**

1. **结合 token 做幂等**：防止重复提交表单或接口被刷。
2. **前端防刷**：
   - 按钮防重复点击（前端禁用按钮 + loading）
   - 请求防抖 / 节流
3. **前后端结合**：
   - 前端 axios 拦截器捕获 429/限流异常
   - 后端统一返回 code/message，提示用户稍后再试

------

💡 **面试示例回答**：

> “接口防刷可以通过后端限流和前端防护结合来实现。
> 后端常用 Sentinel 做流量控制、Redis 做滑动窗口限流，同时 token 验证保证幂等性。
> 前端可以做按钮防重复点击、请求防抖，限流失败时统一提示用户稍后重试。”

------

如果你愿意，我可以帮你把 **前八大模块（参数传递 + 跨域 + token登录 + axios封装 + 统一返回 + 异常处理 + 文件上传 + 限流防刷）** 整合成 **一张完整 Vue3 ↔ SpringBoot 接口调用流程图**，面试讲起来几句话就能把整个流程梳理清楚，非常直观。

你希望我画吗？

------

## 九、开发环境代理（非常常见）

### 9 Vue 如何解决开发环境跨域？

使用 **Vite代理**

Vue3（vite.config.js）：

```js
server:{
  proxy:{
    "/api":{
      target:"http://localhost:8080",
      changeOrigin:true
    }
  }
}
```

请求：

```
/api/user/list
```



你整理的这一部分就是 **前端开发环境跨域处理**，面试中非常常问，尤其是 Vue3 + Vite 项目。下面帮你整理成 **面试标准回答模板**，逻辑清晰，条理完整。



------

### **1. 为什么开发环境会跨域？**

- 前端开发环境端口：`5173`
- 后端 SpringBoot 端口：`8080`
- **协议 / 域名 / 端口 任意不同** → 浏览器认为是跨域 → 请求被阻止

------

### **2. 解决方案：Vite 代理**

#### **vite.config.js 配置示例**

```js
export default {
  server: {
    proxy: {
      "/api": {
        target: "http://localhost:8080",  // 后端接口地址
        changeOrigin: true,               // 修改请求头中的 origin
        rewrite: path => path.replace(/^\/api/, "") // 可选，去掉 /api 前缀
      }
    }
  }
}
```

### **前端调用接口**

```js
request.get("/api/user/list")  // 会被代理转发到 http://localhost:8080/user/list
```

------

### **3. 面试可能追问**

1. **changeOrigin 的作用？**
   - 修改请求头中的 `Origin` 字段，使目标服务器认为请求来源于目标域，避免跨域限制。
2. **rewrite 的作用？**
   - 去掉前端请求中的 `/api` 前缀，匹配后端实际接口路径。
3. **代理只在开发环境生效吗？**
   - 是的，生产环境通常通过 **Nginx 反向代理** 或后端 CORS 配置解决跨域。
4. **为什么不直接在前端加 `@CrossOrigin`？**
   - 生产环境跨域可能涉及安全策略，开发环境代理更方便，减少修改后端代码。

------

### **4. 面试示例回答**

> “开发环境跨域是因为前端端口和后端端口不同，浏览器同源策略阻止请求。
> 在 Vue3 + Vite 项目中，可以通过配置 `server.proxy` 将 `/api` 请求转发到后端，`changeOrigin` 修改请求头，`rewrite` 可去掉前缀。
> 这样开发环境就不受跨域限制，生产环境可通过 Nginx 或后端 CORS 解决。”

------

💡 **面试加分点**：

- 可以顺带提到 **跨域和 token 认证结合**：
  - 代理请求自动带 token 不会触发跨域问题
  - 生产环境时要确保 token 可跨域传递（Cookie 或 header）

------

我可以把你整理的 **前九大模块（参数传递、跨域、token登录、axios封装、统一返回、异常处理、文件上传、限流、开发环境代理）** 整合成 **一张完整 Vue3 ↔ SpringBoot 接口调用流程图**，让面试讲起来一眼看懂整个流程。

你希望我画吗？

------

## 十、部署问题（高频）

### 10 Vue 打包后如何部署到 SpringBoot？

Vue：

```
npm run build
```

生成：

```
dist/
```

SpringBoot：

```
resources/static
```

访问：

```
http://localhost:8080/index.html
```

或者

- Nginx部署
- 前后端分离部署

你整理的这一部分就是 **前端打包与 SpringBoot 部署**，面试中非常常问。我帮你整理成 **面试标准回答模板**，讲起来逻辑清晰、条理明确。

------

### **1. 打包流程**

1. 前端执行：

```bash
npm run build
```

1. 生成打包文件夹：

```
dist/
```

- 里面包含 `index.html`、JS、CSS、静态资源等。

------

### **2. 部署到 SpringBoot**

1. 将 `dist/` 文件夹内容复制到 SpringBoot 项目：

```
src/main/resources/static/
```

1. 启动 SpringBoot 后端服务：

```
http://localhost:8080/index.html
```

- SpringBoot 会自动把 `static/` 目录下的资源作为静态资源提供访问。

------

### **3. 其他部署方式**

| 方式               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| **Nginx 部署**     | 前后端分离，Nginx 提供静态资源，反向代理 API 请求到 SpringBoot |
| **前后端分离部署** | 前端独立部署在 CDN / 静态服务器，后端 API 独立部署，跨域通过 CORS 或反向代理解决 |

------

### **4. 面试可能追问**

1. **为什么要放到 `resources/static`？**

   - SpringBoot 默认会把 `static`、`public`、`resources` 下的资源作为静态资源访问。

2. **前后端分离部署和打包到 SpringBoot 有什么区别？**

   - **打包到 SpringBoot**：前后端同一服务，访问方便，但不利于前端独立升级。
   - **前后端分离部署**：前端单独发布，可用 CDN 缓存，API 与前端独立升级，更灵活。

3. **Nginx 部署常用配置**

   - 反向代理 API 请求：

     ```nginx
     location /api/ {
       proxy_pass http://localhost:8080/api/;
     }
     location / {
       root /usr/share/nginx/html;
       index index.html;
       try_files $uri /index.html;
     }
     ```

4. **刷新页面为什么会 404？**

   - SPA 前端路由刷新时，需要 Nginx 或 SpringBoot 返回 `index.html`，否则会找不到对应路径。
   - 解决方式：`try_files $uri /index.html` 或 SpringBoot 配置 `WebMvcConfigurer`。

------

💡 **面试示例回答**：

> “Vue3 前端打包后生成 `dist/` 文件夹，可直接放到 SpringBoot `resources/static` 下部署访问。
> 在生产环境，也可以选择前后端分离部署，前端通过 Nginx 提供静态资源，反向代理 API 请求到 SpringBoot 后端。
> 打包部署时注意前端路由刷新问题，需要返回 `index.html` 以保证 SPA 正常访问。”

------

如果你愿意，我可以帮你把 **前十模块（参数传递 + 跨域 + token登录 + axios封装 +统一返回 +异常处理 +文件上传 +限流 +开发环境代理 +部署）** 整合成 **一张完整 Vue3 ↔ SpringBoot 面试流程图**，让面试讲起来几句话就能把整个流程梳理清楚，非常直观。

你希望我画吗？

------

## 十一、面试官最喜欢问的 10 个问题（总结）

几乎 **80%都会问**：

1. Vue3 怎么调用 SpringBoot 接口？
2. axios 为什么要封装？
3. Vue 与 SpringBoot 为什么会跨域？
4. 如何解决跨域？
5. token 登录流程？
6. axios 拦截器作用？
7. 如何统一接口返回？
8. 文件上传怎么做？
9. Vue 开发环境代理怎么配置？
10. Vue 打包后如何部署？

这份总结非常实用，确实涵盖了大多数 Vue3 和 Spring Boot 前后端对接时面试官常问的问题。我可以帮你扩展每个问题的 **典型回答要点**，确保你准备得更全面。

------

### 1. Vue3 怎么调用 SpringBoot 接口？

**典型回答要点：**

- 使用 **Axios** 或者其他请求库（如 Fetch）来发送 HTTP 请求。
- 可以通过 `GET`、`POST`、`PUT`、`DELETE` 等请求方法进行交互。
- 使用 **封装请求方法**，集中管理 API 请求和响应处理。

**代码示例：**

```js
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:8080/api',
  timeout: 5000
});

export const getUserList = () => api.get('/users');
export const login = (data) => api.post('/login', data);
```

------

### 2. axios 为什么要封装？

**典型回答要点：**

- **统一管理**：封装后可以集中处理请求的配置、拦截器、错误处理等。
- **代码复用**：统一的封装方法避免重复代码。
- **便于修改**：更改 API 基础路径或添加请求头时，只需要修改封装部分。

------

### 3. Vue 与 SpringBoot 为什么会跨域？

**典型回答要点：**

- **跨域问题**发生在 **浏览器端**，因浏览器的同源策略导致请求被拦截。
- **Spring Boot** 和 **Vue3** 各自运行在不同的端口下，构成了不同的域名和端口（例如：前端 `localhost:5173`，后端 `localhost:8080`）。
- **同源策略**限制了跨域请求。

------

### 4. 如何解决跨域？

**典型回答要点：**

- **Spring Boot** 使用 `@CrossOrigin` 注解，允许跨域请求。
- 可以配置 **CORS 策略**，在响应头中添加 `Access-Control-Allow-Origin`。
- 在开发环境中，使用 **代理配置**（例如 Vite 配置 `server.proxy`）来绕过跨域限制。

**代码示例：**

```java
@CrossOrigin(origins = "*")
@RestController
public class UserController {
}
```

------

### 5. token 登录流程？

**典型回答要点：**

1. 用户 **登录** 提交用户名和密码。
2. 后端验证成功后生成 **token**（例如 JWT），并将其返回前端。
3. 前端存储 token，一般存储在 `localStorage` 或 `sessionStorage`。
4. 后续的请求都将携带 token（通过 HTTP 请求头中的 `Authorization: Bearer token`）。
5. 后端通过 token 验证用户身份。

------

### 6. axios 拦截器作用？

**典型回答要点：**

- **请求拦截器**：在请求发送前对请求进行处理，例如添加 token 或者显示加载动画。
- **响应拦截器**：在接收到响应后，统一处理错误信息或成功的响应。

**代码示例：**

```js
// 请求拦截器
axios.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers['Authorization'] = `Bearer ${token}`;
  }
  return config;
});

// 响应拦截器
axios.interceptors.response.use(
  response => response,
  error => {
    if (error.response.status === 401) {
      // 处理 token 失效等
    }
    return Promise.reject(error);
  }
);
```

------

### 7. 如何统一接口返回？

**典型回答要点：**

- 统一定义接口的返回结构，确保每个接口都返回一致的格式（例如：`code`, `message`, `data`）。
- 在 Vue 中进行统一处理，方便后期维护和错误处理。

**代码示例：**

**Spring Boot** 返回：

```java
public class ApiResponse {
    private int code;
    private String message;
    private Object data;
}
```

**Vue 中处理：**

```js
axios.interceptors.response.use(response => {
  const { code, message, data } = response.data;
  if (code === 200) {
    return data;
  } else {
    alert(message);
    return Promise.reject(message);
  }
});
```

------

### 8. 文件上传怎么做？

**典型回答要点：**

- 前端使用 `FormData` 来上传文件。
- 后端使用 `@RequestParam` 或 `@RequestBody` 接收文件。

**代码示例：**

**Vue3** 文件上传：

```js
const formData = new FormData();
formData.append("file", fileInput.files[0]);

axios.post("/upload", formData, {
  headers: { 'Content-Type': 'multipart/form-data' }
})
.then(response => console.log(response));
```

**Spring Boot** 接收文件：

```java
@PostMapping("/upload")
public String uploadFile(@RequestParam("file") MultipartFile file) {
  // 处理文件
  return "File uploaded successfully!";
}
```

------

### 9. Vue 开发环境代理怎么配置？

**典型回答要点：**

- Vue3 开发时，通过配置 `vite.config.js` 中的 `server.proxy`，将请求代理到后端接口，避免跨域问题。

**代码示例：**

```js
// vite.config.js
export default {
  server: {
    proxy: {
      '/api': 'http://localhost:8080',
    },
  },
};
```

------

### 10. Vue 打包后如何部署？

**典型回答要点：**

- **Vue3 打包**：执行 `npm run build`，生成 `dist/` 文件夹。
- **部署方式**：
  1. 将 `dist/` 文件夹中的文件拷贝到 Spring Boot 项目的 `resources/static/` 文件夹。
  2. 使用 **Nginx** 部署前后端分离项目。

**代码示例（Spring Boot 配置）：**

```java
// 将 Vue 打包后的文件放在 resources/static 目录下即可
```

------

这十个问题和解答覆盖了大多数 Vue3 与 Spring Boot 对接时的关键点，准备这些问题，能让你应对面试中的大部分情况。

------

## 十二、真正“高级一点”的问题

面试 10k+ 会问：

- token 为什么不用 session？
- token 过期如何处理？
- refresh token 怎么设计？
- 前端如何防止重复请求？
- 如何防止 CSRF？
- 如何做接口幂等？

这些问题属于 **高级问题**，面试官问这些问题是为了评估你对 **安全性、性能优化和架构设计** 的理解。在这里，我给你详细解析这些问题的回答要点，并提供一些常见的解决方案。

------

### 1. **token 为什么不用 session？**

**典型回答要点：**

- **无状态**：`token` 采用 **无状态** 机制，后端不需要保存会话信息，所有的身份验证信息都保存在 **token** 本身。与传统的 **session** 不同，`session` 会在后端存储用户的登录状态，导致 **状态共享问题** 和 **可扩展性** 问题。
- **跨域**：`token` 适用于 **前后端分离** 和 **跨域请求**，在每次请求时，前端将 `token` 作为请求头传递给后端，后端可以独立地验证其合法性。
- **性能**：使用 `token` 可以减少 **后端存储负担**，没有 session 相关的数据库查询或存储操作，因此提高了性能。
- **适应分布式系统**：`token` 是 **自包含** 的，容易与 **微服务架构** 配合使用。多个后端服务可以共享相同的 `token`，不需要在各个服务中维护 session。

------

### 2. **token 过期如何处理？**

**典型回答要点：**

- **Token 过期机制**：通常情况下，`token` 会有一个 **过期时间**（如 1小时、24小时），当 `token` 过期时，用户需要重新进行身份验证。
- **如何处理过期？**：
  - **前端处理**：前端可以在收到 `401 Unauthorized` 错误时，判断 `token` 是否过期。如果过期，可以清除本地存储的 `token`，并引导用户重新登录。
  - **刷新 Token（Refresh Token）**：为防止频繁登录，通常会引入 `refresh token`，它有较长的过期时间（如 7天、30天），当 `access token` 过期时，前端可以使用 `refresh token` 来获取新的 `access token`。

**代码示例：**

前端：

```js
axios.interceptors.response.use(response => response, error => {
  if (error.response.status === 401) {
    // 清除无效 token，跳转到登录页
    localStorage.removeItem('token');
    router.push('/login');
  }
  return Promise.reject(error);
});
```

------

### 3. **refresh token 怎么设计？**

**典型回答要点：**

- **设计思路**：
  - **access token**：短期有效，通常是 1小时或更短。用于在每次请求时验证用户身份。
  - **refresh token**：长期有效，通常是 7天、30天等，用于在 `access token` 过期时，获取新的 `access token`。
- **刷新过程**：
  - 用户首次登录时，后端生成 **access token** 和 **refresh token**。
  - 前端将 **access token** 存储在 `localStorage` 或 `sessionStorage` 中，**refresh token** 存储在 **HttpOnly Cookie** 中，增加安全性。
  - 当 **access token** 过期时，前端会使用 **refresh token** 向后端请求新的 `access token`。
- **安全性考虑**：
  - **refresh token** 应该只存储在 **HttpOnly** 和 **Secure** 的 Cookie 中，避免被 JavaScript 获取。
  - 如果 **refresh token** 被泄露，应立即注销或失效。

**代码示例：**

```java
public class TokenResponse {
  private String accessToken;
  private String refreshToken;
}

@PostMapping("/refresh-token")
public TokenResponse refreshToken(@RequestParam("refreshToken") String refreshToken) {
  // 验证 refresh token 是否有效
  // 如果有效，返回新的 access token 和 refresh token
}
```

------

### 4. **前端如何防止重复请求？**

**典型回答要点：**

- **前端防重复请求的原因**：防止用户快速重复点击导致多次发送相同请求，增加后端负担，导致冗余操作。
- **防止方式**：
  - **禁用按钮**：请求发起后禁用按钮，直到响应返回。
  - **请求去重**：使用 **标识符** 来标记是否已有请求在进行。可以通过 `axios` 拦截器等方式去重。

**代码示例：**

```js
// 使用全局变量防止重复请求
let isRequesting = false;

axios.interceptors.request.use(config => {
  if (isRequesting) {
    return Promise.reject("请求正在进行中");
  }
  isRequesting = true;
  return config;
});

axios.interceptors.response.use(response => {
  isRequesting = false;
  return response;
}, error => {
  isRequesting = false;
  return Promise.reject(error);
});
```

------

### 5. **如何防止 CSRF？**

**典型回答要点：**

- **CSRF（跨站请求伪造）** 是一种攻击，攻击者诱导用户发送恶意请求，利用用户的身份信息执行非法操作。
- **防护策略**：
  - **使用 SameSite Cookie**：设置 Cookie 的 `SameSite` 属性为 `Strict` 或 `Lax`，防止跨站请求携带 Cookie。
  - **Token 机制**：使用 **CSRF token** 来验证请求是否合法。每次请求时，前端都会从后端获取一个唯一的 **CSRF token**，并将其添加到请求头或表单中。后端验证该 token 是否匹配。

**代码示例：**

前端：

```js
axios.defaults.headers.common['X-CSRF-TOKEN'] = csrfToken;
```

后端（Spring Boot）：

```java
@RequestMapping(value = "/submit", method = RequestMethod.POST)
public String handlePost(@RequestHeader("X-CSRF-TOKEN") String csrfToken) {
  // 验证 token 是否有效
}
```

------

### 6. **如何做接口幂等？**

**典型回答要点：**

- **幂等性** 是指多次执行同一操作的结果是相同的，即使重复请求也不会对系统状态产生副作用。
- **常见方法**：
  - **使用唯一标识符**：例如生成一个唯一的请求 ID，将其作为请求的一部分（例如在请求头或参数中），后端对该请求 ID 进行记录，防止相同请求重复执行。
  - **数据库事务控制**：确保数据库操作是原子性的，避免因重复请求产生不一致的状态。
  - **乐观锁**：利用数据库中的版本字段，确保操作时不会出现并发冲突。

**代码示例：**

前端：

```js
const requestId = uuidv4();
axios.post("/api/submit", data, { headers: { 'Request-ID': requestId } });
```

后端：

```java
@PostMapping("/submit")
public ResponseEntity submit(@RequestHeader("Request-ID") String requestId) {
    if (requestService.isDuplicateRequest(requestId)) {
        return ResponseEntity.status(HttpStatus.CONFLICT).body("Request already processed");
    }
    requestService.process(requestId);
    return ResponseEntity.ok("Success");
}
```

------

这些问题考察的不仅是基础技术的掌握，还有对 **系统安全性、性能优化、设计模式** 的理解。掌握这些高级问题并在面试中流畅回答，会显著提高面试的成功率。