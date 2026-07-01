# 头条新闻后端服务

[![Python](https://img.shields.io/badge/python-3.12+-blue)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688)](https://fastapi.tiangolo.com/)
[![MySQL](https://img.shields.io/badge/MySQL-8.0+-4479A1)](https://www.mysql.com/)
[![Redis](https://img.shields.io/badge/Redis-7.x-DC382D)](https://redis.io/)

FastAPI 新闻应用后端服务，提供新闻浏览、用户认证、收藏和浏览历史的 RESTful API。与 [xwzx-news](../xwzx-news/) 前端项目配合使用。

---

## 📂 项目结构

```text
toutiao_backend/
├── main.py                  # 应用入口
├── config/
│   ├── db_conf.py           # 数据库配置
│   └── cache_conf.py        # Redis 配置
├── models/                  # SQLAlchemy ORM 模型
│   ├── news.py              #   新闻表
│   ├── users.py             #   用户表
│   ├── favorite.py          #   收藏表
│   └── history.py           #   历史表
├── schemas/                 # Pydantic 数据校验
│   ├── base.py              #   基础模型
│   ├── news.py              #   新闻模型
│   ├── users.py             #   用户模型
│   ├── favorite.py          #   收藏模型
│   └── history.py           #   历史模型
├── crud/                    # 业务逻辑
│   ├── news.py              #   新闻 CRUD
│   ├── users.py             #   用户 CRUD
│   ├── favorite.py          #   收藏 CRUD
│   ├── history.py           #   历史 CRUD
│   └── news_cache.py        #   新闻缓存
├── routers/                 # 路由模块
│   ├── news.py
│   ├── users.py
│   ├── favorite.py
│   └── history.py
├── utils/                   # 工具函数
│   ├── auth.py              #   JWT 认证
│   ├── security.py          #   密码哈希（bcrypt）
│   ├── exception.py         #   自定义异常
│   ├── exception_handlers.py #  全局异常处理
│   └── response.py          #   统一响应格式
├── cache/news_cache.py      # 本地缓存
├── pyproject.toml            # 依赖配置（uv）
├── .env.example              # 环境变量模板
└── test_main.http            # REST Client 测试文件
```

---

## 🚀 快速开始

环境要求：

| 组件 | 版本 |
|---|---|
| Python | 3.12+ |
| MySQL | 8.0+ |
| Redis | 7.x |
| uv | 最新版 |

```bash
# 1. 安装依赖并配置
uv sync
cp .env.example .env                         # 编辑 .env 填入数据库/Redis 配置

# 2. 创建数据库
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS toutiao CHARACTER SET utf8mb4;"

# 3. 启动
uv run uvicorn main:app --reload             # → http://localhost:8000/docs
```

**.env 示例：**

```env
ASYNC_DATABASE_URL=mysql+aiomysql://root:password@localhost:3306/toutiao
REDIS_HOST=localhost
REDIS_PORT=6379
SECRET_KEY=your-secret-key
```

---

## 📡 接口概览

### 新闻

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/api/news` | 新闻列表（支持分页、分类筛选） |
| GET | `/api/news/{id}` | 新闻详情 |
| GET | `/api/news/categories` | 分类列表 |

### 用户

| 方法 | 路径 | 说明 |
|---|---|---|
| POST | `/api/users/register` | 注册 |
| POST | `/api/users/login` | 登录 |
| GET | `/api/users/me` | 当前用户信息 |
| PUT | `/api/users/me` | 更新资料 |

### 收藏

| 方法 | 路径 | 说明 |
|---|---|---|
| POST | `/api/favorites` | 收藏新闻 |
| GET | `/api/favorites` | 收藏列表 |
| DELETE | `/api/favorites/{id}` | 取消收藏 |

### 历史

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/api/history` | 浏览历史 |
| DELETE | `/api/history/{id}` | 删除单条 |
| DELETE | `/api/history` | 清空全部 |

---

## 🗄️ 数据库设计

| 表 | 说明 | 主要字段 |
|---|---|---|
| `news` | 新闻资讯 | id, title, content, category, source, publish_time |
| `users` | 用户 | id, username, email, hashed_password |
| `favorites` | 收藏 | id, user_id, news_id, created_at |
| `history` | 浏览历史 | id, user_id, news_id, viewed_at |

---

## 🛠️ 技术特性

| 特性 | 实现 |
|---|---|
| 异步数据库 | SQLAlchemy 2.0+ asyncio + aiomysql |
| 密码安全 | bcrypt 哈希（passlib） |
| 认证 | JWT Token |
| 缓存 | Redis 缓存热点新闻 |
| 跨域 | CORS 中间件 |
| 异常处理 | 全局异常捕获 + 统一响应格式 |

---

