## 3月29日 第6课

### 本课内容：关注系统（UserFollowing）

---

### 项目整体架构回顾

```
HTTP请求
   ↓
bilibili-api（接口层）   ← 只负责接收请求、返回响应
   ↓ @Autowired
bilibili-service（业务层）← 核心逻辑都在这里
   ↓ @Autowired
bilibili-dao（数据层）   ← 只管数据库读写
   ↓ MyBatis
MySQL
```

---

### 已完成模块

#### 模块一：用户系统（User）

| 方法 | 路径 | 是否需要Token | 说明 |
|------|------|-------------|------|
| GET | `/rsa-pks` | 否 | 获取RSA公钥 |
| POST | `/users` | 否 | 注册 |
| POST | `/user-tokens` | 否 | 登录，返回JWT |
| GET | `/users` | 是 | 查当前用户信息 |
| PUT | `/users` | 是 | 更新用户 |
| PUT | `/user-infos` | 是 | 更新用户详情 |

核心安全流程：
- 注册：RSA解密前端密码 → 加盐 → MD5哈希 → 存库
- 登录：解密+哈希 → 与库中比对 → 生成JWT返回
- 鉴权：UserSupport 从 Header 取 token → TokenUtil 解析出 userId

---

#### 模块二：关注系统（UserFollowing）—— 本课新增

**`UserFollowingService.java` 实现的功能：**

**1. 添加关注 `addUserFollowings()`**
```
检查分组存在 → 检查被关注用户存在 → 先删再插（防重复关注）
```

**2. 获取关注列表 `getUserFollowings()`**
```
查我关注了谁 → 查这些人的UserInfo → 按分组归类返回
```

**3. 获取粉丝列表 `getUserFans()`**
```
查谁关注了我 → 查粉丝的UserInfo → 标记我是否也关注了他（互关标记）
```

**4. 关注分组管理**
- 创建自定义分组
- 查询用户的所有分组
- 更新 / 删除关注关系

---

### 待完成

- [ ] 实现 `getUserInfoByUserIds()`（批量查用户信息）
  - 当前在 `getUserFollowings()` 和 `getUserFans()` 中被注释掉
  - 补全后关注列表和粉丝列表才能返回用户昵称、头像等信息

---
### 7 — 用户动态 + RocketMQ 消息队列（3月29日）

#### 核心概念：两种通知模式

**观察者模式**：A 发动态，代码里直接逐个通知 B、C、D。
- 缺点：100 万粉丝就得通知 100 万次，发帖操作会卡死。

**发布-订阅模式（本课采用）**：A 把动态扔给中间商（RocketMQ），然后去忙别的，中间商负责通知所有粉丝。
- 核心差异：多了个 Broker，发送者和接收者解耦，互不干扰。

---

#### 新增文件

| 文件 | 说明 |
|------|------|
| `UserMomentsConstant.java` | Topic 名、Consumer Group 名等常量 |
| `RocketMQConfig.java` | 注册 Producer / Consumer Bean |
| `RocketMQUtil.java` | 同步/异步发消息工具方法 |
| `UserMomentsService.java` | 发动态存库 + 投递 MQ；从 Redis 读订阅列表 |
| `UserMomentsApi.java` | `POST /user-moments` 发布动态（需 Token） |

#### 数据流

```
用户发动态
   ↓ 存入数据库
   ↓ Producer 投递消息到 RocketMQ（TOPIC_MOMENTS）
   ↓ Consumer 监听到消息
   ↓ （TODO）把动态写入粉丝的 Redis：subscribed-{粉丝ID}
   ↓ 粉丝调接口 → 从 Redis 读到更新
```

#### RocketMQ 本地启动命令

```bash
# 启动（进入 bin 目录后执行）
nohup sh mqnamesrv &
nohup sh mqbroker -n localhost:9876 --enable-proxy &

# 验证
tail -f ~/logs/rocketmqlogs/namesrv.log
tail -f ~/logs/rocketmqlogs/proxy.log

# 关闭
sh mqshutdown broker
sh mqshutdown namesrv
```

#### 概念
- Bean：你不用自己 new 对象，Spring                                 
启动时帮你创建好放进容器，用的时候 @Autowired 直接取。

- XML：写着"哪个 Java 方法对应哪条 SQL"，MyBatis                    
负责对照这张表把方法翻译成 SQL 执行。

- 生产者/消费者：生产者把消息扔进 RocketMQ                          
就走，消费者在旁边一直监听，有消息来了就处理——两边互不等待，解耦。

- [ ] Consumer 收到消息后，把动态写入粉丝的 Redis 订阅列表

### 8
### 9
### 10
### 11
### 12
### 13
### 14
### 15
### 16
### 17
### 18
### 19
### 20