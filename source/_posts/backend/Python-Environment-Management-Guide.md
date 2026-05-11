---
title: Python 环境管理终于不再混乱
date: 2021-08-17
category:
  - 后端开发
  - Python
tag:
  - 虚拟环境
  - 环境管理
---

# Python 环境管理终于不再混乱

Python 生态有一个很典型的问题：

> 开发效率极高，但环境极容易混乱。

尤其：

- 新手
- AI 项目
- 多 Python 版本
- 多项目并行开发

几乎都会踩环境坑。

最经典的问题包括：

```text
明明已经 pip install 了却 import 不到
```

```text
项目昨天还能跑今天突然炸了
```

```text
不同项目依赖版本冲突
```

```text
CUDA、Torch、Pydantic 版本地狱
```

而 Python 环境管理，本质上就是解决：

- Python 解释器管理
- 包管理
- 依赖隔离
- 版本锁定
- 系统依赖管理

这些问题。

---

# Python 环境体系到底是什么关系

很多新人最容易混乱的地方就是：

```text
pip
venv
poetry
conda
```

到底谁管谁。

实际上：

---

# pip

负责：

```text
安装 Python 包
```

例如：

```bash
pip install requests
```

---

# venv

负责：

```text
创建独立 Python 虚拟环境
```

避免不同项目依赖互相污染。

---

# poetry

负责：

```text
现代化 Python 项目与依赖管理
```

包括：

- 依赖管理
- 虚拟环境
- 锁版本
- 项目配置

---

# conda

负责：

```text
更底层的环境管理
```

不仅管理 Python 包。

还管理：

- Python 本身
- CUDA
- C/C++ 依赖
- 科学计算库

AI 项目特别常见。

---

# 从 0 开始：Python 安装

先确认：

```bash
python --version
```

或者：

```bash
python3 --version
```

---

# Windows 常见问题

很多人安装 Python 后：

```text
python 不是内部命令
```

原因通常是：

```text
没有加入 PATH
```

安装 Python 时：

必须勾选：

```text
Add Python to PATH
```

---

# pip 是什么

pip 是 Python 官方包管理器。

作用类似：

| 语言    | 包管理器 |
| ------- | -------- |
| Python  | pip      |
| Node.js | npm      |
| Java    | Maven    |
| Rust    | cargo    |

---

# pip 常用命令

---

# 安装包

```bash
pip install requests
```

---

# 指定版本

```bash
pip install fastapi==0.110.0
```

---

# 升级包

```bash
pip install -U requests
```

---

# 卸载包

```bash
pip uninstall requests
```

---

# 查看已安装包

```bash
pip list
```

---

# 查看依赖详情

```bash
pip show requests
```

---

# 导出依赖

```bash
pip freeze > requirements.txt
```

---

# 安装 requirements

```bash
pip install -r requirements.txt
```

---

# pip 国内镜像源

国内直接连 PyPI 经常很慢。

常见做法：

配置镜像源。

---

# 临时使用镜像

```bash
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple requests
```

---

# 永久配置镜像

Windows：

```text
%APPDATA%\pip\pip.ini
```

内容：

```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

---

# Linux/macOS

```text
~/.pip/pip.conf
```

---

# pip 缓存问题

pip 默认有缓存。

很多时候：

```text
你以为重新下载了
实际上走的是缓存
```

---

# 查看缓存目录

```bash
pip cache dir
```

---

# 清空缓存

```bash
pip cache purge
```

---

# requirements.txt 到底是什么

本质：

```text
项目依赖清单
```

例如：

```text
fastapi==0.110.0
uvicorn==0.29.0
pydantic==2.6.1
```

作用：

```text
保证团队环境一致
```

---

# requirements 最大的问题

它只能：

```text
记录最终结果
```

不能很好管理：

- 依赖关系
- 项目配置
- 开发依赖

所以后来：

Poetry 开始流行。

---

# venv：官方虚拟环境

Python 默认全局环境非常危险。

因为：

```text
所有项目共享同一个 site-packages
```

容易：

- 版本冲突
- 项目污染
- 升级炸环境

---

# 创建 venv

```bash
python -m venv venv
```

这里：

```text
venv
```

是环境目录名。

---

# 激活环境

Windows：

```bash
venv\Scripts\activate
```

Linux/macOS：

```bash
source venv/bin/activate
```

---

# 激活后会发生什么

实际上：

```text
PATH 被临时修改
```

当前终端：

优先使用：

```text
venv 内部 Python
```

---

# 退出虚拟环境

```bash
deactivate
```

---

# 为什么必须使用虚拟环境

例如：

项目 A：

```text
django==3
```

项目 B：

```text
django==5
```

如果全局安装。

基本必炸。

---

# 一个标准 Python 项目流程

---

# 1. 创建项目

```bash
mkdir my_project
cd my_project
```

---

# 2. 创建虚拟环境

```bash
python -m venv venv
```

---

# 3. 激活环境

```bash
source venv/bin/activate
```

---

# 4. 安装依赖

```bash
pip install fastapi uvicorn
```

---

# 5. 导出依赖

```bash
pip freeze > requirements.txt
```

---

# 6. 提交 Git

通常：

```text
venv/
```

不会提交。

需要：

```gitignore
venv/
```

---

# Poetry：现代 Python 项目管理

Poetry 本质上是：

```text
pip + venv + dependency resolver + project manager
```

的整合。

---

# 为什么 Poetry 越来越流行

因为：

传统 pip 工作流：

太散了。

---

# Poetry 安装

官方推荐：

```bash
pip install poetry
```

或者：

```bash
curl -sSL https://install.python-poetry.org | python3 -
```

---

# 创建项目

```bash
poetry new my_project
```

---

# 初始化已有项目

```bash
poetry init
```

---

# 安装依赖

```bash
poetry add fastapi
```

---

# 安装开发依赖

```bash
poetry add pytest --group dev
```

---

# 删除依赖

```bash
poetry remove fastapi
```

---

# 安装全部依赖

```bash
poetry install
```

---

# 进入虚拟环境

```bash
poetry shell
```

---

# poetry.lock 是什么

这是 Poetry 最重要的东西之一。

作用：

```text
锁定所有依赖版本
```

包括：

- 一级依赖
- 二级依赖
- 子依赖

---

# 为什么锁版本极其重要

AI 项目里。

尤其：

- LangChain
- OpenAI SDK
- Pydantic

更新非常频繁。

很多时候：

```text
昨天教程今天失效
```

就是因为：

依赖版本变了。

---

# pyproject.toml

Poetry 使用：

```text
pyproject.toml
```

作为项目配置。

现在很多新项目：

即便不用 Poetry。

也会使用它。

---

# conda：AI 生态常客

很多人第一次接触 conda。

都是因为：

```text
PyTorch
TensorFlow
CUDA
```

---

# conda 和 pip 最大区别

pip：

```text
只管理 Python 包
```

---

# conda

还能管理：

- Python 解释器
- C 库
- CUDA
- 系统依赖

---

# 创建 conda 环境

```bash
conda create -n ai python=3.10
```

---

# 激活环境

```bash
conda activate ai
```

---

# 查看环境

```bash
conda env list
```

---

# 删除环境

```bash
conda remove -n ai --all
```

---

# conda 为什么适合 AI

因为很多 AI 包：

并不只是 Python 包。

例如：

```text
torch + cuda
```

本质涉及：

- GPU 驱动
- CUDA Toolkit
- cuDNN
- 编译环境

pip 很难处理。

---

# AI 项目最经典的环境地狱

例如：

```text
torch 版本不兼容 cuda
```

或者：

```text
pydantic v1/v2 冲突
```

尤其：

LangChain 生态。

曾经大量出现：

```text
依赖互相锁死
```

---

# 为什么 Docker 后来越来越重要

本质：

```text
冻结整个运行环境
```

包括：

- Python
- 系统库
- CUDA
- pip依赖

---

# 一个现代 Python 项目常见组合

现在很多项目：

已经逐渐变成：

```text
Docker
+ Poetry
+ pyproject.toml
+ lock file
```

---

# 我的实际建议

---

# 普通 Web 项目

推荐：

```text
venv + pip
```

足够。

---

# 中大型团队项目

推荐：

```text
Poetry
```

依赖管理体验明显更好。

---

# AI / 深度学习项目

推荐：

```text
conda
```

尤其涉及：

- CUDA
- Torch
- TensorFlow

时。

---

# 最后一个关键问题

很多人学 Python。

只关注：

```text
代码怎么写
```

但实际工程里。

大量时间：

都花在：

- 环境
- 依赖
- 版本
- 部署

这些问题上。

---

# Python 环境管理真正解决的

其实不是：

```text
怎么安装包
```

而是：

> 如何让项目长期稳定运行。

这才是工程里的核心问题。
