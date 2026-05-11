---
title: 我研究过的网站反爬机制与破解思路整理
date: 2021-02-11
category:
  - Web安全
tag:
  - 爬虫
  - 反爬
---

# 我研究过的网站反爬机制与破解思路整理

很多人刚学 Python 爬虫时。

都会经历一个阶段：

```python
requests.get(url)
```

然后：

```python
response.text
```

就觉得：

```text
“原来爬虫这么简单”
```

但真正开始做：

- Twitter
- Pixiv
- 电商平台
- 社交网站
- AI平台
- 视频网站

之后才会意识到：

> 现代网站对抗的已经不是“脚本”，而是“自动化系统”。

后来逐渐发现：

现代爬虫真正难的地方不是：

```text
怎么发请求
```

而是：

```text
怎么伪装成“正常人”
```

---

# 现代反爬的核心思路

反爬从来不是单点策略，而是分层体系：

```text
请求层
↓
账号层
↓
行为层
↓
浏览器层
↓
设备层
↓
网络层
```

任意一层异常都会触发：

- 403
- 限流
- 风控
- 验证码
- 登录失效

---

# 第一阶段：Header 检测

这是最早遇到的一层门槛。

最初写法：

```python
import requests

requests.get(url)
```

结果直接：

```text
403 Forbidden
```

---

# User-Agent 只是最低门槛

补上：

```python
headers = {
    "User-Agent": "Mozilla/5.0"
}
```

部分站点可用，但很快会失效。

后来逐渐意识到：

```text
网站判断的不是“有没有UA”
而是“像不像浏览器”
```

---

# 更完整的 Header 体系

通过 F12 抓包后，开始复现浏览器请求：

```python
headers = {
    "User-Agent": "...",
    "Referer": "...",
    "Accept-Language": "...",
    "sec-fetch-site": "...",
    "sec-ch-ua": "...",
}
```

结论很明确：

```text
Header不能随便拼，只能“复刻浏览器”
```

---

# Session 的重要性

开始用：

```python
session = requests.Session()
```

原因很直接：

```text
Cookie + 连接复用 = 登录态维持基础
```

---

# 第二阶段：Cookie 与 Session

真正的分水岭是理解：

```text
登录状态本质是 Cookie
```

浏览器自动携带：

```http
Cookie: sessionid=xxx
```

---

# 常见三种处理方式

## 1. 手动 Cookie

直接从浏览器复制。

适用于快速验证。

---

## 2. Session 维持

```python
session = requests.Session()
```

通过登录接口自动获取 Cookie。

---

## 3. 浏览器复用

后期更稳定方案：

```text
直接复用真实浏览器登录态
```

---

# 第三阶段：动态参数与签名

进入真正的 JS 对抗阶段。

典型特征：

```text
请求参数每次都不同
```

常见字段：

- sign
- token
- nonce
- timestamp

---

# 排查流程

标准分析路径：

## 1. 抓包定位接口

```text
Network → XHR / Fetch
```

---

## 2. 请求重放

Python 复现请求。

---

## 3. 逐参数排除

删除：

- sign
- token
- timestamp

定位失败点。

---

## 4. JS 关键词搜索

```text
md5
sha1
CryptoJS
sign
```

---

# 动态参数分类

## 时间戳

```python
int(time.time() * 1000)
```

---

## 随机值

```text
uuid / nonce
```

---

## 哈希签名

```python
md5(token + timestamp + data)
```

---

# 三种实现策略

## 1. Python复刻

适用于简单签名逻辑。

---

## 2. execjs执行原JS

```python
import execjs
```

直接复用前端逻辑。

---

## 3. 浏览器执行

```text
Playwright / Selenium
```

让浏览器自己生成参数。

---

# JavaScript 混淆分析

混淆的本质不是加密，而是：

```text
增加阅读成本
```

常见形式：

- 字符串数组映射
- eval 执行
- 控制流平坦化

---

# 常见破解方式

## 1. 关键字搜索

```text
sign / token / md5 / CryptoJS
```

---

## 2. 断点调试

```text
Sources → Breakpoint
```

---

## 3. Hook API

```javascript
fetch;
XMLHttpRequest;
```

---

# Selenium 反检测问题

现代网站检测：

```javascript
navigator.webdriver;
```

Selenium 默认：

```text
true
```

---

# 解决方案演进

## undetected-chromedriver

基础绕过方案。

---

## Playwright

更现代方案：

- 自动等待
- 更真实行为
- 更稳定上下文

---

## 行为控制

避免：

- 高频请求
- 瞬时翻页
- 固定节奏操作

---

# 验证码问题的本质

验证码本质不是识别问题，而是：

```text
判断是否为自动化行为
```

---

# 实战策略

核心思路：

## 1. 降低风控触发

- 限速
- 随机等待
- 模拟真实操作

---

## 2. 复用登录态

避免频繁登录。

---

## 3. 使用真实浏览器

优先 Playwright。

---

# Cloudflare 风控体系

典型页面：

```text
Checking your browser...
```

---

# 检测维度

不仅是 HTTP：

- TLS 指纹
- JS执行环境
- 浏览器指纹
- Cookie状态
- IP信誉

---

# 应对路径

## cloudscraper

适用于轻度站点。

---

## Selenium

兼容性强但较重。

---

## Playwright

当前最稳定方案。

---

## 持久化浏览器上下文

```python
launch_persistent_context()
```

保存：

- Cookie
- LocalStorage
- 登录态

---

# 浏览器指纹体系

网站可识别：

- Canvas
- WebGL
- 字体
- 分辨率
- 时区
- GPU特征

结论：

```text
换IP不等于匿名
```

---

# IP 与代理策略

常见问题：

- 免费代理不可用
- 机房IP容易风控
- 住宅IP更稳定

---

# 分层策略

## 小规模

单 IP + 限速

---

## 中规模

代理池轮换

---

## 大规模

真实浏览器 + 行为模拟

---

# Playwright 的关键优势

相比 Selenium：

- 更稳定
- 更现代 API
- 自动等待机制
- 多上下文支持

---

# 对爬虫认知的变化

早期认知：

```text
爬虫 = requests
```

后期认知：

```text
爬虫 = 浏览器行为模拟系统
```

---

# 现代反爬的本质

从阻止脚本执行，变成：

```text
识别非人类行为
```

---

# 最终结论

真正稳定的策略不是对抗规则，而是：

```text
尽可能贴近真实用户行为
```

包括：

- 真实浏览器环境
- 合理访问频率
- 登录态复用
- 行为自然化

---

# 核心能力迁移

最终能力不在代码层面，而在：

```text
理解 Web 系统如何运行
```

包括：

- HTTP
- Cookie / Session
- JS逻辑
- 浏览器机制
- 风控体系
- 前端行为

---

# 终点认知

所有请求最终都会落到：

```text
Network
```

而真正的信息，也全部在那里。
