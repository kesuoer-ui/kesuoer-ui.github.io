---
title: LangChain、LangGraph 与 AI Agent 工程实践
date: 2025-09-16
category:
  - AI
tag:
  - LangChain
  - LangGraph
  - AI Agent
  - LLM
  - Harness
---

# LangChain、LangGraph 与 AI Agent 工程实践

这篇不是“框架介绍”，更偏向一次工程实现过程的整理。

核心问题其实很简单：

> 当 LLM 从“聊天工具”变成“执行单元”之后，系统怎么设计才能不崩？

---

# 1. Agent 真正开始变复杂的临界点

最早的模型使用方式非常直接：

```text id="h0q1a1"
LLM → 输出
```

````

后来变成：

```text id="h0q1a2"
LLM → Tool → 输出
```

再往后：

```text id="h0q1a3"
LLM → Tool → 多轮 → 状态 → 再执行
```

问题开始出现：

- 多轮后状态不可控
- Tool 调用顺序漂移
- 上下文污染
- 输出不可复现
- 错误无法恢复

本质问题不是模型，而是：

> 没有执行控制系统（Execution Control Plane）。

---

# 2. 为什么 LangChain 最早能火起来

LangChain 本质做了一件事：

> 把“LLM调用链”变成工程结构。

它提供的不是能力，而是：

- Prompt 组织方式
- Tool 抽象
- Memory 模块
- Chain 执行结构
- RAG 流程封装

但早期问题也很明显：

- 抽象过多
- API 变化频繁
- 状态不可控
- 调试困难

工程上有一个典型问题：

> 你很难知道 Agent “为什么做错了”。

---

# 3. Agent 系统真正的分界线：有没有 Harness

后来逐渐意识到一个关键概念：

> Agent 是否工程化，本质看有没有 Harness（执行框架）。

这里的 Harness 可以理解为：

```text id="h0q1a4"
LLM不是系统核心
而是被控制执行的“计算单元”
```

一个完整 Harness 通常包含：

- Execution Loop（执行循环）
- State Manager（状态管理）
- Tool Router（工具调度）
- Policy Layer（执行约束）
- Memory Layer（状态持久化）
- Observability（可观测性）

没有 Harness 的 Agent：

> 本质就是“随机调用 LLM 的脚本”。

---

# 4. 从 Chat Agent 到 Workflow Agent

早期模式：

```text id="h0q1a5"
User → LLM → Answer
```

Agent 模式：

```text id="h0q1a6"
User
→ Planner
→ Tool Execution
→ Memory Update
→ Response
```

Workflow 引入之后，系统变化是：

- 不再追求自由生成
- 转向流程约束执行
- 每一步都有状态输入输出

---

# 5. LangGraph 的本质：从链式执行到状态机

LangChain 的核心问题：

> Chain 是线性的，复杂任务无法表达分支状态。

LangGraph 引入的是：

> 有状态的执行图（Stateful Graph）

本质结构：

```text id="h0q1a7"
Node → Node → Node
  ↘       ↙
     State
```

关键变化：

- 支持分支
- 支持回环
- 支持条件跳转
- 支持状态共享

---

# 6. 为什么 Graph 比 Chain 更接近工程系统

现实 Agent 任务通常不是线性的：

例如：

- 识别意图
- 判断是否需要工具
- 调用多个工具
- 汇总结果
- 失败重试
- fallback路径

Chain 的问题：

```text id="h0q1a8"
一条路走到底
```

Graph 的优势：

```text id="h0q1a9"
允许系统“走错路再修正”
```

---

# 7. Tool Calling 的工程问题本质

Tool Calling 表面是：

```text id="h0q1b1"
LLM调用函数
```

但工程问题是：

- 参数合法性
- schema一致性
- tool失败恢复
- 并发调用冲突
- tool结果可信度

典型失败场景：

- JSON格式错误
- 参数幻觉
- 无限循环调用
- tool结果污染上下文

---

# 8. 为什么 Tool 需要 Policy Layer（控制层）

如果没有 policy：

```text id="h0q1b2"
LLM → 随便调工具
```

有 policy：

```text id="h0q1b3"
LLM → Tool Router → 权限判断 → 执行
```

Policy 做的事情：

- 限制工具使用范围
- 控制调用频率
- 校验参数合法性
- 防止危险操作
- 控制成本（token / api）

---

# 9. Memory 的本质：不是“记忆”，而是状态压缩

常见误区：

```text id="h0q1b4"
Memory = 聊天记录
```

实际工程中：

Memory 是：

> 状态压缩 + 检索系统

通常包括：

- Summary Memory（摘要）
- Vector Memory（语义检索）
- Structured State（结构化状态）

核心目标：

> 控制 token 增长，而不是无限堆历史。

---

# 10. Context Engineering（比 Prompt 更重要）

系统运行久之后真正的问题是：

- context越来越大
- 噪声越来越多
- 决策越来越不稳定

因此核心变成：

> 如何组织上下文，而不是如何写 prompt

包括：

- 信息裁剪
- 状态摘要
- 任务相关性过滤
- 历史分层

---

# 11. MCP 的意义：Tool 标准化协议

当 Tool 数量上升之后，会遇到问题：

- tool接口不统一
- 调用方式混乱
- 权限体系缺失

MCP 的作用本质是：

> 统一 Agent 与工具系统的通信协议

类似：

- HTTP 解决 Web 通信
- MCP 解决 AI Tool 通信

---

# 12. FastAPI + Agent 的工程结构演化

早期结构：

```text id="h0q1b5"
FastAPI → LLM
```

后期结构：

```text id="h0q1b6"
FastAPI
→ Agent Harness
   → Workflow Engine
      → Tool Layer
      → Memory Layer
      → LLM Layer
```

---

# 13. 最大的工程认知变化：Agent不是模型问题

总结下来变化是：

| 阶段 | 认知         |
| ---- | ------------ |
| 早期 | LLM能力问题  |
| 中期 | Prompt问题   |
| 后期 | 系统工程问题 |

最终结论：

> Agent 的上限不是模型，而是系统设计。

---

# 14. 一个更现实的理解

当前 Agent 系统更像：

- 一个执行引擎
- 一个状态机
- 一个带工具权限的运行时

而不是：

- 一个聊天机器人

---

# 总结

LangChain / LangGraph / MCP / Tool Calling 这些组件单独看是框架。

但组合起来后本质是：

> 一个“自然语言驱动的执行系统雏形”。

而 Harness（执行控制层）才是决定系统能否进入工程阶段的关键。
````
