# 头条新闻后端服务

![Python](https://img.shields.io/badge/python-3.12+-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688)
![MySQL](https://img.shields.io/badge/MySQL-8.0+-4479A1)
![Redis](https://img.shields.io/badge/Redis-7.x-DC382D)
![Status](https://img.shields.io/badge/status-active-brightgreen)

FastAPI 新闻应用后端服务，提供新闻浏览、用户认证、收藏和历史记录的完整 RESTful API。与 [`xwzx-news`](../xwzx-news/) 前端项目配合使用。

## 项目结构

```text
toutiao_backend/
├── main.py                  # FastAPI 应用入口
├── config/
│   ├── db_conf.py           # 数据库连接配置
│   └── cache_conf.py        # Redis 缓存配置
├── models/                  # SQLAlchemy ORM 模型
│   ├── news.py              # 新闻表
│   ├── users.py             # 用户表
│   ├── favorite.py          # 收藏表
│   └── history.py           # 浏览历史表
├── schemas/                 # Pydantic 请求/响应模型
│   ├── base.py              # 基础 Schema
│   ├── news.py              # 新闻 Schema
│   ├── users.py             # 用户 Schema
│   ├── favorite.py          # 收藏 Schema
│   └── history.py           # 历史 Schema
├── crud/                    # 数据库 CRUD 操作
│   ├── news.py              # 新闻业务逻辑
│   ├── users.py             # 用户业务逻辑
│   ├── favorite.py          # 收藏业务逻辑
│   ├── history.py           # 历史业务逻辑
│   └── news_cache.py        # 新闻缓存逻辑
├── routers/                 # FastAPI 路由模块
│   ├── news.py              # 新闻路由
│   ├── users.py             # 用户路由
│   ├── favorite.py          # 收藏路由
│   └── history.py           # 历史路由
├── utils/                   # 工具函数
│   ├── auth.py              # JWT 认证
│   ├── security.py          # 密码哈希（bcrypt）
│   ├── exception.py         # 自定义异常
│   ├── exception_handlers.py # 全局异常处理
│   └── response.py          # 统一响应格式
├── cache/
│   └── news_cache.py        # 本地缓存工具
├── pyproject.toml            # 项目依赖 (uv)
├── .env.example              # 环境变量模板
└── test_main.http            # REST Client 测试文件
```

## 快速开始

### 环境要求

| 组件 | 版本 |
|---|---|
| Python | 3.12+ |
| MySQL | 8.0+ |
| Redis | 7.x |
| uv | 最新版 |

### 安装与配置

```bash
# 1. 安装依赖
uv sync

# 2. 创建环境变量文件
cp .env.example .env

# 3. 编辑 .env，配置数据库和 Redis
```

`.env` 示例：

```env
ASYNC_DATABASE_URL=mysql+aiomysql://root:password@localhost:3306/toutiao
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
SECRET_KEY=your-secret-key
```

### 初始化数据库

```bash
# 确保 MySQL 中已创建对应数据库
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS toutiao CHARACTER SET utf8mb4;"

# 启动服务后 ORM 会自动建表
```

### 启动

```bash
uv run uvicorn main:app --reload
```

访问 Swagger 文档：http://localhost:8000/docs

## API 概览

### 新闻接口

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/api/news` | 新闻列表（支持分页、分类筛选） |
| GET | `/api/news/{id}` | 新闻详情 |
| GET | `/api/news/categories` | 新闻分类列表 |

### 用户接口

| 方法 | 路径 | 说明 |
|---|---|---|
| POST | `/api/users/register` | 用户注册 |
| POST | `/api/users/login` | 用户登录 |
| GET | `/api/users/me` | 获取当前用户信息 |
| PUT | `/api/users/me` | 更新用户信息 |

### 收藏接口

| 方法 | 路径 | 说明 |
|---|---|---|
| POST | `/api/favorites` | 收藏新闻 |
| GET | `/api/favorites` | 收藏列表 |
| DELETE | `/api/favorites/{id}` | 取消收藏 |

### 历史接口

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/api/history` | 浏览历史列表 |
| DELETE | `/api/history/{id}` | 删除单条历史 |
| DELETE | `/api/history` | 清空全部历史 |

## 数据库设计

### 核心表

| 表名 | 说明 | 主要字段 |
|---|---|---|
| `news` | 新闻资讯 | id, title, content, category, source, publish_time |
| `users` | 用户 | id, username, email, hashed_password |
| `favorites` | 收藏 | id, user_id, news_id, created_at |
| `history` | 浏览历史 | id, user_id, news_id, viewed_at |

## 技术特性

| 特性 | 实现 |
|---|---|
| 异步数据库 | SQLAlchemy 2.0+ asyncio + aiomysql |
| 密码安全 | bcrypt 哈希（passlib） |
| 认证机制 | JWT Token |
| 缓存加速 | Redis 缓存热点新闻 |
| 跨域支持 | CORS 中间件 |
| 异常处理 | 全局异常捕获 + 统一响应格式 |
| 数据校验 | Pydantic v2 |

## 与前端联调

1. 确保后端服务已启动
2. 前端 `xwzx-news` 配置 API 基础地址指向本服务
3. 如遇跨域问题，检查 `main.py` 中 CORS `allow_origins` 配置

## 相关链接

- 前端项目：[xwzx-news](../xwzx-news/)
- FastAPI 学习笔记：[FastAPI_first](../FastAPI_first/)
