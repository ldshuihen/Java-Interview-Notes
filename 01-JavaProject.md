## 加密算法

在系统登录场景中，加密算法的核心作用是保障 **用户凭证（如密码）的传输安全** 和 **身份验证信息的完整性/防篡改**，不同环节会选用不同类型的加密算法。以下是常用加密算法及对应应用场景：

### 一、密码存储：单向哈希算法（核心场景）
用户密码不会以明文存储，而是通过单向哈希算法生成摘要后存储，即使数据库泄露，攻击者也难以还原明文密码。

#### 1. 主流算法：Argon2

- **定位**：当前密码哈希的最优选择（2015年密码哈希竞赛冠军）。
- **优势**：
  - 可灵活调节内存占用、计算时间、并行度，能有效抵抗暴力破解和ASIC/FPGA加速攻击。
  - 支持内存硬化（Memory-Hard）设计，比传统算法更难被硬件加速破解。
- **应用场景**：新系统密码存储的首选，如Linux系统、部分Web框架（Spring Security 5+支持）。

#### 2. 经典算法：bcrypt

- **定位**：基于Blowfish对称加密算法的衍生哈希算法，长期广泛应用。
- **优势**：
  - 内置盐值（Salt）机制，避免彩虹表攻击（每个密码的盐值不同）。
  - 可通过“工作因子”调节计算复杂度（工作因子越高，破解越耗时）。
- **应用场景**：传统Web系统、移动端APP的密码存储，兼容性好。

#### 3. 兼容类算法：PBKDF2
- **定位**：基于口令的密钥推导函数，通过迭代哈希增强安全性。
- **优势**：
  - 可结合任意哈希算法（如SHA-256），迭代次数可配置，灵活性高。
  - 被纳入多个安全标准（如RFC 2898），兼容性极强（支持所有主流语言和框架）。
- **应用场景**：需要兼容多平台的系统，如金融系统、老系统升级改造。

#### 4. 淘汰/不推荐算法
- **MD5**：哈希碰撞风险极高，已完全淘汰。
- **SHA-1**：安全性不足，2017年已被破解，禁止用于密码存储。
- **单纯SHA-256/SHA-512**：无内置盐值机制，需手动实现，易因盐值管理不当导致安全问题。

### 二、登录数据传输：对称加密算法（配合HTTPS）
用户登录时，用户名、密码等数据需通过网络传输，通常依赖HTTPS协议保障安全，其核心是对称加密算法。
#### 1. 主流算法：AES
- **定位**：当前最广泛使用的对称加密算法，替代DES。
- **优势**：
  - 加密效率高，适合大量数据传输（如登录请求体）。
  - 密钥长度支持128/192/256位，安全性强（256位AES目前无有效破解方法）。
- **应用场景**：HTTPS传输层加密（TLS协议的核心算法）、登录数据加密传输。

#### 2. 淘汰算法
- **DES**：密钥长度仅56位，安全性不足，已淘汰。
- **3DES**：DES的改进版，效率低，逐渐被AES替代。

### 三、身份令牌（Token）安全：非对称加密/签名算法
登录成功后，系统通常会生成Token（如JWT）用于后续身份验证，需通过非对称算法保障Token的签名防篡改。

- **RSA/ECDSA**：用于 Token 的签名生成和验证（解决 “Token 防篡改” 问题）。

#### 1. 主流算法：RSA
- **定位**：应用最广泛的非对称加密算法。
- **优势**：
  - 支持密钥对（公钥/私钥）操作，私钥签名Token，公钥验证签名，防止Token被篡改。
  - 密钥长度可配置（推荐2048位及以上），安全性稳定。
- **应用场景**：JWT Token签名、OAuth2.0授权流程。

#### 2. 轻量型算法：ECDSA
- **定位**：基于椭圆曲线的非对称签名算法。
- **优势**：
  - 相同安全性下，密钥长度远小于RSA（如256位ECDSA安全性等价于3072位RSA），运算效率更高。
- **应用场景**：移动端APP、物联网设备等资源受限场景的Token签名。

#### 3. 其他算法：EdDSA
- **定位**：基于爱德华兹曲线的改进型ECDSA，安全性和效率更优。
- **优势**：抗攻击性强，运算速度快，密钥生成简单。
- **应用场景**：新兴系统、对性能要求高的分布式系统。

####  .jks 文件和RSA密钥对

生成 Token 时，**RSA 密钥对是核心依赖**，而 `.jks 文件是 RSA 密钥对的存储载体（仅用于 Java 生态）**。两者的关系是“内容”与“容器”的关系，并非二选一，而是根据技术栈选择合适的密钥存储方式。

##### 结论先行：

- **如果是 Java 项目**：推荐将 RSA 密钥对存储在 `.jks 文件` 中，通过 Java 原生的 `KeyStore` 工具读取，更符合 Java 生态的安全规范。
- **如果是多语言项目（如 Java + Python + Go）**：推荐直接使用 **PEM 格式的 RSA 密钥对**（纯文本密钥文件），跨语言兼容性更好。

##### 1. RSA 密钥对：生成 Token 的“核心”

生成 Token（如 JWT）的核心是 **用 RSA 私钥签名，用 RSA 公钥验签**，这一步必须依赖 RSA 密钥对（算法本身的要求）。  
无论密钥存储在何处（.jks 文件、PEM 文件、甚至硬件设备），最终都需要提取出 RSA 私钥和公钥才能完成 Token 的生成和验证。

##### 2. .jks 文件：Java 生态中存储 RSA 密钥的“容器”

.jks 文件本质是 Java 专用的密钥存储格式，它可以安全地存储 RSA 密钥对（包含私钥和公钥证书），并通过密码保护访问。  
在 Java 项目中使用 .jks 文件的优势：
- **安全性**：密钥被加密存储，避免明文暴露，符合 Java 安全规范。
- **便捷性**：Java 原生 `KeyStore` 类可直接读取，无需手动解析密钥格式。
- **可管理性**：一个 .jks 文件可存储多个密钥对（用别名区分），便于集中管理。

例如，Spring Boot 项目从 .jks 文件加载 RSA 私钥生成 JWT：
```java
// 从 .jks 文件加载 RSA 私钥
KeyStore keyStore = KeyStore.getInstance("JKS");
keyStore.load(new FileInputStream("token.jks"), "storepass".toCharArray());
PrivateKey privateKey = (PrivateKey) keyStore.getKey("token-alias", "keypass".toCharArray());

// 用私钥生成 JWT
JWT jwt = JWT.create().setPayload("userId", 123).sign(Keys.hmacShaKeyFor(privateKey.getEncoded()));
```

##### 3. PEM 格式 RSA 密钥：跨语言场景的“首选”

如果项目涉及多语言协作（如 Java 生成 Token，Python 验证 Token），则更适合使用 **PEM 格式的 RSA 密钥对**（纯文本，以 `-----BEGIN RSA PRIVATE KEY-----` 开头）。  
优势：
- **跨语言兼容**：几乎所有语言（Java、Python、Go、Node.js）都支持解析 PEM 格式密钥。
- **操作简单**：直接读取文本文件即可提取密钥，无需依赖 Java 特定工具。

例如，Java 从 PEM 文件加载 RSA 私钥：
```java
// 从 PEM 文件读取 RSA 私钥（使用 Hutool 工具）
String privateKeyStr = FileUtil.readUtf8String("private_key.pem");
PrivateKey privateKey = new RSA(privateKeyStr, null).getPrivateKey();

// 生成 JWT
JWT jwt = JWT.create().setPayload("userId", 123).sign(JWTSignerUtil.createSigner("RS256", privateKey));
```

##### 总结：

- **RSA 密钥对是生成 Token 的“必需品”**，没有它无法完成签名和验签。
- **.jks 文件是 Java 项目中存储 RSA 密钥的“优选容器”**，适合纯 Java 生态。
- **PEM 格式密钥是跨语言场景的“优选”**，兼容性更强。

选择的核心是：**根据项目技术栈（是否纯 Java）选择 RSA 密钥的存储方式**，而非在“RSA 密钥对”和“.jks 文件”之间二选一。

#### .jks文件生成

是的，**这个 `keytool` 命令是在操作系统的命令行（终端）中运行的**，不是在 Java 代码或 IDE 里执行。

##### 1. **在哪里运行？**

- **Windows**：打开 **命令提示符（CMD）** 或 **PowerShell**
- **macOS / Linux**：打开 **终端（Terminal）**

> 💡 前提：你的系统已安装 **JDK**（不是仅 JRE），并且 `keytool` 在系统 PATH 中（通常安装 JDK 后自动配置）。

你可以先验证是否可用：
```bash
keytool -help
```
如果看到帮助信息，说明环境已准备好。

---

##### 2. **命令格式说明**

你提供的命令是 **跨平台的**，但在不同系统中 **换行符写法略有不同**：

###### ✅ Windows（CMD / PowerShell）—— **不要用 `\` 换行**

```cmd
keytool -genkeypair -alias mykey -keyalg RSA -keysize 2048 -sigalg SHA256withRSA -validity 3650 -keystore mykeystore.jks -storepass changeit -keypass changeit -dname "CN=MyApp, OU=Dev, O=MyOrg, L=Beijing, ST=Beijing, C=CN"

keytool -genkeypair -alias jwt -keyalg RSA -keysize 2048 -sigalg SHA256withRSA -validity 3650 -keystore jwt.jks -storepass 123456 -keypass 123456 -dname "CN=MyApp, OU=Dev, O=MyOrg, L=Beijing, ST=Beijing, C=CN"
```

> ⚠️ Windows 的 CMD 不支持 `\` 作为续行符，必须写在一行。

###### ✅ macOS / Linux（Bash / Zsh）—— 支持 `\` 换行

```bash
keytool -genkeypair \
  -alias mykey \
  -keyalg RSA \
  -keysize 2048 \
  -sigalg SHA256withRSA \
  -validity 3650 \
  -keystore mykeystore.jks \
  -storepass changeit \
  -keypass changeit \
  -dname "CN=MyApp, OU=Dev, O=MyOrg, L=Beijing, ST=Beijing, C=CN"
  
keytool -genkeypair \
  -alias jwt \                          # ← 对应 alias: jwt。.jks 是一个“密钥库”，可以存多个密钥对，每个用 别名（alias） 区分。
  -keyalg RSA \
  -keysize 2048 \
  -sigalg SHA256withRSA \
  -keystore jwt.jks \                   # ← 文件名，对应 location: classpath:jwt.jks
  -storepass 123456 \                   # ← 对应 password: 123456
  -keypass 123456 \                     # ← 通常与 storepass 相同
  -validity 3650 \
  -dname "CN=JWT, OU=Auth, O=MyApp"
```

---

##### 3. **执行后会发生什么？**

- 在当前目录生成一个文件：`mykeystore.jks`
- 该文件包含：
  - 一个 RSA 私钥（受密码保护）
  - 一个自签名证书（公钥 + 身份信息）
- 密码为：`changeit`（建议你改成自己的强密码）

---

##### 4. **如何在项目中使用？**

将生成的 `mykeystore.jks` 文件放到你的 Spring Boot 项目中，例如：
```
src/main/resources/mykeystore.jks
```

然后可以在代码中加载（如用于 HTTPS）：
```yaml
# application.yml
server:
  ssl:
    enabled: true
    key-store: classpath:mykeystore.jks
    key-store-password: changeit
    key-password: changeit
    key-alias: mykey
```

> 🔐 注意：**不要将 `.jks` 文件提交到 Git 公开仓库！** 它包含私钥！

---

##### ✅ 小技巧：快速测试

你可以在项目根目录打开终端，直接运行（以 macOS/Linux 为例）：

```bash
cd your-spring-boot-project
keytool -genkeypair \
  -alias jwtkey \
  -keyalg RSA \
  -keysize 2048 \
  -keystore src/main/resources/jwt.jks \
  -storepass mypass123 \
  -keypass mypass123 \
  -dname "CN=JWT, OU=Auth, O=MyApp" \
  -validity 3650
```

这样就直接生成到 `resources` 目录下了。

---

##### ❓常见问题

**Q：提示 `'keytool' 不是内部或外部命令`？**  
A：说明 JDK 未正确安装或 PATH 未配置。  
✅ 解决方案：
- 确认已安装 JDK（如 Oracle JDK 或 OpenJDK）
- 找到 `keytool.exe`（通常在 `C:\Program Files\Java\jdk-xx\bin\`）
- 将该路径加入系统环境变量 `PATH`

---

##### ✅ 总结

| 项目         | 说明                                                       |
| ------------ | ---------------------------------------------------------- |
| **运行位置** | 操作系统命令行（CMD / Terminal）                           |
| **依赖工具** | JDK 自带的 `keytool`                                       |
| **输出文件** | `mykeystore.jks`（二进制密钥库）                           |
| **适用场景** | HTTPS、Java 安全、证书管理等                               |
| **JWT 建议** | 若仅用于 JWT，更推荐直接用 OpenSSL 生成 PEM 文件（更轻量） |

---

如果你只是做 JWT 签名，其实可以跳过 JKS，直接用：
```bash
openssl genrsa -out private_key.pem 2048
openssl rsa -in private_key.pem -pubout -out public_key.pem
```
然后 Hutool 直接读取 PEM，更简单 😊

需要我帮你写一个自动生成 PEM 的 Java 工具类吗？

### 四、登录场景加密算法选型建议

| 环节                | 推荐算法                | 核心考量                                  |
|---------------------|-------------------------|-------------------------------------------|
| 密码存储            | Argon2 > bcrypt > PBKDF2 | 优先选抗破解能力强、配置灵活的算法        |
| 数据传输            | AES（配合HTTPS）        | 效率高、安全性足，依赖TLS协议保障传输层安全|
| Token签名/身份验证  | RSA（通用）、ECDSA（轻量）| 结合系统资源和安全性需求选择              |

### 五、关键安全补充
1. **盐值（Salt）机制**：密码哈希时必须添加随机盐值，避免彩虹表攻击（bcrypt、Argon2内置盐值，PBKDF2需手动实现）。
2. **密钥管理**：非对称算法的私钥（如RSA私钥）需妥善存储（如硬件安全模块HSM、密钥管理服务KMS），禁止明文暴露。
3. **HTTPS强制启用**：登录数据传输必须通过HTTPS，避免中间人攻击，禁止明文传输密码。
4. **定期升级**：随着算力提升，及时升级算法或调整参数（如RSA密钥从1024位升级到2048位，Argon2提高工作因子）。

## 系统登录

Java 后台系统登录的实现思路，核心目标是**验证用户身份、建立会话、保障安全**。根据系统规模、安全要求和架构风格（单体 or 分布式），有多种实现方案。以下是主流的实现思路、技术选型及演进路径：

---

### 一、基于 Session 的传统登录（适合单体应用）

#### 流程：

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant Backend
    participant DB

    User->>Frontend: 输入用户名/密码
    Frontend->>Backend: POST /login {username, password}
    Backend->>DB: 查询用户 + 密码比对（BCrypt）
    alt 验证成功
        Backend->>Backend: 创建 HttpSession
        Backend-->>Frontend: Set-Cookie: JSESSIONID=xxx
        Frontend->>Backend: 后续请求自动携带 Cookie
        Backend->>Backend: 通过 session 验证身份
    else 失败
        Backend-->>Frontend: 401 Unauthorized
    end
```

**关键点：**

- **密码存储**：必须使用 **BCrypt** 等不可逆哈希算法加密存储。
- **Session 管理**：Tomcat 等容器自动管理，`HttpSession` 存用户 ID 或 User 对象。
- **安全性**：
  - 开启 `HttpOnly` 和 `Secure`（HTTPS 下）的 Cookie。
  - 设置合理的 Session 超时时间（如 30 分钟）。
  - 防止 Session Fixation（登录后调用 `request.changeSessionId()`）。

优点：

- 简单、成熟、浏览器自动管理 Cookie。
- 适合内部管理系统（如 ERP、OA）。

缺点：

- **无法水平扩展**：Session 默认存在单机内存，集群需共享 Session（如 Redis）。
- 前后端耦合（依赖 Cookie）。



基于 Session 的传统登录是 Java Web 应用中最经典、最稳定的认证方式，尤其适用于**内部管理系统、单体应用、前后端未完全分离**的场景。下面从 **整体流程、关键代码、安全加固、常见问题** 四个维度，给出一套**完整、可落地、生产级**的实现方案。

---

#### 一、整体流程（含安全细节）

```mermaid
sequenceDiagram
    participant User as 用户
    participant Browser as 浏览器
    participant Server as Java 后端 (Spring Boot)
    participant DB as 数据库

    User->>Browser: 访问 /login 页面
    Browser->>Server: GET /login → 返回登录页
    User->>Browser: 输入用户名/密码，点击登录
    Browser->>Server: POST /doLogin {username, password}
    Server->>DB: 查询用户（根据 username）
    alt 用户不存在 或 密码错误
        Server-->>Browser: 返回错误提示（不暴露具体原因）
    else 验证成功
        Server->>Server: 1. 调用 request.changeSessionId() 防 Session Fixation<br/>2. 将用户信息存入 HttpSession<br/>3. 记录登录日志（IP、时间）
        Server-->>Browser: 302 重定向到 /dashboard<br/>Set-Cookie: JSESSIONID=xxx; HttpOnly; Secure
    end

    Browser->>Server: 访问 /dashboard（自动携带 Cookie）
    Server->>Server: 从 session 获取用户信息，验证是否登录
    alt 未登录（session 为空或过期）
        Server-->>Browser: 302 重定向到 /login
    else 已登录
        Server-->>Browser: 返回页面内容
    end
```



当然可以。以下是**基于 Session 的传统登录**的完整实现流程，用清晰、简洁的文字描述，适用于 Java Web 后台系统（如 Spring Boot、Servlet 等）：

---

##### 一、用户访问登录页面

1. 用户在浏览器中输入系统地址（如 `http://example.com`）。
2. 后端检测用户未登录（Session 为空或无效），自动重定向到登录页 `/login`。
3. 后端返回登录页面 HTML（含用户名、密码输入框和登录按钮）。

---

##### 二、用户提交登录请求

1. 用户在登录页面输入用户名和密码，点击“登录”按钮。
2. 浏览器将用户名和密码通过 **POST 请求**发送到后端登录接口（如 `/doLogin`）。

---

##### 三、后端验证用户身份

1. 后端接收到登录请求，从请求中提取用户名和密码。
2. 根据用户名查询数据库，获取对应的用户记录（包括加密后的密码）。
3. 使用 **BCrypt 等安全哈希算法**，将用户输入的密码与数据库中存储的加密密码进行比对。
   - 如果用户不存在，或密码不匹配，则登录失败。
   - 如果验证通过，继续下一步。

---

##### 四、创建并绑定用户会话（Session）

1. **防止 Session Fixation 攻击**：调用 `request.changeSessionId()` 生成新的 Session ID，替换原有（可能被攻击者预设的）ID。
2. 获取当前请求的 `HttpSession` 对象（若不存在则自动创建）。
3. 将用户关键信息（如用户 ID、用户名、角色等）存入 Session 中，例如：
   ```java
   session.setAttribute("userId", user.getId());
   session.setAttribute("username", user.getUsername());
   ```
4. （可选）记录登录日志，包括用户 ID、登录时间、IP 地址等。

---

##### 五、返回登录结果

- **登录成功**：后端返回 **302 重定向**，跳转到系统首页或仪表盘页面（如 `/dashboard`）。  
  同时，服务器在响应头中自动设置 Cookie：
  ```
  Set-Cookie: JSESSIONID=abc123; Path=/; HttpOnly; Secure
  ```
  浏览器会自动保存该 Cookie，并在后续请求中自动携带。

- **登录失败**：返回登录页面，并提示“用户名或密码错误”（不暴露具体是用户名错还是密码错，防止信息泄露）。

---

##### 六、后续请求的身份验证

1. 用户访问受保护的页面（如 `/dashboard`）。
2. 浏览器自动在请求头中携带 `Cookie: JSESSIONID=abc123`。
3. 后端根据 JSESSIONID 找到对应的 Session。
4. 检查 Session 中是否存在用户信息：
   - 如果存在，说明用户已登录，正常处理请求。
   - 如果不存在（Session 过期、未登录、被清除），则重定向回 `/login` 页面。

---

##### 七、用户退出登录

1. 用户点击“退出”按钮，浏览器请求 `/logout` 接口。
2. 后端执行：
   - 调用 `session.invalidate()` 销毁当前 Session。
   - （可选）清除相关 Cookie。
3. 重定向回登录页，提示“已安全退出”。

---

##### 八、安全补充说明（生产环境必备）

- **密码存储**：必须使用 BCrypt、SCrypt 等不可逆加密算法，禁止明文或 MD5。
- **传输安全**：生产环境必须启用 HTTPS，防止密码和 Session ID 被窃听。
- **Cookie 安全**：
  - 设置 `HttpOnly`：防止 JavaScript 读取 Cookie（防 XSS）。
  - 设置 `Secure`：仅在 HTTPS 下传输 Cookie。
- **防暴力破解**：对同一 IP 或用户名，限制登录失败次数（如 5 次后锁定 15 分钟）。
- **Session 超时**：设置合理的 Session 有效期（如 30 分钟无操作自动过期）。

---

##### 总结一句话：

> 基于 Session 的登录，本质是：**验证用户 → 创建会话 → 用 Cookie 绑定会话 → 后续请求靠 Session 识别身份**。

这种方式简单可靠，特别适合内部管理系统、单体应用等场景。

#### 二、核心代码实现（Spring Boot + Thymeleaf）

> 假设使用 Spring Boot 3.x + Spring Security（推荐）或手动实现（教学目的）

##### 方案 A：**推荐 —— 使用 Spring Security（自动处理 Session、CSRF、安全头）**

###### 1. 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity6</artifactId>
</dependency>
```

###### 2. 用户实体（User）

```java
public class User {
    private Long id;
    private String username;
    private String password; // 已 BCrypt 加密
    private String role; // 如 "ADMIN", "USER"
    // getter/setter
}
```

###### 3. 自定义 UserDetailsService（对接数据库）

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username);
        if (user == null) {
            throw new UsernameNotFoundException("用户不存在");
        }
        // Spring Security 的 UserDetails 实现
        return org.springframework.security.core.userdetails.User
            .withUsername(user.getUsername())
            .password(user.getPassword()) // 必须是 BCrypt 加密后的
            .roles(user.getRole())
            .build();
    }
}
```

###### 4. Security 配置（启用表单登录 + Session 管理）

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(); // 密码加密器
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/login", "/css/**", "/js/**").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")           // 自定义登录页
                .loginProcessingUrl("/doLogin") // 登录提交地址
                .defaultSuccessUrl("/dashboard", true) // 登录成功跳转
                .failureUrl("/login?error=true")       // 登录失败跳转
                .permitAll()
            )
            .logout(logout -> logout
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login?logout=true")
                .invalidateHttpSession(true)
                .clearAuthentication(true)
                .deleteCookies("JSESSIONID")
            )
            .sessionManagement(session -> session
                .maximumSessions(1) // 同一用户只允许一个会话（可选）
                .maxSessionsPreventsLogin(false) // 新登录踢掉旧会话
            )
            .csrf().disable(); // 若用 Thymeleaf 表单，建议开启 CSRF（默认开启），此处为简化演示关闭

        return http.build();
    }
}
```

###### 5. 登录页面（`templates/login.html`）

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>登录</title>
</head>
<body>
    <h2>系统登录</h2>
    
    <!-- 错误提示 -->
    <div th:if="${param.error}">
        <p style="color:red;">用户名或密码错误</p>
    </div>
    <div th:if="${param.logout}">
        <p>您已成功退出登录</p>
    </div>

    <form th:action="@{/doLogin}" method="post">
        <div>
            <label>用户名:</label>
            <input type="text" name="username" required />
        </div>
        <div>
            <label>密码:</label>
            <input type="password" name="password" required />
        </div>
        <button type="submit">登录</button>
    </form>
</body>
</html>
```

###### 6. 受保护页面（`/dashboard`）

```java
@Controller
public class DashboardController {
    @GetMapping("/dashboard")
    public String dashboard(Model model, HttpSession session) {
        // Spring Security 已自动验证登录状态
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        model.addAttribute("username", auth.getName());
        return "dashboard";
    }
}
```

> ✅ **Spring Security 自动完成**：
> - 密码比对（使用 `PasswordEncoder`）
> - Session 创建与管理
> - 登录成功/失败跳转
> - 退出登录清理
> - 防重复提交（CSRF Token，若开启）

---

##### 方案 B：**手动实现（教学/轻量级场景）**

> 不推荐生产使用，但有助于理解原理

###### 1. 登录 Controller

```java
@PostMapping("/doLogin")
public String doLogin(
        @RequestParam String username,
        @RequestParam String password,
        HttpServletRequest request,
        HttpServletResponse response,
        RedirectAttributes redirectAttributes) {

    // 1. 查询用户
    User user = userService.findByUsername(username);
    if (user == null || !passwordEncoder.matches(password, user.getPassword())) {
        redirectAttributes.addFlashAttribute("error", "用户名或密码错误");
        return "redirect:/login";
    }

    // 2. 防 Session Fixation：更换 session ID
    request.changeSessionId();

    // 3. 存入 session
    HttpSession session = request.getSession();
    session.setAttribute("user", user);
    session.setMaxInactiveInterval(30 * 60); // 30 分钟超时

    // 4. 记录登录日志（可选）
    logService.logLogin(user.getId(), getClientIP(request));

    return "redirect:/dashboard";
}
```

###### 2. 登录拦截器（验证是否登录）

```java
@Component
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        HttpSession session = request.getSession(false);
        if (session == null || session.getAttribute("user") == null) {
            response.sendRedirect("/login");
            return false;
        }
        return true;
    }
}

// 注册拦截器
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loginInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/login", "/doLogin", "/static/**");
    }
}
```

---

#### 三、安全加固措施（生产必备）

| 风险                 | 解决方案                                                |
| -------------------- | ------------------------------------------------------- |
| **Session Fixation** | 登录成功后调用 `request.changeSessionId()`              |
| **密码明文传输**     | 全站 HTTPS（生产环境强制）                              |
| **暴力破解**         | 登录失败 5 次后锁定账号 15 分钟（用 Redis 计数）        |
| **XSS 攻击**         | 模板引擎自动转义（Thymeleaf 默认安全）                  |
| **CSRF 攻击**        | Spring Security 默认开启 CSRF（表单需带 `_csrf` token） |
| **Session 劫持**     | Cookie 设置 `HttpOnly` + `Secure`（Spring Boot 默认）   |
| **敏感信息泄露**     | 错误提示统一为“用户名或密码错误”，不区分具体原因        |

> 🔒 **Cookie 安全设置（Spring Boot 自动处理）**：
> ```java
> server.servlet.session.cookie.http-only=true
> server.servlet.session.cookie.secure=true  # 仅 HTTPS 传输
> server.servlet.session.timeout=1800       # 30 分钟
> ```

---

#### 四、集群环境下的 Session 共享（扩展）

单机没问题，但集群需共享 Session：

##### 方案：**Spring Session + Redis**

###### 1. 依赖

```xml
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

###### 2. 配置

```yaml
spring:
  redis:
    host: localhost
    port: 6379
  session:
    store-type: redis
    timeout: 30m
```

> ✅ 效果：所有节点共享 Redis 中的 Session，用户登录后访问任意节点都有效。

---

#### 五、总结：基于 Session 登录的核心要点

| 项目         | 说明                                                   |
| ------------ | ------------------------------------------------------ |
| **认证方式** | 用户名 + 密码（BCrypt 加密）                           |
| **会话管理** | `HttpSession`（Tomcat 内存 or Redis 共享）             |
| **安全防护** | HTTPS + HttpOnly Cookie + 防爆破 + 防 Session Fixation |
| **适用场景** | 内部管理系统、ERP、OA、CMS 等                          |
| **优势**     | 简单、稳定、浏览器自动管理 Cookie                      |
| **劣势**     | 集群需共享 Session，不适合 App/小程序                  |

---

> 💡 **一句话口诀**：  
> **“密码加密存，登录换 session，Cookie 设安全，集群用 Redis。”**

如果你需要：
- 完整可运行的 GitHub 示例项目
- 集成图形验证码（Kaptcha）
- 登录失败锁定 + 邮箱通知
- 多角色权限控制（RBAC）

我可以继续为你扩展。是否需要？

---

### 二、主流实现思路（前后端分离 / 分布式）

### 2. **基于 Token 的无状态登录（JWT 为主）**

#### 流程：

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant Backend
    participant DB

    User->>Frontend: 输入用户名/密码
    Frontend->>Backend: POST /login {username, password}
    Backend->>DB: 验证用户
    alt 成功
        Backend->>Backend: 生成 JWT（含用户ID、角色、过期时间）
        Backend-->>Frontend: { "token": "xxx" }
        Frontend->>Frontend: 存储 token（localStorage / sessionStorage）
        Frontend->>Backend: 后续请求在 Header 中携带 Authorization: Bearer xxx
        Backend->>Backend: 验证 JWT 签名 + 过期时间
    else 失败
        Backend-->>Frontend: 401
    end
```

##### 关键点：

- **JWT 结构**：
  ```json
  {
    "sub": "user123",
    "roles": ["admin"],
    "exp": 1730000000,
    "iat": 1729996400
  }
  ```
- **签名算法**：HS256（对称）或 RS256（非对称，更安全）。
- **刷新机制**：
  - Access Token（短有效期，如 30 分钟）
  - Refresh Token（长有效期，如 7 天），用于获取新 Access Token。
- **安全性**：
  - Token 存储在前端需防 XSS（避免存 Cookie，或用 `HttpOnly` Cookie 存 Refresh Token）。
  - 敏感操作需二次验证（如修改密码）。
  - **无法主动注销**（除非引入黑名单或短期 Token）。

##### 优点：

- 无状态，天然支持分布式、微服务。
- 前后端完全解耦。
- 适合 Web + App + 小程序多端。

##### 缺点：

- Token 一旦泄露，在过期前都有效（需配合短有效期 + 刷新机制）。
- 无法像 Session 一样服务端主动失效（需额外设计）。

#### **完整实现流程**

基于 Token 的无状态登录（以 JWT 为主）是现代 Web 应用（尤其是前后端分离架构）中最主流的身份认证方式。其核心思想是：**用户登录成功后，服务端签发一个包含用户身份信息的 Token，客户端在后续请求中携带该 Token，服务端通过验证 Token 的合法性来识别用户，全程无需保存会话状态**。

以下是该方案的**完整实现流程**，以 Java（Spring Boot） + JWT 为例，用清晰、结构化的文字描述：

---

##### 一、用户发起登录请求

1. 用户在前端（Web / App / 小程序）输入用户名和密码。
2. 前端通过 HTTPS 发送 **POST 请求** 到后端登录接口（如 `/api/auth/login`），携带 JSON 数据：
   ```json
   {
     "username": "admin",
     "password": "123456"
   }
   ```

---

##### 二、后端验证用户身份

1. 后端接收到登录请求，解析出用户名和密码。
2. 根据用户名查询数据库，获取用户记录（包括加密后的密码和用户角色等信息）。
3. 使用 **BCryptPasswordEncoder** 等安全算法，比对用户输入的密码与数据库中存储的加密密码：
   - 若用户不存在、密码错误、账户被禁用等，返回 `401 Unauthorized` 错误。
   - 若验证通过，继续下一步。

---

##### 三、生成并返回 JWT Token

1. 后端使用**私钥**（对称密钥如字符串，或非对称密钥如 RSA 私钥）创建一个 JWT（JSON Web Token）。
2. JWT 的 **Payload（载荷）** 中通常包含以下声明（claims）：
   - `sub`（subject）：用户唯一标识（如用户 ID）
   - `username`：用户名（可选）
   - `roles`：用户角色列表（如 `["ADMIN", "USER"]`）
   - `iat`（Issued At）：签发时间（时间戳）
   - `exp`（Expiration Time）：过期时间（如 30 分钟后）
3. 对 Header + Payload 进行 Base64Url 编码，并用指定算法（如 HS256）签名，生成最终 Token。
4. 后端将 Token 封装在 JSON 响应中返回给前端：
   ```json
   {
     "code": 200,
     "message": "登录成功",
     "data": {
       "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.xxxxx",
       "tokenType": "Bearer",
       "expiresIn": 1800
     }
   }
   ```

> ✅ **注意**：  
> - **Access Token 有效期应较短**（如 15~60 分钟），降低泄露风险。  
> - 可同时返回一个 **Refresh Token**（有效期较长，如 7 天），用于静默刷新 Access Token（见后文）。

---

##### 四、前端存储 Token

1. 前端收到响应后，将 `accessToken` 存储在安全位置：
   - **Web 应用**：推荐存入 `HttpOnly` Cookie（防 XSS）或内存变量（结合路由守卫）；  
     若存 `localStorage`，需防范 XSS 攻击。
   - **App / 小程序**：存入本地安全存储（如 Android Keystore、iOS Keychain）。
2. 后续所有需要认证的请求，前端在 **HTTP 请求头（Header）** 中添加：
   ```
   Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.xxxxx
   ```

---

##### 五、后续请求的身份验证（Token 校验）

1. 用户访问受保护接口（如 `/api/user/profile`）。
2. 前端自动在请求头中携带 `Authorization: Bearer <token>`。
3. 后端通过**拦截器（Interceptor）或过滤器（Filter）** 拦截请求：
   - 从 Header 中提取 Token。
   - 验证 Token 的合法性，包括：
     - 签名是否正确（防止篡改）
     - 是否过期（检查 `exp`）
     - 是否被吊销（可选，需配合黑名单机制）
4. 若 Token 有效：
   - 从 Payload 中解析出用户 ID、角色等信息。
   - 将用户身份信息存入当前线程上下文（如 Spring Security 的 `SecurityContext`）。
   - 放行请求，业务逻辑可直接获取当前用户。
5. 若 Token 无效（过期、签名错误、缺失等）：
   - 返回 `401 Unauthorized`，前端跳转到登录页或尝试刷新 Token。

---

##### 六、Token 刷新机制（可选但推荐）

为避免用户频繁登录，引入 **Refresh Token**：

1. 登录成功时，后端同时返回：
   - `accessToken`（短有效期，如 30 分钟）
   - `refreshToken`（长有效期，如 7 天，通常存储在服务端或安全 Cookie 中）
2. 当 `accessToken` 过期时，前端调用 `/api/auth/refresh` 接口，携带 `refreshToken`。
3. 后端验证 `refreshToken` 有效性（检查是否过期、是否属于该用户、是否被使用过）。
4. 若有效，签发新的 `accessToken`（和可选的新 `refreshToken`）返回给前端。
5. 前端用新 Token 替换旧 Token，继续正常请求。

> 🔒 **安全建议**：
> - Refresh Token 应存储在 `HttpOnly + Secure` Cookie 中，避免 XSS。
> - 每次使用 Refresh Token 后可使其失效（一次性），并发放新 Token（防重放）。

---

##### 七、用户登出（Logout）

由于 JWT 是无状态的，**服务端无法主动使 Token 失效**。常见处理方式：

1. **前端删除 Token**：清除本地存储的 `accessToken` 和 `refreshToken`。
2. **服务端维护短期黑名单（可选）**：
   - 用户登出时，将 `accessToken` 的 `jti`（JWT ID）或用户 ID + 过期时间存入 Redis，设置 TTL 为 Token 剩余有效期。
   - 拦截器校验 Token 时，额外检查是否在黑名单中。
   - 适用于对安全性要求极高的系统（如金融）。

> ⚠️ 注意：若不使用黑名单，登出后旧 Token 在过期前仍可使用（但因前端已删除，一般无影响）。

---

##### 八、安全加固措施（生产必备）

| 风险           | 防护措施                                                     |
| -------------- | ------------------------------------------------------------ |
| **Token 泄露** | 使用 HTTPS；Access Token 短有效期；敏感操作需二次验证        |
| **XSS 攻击**   | Token 不存 `localStorage`；使用 `HttpOnly` Cookie 存 Refresh Token |
| **CSRF 攻击**  | JWT 无状态，天然免疫 CSRF（因不依赖 Cookie 自动发送）        |
| **重放攻击**   | 使用 `jti`（JWT ID）+ 黑名单；Refresh Token 一次性使用       |
| **暴力破解**   | 登录接口限流、失败次数限制、验证码                           |
| **算法降级**   | 禁用 `none` 算法；使用强签名算法（HS256/RS256）              |

---

##### 九、适用场景总结

✅ **适合**：
- 前后端分离项目（Vue/React + Spring Boot）
- 移动 App、小程序后端
- 微服务架构（各服务独立验证 Token）
- 多端统一认证（Web、App、API）

❌ **不适合**：
- 需要服务端强控制会话（如强制下线所有设备）
- 极低延迟要求（JWT 解析有轻微开销）
- 无法使用 HTTPS 的环境（Token 明文传输极危险）

---

##### 十、一句话总结流程

> 用户登录 → 服务端验证 → 签发 JWT → 前端存储 → 后续请求携带 Token → 服务端验签 → 识别用户身份 → 无状态完成认证。

---

如需，我可提供：
- Spring Boot + JWT 完整代码模板（含拦截器、工具类、刷新机制）
- 基于 Redis 的 Token 黑名单实现
- 前端 Axios 拦截器自动处理 Token 刷新

#### 代码实现

当然可以！以下是一个 **基于 Spring Boot + JWT 的完整无状态登录系统** 的代码实现，包含：

- 用户登录（返回 Access Token + Refresh Token）
- Token 验证拦截器
- Token 刷新接口
- 安全配置（BCrypt 密码、HTTPS 建议、CORS）
- 简易用户模拟（无数据库，便于运行）

> ✅ 适用于前后端分离项目，可直接运行测试。

---

##### 一、项目依赖（`pom.xml`）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.0</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>jwt-demo</artifactId>
    <version>1.0.0</version>
    <name>jwt-demo</name>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <!-- Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Security (可选，本例手动实现，不依赖 Spring Security) -->
        <!-- 若想用 Spring Security + JWT，可引入，但此处为清晰展示原理，不使用 -->

        <!-- JWT -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.12.5</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>0.12.5</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>0.12.5</version>
            <scope>runtime</scope>
        </dependency>

        <!-- Lombok (简化代码) -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

###### `jjwt` 这些依赖

---

 **Java 标准库（JDK）本身不提供 JWT 的实现**

Java SE（标准版）**没有内置对 JWT（JSON Web Token）的原生支持**。  
也就是说，**你不能只靠 `java.util` 或 JDK 自带的类库来生成或解析 JWT**。

JWT 是一个 **开放标准（RFC 7519）**，它定义了 Token 的结构（Header.Payload.Signature）和编码方式（Base64Url + 签名），但 **JDK 并未实现这个标准**。

---

那么为什么需要引入 `jjwt` 这些依赖？

`io.jsonwebtoken:jjwt-*` 是目前 **Java 生态中最主流、官方推荐的 JWT 库**（由 JWT 官方团队维护），它提供了：

| 功能               | 说明                                      |
| ------------------ | ----------------------------------------- |
| ✅ 生成 JWT         | 自动处理 Header、Payload、签名            |
| ✅ 解析 & 验证 JWT  | 自动校验签名、过期时间、算法等            |
| ✅ 支持多种签名算法 | HS256、HS512、RS256（RSA）等              |
| ✅ 安全性保障       | 防止常见漏洞（如算法降级攻击）            |
| ✅ 与 Jackson 集成  | 自动序列化/反序列化 Claims 中的 JSON 对象 |

如果你**不引入任何第三方库**，你将需要：

- 手动拼接 JSON 字符串
- 手动做 Base64Url 编码（注意：不是普通 Base64！）
- 手动实现 HMAC-SHA256 签名
- 手动解析 Token、验证时间、防止篡改
- 处理各种边界情况和安全漏洞

这不仅**极其繁琐**，而且**极易出错**（比如 Base64Url 的 `+`/`/` 替换、`=` 填充处理），**强烈不建议自己造轮子**。

---

 🆚 有没有其他替代库？

有，但 `jjwt` 是最推荐的：

| 库                         | 说明                                                    |
| -------------------------- | ------------------------------------------------------- |
| **`io.jsonwebtoken:jjwt`** | ✅ **官方推荐**，活跃维护，文档完善，Spring 官方示例常用 |
| `com.auth0:java-jwt`       | 也不错，由 Auth0 提供，但生态略小                       |
| 自己实现                   | ❌ 不推荐，安全风险高                                    |

> ⚠️ 注意：旧版 `jjwt`（如 0.9.x）是单个 jar，从 **0.10+ 开始拆分为 `api` / `impl` / `jackson`** 三个模块，这是为了模块化和兼容 JPMS（Java Platform Module System）。

---

 📦 为什么需要三个依赖？

```xml
<!-- 1. API 接口定义（编译时需要） -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.5</version>
</dependency>

<!-- 2. 默认实现（运行时需要） -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.5</version>
    <scope>runtime</scope>
</dependency>

<!-- 3. JSON 序列化支持（用 Jackson 处理 Claims） -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.12.5</version>
    <scope>runtime</scope>
</dependency>
```

- **`jjwt-api`**：你的代码编译时依赖的接口（比如 `Jwts.builder()`）。
- **`jjwt-impl`**：实际的 JWT 签名/解析逻辑（运行时加载）。
- **`jjwt-jackson`**：因为 JWT 的 Payload 是 JSON，需要用 Jackson（或 Gson）来处理对象 ↔ JSON 转换。Spring Boot 默认用 Jackson，所以选这个。

> 💡 如果你用的是 **Gson**，则应引入 `jjwt-gson` 而不是 `jjwt-jackson`。

---

 ✅ 总结

| 问题                       | 回答                                                         |
| -------------------------- | ------------------------------------------------------------ |
| **Java 自带 JWT 支持吗？** | ❌ 没有，JDK 不包含 JWT 实现                                  |
| **必须引入 `jjwt` 吗？**   | ✅ **是的**，除非你愿意自己实现（不推荐）                     |
| **能只引一个依赖吗？**     | ❌ 从 0.10+ 版本开始，必须至少引入 `api` + `impl` + JSON 适配器（共 3 个） |
| **有没有轻量替代？**       | 有（如 Auth0 的库），但 `jjwt` 是事实标准，最安全可靠        |

---

 🔐 安全提示

自己实现 JWT 极易引入安全漏洞，例如：

- 忽略 `alg: none` 攻击
- 使用弱密钥
- 未验证 `exp`（过期时间）
- Base64Url 编码错误导致 Token 无效

因此，**强烈建议使用成熟库如 `jjwt`**。

###### Hutool包

非常好的问题！我们来详细对比 **Hutool 的 JWT 功能** 和 **JJWT（`io.jsonwebtoken:jjwt`）**，看看是否可以用 Hutool 替代。

---

 ✅ 简短回答：

> **Hutool 从 v5.8.0 开始确实内置了 JWT 支持**，**可以用于简单场景**，但 **功能、安全性和标准兼容性不如 JJWT**。  
> **如果你的项目已经引入 Hutool，且需求简单（如仅用 HS256 + 简单 Claims）**，可以用它避免额外依赖；  
> **但如果是生产级系统、需要高安全性或复杂功能（如 RSA、刷新机制、严格校验）**，**仍推荐使用 JJWT**。

---

###### 🔍 详细对比

| 特性                              | **Hutool JWT**                                              | **JJWT (`io.jsonwebtoken`)**                                 |
| --------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| **是否支持 JWT 标准（RFC 7519）** | ✅ 基本支持                                                  | ✅ 完整、严格遵循                                             |
| **签名算法支持**                  | HS256、HS384、HS512（对称）<br>✅ 不支持 RSA/ECDSA（非对称） | ✅ HS256/384/512<br>✅ RS256/384/512（RSA）<br>✅ ES256（椭圆曲线） |
| **自动校验 `exp`、`nbf`、`iat`**  | ✅ 支持（需手动开启）                                        | ✅ 默认自动校验，可配置                                       |
| **防 `alg: none` 攻击**           | ✅ 已修复（v5.8.10+）                                        | ✅ 原生防护，久经考验                                         |
| **Claims 自定义对象序列化**       | ❌ 仅支持 `Map<String, Object>`<br>不支持复杂对象自动转 JSON | ✅ 通过 Jackson/Gson 支持任意 POJO                            |
| **Token 解析异常处理**            | 较简单                                                      | 详细的异常类型（`ExpiredJwtException`, `SignatureException` 等） |
| **社区与维护**                    | 国产优秀工具库，活跃                                        | JWT 官方团队维护，全球广泛使用                               |
| **Spring Boot 集成友好度**        | 一般                                                        | 极高（大量官方/社区示例）                                    |
| **依赖体积**                      | `hutool-all` ≈ 3MB（包含所有工具）                          | `jjwt-*` 合计 ≈ 300KB（轻量）                                |

---

###### 🧪 Hutool JWT 使用示例

如果你决定用 Hutool，代码大致如下：

```java
import cn.hutool.jwt.JWT;
import cn.hutool.jwt.JWTUtil;
import cn.hutool.jwt.signers.HMacJWTSigner;

// 生成 Token
String key = "mySecretKey1234567890123456789012"; // 至少 32 字节
String token = JWT.create()
    .setPayload("userId", "1")
    .setExpiresAt(System.currentTimeMillis() + 30 * 60 * 1000) // 30分钟
    .sign(new HMacJWTSigner(key, "HmacSHA256"))
    .toString();

// 验证 Token
boolean isValid = JWTUtil.verify(token, key.getBytes(), "HmacSHA256");
if (isValid) {
    JWT jwt = JWT.of(token);
    String userId = (String) jwt.getPayload("userId");
}
```

> ⚠️ 注意：Hutool 的 JWT **不自动校验过期时间**！你需要手动检查：
> ```java
> long exp = (Long) jwt.getPayload("exp");
> if (exp < System.currentTimeMillis()) { /* 已过期 */ }
> ```

---

🚫 Hutool JWT 的主要限制

1. **不支持非对称加密（RSA）**  
   → 无法用于 OAuth2、OpenID Connect 等需要公私钥的场景。

2. **Claims 不能直接放对象**  
   → 你无法像 JJWT 那样直接 `setClaims(user)`，必须手动转 Map。

3. **缺少细粒度异常**  
   → 无法区分“签名错误”和“Token 过期”，调试困难。

4. **`hutool-all` 体积大**  
   → 如果你只是为了 JWT 而引入整个 Hutool（3MB），有点“杀鸡用牛刀”。  
     （可改用 `hutool-jwt` 模块，但需 Hutool ≥ 5.8.0）

---

✅ 何时可以用 Hutool 替代 JJWT？

- 你的项目**已经重度使用 Hutool**（如 `HutoolHttp`, `HutoolJson` 等）
- 只需要**简单的 HS256 签名**
- **不需要复杂 Claims 或非对称加密**
- **内部系统，安全性要求不高**

---

🛡️ 何时必须用 JJWT？

- **生产环境 Web/API 服务**
- 需要 **RSA 签名**（如对接微信、支付宝、Auth0）
- 需要 **严格的 Token 校验**（自动过期、算法锁定等）
- 使用 **Spring Security 集成**
- 团队遵循 **行业最佳实践**

---

###### 🔚 结论

| 场景                              | 推荐方案          |
| --------------------------------- | ----------------- |
| 学习 / 小型内部工具 / 已用 Hutool | ✅ 可用 Hutool JWT |
| 正式项目 / 对外 API / 高安全要求  | ✅ **必须用 JJWT** |

> 💡 **建议**：即使你用了 Hutool，**JWT 认证这种核心安全功能，仍建议单独引入 JJWT**，以确保可靠性和可维护性。

---

如果你希望我提供一个 **基于 Hutool JWT 的完整登录示例**，也可以告诉我！

---

##### 二、配置类

###### 1. JWT 工具类（`JwtUtil.java`）

```java
package com.example.jwtdemo.util;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.security.Key;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

@Component
public class JwtUtil {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.access-token-expiration}")
    private long accessTokenExpiration; // 单位：毫秒

    @Value("${jwt.refresh-token-expiration}")
    private long refreshTokenExpiration;

    private Key getSignInKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }

    public String generateToken(String subject, Map<String, Object> claims) {
        return Jwts.builder()
                .setClaims(claims)
                .setSubject(subject)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + accessTokenExpiration))
                .signWith(getSignInKey(), SignatureAlgorithm.HS256)
                .compact();
    }

    public String generateAccessToken(String userId) {
        return generateToken(userId, new HashMap<>());
    }

    public String generateRefreshToken(String userId) {
        return Jwts.builder()
                .setSubject(userId)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + refreshTokenExpiration))
                .signWith(getSignInKey(), SignatureAlgorithm.HS256)
                .compact();
    }

    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder().setSigningKey(getSignInKey()).build().parseClaimsJws(token);
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    public String extractUserId(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(getSignInKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
    }
}
```

---

###### 2. 配置文件（`application.yml`）

```yaml
server:
  port: 8080

jwt:
  secret: ${JWT_SECRET:base64EncodedSecretKeyMustBeAtLeast32BytesLong!} # 至少 32 字节
  access-token-expiration: 1800000   # 30 分钟（毫秒）
  refresh-token-expiration: 604800000 # 7 天

# 生成 base64 secret 的方法（Java）：
# System.out.println(java.util.Base64.getEncoder().encodeToString("mySecretKey1234567890123456789012".getBytes()));
```

---

##### 三、DTO 和响应封装

###### 1. 登录请求（`LoginRequest.java`）

```java
package com.example.jwtdemo.dto;

import lombok.Data;

@Data
public class LoginRequest {
    private String username;
    private String password;
}
```

###### 2. 登录响应（`AuthResponse.java`）

```java
package com.example.jwtdemo.dto;

import lombok.Data;

@Data
public class AuthResponse {
    private String accessToken;
    private String refreshToken;
    private Long expiresIn; // 秒
}
```

###### 3. 通用响应（`ApiResponse.java`）

```java
package com.example.jwtdemo.dto;

import lombok.Data;

@Data
public class ApiResponse<T> {
    private int code;
    private String message;
    private T data;

    public static <T> ApiResponse<T> success(T data) {
        ApiResponse<T> res = new ApiResponse<>();
        res.code = 200;
        res.message = "success";
        res.data = data;
        return res;
    }

    public static <T> ApiResponse<T> error(String message) {
        ApiResponse<T> res = new ApiResponse<>();
        res.code = 401;
        res.message = message;
        return res;
    }
}
```

---

##### 四、模拟用户服务（无数据库）

###### 1. 用户实体（`User.java`）

```java
package com.example.jwtdemo.model;

import lombok.Data;

@Data
public class User {
    private String id;
    private String username;
    private String password; // BCrypt encoded
}
```

###### 2. 用户服务（`UserService.java`）

```java
package com.example.jwtdemo.service;

import com.example.jwtdemo.model.User;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;

@Service
public class UserService {

    // 模拟数据库（生产环境应使用 JPA/MyBatis）
    private static final Map<String, User> USER_DB = new HashMap<>();

    private final BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();

    public UserService() {
        // 初始化一个测试用户：username=admin, password=123456
        String encodedPassword = passwordEncoder.encode("123456");
        USER_DB.put("admin", new User() {{
            setId("1");
            setUsername("admin");
            setPassword(encodedPassword);
        }});
    }

    public User findByUsername(String username) {
        return USER_DB.get(username);
    }

    public boolean validatePassword(String rawPassword, String encodedPassword) {
        return passwordEncoder.matches(rawPassword, encodedPassword);
    }
}
```

---

##### 五、认证控制器（`AuthController.java`）

```java
package com.example.jwtdemo.controller;

import com.example.jwtdemo.dto.ApiResponse;
import com.example.jwtdemo.dto.AuthResponse;
import com.example.jwtdemo.dto.LoginRequest;
import com.example.jwtdemo.model.User;
import com.example.jwtdemo.service.UserService;
import com.example.jwtdemo.util.JwtUtil;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/auth")
@CrossOrigin(origins = "*") // 生产环境应限制来源
public class AuthController {

    private final UserService userService;
    private final JwtUtil jwtUtil;

    @Value("${jwt.access-token-expiration}")
    private long accessTokenExpiration;

    public AuthController(UserService userService, JwtUtil jwtUtil) {
        this.userService = userService;
        this.jwtUtil = jwtUtil;
    }

    @PostMapping("/login")
    public ApiResponse<AuthResponse> login(@RequestBody LoginRequest request) {
        User user = userService.findByUsername(request.getUsername());
        if (user == null || !userService.validatePassword(request.getPassword(), user.getPassword())) {
            return ApiResponse.error("用户名或密码错误");
        }

        String accessToken = jwtUtil.generateAccessToken(user.getId());
        String refreshToken = jwtUtil.generateRefreshToken(user.getId());

        AuthResponse response = new AuthResponse();
        response.setAccessToken(accessToken);
        response.setRefreshToken(refreshToken);
        response.setExpiresIn(accessTokenExpiration / 1000); // 转为秒

        return ApiResponse.success(response);
    }

    @PostMapping("/refresh")
    public ApiResponse<AuthResponse> refreshToken(@RequestBody String refreshToken) {
        if (!jwtUtil.validateToken(refreshToken)) {
            return ApiResponse.error("无效的 Refresh Token");
        }

        String userId = jwtUtil.extractUserId(refreshToken);
        // 可在此处校验用户是否仍有效（如未被禁用）

        String newAccessToken = jwtUtil.generateAccessToken(userId);

        AuthResponse response = new AuthResponse();
        response.setAccessToken(newAccessToken);
        response.setRefreshToken(refreshToken); // 或生成新的 refreshToken
        response.setExpiresIn(accessTokenExpiration / 1000);

        return ApiResponse.success(response);
    }
}
```

---

##### 六、Token 验证拦截器（`JwtAuthInterceptor.java`）

作用：拦截请求，验证 JWT Token 是否有效。

```java
package com.example.jwtdemo.interceptor;

import com.example.jwtdemo.util.JwtUtil;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;

@Component
public class JwtAuthInterceptor implements HandlerInterceptor {

    @Autowired
    private JwtUtil jwtUtil;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String authHeader = request.getHeader("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return false;
        }

        String token = authHeader.substring(7); // 去掉 "Bearer "

        if (!jwtUtil.validateToken(token)) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return false;
        }

        // 可选：将用户 ID 存入 request attribute，供 Controller 使用
        String userId = jwtUtil.extractUserId(token);
        request.setAttribute("userId", userId);

        return true;
    }
}
```

---

##### 七、注册拦截器（`WebConfig.java`）

作用：将 `JwtAuthInterceptor` 注册到 Spring MVC 的拦截器链中。

```java
package com.example.jwtdemo.config;

import com.example.jwtdemo.interceptor.JwtAuthInterceptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private JwtAuthInterceptor jwtAuthInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(jwtAuthInterceptor)
                .addPathPatterns("/api/**")
                .excludePathPatterns("/api/auth/login", "/api/auth/refresh");
    }
}
```





---

##### 八、测试受保护接口（`TestController.java`）

```java
package com.example.jwtdemo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import jakarta.servlet.http.HttpServletRequest;

@RestController
public class TestController {

    @GetMapping("/api/hello")
    public String hello(HttpServletRequest request) {
        String userId = (String) request.getAttribute("userId");
        return "Hello, User ID: " + userId;
    }
}
```

---

##### 九、运行与测试

1. 启动 Spring Boot 应用。
2. 使用 Postman 或 curl 测试：

###### 1. 登录

```bash
POST http://localhost:8080/api/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "123456"
}
```
✅ 返回：
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiJ9.xxxxx",
    "refreshToken": "eyJhbGciOiJIUzI1NiJ9.yyyyy",
    "expiresIn": 1800
  }
}
```

###### 2. 调用受保护接口

```bash
GET http://localhost:8080/api/hello
Authorization: Bearer <accessToken>
```
✅ 返回：`Hello, User ID: 1`

###### 3. 刷新 Token

```bash
POST http://localhost:8080/api/auth/refresh
Content-Type: text/plain

<refreshToken>
```

---

##### 十、生产环境增强建议

1. **替换模拟用户为真实数据库**（JPA / MyBatis）
2. **Refresh Token 存入 Redis**，并设置 TTL，支持登出时删除
3. **启用 HTTPS**
4. **前端 Token 存储使用 HttpOnly Cookie（Refresh Token） + 内存（Access Token）**
5. **添加全局异常处理**（`@ControllerAdvice`）
6. **限制 CORS 来源**

---

> 💡 本项目已完整实现 JWT 无状态登录核心逻辑，可直接用于学习或作为项目基础模板。

如需 **集成 Spring Security + JWT**、**Redis 黑名单登出**、**前端 Vue 示例**，欢迎继续提问！

#### 将 `JwtAuthInterceptor` 注册到 Spring MVC 拦截器链中

这是一个非常关键的问题！**将 `JwtAuthInterceptor` 注册到 Spring MVC 拦截器链中，确实会影响执行顺序，而顺序直接关系到安全性与功能正确性**。

下面从 **机制、顺序规则、影响分析、最佳实践** 四个方面详细说明：

---

##### ✅ 一、Spring MVC 拦截器的执行机制

Spring MVC 的拦截器（`HandlerInterceptor`）执行顺序如下：

```
preHandle 1 → preHandle 2 → ... → Controller → postHandle 2 → postHandle 1 → afterCompletion 1 → afterCompletion 2
```

- `preHandle`：**按注册顺序执行**（先注册，先执行）
- `postHandle` / `afterCompletion`：**按注册顺序的反向执行**（后注册，先执行）

> ⚠️ 只要任意一个 `preHandle` 返回 `false`，后续拦截器和 Controller 都不会执行。

---

##### ✅ 二、你注册的 `JwtAuthInterceptor` 会排在什么位置？

在 `WebConfig` 中：

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(jwtAuthInterceptor)
            .addPathPatterns("/**");
}
```

- **这是你手动注册的第一个（或唯一）拦截器**
- Spring Boot **默认没有内置的业务拦截器**（如日志、权限等），除非你用了 Spring Security 或其他 starter
- 所以：**你的 `JwtAuthInterceptor` 通常是链中的第一个（也是唯一一个）**

✅ **结论**：  
> **只要你不注册其他拦截器，就不会有顺序冲突**。  
> **如果你注册了多个，顺序 = 代码中 `addInterceptor()` 的调用顺序**。

---

##### ✅ 三、如果有多个拦截器，顺序如何控制？

###### 示例：注册多个拦截器

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 1. 日志拦截器（最先执行）
        registry.addInterceptor(new LogInterceptor())
                .addPathPatterns("/**");

        // 2. JWT 认证拦截器（第二执行）
        registry.addInterceptor(jwtAuthInterceptor)
                .addPathPatterns("/**");

        // 3. 权限校验拦截器（最后执行）
        registry.addInterceptor(new PermissionInterceptor())
                .addPathPatterns("/admin/**");
    }
}
```

###### 执行顺序（preHandle）：

1. `LogInterceptor.preHandle()` → 记录请求开始
2. `JwtAuthInterceptor.preHandle()` → 验证 token
3. `PermissionInterceptor.preHandle()` → 检查角色权限
4. Controller

> ✅ 这是**推荐顺序**：**日志 → 认证 → 授权**

---

##### ✅ 四、原有拦截器是否受影响？

| 场景                         | 是否受影响     | 说明                                                         |
| ---------------------------- | -------------- | ------------------------------------------------------------ |
| 项目原本**没有其他拦截器**   | ❌ 不影响       | 你是第一个，无冲突                                           |
| 项目用了 **Spring Security** | ⚠️ **有影响！** | Security 的 Filter 在 **Servlet 层**，比 MVC 拦截器**更早执行**，两者不在同一层 |
| 项目有**自定义拦截器**       | ✅ 可能影响     | 顺序由注册顺序决定，需人工协调                               |

> 🔍 **关键区别**：
> - **Filter（如 Spring Security）**：在 `DispatcherServlet` 之前执行（Servlet 规范）
> - **HandlerInterceptor（MVC 拦截器）**：在 `DispatcherServlet` 内部执行（Spring MVC 层）
>
> 所以：**Security 不属于“原有拦截器”**，它和你的 `JwtAuthInterceptor` 是**互补关系**，不是竞争关系。

---

##### ✅ 五、最佳实践建议

###### ✅ 1. **明确拦截器职责**

| 拦截器        | 职责           | 建议顺序 |
| ------------- | -------------- | -------- |
| 日志/耗时统计 | 记录请求入口   | 最先     |
| **JWT 认证**  | 验证用户身份   | 第二     |
| 权限/角色校验 | 检查是否可访问 | 最后     |

###### ✅ 2. **避免在拦截器中做耗时操作**

- 拦截器应**快速失败**（如 token 无效立即返回 401）
- 不要查数据库（除非缓存）

###### ✅ 3. **白名单要双重保险**

- 在 `JwtAuthInterceptor` 内部判断
- 在 `WebConfig` 中用 `excludePathPatterns` 排除

```java
registry.addInterceptor(jwtAuthInterceptor)
        .addPathPatterns("/**")
        .excludePathPatterns("/auth/**", "/static/**", "/error");
```

###### ✅ 4. **不要和 Spring Security 混用（除非必要）**

- 如果你用 JWT + 拦截器，**就不要引入 Spring Security 的 WebSecurityConfigurer**
- 否则会出现 **双重认证** 或 **冲突**

---

##### ✅ 六、如何查看当前拦截器链？

可以在启动时加日志：

```java
@Component
public class DebugInterceptor implements HandlerInterceptor {
    private static final Logger log = LoggerFactory.getLogger(DebugInterceptor.class);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        log.info("进入拦截器: {}", this.getClass().getSimpleName());
        return true;
    }
}
```

或者调试 `DispatcherServlet` 的 `getHandler()` 方法。

---

##### ✅ 总结

| 问题                                               | 答案                                                         |
| -------------------------------------------------- | ------------------------------------------------------------ |
| **注册 `JwtAuthInterceptor` 会影响原有拦截器吗？** | 如果没有其他拦截器 → **不影响**；如果有 → **按注册顺序执行** |
| **执行顺序怎么定？**                               | `addInterceptor()` 的调用顺序决定 `preHandle` 顺序           |
| **Spring Security 会冲突吗？**                     | 不会，它在更早的 Filter 层，但**建议二选一**                 |
| **最佳顺序是什么？**                               | 日志 → 认证（JWT）→ 授权（权限）                             |

---

✅ **你的当前做法是安全的**：只要项目没有其他 `HandlerInterceptor`，`JwtAuthInterceptor` 就是唯一的认证关卡，不会有问题。

需要我帮你设计一个“日志 + JWT + 权限”三层拦截器的完整示例吗？

---

### 3. **基于 OAuth2 / OIDC 的第三方登录（企业级）**

适用于需要统一身份认证（SSO）或集成第三方（如微信、钉钉、企业微信）的场景。

##### 常见模式：

- **Authorization Code Flow**（Web 应用推荐）
- **Password Flow**（已不推荐，仅限高度信任的内部应用）
- **Client Credentials Flow**（服务间调用）

##### 技术栈：

- **Spring Security OAuth2**（旧） → **Spring Authorization Server**（新）
- **Keycloak**、**Auth0**、**Okta** 等 IDP（身份提供商）

##### 流程简述：

1. 用户访问系统，重定向到认证中心（如 Keycloak）。
2. 用户在认证中心登录。
3. 认证中心返回 ID Token（JWT）和 Access Token。
4. 后端验证 Token，建立本地会话或直接信任 Token。

##### 优点：

- 统一登录、单点登录（SSO）。
- 支持多租户、多应用。
- 安全标准成熟。

##### 缺点：

- 架构复杂，适合中大型系统。

---

### 三、增强安全措施（无论哪种方案都应考虑）

| 安全项                | 实现建议                                                     |
| --------------------- | ------------------------------------------------------------ |
| **密码安全**          | BCrypt 加密 + 强制复杂度策略                                 |
| **防暴力破解**        | 登录失败次数限制（如 5 次锁定 15 分钟），可用 Redis 记录失败次数 |
| **验证码**            | 登录页加图形验证码（如 Kaptcha）或滑块验证                   |
| **HTTPS**             | 全站强制 HTTPS，防止中间人攻击                               |
| **CSRF 防护**         | Session 方案需开启 CSRF Token（Spring Security 默认支持）；JWT 无状态可关闭 |
| **敏感操作审计**      | 记录登录日志（IP、设备、时间）                               |
| **多因素认证（MFA）** | 重要系统支持短信/邮箱/OTP 二次验证                           |

---

### 四、技术选型对比

| 方案            | 适用场景                | 扩展性               | 安全性                  | 实现复杂度 |
| --------------- | ----------------------- | -------------------- | ----------------------- | ---------- |
| **Session**     | 内部管理系统、单体应用  | 低（需共享 Session） | 中                      | 低         |
| **JWT**         | 前后端分离、App、微服务 | 高                   | 中高（需设计刷新/注销） | 中         |
| **OAuth2/OIDC** | 企业级 SSO、多系统集成  | 极高                 | 高                      | 高         |

---

### 五、Spring Boot 实现示例（JWT 方案）

#### 1. 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

#### 2. 登录接口

```java
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest req) {
    // 1. 验证用户名密码
    User user = userService.findByUsername(req.getUsername());
    if (user == null || !passwordEncoder.matches(req.getPassword(), user.getPassword())) {
        return ResponseEntity.status(401).body("认证失败");
    }
    // 2. 生成 JWT
    String token = jwtUtil.generateToken(user.getId(), user.getRoles());
    return ResponseEntity.ok(new JwtResponse(token));
}
```

#### 3. 拦截器验证 Token

```java
@Component
public class JwtFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(...) {
        String token = extractToken(request);
        if (token != null && jwtUtil.validate(token)) {
            // 设置 SecurityContext
            Authentication auth = new UsernamePasswordAuthenticationToken(userId, null, authorities);
            SecurityContextHolder.getContext().setAuthentication(auth);
        }
        chain.doFilter(request, response);
    }
}
```

#### 4. 配置 Security

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/login").permitAll()
                .anyRequest().authenticated()
            );
        return http.build();
    }
}
```

---

### 六、总结：如何选择？

- **内部小系统** → Session + Spring Security（简单可靠）
- **前后端分离 / App** → JWT + Refresh Token（主流选择）
- **多系统统一登录** → OAuth2 / OIDC + Keycloak（企业级）

> 💡 **最佳实践建议**：
> - 永远不要明文存密码。
> - 生产环境必须用 HTTPS。
> - 登录接口做限流防爆破。
> - 敏感系统加上操作日志和 MFA。

---

如果你有具体场景（如“要做一个后台管理系统，只有 5 个管理员”或“需要支持微信扫码登录”），我可以给出更精准的技术方案和代码模板。是否需要？

## 微信二维码扫码登录实现流程

### 网站应用扫码登录

微信扫码登录（即“微信 OAuth2.0 网站应用扫码登录”）是一种常见的第三方登录方式，广泛用于 PC 网站。其核心思想是：**用户在 PC 端打开登录页 → 扫码 → 手机确认 → PC 端自动登录**。

下面是完整的实现流程（基于微信官方文档），适用于 **网站应用（非公众号）**。

---

#### ✅ 一、前提条件

1. **已注册微信开放平台账号**（[https://open.weixin.qq.com](https://open.weixin.qq.com)）
2. **创建“网站应用”**，并审核通过（获取 `appid` 和 `appsecret`）
3. **配置授权回调域名**（如 `https://yourdomain.com`）

> ⚠️ 注意：**公众号扫码登录 ≠ 网站应用扫码登录**！  
> 本文讲的是 **网站应用** 的扫码登录（PC 端用）。

---

#### ✅ 二、整体流程图

```mermaid
sequenceDiagram
    participant User as 用户(PC)
    participant Web as 你的网站
    participant WeChat as 微信服务器
    participant Mobile as 用户手机(微信APP)

    User->>Web: 访问登录页
    Web->>WeChat: 请求获取二维码 ticket
    WeChat-->>Web: 返回 ticket + 二维码图片URL
    Web->>User: 展示二维码
    User->>Mobile: 扫码
    Mobile->>WeChat: 确认登录
    WeChat->>Web: (异步) 通知登录成功（可选）
    Web->>WeChat: 轮询查询扫码状态
    WeChat-->>Web: 返回 code
    Web->>WeChat: 用 code 换 access_token + openid
    WeChat-->>Web: 返回用户标识
    Web->>User: 登录成功，跳转首页
```

---

#### ✅ 三、详细步骤（后端 + 前端）

##### 步骤 1️⃣：前端请求生成二维码

**后端接口**：`GET /api/wechat/qrcode`

```java
// 1. 向微信请求 ticket
String appId = "your_appid";
String redirectUri = URLEncoder.encode("https://yourdomain.com/api/wechat/callback", "UTF-8");
String url = "https://open.weixin.qq.com/connect/qrconnect" +
             "?appid=" + appId +
             "&redirect_uri=" + redirectUri +
             "&response_type=code" +
             "&scope=snsapi_login" +
             "&state=STATE#wechat_redirect";

// 但实际获取二维码，应调用微信 API 获取 ticket
String ticketUrl = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=SECRET";
// 先获取 access_token（注意：这里是基础 access_token，不是用户 token）
// 再用 access_token 换取 ticket
```

✅ **更推荐做法**：直接返回微信二维码链接（前端拼接）

```java
@GetMapping("/qrcode")
public Map<String, String> getQrCodeUrl() {
    String redirectUri = "https://yourdomain.com/api/wechat/callback";
    String url = "https://open.weixin.qq.com/connect/qrconnect?" +
        "appid=" + APPID +
        "&redirect_uri=" + URLEncoder.encode(redirectUri, StandardCharsets.UTF_8) +
        "&response_type=code" +
        "&scope=snsapi_login" +
        "&state=" + UUID.randomUUID().toString();
    
    // 前端用此 URL 生成二维码
    return Map.of("qrCodeUrl", url);
}
```

> 💡 实际上，**前端直接用这个 URL 生成二维码即可**，无需后端调 API 拿 ticket（微信会自动处理）。

---

##### 步骤 2️⃣：前端展示二维码

```html
<!-- 使用 qrcode.js 或其他库生成二维码 -->
<div id="qrcode"></div>
<script src="qrcode.min.js"></script>
<script>
  fetch('/api/wechat/qrcode')
    .then(res => res.json())
    .then(data => {
      new QRCode(document.getElementById("qrcode"), data.qrCodeUrl);
    });
</script>
```

> 📌 二维码内容是一个 URL，扫码后会跳转到微信 APP 内的授权页。

---

##### 步骤 3️⃣：用户扫码 + 确认授权

- 用户打开微信 → 扫码 → 弹出“登录确认”页
- 点击“允许”后，微信会**重定向到你配置的 `redirect_uri`**

---

##### 步骤 4️⃣：处理回调（获取 code）

微信会跳转到：
```
https://yourdomain.com/api/wechat/callback?code=CODE&state=STATE
```

**后端处理回调**：

```java
@GetMapping("/callback")
public void wechatCallback(@RequestParam String code, @RequestParam String state, HttpServletResponse response) {
    // 1. 验证 state（防 CSRF）
    if (!isValidState(state)) {
        throw new RuntimeException("Invalid state");
    }

    // 2. 用 code 换取 access_token 和 openid
    String tokenUrl = "https://api.weixin.qq.com/sns/oauth2/access_token" +
        "?appid=" + APPID +
        "&secret=" + APPSECRET +
        "&code=" + code +
        "&grant_type=authorization_code";

    // 发起 HTTP 请求获取 token
    String tokenResponse = restTemplate.getForObject(tokenUrl, String.class);
    // 解析 JSON: { "access_token": "...", "openid": "..." }

    // 3. （可选）用 access_token 获取用户信息
    String userInfoUrl = "https://api.weixin.qq.com/sns/userinfo?access_token=...&openid=...";
    
    // 4. 根据 openid 查询/创建本地用户，生成登录态（如 JWT 或 Session）
    User user = userService.findByOpenid(openid);
    if (user == null) {
        user = userService.createByWechat(openid, userInfo);
    }
    
    // 5. 设置登录状态（如写 Cookie / 返回 JWT）
    String jwt = jwtUtil.generate(user);
    response.sendRedirect("/?token=" + jwt); // 或写入 Session
}
```

---

##### 步骤 5️⃣：前端自动跳转（登录成功）

- 回调页面设置后，浏览器会跳转到你的首页，并携带登录凭证（如 token）
- 前端读取 token 并保存到 localStorage，完成登录

---

#### ✅ 四、进阶：轮询检测扫码状态（提升体验）

上述流程依赖“用户扫码后跳转”，但 PC 端页面无法感知是否已扫码。

**优化方案**：前端轮询后端，检测是否已授权。

##### 实现方式：

1. 生成二维码时，后端生成一个唯一 `scene_id`（如 UUID）
2. 前端轮询 `/api/wechat/poll?scene=xxx`
3. 用户扫码确认后，微信会**异步通知**你的服务器（需配置）**或**你在回调中记录状态
4. 轮询接口返回 `{ status: "scanned" | "confirmed" | "timeout" }`

> 📌 微信**不提供主动推送**，所以通常做法是：
> - 回调接口中将 `openid` 与 `scene_id` 关联存入 Redis
> - 轮询接口查 Redis 是否有结果

---

#### ⚠️ 五、安全注意事项

1. **验证 `state` 参数**：防止 CSRF 攻击
2. **`redirect_uri` 必须严格匹配**：微信会校验域名
3. **不要泄露 `appsecret`**：仅在后端使用
4. **`openid` 是用户唯一标识**：可作为本地用户关联依据
5. **用户信息需二次获取**：`access_token` 有效期 2 小时

---

#### ✅ 六、官方文档参考

- 微信网站应用扫码登录：  
  [https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Wechat_Login.html](https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Wechat_Login.html)

---

#### ✅ 总结：关键步骤

| 步骤 | 动作                                          |
| ---- | --------------------------------------------- |
| 1    | 前端请求二维码 URL（含 appid + redirect_uri） |
| 2    | 前端生成二维码展示                            |
| 3    | 用户微信扫码 + 确认                           |
| 4    | 微信跳转到你的回调 URL（带 code）             |
| 5    | 后端用 code 换 openid + access_token          |
| 6    | 创建/登录本地用户，返回登录态                 |

> 🚀 **一句话口诀**：  
> **“前端要码，用户扫码，微信回调，后端换号，登录搞定！”**

如果你使用的是 **公众号内网页授权**（在微信内打开网页），流程不同（用 `snsapi_base` / `snsapi_userinfo`），但本文适用于 **PC 网站扫码登录** 场景。

### 微信扫码登录

**不是的！微信扫码登录在本地开发环境也可以使用**，但需要满足微信开放平台的 **域名白名单限制**。下面详细说明如何在本地（如 `localhost`）调试微信扫码登录。

---

#### ❌ 为什么默认不能用 `localhost`？

微信出于安全考虑，**强制要求**：
- 授权回调域名必须是 **已备案的公网域名**
- **不能是 IP 地址或 `localhost`、`127.0.0.1`**

所以直接用 `http://localhost:8080` 会报错：
> **“redirect_uri 参数错误”**

---

#### ✅ 解决方案：本地调试微信扫码登录

##### 方案一：使用内网穿透工具（推荐 ✅）

通过工具将本地服务映射到一个 **公网可访问的 HTTPS 域名**，满足微信要求。

###### 常用工具：

| 工具                | 特点                       |
| ------------------- | -------------------------- |
| **ngrok**           | 免费，简单，有随机域名     |
| **localtunnel**     | 免费，命令行一键           |
| **frp**             | 自建，稳定，需有公网服务器 |
| **花生壳 / NATAPP** | 国内，有免费版             |

###### 操作步骤（以 ngrok 为例）：

1. 下载 [ngrok](https://ngrok.com/)
2. 注册账号，获取 authtoken
3. 启动本地服务（如 `http://localhost:8080`）
4. 执行：
   ```bash
   ngrok http 8080
   ```
5. 得到类似地址：
   ```
   https://abcd1234.ngrok.io
   ```
6. **在微信开放平台配置授权回调域名**：
   
   - 域名填：`abcd1234.ngrok.io`（**不要带 https://**）
7. 本地代码中回调地址改为：
   ```java
   String redirectUri = "https://abcd1234.ngrok.io/api/wechat/callback";
   ```

> ✅ 这样扫码后，微信会跳转到 ngrok 提供的公网地址，再由它转发到你的 `localhost:8080`。

---

##### 方案二：修改 Hosts + 公网域名（适合有域名者）

如果你有已备案的域名（如 `example.com`）：
1. 在 DNS 添加一条 A 记录：
   ```
   dev.example.com → 你的公网 IP（或 127.0.0.1 如果在本地测试）
   ```
2. 本地修改 `hosts` 文件（`C:\Windows\System32\drivers\etc\hosts` 或 `/etc/hosts`）：
   ```
   127.0.0.1 dev.example.com
   ```
3. 微信开放平台配置回调域名为：`dev.example.com`
4. 本地启动服务监听 `dev.example.com:8080`
5. 访问 `https://dev.example.com:8080` 进行测试

> ⚠️ 注意：微信要求 **HTTPS**，本地需配置 SSL 证书（可用自签名 + 浏览器信任，或用 ngrok 自动提供 HTTPS）。

---

##### 方案三：临时部署到测试服务器（简单粗暴）

- 将代码临时部署到一台有公网 IP 和域名的测试服务器（如阿里云 ECS）
- 配置好域名和 HTTPS
- 直接调试

---

##### ⚠️ 重要注意事项

1. **必须使用 HTTPS**  
   微信扫码登录 **强制要求回调地址是 HTTPS**（`http://` 会失败，除非是微信认证的特殊内网环境，一般开发者没有）。
   - ngrok / localtunnel 默认提供 HTTPS
   - 本地自建需配置 SSL（可用 [mkcert](https://github.com/FiloSottile/mkcert) 生成本地可信证书）

2. **域名不能带端口**  
   微信开放平台填写的授权域名 **不能包含端口**（如不能写 `example.com:8080`）  
   → 解决方案：用 nginx 反向代理，或让服务跑在 443 端口

3. **路径可以自定义**  
   域名只需匹配到主机名，路径可以任意：
   - 配置域名：`abcd1234.ngrok.io`
   - 回调地址可以是：`https://abcd1234.ngrok.io/login/wechat/callback`

---

##### ✅ 推荐开发流程

```mermaid
graph LR
    A[本地启动 Spring Boot 服务] --> B[启动 ngrok 映射到公网]
    B --> C[微信开放平台配置 ngrok 域名]
    C --> D[访问 https://ngrok地址 进行扫码测试]
    D --> E[调试通过后部署线上]
```

---

##### 💡 小技巧：避免频繁改代码

在配置中使用 **环境变量** 控制回调地址：

```java
@Value("${wechat.redirect-uri}")
private String redirectUri;

// application-dev.yml
wechat:
  redirect-uri: https://abcd1234.ngrok.io/api/wechat/callback

// application-prod.yml
wechat:
  redirect-uri: https://yourdomain.com/api/wechat/callback
```

---

##### ✅ 总结

| 问题                                   | 答案                                                |
| -------------------------------------- | --------------------------------------------------- |
| **必须部署线上才能用微信扫码登录吗？** | ❌ 不是                                              |
| **本地能调试吗？**                     | ✅ 能，通过内网穿透（如 ngrok）                      |
| **关键限制是什么？**                   | 回调域名必须是 **已配置的公网 HTTPS 域名**          |
| **最简单方案？**                       | 用 **ngrok** 或 **localtunnel** 生成临时 HTTPS 地址 |

> 🚀 **一句话**：  
> **“本地开发用微信扫码？上 ngrok，配域名，HTTPS 走起！”**

这样你就可以在 `localhost` 写代码，同时正常测试微信扫码登录功能了。

## token类型

在 Web 认证和授权体系中，`tokenType`（令牌类型）主要用于 **标识 Token 的格式、用途和验证方式**。它常见于以下两个场景：

1. **HTTP `Authorization` 请求头中的 `tokenType`**（如 `Bearer`、`Basic`）  
2. **OAuth 2.0 / OpenID Connect 协议中返回的 `token_type` 字段**

---

### ✅ 一、HTTP `Authorization` 头中的标准 Token 类型（RFC 规范）

根据 [RFC 7235](https://datatracker.ietf.org/doc/html/rfc7235) 和 [RFC 6750](https://datatracker.ietf.org/doc/html/rfc6750)，常见的 `tokenType` 有：

| Token Type   | 说明                                                         | 使用示例                                                     |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **`Bearer`** | **最常用！** 表示“持有者即可访问”，用于 JWT、OAuth 2.0 access_token | `Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...` |
| **`Basic`**  | 基本身份认证，用户名密码 Base64 编码                         | `Authorization: Basic dXNlcjpwYXNz`                          |
| **`Digest`** | 摘要认证（比 Basic 更安全，但已少用）                        | `Authorization: Digest username="user", realm="..."`         |
| **`OAuth`**  | 旧版 OAuth 1.0 使用（**已废弃**）                            | `Authorization: OAuth oauth_consumer_key="..."`              |
| **`ApiKey`** | 非标准，某些 API 自定义使用（如 `X-API-Key` 更常见）         | `Authorization: ApiKey your-secret-key`                      |

> ✅ **现代 Web 开发中，99% 的场景使用 `Bearer`**（尤其是 JWT）。

---

### ✅ 二、OAuth 2.0 / OpenID Connect 中的 `token_type`

当你调用 OAuth 2.0 授权服务器（如 Auth0、Keycloak、微信 OAuth）获取 token 时，响应中会包含：

```json
{
  "access_token": "eyJ...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "..."
}
```

#### 常见的 `token_type` 值：

| 值                                             | 说明                                                      |
| ---------------------------------------------- | --------------------------------------------------------- |
| **`Bearer`**                                   | 默认且最常用，表示“谁持有 token 谁就能访问”               |
| **`MAC`** (Message Authentication Code)        | 更安全，需签名请求（但实现复杂，**极少使用**）            |
| **`DPoP`** (Demonstrating Proof-of-Possession) | 新兴标准（RFC 9449），防止 token 被盗用（用于高安全场景） |

> 🔒 `Bearer` 的缺点：**Token 一旦泄露，攻击者可直接冒用**。  
> 所以敏感系统会考虑 `DPoP` 或结合设备指纹、IP 绑定等增强安全。

---

### ✅ 三、JWT 本身有没有 `tokenType`？

JWT（JSON Web Token）**本身不包含 `tokenType` 字段**，但它的用途决定了它属于哪种类型：

- 如果你用 JWT 作为 **OAuth 2.0 的 access_token** → 它就是 `Bearer` token
- 如果你用 JWT 作为 **自定义认证令牌** → 通常也按 `Bearer` 处理

JWT 的 Header 中有 `typ` 字段，但那是 **JWT 类型标识**，不是 HTTP 的 `tokenType`：

```json
// JWT Header 示例
{
  "alg": "HS256",
  "typ": "JWT"   // ← 表示这是一个 JWT，不是 JWS 或 JWE
}
```

> ❗ `typ: "JWT"` ≠ HTTP `Authorization: Bearer` 中的 `Bearer`

---

### ✅ 四、实际开发建议

| 场景                     | 推荐 `tokenType`                             |
| ------------------------ | -------------------------------------------- |
| 自研 JWT 认证系统        | **`Bearer`**                                 |
| OAuth 2.0 接入第三方登录 | 按对方返回的 `token_type`（通常是 `Bearer`） |
| 高安全要求（如金融）     | 考虑 `DPoP` 或自定义方案（如绑定设备）       |
| 内部服务调用             | 可用 `Bearer` + 短有效期，或改用 mTLS        |

---

### ✅ 五、Knife4j / Swagger 中如何配置？

在 OpenAPI 3.0 中，`tokenType` 由 `SecurityScheme.scheme` 决定：

```java
SecurityScheme securityScheme = new SecurityScheme()
    .type(SecurityScheme.Type.HTTP)
    .scheme("bearer")        // ← 这里决定 HTTP 头的 tokenType
    .bearerFormat("JWT");
```

- `.scheme("bearer")` → 生成 `Authorization: Bearer <token>`
- `.scheme("basic")` → 生成 `Authorization: Basic <credentials>`

---

### ✅ 总结

| 名称                                | 是否标准     | 常见值                  | 用途                     |
| ----------------------------------- | ------------ | ----------------------- | ------------------------ |
| **HTTP `Authorization` 头的类型**   | ✅ RFC 标准   | `Bearer`, `Basic`       | 客户端发送认证凭证       |
| **OAuth 2.0 响应中的 `token_type`** | ✅ OAuth 标准 | `Bearer`, `MAC`, `DPoP` | 告诉客户端如何使用 token |
| **JWT Header 中的 `typ`**           | ✅ JWT 标准   | `"JWT"`                 | 标识 payload 是 JWT 格式 |

> 💡 **记住一句话：只要你用的是 JWT 做认证，HTTP 头就写 `Authorization: Bearer <your_token>`，`tokenType` 就是 `Bearer`。**

如有特殊安全需求（如防 token 泄露），再考虑 `DPoP` 等高级方案。