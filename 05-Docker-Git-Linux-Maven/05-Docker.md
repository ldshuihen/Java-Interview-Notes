# Docker

## 一、 环境准备与全局配置：Docker 配置镜像源与加速器

（CentOS Stream 9 / Linux通用）

> 原理：修改 `daemon.json`，配置国内镜像加速器，解决 `docker pull` 超时、下载慢问题

#### 1. 检查是否已有配置文件

```bash
# 查看文件是否存在
ls /etc/docker/daemon.json
```
- 如果文件不存在：直接新建
- 如果已经存在：**不要直接覆盖**，把 registry-mirrors 追加进去，防止覆盖原有配置（代理、log等）

#### 2. 编辑 daemon.json

```bash
vim /etc/docker/daemon.json
```
写入下面配置（推荐中科大、网易镜像）
```json
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com"
  ]
}
```
> 可选阿里云镜像：需要登录阿里云控制台获取专属加速地址，格式类似 `https://xxxx.mirror.aliyuncs.com`

#### 3. 重载配置并重启docker（关键步骤，缺一不可）

```bash
# 重新加载系统服务配置
systemctl daemon-reload

# 重启docker
systemctl restart docker
```

#### 4. 验证镜像源是否生效

```bash
docker info
```
往下找 `Registry Mirrors`，能看到你配置的地址，代表成功。

#### 5. 常见坑

1. **JSON语法错误**：`daemon.json` 必须严格json，不能有多余逗号，否则docker启动失败
   - 校验json：`cat /etc/docker/daemon.json`
2. **docker启动失败**
```bash
# 查看docker日志排查
journalctl -u docker
```
3. 如果你用的是 `docker-compose`，**不需要单独配置**，compose会复用docker全局镜像源

#### 备选镜像源清单（按需替换）

```json
"registry-mirrors": [
  "https://docker.mirrors.ustc.edu.cn",
  "https://hub-mirror.c.163.com",
  "https://mirror.baidubce.com"
]
```

> ⚠️ 注意：Docker 官方已经不再维护 `docker.io` 的国内免费公共镜像源，部分旧镜像地址会失效，优先用上面中科大、百度云、网易。

---

## 二、 镜像构建与打包：Dockerfile 详解

Dockerfile 是构建 Docker 镜像的**文本蓝图**，由一组按顺序执行的指令组成，每条指令生成一个镜像层，最终打包为可分发的镜像。下面从**基础概念、核心指令、构建流程、多阶段构建、最佳实践、完整示例**六个维度详细介绍。

---

#### 一、基础概念

##### 1. 什么是 Dockerfile？

- 纯文本文件，命名为 `Dockerfile`（首字母大写）。
- 包含一系列指令，自动化构建镜像，确保环境一致性。
- 核心价值：**自动化、可重复、可版本化、透明化**。

##### 2. 基本语法规则

- 指令**大写**（约定俗成，区分参数），参数小写。
- 从上到下顺序执行，**每条指令生成一个镜像层**。
- `#` 开头为注释，不影响构建。
- 必须以 `FROM` 开头（解析器指令、`ARG` 除外）。
- 支持**行续符 `\`**，跨行写指令。

##### 3. 构建上下文（Build Context）

- `docker build` 命令执行目录下的所有文件，是镜像构建的“资源池”。
- `.dockerignore` 排除无关文件（如 `.git`、`node_modules`、日志），减小构建体积、加速构建。

---

#### 二、核心指令详解（按常用顺序）

##### 1. `FROM`：指定基础镜像（必选）

- **作用**：定义新镜像的基础，所有镜像都基于此构建。
- **语法**：
  ```dockerfile
  FROM <image>[:<tag>] [AS <name>]
  ```
- **示例**：
  ```dockerfile
  FROM ubuntu:22.04
  FROM node:18-alpine AS builder  # 多阶段构建，命名阶段
  ```
- **最佳实践**：使用**官方轻量镜像**（如 `alpine`），固定 `tag`（避免 `latest` 不确定性）。

##### 2. `ARG`：构建时参数

- **作用**：定义构建过程中的变量，仅构建时有效，容器运行时不可见。
- **语法**：
  ```dockerfile
  ARG <name>[=<default>]
  ```
- **示例**：
  ```dockerfile
  ARG VERSION=1.0
  FROM ubuntu:${VERSION}
  ```
- **传参**：`docker build --build-arg VERSION=2.0 -t my-img .`

##### 3. `LABEL`：镜像元数据

- **作用**：添加镜像描述信息（作者、版本、描述），替代旧指令 `MAINTAINER`。
- **语法**：
  ```dockerfile
  LABEL <key>=<value> <key>=<value> ...
  ```
- **示例**：
  ```dockerfile
  LABEL maintainer="user@example.com"
  LABEL version="1.0" description="My Docker Image"
  ```

##### 4. `ENV`：环境变量

- **作用**：设置容器运行时的环境变量，构建和运行时均有效。
- **语法**：
  ```dockerfile
  ENV <key>=<value>
  ENV <key1>=<value1> <key2>=<value2>
  ```
- **示例**：
  ```dockerfile
  ENV PATH=/app/bin:$PATH
  ENV NODE_ENV=production
  ```

##### 5. `WORKDIR`：工作目录

- **作用**：指定后续 `RUN`、`CMD`、`ENTRYPOINT`、`COPY`、`ADD` 的默认工作目录。
- **语法**：
  ```dockerfile
  WORKDIR /path/to/dir
  ```
- **示例**：
  ```dockerfile
  WORKDIR /app
  RUN pwd  # 输出 /app
  ```
- **最佳实践**：用**绝对路径**，避免多次 `RUN cd`，直接用 `WORKDIR` 替代。

##### 6. `COPY`：复制文件（推荐）

- **作用**：从**构建上下文**复制文件/目录到镜像内。
- **语法**：
  ```dockerfile
  COPY <src> <dest>
  COPY ["<src>", "<dest>"]  # 路径含空格时用
  ```
- **示例**：
  ```dockerfile
  COPY . /app
  COPY target/app.jar /app/
  ```
- **特点**：仅本地文件、不解压、不支持 URL、**可缓存、安全**。

##### 7. `ADD`：增强复制（慎用）

- **作用**：类似 `COPY`，额外支持：
  - 自动解压本地 `tar.gz` 到目标目录。
  - 从 URL 下载文件到镜像。
- **语法**：同 `COPY`。
- **示例**：
  ```dockerfile
  ADD https://example.com/file.tar.gz /tmp/
  ADD app.tar.gz /app/
  ```
- **最佳实践**：优先用 `COPY`；仅需解压或下载时用 `ADD`。

##### 8. `RUN`：构建时执行命令

- **作用**：在构建镜像时执行命令（如安装软件、编译代码），**生成新镜像层**。
- **语法**：
  - Shell 格式（默认 `/bin/sh -c`）：
    ```dockerfile
    RUN <command>
    ```
  - Exec 格式（推荐，避免 shell 问题）：
    ```dockerfile
    RUN ["executable", "param1", "param2"]
    ```
- **示例**：
  ```dockerfile
  RUN apt update && apt install -y nginx
  RUN ["apt", "install", "-y", "curl"]
  ```
- **最佳实践**：
  - **合并多个 `RUN`**，减少镜像层数（如 `RUN apt update && apt install -y nginx curl`）。
  - 清理缓存（如 `apt clean`），减小镜像体积。

##### 9. `EXPOSE`：声明端口

- **作用**：声明容器运行时监听的端口，**仅文档说明，不自动开放端口**。
- **语法**：
  ```dockerfile
  EXPOSE <port> [<port>/<protocol>]
  ```
- **示例**：
  ```dockerfile
  EXPOSE 80
  EXPOSE 443/tcp
  ```
- **开放端口**：运行时用 `-p` 映射端口：`docker run -p 8080:80 my-img`。

##### 10. `VOLUME`：数据卷

- **作用**：创建匿名卷，挂载到容器指定目录，**持久化数据、共享数据**。
- **语法**：
  ```dockerfile
  VOLUME ["/path/to/dir"]
  ```
- **示例**：
  ```dockerfile
  VOLUME /data
  ```
- **注意**：运行时可挂载命名卷或宿主机目录：`docker run -v my-volume:/data my-img`。

##### 11. `USER`：运行用户

- **作用**：指定容器运行时的用户（默认 `root`），**提升安全性**。
- **语法**：
  ```dockerfile
  USER <user>[:<group>]
  ```
- **示例**：
  ```dockerfile
  USER appuser
  ```
- **最佳实践**：避免用 `root` 运行容器，创建普通用户并授权。

##### 12. `CMD`：容器启动命令

- **作用**：定义容器启动后默认执行的命令，**运行时可被覆盖**。
- **语法**：
  - Shell 格式：
    ```dockerfile
    CMD <command>
    ```
  - Exec 格式（推荐）：
    ```dockerfile
    CMD ["executable", "param1", "param2"]
    ```
- **示例**：
  ```dockerfile
  CMD ["java", "-jar", "app.jar"]
  CMD echo "Container started"
  ```
- **注意**：一个 Dockerfile 仅**最后一个 `CMD` 生效**。
- **覆盖**：`docker run my-img java -jar custom.jar`。

##### 13. `ENTRYPOINT`：入口点（不可覆盖）

- **作用**：与 `CMD` 类似，但**运行时不可被覆盖**，`CMD` 可作为其参数。
- **语法**：同 `CMD`。
- **示例**：
  ```dockerfile
  ENTRYPOINT ["java", "-jar"]
  CMD ["app.jar"]
  ```
- **运行**：`docker run my-img custom.jar` → 执行 `java -jar custom.jar`。

##### 14. `HEALTHCHECK`：健康检查

- **作用**：定义容器健康状态检查命令，Docker 定期执行，判断容器是否正常运行。
- **语法**：
  ```dockerfile
  HEALTHCHECK [OPTIONS] CMD <command>
  ```
  - 选项：`--interval`（检查间隔，默认 30s）、`--timeout`（超时，默认 30s）、`--retries`（重试次数，默认 3）。
- **示例**：
  ```dockerfile
  HEALTHCHECK --interval=5s --timeout=3s CMD curl -f http://localhost/ || exit 1
  ```

##### 15. `ONBUILD`：延迟执行指令

- **作用**：当该镜像作为**基础镜像**被其他 Dockerfile 构建时，触发执行 `ONBUILD` 定义的指令。
- **语法**：
  ```dockerfile
  ONBUILD <instruction>
  ```
- **示例**：
  ```dockerfile
  ONBUILD COPY . /app
  ONBUILD RUN npm install
  ```

---

#### 三、镜像构建流程

1. 编写 `Dockerfile` 和 `.dockerignore`。
2. 执行构建命令：
   ```bash
   docker build -t my-image:1.0 .
   ```
   - `-t`：指定镜像标签（名称:版本）。
   - `.`：构建上下文为当前目录。
3. Docker 按顺序执行指令，每条指令生成一个镜像层，最终合并为新镜像。
4. 查看镜像：`docker images`。
5. 运行容器：`docker run -d -p 8080:80 my-image:1.0`。

---

#### 四、多阶段构建（Multi-stage Builds）

##### 1. 作用

- **减小最终镜像体积**：将构建过程（编译、依赖安装）与运行环境分离，最终镜像仅保留运行时必需文件。
- **提高构建效率**：并行执行构建阶段。

##### 2. 语法

- 用 `AS <name>` 命名构建阶段，后续阶段可复制前一阶段文件。

##### 3. 示例（Node.js 应用）

```dockerfile
# 构建阶段：编译代码、安装依赖
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# 运行阶段：仅保留运行时必需文件
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm install --production
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

---

#### 五、最佳实践

1. **最小化基础镜像**：优先用 `alpine`、`slim` 等轻量镜像，减小体积、提升安全性。
2. **合并 `RUN` 指令**：减少镜像层数，加速构建、减小体积。
3. **使用 `.dockerignore`**：排除无关文件，减小构建上下文大小。
4. **固定镜像版本**：避免用 `latest`，确保构建一致性。
5. **非 root 用户运行**：创建普通用户，提升容器安全性。
6. **合理使用缓存**：将不常变动的指令（如 `COPY package*.json`）放在前面，利用 Docker 缓存加速构建。
7. **多阶段构建**：分离构建与运行环境，减小最终镜像体积。
8. **清理无用依赖**：`RUN` 后清理缓存（如 `apt clean`、`npm cache clean`）。

---

#### 六、完整示例（Nginx 静态网站）

##### 1. 项目结构

```
my-nginx/
├── Dockerfile
├── .dockerignore
└── html/
    └── index.html
```

##### 2. `.dockerignore`

```
.git
node_modules
*.log
.DS_Store
```

##### 3. `Dockerfile`

```dockerfile
# 基础镜像
FROM nginx:1.25-alpine

# 元数据
LABEL maintainer="user@example.com"
LABEL version="1.0" description="Nginx Static Website"

# 复制静态文件
COPY html/ /usr/share/nginx/html/

# 声明端口
EXPOSE 80

# 启动命令
CMD ["nginx", "-g", "daemon off;"]
```

##### 4. 构建与运行

```bash
# 构建镜像
docker build -t my-nginx:1.0 .

# 运行容器
docker run -d -p 8080:80 --name my-nginx-container my-nginx:1.0

# 访问
curl http://localhost:8080
```

---

#### 总结

Dockerfile 是容器化应用的核心，通过**指令化、自动化**构建镜像，确保环境一致性、可重复性。掌握核心指令、构建流程、多阶段构建和最佳实践，可高效构建**轻量、安全、可维护**的 Docker 镜像。

### Docker 构建 Java 应用

（超详细 + 最佳实践）

我给你一份**生产环境可用、体积最小、速度最快**的 Java 项目 Docker 构建方案，包含 **Maven / Gradle 项目**、**多阶段构建**、**JDK/JRE 区分**、**启动优化**，直接复制就能用。

#### 一、先搞懂：Java 镜像怎么选？

Java 构建 Docker 镜像分两步：
1. **构建阶段**：需要 JDK（编译代码、打包）
2. **运行阶段**：只需要 JRE（运行 jar，更小更安全）

**最佳基础镜像**（轻量 + 官方 + 安全）：
- `eclipse-temurin:17-jdk-jammy` （构建用 JDK）
- `eclipse-temurin:17-jre-alpine` （运行用 JRE，极小）

> 不推荐 openjdk:8 之类，已停止维护！

---

#### 二、最简单 Dockerfile（单阶段，适合测试）

适合快速打包，**不追求镜像大小**

```dockerfile
# 基础镜像（带JDK）
FROM eclipse-temurin:17-jre-jammy

# 工作目录
WORKDIR /app

# 复制jar包（把你本地打好的jar放进来）
COPY target/demo-0.0.1-SNAPSHOT.jar app.jar

# 暴露端口
EXPOSE 8080

# 启动命令
ENTRYPOINT ["java", "-jar", "app.jar"]
```

##### 构建 & 运行

```bash
# 先打包jar
mvn clean package -DskipTests

# 构建镜像
docker build -t java-app:1.0 .

# 运行容器
docker run -d -p 8080:8080 --name my-java-app java-app:1.0
```

---

#### 三、生产级方案：多阶段构建（强烈推荐）

**优点**：
- 最终镜像**极小**（只有几十M）
- 不包含源码、编译依赖
- 安全、干净、速度快

##### 最终 Dockerfile（直接复制使用）

```dockerfile
# ======================
# 第一阶段：构建（JDK）
# ======================
FROM eclipse-temurin:17-jdk-jammy AS builder
WORKDIR /app

# 先复制pom，利用缓存（重点！加速构建）
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
RUN ./mvnw dependency:go-offline

# 复制源码
COPY src ./src

# 打包（跳过测试）
RUN ./mvnw clean package -DskipTests

# ======================
# 第二阶段：运行（仅JRE）
# ======================
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# 从构建阶段复制jar包
COPY --from=builder /app/target/*.jar app.jar

# 非root运行（安全）
RUN addgroup -S app && adduser -S app -G app
USER app

# 端口 + 启动
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

##### 这个镜像有多小？

- 普通单阶段：~500MB  
- 多阶段 alpine：~80MB  

**差 6 倍！**

---

#### 四、SpringBoot 专用优化（必加）

给 Java 启动加 JVM 参数，让容器更稳定：

```dockerfile
ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-jar", "app.jar"]
```

解释：
- `UseContainerSupport`：让 JVM 识别容器内存
- `MaxRAMPercentage=75`：限制最大内存占比（避免OOM）

---

#### 五、常用命令速查

```bash
# 构建镜像
docker build -t java-app:1.0 .

# 运行
docker run -d -p 8080:8080 java-app:1.0

# 查看日志
docker logs -f my-java-app

# 进入容器
docker exec -it my-java-app sh
```

---

#### 六、我帮你定制（你只需告诉我）

你直接回复我这 3 个信息，我可以给你**生成专属完美 Dockerfile**：
1. JDK 版本（8/11/17/21）
2. SpringBoot 还是普通 Java？
3. 你的 jar 包名称/路径

### Docker 构建 Vue3 项目

（超简单 + 生产最小镜像）

Vue3 是**前端静态项目**，Docker 构建最佳方案：
**多阶段构建** → 先用 Node 环境打包编译，再用 Nginx 运行静态文件，最终镜像只有 **20MB 左右**！

---

#### 一、先看你的项目结构（标准 Vue3/Vite 项目）

```
你的项目/
├── Dockerfile         # 我们要创建的
├── .dockerignore      # 可选，推荐
├── package.json
├── vite.config.js     或  vue.config.js
└── src/
```

---

#### 二、直接可用的 Dockerfile（复制即用）

```dockerfile
# ======================================
# 阶段1：构建阶段 —— 编译 Vue3 代码
# ======================================
FROM node:18-alpine AS builder

# 设置工作目录
WORKDIR /app

# 复制依赖文件（利用Docker缓存，加速构建）
COPY package*.json ./

# 安装依赖（淘宝镜像，速度更快）
RUN npm config set registry https://registry.npmmirror.com && \
    npm install

# 复制全部项目代码
COPY . .

# 打包构建（生成 dist 文件夹）
RUN npm run build

# ======================================
# 阶段2：运行阶段 —— Nginx 托管静态文件
# ======================================
FROM nginx:stable-alpine

# 复制构建好的 dist 到 Nginx 目录
COPY --from=builder /app/dist /usr/share/nginx/html

# 暴露 80 端口
EXPOSE 80

# 启动 Nginx
CMD ["nginx", "-g", "daemon off;"]
```

---

#### 三、创建 .dockerignore（必须加，减小体积）

创建 `.dockerignore` 文件，写入：
```
node_modules
dist
.DS_Store
.git
.gitignore
*.log
```

---

#### 四、一键构建 + 运行

##### 1. 构建镜像

```bash
docker build -t vue3-app:1.0 .
```

##### 2. 启动容器

```bash
docker run -d -p 8080:80 --name my-vue3 vue3-app:1.0
```

##### 3. 访问

打开浏览器访问：
```
http://localhost:8080
```

---

#### 五、如果你的 Vue 项目路由是 history 模式（必看）

history 模式直接用上面的 Dockerfile 会**刷新 404**，需要加 Nginx 配置：

##### 1. 项目根目录创建 `nginx.conf`

```nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    # 解决 history 路由刷新 404 问题
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

##### 2. 修改 Dockerfile，添加配置

```dockerfile
# 在运行阶段增加一行
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

完整运行阶段：
```dockerfile
FROM nginx:stable-alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

#### 六、核心优势

- **超小体积**：最终镜像 ≈ 20MB
- **生产可用**：Nginx 高性能、稳定
- **安全干净**：不包含源码、依赖包
- **速度快**：利用缓存，构建超快

---

#### 七、常用命令

```bash
# 构建
docker build -t vue3-app .

# 运行
docker run -d -p 8080:80 vue3-app

# 停止
docker stop my-vue3

# 删除容器
docker rm my-vue3
```

需要我帮你适配 **Vite + Vue3** 或 **Vue-CLI** 专属版本吗？

## 三、 多容器编排：Docker Compose 详解

> 定位：**Docker Compose 是用于定义和运行多容器 Docker 应用的工具**。
> 一个项目有 MySQL、Redis、Nacos、MinIO、Nginx、Java微服务多个容器时，不用手动敲一堆 `docker run`，全部写在 `docker-compose.yml`，一条命令启停整套服务。
>
> ✅ V2：`docker compose`（CLI插件，官方主推，推荐）
> ✅ V1：`docker-compose`（独立Python程序，已废弃）

#### 一、核心概念

1. **服务(service)**：一个容器，比如 mysql、redis，对应 yml 里的 `services` 节点
2. **项目(project)**：compose 管理的整套应用，默认以 yml 所在文件夹名称作为项目名；容器/网络/卷都会带上项目名前缀隔离
3. **网络(network)**：compose 默认自动创建一个网桥，同一个compose内的容器**可以直接用服务名互相访问**，不需要宿主机IP。例如 java 服务连接 mysql，host直接写 `mysql`，端口写容器内端口 `3306`，不用宿主机映射端口。
4. **数据卷(volume)**：持久化数据，容器删除数据不丢失；支持命名卷、绑定挂载（宿主机目录映射）

#### 二、文件：docker-compose.yml

默认文件名：`docker-compose.yml` 或者 `docker-compose.yaml`
> 执行命令时默认读取当前目录下这个文件；自定义文件名用 `-f` 指定：`docker compose -f my-compose.yml up -d`

##### yml 顶层关键字（最常用）

```yaml
# 1. 指定compose文件版本（新版compose v2 已经不需要写version，老教程才有）
# version: '3.8'

# 2. 服务列表，核心
services:
  mysql:
    image: mysql:8.0   # 使用镜像；如果是自己构建写 build
    container_name: mysql8  # 容器名，可选，不写则自动生成【项目名-服务名-序号】
    restart: always  # 重启策略：always/on-failure/no/unless-stopped
    ports:
      - "3306:3306"  # 宿主机端口:容器端口，字符串加引号防止解析错误
    environment:     # 环境变量
      MYSQL_ROOT_PASSWORD: root123
    volumes:
      - ./mysql-data:/var/lib/mysql  # 绑定挂载：宿主机目录:容器内目录
      - ./conf/my.cnf:/etc/mysql/my.cnf
    networks:
      - my-net # 加入自定义网络
    depends_on:
      - redis # 依赖：启动顺序，先启动redis，再启动mysql
    # depends_on 只保证启动顺序！**不等待应用就绪**（mysql启动进程起来≠数据库可连接），代码里自己做重试！
    extra_hosts: # 主机映射
      - "host.docker.internal:host-gateway"

  redis:
    image: redis:7-alpine
    networks:
      - my-net

# 3. 数据卷定义（命名卷）
volumes:
  mysql-volume:

# 4. 自定义网络
networks:
  my-net:
    driver: bridge
```

##### 关键字详细说明

1. **image**：拉取已有镜像 `mysql:8.0`
2. **build**：本地构建镜像（替代image）
```yaml
build:
  context: ./java-app  # Dockerfile所在目录
  dockerfile: Dockerfile
```
3. **restart 重启策略**
    - `no`：默认，容器退出不重启
    - `always`：无论怎么退出，永远重启（生产数据库推荐）
    - `unless-stopped`：除非手动stop，否则一直重启（最常用）
    - `on-failure`：仅异常退出（返回码非0）才重启
4. **volumes 两种写法**
    - 绑定挂载：`./mysql-data:/var/lib/mysql`，宿主机目录，修改宿主机文件容器同步，目录权限坑多
    - 命名卷：`mysql-volume:/var/lib/mysql`，docker管理，存在docker数据目录，性能更好
5. **depends_on**
   
    > ⚠️ 重点面试坑：**只控制容器启动顺序，不会等待服务就绪**。比如mysql容器进程启动了，但数据库还在初始化，java直接连会报错。解决方案：代码层重试、wait-for-it脚本。
6. **ports**
    - `"宿主机IP:宿主机端口:容器端口"` 例如 `"127.0.0.1:3306:3306"` 只允许本机访问
    - `"3306:3306"` 监听所有网卡
7. **env_file**：从外部文件加载环境变量，适合配置分离
```yaml
env_file:
  - ./.env
```
`.env` 文件：
```properties
MYSQL_PWD=root123
```
> compose会自动读取当前目录 `.env`，不需要写env_file也会加载。

#### 三、常用命令（V2版本 `docker compose`）

```bash
# 后台启动所有服务（最常用）
docker compose up -d

# 前台启动，看日志，调试用
docker compose up

# 构建镜像并启动（build节点才需要）
docker compose up -d --build

# 查看运行中的服务状态
docker compose ps

# 查看日志，实时跟踪
docker compose logs -f

# 只看mysql服务日志
docker compose logs -f mysql

# 停止服务，容器保留，数据卷保留
docker compose stop

# 启动已经停止的服务
docker compose start

# 停止并且删除容器、默认网络；**数据卷不会删除！**
docker compose down

# down 同时删除数据卷（谨慎！数据全部清空）
docker compose down -v

# 重启指定服务，例如只重启mysql
docker compose restart mysql

# 拉取yml里所有镜像最新版本
docker compose pull

# 构建镜像，不启动
docker compose build

# 进入容器内部
docker compose exec mysql bash

# 查看compose解析后的完整配置（排查yml语法神器）
docker compose config
```

#### 四、网络机制（高频面试点）

1. 默认行为：执行 `up`，compose自动创建**项目名_default**网桥，所有service默认加入这个网络。
2. 同compose内容器，**直接使用服务名作为hostname访问**
    - java服务连接mysql，host写 `mysql`，端口写容器内端口 `3306`，**不需要宿主机IP，不需要宿主机端口映射**！
    - 端口映射 `ports` 只有**外部（宿主机/其他机器）访问容器才需要**；容器之间通信不需要ports。
3. 不同compose项目，默认网络隔离，不能互相访问；如需互通，可以加入同一个自定义network。

#### 五、环境变量 .env 文件

目录结构：
```
project/
├── docker-compose.yml
└── .env
```
`.env`
```properties
MYSQL_IMAGE=mysql:8.0
MYSQL_ROOT_PASSWORD=Root@123456
MYSQL_PORT=3306
```
yml中引用：
```yaml
services:
  mysql:
    image: ${MYSQL_IMAGE}
    ports:
      - "${MYSQL_PORT}:3306"
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
```

#### 六、compose 部署典型坑点

1. **yml语法严格，缩进是2空格，不能用tab**，写错直接启动失败；`docker compose config` 校验
2. depends_on 不等待服务就绪，只是容器启动顺序
3. 绑定挂载宿主机目录权限问题：CentOS Stream9 selinux，目录权限不足，容器无法读写，加 `:z` 标签
```yaml
volumes:
  - ./mysql-data:/var/lib/mysql:z
```
4. V1和V2容器命名规则不一样，混用会两套容器，不要交替使用 `docker-compose` 和 `docker compose`
5. down 默认**不会删除volume**，持久化数据还在；`down -v` 删除卷，谨慎使用
6. 镜像拉取慢：复用docker全局daemon.json镜像加速器，compose不需要单独配置

#### 七、适用场景 & 不适用场景

✅ 适合：本地开发、测试环境、单机多容器部署（你现在的微服务整套：mysql redis nacos minio nginx）
❌ **不适合生产集群**：compose只能管理单机容器；多节点集群用 Docker Swarm / K8s

#### 八、示例完整 docker-compose.yml（微服务基础套件，可直接用）

```yaml
services:
  mysql:
    image: mysql:8.0
    container_name: mysql8
    restart: unless-stopped
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: Root@123456
      MYSQL_DATABASE: demo_db
    volumes:
      - ./mysql-data:/var/lib/mysql:z
    networks:
      - app-net

  redis:
    image: redis:7-alpine
    container_name: redis7
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - ./redis-data:/data:z
    networks:
      - app-net

networks:
  app-net:
    driver: bridge
```

如果你需要，我可以单独整理一份**Docker Compose面试题精简版**，或者给你一份包含 Nacos、Seata、MinIO、MySQL、Redis、Nginx 的完整可一键部署的 docker-compose.yml。





---

## 四、 容器网络、端口映射与防火墙机制（含 iptables 避坑）

### （一） 容器端口映射详解（面试与生产实战）

（面试背诵版，适配微服务与容器网络通信场景）

#### 1. 核心概念与工作本质
容器默认拥有独立的网络命名空间（Network Namespace），**外部不能直接访问容器内部 IP 和端口**。
端口映射（`-p`）本质：**在宿主机 Linux 内核 iptables 中添加 DNAT 转发规则**，访问宿主机端口时，流量被自动转发到容器内部端口。

> 语法：`-p [宿主机IP:]宿主机端口:容器端口[/协议]`
> 👉 **顺序：宿主机端口 : 容器端口，千万别写反！面试高频坑**

#### 2. -p 五种常用写法
```bash
# 1. 最常用：宿主机8080 → 容器80（0.0.0.0，所有网卡都监听，公网可访问）
docker run -d -p 8080:80 nginx

# 2. 只绑定本机localhost，外部机器访问不了（安全，仅服务器内部访问）
docker run -d -p 127.0.0.1:8080:80 nginx

# 3. 随机宿主机端口，容器端口固定
docker run -d -p 80 nginx
# 查看自动分配的端口
docker port 容器名/容器ID

# 4. 多端口映射，多个-p
docker run -d -p 8080:80 -p 443:443 nginx

# 5. 指定udp协议（dns等）
docker run -d -p 53:53/udp dns
```

#### 3. 大写 `-P` 和小写 `-p` 的区别（面试必问）
- `-p`（小写 publish）：**手动指定宿主机端口:容器端口**
- `-P`（大写 Publish all）：自动把 Dockerfile 里 `EXPOSE` 声明的所有容器端口，**随机映射到宿主机高位端口（32768起）**

> ⚠️ `EXPOSE 8080` 在 Dockerfile 里**只是元数据注释声明，不会真正开放端口**！
> EXPOSE 只是告诉别人镜像会用到 8080，**不做端口转发，外部无法访问**；只有 `-p / -P` 才会创建 iptables 转发规则。

#### 4. docker-compose.yml 端口写法（微服务项目实操）
```yaml
services:
  gateway:
    image: gateway:1.0
    ports:
      # 方式1：常用暴露
      - "8080:8080"
      # 方式2：仅本机访问
      # - "127.0.0.1:8080:8080"
  vue-nginx:
    image: vue-nginx:1.0
    ports:
      - "80:80"
  service-a:
    # ❌ 内部业务微服务不需要写 ports！
    # 网关 gateway 访问 service-a，同一 docker 网络直接通过 service-a:8081 通信，无需端口映射更安全
```

#### 5. 查看端口映射命令
```bash
# 查看容器端口映射
docker port 容器名称/ID

# 详细信息
docker inspect 容器名
```

#### 6. 底层原理（面试口述）
Docker 启动容器，在宿主机 **iptables 添加 DNAT 转发规则**：
`访问宿主机 IP:宿主机端口 → NAT 转发 → 容器独立 IP:容器内部端口`。
> 💡 **微服务通信关键**：同 docker 自定义网络下，**容器之间互相访问不需要端口映射！**
> 比如 gateway 访问 nacos，直接用容器名+内部端口 `nacos:8848`，不用做 -p 映射。只有**外部（浏览器/宿主机以外机器）访问容器才需要端口映射**。

#### 7. 高频坑点（微服务部署避坑）
1. **端口写反**：`-p 80:8080` 宿主机 80 转发到容器 8080，不要搞反。
2. **宿主机端口占用**：宿主机该端口已经被 Nginx/其他程序占用，容器启动失败。
3. **防火墙/安全组**：端口映射成功，但是云服务器安全组、firewalld 没开放端口，外部访问超时。
4. **容器内部程序监听地址**：容器内服务必须监听 `0.0.0.0`，不能只监听 127.0.0.1，否则端口映射后外部连不上。
> 例如 Spring Boot，配置 `server.address=0.0.0.0`，否则容器内部只能本地访问。

#### 8. 面试简答
* **Q：容器之间通信需要端口映射吗？**
  
  > 不需要。在同一个 docker 自定义网络内，容器之间直接通过容器名+内部端口通信，不需要 -p 端口映射。端口映射是给**宿主机外部**访问容器用的。
* **Q：EXPOSE 和 -p 的区别？**
  
  > EXPOSE 只是镜像元数据，仅文档说明，**不产生任何网络转发**；-p 是真正创建 NAT 端口转发，对外开放访问。
* **Q：为什么 Spring Boot 容器启动，端口映射后访问不通？**
  
  > 大概率是 Spring Boot 只监听 127.0.0.1，容器内 127.0.0.1 是容器自身回环，外部无法转发，需要配置监听 0.0.0.0。

---

### （二） Docker 与 firewalld 防火墙冲突及失效排查

在 CentOS 7 虚拟机上，**Docker 会自动修改 iptables 规则**，导致 `firewalld` 防火墙对容器端口的限制失效。下面是完整的配置方案，让你能正常控制 Docker 端口访问。

---

### 一、核心问题：Docker 与 firewalld 的冲突
- **Docker 默认行为**：启动容器时，自动在 iptables 的 `DOCKER` 和 `DOCKER-USER` 链添加规则，**绕过 firewalld**。
- **后果**：即使防火墙开启，外部仍能直接访问容器映射端口。

---

### 二、方案1：关闭 firewalld（简单但不安全）
适合测试环境，不建议生产使用。
```bash
# 停止并禁用防火墙
systemctl stop firewalld
systemctl disable firewalld

# 重启 Docker（可选）
systemctl restart docker
```

---

### 三、方案2：让 firewalld 控制 Docker（推荐）
#### 1. 修改 Docker 配置，禁用自动 iptables
```bash
# 编辑配置文件
vi /etc/docker/daemon.json

# 添加/修改内容（如果文件为空，直接复制）
{
  "iptables": false
}

# 重启 Docker
systemctl restart docker
```
- **作用**：Docker 不再自动修改 iptables，交由 firewalld 管理。

#### 2. 开放 Docker 端口（firewalld 规则）
```bash
# 开放单个端口（如 8080）
firewall-cmd --permanent --add-port=8080/tcp

# 开放端口范围（如 3000-3010）
firewall-cmd --permanent --add-port=3000-3010/tcp

# 重载生效
firewall-cmd --reload

# 查看已开放端口
firewall-cmd --list-ports
```

#### 3. 允许 Docker 网桥（docker0）通信
```bash
# 将 docker0 加入信任区域
firewall-cmd --permanent --zone=trusted --change-interface=docker0

# 重载生效
firewall-cmd --reload
```

#### 4. 限制特定 IP 访问（白名单）
```bash
# 允许 192.168.1.100 访问 8080 端口
firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='192.168.1.100' port protocol='tcp' port='8080' accept"

# 重载生效
firewall-cmd --reload
```

---

### 四、方案3：直接操作 iptables（进阶）
适合需要精细控制的场景。
```bash
# 查看 Docker 相关规则
iptables -L DOCKER-USER -n -v

# 清空并重建 DOCKER-USER 链（需停止 Docker）
systemctl stop docker
firewall-cmd --permanent --direct --remove-chain ipv4 filter DOCKER-USER
firewall-cmd --permanent --direct --add-chain ipv4 filter DOCKER-USER

# 设置默认拒绝
firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 0 -j DROP

# 允许特定 IP 访问 3000 端口
firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 1 -o docker0 -p tcp --dport 3000 -s 192.168.1.100 -j ACCEPT

# 重载并启动 Docker
firewall-cmd --reload
systemctl start docker
```

---

### 五、常用命令汇总
```bash
# 防火墙状态
firewall-cmd --state

# 查看所有规则
firewall-cmd --list-all

# 开放端口
firewall-cmd --add-port=端口/协议 --permanent

# 删除端口
firewall-cmd --remove-port=端口/协议 --permanent

# 重载生效
firewall-cmd --reload

# 查看 Docker 网络
docker network inspect bridge
```

---

### 六、总结
- **测试环境**：直接关闭 firewalld。
- **生产环境**：**方案2最佳**：
  1. `daemon.json` 设置 `"iptables": false`。
  2. firewalld 开放端口。
  3. docker0 加入 trusted 区域。

### 七、示例

你这句 **Docker 启动 Seata 命令**，明明 CentOS7 开着防火墙（firewalld），**为什么外部还能直接访问 8091 端口？**

```
docker run -d \
  --name seata-theater \
  --restart always \
  --network net-jc-club \
  -p 8091:8091 \
  -e SEATA_IP=192.168.137.128 \
  -e SEATA_PORT=8091 \
  -e SEATA_REGISTRY_TYPE=nacos \
  -e SEATA_REGISTRY_NACOS_SERVER_ADDR=nacos-jc-club:8848 \
  -e SEATA_REGISTRY_NACOS_NAMESPACE= \
  -e SEATA_REGISTRY_NACOS_GROUP=DEFAULT_GROUP \
  -e SEATA_REGISTRY_NACOS_USERNAME=nacos \
  -e SEATA_REGISTRY_NACOS_PASSWORD=nacos \
  -e SEATA_REGISTRY_NACOS_APPLICATION=seata-server \
  seataio/seata-server:1.5.2
```



#### 🔥 核心原因

**因为 Docker 默认会绕过 firewalld，直接修改 iptables 放行端口！**

你虽然运行了：
```
-p 8091:8091
```
Docker 就会**自动在 iptables 里加一条放行规则**，这条规则**优先级比 firewalld 更高**。

所以：
✅ firewalld 没开 8091
✅ 但外部依然能访问 8091
✅ 因为 Docker 直接开了后门

---

#### 🧾 给你看证据（你可以直接执行）

```bash
iptables -t nat -L DOCKER -n
```
你会看到一条：
```
ACCEPT     tcp  --  0.0.0.0/0            172.18.0.3           tcp dpt:8091
```
这就是 Docker 自动加的，**防火墙拦不住**。

---

#### ✅ 怎么让防火墙真正控制 8091？

只需要两步：

##### 1. 让 Docker 不再自动修改 iptables

```bash
vi /etc/docker/daemon.json
```
写入：
```json
{
  "iptables": false
}
```

##### 2. 重启 Docker

```bash
systemctl daemon-reload
systemctl restart docker
```

##### 3. 防火墙放行 8091（需要才放）

```bash
firewall-cmd --permanent --add-port=8091/tcp
firewall-cmd --reload
```

---

#### 🎯 最终结论

**你现在能访问 8091，不是防火墙没生效，是 Docker 直接绕过了防火墙。**

只要加上 `"iptables": false`，防火墙就能真正管住 Docker 端口了。

---

需要我帮你把**一键修复命令**发给你吗？复制粘贴就能解决。

这里给你整理一份**最常用、最实用的 Docker 命令清单**，按场景分类，新手直接照着用就行！

## 五、 Docker 常用命令清单

#### 一、镜像相关（下载、查看、删除）

```bash
# 1. 从 Docker Hub 下载镜像
docker pull 镜像名:标签
# 例：docker pull nginx
# 例：docker pull ubuntu:22.04

# 2. 查看本地所有镜像
docker images

# 3. 删除本地镜像
docker rmi 镜像ID/镜像名
docker rm -f seata-server
# 例：docker rmi nginx

# 4. 给镜像打标签（改名）
docker tag 原镜像 新镜像:新标签
# 例：docker tag nginx nginx:v1

# 一键删除所有 theater 项目容器
docker rm -f $(docker ps -a | grep theater | awk '{print $1}')

# 执行完查看确认：
docker ps -a | grep theater

# 顺带清理无用镜像（可选，节省磁盘）,只删带 theater-project 前缀的镜像
docker rmi -f $(docker images | grep 'theater-project' | awk '{print $2}')

# 一行快速查看所有正在运行容器 + 所属网络
for c in $(docker ps --format {{.Names}});do echo -n "$c ===> ";docker inspect $c | jq -r '.[0].NetworkSettings.Networks | keys | join(",")';done



docker run -d \
--name seata-server \
--network net-common \
-p 7091:7091 \
-p 8091:8091 \
-e SEATA_PORT=8091 \
-e STORE_MODE=db \
-e STORE_DB_DATASOURCE_TYPE=mysql \
-e STORE_DB_HOST=mysql \
-e STORE_DB_PORT=3306 \
-e STORE_DB_DATABASE=seata \
-e STORE_DB_USER=root \
-e STORE_DB_PASSWORD="123456" \
-e SEATA_REGISTRY_TYPE=nacos \
-e SEATA_NACOS_SERVER_ADDR=nacos:8848 \
-e SEATA_NACOS_APPLICATION=seata-server \
-e SEATA_NACOS_GROUP=DEFAULT_GROUP \
seataio/seata-server:1.5.2
```

---

#### 二、容器相关（启动、停止、查看、进入）

```bash
# 1. 运行容器（最常用）
docker run -d --name 自定义名称 -p 主机端口:容器端口 镜像名
# -d 后台运行
# --name 给容器起名字
# -p 端口映射

# 示例：启动 nginx
docker run -d --name my-nginx -p 80:80 nginx

# 2. 查看正在运行的容器
docker ps

# 3. 查看所有容器（包括停止的）
docker ps -a

# 4. 停止容器
docker stop 容器名/容器ID

# 5. 启动已停止的容器
docker start 容器名/容器ID

# 6. 重启容器
docker restart 容器名/容器ID

# 7. 删除容器（必须先停止）
docker rm 容器名/容器ID

# 8. 强制删除正在运行的容器
docker rm -f 容器名/容器ID

# 9. 进入容器内部（操作命令行）
docker exec -it 容器名/容器ID /bin/bash
# 退出容器：输入 exit
```

---

#### 三、日志与信息

```bash
# 查看容器日志
docker logs 容器名/容器ID

# 实时查看日志
docker logs -f 容器名/容器ID

# 查看容器详细信息
docker inspect 容器名/容器ID
```

---

#### 四、清理（释放空间）

```bash
# 删除所有停止的容器
docker container prune

# 删除所有无用镜像
docker image prune

# 一键清理所有无用资源（镜像、容器、缓存）
docker system prune
```

---

#### 五、最常用的 10 条命令（背会就够用）

1. `docker pull` → 下载镜像
2. `docker images` → 看镜像
3. `docker run` → 启动容器
4. `docker ps` → 看运行中的容器
5. `docker ps -a` → 看所有容器
6. `docker exec -it` → 进入容器
7. `docker stop` → 停止容器
8. `docker rm` → 删除容器
9. `docker rmi` → 删除镜像
10. `docker logs` → 看日志

---

#### 总结

- **镜像**：下载、查看、删除、打标签
- **容器**：运行、停止、进入、删除、看日志
- 日常开发**记住前 10 条命令**就足够用了





## 六、 docker stats 容器性能监控

```bash
docker stats
```
✅ 作用：**实时监控所有正在运行的容器资源，1秒自动刷新一次，持续滚动**，类似Linux的`top`命令。
> 按 `Ctrl + C` 退出监控面板。

#### 常用参数

```bash
# 只看指定容器（你排查seata和pay时用这个最方便）
docker stats theater-service-pay seata-server

# --no-stream：只输出当前快照，一次性打印，不持续刷新（你之前用的）
docker stats --no-stream

# -a / --all：包含已经停止的容器（默认只展示running的）
docker stats -a
```

#### 输出字段回顾（你最关心）

- `CPU %`：容器CPU使用率；**>100% 代表占用多核**，180%≈占用接近2个CPU核心
- `MEM USAGE / LIMIT`：容器当前占用内存 / 内存上限（你没做cgroup限制时上限是宿主机全部内存7.5G）
- `MEM %`：内存占上限百分比
- `NET I/O`：容器累计网络收发流量
- `BLOCK I/O`：磁盘读写
- `PIDS`：容器内部进程数量

#### 两个命令对比

1. `docker stats`：**实时持续刷新**，观察CPU动态变化，看是否持续飙升
2. `docker stats --no-stream`：**快照，一次采样**，适合多次采样对比、脚本采集

#### 配套排错命令（你微服务环境常备）

```bash
# 宿主机整机内存
free -h
# 宿主机CPU
top
# 查看容器日志
docker logs --tail 300 容器名
```

> 现在你的故障关键点：把seata-server配置里的mysql地址从`127.0.0.1`改成`mysql:3306`，改完重启seata-server，再用`docker stats`观察它CPU是否下降。

需要我给你一条一键持续监控seata和pay的命令吗？自动每秒刷新，只看这两个容器。



## **一、Docker 基础概念**

1. **Docker 是什么？它与虚拟机有什么区别？**
   - **思路**：容器是轻量级隔离环境，共享宿主机内核；虚拟机有完整操作系统，资源占用大，启动慢。
2. **Docker 的主要组成部分有哪些？**
   - **思路**：
     - Docker Engine（守护进程 + CLI）
     - Docker 镜像
     - Docker 容器
     - Docker Hub（镜像仓库）
3. **Docker 镜像与容器有什么区别？**
   - **思路**：镜像是静态模板，容器是镜像的运行实例，有可写层。
4. **Docker 的优点有哪些？**
   - **思路**：轻量、快速启动、环境一致性、易于扩展与微服务化。



你梳理的Docker基础概念框架很清晰，覆盖了核心定义和区别。下面我会基于你的思路，用更具体、结构化的方式拆解每个问题，补充关键细节，让答案更完整易懂。


### 1. Docker 是什么？它与虚拟机有什么区别？
#### Docker 定义
Docker 是一个**开源的容器化平台**，通过“容器”技术将应用及其依赖（如库、配置文件）打包成标准化单元（镜像），实现“一次构建，到处运行”，确保环境一致性和快速部署。

#### 与虚拟机（VM）的核心区别
两者本质都是“隔离环境”，但底层架构不同，导致资源占用、启动速度等差异显著：

| 对比维度       | Docker 容器                          | 虚拟机（VM）                          |
|----------------|--------------------------------------|---------------------------------------|
| 底层架构       | 共享宿主机操作系统内核，仅隔离应用进程 | 包含完整操作系统（内核+用户空间），完全隔离 |
| 资源占用       | 轻量：仅占用应用运行所需资源（MB级）  | 重量级：需占用完整OS资源（GB级）       |
| 启动速度       | 极快（秒级，如1-3秒）                | 较慢（分钟级，如30秒-2分钟）           |
| 镜像大小       | 小巧（基础镜像仅几十MB）              | 庞大（基础OS镜像数GB）                 |
| 隔离级别       | 进程级隔离（隔离性较弱）              | 系统级隔离（隔离性强）                 |
| 兼容性         | 依赖宿主机内核，跨OS（如Linux→Windows）兼容性有限 | 完全独立，跨OS兼容性强                 |


### 2. Docker 的主要组成部分有哪些？
Docker 生态核心由4个部分构成，各组件分工明确，协同实现容器化流程：

#### （1）Docker Engine（核心引擎）
Docker 的运行核心，包含3个组件：
- **Docker Daemon（守护进程）**：后台运行的服务，负责管理镜像、容器的生命周期（如创建、启动、停止容器）。
- **Docker CLI（命令行工具）**：用户与 Daemon 交互的入口，通过命令（如`docker run`、`docker pull`）发送指令给 Daemon 执行。
- **REST API**：Daemon 对外提供的接口，CLI 或第三方工具（如 Docker Compose）通过 API 与 Daemon 通信。

#### （2）Docker 镜像（Image）
- 定义：容器的“静态模板”，包含运行应用所需的代码、依赖、配置、环境变量等（如`nginx:latest`镜像包含 Nginx 服务及基础运行环境）。
- 特性：只读（Read-Only），基于分层文件系统（Layered FS）构建，分层可复用（如多个镜像可共享基础层，减少存储空间）。

#### （3）Docker 容器（Container）
- 定义：镜像的“运行实例”，是动态的、可读写的环境（在镜像只读层之上添加一层可写层）。
- 特性：独立运行的进程，拥有自己的网络、文件系统、进程空间，可通过`docker start/stop`等命令管理。

#### （4）Docker 仓库（Repository）
- 定义：存储 Docker 镜像的远程仓库，类似“代码仓库”，用于镜像的共享和分发。
- 分类：
  - **公共仓库**：如 Docker Hub（官方仓库，包含大量开源镜像，如`ubuntu`、`mysql`）。
  - **私有仓库**：企业或个人搭建的内部仓库（如 Harbor），用于存储私有镜像，保障安全性。


### 3. Docker 镜像与容器有什么区别？
镜像和容器是 Docker 中“静态”与“动态”的核心对应关系，关键区别如下：

| 对比维度       | Docker 镜像（Image）                 | Docker 容器（Container）              |
|----------------|--------------------------------------|---------------------------------------|
| 状态           | 静态（模板），无运行状态             | 动态（实例），有运行/停止/暂停等状态   |
| 可写性         | 只读（所有层均为 Read-Only）         | 可读写（在镜像层之上添加“可写层”，修改仅保存在可写层） |
| 生命周期       | 创建后长期存在，可重复使用           | 随启动/停止变化，删除后可写层数据丢失（除非挂载数据卷） |
| 核心作用       | 用于打包应用和依赖，作为分发单元     | 用于运行应用，是实际的执行环境         |
| 命令关联       | 通过`docker build/pull`创建/获取     | 通过`docker run`从镜像创建，`docker rm`删除 |

**简单类比**：镜像 = 安装包（.exe 文件），容器 = 安装后运行的程序（进程）。


### 4. Docker 的优点有哪些？
Docker 能成为主流容器化工具，核心源于以下5个关键优势：

1. **环境一致性，解决“开发-生产不一致”问题**  
   开发环境打包的镜像，可直接在测试、生产环境运行，避免因“本地能跑，线上报错”（如依赖版本、配置差异）导致的问题。

2. **轻量高效，资源占用低**  
   容器共享宿主机内核，无需额外占用完整 OS 资源，相比虚拟机，启动速度快10倍以上，相同硬件可部署更多容器，提升资源利用率。

3. **快速部署与扩展，适配微服务**  
   基于镜像创建容器仅需秒级，配合编排工具（如 Kubernetes）可实现容器的自动扩缩容（如流量高峰时增加容器实例），适合微服务架构的动态部署需求。

4. **隔离性好，进程间互不干扰**  
   每个容器有独立的网络、文件系统和进程空间，应用运行时互不影响（如一个容器的崩溃不会导致其他容器故障），同时隔离性低于虚拟机，兼顾安全与效率。

5. **易于版本管理与回滚**  
   镜像支持版本标签（如`nginx:1.21`、`nginx:1.22`），可快速切换不同版本的应用；若新版本出现问题，只需基于旧版本镜像重新创建容器，实现秒级回滚。

为了帮你更直观理解镜像与容器的关系，要不要我帮你整理一份**Docker 基础命令实战清单**？里面会包含“拉取镜像→创建容器→启动/停止容器→删除镜像”的完整命令示例，你可以直接在终端实操练习。

### **Docker 与 Java 开发相关的关系图示**

明白了，你想要一张 **Docker 与 Java 开发相关的关系图示**，方便面试时解释 Docker 在 Java 后端开发中的使用场景、组件和关系。我可以帮你生成一个结构清晰的图示。下面是概念设计：

------

#### Docker 与 Java 后端关系示意图（文字版 Mermaid 示例）

```mermaid
graph TD
    %% 基础层
    A["宿主机 OS (Linux/Windows/Mac)"] --> B["Docker Engine"]
    
    %% Docker 核心
    B --> C["Docker Daemon"]
    B --> D["Docker CLI"]
    C --> E["Docker Images"]
    C --> F["Docker Containers"]

    %% 镜像层
    E --> G["基础镜像 (OpenJDK / Alpine / Ubuntu)"]
    G --> H["Java 应用 Jar / War"]
    
    %% 容器层
    F --> I["运行中的 Java 容器"]
    I --> J["端口映射 (如 8080)"]
    I --> K["挂载卷 (Volumes / Config)"]
    I --> L["网络 (Bridge / Host / Overlay)"]
    
    %% 容器与数据库 / 服务交互
    I --> M["数据库容器 (MySQL / MongoDB)"]
    I --> N["消息队列容器 (RabbitMQ / Kafka)"]
    
    %% 构建与部署
    H --> O["Dockerfile 构建镜像"]
    O --> E
```

------

#### 说明：

1. **宿主机 OS**：Docker 依赖操作系统提供内核功能（如 Linux 的 cgroups 和 namespace）。
2. **Docker Engine**：核心，包含 Docker Daemon（后台服务）和 CLI（命令行交互）。
3. **镜像**：静态模板，包含 OpenJDK 等基础环境和 Java 应用。
4. **容器**：镜像的运行实例，隔离运行环境、端口、卷和网络。
5. **数据库与消息队列**：常见的 Java 后端服务依赖，可以容器化部署。
6. **构建流程**：使用 Dockerfile 构建镜像，生成可移植的应用部署单元。

------

如果你愿意，我可以帮你生成一个 **更美观的图形化版本**，带图标风格和层级感，适合直接打印或面试展示。

你希望我帮你生成这个可视化版本吗？

------

## **二、Docker 镜像管理**

1. **Dockerfile 常用指令有哪些？**

   - **思路**：FROM、RUN、COPY、ADD、WORKDIR、ENV、EXPOSE、CMD、ENTRYPOINT。

2. **如何优化 Docker 镜像体积？**

   - **思路**：使用精简基础镜像（alpine）、多阶段构建、合并 RUN 命令、删除临时文件、使用 `.dockerignore`。

3. **如何查看、删除 Docker 镜像？**

   - **思路**：

     ```bash
     docker images       # 查看镜像
     docker rmi <image>  # 删除镜像
     ```



你整理的Docker镜像管理思路很精准，这三个问题是镜像操作的核心。下面我会基于你的你的思路，补充每个指令的细节用法、优化技巧的原理，以及实际操作中的注意事项，让内容更具实操性。


### 1. Dockerfile 常用指令有哪些？
Dockerfile 是构建镜像的“脚本文件”，通过指令指令定义镜像的构建步骤。核心指令及用法如下：

| 指令         | 作用                                                                 | 示例                                                                 |
|--------------|----------------------------------------------------------------------|----------------------------------------------------------------------|
| `FROM`       | 指定基础镜像（必须是 Dockerfile 首行，除了注释）                     | `FROM ubuntu:22.04`（基于 Ubuntu 22.04 构建）<br>`FROM openjdk:8-jre-alpine`（基于轻量 Java 镜像） |
| `RUN`        | 执行命令，用于安装依赖、配置环境（每行会生成一个新的镜像层）         | `RUN apt-get update && apt-get install -y nginx`（合并命令减少层数） |
| `COPY`       | 从宿主机复制文件/目录到镜像中（仅复制，不解压）                       | `COPY app.jar /app/`（复制本地 app.jar 到镜像的 /app 目录）          |
| `ADD`        | 类似 `COPY`，但支持自动解压压缩包（如 .tar.gz）和 URL 下载（不推荐） | `ADD app.tar.gz /app/`（复制并解压 app.tar.gz 到 /app）              |
| `WORKDIR`    | 设置后续命令的工作目录（类似 `cd`，推荐使用绝对路径）                 | `WORKDIR /app`（后续命令默认在 /app 目录执行）                       |
| `ENV`        | 定义环境变量（构建时和容器运行时均有效）                             | `ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk`                         |
| `EXPOSE`     | 声明容器运行时监听的端口（仅作文档说明，不实际映射）                 | `EXPOSE 8080`（声明容器会监听 8080 端口）                            |
| `CMD`        | 定义容器启动时执行的命令（可被 `docker run` 后的命令覆盖）           | `CMD ["java", "-jar", "app.jar"]`（启动 Java 应用）                  |
| `ENTRYPOINT` | 定义容器启动时的“入口命令”（不可被覆盖，`docker run` 后的参数作为其参数） | `ENTRYPOINT ["java", "-jar"]`，启动时执行 `docker run app.jar` 等价于 `java -jar app.jar` |

**关键区别**：  
- `CMD` 和 `ENTRYPOINT` 都用于定义启动命令，推荐组合使用：`ENTRYPOINT` 写固定命令，`CMD` 写默认参数（如 `ENTRYPOINT ["echo"]; CMD ["hello"]`，`docker run` 不加参数输出 hello，加参数则替换 CMD）。  
- `COPY` 和 `ADD`：优先用 `COPY`（更明确），仅在需要解压时用 `ADD`。


### 2. 如何优化 Docker 镜像体积？
镜像体积过大会导致存储占用高、传输慢、部署耗时，优化核心是“减少镜像层数”和“删除冗余文件”，具体方法如下：

#### （1）使用精简基础镜像
- 优先选择 `alpine` 版本（基于 Alpine Linux，体积仅几 MB），如：  
  - 替代 `ubuntu:latest`（≈700MB）→ `alpine:latest`（≈5MB）  
  - 替代 `openjdk:8`（≈600MB）→ `openjdk:8-jre-alpine`（≈80MB）  
- 对 Java 应用，可使用 `distroless` 镜像（仅包含运行时依赖，无 shell，更安全）。

#### （2）多阶段构建（核心技巧）
将构建过程拆分为“构建阶段”和“运行阶段”，仅将运行所需文件复制到最终镜像，丢弃构建工具（如编译器、Maven）。  
**示例（Java 应用）**：  
```dockerfile
# 第一阶段：构建应用（使用带 Maven 的镜像）
FROM maven:3.8-openjdk-8 AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests  # 编译打包，生成 target/app.jar

# 第二阶段：运行应用（仅保留 JRE 和 jar 包）
FROM openjdk:8-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/app.jar .  # 仅复制构建产物
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```
**效果**：最终镜像仅包含 JRE 和 app.jar，体积从数百 MB 降至 100MB 以内。

#### （3）合并 `RUN` 命令，清理缓存
- 用 `&&` 合并多个命令，减少镜像层数（每层都会占用空间）。  
- 命令执行后立即删除临时文件（如 apt 缓存、编译中间文件）。  
**示例**：  
```dockerfile
# 不好的写法（多图层，未清理缓存）
RUN apt-get update
RUN apt-get install -y nginx
RUN rm -rf /var/lib/apt/lists/*

# 优化写法（单图层，即时清理）
RUN apt-get update && \
    apt-get install -y nginx && \
    rm -rf /var/lib/apt/lists/*  # 删除 apt 缓存，减少体积
```

#### （4）使用 `.dockerignore` 文件
排除不需要打包到镜像的文件（如本地日志、IDE 配置、`node_modules`），避免镜像包含冗余内容。  
**示例（.dockerignore）**：  
```
.git          # 排除 git 版本控制文件
logs/         # 排除日志目录
target/*.jar  # 若用多阶段构建，排除本地构建的 jar（避免覆盖构建阶段的产物）
```

#### （5）其他技巧
- 避免在镜像中存储敏感信息（如密码、密钥），通过环境变量或外部挂载传入。  
- 对前端应用，可使用 `nginx:alpine` 作为基础镜像，仅复制打包后的 `dist` 目录。


### 3. 如何查看、删除 Docker 镜像？
#### 查看镜像
```bash
# 查看所有本地镜像（包含标签、ID、大小）
docker images
# 或简写
docker image ls

# 查看指定镜像（如 nginx 相关）
docker images nginx

# 查看镜像的详细信息（包括分层、创建时间等）
docker inspect <镜像ID或镜像名:标签>
# 示例：docker inspect nginx:latest
```

#### 删除镜像
```bash
# 删除指定镜像（需先停止并删除依赖该镜像的容器）
docker rmi <镜像ID或镜像名:标签>
# 示例：docker rmi nginx:latest 或 docker rmi a1b2c3d4e5f6

# 强制删除（即使有容器依赖，不推荐，可能导致残留）
docker rmi -f <镜像ID>

# 删除所有未被使用的镜像（无标签的镜像，即 "none" 镜像）
docker image prune

# 删除所有不被容器使用的镜像（包括有标签但未被引用的）
docker image prune -a
```

**注意事项**：  
- 镜像被容器引用时（即使容器未运行），无法直接删除，需先删除容器（`docker rm <容器ID>`）。  
- 镜像 ID 可只写前3-4位（唯一即可），如 `docker rmi a1b2` 等价于 `docker rmi a1b2c3d4e5f6`。

### Docker Tag

#### Docker Tag 是什么？

简单说：**Docker Tag 就是 Docker 镜像的「版本号 + 别名」**，用来给镜像打标签、区分版本、重命名、标记镜像。

#### 核心作用

1. **区分镜像版本**  
   同一个镜像（比如 `nginx`），可以有 `latest`、`1.25`、`alpine` 等不同标签，代表不同版本。
2. **给镜像重命名**  
   把本地镜像改名，方便推送到私有仓库/远程仓库。
3. **标记镜像**  
   给同一个镜像打多个标签（一个镜像可以对应多个 tag）。

---

#### 标准格式

```
[仓库地址/]镜像名:标签
```

- **仓库地址**：可选，比如 `docker.io`、`harbor.xxx.com`
- **镜像名**：比如 `nginx`、`ubuntu`、`my-app`
- **标签**：版本号/标识，比如 `latest`、`v1.0`、`stable`

最常见的默认标签是 **`latest`**（最新版）。

---

#### 常用命令

##### 1. 查看镜像的 tag

```bash
docker images
```
输出里的 **TAG 列**就是标签：
```
REPOSITORY   TAG       IMAGE ID
nginx        latest    abc123
nginx        1.25      def456
```

##### 2. 给镜像打标签（重命名）

```bash
docker tag 原镜像:原标签 新镜像:新标签
```

示例：
```bash
# 给 nginx:latest 打一个新标签 v1.0
docker tag nginx:latest nginx:v1.0

# 重命名镜像，准备推送到私有仓库
docker tag my-app:latest harbor.xxx.com/my-app:v2.1
```

---

#### 一句话总结

**Tag = 镜像的版本号/别名**，用来区分、命名、管理 Docker 镜像。

#### 总结

1. Docker Tag 是镜像的**版本标识 + 别名**
2. 格式：`镜像名:标签`
3. 常用命令：`docker tag` 打标签、`docker images` 查看标签
4. 一个镜像可以拥有**多个不同标签**

------

## **三、Docker 容器管理**

1. **如何创建和启动容器？**

   - **思路**：

     ```bash
     docker run -d --name my_container image_name
     docker start/stop/restart <container>
     ```

2. **如何查看容器状态和日志？**

   - **思路**：

     ```bash
     docker ps            # 查看运行容器
     docker ps -a         # 查看所有容器
     docker logs <container>  # 查看日志
     ```

3. **如何进入运行中的容器进行调试？**

   - **思路**：

     ```bash
     docker exec -it <container> /bin/bash
     docker attach <container>
     ```

4. **容器的生命周期包括哪些阶段？**

   - **思路**：创建（created）、运行（running）、暂停（paused）、停止（stopped）、退出（exited）、删除（removed）。



你总结的容器管理核心命令和生命周期阶段非常准确，这些是日常操作容器的基础。下面我会基于你的思路，补充每个操作的细节、场景差异及注意事项，让命令使用更精准。


### 1. 如何创建和启动容器？
`docker run` 是创建并启动容器的核心命令，支持丰富的参数配置，满足不同场景需求。

#### 基础命令
```bash
# 创建并启动容器（后台运行，指定名称和镜像）
docker run -d --name my_container nginx:latest
# -d：后台运行（detach）
# --name：指定容器名称（不指定则自动生成）
# nginx:latest：基于该镜像创建容器
```

#### 常用参数（扩展场景）
| 参数         | 作用                                                                 | 示例                                                                 |
|--------------|----------------------------------------------------------------------|----------------------------------------------------------------------|
| `-p`         | 端口映射（宿主机端口:容器端口），让外部可访问容器服务                 | `docker run -d -p 8080:80 nginx`（宿主机8080端口映射到容器80端口）  |
| `-v`         | 数据卷挂载（宿主机路径:容器路径），持久化数据（容器删除后数据不丢失） | `docker run -d -v /host/data:/container/data mysql`（挂载数据目录）   |
| `--env`/`-e` | 设置环境变量                                                         | `docker run -d -e "SPRING_PROFILES_ACTIVE=prod" app:latest`          |
| `--rm`       | 容器停止后自动删除（适合临时任务，如一次性脚本执行）                 | `docker run --rm ubuntu:latest echo "hello"`                        |
| `--network`  | 指定容器所属网络（默认使用 bridge 网络，可自定义网络实现容器互通）   | `docker run -d --network my_net --name app1 app:v1`                  |

#### 启动/停止/重启已有容器
```bash
# 启动已停止的容器
docker start my_container  # 按名称启动
docker start 1a2b3c        # 按容器ID启动（前几位即可）

# 停止运行中的容器
docker stop my_container

# 重启容器（先停止再启动）
docker restart my_container
```


### 2. 如何查看容器状态和日志？
#### 查看容器状态
```bash
# 查看正在运行的容器（默认）
docker ps
# 输出字段：容器ID、镜像、启动命令、创建时间、状态、端口映射、名称

# 查看所有容器（包括停止的、暂停的）
docker ps -a

# 查看容器的详细信息（配置、网络、挂载等）
docker inspect my_container  # 按名称
docker inspect 1a2b3c        # 按ID
```

#### 查看容器日志
```bash
# 查看容器实时日志（默认最后10行）
docker logs my_container

# 实时跟踪日志（类似 tail -f）
docker logs -f my_container

# 查看指定行数的日志（如最后100行）
docker logs -n 100 my_container

# 查看最近30分钟的日志
docker logs --since 30m my_container

# 显示日志时包含时间戳
docker logs -t my_container
```

**注意**：若容器未启动（如启动失败），`docker logs` 仍能查看启动过程的日志，帮助排查失败原因（如端口被占用、配置错误）。


### 3. 如何进入运行中的容器进行调试？
进入容器主要用于临时查看文件、执行命令或调试问题，常用两种方式：`docker exec` 和 `docker attach`，适用场景不同。

#### 方式1：`docker exec`（推荐）
在容器中**新启动一个进程**（如 bash），不影响容器主进程，退出后容器仍正常运行。
```bash
# 进入容器并启动交互式终端（最常用）
docker exec -it my_container /bin/bash
# -i：保持标准输入打开
# -t：分配伪终端（终端交互）
# /bin/bash：启动 bash shell（若容器没有 bash，可用 /bin/sh）

# 示例：进入 alpine 容器（默认无 bash，用 sh）
docker exec -it alpine_container /bin/sh
```

#### 方式2：`docker attach`（慎用）
直接**连接到容器的主进程**（如 nginx 的主进程），退出时可能导致容器停止（按 `Ctrl+C` 会终止主进程）。
```bash
docker attach my_container
# 退出需按 Ctrl+P+Q（不终止容器），若按 Ctrl+C 会停止容器
```

**最佳实践**：优先用 `docker exec -it`，安全且灵活；`docker attach` 仅用于调试主进程输出（如查看启动日志）。


### 4. 容器的生命周期包括哪些阶段？
容器从创建到删除会经历6个核心阶段，状态可通过 `docker ps -a` 查看：

1. **创建（Created）**  
   - 触发：`docker create` 命令（仅创建容器，不启动）或 `docker run` 初始化未完成时。  
   - 状态：容器已初始化文件系统和配置，但未运行任何进程。  

2. **运行（Running）**  
   - 触发：`docker start` 启动容器，或 `docker run` 直接创建并启动。  
   - 状态：容器主进程（如 nginx、java）正在运行，可对外提供服务。  

3. **暂停（Paused）**  
   - 触发：`docker pause <容器>` 命令，将容器进程冻结（保留内存状态）。  
   - 状态：进程暂停，CPU 占用为0，可通过 `docker unpause` 恢复运行。  

4. **停止（Stopped）**  
   - 触发：`docker stop` 命令（优雅终止进程）或 `docker kill` 命令（强制终止）。  
   - 状态：进程已终止，容器保留文件系统和配置，可通过 `docker start` 重新启动。  

5. **退出（Exited）**  
   - 触发：容器主进程自然结束（如一次性任务执行完成）或异常崩溃。  
   - 状态：同 Stopped，区别在于 Exited 通常是进程主动退出（有退出码，可通过 `docker logs` 查看原因）。  

6. **删除（Removed）**  
   - 触发：`docker rm <容器>` 命令，删除已停止的容器。  
   - 状态：容器彻底从系统中移除，文件系统（未挂载数据卷的部分）被删除。  


**生命周期流转示例**：  
`Created` →（`docker start`）→ `Running` →（`docker stop`）→ `Stopped` →（`docker rm`）→ `Removed`  

如果需要练习容器管理的完整流程（从创建到删除，含数据持久化），我可以提供一个“Nginx 容器实战示例”，包含端口映射、日志查看、进入调试和数据卷挂载的具体操作~

------

## **四、Docker 网络与存储**

1. **Docker 网络模式有哪些？**

   - **思路**：bridge、host、none、overlay。

2. **容器如何进行端口映射？**

   - **思路**：

     ```bash
     docker run -p 宿主机端口:容器端口 image_name
     ```

3. **Docker 数据卷（Volume）和绑定挂载（Bind Mount）区别？**

   - **思路**：Volume 由 Docker 管理，跨容器共享；Bind Mount 依赖宿主机路径，更灵活但受宿主环境影响。

你梳理的Docker网络与存储核心点非常清晰，这两块是容器持久化和跨容器通信的关键。下面我会基于你的思路，补充每种网络模式的适用场景、端口映射的细节，以及数据卷与绑定挂载的底层差异和最佳实践。


### 1. Docker 网络模式有哪些？

Docker 提供4种核心网络模式，默认使用 `bridge`，不同模式适用于不同的通信场景，底层隔离和配置方式差异显著：

| 网络模式                | 核心特点                                                     | 适用场景                                                     |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **bridge（桥接模式）**  | Docker 默认网络模式，容器通过虚拟网桥（docker0）与宿主机及其他容器通信；每个容器有独立IP，与宿主机通过NAT转换通信。 | 最常用，适合容器间需要通信（如前端容器调用后端容器），但需隔离宿主机网络的场景。 |
| **host（主机模式）**    | 容器直接使用宿主机的网络栈（IP、端口），无网络隔离；容器内的端口直接绑定到宿主机端口。 | 追求网络性能（无NAT开销），或需要容器使用宿主机特定网络配置（如绑定特权端口）的场景（如DNS服务、监控代理）。 |
| **none（无网络模式）**  | 容器无网络接口，完全隔离网络；仅能通过 `docker exec` 本地访问。 | 不需要网络的离线任务（如本地数据处理、日志分析）。           |
| **overlay（覆盖网络）** | 跨宿主机的容器网络（需配合Docker Swarm或Kubernetes），让不同宿主机的容器在同一虚拟网络中通信，仿佛在同一主机。 | 分布式集群场景（如微服务部署在多台服务器，需要跨机通信）。   |

#### 补充：自定义网络（推荐）

除了上述内置模式，还可创建自定义bridge网络，相比默认bridge网络，支持**容器名直接通信**（无需记IP），且隔离性更好：

```bash
# 创建自定义网络
docker network create my_net

# 启动容器时指定网络
docker run -d --name app1 --network my_net nginx
docker run -d --name app2 --network my_net nginx

# 容器间可通过名称通信（如 app1 中 ping app2 会解析到对应IP）
```

#### 容器切换网络

**Docker 容器不能直接在线切换主网络（如 bridge → host）**，但有两种常用方案：**在线加新网络（多网卡）**、**停容器重建（彻底切换）**。

##### 一、先查看网络（必备）

```bash
# 列出所有网络
docker network ls

# 查看容器当前网络
docker inspect <容器名/ID> | grep NetworkMode
```

---

##### 二、方案1：在线添加/切换网络（不停机）

**适用：** 给容器**加一个新网络**（多网卡），保留原网络。

###### 1. 连接到新网络（添加）

```bash
# 格式：docker network connect <网络名> <容器名/ID>
docker network connect my-custom-net seata-server
```

###### 2. 断开旧网络（可选）

```bash
# 格式：docker network disconnect <旧网络名> <容器名/ID>
docker network disconnect bridge seata-server
```

###### 3. 验证

```bash
docker inspect seata-server | grep Network
docker inspect seata-theater | grep Network
```

**效果：** 容器同时在 **bridge + my-custom-net** 两个网络。

---

##### 三、方案2：彻底切换主网络（必须停机）

**适用：** 从 `bridge` → `host` / `none` / 自定义网络。

###### 1. 停止容器

```bash
docker stop seata-server
```

###### 2. 删除旧容器（保留镜像）

```bash
docker rm seata-server
```

###### 3. 重建容器（指定新网络）

```bash
# 示例1：切换到 host 网络
docker run -d \
  --name seata-server \
  --network host \
  -p 8091:8091 \
  seata-server:latest

# 示例2：切换到自定义网络
docker run -d \
  --name seata-server \
  --network my-custom-net \
  -p 8091:8091 \
  seata-server:latest
```

---

##### 四、常见网络模式说明

- **bridge（默认）**：独立IP，NAT转发，适合大多数场景。
- **host**：共享宿主机网络（无端口映射），性能最高。
- **none**：无网络，完全隔离。
- **自定义网络**：可指定网段、网关，容器间通过**服务名**互通。

---

##### 五、你当前问题（Seata 连不上 Nacos）

**建议操作：**
1. 给 `seata-server` **加一个与 Nacos 同网段的自定义网络**
2. 或直接把 `seata-server` 切到 **host 网络**（最简单）

```bash
# 快速切换到 host 网络（解决网络不通）
docker stop seata-server
docker rm seata-server
docker run -d --name seata-server --network host seata-server:latest
```

---

##### 六、总结

- **在线加网络**：`docker network connect`（不停机）
- **彻底切换**：**停→删→重建**（指定 `--network`）
- **Seata 连 Nacos**：推荐切到 **host 网络** 或 **同网段自定义网络**

### 删除网络

#### 一、删除单个网络（你现在需要的）

你要删的是：**net-jc-club、theatre-ticket-network**

##### 1. 先看网络列表（确认名字）

```bash
docker network ls
```

##### 2. 删除网络（必须先断开容器）

**第一步：断开容器（关键！）**

```bash
# 断开 seata 容器与 net-jc-club 的连接
docker network disconnect net-jc-club seata

# 断开 seata 容器与 theatre-ticket-network 的连接
docker network disconnect theatre-ticket-network seata
```

**第二步：删除网络**

```bash
# 删除 net-jc-club
docker network rm net-jc-club

# 删除 theatre-ticket-network
docker network rm theatre-ticket-network
```

---

#### 二、批量清理所有未使用网络（一键清理）

```bash
# 清理所有没被容器使用的网络
docker network prune -f
```

---

#### 三、你现在的操作（直接复制执行）

```bash
# 1. 断开 seata 容器
docker network disconnect net-jc-club seata
docker network disconnect theatre-ticket-network seata

# 2. 删除两个网络
docker network rm net-jc-club
docker network rm theatre-ticket-network

# 3. 验证删除
docker network ls
```

---

#### 四、常见报错：`network has active endpoints`

**原因**：网络还被容器占用
**解决**：
```bash
# 查看哪些容器在用这个网络
docker network inspect net-jc-club

# 强制断开（容器在运行也能断）
docker network disconnect -f net-jc-club seata
```

---

#### 五、安全提醒

- **不要删默认网络**：`bridge、host、none`（删了会出大问题）
- **自定义网络**：删之前**必须断开所有容器**

---

#### 六、你现在做完这些就可以

1. 删除旧网络
2. 按我之前给你的命令重建 **Seata 容器（host网络）**
3. 重启项目，错误消失！

要不要我帮你写一个一键清理+重建Seata的完整脚本？

### 2. 容器如何进行端口映射？

端口映射是让外部（宿主机或其他机器）访问容器服务的核心方式，通过 `-p` 或 `-P` 参数配置，细节如下：

#### 基础语法

```bash
# 格式：宿主机端口:容器端口
docker run -d -p 8080:80 nginx
# 宿主机的8080端口映射到容器的80端口，外部访问 http://宿主机IP:8080 即可访问容器内的Nginx
```

#### 扩展用法

| 场景                 | 命令示例                                       | 说明                                                         |
| -------------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| 绑定到宿主机特定IP   | `docker run -d -p 192.168.1.100:8080:80 nginx` | 仅允许通过宿主机的192.168.1.100IP访问8080端口，增强安全性。  |
| 映射UDP端口          | `docker run -d -p 53:53/udp dns-server`        | 容器内的UDP 53端口（DNS服务）映射到宿主机UDP 53端口。        |
| 随机映射容器暴露端口 | `docker run -d -P nginx`（大写P）              | 自动将容器 `EXPOSE` 声明的端口（如80）映射到宿主机的随机端口（32768+），适合临时测试。 |
| 映射多个端口         | `docker run -d -p 8080:80 -p 8443:443 nginx`   | 同时映射80→8080（HTTP）和443→8443（HTTPS）。                 |

#### 查看端口映射

```bash
# 查看容器的端口映射关系
docker port my_container
# 输出示例：80/tcp -> 0.0.0.0:8080
```

### 3. Docker 数据卷（Volume）和绑定挂载（Bind Mount）区别？

两者都是实现“容器数据持久化”或“宿主机与容器数据共享”的方式，但底层管理和适用场景差异很大：

- 网络模式：默认用 `bridge`，跨机通信用 `overlay`，追求性能用 `host`，推荐创建自定义网络简化容器间通信。  
- 端口映射：通过 `-p` 精确控制，生产环境避免随机端口，确保端口占用可管理。  
- 数据持久化：生产环境优先用 `Volume`（安全、易管理），开发环境用 `Bind Mount`（方便调试）。

如果需要实践“多容器协作（如Nginx+MySQL通过自定义网络通信，数据持久化到Volume）”的完整流程，可以告诉我，我会提供详细步骤~

| 对比维度         | 数据卷（Volume）                                             | 绑定挂载（Bind Mount）                                       |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **管理方式**     | 由 Docker 引擎完全管理，存储路径由 Docker 自动分配（默认在 `/var/lib/docker/volumes/`）。 | 依赖宿主机的文件系统路径，需手动指定宿主机目录（如 `/host/path:/container/path`）。 |
| **跨平台兼容性** | 完全兼容（Docker 负责适配不同OS的存储机制），在Windows、Linux、Mac上行为一致。 | 依赖宿主机路径格式（如Windows的 `C:\` 和Linux的 `/` 不同），跨平台需调整路径。 |
| **数据隔离性**   | 与宿主机文件系统隔离，仅通过 Docker 命令管理，避免误操作删除。 | 宿主机可直接修改挂载的文件/目录，可能因权限、格式问题导致容器异常。 |
| **跨容器共享**   | 支持多个容器挂载同一个Volume，数据自动同步（适合多容器协作，如前端+后端共享静态资源）。 | 可通过共享宿主机路径实现跨容器共享，但需手动保证宿主机路径一致，管理复杂。 |
| **备份与迁移**   | 支持 `docker volume cp` 复制数据，`docker volume export` 导出，迁移方便。 | 需手动备份宿主机路径下的数据，迁移时需确保目标机路径一致。   |
| **适用场景**     | 数据库数据（如MySQL的 `/var/lib/mysql`）、容器间共享数据、需要长期持久化的数据。 | 开发环境（如本地代码目录挂载到容器，实时生效）、临时数据共享、需要宿主机直接访问的场景。 |

#### 实战示例

- **数据卷（Volume）**：

  ```bash
  # 创建数据卷
  docker volume create mysql_data
  # 挂载到容器（MySQL数据持久化）
  docker run -d -v mysql_data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mysql
  ```

- **绑定挂载（Bind Mount）**：

  ```bash
  # 本地代码目录挂载到Nginx容器（开发时实时更新）
  docker run -d -p 8080:80 -v /local/project/html:/usr/share/nginx/html nginx
  ```



**Docker 数据卷（Volume）** 和 **绑定挂载（Bind Mount）** 是两种实现容器数据持久化的核心方式，主要区别在于**管理主体、存储位置、可移植性**和**适用场景**。

#### 一、数据卷 (Volume)

**定义**：由 Docker 统一管理的宿主机文件系统目录，独立于容器生命周期。
**存储位置**：Linux 上默认在 `/var/lib/docker/volumes/`。
**核心特性**：

- **Docker 全权管理**：创建、删除、备份、迁移均通过 `docker volume` 命令。
- **自动创建**：挂载不存在的卷时，Docker 自动创建。
- **高可移植性**：不依赖宿主机路径，跨主机、跨平台（Windows/macOS/Linux）兼容。
- **权限隔离**：Docker 自动处理权限，安全性更高。
- **高性能**：绕过 UnionFS，读写性能接近宿主机文件系统。

**常用命令**：
```bash
# 创建数据卷
docker volume create my-volume

# 查看数据卷
docker volume ls

# 挂载数据卷（--mount 更推荐）
docker run -d --name my-app --mount source=my-volume,target=/app/data nginx
# 或 -v 语法
docker run -d --name my-app -v my-volume:/app/data nginx

# 删除数据卷
docker volume rm my-volume
```

#### 二、绑定挂载 (Bind Mount)

**定义**：将宿主机**任意指定目录/文件**直接映射到容器内。
**存储位置**：用户指定的宿主机路径（如 `/home/user/code`）。
**核心特性**：
- **用户管理**：完全由用户控制路径，Docker 不干预。
- **不自动创建**：宿主机路径不存在时，挂载失败。
- **低可移植性**：强依赖宿主机绝对路径，跨平台/主机兼容性差。
- **权限依赖**：容器内权限完全继承宿主机目录权限。
- **实时同步**：宿主机修改文件，容器内立即生效。

**常用命令**：
```bash
# 挂载宿主机目录到容器
docker run -d --name my-app --mount type=bind,source=/home/user/code,target=/app/code nginx
# 或 -v 语法
docker run -d --name my-app -v /home/user/code:/app/code nginx

# 挂载单个文件
docker run -d --name my-app -v /home/user/nginx.conf:/etc/nginx/nginx.conf nginx
```

#### 三、核心区别对比

| 特性         | **数据卷 (Volume)**                | **绑定挂载 (Bind Mount)**          |
| :----------- | :--------------------------------- | :--------------------------------- |
| **管理方**   | Docker 管理                        | 用户管理                           |
| **存储位置** | `/var/lib/docker/volumes/`         | 宿主机任意路径                     |
| **自动创建** | 是                                 | 否                                 |
| **可移植性** | 高（跨平台/主机）                  | 低（依赖宿主机路径）               |
| **权限控制** | Docker 自动处理                    | 依赖宿主机权限                     |
| **安全性**   | 高（隔离性好）                     | 低（容器可修改宿主机文件）         |
| **适用场景** | 生产环境、数据库、日志、多容器共享 | 开发环境、代码热更新、配置文件同步 |

#### 四、如何选择？

- **优先用数据卷（Volume）**：
  - **生产环境**：数据库、缓存、应用数据。
  - **需要备份/迁移**：跨主机部署。
  - **多容器共享数据**：日志聚合、数据管道。
  - **不想手动管理路径**：让 Docker 全权处理。

- **优先用绑定挂载（Bind Mount）**：
  - **开发环境**：本地代码实时同步到容器。
  - **配置文件**：直接修改宿主机配置，容器内立即生效。
  - **日志收集**：容器日志直接输出到宿主机。
  - **需要直接操作宿主机文件**。

#### 五、总结

- **数据卷**：Docker 推荐、**安全、可移植、适合生产**。
- **绑定挂载**：**灵活、实时同步、适合开发**。

**一句话口诀**：**开发用绑定，生产用数据卷**。

你应该是想问：**Docker 容器数据持久化到底有哪几种？**
我给你用最简单、最清晰的方式总结一遍，只讲重点。

### Docker 容器数据持久化 三种方式

1. **绑定挂载（Bind Mount）**
   把宿主机**任意目录/文件**直接挂进容器。
   - 路径自己定
   - 开发常用，改代码实时生效

2. **数据卷（Volume）**
   Docker 自己管理的目录，存放在 `/var/lib/docker/volumes`。
   - 生产推荐
   - 安全、可移植、可备份
   - 数据库、业务数据都用它

3. **tmpfs 挂载（临时挂载）**
   数据只存在内存里，容器停了就没。
   - 存临时文件、密码、缓存
   - 不落地磁盘

---

#### 最简单一句话区分

- **开发 → 绑定挂载**
- **生产数据 → 数据卷**
- **临时机密 → tmpfs**

### **容器数据持久化命令**

分三类：**数据卷 Volume、绑定挂载 Bind Mount、临时挂载 tmpfs**。

---

#### 一、数据卷（Volume）命令（生产推荐）

##### 1. 创建数据卷

```bash
docker volume create my-vol
```

##### 2. 运行容器并挂载数据卷

```bash
docker run -d -v my-vol:/容器内路径 镜像名
```
示例（MySQL）：
```bash
docker run -d -v mysql-data:/var/lib/mysql mysql
```

##### 3. 查看数据卷

```bash
docker volume ls
```

##### 4. 查看数据卷详情

```bash
docker volume inspect my-vol
```

##### 5. 删除数据卷

```bash
docker volume rm my-vol
```

##### 6. 清理无用数据卷

```bash
docker volume prune
```

---

#### 二、绑定挂载（Bind Mount）命令（开发常用）

##### 挂载宿主机目录到容器

```bash
docker run -d -v /宿主机绝对路径:/容器内路径 镜像名
```

示例：
```bash
docker run -d -v /home/user/app:/app nginx
```

##### 只读挂载（安全）

（防止容器改坏宿主机文件）

```bash
// -v 加 :ro
docker run -d -v /host/path:/container/path:ro 镜像名

// --mount 加 readonly
docker run -d --mount type=bind,src=/host,dst=/app,readonly nginx
```

---

#### 三、tmpfs 临时挂载（内存存储，不落地）

```bash
docker run -d --tmpfs /容器内临时目录 镜像名
```

---

#### 四、统一推荐写法（--mount 更清晰）

##### Volume

```bash
docker run -d --mount type=volume,src=my-vol,dst=/data nginx
```

##### Bind Mount

```bash
docker run -d --mount type=bind,src=/host/path,dst=/data nginx
```

---

#### 五、Docker Compose 持久化写法

```yaml
volumes:
  # 数据卷
  my-vol:

services:
  app:
    image: nginx
    volumes:
      # 绑定挂载
      - ./local:/app
      # 数据卷
      - my-vol:/data
```

---

### **Linux / Docker 命令的标准语法规则**

#### 核心原因

- **`-v` 是短参数** → 只用 **一个横杠 `-`**
- **`--mount` 是长参数** → 必须用 **两个横杠 `--`**

---

#### 1. 短参数：-v

- 格式：**`-` + 单个字母**
- 例子：`-v`、`-d`、`-p`、`-i`、`-t`
- 特点：**简写、快速、方便敲**
```bash
-v  # volume 缩写
-d  # detach 缩写
-p  # port 缩写
```

---

#### 2. 长参数：--mount

- 格式：**`--` + 完整单词**
- 例子：`--mount`、`--name`、`--help`、`--version`
- 特点：**语义清晰、不容易写错**
```bash
--mount   # 完整单词
--name    # 完整单词
--help    # 完整单词
```

---

#### 3. 它们功能完全一样！

只是**写法不同**，不是功能不同。

```bash
-v my-vol:/data           # 短参数（一个横杠）
--mount src=my-vol,dst=/data  # 长参数（两个横杠）
```

**效果 100% 相同**。

---

#### 4. 超级好记的规则

- **一个字母 → 一个横杠 `-`**
  `-v` `-d` `-p`

- **多个字母/单词 → 两个横杠 `--`**
  `--mount` `--name` `--help`

---

#### 总结

- `-v` = 短参数 → **`-`**
- `--mount` = 长参数 → **`--`**
- 功能一样，只是写法规范不同！

现在彻底懂了吧？😀

------

## **五、Docker 安全与优化**

1. **如何限制容器的 CPU、内存使用？**

   - **思路**：

     ```bash
     docker run -m 512m --cpus="1.5" image_name
     ```

2. **如何保证 Docker 容器安全？**

   - **思路**：使用非 root 用户、最小化基础镜像、限制网络权限、扫描漏洞、定期更新镜像。

3. **Docker 镜像和容器的清理策略？**

   - **思路**：

     ```bash
     docker system prune -a  # 清理未使用的镜像、容器、网络
     ```





你整理的Docker安全与优化要点很实用，这部分直接关系到容器的稳定性、资源利用率和安全性。下面我会基于你的思路，补充资源限制的细节参数、安全措施的具体落地方式，以及清理策略的精细化操作，让方案更具可操作性。


### 1. 如何限制容器的 CPU、内存使用？
Docker 提供了精细的资源限制参数，防止单个容器过度占用宿主机资源（如“内存泄漏”导致整机崩溃），核心参数如下：

#### 内存限制
```bash
# 基础内存限制（最多使用512MB内存）
docker run -d -m 512m --name app1 image:latest
# -m/--memory：设置内存上限（支持单位 b/k/m/g，如 512m=512MB）

# 扩展：设置内存+交换分区总限制（默认内存限制=交换分区限制）
docker run -d -m 512m --memory-swap 1g image:latest
# 总可用内存（内存+swap）=1GB，其中swap=512MB（1g - 512m）

# 内存使用达到阈值时的策略（默认是“OOM杀死容器”）
docker run -d -m 512m --oom-kill-disable image:latest
# --oom-kill-disable：禁止OOM杀死容器（仅建议在确保容器不会耗尽内存时使用，否则可能导致宿主机OOM）
```

#### CPU 限制
```bash
# 限制CPU核心数（最多使用1.5个核心）
docker run -d --cpus="1.5" image:latest
# 适用于多核CPU，如4核机器中，容器最多使用1.5个核心的算力

# 限制CPU使用率（相对权重，默认1024，仅在CPU竞争时生效）
docker run -d --cpu-shares 512 image:latest
# 若容器A（512）和容器B（1024）竞争CPU，A会获得1/3算力，B获得2/3（非绝对限制，空闲时可超用）

# 绑定CPU核心（仅允许使用宿主机的0和1号核心）
docker run -d --cpuset-cpus="0,1" image:latest
# 适合对CPU亲和性有要求的场景（如高性能计算）
```

**最佳实践**：  
- 为所有容器设置内存上限（`-m`），避免单个容器耗尽内存导致宿主机崩溃。  
- CPU 限制优先用 `--cpus`（直观且精确），`--cpu-shares` 仅用于柔性分配。


### 2. 如何保证 Docker 容器安全？
容器安全需从“镜像构建→运行配置→生命周期管理”全流程防护，核心措施如下：

#### （1）镜像层面：减少攻击面
- **使用最小化基础镜像**：优先选择 `alpine` 或 `distroless` 镜像（无冗余工具和库，如 `nginx:alpine` 比 `nginx:latest` 少90%以上的文件），减少漏洞载体。  
- **非 root 用户运行**：在 Dockerfile 中创建普通用户，避免容器内进程以 root 权限运行（即使容器逃逸，也能降低危害）：  
  ```dockerfile
  # 示例：创建 appuser 并切换
  RUN addgroup -S appgroup && adduser -S appuser -G appgroup
  USER appuser  # 后续命令及容器运行时均以 appuser 执行
  ```
- **禁用不必要的工具**：镜像中删除 `bash`、`ssh` 等非必需工具（如 `distroless` 镜像默认无 shell），防止攻击者进入容器后横向移动。  
- **定期扫描镜像漏洞**：使用工具（如 `trivy`、`clair`）扫描镜像中的系统漏洞和依赖漏洞，拒绝部署高危镜像：  
  ```bash
  # 安装 trivy 后扫描镜像
  trivy image nginx:latest
  ```

#### （2）运行层面：限制权限
- **禁止特权模式**：不使用 `--privileged` 启动容器（该参数会赋予容器几乎所有宿主机权限，极易导致逃逸）。  
- **限制容器 capabilities**：仅保留必要的系统调用权限（如禁止 `CAP_NET_RAW` 防止容器发送原始网络包）：  
  ```bash
  docker run -d --cap-drop ALL --cap-add NET_BIND_SERVICE image:latest
  # --cap-drop ALL：禁用所有权限；--cap-add NET_BIND_SERVICE：仅允许绑定端口
  ```
- **限制网络访问**：通过自定义网络隔离容器，禁止不必要的端口暴露（如数据库容器仅允许应用容器访问，不暴露到公网）。  
- **只读文件系统**：对不需要写入的容器（如静态资源服务）设置只读根文件系统，仅挂载必要的可写目录：  
  ```bash
  docker run -d --read-only -v /tmp:/tmp image:latest
  # --read-only：根目录只读；-v：仅允许 /tmp 目录可写
  ```

#### （3）生命周期管理：动态防护
- **定期更新镜像和 Docker 引擎**：及时修复 Docker 引擎本身及基础镜像的安全漏洞（如 CVE 漏洞）。  
- **监控容器行为**：通过工具（如 `sysdig`、`falco`）监控容器的异常行为（如异常文件写入、网络连接），触发告警。  
- **禁止容器挂载敏感目录**：避免挂载宿主机的 `/proc`、`/sys` 等目录（可能被用于容器逃逸）。


### 3. Docker 镜像和容器的清理策略？
Docker 运行一段时间后会产生大量未使用的资源（如停止的容器、悬空镜像），需定期清理释放磁盘空间，核心命令和策略如下：

#### 基础清理命令
```bash
# 清理所有未被使用的资源（镜像、容器、网络、构建缓存）
docker system prune
# 注意：默认不删除未被引用的Volume（需手动确认）

# 强制清理所有未使用资源（包括Volume）
docker system prune -a --volumes
# -a：删除所有未被容器使用的镜像（包括有标签的）；--volumes：删除未被使用的Volume

# 单独清理容器（删除所有已停止的容器）
docker container prune

# 单独清理镜像（删除悬空镜像，即标签为 <none> 的镜像）
docker image prune

# 单独清理Volume（删除未被使用的Volume，谨慎使用！）
docker volume prune
```

#### 精细化清理策略
- **按条件删除容器**：删除指定时间前停止的容器（如7天前）：  
  ```bash
  docker rm $(docker ps -a -q --filter "status=exited" --filter "created-at<7d ago")
  ```
- **按条件删除镜像**：删除指定仓库（如 `myapp`）且7天未使用的镜像：  
  ```bash
  docker rmi $(docker images --filter "reference=myapp/*" --filter "dangling=false" --format "{{.ID}}" --filter "since=7d ago")
  ```
- **自动清理**：在生产环境中，通过 `cron` 定时执行清理命令（如每天凌晨3点清理）：  
  ```bash
  # 添加到 crontab（每天3点执行）
  0 3 * * * docker system prune -a -f --volumes  # -f：强制清理，无需确认
  ```

**注意事项**：  
- 清理前确保已停止的容器和未使用的镜像无保留价值（如备份数据）。  
- Volume 清理需格外谨慎（可能包含数据库等核心数据），建议手动确认后再删除。  


通过以上措施，可有效提升容器的资源利用率和安全性。如果需要具体场景的安全配置示例（如“Nginx 容器的最小权限配置”）或清理脚本，我可以进一步补充~

------

## **六、Docker 实战部署**

1. **如何使用 Docker Compose 启动多容器应用？**
   - **思路**：使用 `docker-compose.yml` 定义服务、网络和卷，然后执行 `docker-compose up -d`。
2. **如何在生产环境中部署 Java 后端应用？**
   - **思路**：多阶段构建减少镜像体积，复制 jar 包，暴露端口，持久化日志，使用 Compose 或 Kubernetes 管理多实例。
3. **容器启动失败如何排查？**
   - **思路**：查看容器日志 `docker logs <container>`，使用 `docker inspect <container>` 查看状态和错误信息，检查端口冲突和依赖服务。

你总结的Docker实战实战部署核心思路非常贴合实际开发需求，这三个问题是从本地调试到生产落地的关键环节。下面我会基于你的思路，补充具体配置示例、部署细节和排查技巧，让方案更具可操作性。


### 1. 如何使用 Docker Compose 启动多容器应用？
Docker Compose 是管理多容器应用的工具，通过 `docker-compose.yml` 定义所有服务（如前端、后端、数据库）的配置，一键启动/停止整个应用栈。

#### 核心步骤
##### （1）编写 `docker-compose.yml`
以“Spring Boot 后端 + MySQL 数据库 + Nginx 前端”为例：
```yaml
# docker-compose.yml
version: '3.8'  # Compose 版本（需与 Docker 版本兼容）

# 定义所有服务
services:
  # 后端服务
  backend:
    build: ./backend  # 基于 ./backend 目录的 Dockerfile 构建镜像
    image: myapp-backend:v1  # 镜像名称
    ports:
      - "8080:8080"  # 端口映射
    environment:  # 环境变量（数据库连接信息）
      - SPRING_DATASOURCE_URL=jdbc:mysql://db:3306/myapp?useSSL=false
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=123456
    depends_on:  # 依赖 db 服务，确保 db 先启动
      - db
    restart: always  # 容器退出时自动重启（生产环境推荐）
    networks:  # 加入自定义网络
      - myapp-net

  # 数据库服务
  db:
    image: mysql:8.0  # 使用官方 MySQL 镜像
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=123456
      - MYSQL_DATABASE=myapp  # 初始化数据库
    volumes:
      - mysql-data:/var/lib/mysql  # 数据卷持久化数据
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql  # 初始化脚本
    networks:
      - myapp-net

  # 前端 Nginx 服务
  frontend:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./frontend/dist:/usr/share/nginx/html  # 挂载前端静态文件
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf  # 自定义 Nginx 配置
    depends_on:
      - backend
    networks:
      - myapp-net

# 定义数据卷（持久化数据）
volumes:
  mysql-data:  # 自动创建数据卷，存储 MySQL 数据

# 定义自定义网络（服务间通过服务名通信）
networks:
  myapp-net:
    driver: bridge  # 桥接模式
```

##### （2）常用命令
```bash
# 构建并启动所有服务（后台运行）
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看某个服务的日志（如 backend）
docker-compose logs -f backend

# 停止并删除所有服务、网络（保留数据卷）
docker-compose down

# 停止并删除所有服务、网络和数据卷（谨慎使用，数据会丢失）
docker-compose down -v

# 重启单个服务（如修改后端代码后）
docker-compose restart backend
```

#### 核心优势
- **一站式配置**：所有服务的依赖、网络、存储集中定义，避免手动逐个启动容器。
- **服务名通信**：同一网络内的服务可通过服务名（如 `db`）访问，无需记 IP。
- **环境隔离**：不同项目的服务在独立网络中，避免端口和资源冲突。


### 2. 如何在生产环境中部署 Java 后端应用？
生产环境部署需兼顾**稳定性、可维护性和资源效率**，核心流程如下：

#### （1）构建优化的 Java 镜像（多阶段构建）
```dockerfile
# ./backend/Dockerfile
# 第一阶段：编译打包（使用 Maven 镜像）
FROM maven:3.8-openjdk-11 AS builder
WORKDIR /app
COPY pom.xml .
# 缓存依赖（避免每次构建重复下载）
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests  # 生成 target/app.jar

# 第二阶段：运行（仅保留 JRE 和 jar 包）
FROM openjdk:11-jre-slim  # 比 alpine 更稳定（alpine 可能有 JDK 兼容性问题）
WORKDIR /app
# 创建非 root 用户运行
RUN addgroup --system --gid 1001 appgroup && \
    adduser --system --uid 1001 --gid 1001 appuser
USER appuser

# 复制构建产物
COPY --from=builder /app/target/app.jar ./app.jar

# 暴露端口
EXPOSE 8080

# 启动命令（推荐用 exec 形式，确保信号能传递给 Java 进程）
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### （2）关键配置（生产环境必加）
- **资源限制**：避免应用占用过多资源，在 `docker-compose.yml` 中配置：
  ```yaml
  services:
    backend:
      # ... 其他配置
      deploy:
        resources:
          limits:
            cpus: '1'  # 最多使用1个CPU核心
            memory: 512M  # 最多使用512MB内存
  ```

- **日志持久化**：将应用日志挂载到宿主机，便于收集和分析：
  ```yaml
  services:
    backend:
      # ... 其他配置
      volumes:
        - ./logs:/app/logs  # 应用日志目录挂载到宿主机
  ```

- **健康检查**：自动检测应用是否存活，异常时重启：
  ```yaml
  services:
    backend:
      # ... 其他配置
      healthcheck:
        test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]  # 检测健康接口
        interval: 30s  # 每30秒检查一次
        timeout: 10s   # 超时时间
        retries: 3     # 连续3次失败视为不健康
        start_period: 60s  # 启动后60秒再开始检查（等待应用初始化）
  ```

- **多实例部署**：通过 `--scale` 快速扩展实例（简单负载均衡）：
  ```bash
  # 启动3个后端实例（配合前端 Nginx 反向代理实现负载均衡）
  docker-compose up -d --scale backend=3
  ```

#### （3）生产环境管理工具
- **小规模**：用 Docker Compose 足够（简单易用，适合单机多容器）。
- **大规模**：用 Kubernetes（K8s）管理（支持多机部署、自动扩缩容、滚动更新）。


### 3. 容器启动失败如何排查？
容器启动失败是常见问题，需按“日志→配置→依赖→资源”的顺序逐步排查：

#### （1）查看启动日志（最关键）
```bash
# 查看容器启动日志（即使容器已退出）
docker logs <容器ID或名称>

# 若日志无明显错误，查看 Docker 引擎事件日志（Linux）
journalctl -u docker  # 查看 Docker 服务日志
```

#### （2）检查容器配置和状态
```bash
# 查看容器详细配置（端口、挂载、环境变量等）
docker inspect <容器ID>

# 重点关注以下字段：
# - "State"：查看容器状态（如 ExitCode 非0表示异常退出）
# - "Mounts"：检查数据卷/挂载是否正确（路径是否存在、权限是否匹配）
# - "NetworkSettings"：检查端口映射是否冲突（"Ports" 字段是否有冲突提示）
# - "Env"：确认环境变量是否正确设置（如数据库地址、密码）
```

#### （3）排查常见失败原因
- **端口冲突**：容器端口已被宿主机或其他容器占用。  
  解决：`netstat -tulpn | grep <端口>` 查看占用进程，修改映射端口。

- **依赖服务未就绪**：如后端启动时数据库还未初始化完成（`depends_on` 仅保证启动顺序，不保证就绪）。  
  解决：在应用中添加重试机制（如 Spring 数据库连接重试），或使用工具（如 `wait-for-it`）等待依赖服务就绪。

- **权限问题**：容器内用户无权限访问文件（如挂载的宿主机目录权限为 `root:root`，但容器用普通用户运行）。  
  解决：调整宿主机目录权限（`chmod 777` 临时测试，生产环境需精确授权），或在 Dockerfile 中配置用户权限。

- **资源不足**：宿主机内存/CPU 不足，导致容器被 OOM 杀死。  
  解决：`docker stats` 查看资源使用，增加宿主机资源或调整容器资源限制。

- **镜像损坏**：镜像下载不完整或构建错误。  
  解决：重新拉取或构建镜像（`docker rmi <镜像> && docker build ...`）。

#### （4）进入容器调试（若能启动但异常）
若容器能启动但服务异常（如应用启动失败），可进入容器手动执行命令排查：
```bash
# 以交互模式启动容器，覆盖默认启动命令（执行 bash）
docker run -it --rm --entrypoint /bin/bash <镜像名>

# 然后手动执行启动命令，观察错误（如 java -jar app.jar）
```

通过以上步骤，可快速定位绝大多数容器启动问题。如果需要具体场景的排查案例（如“Spring Boot 容器因数据库连接失败启动失败”），我可以补充详细的排查流程~

# Docker 面试

全套（后端秋招版）

> 分为：基础概念、镜像&容器、网络、数据持久化、Docker Compose、原理、常见故障、面试简答版、20k+深度定薪专题，适合Java后端。
> 重点：面试官喜欢对比题（镜像vs容器、docker vs k8s、compose适用场景）、底层坑点（Cgroups与JVM兼容性、写时复制、depends_on、网络模式）

## 一、基础概念
### 1. 什么是Docker？Docker和虚拟机的区别（高频）
Docker是**容器化引擎**，基于Linux内核的Namespace、Cgroups，实现资源隔离与限制，打包应用+依赖，做到一次打包到处运行。

| 对比 | Docker容器 | 虚拟机VM |
|---|---|---|
| 内核 | 共享宿主机Linux内核 | 独立完整操作系统内核 |
| 体积 | MB级别，镜像很小 | GB级别 |
| 启动速度 | 秒级 | 分钟级 |
| 资源开销 | 低，只隔离进程 | 高，完整模拟硬件 |
| 隔离级别 | 进程级隔离（Namespace） | 完全硬件隔离 |

> 一句话背诵：虚拟机是**完整操作系统**；容器只是**隔离的进程**，共享宿主机内核。

### 2. 容器和镜像的区别（必问）
- **镜像（Image）**：只读模板，静态文件包，包含程序、依赖、配置；不能运行，类似安装包。分层只读。
- **容器（Container）**：镜像运行实例，动态进程；镜像启动时会在最上层加**可写层**。一个镜像可以启动多个容器。

> 类比：镜像=类Class；容器=new出来的实例对象。

### 3. Docker三大核心组件
1. **镜像 image**：只读分层文件系统
2. **容器 container**：镜像运行实例
3. **仓库 registry**：存放镜像（Docker Hub，阿里云镜像仓库）

## 二、镜像原理（面试重点：联合文件系统 UnionFS / COW写时复制）
### 1. 镜像分层原理
镜像由**多层只读层叠加**组成。
- 基础层：操作系统文件（alpine/ubuntu）
- 上层：安装依赖、复制代码、配置环境
- 好处：**复用层，节省存储空间**，多个镜像共用基础层，不需要重复下载。

### 2. COW 写时复制 Copy-on-Write（高频）
镜像所有层都是只读。容器启动时新增一层**可写层**。
- 读文件：从上往下找，找到第一个存在的文件
- 修改文件：**不会修改只读镜像层**，把文件复制到顶层可写层，修改副本。底层原文件保持不变。
> 坑：如果镜像里大文件频繁修改，会每次复制，性能变差。优化：把经常变动的数据放到volume挂载，不要写容器可写层。

### 3. Dockerfile 常用指令 & 镜像优化（Java后端常问）
```dockerfile
FROM：基础镜像
WORKDIR：工作目录
COPY：复制本地文件到镜像（推荐）
ADD：类似COPY，额外支持解压tar/远程url
RUN：构建镜像时执行命令（每一条RUN新增一层！）
EXPOSE：声明端口，**不实际开放端口，只是文档**
ENV：设置环境变量
CMD：容器启动默认命令，会被docker run后面命令覆盖
ENTRYPOINT：容器入口，不会被覆盖
```
✅ 镜像优化要点（面试加分）
1. 合并多个RUN，减少镜像层数
2. 选用轻量基础镜像：`openjdk:17-jdk-slim` / alpine，不要用完整版ubuntu
3. `.dockerignore` 排除无关文件：target、node_modules、日志
4. 多阶段构建（Java项目强烈推荐）：构建阶段编译打包，最终镜像只保留jar，不带maven/jdk源码，大幅减小镜像体积
5. **分层缓存优化**：先 `COPY pom.xml` 并下载依赖，再 `COPY src`。这样业务代码修改时，依赖层不会重新下载，直接命中构建缓存，加速构建。
6. **基础镜像选型**：推荐使用 Alpine 或 Distroless 极简镜像，不选臃肿的 CentOS/Ubuntu，体积小 5~10 倍且攻击面小、安全性高。
```dockerfile
# 多阶段构建示例（结合缓存优化）
FROM maven:3.8-openjdk-17 AS builder
WORKDIR /app
# 利用分层缓存：先下依赖再拷代码
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

FROM openjdk:17-jdk-slim
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
ENTRYPOINT ["java","-jar","app.jar"]
```

## 三、容器：Namespace & Cgroups（底层原理，容易深挖）
> Docker容器隔离底层靠Linux内核两个能力
1. **Namespace：做资源隔离**（PID、网络、文件系统挂载、用户ID、主机名），容器内看到的资源是独立视图；但是**不能限制资源用量**。
2. **Cgroups：资源限制**，限制CPU、内存、磁盘IO。可以限制容器最多用多少内存，OOM kill。

> 一句话区分：Namespace **看不见别的进程**；Cgroups **不能超过资源上限**。
> ⚠️ **高频定薪坑点（Cgroups 与 JVM 兼容性）**：早期 JDK 8（8u131 之前）无法识别 Cgroups 限制，仍按宿主机总内存计算堆大小，导致容器频繁被内核 OOM Killer 杀掉（详见第十二章）。

### 容器生命周期
`created` → `running` → `paused` → `stopped` → `deleted`
- `docker run`：创建并启动
- `docker stop`：优雅关闭，发送SIGTERM信号，等待超时后SIGKILL强制杀进程
- `docker kill`：直接SIGKILL强制杀死

## 四、数据持久化 Volumes（必考）

容器删除，容器可写层数据会丢失。三种持久化方式：
1. **Volume 命名卷（推荐）**：Docker管理，宿主机目录在docker数据目录，性能好，权限问题少。`-v mysql-data:/var/lib/mysql`
2. **Bind Mount 绑定挂载**：宿主机指定目录映射容器目录，`-v ./data:/var/lib/mysql`；适合开发，宿主机直接改文件。CentOS SELinux要加`:z`。
3. **tmpfs挂载**：数据放在宿主机内存，临时，断电丢失。

> 面试区分：Volume由docker管理；Bind Mount是你自己指定宿主机路径。生产数据库优先Volume。

> 核心前提：容器的**可写层是临时的**，容器删除，可写层数据全部丢失；三种挂载都是把数据放到容器外部实现持久化。

#### 1. Volume 命名卷（生产推荐）

```bash
# -v简写
docker run -d --name mysql -v mysql-data:/var/lib/mysql mysql:8
# --mount显式写法
docker run -d --name mysql --mount type=volume,source=mysql-data,target=/var/lib/mysql mysql:8
```
- 归属：**Docker管理**，卷保存在Docker的数据目录 `/var/lib/docker/volumes/卷名/_data`

- source：**卷名称**，不是宿主机路径

- 创建：`-v`不存在自动创建；`--mount`必须提前`docker volume create`

- 优点：性能好，权限问题少；**容器删除，卷不会自动删除**

- 缺点：宿主机不能直接随意操作底层文件

- 使用场景：数据库(Mysql/Redis)、生产环境

- `source=mysql-data` → 不是路径（没有 `/`、`./`），**是卷名称**。 Docker 会把这个卷存到 Docker 自己管理的目录：`/var/lib/docker/volumes/mysql-data/_data`

  > 这个目录是宿主机磁盘上的目录，只是交给 Docker 管理，**不是内存**。

查看卷真实位置命令：

```
docker volume inspect mysql-data
```

输出里面 `Mountpoint` 字段，就是宿主机磁盘路径。

#### 2. Bind Mount 绑定挂载（开发调试）

```bash
# -v简写
docker run -d --name mysql -v ./data:/var/lib/mysql:z mysql:8
# --mount显式写法
docker run -d --name mysql --mount type=bind,source=$(pwd)/data,target=/var/lib/mysql,z mysql:8
```
- 归属：**用户自己管理宿主机目录**
- source：宿主机**绝对/相对路径**
- 创建：`-v`宿主机目录不存在会自动创建；`--mount`目录不存在直接报错
- 优点：宿主机直接修改文件，实时同步到容器（前后端开发挂载代码）
- 缺点：SELinux容易权限报错；需要自己维护宿主机目录；移植性差
- 使用场景：本地开发，挂载代码、配置文件
- `source=./data` → 是宿主机路径（带 `./` / `/`），**用户自己指定的宿主机目录**。 数据就放在你当前目录的 `data` 文件夹，你可以直接进去修改文件。

#### 3. tmpfs 挂载（内存临时存储，不持久）

```bash
docker run -d --name nginx --mount type=tmpfs,target=/tmpdata,tmpfs-size=50M nginx
```
> ⚠️ `-v` 不支持 tmpfs，只能用 `--mount`
- 归属：宿主机**内存**，不落地磁盘
- 特点：速度极快；容器停止/宿主机断电，数据直接消失
- 使用场景：临时敏感数据、缓存，**绝对不能存放业务持久数据**

#### 对比表（面试直接背）

| 类型 | 存储位置 | 管理方 | 持久化 | 典型场景 |
|---|---|---|---|---|
| Volume命名卷 | Docker数据目录 | Docker | ✅持久 | 生产数据库 |
| Bind Mount | 用户指定宿主机目录 | 用户 | ✅持久 | 开发挂载代码 |
| tmpfs | 宿主机内存 | 内存 | ❌不持久 | 临时缓存、密钥 |

#### 面试高频问答

##### Q1：容器rm之后，Volume和Bind Mount的数据还在吗？

都还在！
- Volume：`docker rm 容器` 不会删卷；只有 `docker volume rm 卷名` / `docker rm -v 容器` 才删除卷。
- Bind Mount：数据在宿主机文件夹，删容器完全不影响宿主机文件。

##### Q2：Volume 和 Bind Mount 核心区别一句话？

**Volume由Docker管理存储目录；Bind Mount是用户指定宿主机已有目录映射到容器。**

##### Q3：什么时候用:z？

CentOS / RHEL / Stream9 SELinux开启环境，使用Bind Mount时，宿主机目录权限被SELinux拦截，加上`:z`给目录打标签，允许多容器读写。

##### Q4：tmpfs为什么不能存业务数据？

数据放在内存，宿主机重启、容器销毁，数据全部丢失，只适合临时数据。

#### Compose yaml 三种写法示例

```yaml
services:
  mysql:
    image: mysql:8
    volumes:
      # 1. Volume命名卷
      - mysql-data:/var/lib/mysql
      # 2. Bind Mount
      # - ./data:/var/lib/mysql:z
      # 3. tmpfs (不能写在volumes下，单独写tmpfs)
    # tmpfs:
    #   - /tmpdata:size=50M

volumes:
  mysql-data: # 声明命名卷
```

要不要我出2道面试真题（简答+选择题）自测一下这个知识点？



## 五、Docker网络（高频面试）
Docker 内置5种网络驱动：
1. **bridge（默认）**：桥接网络。宿主机docker0网桥。容器默认接入bridge。容器之间通过容器IP通信。
2. **host**：共享宿主机网络栈，**没有端口映射**，容器直接使用宿主机端口；性能最高；隔离差。适合监控组件。
3. **none**：无网络，完全隔离。
4. **container**：复用另一个容器网络。比如nginx容器共享pod网络。
5. **自定义bridge**（compose默认）：同一个自定义bridge里，**可以用服务名/容器名DNS互相访问**。不同自定义bridge网络隔离。

> Compose网络面试坑：
> compose会自动创建自定义bridge；同一compose服务之间通信**不需要端口映射ports**，ports只用于宿主机访问容器。

## 六、Docker Compose 面试专题（你前面问的，必背）

1. **Compose作用**：单机多容器编排工具，用yaml定义多个服务，一条命令管理整套应用。本地开发、测试环境。**不能用于集群生产环境，集群用K8s**。
2. V1 `docker-compose`（python，废弃） vs V2 `docker compose`（cli插件，官方推荐）。
3. `depends_on`：**只保证容器启动顺序，不保证应用就绪**。例如mysql容器进程启动成功，但数据库还没初始化完毕，java服务直接连接报错。解决方案：代码重试、wait-for-it脚本。
4. 常用命令：`up -d`后台启动；`down`删除容器网络，**默认不删除volume；down -v 删除卷**。
5. `.env`：compose自动读取当前目录.env环境变量。yml中 `${变量名}` 引用。

## 七、Docker镜像加速 & 常见问题
1. 镜像加速器：修改 `/etc/docker/daemon.json` 配置 `registry-mirrors`，`daemon-reload && restart docker`。`docker info` 验证。
2. daemon.json 语法错误，docker启动失败，JSON不能多余逗号。

## 八、Docker 与 K8s区别（经常追问）
- Docker：打包镜像、单机容器管理；Compose：单机多容器编排。
- K8s：多节点集群编排，负责容器调度、自愈、扩缩容、负载均衡。
> 一句话：Docker是容器引擎；K8s是容器集群管理平台。compose只能单机。

## 九、Docker线上故障排查（后端面试加分项）
1. 容器启动失败：`docker logs 容器id` 看日志
2. 进入容器：`docker exec -it xxx bash`；容器没启动exec进不去，只能看logs
3. 查看容器资源占用：`docker stats`
4. 容器OOM：dmesg 查看内核OOM日志，检查内存限制

## 十、精简简答版（面试口述，1分钟版）
> 适合一面口述，直接背
Docker是容器化技术，基于Linux Namespace隔离、Cgroups限制资源。镜像是只读分层模板，容器是镜像运行实例。写时复制COW，修改文件复制到可写层。持久化三种方式volume、bind mount、tmpfs。网络默认bridge，自定义bridge支持服务名DNS通信。Docker Compose单机多容器编排，适合开发测试，depends_on只控制容器启动顺序，不等待服务就绪。容器共享宿主机内核，比虚拟机轻量。集群部署使用K8s。

## 十一、面试坑题集合（容易踩坑）
1. EXPOSE 会开放端口吗？❌ 只是声明，不实际暴露。端口映射靠 `-p`。
2. docker run 不加-p，容器外部能访问吗？宿主机外不行；同一网络内其他容器可以访问。
3. docker down 会删除数据卷吗？❌ 默认不会，`down -v` 才删除。
4. 容器删除，volume数据还在吗？✅ volume独立于容器。
5. Docker容器是虚拟机吗？❌ 不是，只是隔离进程。

---

## 十二、20k+ Docker 核心考察点（Java 生产避坑与高频真题精炼版）

> **🎯 考察定位与定薪分水岭**：
> 面试官**不会把你当成运维工程师（SRE）来考**（不要求背诵冷门 Linux 内核参数或复杂安全配置）。
> 核心考察的是：**Java 进程在容器环境下的底层行为特征（内存/CPU/线程/信号）、高并发与云原生下的隐藏陷阱、Dockerfile 生产级工业规范，以及遇到线上容器崩溃或发版抖动时的排查能力**。

---

### 一、 JVM 与容器资源隔离冲突（定薪核心·必问）

#### 1. 容器内存超限被杀（ExitCode 137 与 JVM RSS 真实全景）
* **面试官提问：** “把 Java 应用打包进 Docker，限制容器内存 1G，运行时经常被杀掉（ExitCode 137），且**没有任何 Java OOM Dump 日志**；后来我升级了高版本 JDK 并把堆设为 750M，为什么高并发期依然被杀？”
* **20k+ 答题要点（直击底层）：**
  * **为什么退出码是 137？**
    Linux 内核强杀信号为 `SIGKILL (kill -9)`。Docker 容器退出码规则是 `128 + 信号值 = 128 + 9 = 137`。因为是内核暴力杀进程，JVM 根本来不及抛出 `OutOfMemoryError`，更来不及触发 `-XX:+HeapDumpOnOutOfMemoryError`。
  * **为什么早期 JDK 8 会直接挂？**
    早期 JDK 8（8u131 之前）不感知 Cgroups 限制，它看到的是宿主机物理内存（如 64G），默认按 1/4 计算最大堆（16G），直接远超容器 1G 限制，启动稍有负载即被内核杀掉。
  * **为什么堆设 75% 依然可能被杀？（JVM 实际物理内存 RSS 结构）**
    **核心误区**：容器内存超限检测的是进程的 **RSS（实际物理内存占用）**，而**不仅是堆内存（Heap）**！
    $$\text{JVM 物理内存 (RSS)} = \text{堆内存 (Heap)} + \text{元空间 (Metaspace)} + (\text{线程栈} \times \text{线程数}) + \text{堆外直接内存 (DirectMemory)} + \text{JVM 自身开销}$$
    - **线程栈消耗**：64 位系统每个线程默认占 1MB 物理栈空间（`-Xss1m`）。若业务线程池+框架线程达到 300 个，直接吃掉 300MB 物理内存！
    - **堆外直接内存**：Netty / gRPC 大量使用堆外零拷贝，若高并发缓冲未及时回收，轻松吃掉几百 MB；
    - **结论**：堆占 750MB + 线程栈 200MB + 元空间 150MB = 1.1GB，瞬间打爆容器 1G 限制！
  * **生产防护标准**：
    - 容器内存配额必须给堆外留出 **25%~30% 的安全缓冲垫**；
    - 在 JVM 参数中显式收敛非堆区域：`-XX:MaxMetaspaceSize=256m -XX:MaxDirectMemorySize=256m -Xss512k`。

#### 2. CPU 核心数误判与线程池爆炸陷阱
* **面试官提问：** “Docker 限制了 `--cpus=2`，为什么应用启动后线程数暴增到几百个，CPU 经常被耗尽，且接口响应时间（RT）严重抖动？”
* **20k+ 答题要点：**
  * **误判机理**：`Runtime.getRuntime().availableProcessors()` 在未开启容器感知的环境下，读取的是宿主机物理核心数（如 64 核）。
  * **连锁反应**：Netty Worker 线程数（默认 `核数 * 2`）、ForkJoinPool（默认 `核数 - 1`）、Tomcat 连接池等均按照 64 核创建数百个线程。
  * **后果**：只有 2 核算力却有几百个线程剧烈争抢，导致**极度频繁的上下文切换（Context Switch）**，CPU 时间大量浪费在内核调度上。同时触发 Linux CFS Quota 限额节流（CPU Throttling 冻结），导致接口 RT 阶梯式暴增。
  * **生产解决**：在 JVM 启动参数显式固定核数：`-XX:ActiveProcessorCount=2`，并严格收敛自定义业务线程池的最大线程数。

#### 3. 容器 JVM 启动参数黄金模板（标准工业版）
```bash
java \
  -XX:+UseContainerSupport \
  -XX:InitialRAMPercentage=70.0 \
  -XX:MaxRAMPercentage=70.0 \
  -XX:MinRAMPercentage=70.0 \
  -XX:ActiveProcessorCount=2 \
  -XX:MaxMetaspaceSize=256m \
  -XX:MaxDirectMemorySize=256m \
  -Xss512k \
  -Dsun.net.inetaddr.ttl=10 \
  -Dfile.encoding=UTF-8 \
  -XX:+ExitOnOutOfMemoryError \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/data/logs/dump/ \
  -jar app.jar
```
> **一句话亮点**：三核统一百分比（Initial = Max = Min = 70%）取代写死 `-Xms/-Xmx`，保证镜像在不同规格容器间灵活移植，同时避免内存动态伸缩引发的 GC 抖动；配合 `-XX:+ExitOnOutOfMemoryError` 确保发生 Java OOM 时快速退出自愈。

---

### 二、 容器生命周期与优雅停机（高可用核心·必问）

#### 1. PID 1 信号阻断陷阱（Shell 格式 vs Exec 格式）
* **面试官提问：** “服务发版执行 `docker stop` 时，为什么 Spring Boot 配置的 `server.shutdown=graceful` 完全无效，在途请求直接报 502/连接断开？”
* **20k+ 答题要点：**
  * **核心根因**：Dockerfile 中启动命令写成了 **Shell 格式**：
    
    ```dockerfile
    # ❌ 错误写法（Shell 格式）
    ENTRYPOINT java -jar app.jar
    ```
    Docker 底层会将其包装为 `/bin/sh -c "java -jar app.jar"` 执行。此时容器内的 **PID 1 是 `/bin/sh`**，Java 进程只是其子进程。当 Docker 发送 `SIGTERM (15)` 停机信号时，**`/bin/sh` 默认不转发信号给子进程**！Java 进程收不到信号，根本不会执行优雅停机。等待 10 秒超时后，Docker 直接发送 `SIGKILL (9)` 强杀容器。
  * **生产标准解法**：必须使用 **Exec 格式（JSON 数组语法）**：
    ```dockerfile
    # ✅ 正确写法（Exec 格式）
    ENTRYPOINT ["java", "-jar", "app.jar"]
    ```
    此时 Java 进程直接作为 PID 1 启动，能原生捕获 `SIGTERM` 信号，顺利触发 Spring Boot `ShutdownHook` 完成在途请求处理后再退出。

#### 2. 生产发版“零抖动”完整保障链路
* **完整闭环（口述链路）**：
  1. **注册中心下线/摘除节点**：K8s / Nginx 停止向当前容器路由新流量；
  2. **延时等待断流**：配置延时钩子（如 K8s `preStop: sleep 10`），给上游网关留出同步摘除 Endpoints 的时间，彻底截断新流量；
  3. **应用优雅停机**：Java 接收到 `SIGTERM`，拒绝新请求，处理完正在执行的在途请求并释放连接池；
  4. **超时兜底强杀**：若超过超时时间（如 30s），再由底层执行 `SIGKILL`。

---

### 三、 Dockerfile 工程化与镜像构建最佳实践（实操核心·必问）

#### 1. Spring Boot 2.3+ Layered Jar（分层 Jar 构建加速）
* **面试官提问：** “每次改动一行业务代码，打出来的镜像都要重新上传 100MB+ 到仓库，CI/CD 流水线极慢，怎么优化？”
* **20k+ 答题要点：**
  * **传统痛点**：单体 Fat Jar 把第三方依赖与业务代码打包在一起，改动一行业务代码会导致整个 Jar 的哈希改变，Docker 无法复用层缓存。
  * **分层打包解法**：利用 Spring Boot 官方 `jarmode=layertools` 将 Jar 拆解为 4 层：`dependencies`（依赖，极少变）、`spring-boot-loader`、`snapshot-dependencies`、`application`（业务代码，频繁变）。
  * **生产 Dockerfile 模板**：
    ```dockerfile
    # 阶段一：解构分层
    FROM eclipse-temurin:17-jre-jammy AS extractor
    WORKDIR /builder
    COPY target/app.jar app.jar
    RUN java -Djarmode=layertools -jar app.jar extract

    # 阶段二：按变动频率顺序复制（利用 Docker Layer 缓存）
    FROM eclipse-temurin:17-jre-jammy
    WORKDIR /app
    COPY --from=extractor /builder/dependencies/ ./
    COPY --from=extractor /builder/spring-boot-loader/ ./
    COPY --from=extractor /builder/snapshot-dependencies/ ./
    COPY --from=extractor /builder/application/ ./

    EXPOSE 8080
    ENTRYPOINT ["java", "org.springframework.boot.loader.launch.JarLauncher"]
    ```
  * **收益**：改动业务代码时，前 3 层完全命中远程与本地缓存，**每次只构建和推送不到 5MB 的 application 层**，构建速度从几分钟降至 10 秒以内。

#### 2. 基础镜像选型权衡：为什么生产谨慎使用 Alpine？
* **面试官提问：** “Alpine 镜像体积只有几兆，为什么你们生产环境没有选用 Alpine 作为 Java 基础镜像？”
* **20k+ 答题要点（真实踩坑经验）：**
  * **C 库差异（musl libc vs glibc）**：Alpine 使用 musl libc，不是标准的 glibc。
  * **三大暗坑**：
    1. **排查利器失效**：诊断工具 **Arthas**、`async-profiler` 以及部分依赖 JNI 的 Native 库（如 snappy 压缩、rocksdb）在 musl 下容易崩溃或无法 attach；
    2. **中文字体与图形缺失**：Alpine 极简镜像缺少中文字体库与 fontconfig，Java 导出 Excel（POI/EasyExcel）或生成图形验证码时直接抛空指针 `FontConfiguration` 异常；
    3. **时区问题**：默认 UTC，需要额外安装 tzdata 配置上海时区。
  * **生产选型结论**：选用 **Debian / Ubuntu Slim 版本**（如 `eclipse-temurin:17-jre-jammy`），体积约 150MB~200MB，兼顾轻量与完整的 glibc 生产排查兼容性。

---

### 四、 JDK 17 / Spring Boot 3 容器化新特性与踩坑（前沿标配）

#### 1. 模块化强封装反射报错（`--add-opens` 生产注入）
* **现象与报错**：Spring Boot 3 / JDK 17 容器启动时，若引入了旧版 MyBatis、CGLIB、MapStruct、Jackson 或反射工具类，常报错：
  `InaccessibleObjectException: Unable to make field private accessible: module java.base does not "opens java.lang"`
* **原因**：JDK 17 默认开启强封装（JEP 403），禁止通过反射绕过权限访问核心类的私有字段。
* **解决**：在容器启动脚本中显式配置开放反射白名单：
  ```bash
  --add-opens java.base/java.lang=ALL-UNNAMED \
  --add-opens java.base/java.lang.reflect=ALL-UNNAMED \
  --add-opens java.base/java.util=ALL-UNNAMED
  ```

#### 2. G1 GC 在小规格容器中的“内存刺客”陷阱
* **现象**：在 1C/1G 或 1C/2G 的小容器里跑 JDK 17，即使堆设了 70%，容器仍偶发被内核 OOM Killer 强杀。
* **底层原因**：JDK 17 默认使用 G1 GC。G1 为了维护跨 Region 引用关系，内部维护的 **RSet（Remembered Set）与 Card Table 会额外占用堆大小 10%~15% 的堆外本地内存**。小容器本身堆外预算非常紧张，很容易被堆外开销挤爆。
* **生产建议**：
  - 1G~2G 小规格容器：建议将堆内存占比压低至 60%（`-XX:MaxRAMPercentage=60.0`），或评估改用轻量收集器 `-XX:+UseParallelGC`；
  - 4G 及以上规格容器：保持 G1 GC 默认配置，低延迟性能最佳。

---

### 五、 微服务容器网络通信陷阱（高频常识）

#### 1. 为什么微服务容器间调用严禁使用 `-p` 宿主机端口映射？
* **面试官提问：** “同主机或者同一网络下的两个微服务容器相互调用，能不能直接用宿主机 IP + `-p` 映射的端口访问？生产规范为什么禁止这样做？”
* **20k+ 答题要点：**
  * **网络损耗**：走宿主机 IP + `-p` 端口映射，流量必须经过 Linux 内核的 **iptables DNAT/SNAT 网络地址转换**，多了一层网络虚拟化栈的封装与解封，在高并发下会造成明显的网络延迟和宿主机软中断（softirq）升高；
  * **端口冲突与维护成本**：如果使用 `-p`，每一个微服务都必须抢占宿主机的一个物理端口，无法利用容器的端口隔离特性（两个服务不能同时绑定 8080）；
  * **生产正确做法**：微服务必须统一接入同一个自定义 Bridge 网络，**直接通过容器名或服务名通信（如 `http://order-service:8080`）**。Docker 内部集成了轻量级 DNS 服务，自动将容器名解析为内网容器 IP，完全在 Linux 虚拟网桥（docker0 或 br-xxx）内二层直接转发，零 NAT 额外损耗。

#### 2. 容器 IP 动态漂移与 JVM DNS 缓存（TTL）死穴
* **面试官提问：** “下游容器故障重启或重新发布后分配了新容器 IP，为什么上游 Java 微服务一直在报 `Connection Refused`，重启上游服务后才恢复？”
* **20k+ 答题要点：**
  * **根因**：Java JVM 默认的 DNS 解析策略为了提高性能，在部分配置下会将域名解析结果**永久缓存（TTL = -1）**。当下游容器销毁重建、Docker DNS 虽已更新为新 IP，但上游 Java 进程内部的 InetAddress 缓存永远不失效，依然往死掉的旧 IP 发送 TCP 请求。
  * **生产标准解法**：在 Java 容器启动参数中显式缩短 DNS 缓存生存时间：
    ```bash
    -Dsun.net.inetaddr.ttl=10
    ```
    强制 JVM 每 10 秒刷新一次 DNS 解析结果，完美应对微服务容器动态伸缩与漂移。

---

### 六、 现场答辩必备：两大高频真实故障排查故事（直接背诵）

#### 故事一：线上 Java 容器频发 ExitCode 137 异常重启排查
* **现象**：
  
  > “我们核心订单服务在促销高峰期偶发性重启。查看容器状态，退出码为 `137`，但是业务日志和 JVM 都没有生成任何 `OutOfMemoryError` 堆 Dump 文件。”
* **排查链路**：
  > “1. 退出码 137 说明容器收到了 `SIGKILL` 强杀信号。我登录宿主机执行 `dmesg -T | grep -i oom`，明确看到内核日志记录了 `Memory cgroup out of memory: Kill process (java)`，证实是触发了内核的 OOM Killer。
  > 2. 进一步排查 JVM 内存配置，虽然堆内存限制为容器的 75%，但通过监控发现该服务集成了 Netty 堆外通信，且异步线程池创建了 300 多个线程（每个线程栈占 1MB）。堆内存（75%）+ 线程栈（300MB）+ 堆外直接内存 突破了容器配额。
  > 3. **解决方案**：我们将基础镜像升级为支持容器感知的 JDK 17，将堆内存百分比调整为 70%（预留 30% 安全缓冲），并显式限制 `-Xss512k -XX:MaxDirectMemorySize=256m`。调整后高峰期内存稳定在容器限额的 85% 以内，彻底根治了偶发重启。”

#### 故事二：发版滚动更新时接口短暂报 502 排查与优雅停机
* **现象**：
  
  > “我们在自动化流水线滚动部署新版本时，网关在发版瞬间会出现短暂几秒的 502 Bad Gateway 和连接重置异常。”
* **排查链路**：
  > “1. 检查了 Spring Boot 配置，确认已经开启了 `server.shutdown=graceful`，但发版依然报错，说明应用没有完成优雅停机就被杀死了。
  > 2. 检查 Dockerfile 发现启动命令写成了 Shell 格式：`ENTRYPOINT java -jar app.jar`。进入容器执行 `ps -ef` 查看，发现 PID 1 是 `/bin/sh`，Java 进程是 PID 7。因为 `/bin/sh` 不转发 `SIGTERM` 信号，Docker 等待 10 秒超时后直接发送 `SIGKILL` 强杀容器，导致在途请求被粗暴掐断。
  > 3. **解决方案**：将 Dockerfile 改为 Exec 格式 `ENTRYPOINT ["java", "-jar", "app.jar"]`，使 Java 进程直接成为 PID 1；同时在容器销毁前增加 10 秒延时断流（如 K8s preStop 钩子 `sleep 10`），给上游网关留出摘除路由的时间。改动后实现了发版过程 0 报错平滑过渡。”

---

### 七、 20k+ 必考综合实战：4核8G 资源受限下 Docker 架构部署与极致优化（内存+磁盘压榨指南）

> **💡 面试官核心动机**：
> “4核8G（4C8G）”是云厂商最主流的高性价比 ECS/CVM 规格，几乎所有中小企业、初创团队、测试预发环境或边缘微服务节点都在使用。
> 面试官抛出这道题，**绝不是考简单的启动命令，而是考察你的工程容量规划能力（Capacity Planning）、技术成本意识（降本增效）、以及在资源紧绷环境下保证系统不 OOM、磁盘不爆盘的实战防护功底**。

#### 1. 业务场景规划：4C8G 容器容量配额矩阵（严禁超卖）

假设要在单台 4C8G 服务器上通过 Docker 跑一套完整的最小闭环业务系统（**Nginx + Spring Cloud Gateway + 核心业务服务 + MySQL 8.0 + Redis 7.0**）：

| 容器组件 | CPU 限制 (`--cpus`) | 内存硬限制 (`-m`) | 核心参数/关键配置 | 规划设计依据 |
| :--- | :--- | :--- | :--- | :--- |
| **宿主机 OS & Docker 守护进程** | 留底 0.5 核 (突发) | **预留 1.5GB** | 保留给 Linux 内核、SSH、监控 Agent、page cache | **防死底线**：严禁把 8G 内存全分完，否则宿主机内核 OOM 崩溃，连 SSH 都登不进去 |
| **MySQL 8.0** | 1.0 核 | **1.5GB** | `innodb_buffer_pool_size=1G`<br>`max_connections=150` | 默认参数会吃 2G+，必须调小 buffer pool 并限制最大连接数 |
| **Redis 7.0** | 0.5 核 | **512MB** | `maxmemory 400mb`<br>`maxmemory-policy allkeys-lru` | 强制限定最大内存与淘汰策略，防止未设限无底线吃光宿主机内存 |
| **Spring Cloud Gateway** | 0.5 核 | **800MB** | `-XX:MaxRAMPercentage=65.0`<br>`-Xss256k` | 基于 Netty 响应式轻量路由，主要消耗堆外直接内存，堆分配 500M 足够 |
| **Core Service (核心业务应用)** | 1.5 核 | **3.5GB** | `-XX:MaxRAMPercentage=65.0`<br>`-Xss512k`<br>`-XX:MaxMetaspaceSize=256m` | 核心业务承载，堆内存约 2.3G，留出 1.2G 给元空间、线程栈与框架堆外开销 |
| **Nginx (前端静态托管+反代)** | 0.2 核 | **200MB** | `worker_processes 2;`<br>`worker_connections 1024;` | 极度轻量，主要支撑静态资源缓存与 SSL 卸载 |
| **总计规划** | **4.2 核 (时分复用)** | **8.0 GB (严密平衡)** | 各组件均配置明确资源配额（`resources.limits`） | 杜绝任意单个容器发生内存泄漏拖死整台服务器 |

##### 附：4C8G 标准生产编排模板 (`docker-compose.yml`)
```yaml
version: '3.8'

services:
  core-service:
    image: myrepo/core-service:v1.0
    restart: always
    deploy:
      resources:
        limits:
          cpus: '1.5'
          memory: 3500M
        reservations:
          cpus: '0.5'
          memory: 1500M
    environment:
      - JAVA_OPTS=-XX:+UseContainerSupport -XX:MaxRAMPercentage=65.0 -Xss512k -XX:MaxMetaspaceSize=256m
    logging:
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "3"
    depends_on:
      - mysql
      - redis

  mysql:
    image: mysql:8.0-debian
    restart: always
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1500M
    command: --innodb_buffer_pool_size=1G --max_connections=150
    volumes:
      - /data/mysql/data:/var/lib/mysql

  redis:
    image: redis:7-alpine
    restart: always
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    command: redis-server --maxmemory 400mb --maxmemory-policy allkeys-lru
```

---

#### 2. 如何极致节省「内存空间」？（JVM 与中间件双向压榨）

##### ① JVM 层面四重内存瘦身
1. **精准控制 RAMPercentage，预留足量堆外安全垫**：
   - 4C8G 环境下单个 Java 容器内存上限通常在 1G~3.5G 之间。
   - 配置 `-XX:MaxRAMPercentage=65.0`（而不是盲目的 75% 或 80%），硬性预留 35% 堆外物理内存给元空间、线程栈和框架 DirectMemory，从根源杜绝 ExitCode 137 内核强杀。
2. **压缩线程栈空间（降低 50%+ 物理占用）**：
   - 默认 64 位 JVM 每个线程栈占 1MB（`-Xss1m`）。若应用线程池包含 200 个线程，启动就霸占 200MB 物理内存！
   - 配置 `-Xss256k` 或 `-Xss512k`。对于一般的 Spring Boot 业务代码，512k 甚至 256k 深度完全不会引起 `StackOverflowError`，却能**瞬间释放 100MB~150MB 物理常驻内存**。
3. **GC 收集器精打细算（拒绝无脑用 G1）**：
   - **核心认知**：G1 GC 的跨 Region 记忆集（RSet）本身会额外霸占堆大小 **10%~15% 的堆外内存**。
   - **选型策略**：
     - 若给容器分配内存 **>= 3GB**（如核心服务）：使用默认 G1 GC，享受高并发低停顿优势；
     - 若给容器分配内存 **<= 1.5GB**（如网关或小服务）：建议显式配置 `-XX:+UseParallelGC` 或 `-XX:+UseSerialGC`，直接省掉 G1 RSet 的 150MB~200MB 堆外额外内存负担，启动速度更快！
4. **Spring Boot 框架级轻量化**：
   - 在 `application.properties` 中开启组件懒加载：
     ```properties
     spring.main.lazy-initialization=true
     ```
     让 Bean 在首次被调用时才初始化，不仅大幅加快启动速度，还能**直接削减 20%~30% 的常驻启动内存**。

##### ② 核心中间件内存强制锁定
1. **MySQL 内存防暴增**：
   - 挂载自定义 `my.cnf`，强制约束：
     ```ini
     [mysqld]
     innodb_buffer_pool_size = 1G
     innodb_log_buffer_size = 16M
     max_connections = 150
     table_open_cache = 400
     ```
2. **Redis 内存防失控**：
   - 必须在启动参数或配置文件中限制内存，绝不允许无上限运行：
     ```bash
     docker run -d --name redis -m 512m redis:7-alpine \
       redis-server --maxmemory 400mb --maxmemory-policy allkeys-lru
     ```

##### ③ 宿主机 Linux 内核两项关键防死调优
1. **防止 Redis BGSAVE Fork 失败 (`vm.overcommit_memory = 1`)**：
   - 在 4C8G 紧绷内存下，Redis 进行 RDB/AOF 重写时会 `fork()` 子进程。Linux 默认 `vm.overcommit_memory = 0` 会按悲观算法检查申请内存，导致 Redis 报错 `Can't save in background: fork: Cannot allocate memory`。
   - **优化**：在宿主机 `/etc/sysctl.conf` 配置 `vm.overcommit_memory = 1`。
2. **避免 JVM 堆内存被置换到 Swap 盘 (`vm.swappiness = 1`)**：
   - 若 Linux 开启了 Swap 且 swappiness 保持默认 60，内核在内存吃紧时会把 JVM 不活跃的堆页面写入 Swap 磁盘。一旦 GC 扫描到 Swap 页面，会引发海量磁盘 I/O 等待，导致 **GC STW 卡顿由毫秒级暴增到数十秒**！
   - **优化**：配置 `vm.swappiness = 1`，优先回收 Page Cache，宁死不将 Java 堆交换到磁盘。

---

#### 3. 如何极致节省「磁盘空间」？（Docker 生产防爆盘四板斧）

很多线上故障不是 CPU 内存满了，而是 **Docker 运行一段时间后磁盘 100% 暴毙**，导致 MySQL 无法写入事务日志而宕机。

##### ① 第一板斧：全局日志限流轮转（防范最致命的隐形炸弹）
* **痛点**：Docker 默认的 `json-file` 日志驱动**不限制单文件大小**，容器内的 `stdout` 会无休止写入，几天即可跑出几十 GB 的大日志直接撑爆宿主机磁盘。
* **全局根治配置**：修改宿主机 `/etc/docker/daemon.json` 并重启 Docker：
  ```json
  {
    "log-driver": "json-file",
    "log-opts": {
      "max-size": "50m",
      "max-file": "3"
    }
  }
  ```
  单个容器日志上限严格锁定为 `50MB * 3 = 150MB`，超额自动滚动覆盖，全机所有容器彻底杜绝日志爆盘。

##### ② 第二板斧：Dockerfile 极简化与构建层瘦身
1. **基础镜像选型**：
   
   - 选用 Debian Slim 或 Alpine（如 `eclipse-temurin:17-jre-jammy`），体积由 Ubuntu/CentOS 的 500MB 降至 150MB 左右；
2. **合并 RUN 命令并即时清理缓存**：
   ```dockerfile
   # ❌ 错误做法：每一条 RUN 都会独立生成一层持久只读层
   RUN apt-get update
   RUN apt-get install -y curl
   
   # ✅ 正确做法：合并命令，且在同一层内彻底删除安装包缓存
   RUN apt-get update && apt-get install -y --no-install-recommends curl \
       && apt-get clean \
       && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
   ```
3. **引入 `.dockerignore`**：
   
   - 在 Dockerfile 同级目录必须创建 `.dockerignore`，排除 `.git`、`target/`、`node_modules`、`*.log`，防止每次 `docker build` 将数十 GB 的无用上下文打入镜像构建引擎。
4. **Spring Boot 分层 Jar（Layered Jar）**：
   
   - 依赖层与业务层分离，每次代码迭代仅推送 5MB 业务差异层，不会在磁盘堆积大量重复臃肿的历史 Jar 镜像层。

##### ③ 第三板斧：自动化磁盘定期清理策略（Crontab 定时器）
* Docker 长期迭代会残留大量无标签的悬空镜像（`<none>` 镜像）、退出的容器和临时的构建缓存。
* **配置生产定时任务**（每周日凌晨执行）：
  ```bash
  # 编辑 crontab
  crontab -e
  
  # 每周日凌晨 3 点自动清理所有未被使用的容器、网络、镜像（保留 Volume 数据卷）
  0 3 * * 0 /usr/bin/docker system prune -af --volumes=false > /dev/null 2>&1
  ```
  此命令可一键回收 5GB~20GB 的废弃镜像与缓存碎片，且不会破坏挂载的业务数据。

##### ④ 第四板斧：数据持久化与日志外挂分离
* **业务持久数据**：MySQL 数据目录务必通过 Volume 挂载在空间充裕的数据盘分区（如 `/data/docker-volumes`），绝对不能直接写在容器的可写层；
* **业务落地日志（Logback）**：通过挂载外挂宿主机目录，并必须在 `logback-spring.xml` 中配置 **`<maxHistory>7</maxHistory>`（仅保留 7 天）** 及 **`<totalSizeCap>3GB</totalSizeCap>`**，防止业务本地日志失控膨胀。

---

#### 4. 面试现场高分答辩模板（口述一分钟版）

* **面试官提问：** “如果给你们团队分配一台 4核8G 的服务器，你要在上面部署一套微服务系统，你会怎么规划？怎么保证内存和磁盘不崩？”
* **20k+ 现场答辩参考话术：**
  > “对于 4C8G 这样的典型资源受限环境，我的核心策略是**‘算力切分留底线、内存分层防 OOM、磁盘轮转防爆盘’**：
  > 
  > 1. **在资源规划上**：我严格杜绝超卖。首要原则是**保留 1.5GB 内存与 0.5 核给宿主机操作系统和守护进程**，防止系统雪崩连 SSH 都登不进。剩下的内存中，核心业务 Java 服务分配 3.5G，MySQL 分配 1.5G，网关分配 800M，Redis 分配 512M，所有容器都在 Docker 或 Compose 中硬性设定 `limits`。
  > 2. **在内存压榨上**：
  >    - **Java 层面**：我将 `MaxRAMPercentage` 设为 65%，预留 35% 给非堆；将 `-Xss` 压缩为 512k 或 256k，立竿见影省下上百兆物理栈内存；针对 1G 左右的轻量服务直接用 Parallel GC 替代 G1，避开 G1 额外的 RSet 堆外开销；开启 Spring Boot 的 `lazy-initialization` 降低启动常驻内存。
  >    - **中间件层面**：锁定 MySQL `innodb_buffer_pool_size=1G`，Redis 强制设置 `maxmemory 400mb` 和 LRU 淘汰，杜绝中间件失控膨胀。
  > 3. **在磁盘防爆上**：
  >    - 在 `/etc/docker/daemon.json` 配置全局日志轮转，单容器日志硬限制 `50m * 3`；
  >    - 采用 Spring Boot Layered Jar 和 Slim 基础镜像，每条 RUN 命令清理 apt 缓存；
  >    - 部署每周 Crontab 定时执行 `docker system prune` 回收悬空镜像，同时业务 Logback 配置 7 天滚动删除。
  > 
  > 这一套方案在我们实际生产环境下，能让 4C8G 节点在保证系统高可用的前提下，长期平稳运行在 80% 内存水位与 30% 磁盘占用率以内。”

---

# Docker简短回答

## **一、Docker 与微服务架构**

1. **Docker 如何支持微服务架构？**
   - **思路**：每个微服务打包成独立容器，实现隔离、快速部署、可扩展；通过容器网络实现服务通信。
2. **微服务中容器通信方式有哪些？**
   - **思路**：Bridge 网络内通信、Host 网络、使用 Docker Compose 或 Service Discovery（Consul/Eureka/DNS）。
3. **如何在微服务中实现容器健康检查？**
   - **思路**：使用 Docker `HEALTHCHECK` 指令，或通过外部监控工具（如 K8s readiness/liveness probe）。
4. **多容器微服务如何管理环境变量和配置？**
   - **思路**：使用 `.env` 文件、Docker Compose environment 配置、或配置中心（Spring Cloud Config）。
5. **如何实现微服务的容器水平扩展？**
   - **思路**：增加容器副本数（`docker-compose scale` 或 Kubernetes Deployment replicas），结合负载均衡。
6. **容器之间的依赖如何处理？**
   - **思路**：使用 Docker Compose `depends_on`、Kubernetes init containers，确保服务按顺序启动。

------

## **二、Docker 与 K8s**

1. **Docker 容器与 Kubernetes Pod 的区别？**
   - **思路**：Pod 是 K8s 最小调度单元，可包含一个或多个容器；Pod 内共享网络和卷，生命周期由 K8s 管理。
2. **Kubernetes 如何调度 Docker 容器？**
   - **思路**：K8s 使用 kube-scheduler 按资源需求、节点标签、亲和性策略调度容器到节点。
3. **K8s 中如何实现容器滚动更新？**
   - **思路**：Deployment 配置 `rollingUpdate`，逐步替换旧容器副本，保证服务可用。
4. **Docker 镜像在 K8s 中的拉取策略有哪些？**
   - **思路**：`Always`（总是拉取）、`IfNotPresent`（本地不存在才拉）、`Never`（仅使用本地）。
5. **如何在 K8s 中实现容器日志收集？**
   - **思路**：容器输出到 stdout/stderr，通过 DaemonSet 部署 ELK/EFK 或 Fluentd 收集日志。
6. **K8s 中的 ConfigMap 和 Secret 如何挂载到 Docker 容器？**
   - **思路**：通过 volume 或 environment variables 挂载配置或密钥，支持动态更新（ConfigMap）。
7. **K8s Pod 启动失败常见原因？如何排查？**
   - **思路**：资源不足、镜像拉取失败、启动命令错误、依赖服务未就绪；使用 `kubectl describe pod` 和 `kubectl logs` 排查。

------

## **三、Docker 与 CI/CD**

1. **如何将 Docker 集成到 CI/CD 流程？**
   - **思路**：在 CI 阶段构建 Docker 镜像、单元测试；在 CD 阶段推送镜像到私有仓库并部署到测试/生产环境。
2. **常用 CI/CD 工具如何与 Docker 配合？**
   - **思路**：Jenkins、GitLab CI、GitHub Actions，可通过 Docker Agent 构建镜像、执行测试和部署。
3. **如何在 CI/CD 中实现镜像版本管理？**
   - **思路**：使用 Git commit hash 或 Semantic Version 生成镜像标签，避免覆盖已有版本。
4. **CI/CD 中如何快速回滚容器版本？**
   - **思路**：在部署时使用版本化镜像标签，出现问题直接切换到旧版本标签重新部署。
5. **如何在 CI/CD 流程中进行多环境部署？**
   - **思路**：使用 Docker Compose override 文件或 Kubernetes Namespace 区分 dev/test/prod 环境，配置环境变量和资源限制。

------

## **四、Docker 性能优化与监控**

1. **Docker 容器启动慢的原因？如何优化？**
   - **思路**：镜像大、启动脚本复杂；优化方式：使用精简镜像、多阶段构建、减少启动依赖。
2. **Docker 容器性能监控指标有哪些？**
   - **思路**：CPU 使用率、内存使用率、磁盘 I/O、网络流量；工具：`docker stats`、cAdvisor、Prometheus+Grafana。
3. **如何优化 Docker 网络性能？**
   - **思路**：使用 Host 网络模式减少 NAT 开销、优化 Overlay 网络、减少容器跨主机通信。
4. **如何减少 Docker 容器磁盘 IO 开销？**
   - **思路**：使用 Volume 持久化数据、避免大量小文件写入、尽量使用 tmpfs 临时存储。
5. **Docker 镜像拉取慢如何优化？**
   - **思路**：使用国内镜像源、构建私有 Registry、使用分层缓存和多阶段构建。
6. **如何监控容器资源超限并自动伸缩？**
   - **思路**：结合 Kubernetes HPA（Horizontal Pod Autoscaler）、Prometheus AlertManager，或 Docker Swarm 的 auto-scaling 工具。
7. **Docker 容器内存泄漏如何排查？**
   - **思路**：使用 JVM 参数（如 `-Xmx` 限制内存）、`docker stats` 监控内存变化、分析应用日志和堆内存 dump。

## 项目开发中，CI/CD 到底体现在哪里？

我用**最接地气、开发能直接感知**的方式告诉你：**CI/CD 不是抽象概念，它就是你写代码后的一整套自动化流程**。

你只需要记住一句话：
**CI = 自动检查 + 自动构建
CD = 自动部署 + 自动发布**

下面我按**你真实开发场景**讲，你立刻就能明白它体现在哪。

---

### 一、CI（持续集成）体现在哪里？

**你一提交代码，它就自动干活。**

你平时开发会做这些事：
- 写代码
- git push 到仓库
- 想知道代码有没有报错
- 想知道编译通不通过
- 想知道测试能不能过

**CI 就是把这些全部自动化。**

#### CI 在项目中的具体体现：

1. **你 push 代码 → 自动触发检查**
2. **自动拉取最新代码**
3. **自动编译/构建**
4. **自动跑单元测试、接口测试**
5. **自动代码规范检查（ESLint、CheckStyle）**
6. **自动生成构建产物（jar、dist、image）**

#### 你能感受到的效果：

- 不用自己手动编译
- 不用自己跑测试
- 提交代码几分钟就告诉你：**能不能合入、有没有bug**
- 团队代码质量自动保证

---

### 二、CD（持续部署/持续交付）体现在哪里？

**构建成功后，自动帮你部署到服务器，不用你手动上传。**

CD 分两层：

#### 1. 持续交付（Delivery）

- 自动把打包好的应用
- 推送到仓库（Docker 镜像、npm包、jar包）
- **随时可以部署，但需要手动点一下部署**

#### 2. 持续部署（Deployment）

- **完全自动**
- 构建成功 → 直接部署到测试环境/预发环境/生产环境

##### CD 在项目中的真实体现：

- 你提交代码 → 自动构建 → **自动部署到测试环境**
- 测试环境自动更新，不用手动上传
- 测试人员马上能测最新版本
- 上线时，一键/自动发布到生产环境

---

### 三、用一句话总结你能感知的 CI/CD

#### **提交代码 → 自动检查 → 自动构建 → 自动测试 → 自动部署 → 最新版本上线**

你不用做：
- 手动打包
- 手动传服务器
- 手动重启服务
- 手动跑测试
- 手动检查代码错误

这些全部**自动完成**。

---

### 四、最常见的 CI/CD 工具（你项目里大概率会见到）

- Jenkins
- GitLab CI/CD
- GitHub Actions
- Docker + Kubernetes（配合CI/CD）

---

### 五、最简单的例子（一看就懂）

你写了一段Vue/Java代码：

1. `git push`
2. GitLab CI 自动触发
3. 自动安装依赖
4. 自动跑测试
5. 自动构建 `dist` 或 `jar`
6. 自动构建 Docker 镜像
7. 自动推送到镜像仓库
8. **自动部署到服务器并启动**

**全程你只干了一件事：提交代码。**

这就是 **CI/CD**。

---

### 总结

- **CI**：代码提交后**自动检查、构建、测试**
- **CD**：构建成功后**自动部署、发布**
- **你感受到的就是：不用手动打包、不用手动部署，提交代码就能用最新版本**

如果你愿意，我可以给你画一张**CI/CD 完整流程图（超清晰）**，或者用你的项目技术栈给你讲**你们团队的 CI/CD 长什么样**。
