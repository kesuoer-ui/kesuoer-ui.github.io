---
title: 从 Scrapy 到 Playwright：现代 Python 爬虫技术整理
date: 2022-09-18
category:
  - 后端开发
  - Python
tag:
  - 爬虫
  - Scrapy
  - Selenium
  - Playwright
---

# 从 Scrapy 到 Playwright：现代 Python 爬虫技术整理

很多人第一次接触 Python。

几乎都会接触：

```text
爬虫
```

因为：

Python 在这个领域生态太强了。

而我自己对爬虫技术的认知变化。

其实也是：

> 从“请求脚本小子”
>
> 到真正理解现代 Web 自动化。

---

# 最开始：requests + BeautifulSoup

很多人入门爬虫。

基本都从：

```python
requests
BeautifulSoup
```

开始。

---

# requests：最经典 HTTP 库

安装：

```bash
pip install requests
```

---

# 最简单请求

```python
import requests

resp = requests.get("https://example.com")

print(resp.text)
```

---

# BeautifulSoup：HTML 解析

安装：

```bash
pip install beautifulsoup4
```

---

# 最简单解析

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(resp.text, "html.parser")

title = soup.title.text
```

---

# 这阶段本质是什么

其实：

```text
伪装浏览器发送HTTP请求
```

然后：

```text
解析返回HTML
```

---

# 为什么早期爬虫很好做

因为：

以前很多网站：

```text
服务端直接返回完整HTML
```

---

# 浏览器拿到源码后

直接：

```text
就有内容
```

于是：

```python
requests.get()
```

就够了。

---

# 这阶段最常见技术

---

# 请求头伪装

```python
headers = {
    "User-Agent": "Mozilla/5.0"
}
```

---

# Cookie

很多网站：

登录状态依赖 Cookie。

---

# Session

```python
session = requests.Session()
```

自动维持会话。

---

# POST 请求

```python
requests.post(url, data=data)
```

---

# JSON 接口

```python
resp.json()
```

---

# 很多人后来才发现

大量网站：

其实：

```text
前端页面只是壳
```

真正数据：

来自：

```text
XHR / Fetch API
```

---

# 所以后来爬虫开始进入第二阶段

即：

> “抓接口”。

---

# 接口爬虫时代

这时候：

开发者工具开始变得极其重要。

---

# Chrome DevTools

按：

```text
F12
```

进入：

```text
Network
```

---

# 真正关键的是

看：

```text
XHR
Fetch
```

---

# 因为很多页面：

本质：

```text
前端调API
```

---

# 例如：

```text
GET /api/list?page=1
```

---

# 所以后来很多爬虫

其实根本不解析 HTML。

而是：

```text
直接请求后端API
```

---

# 这阶段最大的提升

是：

```text
速度极快
```

因为：

不用：

- 渲染页面
- 执行JS
- 加载浏览器

---

# 但问题也来了

越来越多网站：

开始：

```text
前端动态渲染
```

---

# 什么叫动态网页

例如：

Vue / React 页面。

初始 HTML：

可能只有：

```html
<div id="app"></div>
```

真正内容：

靠：

```text
JavaScript运行后生成
```

---

# 于是 requests 开始失效

因为：

```python
requests.get()
```

拿到的：

只是：

```text
未渲染HTML
```

---

# 所以后来 Selenium 开始流行

---

# Selenium 是什么

本质：

```text
浏览器自动化工具
```

---

# 安装

```bash
pip install selenium
```

---

# Selenium 核心思想

不是：

```text
模拟HTTP请求
```

而是：

```text
真正启动浏览器
```

---

# 最简单例子

```python
from selenium import webdriver

driver = webdriver.Chrome()

driver.get("https://example.com")

print(driver.page_source)
```

---

# Selenium 真正强大的地方

是：

```text
它会执行JS
```

---

# 所以：

动态网页终于能爬了。

---

# Selenium 能做什么

不仅：

- 获取HTML

还可以：

- 点击按钮
- 输入文本
- 登录
- 上传文件
- 模拟用户行为

---

# 定位元素

```python
driver.find_element(By.ID, "username")
```

---

# 点击按钮

```python
button.click()
```

---

# 输入文本

```python
input.send_keys("hello")
```

---

# Selenium 为什么当年统治爬虫圈

因为：

它第一次真正解决了：

```text
动态网页问题
```

---

# 但 Selenium 问题也很明显

---

# 1. 太重

因为：

```text
真浏览器
```

资源占用巨大。

---

# 2. 很慢

尤其：

批量并发。

---

# 3. WebDriver 地狱

这是经典噩梦。

例如：

```text
Chrome版本不匹配
```

---

# 常见错误

```text
session not created
```

---

# 原因通常：

```text
chromedriver版本不对
```

---

# 后来 webdriver-manager 才缓解一点

```bash
pip install webdriver-manager
```

---

# 4. 异步页面等待困难

经典问题：

```text
元素还没加载出来
```

---

# 于是：

大量：

```python
time.sleep(5)
```

开始出现。

---

# 后来 Selenium 进入现代化阶段

出现：

```python
WebDriverWait
```

---

# 显式等待

```python
from selenium.webdriver.support.ui import WebDriverWait
```

---

# Scrapy：真正的工程化爬虫框架

后来接触 Scrapy 时。

第一次意识到：

> 原来爬虫也能工程化。

---

# Scrapy 是什么

本质：

```text
异步爬虫框架
```

---

# 安装

```bash
pip install scrapy
```

---

# 创建项目

```bash
scrapy startproject myspider
```

---

# 运行爬虫

```bash
scrapy crawl demo
```

---

# Scrapy 最大特点

不是：

```text
会不会发请求
```

而是：

```text
完整工程体系
```

---

# Scrapy 包含：

- 调度器
- 请求队列
- 中间件
- Pipeline
- 去重
- 并发控制

---

# Spider

负责：

```text
定义爬取逻辑
```

---

# Pipeline

负责：

```text
处理数据
```

例如：

- 存数据库
- 清洗
- 导出

---

# Middleware

负责：

```text
请求拦截
```

例如：

- 代理
- UA轮换
- Cookie处理

---

# Scrapy 最大优势

---

# 1. 性能极强

因为：

```text
Twisted异步架构
```

---

# 2. 非常适合大规模采集

例如：

- 新闻站
- 电商
- Wiki
- 文档站

---

# 3. 工程化成熟

很多公司：

内部爬虫平台：

其实就是：

```text
Scrapy二次封装
```

---

# Scrapy 最大缺点

就是：

```text
动态网页支持弱
```

---

# 所以后来大量项目

变成：

```text
Scrapy + Selenium
```

---

# 即：

Scrapy负责调度。

Selenium负责渲染。

---

# 但后来问题越来越明显

现代网站：

JS 越来越重。

---

# React/Vue SPA

开始普及。

---

# Cloudflare

开始流行。

---

# 各种反爬升级

包括：

- 指纹检测
- webdriver检测
- canvas检测
- 行为分析

---

# Selenium 开始越来越吃力

于是：

Playwright 出现了。

---

# Playwright：现代浏览器自动化

这是后来我认为：

真正改变 Python Web 自动化体验的工具。

---

# Playwright 是什么

微软推出的：

```text
现代浏览器自动化框架
```

---

# 安装

```bash
pip install playwright
playwright install
```

---

# 最简单例子

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()

    page = browser.new_page()

    page.goto("https://example.com")

    print(page.content())
```

---

# Playwright 为什么体验飞跃

因为：

它本质：

已经不是：

```text
传统WebDriver体系
```

了。

---

# 最大变化

是：

```text
直接控制浏览器协议
```

---

# Playwright 最强的地方

---

# 1. 自动等待

这是巨大提升。

以前 Selenium：

```python
time.sleep()
```

满天飞。

---

# Playwright：

很多操作：

自动等待元素加载。

---

# 例如：

```python
page.click("button")
```

内部：

已经自动等待。

---

# 2. 更稳定

尤其：

动态页面。

---

# 3. 更现代

天然适配：

- SPA
- React
- Vue
- WebSocket

---

# 4. 多浏览器支持极好

支持：

- Chromium
- Firefox
- WebKit

---

# 5. API 非常统一

开发体验明显比 Selenium 好。

---

# Playwright 另一个重要用途

其实是：

```text
网页截图
```

---

# 截图

```python
page.screenshot(path="test.png")
```

---

# 后来很多 Bot 项目

其实：

都是：

```text
网页渲染 -> 截图 -> 发图片
```

---

# Playwright 最大问题

---

# 1. 浏览器依赖重

首次安装很大。

---

# 2. Linux 部署复杂

尤其 Docker。

经常：

```text
缺依赖
```

---

# 常见问题

```text
browser closed unexpectedly
```

---

# 3. 并发内存高

浏览器非常吃内存。

---

# 所以后来大型项目

通常会做：

```text
浏览器池
```

---

# Playwright 为什么越来越重要

因为：

现代 Web：

已经逐渐：

```text
浏览器化
```

---

# 很多内容：

甚至：

```text
只有浏览器执行JS后才存在
```

---

# 所以：

现在越来越多“爬虫”。

本质已经变成：

> Web 自动化。

---

# 现代爬虫技术路线变化

---

# 第一阶段

```text
requests + bs4
```

静态网页时代。

---

# 第二阶段

```text
抓XHR接口
```

接口分析时代。

---

# 第三阶段

```text
Selenium
```

动态网页时代。

---

# 第四阶段

```text
Playwright
```

现代浏览器自动化时代。

---

# 现在很多“爬虫”

其实已经接近：

```text
自动化QA
RPA
浏览器机器人
```

了。

---

# 反爬也是越来越复杂

现在常见：

---

# 1. IP限制

需要：

- 代理
- IP池

---

# 2. 指纹检测

例如：

- webdriver
- canvas
- WebGL

---

# 3. 行为检测

例如：

```text
鼠标轨迹
滚动行为
点击间隔
```

---

# 4. 登录验证

例如：

- 验证码
- 风控
- 二次验证

---

# 所以后来我最大的认知变化

以前觉得：

```text
爬虫 = requests.get()
```

后来发现：

现代爬虫本质：

已经是：

> “模拟真实用户行为”。

---

# 最后总结

---

# requests + bs4

适合：

```text
简单静态网页
```

---

# Scrapy

适合：

```text
大规模工程化采集
```

---

# Selenium

适合：

```text
传统动态网页
```

---

# Playwright

适合：

```text
现代Web自动化
复杂SPA网站
```

---

# 而现代爬虫技术的发展路线

本质上其实是：

```text
从HTTP请求
逐渐走向浏览器控制
```

因为：

现代互联网本身。

已经越来越依赖：

- JavaScript
- 前端渲染
- 浏览器环境

而不是：

传统静态 HTML 了。
