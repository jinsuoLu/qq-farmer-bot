# QQ 农场自动挂机机器人 - 完整项目架构文档

## 📋 项目概述

这是一个基于 Node.js + Vue 3 的 QQ 农场自动挂机机器人，支持多账号管理、Web 控制面板、实时日志和数据分析。

---

## 🏗️ 系统架构

### 总体架构

```
┌─────────────────────────────────────────────────────────┐
│                        前端 (Vue 3)                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │  概览    │  │  个人    │  │  好友    │  │  分析    │ │
│  │Dashboard │  │ Personal │  │ Friends  │  │Analytics │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘ │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │  设置    │  │  后台    │  │  登录    │              │
│  │ Settings │  │ Admin    │  │  Login   │              │
│  └──────────┘  └──────────┘  └──────────┘              │
└─────────────────────────────────────────────────────────┘
                        ↕️ (HTTP + WebSocket)
┌─────────────────────────────────────────────────────────┐
│                    后端 (Node.js + Express)              │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Express HTTP Server + Socket.IO                   │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Runtime Engine (多进程/多线程管理器)               │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐       │
│  │  Worker 1   │ │  Worker 2   │ │  Worker N   │       │
│  │  (账号1)    │ │  (账号2)    │ │  (账号N)    │       │
│  └─────────────┘ └─────────────┘ └─────────────┘       │
└─────────────────────────────────────────────────────────┘
                        ↕️
                    QQ 农场 API
```

---

## 📁 项目目录结构

```
qq-farm-automation-bot/
├── core/                           # 后端核心代码
│   ├── src/
│   │   ├── config/                 # 配置文件
│   │   │   ├── config.js           # 主配置
│   │   │   ├── gameConfig.js       # 游戏配置
│   │   │   └── runtime-paths.js    # 运行时路径
│   │   ├── controllers/            # API 控制器
│   │   │   └── admin.js            # 管理面板接口
│   │   ├── core/                   # 核心功能
│   │   │   └── worker.js           # 工作进程
│   │   ├── models/                 # 数据模型
│   │   │   ├── store.js            # 数据存储
│   │   │   └── user-store.js       # 用户存储
│   │   ├── runtime/                # 运行时管理
│   │   │   ├── runtime-engine.js   # 运行引擎
│   │   │   ├── worker-manager.js   # Worker 管理器
│   │   │   ├── runtime-state.js    # 运行状态
│   │   │   ├── data-provider.js    # 数据提供器
│   │   │   └── relogin-reminder.js # 重登录提醒
│   │   ├── services/               # 业务服务
│   │   │   ├── logger.js           # 日志服务
│   │   │   ├── farm.js             # 农场服务
│   │   │   ├── friend.js           # 好友服务
│   │   │   ├── task.js             # 任务服务
│   │   │   ├── qrlogin.js          # 二维码登录
│   │   │   ├── push.js             # 推送服务
│   │   │   └── analytics.js        # 分析服务
│   │   ├── gameConfig/             # 游戏静态配置
│   │   │   ├── seed_images_named/  # 种子图片
│   │   │   ├── Plant.json
│   │   │   └── ItemInfo.json
│   │   └── proto/                  # Protocol Buffers
│   ├── data/                       # 运行时数据
│   │   ├── accounts.json
│   │   ├── users.json
│   │   ├── logs/
│   │   └── known_friend_gids/
│   ├── client.js                   # 后端入口
│   ├── package.json
│   └── Dockerfile
├── web/                            # 前端代码
│   ├── src/
│   │   ├── views/                  # 页面组件
│   │   │   ├── Login.vue
│   │   │   ├── Dashboard.vue
│   │   │   ├── Personal.vue
│   │   │   ├── Friends.vue
│   │   │   ├── Analytics.vue
│   │   │   ├── Settings.vue
│   │   │   └── AdminPanel.vue
│   │   ├── components/             # UI 组件
│   │   ├── layouts/                # 布局组件
│   │   ├── stores/                 # Pinia 状态管理
│   │   │   ├── account.ts
│   │   │   ├── status.ts
│   │   │   ├── farm.ts
│   │   │   ├── bag.ts
│   │   │   ├── friend.ts
│   │   │   ├── toast.ts
│   │   │   ├── app.ts
│   │   │   ├── user.ts
│   │   │   ├── plant-blacklist.ts
│   │   │   └── wx-login.ts
│   │   ├── router/                 # 路由配置
│   │   │   ├── index.ts
│   │   │   └── menu.ts
│   │   ├── api/                    # API 封装
│   │   │   └── index.ts
│   │   └── main.ts
│   ├── package.json
│   └── vite.config.ts
└── package.json                    # 根项目配置
```

---

## 🔌 后端 API 接口文档

### 认证与授权

所有需要认证的接口必须在请求头中携带：
```
x-admin-token: <你的token>
```

对于账号相关接口还需要：
```
x-account-id: <账号ID>
```

### 接口列表

#### 认证接口

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| POST | `/api/login` | 登录 | ❌ |
| POST | `/api/register` | 用户注册 | ❌ |
| POST | `/api/logout` | 登出 | ✅ |
| GET | `/api/auth/validate` | 验证 Token | ✅ |
| GET | `/api/ping` | 健康检查 | ❌ |
| GET | `/api/game-version` | 获取游戏版本 | ❌ |

#### 用户管理

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| POST | `/api/user/change-password` | 修改密码 | ✅ |
| POST | `/api/user/renew` | 续费会员 | ✅ |
| GET | `/api/card/info/:code` | 查询卡密信息 | ❌ |

#### 账号管理

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| GET | `/api/accounts` | 获取账号列表 | ✅ |
| POST | `/api/accounts` | 添加/更新账号 | ✅ |
| DELETE | `/api/accounts/:id` | 删除账号 | ✅ |
| POST | `/api/accounts/:id/start` | 启动账号 | ✅ |
| POST | `/api/accounts/:id/stop` | 停止账号 | ✅ |
| POST | `/api/accounts/:id/restart` | 重启账号 | ✅ |

#### 农场操作

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| GET | `/api/status` | 获取账号状态 | ✅ |
| GET | `/api/lands` | 获取农田详情 | ✅ |
| GET | `/api/seeds` | 获取种子列表 | ✅ |
| GET | `/api/daily-gifts` | 获取每日礼包 | ✅ |
| POST | `/api/automation` | 设置自动化配置 | ✅ |

#### 好友操作

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| GET | `/api/friends` | 获取好友列表 | ✅ |
| POST | `/api/friends/clear-cache` | 清除好友缓存 | ✅ |
| GET | `/api/friend/:gid/lands` | 获取好友农田 | ✅ |
| POST | `/api/friend/:gid/op` | 对好友执行操作 | ✅ |
| GET | `/api/interact-records` | 获取互动记录 | ✅ |

#### 背包与商店

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| GET | `/api/bag` | 获取背包物品 | ✅ |
| GET | `/api/bag/seeds` | 获取背包种子 | ✅ |
| POST | `/api/bag/use` | 使用物品 | ✅ |
| POST | `/api/bag/sell` | 出售物品 | ✅ |
| POST | `/api/fertilizer/buy` | 购买化肥 | ✅ |
| POST | `/api/fertilizer/check-and-buy` | 检查并自动购买化肥 | ✅ |

#### 黑名单管理

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| GET | `/api/friend-blacklist` | 获取好友黑名单 | ✅ |
| POST | `/api/friend-blacklist/toggle` | 切换好友黑名单 | ✅ |
| GET | `/api/friend-known-gids` | 获取已知好友 GID | ✅ |
| POST | `/api/friend-known-gids` | 保存已知好友 GID | ✅ |
| POST | `/api/friend-known-gids/remove` | 移除好友 GID | ✅ |
| POST | `/api/friend-known-gids/batch-add` | 批量添加好友 GID | ✅ |
| POST | `/api/friend-known-gids/batch-remove` | 批量移除好友 GID | ✅ |
| GET | `/api/plant-blacklist` | 获取植物黑名单 | ✅ |
| POST | `/api/plant-blacklist` | 添加植物黑名单 | ✅ |
| DELETE | `/api/plant-blacklist/:seedId` | 删除植物黑名单 | ✅ |
| POST | `/api/plant-blacklist/batch` | 批量添加植物黑名单 | ✅ |
| DELETE | `/api/plant-blacklist` | 清空植物黑名单 | ✅ |

#### 日志与状态

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| GET | `/api/logs` | 获取全局日志 | ✅ |
| GET | `/api/account-logs` | 获取账号日志 | ✅ |
| GET | `/api/scheduler` | 获取调度状态 | ✅ |

#### 管理员接口

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| GET | `/api/admin/login-logs` | 获取登录日志 | ✅ (仅管理员) |
| DELETE | `/api/admin/login-logs` | 清空登录日志 | ✅ (仅管理员) |

---

## 🌐 WebSocket 实时通信

### 连接方式

```javascript
import { io } from 'socket.io-client'

const socket = io('/', {
  path: '/socket.io',
  transports: ['websocket'],
  auth: { token: '<你的token>' }
})
```

### Socket 事件

#### 客户端发送事件

| Event | Payload | Description |
|-------|---------|-------------|
| `subscribe` | `{ accountId: string }` | 订阅指定账号的实时更新 |

#### 服务端推送事件

| Event | Payload | Description |
|-------|---------|-------------|
| `status:update` | `{ accountId, status }` | 账号状态更新 |
| `log:new` | 日志对象 | 新日志记录 |
| `account-log:new` | 日志对象 | 账号操作日志 |
| `logs:snapshot` | `{ logs: [] }` | 历史日志快照 |
| `account-logs:snapshot` | `{ logs: [] }` | 账号日志快照 |

---

## 🎨 前端页面路由

| Path | Name | Component | Description |
|------|------|-----------|-------------|
| `/login` | `login` | Login.vue | 登录页面 |
| `/` | `dashboard` | Dashboard.vue | 概览页面 |
| `/personal` | `personal` | Personal.vue | 个人农场 |
| `/friends` | `friends` | Friends.vue | 好友农场 |
| `/analytics` | `analytics` | Analytics.vue | 数据分析 |
| `/settings` | `Settings` | Settings.vue | 系统设置 |
| `/admin` | `admin` | AdminPanel.vue | 管理后台 |

---

## 🔧 核心模块详解

### 1. Runtime Engine

**文件**: `core/src/runtime/runtime-engine.js`

核心运行引擎，负责：
- Worker 进程/线程管理
- 账号状态同步
- 实时事件分发
- 配置广播

### 2. Worker Manager

**文件**: `core/src/runtime/worker-manager.js`

Worker 管理器，负责：
- 启动/停止/重启 Worker
- 进程间通信 (IPC)
- 错误处理与恢复

### 3. Admin Controller

**文件**: `core/src/controllers/admin.js`

API 控制器，提供：
- RESTful HTTP 接口
- WebSocket 实时通信
- 用户认证与授权
- 跨域 (CORS) 处理

### 4. Data Provider

**文件**: `core/src/runtime/data-provider.js`

数据提供器，桥接 API 与 Worker

---

## 🔐 安全机制

1. **Token 认证**：所有 API 调用 (除登录/注册外) 都需要 token
2. **用户隔离**：普通用户只能访问自己的账号
3. **密码加密**：使用 SHA256 哈希存储密码
4. **登录日志**：记录所有登录尝试
5. **速率限制**：防止暴力破解

---

## 📊 数据存储

所有数据以 JSON 文件形式存储在 `core/data/` 目录：
- `accounts.json` - 账号配置
- `users.json` - 用户信息
- `store.json` - 系统配置
- `login-logs.json` - 登录日志
- `accounts.json` - 账号数据
- `logs/combined.log` - 运行日志

---

## 🚀 启动流程

### 后端启动流程

1. **入口** (`client.js`)
   - 判断是主进程还是 Worker 进程
   - 创建 Runtime Engine

2. **Runtime Engine** (`runtime-engine.js`)
   - 加载系统配置
   - 启动 Admin Server
   - 启动 Worker 进程 (根据配置)

3. **Worker** (`worker.js`)
   - 加载账号配置
   - 连接农场 API
   - 执行自动化任务

### 前端启动流程

1. **路由守卫** 检查 token 有效性
2. **登录验证** 未登录跳转 /login
3. **API 初始化** 配置 axios 拦截器
4. **Socket 连接** 建立实时通信
5. **状态同步** 获取账号列表和状态

---

## 📝 开发说明

### 环境要求

- Node.js >= 20
- pnpm (推荐) 或 npm

### 开发命令

```bash
# 根目录
pnpm install -r          # 安装所有依赖
pnpm build:web           # 构建前端
pnpm dev:core            # 启动后端

# 仅前端开发
cd web
pnpm dev                 # 开发模式 (端口 5173)

# 仅后端开发
cd core
pnpm dev                 # 开发模式 (端口 3007)
```

---

## 📍 关键文件路径

| 文件 | 路径 | 说明 |
|------|------|------|
| 后端入口 | `core/client.js` | 主程序入口 |
| API 控制器 | `core/src/controllers/admin.js` | 所有 HTTP 接口定义 |
| 运行引擎 | `core/src/runtime/runtime-engine.js` | 核心运行时管理 |
| Worker 进程 | `core/src/core/worker.js` | 农场自动化执行进程 |
| 前端入口 | `web/src/main.ts` | Vue 应用入口 |
| API 封装 | `web/src/api/index.ts` | axios 配置和拦截器 |
| 路由配置 | `web/src/router/index.ts` | Vue Router 配置 |
| 状态管理 | `web/src/stores/` | Pinia stores 目录 |