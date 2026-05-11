---
title: Linux、WSL 与 Windows 开发环境折腾记录
date: 2022-07-18
tags:
  - Linux
  - WSL
categories:
  - 运维
description: 从 Windows 原生开发，到 WSL、Ubuntu、SSH、screen 的完整开发环境折腾记录，以及踩过的各种环境坑与排查方法。
---

# Linux、WSL 与 Windows 开发环境折腾记录

现在我的开发环境基本是：

- Windows 做日常使用
- WSL 跑 Linux 开发环境
- SSH 连远程服务器
- Docker 跑服务
- VSCode Remote 做远程开发

于是一个很现实的问题一直存在：

> 为什么本地能跑，服务器不能跑？

随着这个问题反复出现，我开始逐步踩到一堆系统层面的差异：

- 操作系统差异
- shell 差异
- 文件系统差异
- 权限模型差异
- 环境变量差异
- 换行符差异
- 网络栈差异

这篇不讲 Linux 概念本身，只记录我是怎么从 Windows 开发，一路“被迫”折腾成半个 Linux 使用者的。

---

# 一、为什么最后还是绕不开 Linux

最开始我的开发环境是：

```text
Windows + IDE + cmd
```

但项目稍微复杂一点后，问题开始集中爆发：

| 场景          | 表现                      |
| ------------- | ------------------------- |
| Python 依赖   | 编译失败 / wheel 构建报错 |
| Node 原生模块 | 编译失败                  |
| Docker        | 性能和兼容问题            |
| shell 脚本    | 不兼容                    |
| nginx         | 配置别扭                  |
| ssh/scp       | 使用不顺手                |
| 服务器模拟    | 和本地环境不一致          |

典型情况是：

```bash
pip install xxx
```

然后突然报：

```text
Microsoft Visual C++ 14.0 required
```

这个阶段基本就会开始考虑：

> 要不要直接上 Linux 环境算了

---

# 二、WSL：Windows 和 Linux 的折中方案

## 1. WSL 是什么

WSL = Windows Subsystem for Linux

本质是：

> 在 Windows 内运行 Linux 用户态环境（WSL2 已接近完整 Linux 内核体验）

---

# 三、WSL 为什么最终变成主力

在 WSL 之前，我的环境是：

```text
Windows 上混装：
- Python
- Node
- Git Bash
- Docker Desktop
- 各种 CLI
```

结果是：

- PATH 冲突
- 工具链不一致
- 环境污染严重

切到 WSL 后变成：

```text
WSL Ubuntu：
- Python
- Node
- Git
- Docker
- SSH
- Linux 工具链
```

环境边界变清晰了。

---

# 四、WSL 安装与初始化

```powershell
wsl --install
```

查看发行版：

```powershell
wsl -l -o
```

安装 Ubuntu：

```powershell
wsl --install -d Ubuntu
```

查看状态：

```powershell
wsl -l -v
```

---

# 五、WSL 最大的坑：文件系统 IO

最早我习惯把项目放在：

```text
/mnt/c/project
```

结果是明显卡顿，尤其是：

- npm install
- pnpm install
- node_modules
- Python venv

原因很简单：

> WSL 访问 Windows 文件系统有额外 IO 转换成本

---

## 正确方式

项目放在 Linux 文件系统：

```text
/home/user/project
```

然后直接用 VSCode Remote 打开。

体验差距非常明显。

---

# 六、Shell 体系混乱问题

一开始我经常混用：

```text
cmd / powershell / git bash / wsl bash
```

后来才逐渐理清：

## CMD

基本废弃状态，能力有限。

## PowerShell

Windows 现代自动化 shell（对象模型，不是纯文本流）。

## Bash

Linux 标准 shell，服务器环境核心。

---

# 七、SSH：开始真正接触服务器

最典型入口：

```bash
ssh root@ip
```

---

## SSH 本质

SSH 是加密的远程 shell 通道：

```text
本地终端 → 加密 → 服务器 shell
```

---

## 常用操作

登录：

```bash
ssh root@ip
```

指定端口：

```bash
ssh -p 2222 root@ip
```

免密：

```bash
ssh-keygen
ssh-copy-id root@ip
```

---

## 常见坑

### 1. key 权限过宽

```text
Permissions 0777 are too open
```

修复：

```bash
chmod 600 ~/.ssh/id_rsa
```

---

### 2. known_hosts 冲突

```bash
ssh-keygen -R ip
```

---

# 八、systemd 与服务管理

开始接触 Linux 后，我逐渐不再是“启动程序”，而是“管理服务”。

---

## systemctl

启动：

```bash
systemctl start nginx
```

状态：

```bash
systemctl status nginx
```

开机自启：

```bash
systemctl enable nginx
```

---

# 九、screen：解决 SSH 断线问题

最早踩坑是：

```bash
python main.py
```

SSH 一断进程直接死。

---

## screen 使用

创建：

```bash
screen -S job
```

恢复：

```bash
screen -r job
```

后台退出：

```text
Ctrl + A + D
```

---

# 十、日志排查方式

实时日志：

```bash
tail -f app.log
```

过滤：

```bash
grep ERROR app.log
```

组合：

```bash
tail -f app.log | grep ERROR
```

---

# 十一、环境变量问题

典型现象：

```text
本地能跑，服务器不能跑
```

最终大概率是：

```text
PATH 不一致
```

检查：

```bash
echo $PATH
which python
which node
```

---

# 十二、换行符问题（经典坑）

Windows：

```text
CRLF
```

Linux：

```text
LF
```

报错：

```text
/bin/bash^M: bad interpreter
```

---

修复方式：

```bash
dos2unix script.sh
```

或 VSCode 切 LF。

---

# 十三、权限模型

Linux 权限问题是一个持续踩坑点。

```bash
chmod +x run.sh
chown user:user file
```

很多“无法写 / 无法删 / 无法执行”本质都是权限问题。

---

# 十四、端口与进程排查

查端口：

```bash
lsof -i:8000
```

或：

```bash
netstat -tunlp
```

杀进程：

```bash
kill -9 pid
```

---

# 十五、开发环境分层意识

后面我逐渐开始强制区分环境：

| 环境    | 用途     |
| ------- | -------- |
| local   | 本地开发 |
| dev     | 开发环境 |
| test    | 测试环境 |
| staging | 预发布   |
| prod    | 生产     |

避免直接把本地逻辑带到生产环境。

---

# 十六、最终稳定的开发结构

最后我的结构固定为：

```text
Windows
 ├─ 浏览器 / IM
 ├─ VSCode
 └─ WSL Ubuntu
      ├─ Python
      ├─ Node
      ├─ Docker
      ├─ Git
      ├─ Redis
      ├─ PostgreSQL
      └─ 项目运行环境
```

远程：

```text
SSH → Linux Server
```

部署：

```text
Docker Compose
```

---

# 十七、总结性认知变化

整个过程下来，认知变化大致是：

## 1. 从“会用工具”

变成理解：

- 文件系统
- 权限系统
- 进程模型
- 网络模型

---

## 2. 从“跑代码”

变成理解：

- runtime
- PATH
- dependency
- shell 执行机制

---

## 3. 从“部署应用”

变成理解：

- 服务化
- 守护进程
- 容器化
- 自动化运行机制

---

很多以前觉得是“玄学”的问题，最后都会收敛成：

```text
路径 / 权限 / 环境变量 / 系统差异
```

而在完整经历 WSL + SSH + Linux + Docker 之后，程序“怎么跑起来的”这件事会变得非常具体。
