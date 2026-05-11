---
title: Python 异步协程、线程与进程的理解整理
date: 2021-12-15
category:
  - 后端开发
  - Python
tag:
  - asyncio
  - 协程
  - 多线程
  - 多进程
---

# Python 异步协程、线程与进程的理解整理

Python 学习过程中。

有一个阶段非常容易混乱：

```text
异步
线程
进程
协程
并发
并行
```

尤其：

FastAPI、爬虫、AI Agent、WebSocket、消息队列。

几乎都会接触这些东西。

但很多人其实：

> “代码会写，但不知道为什么这样写”。

我自己也是后来真正做：

- 高并发 HTTP 请求
- Bot 框架
- AI Workflow
- 长连接服务
- 批量任务系统

之后。

才逐渐真正理解：

Python 的并发体系。

---

# 一切问题的起点：程序为什么会卡住

先看一个最简单的问题。

---

# 同步代码

```python
import time

def task():
    time.sleep(3)
    print("done")

task()
task()
```

运行时间：

```text
6秒
```

因为：

```text
前一个任务没结束
后一个任务不能开始
```

这就是：

```text
同步阻塞
```

---

# 什么是阻塞

例如：

- 网络请求
- 数据库
- 文件 IO
- sleep

这些操作期间。

CPU 很多时候其实：

```text
什么都没干
```

只是：

```text
等待
```

---

# 并发的本质

后来我最大的认知变化之一：

> 并发的核心不是“同时运行”。

而是：

> “不要让等待浪费时间”。

---

# 最经典的场景：HTTP 请求

例如：

```python
requests.get(url)
```

期间：

程序需要等待：

- DNS
- TCP
- 网络传输
- 服务端响应

这时候 CPU 其实大部分时间：

```text
空闲
```

---

# 所以最早出现的是：多线程

---

# Threading

Python 标准库：

```python
import threading
```

---

# 一个最简单的线程例子

```python
import threading
import time

def task(name):
    time.sleep(3)
    print(name)

t1 = threading.Thread(target=task, args=("A",))
t2 = threading.Thread(target=task, args=("B",))

t1.start()
t2.start()

t1.join()
t2.join()
```

运行时间：

```text
3秒左右
```

---

# 为什么线程会变快

因为：

当一个线程：

```text
等待 IO
```

时。

另一个线程可以继续执行。

---

# 线程适合什么

线程特别适合：

```text
IO密集型
```

例如：

- 网络请求
- Web 服务
- 爬虫
- 文件操作

---

# 但后来很快就遇到一个词

```text
GIL
```

---

# 什么是 GIL

Global Interpreter Lock。

中文：

```text
全局解释器锁
```

---

# GIL 最大影响

同一个 Python 进程里：

```text
同一时刻只能有一个线程执行 Python 字节码
```

---

# 这句话非常重要

很多人第一次看到：

会觉得：

```text
那线程不是废了？
```

实际上：

不是。

---

# GIL 不影响 IO 并发

因为：

线程等待 IO 时。

会主动释放 GIL。

于是：

其他线程可以继续运行。

---

# 所以：

---

# IO密集型

推荐：

```text
线程
异步协程
```

---

# CPU密集型

例如：

- 图像处理
- AI推理
- 大量计算
- 视频编码

推荐：

```text
多进程
```

---

# 为什么 CPU 密集线程不行

例如：

```python
def calc():
    while True:
        pass
```

两个线程跑。

CPU 并不会真正同时计算。

因为：

GIL 会轮流抢执行权。

---

# 所以 Python 真正并行要靠进程

---

# multiprocessing

```python
from multiprocessing import Process
```

---

# 最简单例子

```python
from multiprocessing import Process
import time

def task():
    time.sleep(3)
    print("done")

p1 = Process(target=task)
p2 = Process(target=task)

p1.start()
p2.start()

p1.join()
p2.join()
```

---

# 为什么进程能真正并行

因为：

每个进程：

都有独立：

- Python解释器
- 内存空间
- GIL

---

# 进程缺点也很明显

因为：

进程太重。

创建成本高。

内存占用高。

---

# 所以：

---

# 线程

更轻量。

共享内存。

但有 GIL。

---

# 进程

真正并行。

但资源更贵。

---

# 线程为什么危险

因为：

共享内存。

---

# 一个经典线程安全问题

```python
count += 1
```

看起来简单。

实际上：

不是原子操作。

---

# 多线程可能出现

```text
数据竞争
```

最终：

结果错误。

---

# 所以需要锁

---

# Lock

```python
lock = threading.Lock()
```

---

# 加锁

```python
with lock:
    count += 1
```

---

# 但锁又带来新问题

例如：

- 死锁
- 性能下降
- 阻塞

---

# 后来 Python 又开始流行 asyncio

这才是真正改变 Python Web 生态的东西。

---

# asyncio 是什么

本质：

```text
单线程协作式并发
```

---

# 它最大的特点

不是：

```text
多个线程
```

而是：

```text
一个线程
大量任务
```

---

# async / await

```python
async def task():
    await asyncio.sleep(1)
```

---

# await 到底是什么

后来我真正理解后。

发现：

它本质就是：

```text
主动让出执行权
```

---

# 也就是说

协程不会：

```text
强行抢占
```

而是：

```text
自己告诉调度器：
我现在在等
你先跑别人
```

---

# 这是协程和线程最大的区别

---

# 线程

操作系统调度。

---

# 协程

程序自己调度。

---

# asyncio 最经典场景

例如：

```python
async def fetch():
    await httpx.get(...)
```

等待网络期间。

事件循环会：

```text
自动切换其他任务
```

---

# 什么是事件循环

Event Loop。

asyncio 的核心。

---

# 事件循环本质

可以理解成：

```text
任务调度中心
```

它不断：

- 检查任务状态
- 切换协程
- 恢复执行

---

# asyncio 为什么性能高

因为：

它避免了：

- 线程切换
- 锁竞争
- 大量线程开销

---

# FastAPI 为什么快

本质上：

就是：

```text
asyncio + uvloop + ASGI
```

---

# uvicorn 是什么

很多人：

只知道：

```bash
uvicorn main:app
```

---

# 实际上

它本质是：

```text
ASGI服务器
```

负责：

- 网络通信
- 事件循环
- 协程调度

---

# ASGI 为什么重要

因为：

传统 WSGI：

不支持真正异步。

而：

```text
FastAPI
Starlette
WebSocket
```

都依赖：

```text
ASGI
```

---

# asyncio 最大的误区

很多人以为：

```text
async = 并行
```

实际上：

不是。

---

# asyncio 本质还是单线程

所以：

CPU密集型任务：

一样会卡死事件循环。

---

# 一个经典错误

```python
async def task():
    while True:
        pass
```

整个事件循环直接卡死。

---

# asyncio 适合什么

特别适合：

```text
高IO并发
```

例如：

- Web 服务
- 聊天系统
- Bot
- WebSocket
- AI API 调用
- HTTP 请求风暴

---

# 我后来做 AI Agent 时

对 asyncio 理解更深了。

因为：

Agent 本质大量：

- HTTP请求
- Tool调用
- IO等待

---

# 例如：

```text
调用LLM
查数据库
搜索网页
读文件
```

全部都是 IO。

asyncio 非常适合。

---

# asyncio 常见核心 API

---

# 创建任务

```python
task = asyncio.create_task(func())
```

---

# 等待多个任务

```python
await asyncio.gather(
    task1(),
    task2()
)
```

---

# 运行事件循环

```python
asyncio.run(main())
```

---

# 超时控制

```python
await asyncio.wait_for(task(), timeout=5)
```

---

# asyncio 为什么后期容易复杂

因为：

异步系统：

最大的敌人是：

```text
状态
```

---

# 最经典问题

例如：

- 忘记 await
- 任务泄漏
- 死循环
- cancel失败
- 并发竞争

---

# 尤其：

```text
Task exception was never retrieved
```

很多人都见过。

---

# Python 并发体系最终怎么选

这是最关键的问题。

---

# CPU密集型

推荐：

```text
multiprocessing
```

---

# IO密集型

推荐：

```text
threading
asyncio
```

---

# 高并发网络服务

推荐：

```text
asyncio
```

---

# AI 推理

推荐：

```text
多进程
GPU
任务队列
```

---

# 实际工程里经常是混合模型

例如：

```text
asyncio处理网络
线程池处理阻塞IO
进程池处理CPU计算
```

---

# asyncio 和线程并不是敌人

很多人喜欢：

```text
同步党
异步党
```

互喷。

实际上：

工程里：

经常一起用。

---

# asyncio 调线程池

```python
loop.run_in_executor()
```

---

# FastAPI 本身也会：

```text
自动线程池化同步接口
```

---

# Python 并发真正难的地方

其实不是 API。

而是：

```text
理解任务调度
```

---

# 后来我最大的认知变化

以前觉得：

```text
并发 = 多任务同时跑
```

后来发现：

真正核心是：

> 如何在“等待”期间充分利用资源。

---

# 最后总结

---

# threading

特点：

```text
轻量
适合IO
共享内存
有GIL
```

---

# multiprocessing

特点：

```text
真正并行
适合CPU
资源开销大
```

---

# asyncio

特点：

```text
单线程高并发
适合大量IO
性能高
复杂度也高
```

---

# 而 Python 整个并发体系

本质上都在解决同一个问题：

> 如何让程序在等待时，不浪费机器资源。

这才是：

异步、线程、进程。

背后真正统一的核心逻辑。
