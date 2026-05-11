---
title: 从 Git 到 GitHub：我终于理解了现代协作开发
date: 2020-10-04
category:
  - 运维
tag:
  - Git
  - GitHub
  - 版本控制
---

## 背景

最开始接触 GitHub，其实和“会不会 Git”没什么关系。

只是某天在网上随手点开一个开源项目，然后被里面的结构震了一下：

- 几万 Star
- 几百贡献者
- 几千 Commit
- 完整 README 和文档
- CI 自动化
- Issue / PR 在持续运转

那种感觉更像一个“活着的软件系统”，而不是单纯代码仓库。

反过来看自己当时的项目：

```text
project/
project_final/
project_final_v2/
project_final_v3_new/
真正最终版/
```

属于典型“文件夹堆叠式开发”。

---

## 当时的误区

那时候一个很典型的认知错误是：

> Git 和 GitHub 是一回事

后来才慢慢拆开：

### Git 是什么

Git 本质是一个本地版本控制工具，核心能力是：

- 记录代码历史
- 支持分支开发
- 支持回滚与对比
- 管理多人修改冲突

和是否联网没有直接关系。

---

### GitHub 是什么

GitHub 本质是建立在 Git 之上的远程协作平台：

- 托管代码仓库
- 支持团队协作
- PR（代码合并流程）
- Issue（问题追踪）
- CI/CD 集成
- 开源社区生态

简单理解：

> Git 负责“怎么管理代码”，GitHub 负责“怎么一起写代码”。

---

## 没有 Git 之前的开发状态

在接触 Git 之前，代码管理基本是“原始模式”。

### 1. 靠复制文件夹版本控制

```text
project/
project_backup/
project_backup_new/
project_backup_final/
project_backup_final2/
```

最后结果就是：

> 自己也不确定哪个版本能运行。

---

### 2. 不敢随便改代码

典型心态：

- 改坏了就回不去
- 不确定哪里动过
- 不敢重构

所以只能：

- 不停备份
- 不停复制

---

### 3. 多人协作基本不可控

两个人同时改：

```python
login()
```

结果：

- 互相覆盖
- 文件冲突
- 人工拼代码

项目稍微复杂一点就会失控。

---

## Git 的核心思想

Git 的核心其实很简单：

> 用“提交记录”保存代码的历史状态

---

## Commit：不是保存，而是快照

当时对 commit 的理解是“保存代码”。

后来才更准确理解为：

> 一次完整的项目状态快照

例如：

```bash
git commit -m "修复登录接口异常"
```

含义是：

> 当前整个项目状态被记录下来，可以随时回退或对比。

---

### 为什么 commit 很重要

后来看项目历史，其实主要依赖 commit：

- 看改了什么
- 为什么改
- 哪一步引入问题

所以 commit message 质量非常关键。

比如：

好的：

```bash
修复 websocket 断线重连问题
```

一般的：

```bash
update
```

---

## Branch：分支开发

第一次真正理解分支时，其实是有冲击感的。

原本思维是：

> 一个项目只有一条代码线

Git 引入了另一种结构：

```text
main
├── feature-login
├── feature-upload
└── fix-bug
```

---

### 分支带来的变化

核心价值是：

> 新功能开发不会影响主线

即使某个分支写崩了：

- main 仍然是稳定版本
- 可以随时切换回去

---

## Merge：合并代码

开发完成后：

```bash
git merge feature-login
```

本质是：

> 把一条分支的修改合并回主线

这是团队开发的基础动作。

---

## Merge Conflict：第一次协作的现实问题

第一次遇到冲突时，状态基本是：

```text
CONFLICT
```

后来才明白原因很简单：

> 多个人改了同一段代码

Git 无法自动判断保留谁的版本。

需要人工处理冲突。

---

## Git 带来的真正变化

后来回头看，Git 最重要的价值不是“版本管理”。

而是：

> 降低试错成本

以前：

- 不敢改
- 不敢重构

之后：

- 可以开分支随便试
- 不行直接丢掉

---

## Git 基础命令（当时学习路径）

### 初始化

```bash
git init
```

### 状态查看

```bash
git status
```

### 添加变更

```bash
git add .
```

### 提交

```bash
git commit -m "init"
```

### 历史记录

```bash
git log
```

### 分支

```bash
git branch feature-login
git switch feature-login
```

### 合并

```bash
git merge feature-login
```

---

## GitHub：从工具到社区

后来真正深入 GitHub 后，感知发生变化。

它不只是代码托管平台，而更像：

> 程序员的公开协作世界

---

## GitHub 项目结构认知

一个成熟项目通常包含：

- README
- Issues
- Pull Requests
- Actions
- Releases
- Wiki

第一次完整看懂这些结构时，才开始理解“工程化开发”。

---

## Clone：进入开源世界的入口

```bash
git clone <repo>
```

典型流程：

- clone 项目
- 跑起来
- 看结构
- 改一点点

---

## Fork：复制仓库

当没有权限修改时：

- fork 一份到自己账号
- 修改
- 提 PR

本质是：

> 在自己的空间完成修改，再请求合并回主仓库

---

## Pull Request：协作核心流程

典型流程：

```text
fork -> 修改 -> push -> PR -> review -> merge
```

这是现代开源协作的标准路径。

---

## Issue：问题与需求系统

Issue 本质是：

> 项目的任务与问题管理系统

可以用于：

- bug 上报
- 功能需求
- 讨论设计

---

### 比较规范的 Issue 写法

通常包含：

- 环境信息
- 复现步骤
- 预期行为
- 实际结果
- 日志或截图

---

## GitHub Actions：自动化

第一次接触 CI/CD 时比较有冲击感。

例如：

```yaml
name: build

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - run: npm install
      - run: npm run build
```

触发逻辑是：

> push 代码 → 自动执行构建

---

## GitHub Pages：静态站部署

后来发现可以直接：

- 托管博客
- 部署文档
- 展示 Demo

不需要服务器。

---

## GitHub 的本质价值

GitHub 的意义其实是：

> 把全球开发者的代码公开连接起来

可以直接看到：

- 架构设计
- 工程实践
- 开源实现

很多技术成长路径，本质都是：

> 在 GitHub 上看别人怎么写代码

---

## GitLab 与 Gitee 的补充理解

### GitLab

更偏企业内部：

- 权限控制更强
- 私有化部署
- CI/CD 更完整

常见组合：

```text
GitLab + CI/CD
```

---

### Gitee

国内环境常用：

- 访问速度快
- 企业使用较多
- 开源生态较弱于 GitHub

---

## 总结

后来才逐渐理解：

Git 并不只是工具。

它带来的变化是：

- 协作方式变化
- 开发流程变化
- 项目结构变化
- 试错方式变化

从那之后，写代码的方式开始从：

> 单人文件式开发

变成：

> 分支驱动的工程化开发
