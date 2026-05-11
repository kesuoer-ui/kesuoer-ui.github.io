---
title: Hexo + Butterfly + GitHub Pages 实战搭建记录（附完整踩坑）
date: 2022-01-01
categories:
  - 项目实践
tags:
  - Hexo
  - GitHub Pages
  - Butterfly
  - 博客
description: 从 0 搭建 Hexo 博客到正式上线，中间踩过的 SSH、GitHub 多账号、Node 版本、GitHub Pages、GitHub Actions、Pages 等各种坑。
---

# 前言

最近开始搭建个人技术博客。

一开始预期很简单：

> “静态博客 + GitHub Pages + 写 Markdown”

实际操作之后才发现问题集中在三块：

- Node / npm 环境兼容
- Git / SSH 多账号冲突
- GitHub Pages + Actions 发布链路

单独 Hexo 本身反而是最稳定的一环。

---

# 为什么选 Hexo + Butterfly

选型主要在几个方案之间：

- WordPress（重、依赖服务器）
- Hugo（生态偏 Go，主题习惯不匹配）
- VuePress（偏文档系统）
- Hexo（成熟 + 资料多 + 上手快）

最终选：

- Hexo
- Butterfly
- GitHub Pages

原因很直接：

## Hexo

特点：

- Markdown 驱动
- 静态站点
- GitHub Pages 直接托管
- 插件生态成熟

对于个人博客属于“够用且稳定”。

---

## Butterfly

选择 Butterfly 主要原因：

- 主题成熟
- 配置项丰富
- 文档多
- UI 现成可用

属于 Hexo 中文生态里最常见的一档主题。

---

# 环境准备

## Node.js

当时环境是 Node 20 系列。

Hexo 对 Node 版本要求比较严格，初始化时曾出现类似：

```bash
engine "node" is incompatible
```

原因是：

- Hexo 对 Node 有最低版本要求
- 不同 Hexo 版本要求不同 Node 小版本

处理方式：

- 升级 Node 到当前 LTS 最新稳定版本
- 使用官方推荐版本区间

---

## 安装 Hexo

```bash
npm install -g hexo-cli
```

初始化项目：

```bash
hexo init blog
cd blog
npm install
```

---

# npm / Windows 权限问题

在 Windows 环境下遇到典型问题：

```bash
EPERM: operation not permitted
```

常见原因：

- npm cache 权限问题
- 杀毒软件占用文件
- 全局安装路径权限不足

处理方式：

```bash
npm cache clean --force
```

并使用管理员权限终端重新执行安装。

---

# 安装 Butterfly

主题安装：

```bash
npm install hexo-theme-butterfly
```

渲染器：

```bash
npm install hexo-renderer-pug hexo-renderer-stylus --save
```

---

# 修改主题配置

```yaml
theme: butterfly
```

---

# GitHub 多账号 SSH 问题

当时本地存在多个 GitHub 账号，导致：

- push 走错账号
- commit identity 混乱
- SSH key 自动匹配错误

---

## 生成新的 SSH Key

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

指定不同文件：

```text
~/.ssh/id_ed25519_blog
```

---

## SSH config 配置

```config
Host github-blog
HostName github.com
User git
IdentityFile ~/.ssh/id_ed25519_blog
```

---

## 测试连接

```bash
ssh -T git@github-blog
```

成功会返回认证信息。

---

## Git identity（关键区别）

SSH ≠ Git 提交身份。

需要单独设置：

```bash
git config user.name "xxx"
git config user.email "xxx"
```

注意不使用 `--global`，避免污染其他项目。

---

# Git 初始化与忽略文件

初始化：

```bash
git init
```

`.gitignore`：

```text
node_modules/
public/
.deploy_git/
db.json
```

---

# GitHub Pages 部署链路

当时踩坑点主要在两种部署模式：

## 1. gh-pages 分支模式

## 2. GitHub Actions 部署模式

实际使用的是 Actions + gh-pages 混合方式。

---

# GitHub Actions 配置

```yaml
name: Deploy Hexo

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm install

      - run: npx hexo generate

      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

---

# 常见问题：Action 权限

报错类型：

```text
Permission denied to github-actions[bot]
```

原因：

- 默认 Actions 没有写权限

解决：

```text
Settings → Actions → General → Workflow permissions
```

改为：

```text
Read and write permissions
```

---

# Pages 404 问题

现象：

- Action 成功
- 网站仍然 404

原因：

- Pages 未指向正确分支

解决：

```text
Settings → Pages
```

设置：

- Branch: gh-pages
- Folder: / (root)

---

# 部署链路理解

实际运行流程是：

```text
Hexo generate
    ↓
GitHub Actions
    ↓
gh-pages branch
    ↓
GitHub Pages 发布
```

---

# Butterfly 默认样式问题

初始效果偏模板化，后续主要调整：

- Banner
- 配色系统
- 卡片样式
- 顶栏透明度
- 字体与间距

---

# 简单 UI 风格调整（示例）

```css
.card-widget,
#recent-posts > .recent-post-item {
  background: rgba(255, 255, 255, 0.55);
  backdrop-filter: blur(18px);
  border-radius: 24px;
}
```

---

# 总结

整个搭建过程的核心问题不在 Hexo：

主要复杂度来自：

- Node 版本兼容
- npm / Windows 权限
- Git 多账号 SSH 管理
- GitHub Actions 权限模型
- Pages 发布配置

Hexo 本身只是“静态生成器”，真正复杂的是：

> 从本地到 GitHub Pages 的整条工程链路。

---

# 后续扩展方向

后面可以继续完善：

- 自动化部署（CI优化）
- 图片 CDN
- 评论系统
- SEO 优化
- 主题深度定制
