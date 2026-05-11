---
title: Python 生成图片的几种方案对比
date: 2021-10-26
category:
  - 后端开发
  - Python
tag:
  - PIL
  - OpenCV
  - matplotlib
  - HTML Render
  - 图像处理
---

# Python 生成图片的几种方案对比

Python 里。

“生成图片” 其实是一个非常大的需求。

例如：

- Bot 生成信息图
- 数据报表
- 海报
- OCR 预处理
- AI 图片处理
- 自动截图
- 可视化
- 动态排行榜

后来做项目时。

会发现：

不同方案差异非常大。

甚至：

> 根本不是同一种思路。

---

# Python 常见生成图片方案

目前主流大概有几类：

| 方案        | 方向         |
| ----------- | ------------ |
| PIL/Pillow  | 传统绘图     |
| OpenCV      | 图像处理     |
| matplotlib  | 数据可视化   |
| HTML Render | 网页渲染截图 |

---

# 这些方案最大的区别

其实是：

```text
谁负责布局
```

---

# PIL

你自己计算坐标。

---

# OpenCV

偏图像矩阵处理。

---

# matplotlib

偏数据图表。

---

# HTML Render

直接交给浏览器排版。

---

# PIL / Pillow：最经典的方案

---

# Pillow 是什么

Pillow：

实际上是：

```text
PIL 的现代维护版
```

现在基本：

```python
from PIL import Image
```

都是 Pillow。

---

# 安装

```bash
pip install pillow
```

---

# 最简单生成图片

```python
from PIL import Image

img = Image.new("RGB", (500, 300), "white")
img.save("test.png")
```

---

# Pillow 最核心的能力

包括：

- 创建图片
- 绘图
- 写文字
- 拼接图片
- 加载字体
- 裁剪缩放
- GIF处理

---

# 绘图

```python
from PIL import ImageDraw

draw = ImageDraw.Draw(img)

draw.rectangle((10, 10, 100, 100), fill="red")
```

---

# 写文字

```python
from PIL import ImageFont

font = ImageFont.truetype("msyh.ttc", 32)

draw.text((50, 50), "Hello", font=font)
```

---

# Pillow 最大用途

我后来项目里最常见用途：

其实是：

```text
信息图生成
```

例如：

- Bot 菜单
- 排行榜
- 状态卡片
- 用户信息图

---

# Pillow 最大优点

---

# 1. 非常轻量

依赖少。

启动快。

---

# 2. 纯 Python 生态兼容好

几乎所有 Python 项目都能直接接。

---

# 3. 对 Bot 非常友好

尤其：

- QQ Bot
- Telegram Bot
- Discord Bot

---

# Pillow 最大缺点

就是：

```text
布局全靠手算
```

---

# 最痛苦的问题

例如：

```text
文字居中
自动换行
动态高度
自适应布局
```

全部要自己算。

---

# 后来项目复杂后会发现

Pillow 很容易变成：

```text
坐标地狱
```

---

# 中文字体是经典坑

很多新人第一次都会遇到：

```text
中文乱码
```

原因：

默认字体不支持中文。

---

# 常见解决方案

Windows：

```python
msyh.ttc
```

Linux：

需要安装：

```text
Noto Sans CJK
```

---

# Linux 字体坑非常多

尤其 Docker。

经常：

```text
本地正常
服务器乱码
```

原因通常是：

```text
服务器没字体
```

---

# Pillow 性能问题

Pillow 本身不算慢。

但：

大量：

- 抗锯齿
- 超大图
- 复杂阴影

后。

CPU 开销明显。

---

# OpenCV：完全不同的世界

---

# OpenCV 是什么

OpenCV：

本质：

```text
计算机视觉库
```

不是专门做 UI 绘图的。

---

# 安装

```bash
pip install opencv-python
```

---

# OpenCV 最大特点

它处理的不是：

```text
图片对象
```

而是：

```text
矩阵
```

---

# 读取图片

```python
import cv2

img = cv2.imread("test.png")
```

---

# OpenCV 本质

其实：

```text
numpy数组
```

---

# 例如：

```python
img.shape
```

输出：

```text
(height, width, channel)
```

---

# OpenCV 擅长什么

特别适合：

- 图像识别
- OCR预处理
- 边缘检测
- 视频处理
- 摄像头
- AI视觉

---

# OpenCV 常见操作

---

# 缩放

```python
cv2.resize(img, (500, 300))
```

---

# 高斯模糊

```python
cv2.GaussianBlur(img, (5,5), 0)
```

---

# 灰度化

```python
cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
```

---

# OpenCV 最大问题

它：

```text
不适合做复杂UI
```

---

# 为什么

因为：

所有东西：

还是：

```text
手工坐标
```

而且：

文字系统非常弱。

---

# 中文更是经典大坑

OpenCV 默认：

```text
几乎不支持中文绘制
```

很多人最后：

还是：

```text
Pillow + OpenCV混合
```

---

# 一个经典组合

例如：

```text
OpenCV处理图像
Pillow负责中文文字
```

---

# OpenCV 与 Pillow 的颜色坑

这是非常经典的问题。

---

# Pillow

默认：

```text
RGB
```

---

# OpenCV

默认：

```text
BGR
```

---

# 所以很多人第一次转换时

图片会：

```text
颜色错乱
```

---

# 转换方式

```python
cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
```

---

# matplotlib：数据可视化路线

---

# matplotlib 是什么

本质：

```text
科学计算绘图库
```

---

# 安装

```bash
pip install matplotlib
```

---

# 最经典用途

例如：

- 折线图
- 柱状图
- 饼图
- 热力图
- 数据分析

---

# 最简单例子

```python
import matplotlib.pyplot as plt

plt.plot([1,2,3], [4,5,6])

plt.savefig("chart.png")
```

---

# matplotlib 最大特点

是：

```text
图表生态极强
```

尤其：

- numpy
- pandas
- jupyter

配套非常成熟。

---

# 很多数据分析项目

本质：

就是：

```text
pandas + matplotlib
```

---

# matplotlib 最大缺点

其实也明显。

---

# 1. 不适合复杂UI

它本质：

不是 UI 引擎。

---

# 2. 中文字体坑很多

尤其 Linux。

经常：

```text
方块字
乱码
```

---

# 解决方式

指定字体：

```python
plt.rcParams['font.sans-serif'] = ['SimHei']
```

---

# 3. 样式不现代

默认图表：

```text
一眼学术风
```

很多人后期：

会转：

- plotly
- echarts
- seaborn

---

# HTML Render：后期越来越主流

这是我后来：

越来越喜欢的方案。

---

# 为什么

因为：

```text
浏览器本身就是最强布局引擎
```

---

# 核心思路

```text
HTML + CSS
-> 浏览器渲染
-> 截图
```

---

# 常见方案

例如：

- Playwright
- Selenium
- Puppeteer
- pyppeteer

---

# Playwright 安装

```bash
pip install playwright
playwright install
```

---

# 最简单截图

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()

    page = browser.new_page()

    page.set_content("""
    <h1>Hello</h1>
    """)

    page.screenshot(path="test.png")
```

---

# HTML Render 最大优势

---

# 1. 天然现代 UI

因为：

直接：

```text
CSS布局
```

---

# 2. 不需要手算坐标

终于不用：

```text
x + 20
y + 50
```

了。

---

# 3. 支持复杂布局

例如：

- flex
- grid
- 阴影
- 渐变
- 动画

---

# 4. 前端生态直接复用

甚至：

```text
Vue页面
React页面
```

都能直接截图。

---

# 这对现代 Bot 非常重要

因为很多：

高质量 Bot 图片。

本质：

已经是：

```text
网页截图
```

---

# HTML Render 最大问题

---

# 1. 重

浏览器非常重。

---

# 2. 部署复杂

尤其 Linux。

经常：

- 缺依赖
- 缺字体
- Chromium启动失败

---

# 3. Docker 坑很多

例如：

```text
no sandbox
```

---

# 常见启动参数

```python
browser = p.chromium.launch(
    args=["--no-sandbox"]
)
```

---

# 4. 字体问题依然存在

服务器：

还是可能：

```text
没中文字体
```

---

# 5. 并发压力大

浏览器实例：

内存占用非常高。

---

# 后来很多项目的做法

是：

```text
浏览器池
```

而不是：

每次启动 Chromium。

---

# HTML Render 最适合什么

特别适合：

- 动态卡片
- 排行榜
- 信息页
- 海报
- AI结果展示
- 复杂UI生成

---

# 各方案本质区别

| 方案        | 本质         |
| ----------- | ------------ |
| Pillow      | 手工绘图     |
| OpenCV      | 图像矩阵处理 |
| matplotlib  | 数据图表     |
| HTML Render | 浏览器渲染   |

---

# 我后来的实际选择

---

# 简单图片生成

用：

```text
Pillow
```

---

# OCR/视觉处理

用：

```text
OpenCV
```

---

# 数据分析图

用：

```text
matplotlib
```

---

# 高质量动态UI

用：

```text
HTML Render
```

---

# 现在很多现代项目

其实已经逐渐：

```text
前端化
```

---

# 即：

```text
前端负责页面
Python负责渲染截图
```

---

# 所以很多后期复杂 Bot

架构会变成：

```text
Vue/React
-> Playwright截图
-> Bot发送图片
```

---

# 这其实已经不是“画图”

而是：

> 用浏览器作为图片生成引擎。

---

# 最后一个重要问题

很多人会问：

```text
为什么不用纯前端Canvas
```

---

# 实际上

很多项目：

就是这么做的。

但：

Python 服务端：

更适合：

- 自动化
- 批量生成
- Bot
- 后台任务

---

# 最后总结

---

# Pillow

适合：

```text
轻量绘图
信息图
Bot图片
```

---

# OpenCV

适合：

```text
视觉处理
OCR
图像算法
```

---

# matplotlib

适合：

```text
统计图
数据分析
科研图表
```

---

# HTML Render

适合：

```text
现代UI
复杂布局
高质量动态页面
```

---

# Python 图片生成生态

本质上其实已经分成：

```text
传统绘图流派
浏览器渲染流派
```

而现在越来越多复杂项目。

其实都在逐渐：

> “前端化”。

因为：

浏览器本身。

已经是现代世界最成熟的渲染引擎之一。
