---
title: 从 Docker 到 K8S：我对容器化的理解
date: 2025-02-20
tags:
  - Docker
  - Kubernetes
  - K8S
  - DevOps
  - Linux
categories:
  - 运维
description: 从 Docker 单机容器，到 Kubernetes 集群编排，整理容器化、K8S 核心概念、生态体系、部署方式与常见工程实践。
---

# 修改说明（精简）

这篇文章主要做了三类调整：

1. **结构问题修正**：原内容偏“概念堆叠”，调整为“问题 → 演进 → K8S 解决方式”的工程路径。
2. **认知层强化**：补齐 Docker → K8S 的核心断层（从运行到编排，从单机到声明式系统）。
3. **表达一致性修正**：统一为第一人称视角，避免第三方总结式口吻。

---

# 从 Docker 到 K8S：我对容器化的理解

很多人第一次接触 Docker 时都会觉得：

```text
“终于不用配环境了”
```

但真正做项目之后很快会发现问题变了：

- 一个容器挂了怎么办？
- 多机器如何部署？
- 如何自动扩容？
- 如何做负载均衡？
- 如何滚动更新？
- 如何服务发现？
- 如何保证高可用？

问题从：

```text
“怎么运行”
```

变成：

```text
“怎么管理大量运行中的容器”
```

Kubernetes（K8S）就是为这个阶段设计的。

---

# 一、Docker 解决的问题

Docker 本质解决的是：

## 环境一致性问题

在 Docker 之前，环境问题是常态：

```text
开发环境 / 测试环境 / 生产环境
```

经常出现：

- 依赖版本不一致
- 系统库不一致
- Python / Node 版本不一致
- OpenSSL / libc 差异

Docker 的核心变化是：

```text
image = 可复现运行环境
```

---

# 二、Docker 为什么不够

Docker 在单机层面很好用，但本质仍然是：

```text
单机容器运行工具
```

当系统规模扩大后，会出现一系列问题：

---

## 1. 容器故障恢复问题

容器挂了需要手动处理：

```bash
docker ps
docker restart
```

无法自动化。

---

## 2. 多机部署问题

当服务扩展到：

```text
Node A / Node B / Node C
```

会出现：

- 服务分布不一致
- 网络复杂化
- 配置分散

---

## 3. 扩容问题

流量上升时：

```text
单容器无法承载
```

需要手动扩容，非常不现实。

---

## 4. 发布问题

传统方式：

```text
停止 → 更新 → 启动
```

会导致服务中断。

---

# 三、K8S 是什么

Kubernetes 本质是：

> 容器编排系统

它解决的是：

```text
“如何持续管理大量容器”
```

---

# 四、K8S 的核心模型

K8S 管理的不是容器本身，而是系统状态：

| 对象       | 作用           |
| ---------- | -------------- |
| Pod        | 最小调度单元   |
| Deployment | 应用部署控制   |
| Service    | 服务发现与访问 |
| ConfigMap  | 配置管理       |
| Secret     | 敏感信息       |
| Volume     | 数据持久化     |
| HPA        | 自动扩缩容     |

---

# 五、K8S 的核心思想

Docker 是：

```text
运行容器
```

K8S 是：

```text
维持期望状态
```

例如：

```text
我希望：
- 3个副本
- 自动重启
- CPU 超过80%自动扩容
```

K8S 会持续修正系统，使其保持一致。

---

# 六、Pod：K8S 的最小单位

很多人容易误解：

```text
Pod = 容器
```

但实际是：

```text
Pod = 容器运行组
```

一个 Pod 内可以包含多个容器，例如：

- 主服务
- 日志 sidecar
- 代理容器

---

# 七、Deployment：应用控制器

Deployment 负责：

- 副本数控制
- 滚动更新
- 自动恢复
- 版本管理

示例：

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

---

# 八、Service：稳定访问入口

Pod IP 是变化的，因此需要 Service：

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80

  type: ClusterIP
```

本质作用：

```text
提供稳定访问入口 + 服务发现
```

---

# 九、ConfigMap / Secret：配置解耦

传统方式：

```python
DB_HOST=localhost
```

问题是：

```text
配置和代码强耦合
```

K8S 做了拆分：

- ConfigMap：普通配置
- Secret：敏感信息

---

# 十、Volume：持久化问题

容器默认是无状态的：

```text
容器删除 = 数据丢失
```

通过 Volume 解决：

```yaml
volumes:
  - /data:/var/lib/data
```

---

# 十一、Ingress：统一入口

当服务变多后：

```text
多个 Service 暴露端口
```

会失控。

Ingress 提供：

- 路由
- 域名
- HTTPS

例如：

```text
api.xxx.com → backend
web.xxx.com → frontend
```

---

# 十二、K8S 的核心能力：自愈系统

K8S 持续对比：

```text
期望状态 vs 实际状态
```

一旦不一致：

```text
自动修复
```

例如：

- Pod 挂掉 → 自动重建
- 节点失效 → 重新调度

---

# 十三、滚动更新（Rolling Update）

传统部署：

```text
停机更新
```

K8S：

```text
逐步替换 Pod
```

实现无感更新。

---

# 十四、HPA：自动扩缩容

根据负载动态调整：

```text
CPU > 80% → 扩容
CPU < 阈值 → 缩容
```

---

# 十五、为什么 K8S 会复杂

因为它解决的是：

```text
分布式系统的全部问题
```

包括：

- 调度
- 网络
- 存储
- 服务发现
- 高可用
- 自动恢复

---

# 十六、生态体系

K8S 本身只是核心，周边生态非常大：

| 工具       | 作用     |
| ---------- | -------- |
| Helm       | 包管理   |
| Prometheus | 监控     |
| Grafana    | 可视化   |
| Istio      | 服务网格 |
| ArgoCD     | GitOps   |
| Harbor     | 镜像仓库 |

---

# 十七、Helm：工程化封装

没有 Helm：

```text
大量 YAML 手写
```

有 Helm：

```bash
helm install redis
```

本质是：

```text
K8S 应用包管理系统
```

---

# 十八、Docker vs K8S 本质差异

Docker：

```text
解决“怎么跑”
```

K8S：

```text
解决“怎么长期稳定运行”
```

---

# 十九、使用边界

## 小规模项目

```text
Docker Compose 足够
```

## 中大型系统

```text
多服务 + 多节点 + 高可用 → K8S
```

---

# 二十、核心认知变化

从 Docker 到 K8S，我的理解发生了几个变化：

## 1. 从运行到治理

不再是“启动服务”，而是“维护系统状态”。

---

## 2. 从单点到系统

应用不再是单程序，而是系统集合。

---

## 3. 从手动到声明式

不再描述步骤，而是描述目标。

---

# 二十一、总结

Docker 解决的是：

```text
环境标准化
```

K8S 解决的是：

```text
运行系统的长期稳定性与规模化管理
```

当这一套体系跑通之后，开发视角会逐渐变成：

```text
代码 + 容器 + 编排 + 自动化 = 系统工程
```

而 K8S 的意义其实很直接：

> 不再关心“怎么跑起来”，而是关心“如何一直稳定跑下去”
