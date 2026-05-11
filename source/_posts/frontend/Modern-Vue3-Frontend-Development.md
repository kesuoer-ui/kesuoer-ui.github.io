---
title: Vue3、Element Plus 与现代前端开发体验
date: 2022-03-12
category:
  - 前端开发
tag:
  - Vue3
  - Element Plus
  - TailwindCSS
  - Vite
---

````

# Vue3、Element Plus 与现代前端开发体验

这篇更多是我在从 Vue2 过渡到 Vue3 过程中，对整个前端开发方式变化的一些记录。

当时的感觉比较明显：前端不再是“写页面”，而是在往工程化应用靠。

---

# Vue3 变化最直观的地方

如果用一句话概括 Vue3 的变化，其实不是性能，而是写代码的方式变了。

Vue2 时代的组件，大致是这样拆的：

```js
data
methods
computed
watch
````

逻辑天然是“按类型分区”的。

项目稍微一大，就会出现一个问题：

同一个功能的代码，被拆散在不同位置。

---

# Vue2 的问题：逻辑分散

比如一个用户模块：

```js
data() {
  return {
    user: {},
    loading: false
  }
},
methods: {
  async getUser() {}
},
watch: {
  user() {}
}
```

当功能复杂后，这种结构的问题会变得很明显：

- 你要改一个功能，需要在多个区块之间跳
- 相关逻辑无法集中阅读
- 复用逻辑很难抽离

当时我在做稍微大一点的页面时，这种割裂感很强。

---

# Vue3 的变化：按“功能”组织代码

Vue3 后，逻辑基本可以集中写在一起：

```ts
const user = ref({});
const loading = ref(false);

const getUser = async () => {
  loading.value = true;
};
```

我当时最直观的感受是：

> 代码终于是“按业务块”组织的，而不是“按语法分类”。

---

# setup 的意义

Vue3 的核心变化其实集中在 `setup`。

或者更极端一点：

```vue
<script setup>
```

这个语法糖之后，组件基本变成：

> 一个函数 + 一段模板

```vue
<script setup lang="ts">
import { ref } from "vue";

const count = ref(0);

const add = () => {
  count.value++;
};
</script>

<template>
  <button @click="add">
    {{ count }}
  </button>
</template>
```

---

# ref / reactive 的理解成本

Vue3 初期最容易卡的点就是这一块。

大致可以粗暴理解为：

- `ref`：基础值（number/string/boolean）
- `reactive`：对象

```ts
const count = ref(0);

const user = reactive({
  name: "test",
});
```

后面那个 `.value` 确实一开始很烦，但实际写多了之后会习惯。

---

# Vue3 + TypeScript 变成默认组合

我后来越来越明显的感受是：

Vue3 不单是框架升级，而是把 TS 强行带进主流开发流程。

问题主要来自 Vue2：

- 类型约束弱
- 重构风险高
- 接口字段容易错
- 大项目维护成本上升

所以 Vue3 + TS 基本变成默认组合。

---

# Vite 带来的体验变化

在 Vite 出来之前，Vue CLI + Webpack 是主流，但体验问题比较明显：

- 启动慢
- 热更新慢
- 配置复杂

Vite 的变化本质是：

> 不再从打包角度思考开发阶段，而是直接利用 ESM。

开发体验上最明显的变化是：

- 启动基本秒级
- HMR 很快
- 配置明显变少

---

# 一个标准 Vue3 项目结构

后来看比较成熟的项目，大致都会变成这样：

```text
src
├── api
├── components
├── composables
├── layouts
├── router
├── stores
├── styles
├── utils
├── views
└── App.vue
```

这里我当时印象比较深的一点是：

> 项目结构越清晰，后期“乱写工具函数”的概率越低。

---

# composables 的实际作用

Vue3 很重要的一点是 composables（类似 hooks）。

比如：

```ts
export function useLoading() {
  const loading = ref(false);

  const start = () => (loading.value = true);
  const stop = () => (loading.value = false);

  return { loading, start, stop };
}
```

我后期基本开始把：

- 请求逻辑
- 状态逻辑
- UI 状态

都往 composable 拆。

本质是把“功能”抽成模块，而不是把“页面”当模块。

---

# Element Plus 的实际使用体验

Element Plus 在后台系统里基本是绕不开的。

它解决的问题很直接：

> 直接提供一套可用的 UI 组件体系。

比如表单：

```vue
<el-form>
  <el-input v-model="username" />
  <el-button type="primary">
    登录
  </el-button>
</el-form>
```

开发速度确实会快很多。

---

# Element Plus 的真实问题

用了一段时间后比较明显的几个问题：

### 1. UI 风格固定

很容易一眼看出“Element UI 项目”。

---

### 2. 深层样式修改成本高

组件嵌套多之后：

- 改样式需要层层覆盖
- CSS 优先级容易变复杂

---

# Ant Design Vue 的对比

后面也接触过 Ant Design Vue，整体更偏企业规范体系：

- 表格能力更强
- 设计规范更严格
- 更偏中后台复杂系统

---

# TailwindCSS 的转变感受

传统 CSS 最大的问题是：

> 项目一大，命名和结构会开始失控。

比如：

```css
.user-card-header-wrapper
```

这种命名会越来越多。

---

# Tailwind 的方式

Tailwind 变成：

```html
<div class="flex items-center p-4 rounded-xl"></div>
```

当时的感受是：

> 样式不再“写在文件里”，而是“写在结构里”。

---

# Tailwind 的优点

- 不需要命名 CSS
- 组合速度快
- 很适合组件化开发

---

# Tailwind 的缺点

- class 会变长
- 可读性依赖习惯
- 初期不适应会觉得乱

---

# Pinia 状态管理的变化

Vuex 在 Vue2 时代基本是标配，但后来逐渐被 Pinia 替代。

原因很直接：

> 写法更接近 Composition API。

```ts
export const useUserStore = defineStore("user", {
  state: () => ({
    token: "",
  }),
});
```

整体更轻。

---

# 前端真正复杂的地方

当时做完一段时间后，我的感受比较明确：

前端难点其实不在页面，而在这些地方：

- 状态同步
- 组件通信
- 请求封装
- 权限控制
- 性能优化
- 项目结构

页面只是表层。

---

# 请求层的统一封装

项目稍微复杂一点后，Axios 基本都会抽一层：

```ts
import axios from "axios";

const request = axios.create({
  baseURL: "/api",
});
```

统一处理：

- token
- 错误处理
- loading
- 超时

否则后期一定会散。

---

# 一点更底层的感受

回头看 Vue3 这一代工具链，我当时最大的变化不是“会不会用”，而是：

> 开始用工程的方式看前端，而不是页面的方式。

---

# 最后

Vue3 本身并没有改变“写页面”这件事，但它改变了组织代码的方式。

再加上：

- Vite
- Pinia
- Tailwind
- Element Plus

这些工具组合起来之后，前端逐渐变成：

> 一个工程系统，而不是 UI 拼装过程。
