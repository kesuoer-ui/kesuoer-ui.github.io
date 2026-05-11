---
title: 我用 FastAPI 开发 Web 应用的实践整理
date: 2022-01-08
category:
  - 后端开发
  - Python
  - Web
tag:
  - FastAPI
  - PostgreSQL
  - ORM
---

# 我用 FastAPI 开发 Web 应用的实践整理

这个项目本质上是一个：

```text
图片社区类 Web 应用
```

功能方向类似：

- Pinterest
- Pixiv
- 图片收藏站

核心功能包括：

- 用户注册登录
- 图片上传
- 点赞收藏
- 评论互动
- 标签系统
- 图片来源管理
- 用户行为记录

项目重点并不在前端。

而在于：

```text id="g1p4t2"
现代 Python 后端工程化
```

包括：

- FastAPI
- 异步 ORM
- PostgreSQL
- JWT
- Redis
- Async 架构

这些完整技术链路。

---

# 技术栈

项目后端技术栈大致如下：

```text id="brpzw0"
FastAPI
PostgreSQL
GINO
asyncpg
Redis
JWT
Uvicorn
Pydantic
```

---

# 为什么选择 FastAPI

这个项目本质是：

```text id="bb9o0r"
前后端分离 API 服务
```

因此：

- 类型系统
- 自动文档
- 异步能力
- 数据校验

会非常重要。

FastAPI 在这些方面体验明显优于传统 Flask。

---

# 为什么选择 PostgreSQL

数据库最开始考虑过 MySQL。

但最终还是选择 PostgreSQL。

原因主要有几个。

---

# 1. JSON 能力更强

很多业务数据：

其实是：

```text id="n8ez6w"
弱结构化
```

例如：

- 标签
- 扩展属性
- 元数据
- 图片信息

使用 PostgreSQL 的 JSON 字段会非常方便。

---

# 2. 类型系统更完整

PostgreSQL：

支持：

- JSON
- ARRAY
- UUID
- ENUM

复杂业务建模会更舒服。

---

# 3. asyncpg 性能很好

Python 异步生态里。

```text id="3mqjlwm"
asyncpg
```

性能一直很强。

而很多异步 ORM：

底层本质也是：

```text id="4yzjlwm"
基于 asyncpg
```

---

# 为什么使用 GINO

项目开发时期。

SQLAlchemy Async：

还不算成熟。

因此选择了：

```text id="xjlwm8"
GINO
```

---

# GINO 本质是什么

GINO 本质上：

可以理解为：

```text id="mjlwm5"
asyncpg + ORM 封装
```

提供：

- Model
- Query
- Async ORM

能力。

---

# 一个简单 Model

例如：

```python id="c8w4x2"
class User(db.Model):
    __tablename__ = "users"

    id = db.Column(db.Integer(), primary_key=True)
    username = db.Column(db.String())
```

---

# 异步查询

查询时：

```python id="1e6r1v"
user = await User.query.where(
    User.id == user_id
).gino.first()
```

这里：

```python id="ixtmcm"
await
```

本质是在等待：

```text id="91x5m8"
数据库 IO
```

完成。

---

# Async 在 Web 服务里的意义

很多人误以为：

异步是：

```text id="p3zjcb"
让代码执行更快
```

实际上：

异步核心意义是：

```text id="1kn93n"
减少 IO 阻塞
```

---

# Web 服务里真正耗时的部分

通常是：

- 数据库
- Redis
- HTTP请求
- 文件读写

而不是 CPU。

---

# 项目目录结构

项目后期基本拆成：

```text id="m57v8j"
app
├── api
├── models
├── schemas
├── services
├── dependencies
├── middleware
├── utils
└── core
```

---

# 各层职责

---

## api

负责：

```text id="15jppq"
路由与接口定义
```

例如：

```python id="2r7g43"
@router.get("/images")
```

---

## models

负责：

```text id="0j84u3"
数据库模型
```

---

## schemas

负责：

```text id="5e6r3w"
Pydantic 数据结构
```

包括：

- 请求体
- 返回结构
- 参数校验

---

## services

负责：

```text id="0gx7p4"
业务逻辑
```

避免：

```text id="44o7hl"
router 里塞满业务代码
```

---

## middleware

负责：

- 日志
- JWT解析
- 请求耗时
- 异常处理

---

# Pydantic 在项目里的作用

FastAPI 真正的核心之一：

其实是：

```text id="efrjlwm"
Pydantic
```

---

# 一个用户创建 Schema

例如：

```python id="sjl97x"
class UserCreate(BaseModel):
    username: str
    password: str
```

接口：

```python id="u6k5dy"
@router.post("/register")
async def register(user: UserCreate):
    pass
```

---

# Pydantic 的价值

它会自动处理：

- 类型校验
- 缺失字段
- JSON解析
- 参数转换

避免：

大量：

```python id="j1r7p2"
request.json.get()
```

式代码。

---

# 用户系统

用户系统核心包括：

- 注册
- 登录
- JWT
- 权限校验

---

# 密码为什么不能明文

密码必须：

```text id="z72udg"
哈希存储
```

常见方案：

- bcrypt
- argon2

---

# JWT 的作用

前后端分离后。

大量项目开始使用：

```text id="jlwm1t"
JWT
```

作为登录态。

---

# JWT 本质

JWT 本质是：

```text id="jlwm2x"
签名令牌
```

不是加密。

---

# JWT 通常包含

- 用户ID
- 过期时间
- 权限信息

---

# FastAPI 鉴权依赖

通常会配合：

```python id="n1s5i5"
Depends()
```

实现。

例如：

```python id="6zv2xg"
async def get_current_user():
    pass
```

---

# 图片上传系统

图片系统实际上比 CRUD 复杂很多。

---

# 需要考虑的问题

包括：

- 文件命名
- 重复上传
- 缩略图
- 原图存储
- CDN
- 图片压缩
- 路径管理

---

# 为什么后期很多项目会对象存储化

因为：

本地文件：

很难处理：

- 扩容
- 多机器部署
- CDN
- 高并发读取

因此很多项目最终都会：

```text id="jlwm7m"
OSS化
```

---

# 点赞系统的实际问题

点赞并不只是：

```sql id="jlwm4s"
count + 1
```

---

# 实际需要考虑

包括：

- 重复点赞
- 取消点赞
- 并发问题
- 热门排序
- Redis缓存

---

# 评论系统的问题

评论系统后期会逐渐复杂。

例如：

- 楼中楼
- 分页
- 删除状态
- 审核状态
- 敏感词

---

# Redis 在项目里的作用

Redis 主要用于：

- 热门数据缓存
- 登录状态
- 验证码
- 高频读取

---

# 为什么需要 Redis

数据库不适合：

```text id="jlwm6y"
高频热点读取
```

尤其：

排行榜。

---

# Middleware 的作用

中间件负责：

```text id="jlwm9o"
请求生命周期处理
```

---

# 常见中间件

包括：

- 请求日志
- Trace ID
- JWT解析
- 异常捕获
- 耗时统计

---

# 异步项目最大的坑

真正踩坑最多的地方。

其实是：

```text id="jlwm0v"
同步库混入异步项目
```

---

# 最典型错误

例如：

```python id="jlwm2q"
requests.get()
```

出现在：

```python id="jlwm7p"
async def
```

内部。

---

# 为什么危险

因为：

requests 是同步阻塞。

会直接：

```text id="jlwm1z"
阻塞事件循环
```

---

# 正确做法

异步项目里：

应该使用：

```text id="jlwm8f"
httpx.AsyncClient
```

等异步库。

---

# FastAPI 自动文档

FastAPI 默认生成：

- Swagger
- OpenAPI

访问：

```text id="jlwm5x"
/docs
```

即可查看接口。

---

# 自动文档的意义

它会自动生成：

- 请求参数
- Schema
- 类型
- 返回值

非常适合：

前后端协作。

---

# 部署方式

项目最终通常使用：

```text id="jlwm2n"
uvicorn
```

启动。

例如：

```bash id="jlwm5l"
uvicorn main:app --host 0.0.0.0 --port 8000
```

生产环境一般会结合：

- nginx
- gunicorn
- docker

一起部署。

---

# 后期最大的认知变化

真正的 Web 项目。

难点通常不是：

```text id="jlwm9w"
功能开发
```

而是：

```text id="jlwm8x"
工程维护
```

---

# 包括：

- 项目结构
- 数据一致性
- 权限
- 缓存
- 并发
- 异步
- 日志
- 部署

这些问题。

---

# FastAPI 真正适合什么场景

FastAPI 最适合：

- API服务
- AI服务
- 微服务
- 异步系统
- WebSocket
- 前后端分离

---

# 最后总结

现代 Python Web 开发。

已经越来越偏向：

```text id="jlwm4r"
API化
异步化
类型化
工程化
```

FastAPI 基本就是围绕这一趋势设计的框架。

而：

- PostgreSQL
- Async ORM
- Redis
- JWT

这些组件。

共同组成了现代 Python Web 后端的核心工程体系。
