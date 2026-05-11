---
title: Stable Diffusion 工作流与模型体系整理
date: 2023-02-26
category:
  - AI
  - AIGC
tag:
  - StableDiffusion
  - SDXL
  - LoRA
  - AIGC
  - ComfyUI
---

# Stable Diffusion 工作流与模型体系整理

AI 绘画刚火的时候，其实很多人第一反应都是：

> “这玩意是不是以后画师要失业了？”

但真正自己上手后才发现：

这东西更像：

```text
新的生产工具
```

而且门槛其实不低。

尤其是：

- 模型
- 参数
- 插件
- 显存
- 工作流
- 提示词
- ControlNet
- LoRA

这一套东西刚接触时真的非常混乱。

---

# 我最开始接触 SD 的方式

最早其实是从 B 站开始。

那时候最出名的应该就是：

- 秋葉aaaki 整合包
- 各种 NovelAI 二创模型
- 二次元炼丹教程

当时很多人：

```text
连 Python 都不会
```

但已经能开始跑 AI 绘画了。

某种意义上：

> SD 能火，很大原因就是社区整合做得太好了。

尤其秋叶整合包。

基本已经做到：

```text
下载
→ 解压
→ 双击启动
```

对普通用户非常友好。

---

# Stable Diffusion 到底是什么

第一次接触时我也很懵。

各种概念满天飞：

- checkpoint
- vae
- embedding
- hypernetwork
- lora
- controlnet

后来慢慢理解后发现：

其实核心逻辑没那么复杂。

---

# SD 本质上在干什么

简单理解：

```text
随机噪声
↓
不断去噪
↓
最后生成图片
```

例如：

```text
一团雪花
↓
逐渐变成二次元老婆
```

这就是 diffusion 的核心思想。

---

# 最核心的几个组件

## checkpoint / 大模型

也就是：

```text
xx.safetensors
```

这是主体模型。

决定：

- 画风
- 角色风格
- 构图倾向
- 色彩习惯

例如：

- Anything
- Counterfeit
- AbyssOrangeMix
- MeinaMix

那时候 C 站几乎天天都在出新模型。

---

# C站是什么

AI 圈里说的 C 站：

一般就是：

## Civitai

大量：

- 模型
- LoRA
- Embedding
- 工作流

都在上面分享。

某种意义上：

它已经算 AI 绘画时代的 NexusMods 了。

---

# HuggingFace 又是什么

相比 C 站偏“社区模型分享”。

HuggingFace 更偏：

```text
AI 开源生态平台
```

上面会有：

- 官方模型
- 论文模型
- Transformers
- Diffusers
- Dataset

很多真正底层的东西其实都在 HF。

---

# VAE 是什么

这玩意一开始我也完全没懂。

后来发现：

## 它更像“色彩修正器”。

有些模型：

```text
不加载 VAE
```

会：

- 发灰
- 发白
- 颜色奇怪

加载后会正常很多。

特别是以前很多 NAI 系模型。

---

# LoRA 才是真正改变生态的东西

LoRA 出现后：

AI 绘画才真正进入：

```text
全民炼丹时代
```

以前训练模型：

- 时间长
- 显存大
- 成本高

LoRA 出现后：

几张图就能训练。

于是开始出现：

- 角色 LoRA
- 服装 LoRA
- 动作 LoRA
- 画风 LoRA

后来甚至：

```text
一个模型挂十几个 LoRA
```

已经成常态了。

---

# 我当时最震惊的一件事

就是：

## “模型套模型”

例如：

```text
底模
+ VAE
+ LoRA
+ Embedding
+ ControlNet
```

最后生成出来的图：

已经完全不是单一模型的效果了。

这和传统软件时代差异非常大。

---

# WebUI 时代

最开始大家基本都在用：

## A1111 WebUI

启动：

```bash
webui-user.bat
```

然后浏览器打开：

```text
127.0.0.1:7860
```

那个界面估计很多人都很熟。

---

# 当时最常踩的坑

## 爆显存

尤其：

```text
AMD 显卡用户
```

体验非常痛苦。

当时：

- CUDA 基本 NVIDIA 专属
- ROCm 支持又一般
- Windows 下兼容更差

于是经常：

```text
torch not compiled with cuda
```

或者：

```text
CUDA out of memory
```

看到想吐。

---

# 我甚至用过 Google Colab 白嫖显卡

这应该很多老 SD 用户都经历过。

因为那时候：

```text
本地显卡根本跑不动
```

于是：

- Colab
- Kaggle
- 云 GPU

开始疯狂流行。

尤其：

```text
免费 T4
```

真的救过命。

---

# 那时候典型工作流

基本长这样：

```text
本地写 Prompt
↓
上传云端
↓
生成图片
↓
下载结果
```

有时候甚至：

```text
Google 一断连
白跑半小时
```

非常真实。

---

# Prompt 也是个大坑

刚开始很多人以为：

> “随便输一句话就行。”

后来发现：

AI 绘画其实非常吃 Prompt。

例如：

```text
masterpiece, best quality, ultra detailed
```

这种“咒语流” Prompt。

一度特别流行。

---

# Negative Prompt 也很重要

尤其二次元模型。

不写负面词：

```text
多手
崩脸
畸形
```

几乎是家常便饭。

于是后来：

```text
EasyNegative
badhand
bad anatomy
```

这些 embedding 基本人手必备。

---

# SDXL 出来后变化很大

SD1.5 时代：

比较偏：

```text
二次元炼丹
```

而 SDXL 后：

- 构图
- 光影
- 真实感

提升明显。

但代价也很明显：

## 更吃显存。

很多：

```text
6G 显卡用户
```

直接阵亡。

---

# ComfyUI 是另一种世界

第一次看 ComfyUI：

我是真懵了。

满屏：

```text
节点
连线
工作流
```

像：

```text
UE蓝图
+
NodeRed
+
AI绘画
```

后来用久了才发现：

## 它本质是“图形化推理流水线”。

例如：

```text
模型加载
↓
Prompt编码
↓
LoRA注入
↓
采样
↓
VAE解码
↓
输出图片
```

全部节点化。

---

# 为什么很多高级玩家后来转 ComfyUI

因为：

## 可控性太强了。

WebUI 更像：

```text
给普通用户用
```

而 ComfyUI：

```text
更像工程化工具
```

特别适合：

- 批量生成
- API化
- 工作流复用
- 自动化处理
- 多模型组合

---

# AI 绘画最容易被低估的一点

很多人觉得：

> “不就是输入 Prompt 出图？”

实际上：

后面已经逐渐变成：

```text
工作流工程
```

了。

尤其：

- ControlNet
- OpenPose
- 局部重绘
- 区域重绘
- 多阶段放大
- 批处理

这些东西组合起来后。

复杂度已经不低了。

---

# 我后来对 AI 绘画的理解

我现在反而觉得：

它不像：

```text
替代创作
```

更像：

```text
提高素材生产效率
```

尤其：

- 二创
- UI素材
- 背景图
- 头像
- 网站配图
- 概念草图

效率确实非常夸张。

---

# 但它也确实很吃硬件

尤其：

- 显卡
- 显存
- SSD
- 内存

模型一多后：

```text
几十 GB 很正常
```

而且：

```text
不同模型还互相抢显存
```

后来甚至：

```text
整理模型
```

本身都快变成一种运维工作了。

---

# 现在再回头看

Stable Diffusion 最厉害的地方其实不是：

> “AI 会画画了”

而是：

## 它把 AI 内容生产真正带进了普通人的电脑。

以前：

AI 基本是：

```text
论文
实验室
大公司
```

而 SD 开始后。

第一次变成：

```text
普通人也能自己部署
自己训练
自己折腾
```

这件事其实影响非常大。

---
