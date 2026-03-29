# Bilibili 项目架构文档

## 项目简介

基于 Spring Boot 3.x 的仿哔哩哔哩后端系统，采用 Maven 多模块架构，按职责分层管理代码。

---

## 模块结构

```
bilibili（根项目）
├── bilibili-api        接口层：接收 HTTP 请求，对外暴露接口
├── bilibili-service    业务层：处理业务逻辑、工具类、配置
└── bilibili-dao        数据层：数据库操作、实体类、SQL
```

### bilibili-api（接口层）
```
BilibiliApp.java                  启动入口，扫描 com.example 包
api/
  DemoApi.java                    演示接口 GET /query
  RESTfulApi.java                 演示 REST 四种方法（GET/POST/PUT/DELETE）
  UserApi.java                    用户模块接口（注册/登录/查询/更新）
  support/
    UserSupport.java              从 Token 中解析当前登录用户 ID
resources/
  application.properties          激活环境 + MyBatis 配置
  application-test.properties     测试环境数据库连接
  application-online.properties   生产环境数据库连接
```

### bilibili-service（业务层）
```
service/
  DemoService.java                演示业务
  UserService.java                用户业务（注册/登录/更新）
config/
  JsonHttpMessageConverterConfig  全局 FastJSON 配置，统一 JSON 格式
handler/
  CommonGlobalExceptionHandler    全局异常拦截，统一错误返回格式
util/
  MD5Util.java                    MD5 单向加密（密码存储）
  AESUtil.java                    AES 对称加密（数据加解密）
  RSAUtil.java                    RSA 非对称加密（登录密码传输）
  TokenUtil.java                  JWT Token 生成与验证
```

### bilibili-dao（数据层）
```
dao/
  DemoDao.java                    演示数据库接口
  UserDao.java                    用户数据库接口
domain/
  User.java                       用户实体类
  UserInfo.java                   用户信息实体类
  JsonResponse.java               统一响应格式封装
  constant/
    UserConstant.java             用户相关常量
  exception/
    ConditionException.java       自定义业务异常
resources/mapper/
  demo.xml                        Demo SQL
  user.xml                        用户 SQL
```

---

## 技术栈

| 技术 | 版本 | 用途 |
|------|------|------|
| Spring Boot | 3.1.6 | 核心框架，简化配置和启动 |
| Spring MVC | 随 Spring Boot | 处理 HTTP 请求，路由分发 |
| MyBatis | 3.0.1 | ORM 框架，Java 对象与 SQL 映射 |
| MySQL | 8.0.31 | 关系型数据库 |
| FastJSON | 1.2.78 | JSON 序列化/反序列化 |
| java-jwt | 3.19.2 | JWT Token 生成与验证 |
| commons-codec | 1.14 | 编码工具（MD5 等） |
| spring-boot-devtools | 3.1.6 | 热部署，开发时自动重启 |

---

## 技术栈通俗讲解

### Spring Boot
**是什么：** 盖房子用的**预制板**。

正常搭一个 Java 后端，要自己配置几十个文件，就像从砖头一块块盖房子。Spring Boot 把这些都预制好了，你只需要写业务代码，直接拎包入住。

---

### Spring MVC
**是什么：** 餐厅的**点餐系统**。

你在 Postman 输入 `GET /users/1`，Spring MVC 负责判断"这个请求该交给哪个方法处理"，然后转发给对应的 `UserApi.java`。

```
GET /users    → 交给 getUserInfo()
POST /users   → 交给 addUser()
PUT /users    → 交给 updateUsers()
```

---

### MyBatis
**是什么：** Java 和数据库之间的**翻译官**。

Java 只懂对象，数据库只懂 SQL，MyBatis 负责翻译：
```
Java：userDao.getUserById(1)
       ↓ MyBatis 翻译
SQL：SELECT * FROM t_user WHERE id = 1
       ↓ 结果翻译回来
Java：User{ id=1, name="张三" }
```

---

### MySQL
**是什么：** 项目的**硬盘**。

所有用户数据、视频数据都存在这里。关掉程序数据不会丢失，就像存在硬盘里一样。

---

### FastJSON
**是什么：** Java 对象和 JSON 之间的**转换器**。

前端只认识 JSON 格式，Java 只认识对象格式，FastJSON 负责互转：
```
Java对象：User{ id=1, name="张三" }
    ↓ FastJSON
JSON：{ "id": 1, "name": "张三" }
```
同时还帮你统一格式，比如日期统一显示成 `2026-02-26 10:00:00`。

---

### JWT（java-jwt）
**是什么：** 登录后发的**临时通行证**。

不用 JWT 的话，每次请求都要带用户名密码，很不安全。用了 JWT：
```
1. 登录成功 → 服务器生成 Token（一串加密字符串）→ 发给前端
2. 前端把 Token 存起来
3. 以后每次请求带上 Token
4. 服务器验证 Token 合法 → 知道你是谁 → 放行
```
就像游乐园的手环，进门验票一次，之后凭手环随便玩。

---

### RSA（非对称加密）
**是什么：** 只能上锁不能开锁的**单向锁**。

用户登录时密码不能明文传输，RSA 解决这个问题：
```
服务器给前端一把锁（公钥）
前端用锁锁住密码发过来
服务器用钥匙（私钥）开锁
→ 就算黑客截获了数据，没有钥匙也打不开
```

---

### MD5
**是什么：** 密码的**单向绞碎机**。

密码不能明文存数据库，MD5 把密码绞碎成一串乱码：
```
"123456" → MD5 → "e10adc3949ba59abbe56e057f20f883e"
```
这个过程**不可逆**，就算数据库被盗，也看不出原始密码。

登录时把输入的密码再绞一遍，对比结果是否一样。搭配**盐值（salt）** 使用更安全，防止相同密码产生相同结果被破解。

---

### AES（对称加密）
**是什么：** 加密数据用的**双向锁**。

和 RSA 不同，AES 加密解密用同一把钥匙，速度更快，适合加密大量数据：
```
数据 + 密钥 → 加密 → 密文
密文 + 密钥 → 解密 → 数据
```

---

### commons-codec
**是什么：** 工具箱里的**瑞士军刀**。

提供各种编码工具，比如 MD5 加密、Base64 编解码等，被 MD5Util 内部使用。

---

### spring-boot-devtools
**是什么：** 开发时的**自动刷新器**。

改完代码保存，项目自动重启，不用每次手动重启，节省开发时间。上线时自动失效，不会影响生产环境。

---

## 请求完整流程

```
用户/Postman 发起请求
        ↓
UserApi（接收请求，校验参数）
        ↓  @Autowired
UserService（业务处理）
  ├── RSAUtil 解密前端传来的密码
  ├── MD5Util 加密后与数据库比对
  └── TokenUtil 登录成功后生成 JWT Token
        ↓  @Autowired
UserDao（数据库接口）
        ↓  MyBatis 自动绑定
user.xml（执行 SQL）
        ↓  mysql-connector-j
MySQL 数据库
        ↓
结果返回 → FastJSON 转 JSON → JsonResponse 包装
        ↓
{ "code": "0", "msg": "success", "data": {...} }
```

---

## 安全体系

```
前端登录流程：
1. 前端请求 GET /rsa-pks 获取公钥
2. 前端用公钥加密密码后发送 POST /user-tokens
3. 服务端用私钥解密（RSAUtil）
4. 解密后的密码加盐 MD5 与数据库比对（MD5Util）
5. 验证通过后生成 JWT Token 返回给前端（TokenUtil）
6. 后续请求携带 Token，服务端用 UserSupport 解析用户 ID
```

---

## 环境配置

```
application.properties          指定当前环境（test/online）
application-test.properties     测试环境（本地开发用）
application-online.properties   生产环境（上线用）
```

切换环境只需修改一行：
```properties
spring.profiles.active=online
```

---

## 数据库表

| 表名 | 说明 |
|------|------|
| t_demo | 演示表 |
| t_user | 用户表（手机号/邮箱/密码/盐值） |
| t_user_info | 用户信息表（昵称/头像/签名/性别/生日） |

---

## Postman 接口测试实战

### 所有接口一览

| 方法 | 路径 | 是否需要 Token | 说明 |
|------|------|--------------|------|
| GET | `/rsa-pks` | 否 | 获取 RSA 公钥 |
| POST | `/users` | 否 | 注册用户 |
| POST | `/user-tokens` | 否 | 登录，返回 JWT Token |
| GET | `/users` | **是** | 获取当前用户信息 |
| PUT | `/users` | **是** | 更新用户信息 |
| PUT | `/user-infos` | **是** | 更新用户详情 |

---

### 第 1 步：生成加密密码

密码不能明文传输，需先用 RSA 公钥加密。最简单的方式是直接运行 `RSAUtil.java` 的 main 方法：

```java
// RSAUtil.java
public static void main(String[] args) throws Exception {
    String str = RSAUtil.encrypt("123456"); // 填你要测试的密码
    System.out.println(str);               // 复制控制台输出的加密字符串
}
```

> 注意：每次运行输出的密文都不同，这是 RSA 的正常特性，都能被同一把私钥解密。

---

### 第 2 步：注册用户

```
POST /users
Content-Type: application/json

{
    "phone": "13590000001",
    "password": "上一步控制台输出的加密字符串"
}
```

成功响应：
```json
{ "code": "0", "msg": "success", "data": null }
```

服务端做了什么：
- RSAUtil 解密密码 → 得到明文
- 用注册时间戳作为 salt
- MD5(明文 + salt) → 存入数据库
- 自动创建默认 UserInfo（昵称 Newbie，生日 1999-10-01）

---

### 第 3 步：登录获取 Token

```
POST /user-tokens
Content-Type: application/json

{
    "phone": "13590000001",
    "password": "同样的加密字符串（重新运行 main 方法生成）"
}
```

成功响应：
```json
{
    "code": "0",
    "msg": "success",
    "data": "eyJ0eXAiOiJKV1QiLCJhbGci..."
}
```

把 `data` 里的 JWT 字符串复制保存，后续请求都要用。Token 有效期 2 小时。

---

### 第 4 步：带 Token 查询用户信息

在 Postman 的 **Headers** 标签页加一行：

| Key | Value |
|-----|-------|
| `token` | 第 3 步得到的 JWT 字符串 |

```
GET /users
Headers: token = eyJ0eXAiOiJKV1QiLCJhbGci...
```

成功响应：
```json
{
    "code": "0",
    "data": {
        "id": 2,
        "phone": "13590000001",
        "password": "474da62b32ebb6...",
        "salt": "1772178020096",
        "userInfo": {
            "nick": "Newbie",
            "birth": "1999-10-01",
            "gender": "0"
        }
    },
    "msg": "success"
}
```

服务端做了什么：
- 从 Header 取 token → TokenUtil 验证 → 解析出 userId
- 查 t_user + t_user_info 两张表 → 合并返回

> 注意：返回了 `password` 和 `salt` 字段，生产环境应过滤，不应暴露给前端。

---

### 常见错误速查

| 错误信息 | 原因 | 解决方法 |
|---------|------|---------|
| 404 Not Found | 路径拼错，如 `/user` | 改成 `/users` |
| Password decryption failed | 密码字符串含空格/非法字符 | 重新运行 RSAUtil.main() 生成 |
| Illegal user token | 请求头没带 token 或 token 过期 | 先登录获取新 token，加入 Header |
| User not found | 手机号未注册 | 先调 POST /users 注册 |