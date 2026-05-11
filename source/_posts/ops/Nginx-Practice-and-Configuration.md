---
title: Nginx 在实际开发中的用途整理
date: 2022-02-08
tags:
  - Nginx
  - Linux
categories:
  - 运维
---

# Nginx 在实际开发中的用途整理

## 前言

我第一次认真接触 Nginx，其实不是因为“想学 Web Server”，而是因为一堆线上问题开始集中爆发：

- 前端部署后 404
- API 跨域
- 502 Bad Gateway
- WebSocket 连不上
- Docker 容器互相访问失败
- 静态资源加载异常

这些问题最后都指向同一个东西：

> Nginx 是整个请求链路的入口。

后来在实际项目里，我逐渐把它理解成一个“流量分发与边界控制层”，而不只是静态文件服务器。

---

# 一、Nginx 到底是什么

本质上我对它的定义是：

> 一个高性能 HTTP 服务器 + 反向代理 + 流量入口控制器

它在我的项目里通常承担这些角色：

- 静态资源服务
- 反向代理
- 负载均衡
- HTTPS 终止
- API 网关雏形
- WebSocket 转发入口

典型链路是：

```text
用户
 ↓
Nginx
 ↓
后端服务（FastAPI / SpringBoot / Node）
 ↓
数据库
```

---

# 二、为什么实际项目离不开 Nginx

在纯开发阶段，我最初其实是直接跑后端服务的，比如：

```text
uvicorn main:app
```

但一旦进入部署阶段，会立刻遇到几个现实问题：

## 1. 静态资源效率问题

直接用后端服务返回静态资源，性能明显不如 Nginx。

所以我后来统一改成：

```text
Nginx 负责静态资源
后端只负责 API
```

---

## 2. 反向代理成为必需

实际部署时，我不会直接暴露后端端口，而是：

```text
公网 → Nginx → 内部服务
```

这样做的好处很直接：

- 后端服务隐藏
- 统一入口
- 安全控制集中

---

## 3. 前后端分离后的路径问题

典型结构：

```text
Vue / React
FastAPI / SpringBoot
```

Nginx 负责：

```text
/      → 前端
/api   → 后端
```

---

# 三、我对 Nginx 工作机制的理解

## 正向代理 vs 反向代理

### 正向代理

```text
客户端 → 代理 → 目标网站
```

（常见于科学上网工具）

---

### 反向代理（Nginx）

```text
客户端 → Nginx → 内部服务
```

核心区别是：

> 用户并不知道真实后端结构

---

# 四、Nginx 基本结构（我实际常用）

配置结构通常是：

```nginx
events {}

http {

    server {

        location / {

        }

    }

}
```

---

# 五、我最常写的几种配置场景

---

## 1. 前端静态站点

```nginx
server {
    listen 80;
    server_name localhost;

    root /www/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### 这里踩过的坑

Vue Router history 模式下：

```text
/user/1
```

如果没有：

```nginx
try_files ... /index.html
```

刷新页面一定 404。

---

## 2. FastAPI 反向代理

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://127.0.0.1:8000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 关键点

如果不加 header：

- 后端拿不到真实 IP
- 鉴权日志会失真

---

## 3. WebSocket 转发

这是我踩坑最严重的一块：

```nginx
location /ws {

    proxy_pass http://127.0.0.1:8000;

    proxy_http_version 1.1;

    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

不加这段基本必炸：

```text
WebSocket handshake failed
```

---

# 六、负载均衡配置

```nginx
upstream backend {
    server 127.0.0.1:8001;
    server 127.0.0.1:8002;
}
```

使用：

```nginx
location / {
    proxy_pass http://backend;
}
```

---

## 常见策略

- 轮询（默认）
- weight 权重
- ip_hash（会话一致性）

---

# 七、HTTPS 配置（实际用法）

```nginx
server {
    listen 443 ssl;

    ssl_certificate     /etc/ssl/fullchain.pem;
    ssl_certificate_key /etc/ssl/private.pem;
}
```

HTTP 强制跳转：

```nginx
server {
    listen 80;
    return 301 https://$host$request_uri;
}
```

---

# 八、我实际踩过的 Nginx 问题

---

## 1. 502 Bad Gateway

本质原因通常是：

```text
Nginx 连不上后端
```

常见排查：

- 后端是否启动
- 端口是否正确
- Docker 网络是否隔离

---

## 2. 413 请求体过大

上传失败时加：

```nginx
client_max_body_size 100m;
```

---

## 3. 跨域问题

后来我基本不再用 CORS 单独解决，而是：

```text
Nginx 统一 API 入口
```

直接变同源。

---

## 4. Docker 场景 localhost 问题

在容器里：

```text
localhost ≠ 宿主机
```

所以我统一改成：

```text
proxy_pass http://service-name
```

---

# 九、Docker + Nginx 实际组合

```yaml
services:
  nginx:
    image: nginx
    ports:
      - "80:80"

  backend:
    build: .
```

容器内访问必须用：

```text
service-name:port
```

---

# 十、Nginx 性能理解（实践层面）

我后来理解它高性能主要来自：

- epoll 事件模型
- 非阻塞 IO
- worker 多进程模型
- sendfile 零拷贝

核心不是“快”，而是：

> 不为每个连接创建线程

---

# 十一、我对 Nginx 的阶段性认知变化

最开始：

```text
只是用来放前端页面
```

后来：

```text
反向代理工具
```

再后来：

```text
整个系统的流量入口层
```

最后实际变成：

```text
后端系统架构的一部分
```

---

# 十二、总结

我现在对 Nginx 的定位是：

> 它不是一个工具，而是系统边界控制层。

在实际项目里，它解决的不是“部署问题”，而是：

- 流量如何进入系统
- 请求如何被分发
- 服务如何隔离
- 外部如何访问内部结构

当项目复杂度上来之后，这一层基本不可绕过。

```

```
