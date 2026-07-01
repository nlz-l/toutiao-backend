# 头条新闻后端服务

这是一个基于 FastAPI 的新闻应用后端练习项目，可与 `xwzx-news` 前端配合使用。项目实现新闻、用户、收藏和浏览历史等模块，并使用异步 SQLAlchemy 连接 MySQL，使用 Redis 辅助缓存或状态管理。

## 功能模块

- 新闻接口：新闻列表、分类、详情等相关能力
- 用户接口：注册、登录、用户信息等相关能力
- 收藏模块：记录和查询用户收藏新闻
- 历史模块：记录和查询用户浏览历史
- 异步数据库访问：SQLAlchemy asyncio + aiomysql
- Redis 工具：缓存或临时数据支持
- CORS 配置和统一异常处理

## 目录结构

```text
config/        配置项和环境变量读取
crud/          数据库增删改查逻辑
models/        SQLAlchemy ORM 模型
routers/       FastAPI 路由模块
schemas/       Pydantic 请求/响应模型
utils/         数据库、Redis、密码等工具函数
cache/         本地缓存或临时数据
main.py        应用入口
test_main.http 接口测试请求
```

## 环境要求

- Python 3.12 或更高版本
- uv
- MySQL
- Redis

## 安装依赖

```bash
uv sync
```

复制环境变量模板：

```bash
copy .env.example .env
```

在 `.env` 中配置数据库和 Redis，例如：

```text
ASYNC_DATABASE_URL=mysql+aiomysql://user:password@localhost:3306/db_name
REDIS_HOST=localhost
REDIS_PORT=6379
```

具体字段以 `.env.example` 和代码读取逻辑为准。

## 启动服务

```bash
uv run uvicorn main:app --reload
```

启动后可以访问：

```text
http://localhost:8000/docs
```

FastAPI 会自动生成 Swagger 接口文档，便于测试新闻、用户、收藏和历史相关接口。

## 联调说明

前端项目 `xwzx-news` 需要把 API 基础地址指向本服务地址。若浏览器出现跨域问题，请检查后端 CORS 配置和前端开发服务器地址是否一致。

## 学习重点

- FastAPI 路由拆分和模块化组织
- Pydantic Schema 与 ORM Model 的分层
- 异步 SQLAlchemy 数据库操作
- 登录密码哈希、用户数据校验和接口异常处理
- 前后端分离新闻项目的基础业务闭环
