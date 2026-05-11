---
title: FastAPI、Flask、Django：我为什么最终选择 FastAPI
date: 2022-02-18
category:
  - 后端开发
  - Python
  - Web
tag:
  - FastAPI
  - Flask
  - Django
---

# FastAPI、Flask、Django：我为什么最终选择 FastAPI

Python Web 开发领域。

长期以来基本绕不开三个框架：

- Flask
- Django
- FastAPI

它们分别代表了：

- 极简灵活
- 大而全
- 现代异步 API 工程

很多新手会问：

```text
“到底该学哪个？”
```

但实际上。

这个问题应该拆成：

```text
“你的项目到底需要什么？”
```

因为：

三个框架的设计目标本身就不一样。

---

# Flask：轻量、自由、低约束

Flask 的核心特点是：

```text
轻量
```

一个最简单的 Flask 项目：

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "hello"
```

直接即可运行。

---

# Flask 的优势

Flask 本质是：

```text
微框架（Micro Framework）
```

它只提供：

- 路由
- 请求响应
- 基础 Web 能力

其余：

- ORM
- 权限
- Session
- 管理后台

都由开发者自行选择。

---

# Flask 适合什么项目

Flask 很适合：

- 小型项目
- 原型验证
- 工具型接口
- 简单后台
- 个人项目

因为：

它的结构自由度非常高。

---

# Flask 的问题

自由也是代价。

项目规模一旦变大。

容易出现：

- 目录结构混乱
- 全局变量泛滥
- 缺少统一规范
- 多人协作困难

尤其：

不同开发者可能写出完全不同的项目结构。

---

# Django：传统 Web 全家桶

Django 的核心特点是：

```text
完整
```

它自带：

- ORM
- Admin 后台
- 用户系统
- Session
- 权限系统
- 模板引擎

属于典型：

```text
开箱即用
```

框架。

---

# Django 最大优势

Django 最大优势其实是：

```text
成熟业务生态
```

尤其适合：

- OA
- CMS
- ERP
- 管理系统
- 后台系统

很多企业内部系统。

至今仍然大量使用 Django。

---

# Django Admin 为什么强

Django Admin 可以根据 Model：

直接生成后台管理界面。

例如：

```python
class User(models.Model):
    name = models.CharField(max_length=20)
```

注册后：

直接生成：

- 增删改查
- 搜索
- 筛选
- 后台页面

开发效率非常高。

---

# Django 的问题

Django 的设计理念。

本质仍偏向：

```text
传统 MVC Web
```

即：

后端负责：

- 页面渲染
- 模板
- Session

但现代 Web：

大量已经转向：

```text
前后端分离
```

例如：

- Vue
- React
- API 服务

这种情况下。

Django 会显得：

```text
偏重
```

---

# FastAPI 的核心定位

FastAPI 从设计开始。

目标就很明确：

```text
现代 Python API 框架
```

重点是：

- 类型提示
- 异步
- OpenAPI
- 数据校验
- API 工程化

它不是传统 MVC 框架。

而是：

```text
API First
```

框架。

---

# FastAPI 为什么会流行

FastAPI 的爆发。

本质上是因为：

现代 Python 开发方向变了。

---

# Python 后端正在 API 化

越来越多项目：

已经不再：

```text
后端渲染 HTML
```

而是：

```text
后端提供 JSON API
```

前端：

- Vue
- React
- App
- 小程序

自行渲染。

---

# AI 与微服务进一步推动 FastAPI

后来：

- AI 服务
- Agent
- 微服务
- 异步任务
- WebSocket

大量出现。

这些都更偏向：

```text
高并发 API 服务
```

而不是传统 Django 页面架构。

---

# FastAPI 最核心的优势

FastAPI 真正强的地方。

并不是：

```text
性能
```

而是：

```text
现代 Python 工程体验
```

---

# 1. 类型提示

例如：

```python
@app.get("/user/{user_id}")
async def get_user(user_id: int):
    return {"id": user_id}
```

这里：

- 参数类型
- 返回结构

都是明确的。

---

# 类型提示的意义

类型提示不仅是：

```text
代码可读性
```

更重要的是：

- IDE 自动补全
- 静态检查
- 参数校验
- 自动文档

都会因此受益。

---

# 2. Pydantic 数据模型

FastAPI 的核心基础之一：

是 Pydantic。

例如：

```python
from pydantic import BaseModel

class UserCreate(BaseModel):
    name: str
    age: int
```

接口：

```python
@app.post("/user")
async def create_user(user: UserCreate):
    return user
```

---

# Pydantic 解决了什么问题

传统 Flask 项目中。

很多参数校验是：

```python
request.json.get("name")
```

手动处理。

容易出现：

- 类型错误
- 字段缺失
- 数据污染

而 Pydantic：

直接实现：

- 类型校验
- 默认值
- 自动转换
- 数据约束

---

# 3. 自动 API 文档

FastAPI 默认集成：

- Swagger UI
- OpenAPI

启动项目后：

直接访问：

```text
/docs
```

即可查看接口文档。

---

# 自动文档的价值

它会自动生成：

- 请求参数
- JSON Schema
- 返回值
- 类型说明

前后端协作效率会高很多。

---

# 4. Async 异步支持

FastAPI 原生支持：

```python
async def
```

这是它与 Flask、Django 最大的区别之一。

---

# 为什么异步重要

现代 Web 服务。

大量时间都消耗在：

- 数据库 IO
- Redis
- HTTP 请求
- 文件读写

真正占 CPU 的时间反而不多。

---

# Async 的意义

异步并不是：

```text
让代码变快
```

而是：

```text
提高 IO 并发能力
```

---

# FastAPI 的异步生态

FastAPI 周围形成了完整异步生态：

- httpx
- asyncpg
- SQLAlchemy Async
- websockets

这一点对于：

- AI 服务
- 实时系统
- 高频 API

非常重要。

---

# 5. Dependency Injection

FastAPI 内置：

```text
依赖注入
```

例如：

```python
from fastapi import Depends

async def get_db():
    return db

@app.get("/")
async def index(db = Depends(get_db)):
    pass
```

---

# 依赖注入解决的问题

大型项目里。

很多对象：

会频繁重复使用。

例如：

- 数据库连接
- Redis
- 当前用户
- 配置对象

Depends 可以：

显式声明依赖关系。

避免：

- 全局变量污染
- 代码耦合

---

# 6. Router 工程化

FastAPI 非常适合模块拆分。

例如：

```python
app.include_router(user_router)
```

大型项目通常会拆成：

```text
routers/
services/
models/
schemas/
dependencies/
```

这种结构。

---

# FastAPI 的问题

FastAPI 并不是没有缺点。

---

# 1. 后台生态弱

它不像 Django：

自带：

- Admin
- ORM 后台
- 权限后台

很多东西需要自行实现。

---

# 2. 异步生态曾经很混乱

早期：

很多异步 ORM：

变化非常频繁。

例如：

- gino
- databases
- tortoise

不同版本兼容问题很多。

---

# 3. 异步会增加复杂度

Async 并不是银弹。

很多新人容易：

- 阻塞事件循环
- 混用同步库
- await 使用错误

导致性能问题。

---

# Flask、Django、FastAPI 怎么选

实际上：

取决于项目需求。

---

# Flask 更适合

- 小型项目
- 工具型接口
- 原型开发
- 极简服务

---

# Django 更适合

- 管理后台
- CMS
- 企业内部系统
- 快速 CRUD

---

# FastAPI 更适合

- 前后端分离
- API 服务
- AI 服务
- 微服务
- WebSocket
- 高 IO 并发

---

# 为什么 AI 项目大量选择 FastAPI

因为 AI 系统天然偏向：

```text
API化
```

同时：

- Streaming
- Async
- WebSocket
- 多模型调用

需求非常多。

FastAPI 非常契合这类场景。

---

# 最后总结

Flask、Django、FastAPI。

本质上代表了：

三种不同的 Web 开发哲学。

---

# Flask

```text
自由优先
```

---

# Django

```text
完整优先
```

---

# FastAPI

```text
现代 API 工程优先
```

---

# 当前 Python Web 的整体趋势

实际上已经越来越明显：

```text
API化
异步化
类型化
工程化
```

而 FastAPI：

基本就是这一趋势下的产物。
