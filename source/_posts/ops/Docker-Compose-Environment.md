---
title: Docker 与 Compose：从能跑到可交付的工程化转变
date: 2023-08-02
tags:
  - Docker
  - DevOps
  - Linux
categories:
  - 运维
description: 从环境不可复现到标准化交付，记录 Docker / Compose 使用过程中的工程认知升级与真实踩坑经验。
---

# Docker 与 Compose：从能跑到可交付的工程化转变

经典问题始终没变：

```text
你本地能跑
我本地不能跑
服务器直接炸
```

排查一圈之后通常会发现：

- Python / Node 版本不一致
- 系统库差异（glibc / openssl）
- 依赖版本漂移
- 环境变量缺失
- 数据库版本不一致
- 甚至时区都不一致

最后沟通会变成一句：

```text
“要不你把环境打包给我？”
```

Docker 本质就是把这句话变成现实。

---

# 一、Docker 解决的不是“部署问题”，而是“环境不可控问题”

早期理解 Docker 很容易停留在：

> 打包代码 + 运行

但实际本质是：

```text
应用 + 依赖 + 运行时 + 系统行为 = 可重复执行单元
```

它解决的是：

> “环境不可复现导致的工程不确定性”

---

# 二、虚拟机 vs Docker：本质差异

## 虚拟机

```text
宿主机
 └── 虚拟硬件
      └── 完整操作系统
```

特点：

- 重
- 慢
- 隔离强但成本高

---

## Docker

```text
宿主机 Linux 内核
 └── Namespace + Cgroups 隔离的进程
```

特点：

- 共享内核
- 轻量
- 启动快
- 更偏“进程级隔离”

---

关键认知：

> Docker 不是虚拟机，是“受控进程”。

---

# 三、Docker 的核心抽象模型

| 概念       | 本质             |
| ---------- | ---------------- |
| image      | 只读分层文件系统 |
| container  | 运行中的进程实例 |
| volume     | 持久化数据映射   |
| network    | 容器间通信抽象   |
| dockerfile | 构建规则         |
| compose    | 多进程系统编排   |

---

# 四、镜像构建：真正重要的是“分层缓存”

很多人写 Dockerfile 是这样的：

```dockerfile
COPY . .
RUN pip install -r requirements.txt
```

问题：

```text
代码一变 → 依赖全部重装
```

---

## 正确结构（关键优化点）

```dockerfile
FROM python:3.11

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .
```

---

### 核心原因

Docker build 是分层缓存机制：

```text
每一层都是可缓存的 snapshot
```

只要前一层不变，后面都可以复用。

---

# 五、容器不是“运行环境”，而是“受控进程”

一个关键误区：

```bash
docker run ubuntu
```

然后发现：

```text
容器秒退
```

原因不是报错，而是：

> 容器生命周期 = 主进程生命周期

---

如果没有前台进程：

```text
容器直接退出
```

---

# 六、网络模型：localhost 的认知陷阱

这是最经典坑之一：

```python
redis://localhost:6379
```

在容器里意味着：

```text
容器自身
```

而不是宿主机。

---

## Compose 下正确方式

```python
redis://redis:6379
```

原因：

```text
Compose 自动创建 DNS 内部解析
服务名 = hostname
```

---

# 七、Docker Compose：从“单容器”到“系统建模”

如果说 Docker 是运行单元，那么 Compose 是：

> “系统结构描述语言”

---

## 示例结构

```yaml
services:
  app:
    build: .
    ports:
      - "8000:8000"

  redis:
    image: redis:7

  db:
    image: postgres:15
```

---

## 本质变化

从：

```text
docker run xxx
```

变成：

```text
声明式系统拓扑
```

---

# 八、Compose 的真正价值：服务间关系建模

现代系统不是单程序，而是：

```text
API + DB + Cache + MQ + Gateway
```

Compose 提供的是：

> 本地微服务编排能力

---

# 九、数据卷：容器“无状态”的关键解耦

如果数据库不挂载 volume：

```bash
docker rm mysql
```

数据直接消失。

---

## 正确方式

```bash
-v /data/mysql:/var/lib/mysql
```

核心思想：

```text
计算与数据解耦
```

---

# 十、构建缓存失效：隐性成本

常见问题：

```text
明明改了代码，镜像没变
```

原因：

```text
Docker layer cache 命中
```

---

## 强制构建

```bash
docker build --no-cache .
```

但更重要的是：

> 理解缓存链路，而不是依赖命令。

---

# 十一、容器日志模型：stdout 才是标准

错误方式：

```python
写文件日志
```

问题：

```text
docker logs 看不到
```

---

正确方式：

```text
标准输出 / stderr
```

Docker 才能统一收集。

---

# 十二、时区 / 环境变量：典型“隐性差异源”

```yaml
environment:
  TZ: Asia/Shanghai
```

或者：

```yaml
volumes:
  - /etc/localtime:/etc/localtime
```

---

这类问题本质不是 Docker bug，而是：

```text
环境默认值不一致
```

---

# 十三、权限问题：容器用户 ≠ 宿主用户

典型问题：

```text
容器创建的文件无法删除
```

原因：

```text
UID/GID 不一致
```

---

解决方式：

```yaml
user: "1000:1000"
```

---

# 十四、Docker 的工程意义：不是工具，而是交付标准

Docker 的价值逐渐从：

```text
开发工具
```

变成：

```text
软件交付标准单元
```

---

现代链路通常是：

```text
Git push
  ↓
CI build
  ↓
Docker image
  ↓
Registry
  ↓
Server pull
  ↓
Compose / K8s deploy
```

---

# 十五、真正的认知变化

从 Docker 开始，开发视角会发生变化：

## 1. 从“写代码”变成“交付系统”

---

## 2. 从“运行成功”变成“可重复运行”

---

## 3. 从“本地环境”变成“环境定义”

---

# 十六、总结

Docker 并不是解决“部署麻烦”的工具。

它解决的是：

> 环境不可复现导致的软件工程不确定性。

而 Compose 的意义也不只是“多容器启动”，而是：

> 把系统结构从口头描述变成机器可执行定义。

最终带来的变化是：

```text
开发者开始具备系统交付能力，而不仅仅是编码能力
```
