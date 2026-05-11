---
title: Twitter 爬虫系统的第二次进化：从 Selenium 到 Playwright + API 并发架构
date: 2021-07-10
category:
  - 项目实践
tag:
  - Python
  - 爬虫
  - Playwright
  - AsyncIO
  - Twitter API
  - 工程化
---

# Twitter 爬虫系统的第二次进化

距离上一次写 Twitter / Pixiv 爬虫实践，大概过去了 7 个月。

这段时间里，我对“爬虫”这件事的理解发生了一个比较明显的变化：

```text
爬虫 ≠ 模拟浏览器
爬虫 = 数据入口工程化调度系统
```

---

# 这次目标其实更“现实”

上一版还停留在：

- Selenium 翻页
- XPath 抓 HTML
- 人工 scroll + sleep

而这一版的目标变成：

```text
尽可能稳定地批量处理 Twitter 推文数据
```

核心需求只有三个：

- 稳定拿到 tweet 结构化数据
- 高并发处理 URL 队列
- 避免浏览器阻塞成为瓶颈

---

# 技术栈的变化

这一版明显分裂成两个层：

## 1. 数据层（API 优先）

```text
Twitter GraphQL API / v2 API
```

直接绕开 HTML。

---

## 2. 展示层（Playwright）

```text
Playwright（替代 Selenium）
```

只在必要场景下使用浏览器：

- 登录态继承
- cookie 获取
- 页面验证辅助

---

# 一个关键转折：不再“解析网页”

上一版核心是：

```text
XPath + DOM结构
```

这一版核心变成：

```python
GraphQL + JSON
```

例如：

```python
https://twitter.com/i/api/graphql/D_jNhjWZeRZT5NURzfJZSQ/TweetResultByRestId
```

---

# 认知变化1：Twitter 本质是 GraphQL API 系统

HTML 只是壳。

真正数据来源：

- tweetResult
- legacy object
- user_results

---

# 认知变化2：数据结构比页面结构稳定得多

之前的问题是：

```text
DOM 改一下 -> 爬虫崩
```

现在变成：

```text
API字段变化 -> 才会影响逻辑
```

稳定性直接提升一个数量级。

---

# 系统结构开始“工程化”

这一版开始有明显系统拆分：

## 1. 爬取层

```python
get_tweet_info()
get_tweets_info()
```

支持：

- 单条
- 批量 asyncio 并发

---

## 2. 解析层

```python
legacy -> user -> media -> entities
```

统一结构化输出：

```json
{
  "tweet_id": "",
  "user_id": "",
  "media": [],
  "favorite_count": 0
}
```

---

## 3. 存储层

```python
twitter-log-YYYY_MM_DD.json
```

实现：

- 增量写入
- 去重 tweet_id
- 历史恢复

---

# 并发体系：第一次真正引入 AsyncIO

这一版最重要变化其实不是 Playwright，而是：

```python
asyncio + httpx.AsyncClient
```

---

## API 批处理：

```python
asyncio.gather(*queries)
```

核心意义：

```text
从“一个一个抓”
变成“批量调度”
```

---

# GraphQL 请求构造

这一部分已经比较“底层工程化”了：

```python
params = {
    "variables": orjson.dumps({...}).decode(),
    "features": self.default_features
}
```

特点：

- 手动拼 variables
- feature flags 控制行为
- 请求结构完全固定化

---

# 一个关键优化：减少 Playwright 依赖

这一版 Playwright 已经不再做“爬虫核心”，而是：

```text
登录态容器 + cookie继承器
```

---

启动方式：

```python
launch_persistent_context(
    user_data_dir=ChromeProfile,
    headless=True
)
```

---

# 认知变化3：浏览器只是“身份工具”

上一版：

```text
浏览器 = 爬虫核心
```

这一版：

```text
浏览器 = 登录态来源
```

---

# 媒体处理逻辑变得更工程化

这一版对 media 做了分类：

```python
photo / video / animated_gif
```

并且做了：

- bitrate 选最高视频源
- url 补全逻辑
- 去重 id_str

---

# 一个关键点：Twitter 图片“原图恢复”

仍然保留了经典逻辑：

```python
media_url + "?format=xxx&name=4096x4096"
```

本质是：

```text
利用 URL 规则恢复原图
```

---

# 文件系统策略开始分层

下载路径结构：

```text
save_path/
    ln/
    mmj/
    vbs/
    ws/
    other/
```

---

来源：

```python
authors_sorted_dict
```

---

# 认知变化4：数据分类前置比后处理更重要

上一版是：

```text
下载完再整理
```

这一版是：

```text
下载前就决定归类路径
```

---

# 点赞系统（GraphQL mutation）

这一版甚至开始做“行为模拟”：

```python
FavoriteTweet mutation
```

本质：

```text
写接口调用，而不是模拟点击
```

---

# 失败处理机制更像“任务系统”

新增：

- 重试机制（try_time < 3）
- Rate limit sleep 5 min
- httpx 异常分类
- failed_tweets 记录

---

# 一个很关键的升级：系统自愈能力

这一版已经具备：

```text
失败 -> 记录 -> 延迟 -> 重试
```

而不是：

```text
失败 -> 直接崩
```

---

# 并发下载第二模块（async Playwright）

后半段代码明显是另一个实验：

```text
多页面并发读取 tweet likes
```

结构：

- start_process
- get_likes
- download(page)

---

核心能力：

```text
批量打开页面 -> 提取点赞数 -> 重命名文件
```

---

# 一个很重要的设计：global_lock

说明当时已经在解决：

```text
并发冲突 + 状态同步问题
```

---

# 这一阶段的核心问题其实是：

## 1. 并发资源管理

```text
多个 page / context / browser 生命周期
```

---

## 2. 任务同步

```text
global_id + global_lock
```

---

## 3. 文件系统竞争

```text
rename + exists check
```

---

# 认知变化5：爬虫开始接近“分布式任务模型”

虽然还是单机，但已经具备：

- worker 思想
- queue 思想
- batch slicing

---

# 一个很明显的趋势

对比上一版：

| 阶段 | 技术                       |
| ---- | -------------------------- |
| v1   | Selenium + XPath           |
| v2   | API + AsyncIO + Playwright |

---

# 本质变化总结

这一版最大的变化不是代码，而是思维：

## 从：

```text
“我要模拟网页行为”
```

## 到：

```text
“我要消费接口数据流”
```

---

# 最后一个现实结论

这一阶段其实已经可以看出来：

```text
浏览器爬虫只是过渡形态
```

真正稳定系统依赖：

- API 逆向
- 请求结构分析
- 并发调度
- 数据结构设计

---

# 后记（当时的状态）

这一版写完之后，有一个很明显的感觉：

```text
爬虫不再是“写脚本”
而是在搭一个小型数据系统
```
