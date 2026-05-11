---
title: 记一次 Serverless 的认识与实践
date: 2026-03-10
category:
  - 运维
  - 云原生
tag:
  - Serverless
  - 云函数
---

# 记一次 Serverless 的认识与实践

前几年刚开始做 Web 项目时，我对“部署”的理解基本停留在一套固定流程：

- VPS
- Linux 环境
- Python / Node
- Nginx 反代
- systemd 挂进程
- 手动配 HTTPS

后面 Docker 用起来之后，这套流程至少不容易炸了。

但再往后做一些 AI Bot、Webhook、自动化工具时，我开始遇到一个问题：

有些功能其实根本不需要常驻一个完整服务。

---

# 什么是 Serverless

我第一次接触 Serverless 时的理解是：

```text
不用服务器了
```

后来才发现完全不是这个意思。

服务器依然存在，只是：

> 我不再负责维护运行环境。

我的实际感受是：

```text
我只写函数
平台负责执行
```

比如传统 FastAPI：

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

我需要处理：

- 进程守护
- 日志
- 端口
- 崩溃恢复
- 部署

而 Serverless 更接近：

```text
上传函数 → 直接执行
```

很多时候甚至不需要自己管 Linux。

---

# 我第一次真正用 Serverless 的场景

最开始不是 Web 项目，而是 AI 工具链。

当时我做了一些 Bot 自动化功能，里面有很多短生命周期逻辑：

- OCR
- 翻译
- OpenAI 请求转发
- 图片压缩
- 文件转换

这些有一个共同点：

> 调用一次就结束，没有长期运行需求。

---

# 一个典型处理流程

比如图片处理：

```text
上传图片
↓
压缩
↓
OCR
↓
返回文本
```

如果按传统方式，我会写成：

```text
Nginx → FastAPI → OCR 服务
```

但换成 Serverless 后：

```text
上传 → 云函数 → 返回结果
```

不需要常驻服务。

---

# 我实际试过的几个平台

## Vercel

主要用在前端项目上。

特点很明显：

- Next.js / Vue 体验很好
- 部署简单

但限制也很明确：

- 更偏前端生态
- 后端能力有限

---

## Cloudflare Workers

第一次写这种代码时有点冲击：

```js
export default {
  async fetch(request) {
    return new Response("ok");
  },
};
```

很轻量，而且延迟很低。

但我实际踩到的问题是：

- Python 支持弱
- 依赖受限
- 长任务不适合
- 文件能力有限

---

## 腾讯云 SCF

在国内项目里用过一段时间。

整体链路是：

- API 网关
- 云函数
- COS 存储
- 定时触发

对小工具来说够用。

---

# 我对 Serverless 的实际理解变化

一开始我以为后端一定是一个完整服务：

```text
一个项目 = 一个长期运行服务
```

后来逐渐拆成：

```text
一个功能 = 一个函数
```

比如：

```python
async def handler(req):
    data = await req.json()

    result = await ai_call(data["msg"])

    return {"result": result}
```

执行结束即销毁。

---

# Serverless 并不是万能的

我踩过几个比较明确的坑：

---

## 冷启动

第一次请求会明显变慢：

```text
启动运行环境 → 执行函数
```

在 Bot 场景里很明显，用户会感知延迟。

---

## 不适合长连接

我一开始试过把 Bot 核心逻辑迁移上去，结果很快发现不对。

例如：

- WebSocket
- IM Bot
- 实时推送

这些需要持续连接，而 Serverless 是短生命周期执行模型。

---

# 我后来实际的架构拆分

最终我采用的是混合结构：

```text
Bot 主体（常驻服务）
    ↓
Serverless（功能模块）
```

例如：

```text
QQ群消息
↓
NoneBot2
↓
调用云函数（AI / OCR / 处理）
↓
返回结果
```

代码大概是：

```python
async with httpx.AsyncClient() as client:
    resp = await client.post(
        API_URL,
        json={"msg": msg}
    )
```

---

# Serverless 更适合做什么

在我实际使用下来，它更适合：

- OCR
- AI 调用封装
- 图片处理
- Webhook
- 定时任务
- 小型工具函数

本质特点是：

> 输入 → 计算 → 输出

---

# Serverless 的问题也很现实

---

## 调试体验差

传统服务：

```text
docker logs
tail -f
```

Serverless：

```text
日志分散在平台控制台
```

定位问题成本更高。

---

## 包体限制

Python 项目经常遇到：

- opencv
- torch
- playwright

体积过大导致无法部署。

---

## 依赖环境不透明

经常出现：

```text
本地正常
线上异常
```

---

# 我现在的实际使用方式

目前比较稳定的结构是：

## 常驻服务

- FastAPI
- NoneBot
- Redis
- PostgreSQL

## Serverless

- OCR
- AI 转发
- Webhook
- 文件处理
- 定时任务

---

# Serverless 和 Docker 的关系

这两者不是替代关系。

在我实际理解里是：

| 技术       | 作用         |
| ---------- | ------------ |
| Docker     | 环境封装     |
| K8S        | 容器编排     |
| Serverless | 函数执行托管 |

很多 Serverless 底层本身也是容器。

---

# 一个更现实的趋势

在我做 AI Agent 相关项目后，这种感觉更明显：

很多能力本质都是函数：

- 搜索
- 翻译
- OCR
- 数据查询
- Prompt 处理

结构越来越像：

```text
输入 → 处理 → 输出
```

---

# 总结

Serverless 在我这里最终落地的理解是：

> 不是不需要服务器，而是不用为每个功能维护一个服务器。

它适合：

- 短任务
- 无状态逻辑
- 工具型功能

但对于：

- 长连接
- 高频服务
- 重计算任务

依然是传统 VPS + Docker 更合理。

```

```
