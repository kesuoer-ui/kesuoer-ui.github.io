---
title: 基于 NoneBot2 的 QQ 机器人开发实践
date: 2021-08-18
category:
  - 项目实践
tag:
  - Python
  - Bot
  - PostgreSQL
  - FastAPI
---

# 基于 NoneBot2 的 QQ 机器人开发实践

这个项目从 2021 年开始搭建，并在后续一段时间持续迭代。

这是我第一次长期维护一个“持续在线”的 Python 服务型项目。

和一般脚本不同，Bot 项目本质上已经不是工具程序，而是一个事件驱动的常驻服务。

---

# 一、项目特点

QQ Bot 相比普通程序，有几个非常明显的特征：

- 长期运行（7×24）
- 高频事件输入（消息流）
- 多群并发环境
- 功能持续增长
- 插件化需求强
- 状态与数据长期持久化

随着功能增加，项目逐渐从“脚本集合”演变为：

> 一个事件驱动的消息处理系统

---

# 二、技术栈选择

核心技术组合如下：

```text
NoneBot2
    +
OneBot
    +
go-cqhttp
    +
PostgreSQL
    +
asyncio
```

---

# 三、为什么选择 NoneBot2

当时在多个方案中做过对比：

- Mirai
- Koishi
- NoneBot
- Yunzai

最终选择 NoneBot2 的原因主要是工程适配性：

---

## 1. Python 生态优势

QQ Bot 的很多功能天然依赖：

- HTTP 请求
- 数据处理
- 图像处理
- AI 接口
- 文件操作

Python 在这些领域具备直接优势。

---

## 2. asyncio 的适配性

Bot 本质是 IO 密集型系统：

- 网络请求
- 消息收发
- 数据库操作
- API 调用

asyncio 对这类模型天然契合。

---

# 四、Bot 架构模型

很多人最初理解 Bot 会认为是：

```text
循环接收消息 → if判断 → 执行逻辑
```

实际结构更接近事件链路：

```text
QQ客户端
    ↓
go-cqhttp
    ↓
OneBot 协议
    ↓
NoneBot2
    ↓
插件系统
```

---

# 五、OneBot 的作用

OneBot 本质是一个统一协议层。

它定义了消息结构，例如：

```json
{
  "message_type": "group",
  "user_id": 123456,
  "group_id": 111111,
  "message": "hello"
}
```

作用类似于：

> 将不同 QQ Bot 实现统一为标准消息协议

---

# 六、go-cqhttp 的角色

go-cqhttp 在当时是核心基础设施之一，它的作用是：

- 实现 QQ 客户端协议
- 提供 WebSocket / HTTP 接口
- 转换为 OneBot 标准事件

从架构上看：

> Bot 框架依赖它完成“消息入口层”

---

# 七、插件化设计

随着功能增长，Bot 必然需要模块化拆分。

典型插件结构：

```text
plugins
├── chat
├── weather
├── music
├── memes
├── admin
├── image
└── tools
```

---

# 插件机制本质

插件系统本质是：

> 消息分发系统

流程如下：

```text
接收消息
    ↓
解析命令
    ↓
匹配插件
    ↓
执行 handler
```

---

# 一个基础插件示例

```python
from nonebot import on_command

ping = on_command("ping")

@ping.handle()
async def _(bot, event):
    await ping.send("pong")
```

---

# 八、系统设计关键点

随着插件增加，核心问题逐渐从“功能实现”转为：

> 如何控制系统复杂度

主要涉及：

- 插件管理
- 权限体系
- 配置注入
- 数据隔离
- 生命周期管理

---

# 九、系统膨胀问题

Bot 项目有一个典型特征：

> 功能增长极快

导致常见问题：

- 插件数量膨胀
- 逻辑重复
- 目录混乱
- 功能耦合

---

# 十、统一能力封装

后期逐步抽象出统一能力层：

- 权限控制
- 冷却机制
- 群配置
- 日志记录
- 数据访问封装

例如权限控制：

```python
if not await check_admin(event):
    return
```

后续进一步封装为：

```python
@admin_required
```

---

# 十一、异步模型的重要性

Bot 系统必须避免阻塞。

典型错误：

```python
time.sleep(5)
```

问题是：

> 会阻塞整个事件循环

正确方式：

```python
await asyncio.sleep(5)
```

---

# 十二、数据库设计演进

随着功能扩展，逐渐引入 PostgreSQL。

主要数据结构包括：

- 用户信息
- 群配置
- 插件状态
- 签到系统
- 消息记录
- 缓存数据

---

# 十三、图片与多媒体处理

后期功能扩展到：

- 表情包生成
- OCR识别
- 图片处理
- 动图生成

依赖逐渐扩展到：

- Pillow
- OpenCV
- FFmpeg

复杂度明显上升。

---

# 十四、依赖冲突问题

Python Bot 生态常见问题：

```text
不同插件依赖版本不一致
```

例如：

- pydantic v1 vs v2 冲突
- 依赖链污染

---

解决方式逐步转向：

```bash
poetry install
poetry lock
```

进行依赖统一管理。

---

# 十五、定时任务系统

后期引入定时任务：

- 每日签到
- 群消息推送
- 自动清理缓存

常用方案：

```python
scheduler.add_job(
    task,
    "cron",
    hour=4
)
```

---

# 十六、日志系统

随着系统复杂化，日志成为基础设施。

常用工具：

- loguru

示例：

```python
logger.info("message received")
logger.error(e)
```

---

# 十七、AI 功能接入

AI 功能并非最初设计，而是后期逐步引入。

典型问题包括：

- Token 成本控制
- 上下文长度管理
- Prompt 压缩
- 历史缓存策略

---

# 十八、系统核心问题总结

Bot 系统核心难点并不是功能实现，而是：

- 长期稳定运行
- 插件增长控制
- 状态一致性
- IO 并发处理
- 依赖管理

---

# 十九、关键认知变化

该项目带来的核心变化是：

从“写脚本”转向理解：

> 事件驱动服务系统的工程结构

---

# 最终总结

NoneBot2 项目本质不是一个 QQ 工具，而是一个：

> 长期运行的事件驱动消息服务系统

它涉及：

- 插件架构
- 异步编程
- 数据库设计
- 权限控制
- 系统稳定性

更重要的是，它是第一次让我真正理解：

> 一个项目从“能用”到“可长期维护”的差距在哪里
