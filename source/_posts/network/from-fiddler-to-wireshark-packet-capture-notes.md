---
title: 从 Fiddler 到 Wireshark：抓包技术整理
date: 2021-04-14
tags:
  - 抓包
  - Fiddler
categories:
  - Web安全
---

# 从 Fiddler 到 Wireshark：抓包技术整理

最开始接触抓包，其实不是为了学习网络协议，而是很功利的目的：

- 看接口返回结构
- 改请求参数
- 找接口逻辑
- 做自动化
- 写 Bot

后来做的事情多了之后才发现，抓包本质是在看：

> 客户端到底是怎么和服务端通信的。

---

# 我接触抓包的工具演进

大致是这么一路过来的：

```text id="zv9k0b"
浏览器 F12
→ Fiddler
→ Charles / 小黄鸟
→ BurpSuite
→ Wireshark
```

不同阶段解决的问题不一样：

| 工具      | 我当时主要用来干什么 |
| --------- | -------------------- |
| F12       | 看接口               |
| Fiddler   | HTTP/HTTPS 抓包      |
| Charles   | 手机抓包             |
| BurpSuite | 改包 / 重放          |
| Wireshark | 看底层网络           |

---

# 抓包的本质

我后来对它的理解其实很简单：

```text id="2kqg7n"
把客户端和服务端之间的通信拦下来
```

绝大多数工具都围绕一个核心机制：

```text id="p0qv3s"
代理（Proxy）
```

链路变成：

```text id="l2m1xq"
客户端 → 代理工具 → 服务器
```

---

# HTTP 抓包阶段：Fiddler

## 为什么 Fiddler 是起点

因为它刚好解决我最早的需求：

- 看接口
- 改参数
- 看返回 JSON
- 调试 APP 请求

界面是直接按请求分块展示：

- Headers
- Body
- Cookies
- Raw

对当时的我来说就是“可视化接口抓取器”。

---

# HTTPS 为什么还能被抓

这是我第一次觉得“网络有点不直觉”的地方。

HTTPS 按理是加密的，但 Fiddler 还是能看到内容。

原因其实是：

## 中间人代理（MITM）

流程变成：

```text id="9v7c0a"
客户端 → Fiddler（伪造证书）→ 真实服务器
```

关键点是：

- Fiddler 生成自己的证书
- 客户端信任这个证书
- 通信被解密再转发

---

# HTTPS 抓包踩过的第一个坑：证书

PC 端还好，安装证书就行。

但移动端开始变复杂。

---

# 安卓抓包的真实问题

## Android 7+ 的限制

我遇到的第一个问题是：

```text id="m2c8xk"
系统不信任用户安装的证书
```

结果就是：

```text
明明配置正确，但抓不到 HTTPS
```

---

## 真机抓包流程

基本流程是：

```text id="q1t8pd"
手机设置代理 → 电脑 IP:端口 → 安装证书
```

但后面会遇到更麻烦的问题：

---

# APP 反抓包机制

很多 APP 开始做：

- 证书校验
- SSL pinning
- 代理检测

结果就是：

```text id="8n4dqp"
抓包工具失效
```

---

# Certificate Pinning（证书锁死）

这是我第一次真正被“反制”的点。

正常 HTTPS：

```text id="c0m9rp"
系统信任 CA
```

但 Pinning 是：

```text id="t7q2nv"
APP 内部写死服务器证书
```

所以即使我装了代理证书：

```text
依然失败
```

---

# 常见绕过方式（实际踩过）

## Frida Hook

通过 hook：

```java id="f8v3js"
checkServerTrusted()
```

直接绕过校验。

---

## Root / LSPosed

修改系统信任链。

---

这些方法我实际都试过，但维护成本很高。

---

# BurpSuite：进入“改请求阶段”

从 Fiddler 到 Burp 是一个明显分界点。

Fiddler 是“看数据”。

Burp 更像：

```text id="b3k9wp"
改数据 + 重放数据
```

---

## Repeater（最关键功能）

第一次用 Repeater 时，我才意识到：

```text id="r9m2qv"
HTTP 请求是可以手工拼的
```

于是可以：

- 改 token
- 改参数
- 改 user-agent
- 重放请求

---

# Proxifier：解决“抓不到流量”的问题

有些程序根本不走系统代理，比如：

- 游戏
- Electron 应用
- 一些客户端 EXE

这时候 Fiddler 是无效的。

---

## 我的解决方式

```text id="p5v1kq"
强制进程走代理 → Proxifier
```

链路变成：

```text id="x8m2wr"
应用 → Proxifier → Fiddler → 服务器
```

---

# Wireshark：进入更底层

前面工具本质都还在 HTTP 层。

Wireshark 是另一个层面：

```text id="w4n9qk"
直接抓 TCP / IP 数据包
```

---

## 我第一次看到 TCP 三次握手

```text id="s2v7dm"
SYN → SYN ACK → ACK
```

课本内容第一次变成真实流量。

---

## Wireshark 能看到什么

- TCP
- UDP
- DNS
- TLS
- WebSocket
- QUIC

---

# Wireshark 的局限

问题也很明显：

```text id="k1p6zv"
看不到业务数据
```

因为 HTTPS 已经加密。

所以结论是：

```text id="h7q0cm"
Fiddler 看业务，Wireshark 看网络
```

---

# WebSocket 和现代接口

后来我抓到越来越多：

- IM
- Bot
- AI接口
- 游戏同步

基本都变成：

```text id="w9c3pv"
WebSocket
```

---

# Protobuf 乱码问题

最头疼的一次是：

```text
抓到数据全是二进制
```

后来才知道是：

```text id="p6n8xt"
protobuf
```

没有 `.proto` 文件基本无法解析。

---

# 我对抓包的最终理解

一开始我觉得抓包是：

```text
改接口 / 看数据
```

后来变成：

```text id="u3q7km"
观察系统真实通信方式
```

因为它直接暴露：

- 鉴权逻辑
- 接口结构
- 数据流向
- 客户端行为
- 安全策略

---

# 总结

抓包不是某个工具的能力。

而是我在开发过程中逐渐形成的一种能力：

> 看清请求是怎么从客户端流向服务端的。

工具只是入口：

- Fiddler → HTTP层
- Burp → 交互层
- Wireshark → 协议层

而真正变化的是：

```text id="z5m0qp"
从“调用接口的人” → “看懂通信的人”
```
