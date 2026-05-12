---
title: 面试问题回答复盘速查
date: 2026-05-09
category:
  - 技术
tag:
  - 面试
  - 八股
---

# Redis

---

## 1. Redis 五种常用数据结构及场景（必背）

### 涉及知识点

Redis 常用数据结构：

| 类型   | 特点                    | 常见场景               |
| ------ | ----------------------- | ---------------------- |
| String | 最常用，支持数字/字符串 | 缓存、计数器、分布式锁 |
| Hash   | 类似对象                | 用户信息、商品信息     |
| List   | 有序可重复              | 消息队列、评论列表     |
| Set    | 无序去重                | 点赞、标签、共同好友   |
| ZSet   | 带分数自动排序          | 排行榜、延迟队列       |

---

### 实际项目场景

#### String

项目里最常用。

例如：

- 登录 token 缓存
- 短信验证码
- 接口限流计数

比如：

```text id="e6r9eb"
token:用户id -> jwt信息
```

---

#### Hash

缓存用户资料：

```text id="r4q8ui"
user:1001
```

字段：

- nickname
- avatar
- level

修改头像时只更新一个字段，不用整个 JSON 重存。

---

#### List

异步日志队列。

请求先写 Redis：

```text id="dbfqk3"
LPUSH queue
```

消费者后台批量写数据库。

作用：

- 削峰
- 减少数据库压力

---

#### Set

文章点赞去重。

```text id="mylmd3"
article:1:likes
```

存 userId。

特点：

- 自动去重
- 判断是否点赞速度快

---

#### ZSet

积分排行榜。

score 存积分。

可以快速：

- 排序
- 查前十
- 查排名

---

### 白话回答（面试可直接说）

> “Redis 常用的数据结构有 String、Hash、List、Set 和 ZSet。
>
> String 一般做缓存、计数器和 token；
> Hash 适合存用户对象；
> List 常做异步队列；
> Set 适合点赞去重；
> ZSet 常做排行榜。
>
> 我项目里主要用 Redis 做热点缓存、登录 token、排行榜和异步削峰。”

---

## 2. 缓存穿透、击穿、雪崩（必背）

### 缓存穿透

#### 什么是穿透

请求的数据：

- Redis 没有
- MySQL 也没有

请求直接打数据库。

---

#### 场景

恶意请求：

```text id="1vfd6y"
userId=-1
```

数据库根本不存在。

大量请求会把数据库打崩。

---

#### 解决方案

##### 布隆过滤器

提前判断数据是否存在。

不存在直接拦截。

---

##### 缓存空值

数据库查不到：
也缓存 null。

避免反复查数据库。

---

### 缓存击穿

#### 什么是击穿

某个热点 key 突然过期。

大量请求同时访问数据库。

---

#### 场景

热门商品缓存失效。

10 万请求同时查 MySQL。

---

#### 解决方案

##### 互斥锁

只有一个线程查数据库。

其他线程等待。

---

##### 逻辑过期

热点数据不过期。

后台异步更新缓存。

---

### 缓存雪崩

#### 什么是雪崩

大量缓存同一时间失效。

数据库瞬间压力暴增。

---

#### 解决方案

##### 过期时间随机化

不要统一：

```text id="5t9vdi"
3600秒
```

而是：

```text id="s1l8ns"
3600 + random
```

---

##### Redis 高可用

比如：

- Redis Sentinel
- Redis Cluster

避免 Redis 整体挂掉。

---

##### 限流降级

限制请求量。

保护数据库。

---

### 顺带解释几个词

#### 热点 Key

访问量特别高的数据。

例如：

- 秒杀商品
- 热门文章

---

#### 限流

限制单位时间请求数量。

防止系统被打爆。

---

#### 降级

系统压力太大时：
关闭部分非核心功能。

---

### 白话回答（高频标准答案）

> “缓存穿透是查不存在的数据，缓存和数据库都没有；一般用布隆过滤器或者缓存空值解决。
>
> 缓存击穿是热点 key 失效导致大量请求同时打数据库；一般用互斥锁或者逻辑过期。
>
> 缓存雪崩是大量缓存同时过期；一般通过随机过期时间、Redis 高可用和限流降级解决。”

---

## 3. Redis 持久化：RDB / AOF

---

### 涉及知识点

| 方式 | 原理       | 优点           | 缺点       |
| ---- | ---------- | -------------- | ---------- |
| RDB  | 定时快照   | 恢复快、文件小 | 可能丢数据 |
| AOF  | 记录写命令 | 数据更安全     | 文件更大   |

---

### AOF 补充

默认：

```text id="4yby5i"
everysec
```

每秒刷盘一次。

最多丢 1 秒数据。

生产环境通常：

```text id="0m9f8l"
RDB + AOF
```

一起用。

---

### 白话回答

> “Redis 持久化主要有 RDB 和 AOF。
>
> RDB 是定时生成数据快照，恢复速度快但可能丢数据；
> AOF 是记录写命令，数据更安全。
>
> 生产环境一般两者一起使用，AOF 通常配置 everysec，最多丢 1 秒数据。”

---

## 4. 淘汰策略、LRU 原理

---

### 涉及知识点

Redis 内存满了后：
会根据淘汰策略删除数据。

---

### 常见策略

| 策略         | 含义                             |
| ------------ | -------------------------------- |
| noeviction   | 不淘汰，直接报错                 |
| allkeys-lru  | 淘汰最近最少使用的数据（最常用） |
| volatile-lru | 只淘汰设置过期时间的数据         |

---

### 什么是 LRU

Least Recently Used。

最近最少使用。

认为：

- 很久没访问的数据
- 未来大概率也不会访问

优先淘汰。

---

### 实际项目场景

用户缓存系统：

- 热门用户经常访问
- 冷数据自动淘汰

一般会配置：

```text id="5qj7tt"
allkeys-lru
```

---

### 白话回答

> “Redis 内存满了会根据淘汰策略删除数据。生产环境最常用的是 allkeys-lru，也就是优先淘汰最近最少使用的数据。”

---

## 5. 如何保证消息一致性

这个问题面试里通常在问：

### 缓存和数据库一致性

---

### 涉及知识点

最常见方案：

```text id="2wq6eq"
先更新数据库
再删除缓存
```

---

### 为什么不是先删缓存

如果：

- 删缓存后
- 数据库还没更新
- 其他线程读到旧数据

旧数据会重新写回缓存。

导致脏数据。

---

### 什么是脏数据

Redis 和数据库数据不一致。

例如：

- MySQL 是新值
- Redis 还是旧值

---

### 高并发优化

### 延迟双删

```text id="pncbcs"
更新DB
删缓存
sleep
再删一次
```

避免并发情况下旧数据回写。

---

### MQ 重试

删除 Redis 失败：
发送 MQ 重试。

---

### 什么是 MQ

消息队列。

例如：

- Kafka
- RabbitMQ
- RocketMQ

用于：

- 异步
- 解耦
- 削峰

---

### 实际项目场景

订单状态更新：

```text id="wnf7ca"
更新MySQL
删除Redis缓存
```

如果 Redis 删除失败：

- 发送 RocketMQ 重试

保证最终一致性。

---

### 白话回答

> “缓存一致性一般采用先更新数据库，再删除缓存。
>
> 高并发下会配合延迟双删或者 MQ 重试机制，保证 Redis 和 MySQL 最终一致。”

---

## 6. 项目中哪里实际用了 Redis

这个问题重点：
不要只说“做缓存”。

一定要说：

- 用在哪
- 为什么用
- 效果是什么

---

### 实际项目回答模板

> “项目里 Redis 主要用于：
>
> 1. 登录 token 缓存
> 2. 热点数据缓存
> 3. 接口限流
> 4. 异步队列削峰
> 5. 排行榜
>
> 例如商品详情页，我们把热点商品缓存到 Redis，减少 MySQL 查询压力。
>
> 登录鉴权则把 token 存 Redis，支持单点登录和踢下线。
>
> 此外日志系统用了 Redis List 做异步队列，避免高并发时频繁写数据库。”

# MySQL（几乎每场都问）

## 1. InnoDB 索引结构

### 知识点

- InnoDB 默认存储引擎
- 索引类型：
  - **B+ 树索引**（InnoDB 默认）
  - **B 树索引**（概念上类似，但 B+ 树存储方式不同）

- B+ 树 vs B 树区别：
  - 所有叶子节点形成链表，便于范围查询
  - 非叶子节点只存 key，不存数据（减少层数，增加效率）
  - B 树非叶子节点就存数据

### 白话回答示例

> “InnoDB 默认使用 B+ 树作为索引结构。B+ 树和普通 B 树的区别在于：B+ 树的所有数据都在叶子节点，非叶子节点只保存索引，叶子节点之间用链表串起来，这样做可以高效支持范围查询；而普通 B 树非叶子节点也存数据，会导致查找效率稍低。”

---

## 2. 聚簇索引 / 非聚簇索引 / 覆盖索引 / 联合索引

### 知识点

- **聚簇索引（Clustered Index）**
  - 表数据和索引存放在一起（InnoDB PK 默认聚簇索引）
  - 一张表只能有一个聚簇索引

- **非聚簇索引（Secondary Index）**
  - 索引节点保存主键，数据不在索引中，需要回表查询

- **覆盖索引（Covering Index）**
  - 查询所需字段都包含在索引中，无需回表

- **联合索引（Composite Index）**
  - 多列组成的索引，涉及最左前缀原则

### 白话回答示例

> “聚簇索引就是数据和索引放在一起，比如主键索引；非聚簇索引单独存，查不到数据要回表。覆盖索引则是查询的字段全被索引覆盖，不用回表，查询效率更高。联合索引是多列组合索引，但要遵循最左匹配原则。”

---

## 3. 最左匹配原则

### 知识点

- 联合索引：`(a,b,c)`
- 查询条件用到最左的连续前缀才会使用索引
- 示例：
  - `WHERE a=1 AND b=2` → 使用索引
  - `WHERE b=2` → 不用索引

### 白话回答示例

> “联合索引查询时，必须从索引最左边的列开始匹配，否则索引失效。比如索引是(a,b,c)，如果查询只用 b 列，就走不了索引。”

---

## 4. 事务四大特性 ACID & 实现原理

### 知识点

- **A**: 原子性（Atomic） → 回滚日志
- **C**: 一致性（Consistency） → 约束、触发器保证
- **I**: 隔离性（Isolation） → 锁或 MVCC
- **D**: 持久性（Durability） → redo log 保证

### 白话回答示例

> “事务的 ACID 保证数据库操作安全：原子性保证要么全成功要么全失败，一致性保证操作后数据符合规则，隔离性防止并发冲突，持久性保证提交的数据不会丢失。InnoDB 通过 undo/redo 日志、锁和 MVCC 实现这些特性。”

---

## 5. 四大隔离级别 & MVCC

### 知识点

| 隔离级别         | 脏读 | 不可重复读 | 幻读 |
| ---------------- | ---- | ---------- | ---- |
| READ UNCOMMITTED | ✔    | ✔          | ✔    |
| READ COMMITTED   | ✖    | ✔          | ✔    |
| REPEATABLE READ  | ✖    | ✖          | ✔    |
| SERIALIZABLE     | ✖    | ✖          | ✖    |

- **MVCC（多版本并发控制）**
  - 通过存储行的多个版本（undo log）实现
  - 查询读取快照版本，保证非锁操作下的一致性

### 白话回答示例

> “MySQL 的四种隔离级别从低到高是 READ UNCOMMITTED、READ COMMITTED、REPEATABLE READ 和 SERIALIZABLE。默认 InnoDB 是 REPEATABLE READ，避免不可重复读。MVCC 是通过保存行的历史版本实现的，读操作不加锁，只看快照版本，提高并发。”

---

## 6. 慢查询排查步骤

### 知识点

1. **开启慢查询日志**：`slow_query_log=ON`
2. **定位慢 SQL**：`long_query_time`
3. **分析执行计划**：`EXPLAIN SELECT ...`
4. **优化手段**：
   - 索引优化
   - 避免全表扫描
   - 查询拆分、分页
   - SQL 调整
   - 数据库参数调优（缓存、并发）

### 白话回答示例

> “排查慢查询一般先开慢查询日志，找到慢 SQL，然后用 EXPLAIN 看执行计划，看是走索引还是全表扫描。慢查询常用优化手段包括加索引、拆分查询、优化 SQL、调缓存或者调整参数。”

---

## 7. 实际项目慢查询优化示例

### 知识点

- 找出慢查询：`show processlist` / 慢查询日志
- 用索引覆盖查询字段
- 避免 SELECT \*，只取必要字段
- 大表分页优化（id 范围分页）
- SQL 重写（IN → JOIN / EXISTS）

### 白话回答示例

> “在项目中，我遇到过用户表的大表查询特别慢。我先开启慢查询日志，定位慢 SQL，然后加了联合索引覆盖查询字段，同时优化了分页逻辑，用 id 范围代替 OFFSET 分页。优化后查询速度提升了 5 倍。”

---

## 8. 常用慢查询定位工具及使用方式

### 工具

- **EXPLAIN** → 查看执行计划
- **SHOW PROFILE / SHOW STATUS** → 查看 SQL 执行耗时
- **pt-query-digest (Percona Toolkit)** → 分析慢查询日志
- **MySQL Workbench / Navicat** → GUI 可视化分析

### 白话回答示例

> “我常用 EXPLAIN 看 SQL 执行计划，确认索引是否命中。复杂慢查询，我会用 pt-query-digest 分析慢查询日志，找出最耗时的 SQL，然后针对性优化。必要时也用 SHOW PROFILE 或 GUI 工具可视化分析。”

---

# AI / 大语言模型

## RAG（检索增强生成）

---

### 1. 什么是 RAG？完整流程是什么？

#### 涉及知识点

RAG（Retrieval-Augmented Generation）：

核心思想：

```text id="01hhne"
先检索知识
再让模型生成答案
```

而不是只依赖模型训练时学到的内容。

完整流程：

```text id="clx1gf"
文档导入
↓
Chunk切分
↓
Embedding向量化
↓
存入向量数据库
↓
用户提问
↓
问题向量化
↓
向量检索TopK
↓
拼接Prompt
↓
LLM生成答案
```

---

#### 实际工程（LangChain）

例如：

```python id="7c7x4r"
from langchain_community.document_loaders import TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma

# 1. 加载文档
loader = TextLoader("data.txt", encoding="utf-8")
docs = loader.load()

# 2. chunk切分
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=100
)

chunks = splitter.split_documents(docs)

# 3. embedding
embedding = OpenAIEmbeddings()

# 4. 存入向量数据库
db = Chroma.from_documents(
    chunks,
    embedding=embedding,
    persist_directory="./chroma_db"
)
```

---

#### 顺带解释几个词

##### Embedding

把文本转成向量。

语义越接近：
向量距离越近。

---

##### 向量数据库

用于：

```text id="9lnylp"
相似度检索
```

例如：

- Chroma
- Milvus
- pgvector

---

#### 白话回答

> “RAG 本质是先检索知识，再让模型生成答案。
>
> 我实际项目里用 LangChain 做文档加载和 chunk 切分，用 embedding 做向量化，再存到 Chroma 向量数据库，查询时先召回相关内容，再拼接 Prompt 给模型。”

---

### 2. 文档分块（Chunk）怎么做？

#### 涉及知识点

为什么要切块：

因为：

- embedding 有长度限制
- 整篇文档太长
- 检索粒度会太粗

---

#### Chunk 原则

太小：

- 上下文断裂
- 信息不完整

---

太大：

- 检索不精准
- token 成本高

---

#### 常见工程方案

例如：

```text id="m7t26n"
chunk_size = 500
chunk_overlap = 100
```

---

#### overlap 是什么

Chunk 之间保留重叠区域。

避免上下文被截断。

---

#### LangChain 实践代码

```python id="wwm45w"
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=100,
    separators=["\n\n", "\n", "。", " "]
)
```

---

#### 为什么用 RecursiveCharacterTextSplitter

因为它会：

```text id="y6b1iz"
优先按语义切
再按长度切
```

例如优先：

- 标题
- 段落
- 换行

比纯固定长度效果更好。

---

#### 白话回答

> “Chunk 我一般控制在 300~800 token，并设置 overlap 防止上下文断裂。
>
> 实际项目里我用 LangChain 的 RecursiveCharacterTextSplitter，优先按段落和换行做语义切分，而不是纯固定长度切割。”

---

### 3. RAG 检索做了哪些优化？

#### 涉及知识点

实际项目里：
不能只做向量检索。

否则容易：

- 语义漂移
- 召回不准
- 噪声太多

---

#### 常见优化

##### Chunk 优化

控制：

- chunk 大小
- overlap

---

##### 双路检索（重点）

同时做：

```text id="x5tztm"
关键词检索（BM25）
+
向量检索（Embedding）
```

---

##### Rerank（重排序）

第一次召回后：
再重新排序。

提高准确率。

---

##### Metadata Filter

先过滤：

- 文档类型
- 时间
- 标签

再检索。

---

#### LangChain 实践代码

##### 向量检索

```python id="h9k9it"
retriever = db.as_retriever(
    search_kwargs={"k": 6}
)
```

---

##### Metadata 过滤

```python id="p2y4jy"
retriever.invoke(
    "mysql事务",
    filter={"type": "mysql"}
)
```

---

#### 什么是语义漂移

用户问：

```text id="4vzux9"
mysql锁
```

结果召回：

```text id="j0y39v"
操作系统锁
```

方向偏了。

---

#### 白话回答

> “RAG 检索我主要做了 chunk 优化、双路检索、metadata 过滤和 rerank。
>
> 因为纯向量检索容易语义漂移，所以我会结合 BM25 做混合召回。”

---

### 4. 双重检索（语义检索 + 关键词检索）讲一下

#### 涉及知识点

实际项目里：

通常不会只做 embedding 检索。

而是：

```text id="ww8igw"
BM25关键词检索
+
向量语义检索
```

一起做。

---

#### 为什么要双路检索

---

#### BM25 优点

适合：

- API 名称
- 专业术语
- 缩写词

例如：

```text id="5vz0rz"
JWT
Redis
MySQL
```

关键词匹配更稳定。

---

#### 向量检索优点

适合：

- 自然语言
- 相似表达

例如：

```text id="9k6lgt"
“mysql索引”
“数据库index”
```

---

#### LangChain 实践代码

##### BM25

```python id="zshv45"
from langchain_community.retrievers import BM25Retriever

bm25 = BM25Retriever.from_documents(chunks)
bm25.k = 3
```

---

##### 向量检索

```python id="qysd9p"
vector_retriever = db.as_retriever(
    search_kwargs={"k": 3}
)
```

---

##### 混合召回

```python id="9cx2fb"
docs1 = bm25.invoke(query)
docs2 = vector_retriever.invoke(query)

docs = docs1 + docs2
```

---

#### 白话回答

> “双重检索一般是 BM25 加 embedding 向量检索。
>
> 因为纯向量检索容易语义漂移，而 BM25 对专业术语更稳定，所以实际项目通常会混合召回。”

---

### 5. TopK 怎么定的？为什么是 6？

#### 涉及知识点

TopK：
表示召回多少条 chunk。

---

#### 为什么不能太小

太小：

- 信息不足
- 容易漏召回

---

#### 为什么不能太大

太大：

- 噪声增加
- token 成本上涨

---

#### 为什么很多项目是 5~10

工程经验值。

---

#### LangChain 实践代码

```python id="g8r6wp"
retriever = db.as_retriever(
    search_kwargs={"k": 6}
)
```

---

#### 为什么是 6（标准回答）

> “我测试过不同 TopK。
>
> 太小容易漏信息，太大又会引入噪声和额外 token 成本。
>
> 后来发现 5~8 区间效果比较稳定，所以最终选了 6，在召回率和生成质量之间比较平衡。”

---

### 6. 相似度阈值和过滤策略是什么？

#### 涉及知识点

向量检索会返回：

```text id="qghah2"
similarity score
```

例如：

```text id="rpp9n7"
0.92
0.83
0.31
```

---

#### 为什么需要阈值

避免召回无关内容。

例如：

```text id="9lpyki"
0.2
```

通常已经不相关。

---

#### 常见阈值

通常：

```text id="crp0h5"
0.7 ~ 0.85
```

---

#### LangChain 实践代码

```python id="pd5clg"
docs = db.similarity_search_with_score(query, k=6)

result = [
    doc for doc, score in docs
    if score > 0.75
]
```

---

#### 过滤策略

除了相似度：

还会过滤：

- 文档类型
- 时间
- 用户权限

---

#### 白话回答

> “相似度阈值主要是避免召回无关内容。
>
> 我一般会设置在 0.75 左右，再结合 metadata 过滤，提高检索准确率。”

# Spring 全家桶

---

## 1. Spring IOC 容器初始化流程（必背）

### 涉及知识点

IOC：

```text id="yv7c2i"
控制反转
```

核心思想：

对象不再自己创建。

而是交给 Spring 容器管理。

---

### IOC 初始化核心流程

```text id="l2q1u4"
启动 Spring
↓
读取配置类 / XML
↓
扫描 Bean
↓
BeanDefinition 注册
↓
实例化 Bean
↓
依赖注入（DI）
↓
初始化 Bean
↓
放入 IOC 容器
```

---

### 顺带解释几个词

#### BeanDefinition

Bean 的定义信息。

里面保存：

- 类名
- 作用域
- 是否懒加载
- 初始化方法

Spring 真正管理的是 BeanDefinition。

不是直接管理对象。

---

#### DI（依赖注入）

例如：

```java id="v8oz2r"
@Autowired
private UserService userService;
```

Spring 自动把依赖对象注入进来。

---

### 实际项目场景

例如：

```java id="u22c0x"
@Service
public class UserService {}
```

启动时：
Spring 会自动：

- 扫描
- 创建对象
- 注入依赖

开发者不用自己 new。

---

### 白话回答

> “IOC 就是对象交给 Spring 管理。
>
> Spring 启动后会先扫描 Bean，生成 BeanDefinition，然后实例化 Bean，完成依赖注入和初始化，最后放入 IOC 容器统一管理。”

---

## 2. Bean 生命周期

### 涉及知识点

Bean 生命周期核心流程：

```text id="vz6m79"
实例化
↓
属性注入
↓
Aware接口回调
↓
BeanPostProcessor前置处理
↓
初始化方法
↓
BeanPostProcessor后置处理
↓
Bean可使用
↓
销毁
```

---

### 顺带解释几个词

#### BeanPostProcessor

Bean 后置处理器。

Spring 很多功能都基于它。

例如：

- AOP
- 自动代理

---

#### Aware 接口

让 Bean 获取 Spring 内部资源。

例如：

```java id="3p5lvn"
BeanNameAware
ApplicationContextAware
```

---

### 初始化方法

例如：

```java id="n2glgx"
@PostConstruct
```

或者：

```java id="e6z2mi"
init-method
```

---

### 实际项目场景

例如：

数据库连接池初始化：

```java id="a2j04z"
@PostConstruct
public void init() {}
```

项目启动时：
自动执行初始化逻辑。

---

### 白话回答

> “Bean 生命周期主要包括实例化、依赖注入、初始化和销毁。
>
> Spring 在初始化过程中还会执行 BeanPostProcessor，这也是 AOP 和自动代理实现的重要基础。”

---

## 3. 循环依赖：三级缓存原理

### 涉及知识点

循环依赖：

例如：

```text id="4n9zc5"
A 依赖 B
B 又依赖 A
```

---

### 为什么会有问题

创建 A：
需要 B。

创建 B：
又需要 A。

互相等待。

---

### Spring 如何解决

核心：

```text id="jlwmrx"
三级缓存
```

---

### 三级缓存

#### 一级缓存

```text id="qit6qt"
singletonObjects
```

已经完成初始化的 Bean。

---

#### 二级缓存

```text id="84m8cg"
earlySingletonObjects
```

提前暴露的 Bean。

半成品对象。

---

#### 三级缓存

```text id="jlwm3m"
singletonFactories
```

对象工厂。

用于生成代理对象。

---

#### 为什么需要三级缓存

因为 Spring 不只是解决循环依赖。

还要支持：

```text id="m7eq0z"
AOP代理对象
```

---

#### 实际流程

例如：

```text id="5y0m5k"
A 依赖 B
B 依赖 A
```

创建 A 时：

```text id="n2xft6"
A实例化
↓
提前放入三级缓存
↓
发现需要B
↓
创建B
↓
B发现需要A
↓
从三级缓存拿A
```

这样循环就断开了。

---

### 为什么构造器循环依赖解决不了

因为：

```text id="1zpwkn"
对象都还没创建出来
```

没法提前暴露。

---

### 白话回答

> “Spring 通过三级缓存解决单例 Bean 的循环依赖。
>
> 核心思想是 Bean 实例化后先提前暴露，再进行属性注入。
>
> 三级缓存主要是为了支持 AOP 代理对象，否则二级缓存其实就够了。”

---

## 4. AOP 底层：JDK 动态代理 / CGLIB

### 涉及知识点

AOP：

```text id="yby3d6"
面向切面编程
```

用于：

- 不修改业务代码
- 统一增强功能

---

### 常见场景

例如：

- 日志
- 事务
- 权限校验
- 接口耗时统计

---

### Spring AOP 两种代理方式

| 方式         | 条件     |
| ------------ | -------- |
| JDK 动态代理 | 有接口   |
| CGLIB        | 没有接口 |

---

### JDK 动态代理

基于：

```text id="pjw1f2"
接口
```

生成代理类。

---

### CGLIB

基于：

```text id="1clxv6"
继承
```

生成子类代理。

---

### 为什么 final 不能 AOP

因为：

```text id="6nnw4v"
CGLIB 需要继承
```

final 方法不能被重写。

---

### 实际项目场景

例如：

```java id="fhkh0x"
@Transactional
```

底层就是 AOP。

Spring 会生成代理对象：
在方法前后加入事务逻辑。

---

### 白话回答

> “Spring AOP 底层主要通过 JDK 动态代理和 CGLIB 实现。
>
> 有接口默认用 JDK 动态代理，没有接口用 CGLIB。
>
> Spring 事务、日志、权限校验本质上都是 AOP。”

---

## 5. Spring Boot 自动装配原理

### 涉及知识点

Spring Boot 核心：

```text id="95wjlwm"
自动装配
```

目的：

减少 XML 配置。

---

### 核心注解

```java id="s7aq4r"
@SpringBootApplication
```

本质包含：

```text id="2y7qes"
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

---

### 自动装配核心流程

```text id="eq9t3n"
@EnableAutoConfiguration
↓
加载 META-INF/spring.factories
↓
读取自动配置类
↓
按条件装配 Bean
```

---

### 为什么 Starter 能自动生效

因为 Starter：

会提前配置：

```text id="1i9ywr"
AutoConfiguration
```

Spring Boot 启动时自动加载。

---

### 实际项目场景

例如：

引入：

```xml id="n9p93n"
spring-boot-starter-web
```

就自动配置：

- Tomcat
- Spring MVC
- JSON 转换器

不需要自己写 XML。

---

### 白话回答

> “Spring Boot 自动装配核心是 @EnableAutoConfiguration。
>
> 启动时会读取 spring.factories 中的自动配置类，再根据条件自动创建 Bean。
>
> Starter 能开箱即用，本质就是提前帮我们写好了自动配置。”

---

## 6. @Conditional 系列注解作用

### 涉及知识点

作用：

```text id="dd8gkr"
按条件装配Bean
```

满足条件才创建 Bean。

---

### 常见注解

| 注解                      | 作用              |
| ------------------------- | ----------------- |
| @ConditionalOnClass       | 类存在才生效      |
| @ConditionalOnBean        | Bean 存在才生效   |
| @ConditionalOnMissingBean | Bean 不存在才生效 |
| @ConditionalOnProperty    | 配置项满足才生效  |

---

### 实际项目场景

例如：

```java id="2j3p8l"
@ConditionalOnMissingBean
```

用户没定义 Bean：
Spring 才自动创建默认 Bean。

避免冲突。

---

### 白话回答

> “@Conditional 系列注解用于条件装配。
>
> Spring Boot 自动配置大量依赖这些注解，满足条件才创建 Bean，避免无效配置和 Bean 冲突。”

---

## 7. Spring MVC 执行流程

### 涉及知识点

Spring MVC 核心流程：

```text id="fp0tpz"
请求
↓
DispatcherServlet
↓
HandlerMapping
↓
Controller
↓
Service
↓
返回ModelAndView/JSON
↓
视图解析
↓
响应
```

---

### 顺带解释几个词

#### DispatcherServlet

Spring MVC 核心前端控制器。

所有请求先到它。

---

#### HandlerMapping

负责：

```text id="m3l2ti"
URL -> Controller
```

映射。

---

#### ViewResolver

负责页面解析。

例如：

```text id="wljlwm"
login
↓
login.jsp
```

---

### 实际项目场景

前后端分离里：

通常：

```text id="jlwm99"
返回JSON
```

不走 JSP 页面。

---

### 白话回答

> “Spring MVC 的核心是 DispatcherServlet。
>
> 请求进入后会通过 HandlerMapping 找到对应 Controller，再调用业务逻辑，最后返回 JSON 或视图响应。”

---

## 8. Spring 事务传播机制、隔离级别

### 涉及知识点

---

### 事务传播机制

核心：

```text id="wvjlwm"
多个事务方法互相调用时
事务怎么传播
```

---

### 常见传播行为

| 传播机制     | 含义                           |
| ------------ | ------------------------------ |
| REQUIRED     | 默认，有事务就加入，没有就新建 |
| REQUIRES_NEW | 新开事务                       |
| SUPPORTS     | 有事务就加入，没有就非事务执行 |

---

### 实际项目场景

例如：

订单主流程失败：
日志保存不能回滚。

日志方法：

```java id="jlwmw1"
REQUIRES_NEW
```

单独事务。

---

### 隔离级别

用于解决并发问题。

---

### 并发问题

| 问题       | 含义                     |
| ---------- | ------------------------ |
| 脏读       | 读到别人未提交数据       |
| 不可重复读 | 同一数据两次读取结果不同 |
| 幻读       | 同条件查询结果数量变化   |

---

### 四种隔离级别

| 隔离级别         | 能解决什么                  |
| ---------------- | --------------------------- |
| READ_UNCOMMITTED | 最低隔离                    |
| READ_COMMITTED   | 避免脏读                    |
| REPEATABLE_READ  | 避免不可重复读（MySQL默认） |
| SERIALIZABLE     | 完全串行化                  |

---

### 白话回答

> “Spring 事务传播机制用于控制事务之间怎么传递。
>
> 最常用的是 REQUIRED 和 REQUIRES_NEW。
>
> 隔离级别则用于解决并发问题，比如脏读、不可重复读和幻读。
>
> MySQL 默认一般是 REPEATABLE_READ。”

---

# Java 基础

---

## 1. HashMap 底层实现、扩容、哈希冲突、JDK1.7/1.8 区别

### 涉及知识点

HashMap 底层：

```text id="gn0g07"
数组 + 链表 + 红黑树（JDK1.8）
```

---

### HashMap 存储流程

```text id="k8q5lc"
key
↓
hash计算
↓
定位数组下标
↓
放入桶(bucket)
```

---

### 什么是哈希冲突

不同 key：

计算后落到同一个数组位置。

例如：

```text id="jlwmj2"
hash(A) % 16 = 1
hash(B) % 16 = 1
```

这就是哈希冲突。

---

### 如何解决冲突

JDK1.7：

```text id="r7c9w4"
链表
```

JDK1.8：

```text id="jlwm23"
链表长度超过8
↓
转红黑树
```

---

### 为什么转红黑树

链表查找：

```text id="jlwm24"
O(n)
```

红黑树：

```text id="jlwm25"
O(logn)
```

---

### 扩容机制

默认容量：

```text id="jlwm26"
16
```

默认负载因子：

```text id="jlwm27"
0.75
```

达到：

```text id="jlwm28"
16 * 0.75 = 12
```

就扩容。

---

### 什么是负载因子

衡量：

```text id="jlwm29"
哈希表装满程度
```

---

### 为什么是 0.75

空间利用率和哈希冲突之间的平衡。

---

### JDK1.7 vs 1.8

| 版本   | 区别             |
| ------ | ---------------- |
| JDK1.7 | 数组+链表        |
| JDK1.8 | 数组+链表+红黑树 |

---

### 1.7 为什么线程不安全更严重

1.7 扩容采用：

```text id="jlwm30"
头插法
```

多线程下可能：

```text id="jlwm31"
链表成环
```

导致死循环。

---

### 1.8 改进

采用：

```text id="jlwm32"
尾插法
```

避免链表成环。

---

### 白话回答

> “HashMap 底层是数组加链表，JDK1.8 后增加了红黑树优化。
>
> key 会先计算 hash，再定位数组位置。
>
> 如果多个 key 落到同一个桶，就发生哈希冲突。
>
> JDK1.8 在链表过长时会转红黑树，提高查询效率。
>
> HashMap 达到负载因子后会扩容，默认容量16，负载因子0.75。”

---

## 2. HashMap 和 ConcurrentHashMap 区别、锁机制

### 涉及知识点

---

### HashMap

特点：

```text id="jlwm33"
线程不安全
```

多线程可能：

- 数据覆盖
- 死循环
- 丢数据

---

### ConcurrentHashMap

特点：

```text id="jlwm34"
线程安全
```

适合并发场景。

---

### JDK1.7 锁机制

```text id="jlwm35"
Segment分段锁
```

多个 Segment 独立加锁。

减少锁竞争。

---

### JDK1.8 锁机制

取消 Segment。

采用：

```text id="jlwm36"
CAS + synchronized
```

---

### 什么是 CAS

Compare And Swap。

比较并交换。

一种无锁并发机制。

---

### ConcurrentHashMap 为什么性能更高

因为：

```text id="jlwm37"
锁粒度更小
```

不是整个 Map 加锁。

---

### 实际项目场景

例如：

缓存统计：

```java id="jlwm38"
ConcurrentHashMap<String, AtomicInteger>
```

统计接口访问次数。

---

### 白话回答

> “HashMap 是线程不安全的，而 ConcurrentHashMap 支持高并发。
>
> JDK1.7 使用 Segment 分段锁，JDK1.8 改成 CAS 加 synchronized，锁粒度更小，所以并发性能更高。”

---

## 3. volatile、synchronized、Lock 区别

### 涉及知识点

---

### volatile

作用：

```text id="jlwm39"
保证可见性
禁止指令重排
```

---

### 什么是可见性

一个线程修改变量：

其他线程立即可见。

---

### 什么是指令重排

CPU 为了优化性能：

可能调整代码执行顺序。

volatile 可以禁止这种重排。

---

### volatile 不能保证什么

不能保证：

```text id="jlwm40"
原子性
```

---

### 什么是原子性

操作：

```text id="jlwm41"
不可被中断
```

例如：

```java id="jlwm42"
i++
```

其实不是原子操作。

---

### synchronized

作用：

```text id="jlwm43"
保证线程同步
```

底层：

```text id="jlwm44"
对象监视器Monitor
```

---

### Lock（ReentrantLock）

JDK 提供的显式锁。

相比 synchronized：

功能更多。

例如：

- 可中断
- 超时锁
- 公平锁

---

### 区别总结

| 关键字       | 特点           |
| ------------ | -------------- |
| volatile     | 保证可见性     |
| synchronized | 自动加锁释放锁 |
| Lock         | 更灵活         |

---

### 白话回答

> “volatile 主要保证可见性和禁止指令重排，但不能保证原子性。
>
> synchronized 用于线程同步，底层基于 Monitor。
>
> Lock 比 synchronized 更灵活，支持公平锁、可中断锁和超时机制。”

---

## 4. 谈谈 ReentrantLock 的理解

### 涉及知识点

Reentrant：

```text id="jlwm45"
可重入
```

意思：

同一个线程：
可以重复获取同一把锁。

---

### 为什么需要可重入

例如：

```java id="jlwm46"
A方法加锁
↓
调用B方法
↓
B方法也需要同一把锁
```

如果不可重入：
会死锁。

---

### ReentrantLock 特点

相比 synchronized：

支持：

- 公平锁
- tryLock
- 可中断锁
- Condition

---

### 什么是公平锁

先等待的线程先获得锁。

---

### 什么是 Condition

类似：

```text id="jlwm47"
wait/notify
```

但功能更强。

---

### 实际项目场景

例如：

异步任务队列：

```java id="jlwm48"
lock.tryLock(1, TimeUnit.SECONDS)
```

避免线程长期阻塞。

---

### 白话回答

> “ReentrantLock 是可重入锁，同一个线程可以重复获取锁。
>
> 它相比 synchronized 更灵活，支持公平锁、tryLock、可中断锁等高级功能。”

---

## 5. 线程池核心知识点

### 涉及知识点

线程池作用：

```text id="jlwm49"
线程复用
```

避免频繁：

- 创建线程
- 销毁线程

降低系统开销。

---

### 为什么不能一直 new Thread

线程创建成本高。

高并发下：
容易：

- CPU 飙升
- OOM
- 系统崩溃

---

### 线程池核心流程

```text id="jlwm50"
提交任务
↓
核心线程执行
↓
队列缓存
↓
创建非核心线程
↓
触发拒绝策略
```

---

### 实际项目场景

例如：

异步日志处理：

- 请求线程快速返回
- 后台线程异步写数据库

---

### 白话回答

> “线程池核心目的是线程复用，降低线程创建销毁开销。
>
> 高并发项目里通常会用线程池处理异步任务，而不是频繁 new Thread。”

---

## 6. 线程池七大参数

### 涉及知识点

```java id="jlwm51"
ThreadPoolExecutor
```

七大参数：

| 参数            | 含义             |
| --------------- | ---------------- |
| corePoolSize    | 核心线程数       |
| maximumPoolSize | 最大线程数       |
| keepAliveTime   | 空闲线程存活时间 |
| unit            | 时间单位         |
| workQueue       | 阻塞队列         |
| threadFactory   | 线程工厂         |
| handler         | 拒绝策略         |

---

### 拒绝策略

| 策略                | 含义         |
| ------------------- | ------------ |
| AbortPolicy         | 抛异常       |
| CallerRunsPolicy    | 调用线程执行 |
| DiscardPolicy       | 丢弃         |
| DiscardOldestPolicy | 丢弃最旧任务 |

---

### 白话回答

> “线程池核心是 ThreadPoolExecutor。
>
> 最重要的参数包括核心线程数、最大线程数、阻塞队列和拒绝策略。
>
> 当线程和队列都满时，就会触发拒绝策略。”

---

## 7. 线程安全集合有哪些、怎么实现安全

### 涉及知识点

---

### Vector

早期线程安全 List。

方法加：

```text id="jlwm52"
synchronized
```

性能较差。

---

### Collections.synchronizedXXX

包装同步集合。

---

### ConcurrentHashMap

高并发 Map。

---

### CopyOnWriteArrayList

写时复制。

适合：

```text id="jlwm53"
读多写少
```

---

### 什么是写时复制

写数据时：
复制新数组。

读线程不加锁。

---

### 实际项目场景

配置缓存：

- 很少修改
- 经常读取

适合：

```text id="jlwm54"
CopyOnWriteArrayList
```

---

### 白话回答

> “常见线程安全集合包括 Vector、ConcurrentHashMap 和 CopyOnWriteArrayList。
>
> ConcurrentHashMap 适合高并发场景，CopyOnWriteArrayList 适合读多写少。”

# Agent 岗位问题

---

## Python 三大器

```text id="wpm3cx"
生成器
迭代器
装饰器
```

---

### 回答

> “Python 常说的三大器一般是生成器、迭代器和装饰器。
>
> 迭代器核心是实现 `__iter__` 和 `__next__`；
>
> 生成器本质是特殊迭代器，通过 yield 实现惰性生成，适合大数据量场景；
>
> 装饰器本质是函数增强，不修改原函数的情况下增加功能，Python Web 框架和日志、权限控制里很常见。”

---

## 异步和多线程

---

### 回答

> “异步主要解决 IO 等待问题，多线程主要解决任务并发问题。
>
> Python 由于 GIL 的存在，多线程在 CPU 密集型场景优势有限，但在网络请求、数据库访问这种 IO 密集型场景依然有效。
>
> AI Agent 场景里我更多会用 asyncio，比如并发调用多个 API、并发向量检索、并发 Tool Calling。”

---

## 如何优化 SQL 查询速度

---

### 回答

> “我一般先通过 explain 看执行计划，重点看有没有走索引、是否全表扫描。
>
> 常见优化包括：
>
> - 增加索引
> - 联合索引
> - 避免 select \*
> - 减少回表
> - 大分页优化
>
> 如果数据量比较大，还会考虑读写分离或者缓存。”

---

## AI 和 Agent 的区别

---

### 回答

> “普通 AI 更像一次性问答，输入 Prompt 返回结果。
>
> Agent 则更强调：
>
> - 自主决策
> - 多步骤执行
> - 工具调用
> - 状态管理
>
> 比如普通 LLM 只能回答天气，而 Agent 可以：
>
> - 自动调用天气 API
> - 解析结果
> - 继续完成后续任务。”

---

## LangChain 常用方法

---

### 回答

> “我实际用得比较多的是：
>
> - PromptTemplate
> - ChatOpenAI
> - RunnableSequence
> - Retriever
> - Memory
> - Tool
> - AgentExecutor
>
> RAG 里主要会用 document loader、text splitter 和 vector store。”

---

## 你一般怎么写 Prompt

---

### 回答

> “我一般会把 Prompt 分成：
>
> - system prompt
> - user prompt
> - context
>
> 然后尽量：
>
> - 明确角色
> - 明确输出格式
> - 限制模型行为
> - 提供 few-shot 示例
>
> RAG 场景里还会明确要求：
>
> ‘只能根据检索内容回答，不允许编造。’”

---

## 常见 RAG 搜索策略

---

### 回答

> “常见策略包括：
>
> - 向量检索
> - BM25关键词检索
> - 混合检索
> - rerank重排序
> - metadata filter
>
> 实际项目里我更倾向双路召回，因为纯向量检索容易语义漂移。”

---

## 如何避免 AI 幻觉

---

### 回答

> “避免幻觉主要有几个方向：
>
> - RAG 提供外部知识
> - Prompt 限制回答范围
> - 降低 temperature
> - 增加相似度阈值过滤
> - 增加 rerank
>
> 对于高风险场景，还会要求模型返回引用来源。”

---

## 多头注意力机制

---

### 回答

> “多头注意力本质是让模型从多个不同语义角度理解文本关系。
>
> 比如一句话里：
>
> - 一个 head 关注主语
> - 一个关注时态
> - 一个关注上下文关联
>
> 最后再把多个 head 的结果融合，提高模型对上下文的理解能力。”

---

## GPT5 相比其他 AI 做了什么优化

---

### 回答

> “我更关注实际使用体验。
>
> 相比很多模型，GPT 系列在：
>
> - 长上下文
> - 工具调用
> - 推理稳定性
> - 指令遵循
> - 多轮对话
>
> 上整体会更成熟。
>
> 特别是复杂 Agent Workflow 场景，稳定性会明显更好。”

---

## 最近写的 Agent 是什么，怎么做的

---

### 回答

> “我最近做的是一个基于 LangChain 的 RAG Agent。
>
> 核心功能包括：
>
> - 文档知识库检索
> - 多轮对话记忆
> - Tool Calling
> - SSE 流式输出
>
> 技术栈是：
>
> - FastAPI
> - LangChain
> - Chroma
> - OpenAI/Ollama
>
> 文档会先 chunk 切分，再 embedding 存入向量库。
>
> 查询时采用 BM25 + 向量检索双路召回，并增加 rerank。
>
> Agent 部分支持工具调用，比如联网搜索和本地知识查询。”

---

## LangGraph

---

### 回答

> “LangGraph 本质是基于状态机的 Agent Workflow。
>
> 相比 LangChain 的链式调用，LangGraph 更适合：
>
> - 多步骤决策
> - 条件分支
> - 状态流转
> - 多 Agent 协作
>
> 我实际用它做过：
>
> - Tool Calling
> - Agent 状态管理
> - 多轮执行流程。”

---

## ES 搜索机制

---

### 回答

> “ES 底层基于倒排索引。
>
> 它会把词和文档映射关系提前建立好，所以搜索速度非常快。
>
> AI 场景里我更多会用 ES 做：
>
> - BM25关键词检索
> - Hybrid Search
> - metadata filter。”

---

## Function Calling 和 MCP

---

### 回答

> “Function Calling 本质是模型输出结构化工具调用参数，由服务端真正执行工具。
>
> MCP 更像统一工具协议，让模型可以标准化接入外部工具、数据库和服务。
>
> 我理解 MCP 更偏：
>
> ‘AI 时代的工具接口标准化。’”

---

## 大量知识文本入库怎么处理

---

### 回答

> “大量文本入库前，我会先做：
>
> - 文档清洗
> - 去重
> - chunk切分
> - metadata提取
>
> 然后异步 embedding。
>
> 检索层面会：
>
> - 控制 chunk 大小
> - 建立 metadata filter
> - 做混合检索
>
> 避免向量库规模过大后检索效率下降。”

---

## 最难的问题是什么

---

### 回答

> “我之前遇到过一个比较难的问题是：
>
> RAG 检索结果不稳定，模型经常答非所问。
>
> 后来排查发现：
>
> - chunk 切分太粗
> - 纯向量检索语义漂移
>
> 后面我重新设计了 chunk 策略，并增加 BM25 混合检索和 rerank，最终回答准确率明显提升。
>
> 整个优化过程大概花了一周左右。”

---

## 做过什么项目，有什么可复刻经验

---

### 回答

> “我之前做过后端、自动化和 AI Agent 相关项目。
>
> 我觉得比较能复刻的是：
>
> - 快速学习新技术
> - 工程落地能力
> - 异步并发和服务化经验
>
> AI 这块虽然接触时间不算特别长，但我进入状态比较快，也比较关注实际落地而不是概念。”

---

## 和同事冲突怎么办

---

### 推荐回答

> “我一般会先确认问题是技术分歧还是沟通问题。
>
> 如果是技术问题，我更倾向用数据、测试结果和实际效果讨论，而不是情绪化争论。
>
> 最终目标还是把项目做好。”

---

## 预期薪资

---

### 回答

> “我目前预期大概在 15K 左右，但我更看重岗位方向、成长空间和团队氛围，具体也可以结合岗位职责沟通。”

---

## 明天能到岗么

---

### 回答

> “如果流程确定，我这边可以尽快配合入职，一般一周内没问题。”
