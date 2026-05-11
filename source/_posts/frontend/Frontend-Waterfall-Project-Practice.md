---
title: 从前后端分离到瀑布流页面：我的 Web 开发实践
date: 2022-08-17
category:
  - 前端开发
tag:
  - Vue3
  - 瀑布流
  - 虚拟滚动
  - 无限加载
---

# 从前后端分离到瀑布流页面：我的 Web 开发实践

这个项目本质上是一个偏 Pinterest / Pixiv 风格的图片社区。

一开始目标其实很简单，就是做一个“能看图的网站”。但做着做着，功能不断往里加：

- 用户登录与权限
- 图片上传与管理
- 评论与互动
- 标签与搜索
- 收藏与点赞
- 推荐流
- 后台管理

到后面已经不是 Demo，更像一个完整的内容社区雏形。

也是从这个项目开始，我明显感觉到：

> Web 开发真正难的地方，从来不是“写页面”，而是控制复杂度。

---

# 一、技术栈与整体结构

整体是典型前后端分离架构：

```text
Vue3 + Vite
        ↓
Axios API
        ↓
FastAPI
        ↓
PostgreSQL
```

技术选型大致如下：

| 模块     | 技术                   |
| -------- | ---------------------- |
| 前端     | Vue3                   |
| UI       | Element Plus           |
| 状态管理 | Pinia                  |
| 路由     | Vue Router             |
| 请求     | Axios                  |
| 后端     | FastAPI                |
| ORM      | Gino                   |
| 数据库   | PostgreSQL             |
| 鉴权     | JWT                    |
| 瀑布流   | 自实现 Virtual Masonry |

---

# 二、这个项目是怎么“变复杂”的

最开始只是想做一个图片展示页。

后来逐步加功能：

- 登录
- 点赞
- 评论
- 标签
- 搜索
- 推荐
- 管理后台

问题也随之出现：

功能不是独立增长的，而是互相开始耦合。

比如：

- 点赞影响推荐
- 评论影响排序
- 标签影响搜索
- 用户行为影响首页流

这个时候才开始意识到：

> 功能堆叠不是问题，功能之间的关系才是问题。

---

# 三、项目结构（当时的整理方式）

前端结构大致如下：

```text
frontend
├── api
├── components
├── pages
├── stores
├── router
├── utils
└── views
```

后端：

```text
backend
├── router
│   ├── auth
│   ├── image
│   ├── comment
│   └── user
├── model
├── service
├── middleware
├── config
└── utils
```

后来回头看，这一阶段最大的问题其实不是技术，而是：

目录结构没有“约束力”，后期很容易变成混乱堆文件。

比如：

```text
utils.py
utils2.py
utils_final.py
utils_new.py
```

这是典型的失控前兆。

---

# 四、登录系统（JWT 实践）

这一块是标准的 JWT + 前后端分离方案。

流程比较直接：

```text
用户登录
  ↓
后端校验账号密码
  ↓
生成 JWT
  ↓
前端保存 token
  ↓
请求自动携带 token
```

FastAPI 登录接口大致如下：

```python
@router.post("/login")
async def login(data: LoginSchema):

    user = await User.query.where(
        User.account == data.account
    ).gino.first()

    if not user:
        raise Exception("用户不存在")

    if not verify_password(data.password, user.password):
        raise Exception("密码错误")

    token = create_access_token(user.id)

    return {
        "token": token
    }
```

前端统一封装 Axios：

```ts
const request = axios.create({
  baseURL: "/api",
  timeout: 10000,
});

request.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization = token;
  }

  return config;
});
```

这一块的体会比较直接：

> 前端项目一旦请求不统一管理，后期基本必炸。

---

# 五、瀑布流：真正的第一个性能坑

这个项目真正开始变“工程化”的地方，其实是瀑布流。

原因很简单：图片网站有几个天然问题：

- 图片高度不固定
- 数量巨大
- 无限滚动
- DOM 数量爆炸

一开始我用的是现成库：

- masonry-layout
- vue-masonry

问题很快出现：卡顿。

本质原因很简单：

> 页面 DOM 数量过多。

几千张图片直接挂在 DOM 上，浏览器完全吃不消。

---

# 六、虚拟瀑布流的实现

后来我自己做了一个 Virtual Masonry。

核心思路是：

> 不渲染所有内容，只渲染可视区域附近的数据。

滚动时只计算：

```ts
indices.value = groupManager.getCellIndices({
  height,
  width,
  x: scrollLeft,
  y: scrollTop,
});
```

本质就是：

当前视口应该看到哪些图片，就只渲染这些。

DOM 数量从几千直接降到几十。

这是第一次比较直观感受到：

> 前端性能优化，本质是减少浏览器的工作量。

---

# 七、瀑布流布局算法

布局逻辑很简单：

每次把新图片放到当前最短的列。

```text
列高度：
1200 | 800 | 1500 | 900

新图片 → 放到 800 那一列
```

这样整体会更均衡。

核心计算：

```ts
const column = computed(() => {
  let col = Math.floor(
    (height + props.gapWidth) / (props.gapWidth + props.itemWidth),
  );

  return props.minColumn > col ? props.minColumn : col;
});
```

用于动态控制列数，实现响应式布局。

---

# 八、滚动性能优化

如果直接监听 scroll，会变成性能灾难：

```text
scroll / scroll / scroll / scroll ...
```

所以引入 throttle：

```ts
export function throttle(fn, delay) {
  let valid = true;

  return function () {
    if (valid) {
      valid = false;

      setTimeout(() => {
        fn.apply(this, arguments);
        valid = true;
      }, delay);
    }
  };
}
```

这里的关键点是：

> 不是避免 scroll，而是控制 scroll 的计算频率。

---

# 九、IntersectionObserver 的替代方案

后来把“是否到底部”判断换成了：

```ts
intersectionObserver = new IntersectionObserver(onContainerScroll);
```

用浏览器原生能力判断元素是否进入视口。

相比 scroll 判断：

- 更稳定
- 更低开销
- 更不容易出 bug

---

# 十、ResizeObserver 解决布局问题

用于监听容器尺寸变化：

```ts
resizeObserver = new ResizeObserver(onContainerResized);
```

主要解决：

- 窗口缩放
- 布局变化
- 响应式重排

---

# 十一、图片才是真正的性能瓶颈

后面继续优化才发现：

真正拖垮性能的不是 Vue，而是图片本身。

问题包括：

- 原图过大（10MB+）
- 未压缩
- 未分级加载

后面做了几件事：

- 缩略图
- 懒加载
- WebP
- 占位图

页面才真正稳定下来。

---

# 十二、状态管理踩坑

随着功能增加，状态开始变复杂：

- 点赞
- 收藏
- 推荐
- 用户状态
- 页面缓存

最容易出现的问题是：

```text
一个地方更新，其他地方不同步
```

后来统一用 Pinia 管理用户状态：

```ts
export const useUserStore = defineStore("user", {
  state: () => ({
    token: "",
    userInfo: null,
  }),
});
```

---

# 十三、评论系统比想象复杂

原本以为只是“发评论”。

后来变成：

- 回复
- 嵌套
- 删除
- 敏感词
- 通知

复杂度直接上升一个量级。

---

# 十四、推荐系统的现实问题

推荐系统实现后才发现一个现实问题：

> 冷启动基本无解。

没有行为数据时：

- 无法推荐
- 无法排序
- 无法建模

只能做兜底策略。

---

# 十五、这个项目带来的真正变化

这个项目真正的收获不是某个技术点，而是：

我开始理解“工程复杂度”本身。

功能不是难点，难点是：

- 功能之间的影响
- 状态同步
- 性能约束
- 数据流控制

---

# 最后总结

回头看，这个项目更像一次完整的 Web 工程入门：

从“写页面”逐渐变成理解：

- 前端性能模型
- 状态管理
- 数据流设计
- 浏览器渲染成本
- 工程拆分方式

也从这里开始，我才逐渐意识到：

> Web 开发的本质，其实更接近客户端工程，而不是单纯的页面开发。

```

```
