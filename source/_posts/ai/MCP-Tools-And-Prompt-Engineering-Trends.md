---
title: MCP、Tools 与 Prompt Engineering 的工程化趋势
date: 2025-11-20
category:
  - AI
tag:
  - MCP
  - AI Agent
  - Prompt Engineering
  - Tool Calling
---

# MCP、Tools 与 Prompt Engineering 的工程化趋势

AI 发展到后面。

我越来越感觉：

很多人其实低估了：

> “工程化” 这件事的重要性。

因为早期 AI 圈子。

太容易沉迷于：

```text id="6l3mzp"
模型又变强了
参数又变大了
Benchmark又提升了
```

但真正开始做 Agent 后。

会越来越意识到：

> 模型能力只是下限。

真正决定系统能不能落地的。

是：

- Workflow
- Tool
- Context
- Memory
- 状态管理
- Prompt Engineering
- 系统稳定性

这些工程问题。

---

# 为什么 AI Agent 后期越来越像“软件工程”

因为：

大模型本质上已经开始：

> “接管软件交互层”。

---

# 以前的软件模式

传统软件：

```text id="m0r9v2"
按钮
菜单
输入框
接口
```

用户必须：

> 学会软件规则。

---

# 现在开始反过来了

AI 模式：

```text id="k5v1tx"
自然语言
-> AI理解
-> Tool执行
```

变成：

> 软件开始理解人。

---

# 这其实是巨大的范式变化

因为：

过去几十年。

人类一直在：

```text id="u3y7fq"
适应机器
```

而 AI 时代开始：

```text id="d7s0qa"
机器适应人
```

---

# 为什么 Prompt Engineering 会爆火

因为：

Prompt 是：

> 人类语言和模型行为之间的桥梁。

---

# 但我后来越来越觉得

“Prompt Engineering” 这个词。

其实已经有点被玩坏了。

---

# 早期很多 Prompt Engineering

本质是：

```text id="j4m9cn"
角色扮演
话术技巧
```

例如：

- 你是顶级专家
- 深呼吸
- 一步一步思考

这些。

---

# 这些有没有用

有。

但：

> 不够工程化。

---

# 真正工业级 Prompt

后来越来越像：

> “自然语言 DSL”。

---

# 什么是 DSL

Domain Specific Language。

领域专用语言。

---

# 我后来越来越觉得

Prompt 本质其实是在：

> “编排模型行为”。

---

# 例如一个成熟 Agent Prompt

其实会包含：

- 身份
- 权限
- Tool规则
- 输出格式
- 安全约束
- Workflow规则
- 错误处理

甚至：

- 禁止行为
- 回退逻辑
- 风险控制

---

# 这时候 Prompt 已经不像“聊天”

而更像：

> “系统控制协议”。

---

# 为什么 Prompt 后期越来越长

因为：

模型并不真正理解：

```text id="q8m5rl"
企业规则
```

于是：

必须：

不断显式约束。

---

# 我后来最大的认知变化

是：

> Prompt 本质上是在“压制模型自由度”。

---

# 为什么自由 Agent 很危险

因为：

LLM 天生：

- 发散
- 联想
- 幻觉
- 自我补全

---

# 但企业系统需要的是

```text id="r2n8jb"
稳定
可预测
可审计
```

于是：

Prompt 工程越来越像：

> AI 行为治理。

---

# 为什么 Tool Calling 是真正的转折点

因为：

LLM 本身其实：

> “没有行动能力”。

---

# 大模型能理解

但：

不能：

- 查数据库
- 发请求
- 改代码
- 操作文件
- 调业务系统

---

# Tool 的本质

后来我越来越觉得。

Tool Calling 本质是：

```text id="n6w1ka"
让LLM调用现实世界能力
```

---

# 这是 AI 真正开始“接地气”的阶段

因为：

它第一次：

不再只是：

```text id="v4q2ms"
聊天
```

而开始：

```text id="y9u5eg"
执行
```

---

# 为什么 Tool 比 Prompt 更重要

因为：

Prompt 再强。

也只能：

> “说”。

Tool 才能：

> “做”。

---

# Agent 为什么会越来越依赖 Tool

因为现实世界：

本质是：

> API 世界。

例如：

- 数据库
- HTTP
- Git
- Office
- ERP
- CRM
- 搜索引擎

全部都是：

Tool。

---

# 我后来对 Agent 的理解

越来越像：

```text id="b2r0pd"
LLM = CPU
Tool = 外设
```

---

# 没有 Tool 的 Agent

其实很像：

```text id="t5k1zm"
断网的大脑
```

---

# 为什么 MCP 会越来越重要

因为：

Tool 一多。

整个生态会迅速失控。

---

# AI 行业正在经历“接口爆炸”

以前：

每个 Agent：

自己定义 Tool。

结果：

- 格式不同
- 权限不同
- 参数不同
- 通信方式不同

最后：

生态无法互通。

---

# MCP 本质在解决什么

后来我越来越觉得。

MCP 真正解决的是：

> “AI 世界的标准化接口问题”。

---

# 很像什么

很像：

- USB
- HTTP
- RPC
- Docker API

这些标准协议。

---

# 为什么标准协议如此重要

因为：

一旦统一。

生态会开始：

> 爆炸式增长。

---

# 例如 HTTP 出现后

互联网：

开始真正：

```text id="g8j6wr"
互联
```

---

# MCP 也在做类似的事情

它试图统一：

```text id="p7s3vn"
AI如何访问工具
```

---

# 为什么这会很关键

因为未来：

Agent 不可能：

只服务一个系统。

---

# 企业真实场景一定是

```text id="f0m2uk"
多个模型
多个系统
多个工具
多个工作流
```

---

# 如果没有统一协议

后期会变成：

```text id="e3c1xr"
AI接口地狱
```

---

# 我后来越来越觉得

MCP 的意义。

可能远大于很多人现在理解的程度。

---

# 它真正影响的是

> AI Agent 生态的“基础设施层”。

---

# 为什么 LangChain 后期争议越来越大

因为：

很多人开始发现。

它虽然：

> “统一了调用”。

但：

没有真正统一：

```text id="m1x4tb"
Agent工程规范
```

---

# 于是行业开始进入下一阶段

从：

```text id="u8p6oq"
能跑起来
```

变成：

```text id="s4n7fw"
怎么稳定落地
```

---

# 这时候工程化开始压倒模型炫技

很多企业后来真正关心的是：

- 稳定率
- Token成本
- 响应时间
- 权限控制
- 安全性
- Workflow治理

而不是：

```text id="w0q8me"
模型会不会吟诗
```

---

# 为什么 AI Agent 后期越来越像“操作系统”

因为：

它开始承担：

- 调度
- 状态管理
- 权限管理
- Tool管理
- Workflow管理

这些传统 OS 的职责。

---

# 我后来甚至觉得

未来 Agent Framework。

会越来越像：

```text id="a6t3lg"
AI Runtime
```

---

# Prompt、Tool、Workflow 的关系

后来我逐渐把它们理解成：

---

# Prompt

负责：

```text id="c1v9na"
行为约束
```

---

# Tool

负责：

```text id="h4m8pr"
现实执行
```

---

# Workflow

负责：

```text id="n9y2ka"
流程控制
```

---

# 三者缺一不可

只有 Prompt：

容易变聊天机器人。

只有 Tool：

只是 API 集合。

只有 Workflow：

又缺乏智能性。

---

# 为什么传统开发者容易误判 AI

因为：

很多人会本能觉得：

```text id="r5u1mc"
AI不过是另一个接口
```

但实际上：

它正在改变：

> 软件交互逻辑本身。

---

# 传统软件是

```text id="x7q4vb"
确定性系统
```

---

# AI 系统是

```text id="y2m8tk"
概率型系统
```

---

# 于是整个工程思想都会变化

例如：

传统系统：

```text id="d0v3rz"
输入固定
输出固定
```

AI 系统：

```text id="l6n5oq"
输入相同
输出可能不同
```

---

# 所以后期重点开始变成

不是：

> “怎么让 AI 更聪明”。

而是：

> “怎么让 AI 更稳定”。

---

# 这也是为什么

现在越来越多企业开始关注：

- Agent Workflow
- Context Engineering
- Prompt治理
- Tool标准化
- AI安全

这些方向。

---

# 我后来最大的感受

AI 行业现在特别像：

```text id="j1f7mp"
互联网早期
```

---

# 大家都知道方向对了

但：

- 标准还没统一
- 基础设施还不成熟
- 工程规范还在形成
- Agent形态还在演化

---

# 但趋势已经很明显

未来的软件。

很可能会从：

```text id="q3z0xt"
GUI驱动
```

逐渐变成：

```text id="k8m6wu"
自然语言驱动
```

---

# 而 Prompt、Tool、MCP、Workflow

这些东西。

本质上都在构建：

> 下一代 AI 软件基础设施。

---

# 现在回头看

我越来越觉得。

真正重要的。

可能已经不是：

```text id="b7v2kp"
某个模型又提升了多少分
```

而是：

> 整个 AI 工程生态，正在逐渐形成自己的“软件工业体系”。

而这件事。

可能才是 AI 真正开始改变世界的起点。
