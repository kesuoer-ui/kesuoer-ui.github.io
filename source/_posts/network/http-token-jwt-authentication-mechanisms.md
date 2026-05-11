---
title: HTTP、Token、JWT 与网页鉴权机制整理
date: 2021-09-19
tags:
  - JWT
  - Token
  - OAuth
categories:
  - Web安全
---

# HTTP、Token、JWT 与网页鉴权机制整理

很多人第一次做前后端分离时都会疑惑：

```text
用户到底是怎么“登录状态持久化”的？
```

浏览器关了。

APP 重启了。

为什么服务端还知道我是谁？

这背后其实就是：

```text
鉴权（Authentication）
```

---

# HTTP 为什么天然无状态

HTTP 本质：

```text
请求 -> 响应
```

服务端默认：

```text
不记得我是谁
```

即：

```text
每一次请求都是独立事件
```

所以：

- 登录状态
- 用户身份
- 权限信息

都必须额外设计机制来承载。

---

# Cookie：最早的状态方案

最传统的方式：

```text
Cookie + Session
```

流程是：

```text
登录成功
    ↓
服务端生成 Session
    ↓
SessionID 写入 Cookie
    ↓
浏览器后续自动携带 Cookie
```

---

# Session 本质

服务端通常存：

```json
{
  "session_id": {
    "user_id": 123
  }
}
```

位置可能是：

- 内存
- Redis
- 数据库

---

# Cookie 的核心特点

浏览器会：

```text
自动携带 Cookie
```

因此早期 Web 系统基本依赖它完成登录态维持。

---

# Cookie 的问题

## 1. 跨域复杂

前后端分离后常见：

```text
前端 localhost:5173
后端 localhost:8000
```

随之出现：

- SameSite
- CORS
- Secure
- HttpOnly

一整套配置问题。

---

# Token 机制的出现

移动端普及之后，Cookie 不再是唯一选择。

因为：

```text
APP 没有浏览器 Cookie 机制
```

于是 Token 成为主流方案。

---

# Token 本质

服务端：

```text
登录成功 -> 生成字符串
```

客户端请求：

```http
Authorization: Bearer xxx
```

服务端只负责验证：

```text
token 是否有效
```

---

# Token 与 Cookie 的区别

| 机制    | 状态位置 |
| ------- | -------- |
| Session | 服务端   |
| Token   | 客户端   |

---

# JWT 为什么被广泛使用

JWT（Json Web Token）核心特性：

```text
无状态认证
```

---

# JWT 结构

典型 JWT：

```text
xxxxx.yyyyy.zzzzz
```

三部分：

| 结构      | 含义     |
| --------- | -------- |
| Header    | 算法信息 |
| Payload   | 用户数据 |
| Signature | 签名校验 |

---

# Payload 的特点

Payload 通常是：

```json
{
  "user_id": 1,
  "username": "admin"
}
```

它可以直接 Base64 解码。

因此需要明确一点：

## JWT 默认不加密，只防篡改

---

# JWT 工作流程

```text
登录
   ↓
签发 JWT
   ↓
客户端保存
   ↓
请求携带 JWT
   ↓
服务端验签
```

---

# JWT 的优势

在分布式系统中：

```text
不需要查 Session
```

每个服务只需要：

```text
共享密钥
```

即可完成验证。

---

# JWT 的核心问题

## 无法主动失效

因为：

```text
服务端不保存状态
```

因此 token 一旦签发：

```text
理论上直到过期都有效
```

---

# Refresh Token 机制

现代系统通常拆分：

- Access Token（短期）
- Refresh Token（长期）

---

# 设计目的

如果 Access Token 泄露：

```text
影响窗口较小
```

Refresh Token 用于：

```text
重新获取 Access Token
```

---

# OAuth2：第三方授权体系

常见场景：

- GitHub 登录
- Google 登录
- 微信登录

核心思想：

```text
不把密码交给第三方
```

---

# OAuth2 流程

```text
用户跳转授权页
    ↓
官方确认授权
    ↓
返回 access_token
```

---

# OAuth2 的意义

本质是：

```text
授权代理机制
```

而不是简单登录替代。

---

# SSO 单点登录

企业系统常见：

```text
一次登录，多系统共享状态
```

常见实现：

- CAS
- OAuth2
- SAML

---

# 实际工程中的组合方案

很多系统并不会只用 JWT。

更常见组合是：

```text
JWT + Redis
```

---

# 为什么需要 Redis

纯 JWT 的问题：

```text
无法强制失效
```

解决方式：

- 黑名单
- token version 控制

---

# 鉴权头的几种形式

## Authorization（标准）

```http
Authorization: Bearer xxx
```

---

## Cookie（传统 Web）

```http
Cookie: sessionid=xxx
```

---

## 自定义 Header

```http
X-Token: xxx
```

---

# 签名机制

部分系统会引入签名：

```text
timestamp + nonce + sign
```

---

# 为什么需要签名

用于防护：

- 重放攻击
- 参数篡改
- 接口伪造

---

# HMAC 签名模型

典型方式：

```text
HMAC-SHA256(secret, data)
```

客户端与服务端使用相同规则计算签名。

---

# WebSocket 鉴权

长连接场景：

```text
WS 建立后持续通信
```

常见做法：

```text
连接时携带 token
```

例如：

```text
wss://xxx?token=abc
```

---

# 鉴权的本质问题

我在实际开发中逐渐明确的一点是：

## 鉴权难点不在“登录”

而在：

- 权限边界
- 数据隔离
- 行为约束

---

# RBAC 权限模型

典型结构：

```text
用户 -> 角色 -> 权限
```

---

# 更细粒度权限系统

例如：

```text
只能访问自己的资源
```

或者：

```text
按部门 / 项目隔离
```

---

# 我对鉴权体系的整体理解

很多人容易把 JWT 等同于安全机制，但实际不是。

真正的安全体系通常由以下构成：

- HTTPS
- 服务端校验
- 权限控制逻辑
- Token 生命周期管理
- 风控与限流
- 密钥管理

---

# JWT 的真实定位

JWT 更准确的定位是：

```text
身份载体
```

而不是安全本身。

---

# 常见误区

很多接口即使带 JWT：

```text
仍然可能越权
```

原因通常在于：

```text
业务层没有做权限判断
```

例如：

```python
DELETE /post/123
```

如果没有校验：

```text
当前用户是否为作者
```

鉴权体系依然是缺失的。

---

# 总结

HTTP 的无状态决定了：

```text
所有登录态必须外置设计
```

而 Cookie、Token、JWT、OAuth2，本质都是：

```text
围绕“如何表达用户身份”这一问题的不同实现方案
```

其中 JWT 解决的是：

```text
跨服务的身份传递问题
```

但安全性本身仍然依赖完整的服务端权限体系。
