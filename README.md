# 新闻资讯后端服务

[![Python](https://img.shields.io/badge/python-3.12+-blue)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688)](https://fastapi.tiangolo.com/)
[![MySQL](https://img.shields.io/badge/MySQL-8.0+-4479A1)](https://www.mysql.com/)
[![Redis](https://img.shields.io/badge/Redis-7.x-DC382D)](https://redis.io/)
[![License](https://img.shields.io/badge/license-MIT-green)](./LICENSE)

> 一个面向移动端的新闻资讯后端服务，提供用户认证、内容管理、收藏与历史记录的完整 RESTful API。
> 配套前端项目：[xwzx-news](https://github.com/your/xwzx-news)

---

## 🏗️ 系统架构

```text
┌──────────────────────────────────────────────────────┐
│                      Nginx (可选)                     │
│                   反向代理 / 负载均衡                   │
└──────────────────────┬───────────────────────────────┘
                       │
┌──────────────────────▼───────────────────────────────┐
│                  FastAPI 应用层                        │
│                                                       │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌─────────┐ │
│  │ 路由层    │ │ 认证层    │ │ 校验层    │ │ 异常层   │ │
│  │ routers/  │ │ JWT       │ │ Pydantic  │ │ 全局处理 │ │
│  └─────┬─────┘ └─────┬─────┘ └─────┬─────┘ └─────────┘ │
│        │              │             │                   │
│  ┌─────▼──────────────▼─────────────▼─────────────────┐ │
│  │                   业务逻辑层 (CRUD)                  │ │
│  │      新闻服务  │  用户服务  │  收藏服务  │  历史服务   │ │
│  └──────────────────────┬─────────────────────────────┘ │
└──────────────────────────┼──────────────────────────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
┌─────────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐
│    MySQL 8.0    │  │   Redis 7   │  │  本地缓存    │
│  持久化存储      │  │  热点缓存    │  │  降级兜底    │
└────────────────┘  └─────────────┘  └─────────────┘
```

## 📂 项目结构

```text
toutiao_backend/
├── main.py                  # 应用入口：初始化 FastAPI、注册路由、配置中间件
├── config/
│   ├── db_conf.py           # 数据库会话工厂（异步引擎）
│   └── cache_conf.py        # Redis 连接池管理
├── models/                  # ORM 模型层 —— 纯表定义，无业务逻辑
│   ├── news.py              #   新闻表（title, content, category, source...）
│   ├── users.py             #   用户表（username, email, hashed_password）
│   ├── favorite.py          #   收藏表（user_id FK → users, news_id FK → news）
│   └── history.py           #   历史表（user_id, news_id, viewed_at）
├── schemas/                 # Pydantic 校验层 —— 请求/响应 DTO
├── crud/                    # 业务逻辑层 —— 纯粹的数据库操作
├── routers/                 # 路由层 —— 薄路由，调用 CRUD
├── utils/                   # 横切关注点
│   ├── auth.py              #   JWT 签发与验证
│   ├── security.py          #   bcrypt 密码哈希
│   ├── exception.py         #   业务异常定义
│   ├── exception_handlers.py #  异常 → HTTP 响应映射
│   └── response.py          #   统一响应格式 {code, message, data}
└── cache/news_cache.py      # 本地内存缓存（Redis 不可用时的降级方案）
```

---

## 🔧 技术选型

| 决策点 | 选择 | 理由 |
|---|---|---|
| **Web 框架** | FastAPI | 原生异步支持、自动生成 OpenAPI 文档、类型安全 |
| **ORM** | SQLAlchemy 2.0 async | 成熟的异步 ORM，2.0 语法更清晰 |
| **数据库驱动** | aiomysql | 纯 Python 异步 MySQL 驱动 |
| **密码存储** | bcrypt (passlib) | 抗暴力破解，自带盐值 |
| **认证方案** | JWT | 无状态认证，适合前后端分离架构 |
| **缓存策略** | Redis + 本地内存降级 | 热数据 Redis 缓存，故障时降级到内存缓存 |
| **依赖管理** | uv + pyproject.toml | 速度比 pip 快 10-100 倍，锁文件保证一致性 |

---

## 🚀 快速开始

### 环境要求

| 组件 | 版本 | 用途 |
|---|---|---|
| Python | 3.12+ | 运行环境 |
| MySQL | 8.0+ | 持久化存储 |
| Redis | 7.x | 缓存服务 |

### 安装运行

```bash
# 1. 克隆项目
git clone https://github.com/your/toutiao_backend.git
cd toutiao_backend

# 2. 安装依赖
uv sync

# 3. 配置环境变量
cp .env.example .env
# 编辑 .env：填入数据库密码、Redis 地址、JWT Secret

# 4. 创建数据库
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS toutiao CHARACTER SET utf8mb4;"

# 5. 启动服务（首次启动自动建表）
uv run uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

启动后访问：
- **Swagger 文档**：http://localhost:8000/docs
- **ReDoc 文档**：http://localhost:8000/redoc

---

## 📡 API 设计

### 统一响应格式

```json
{
  "code": 200,
  "message": "success",
  "data": { ... }
}
```

### 接口一览

#### 新闻模块

| 方法 | 端点 | 说明 | 认证 |
|---|---|---|---|
| GET | `/api/news?page=1&category=tech` | 分页 + 分类筛选 | 否 |
| GET | `/api/news/{id}` | 新闻详情 | 否 |
| GET | `/api/news/categories` | 分类列表 | 否 |

#### 用户模块

| 方法 | 端点 | 说明 | 认证 |
|---|---|---|---|
| POST | `/api/users/register` | 注册（username/password/email） | 否 |
| POST | `/api/users/login` | 登录 → 返回 JWT | 否 |
| GET | `/api/users/me` | 当前用户信息 | JWT |
| PUT | `/api/users/me` | 更新个人资料 | JWT |

#### 收藏模块

| 方法 | 端点 | 说明 | 认证 |
|---|---|---|---|
| POST | `/api/favorites` | 收藏新闻 | JWT |
| GET | `/api/favorites` | 收藏列表 | JWT |
| DELETE | `/api/favorites/{id}` | 取消收藏 | JWT |

#### 历史模块

| 方法 | 端点 | 说明 | 认证 |
|---|---|---|---|
| GET | `/api/history` | 浏览历史 | JWT |
| DELETE | `/api/history/{id}` | 删除单条 | JWT |
| DELETE | `/api/history` | 清空全部 | JWT |

---

## 🗄️ 数据库设计

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│    users      │     │     news      │     │  favorites    │
├──────────────┤     ├──────────────┤     ├──────────────┤
│ id (PK)      │     │ id (PK)      │     │ id (PK)      │
│ username     │     │ title        │     │ user_id (FK) ├──→ users
│ email        │     │ content      │     │ news_id (FK) ├──→ news
│ password     │     │ category     │     │ created_at   │
│ created_at   │     │ source       │     └──────────────┘
│ updated_at   │     │ publish_time │
└──────────────┘     │ created_at   │     ┌──────────────┐
       ↑              └──────────────┘     │   history     │
       │                          │        ├──────────────┤
       │              ┌───────────┘        │ id (PK)      │
       ├──────────────┤                    │ user_id (FK) ├──→ users
       │              │                    │ news_id (FK) ├──→ news
       │              │                    │ viewed_at    │
       │              │                    └──────────────┘
       │              │
  favorites.user_id    history.user_id
```

**设计要点**：
- 收藏和历史通过外键关联用户和新闻，保证数据一致性
- 密码使用 bcrypt 哈希存储，即使数据库泄露也无法还原明文
- 时间字段使用 UTC，前端展示时转换时区

---

