---
title: 学生选课系统开发记录
date: 2022-10-11
tags:
  - Java
  - SpringBoot
  - SpringMVC
  - MyBatisPlus
  - Vue3
  - MySQL
  - Nginx
categories:
  - 项目实践
description: 一次基于 SpringMVC + MyBatisPlus + Vue3 的完整课程设计实践，从数据库设计、后端接口、前端交互到 Nginx 部署，对学生选课系统开发过程的完整整理。
---

# 学生选课系统开发记录

这是第一次独立完成一个完整的前后端分离系统。

当时目标很明确：

# 复刻学校选课系统

但原系统存在明显问题：

- UI 老旧
- 操作卡顿
- 高峰期响应慢
- 功能体验不一致

因此决定自行实现一套替代方案。

---

# 项目目标

核心需求非常直接：

- 学生登录
- 课程列表展示
- 学生选课 / 退课
- 教师课程管理
- 基本数据统计

本质是一个典型：

```text
CRUD + 关系建模系统
```

````

---

# 技术栈

## 后端

| 技术        | 作用     |
| ----------- | -------- |
| SpringMVC   | Web接口  |
| MyBatisPlus | ORM层    |
| MySQL       | 数据库   |
| Maven       | 依赖管理 |
| Tomcat      | 运行容器 |

---

## 前端

| 技术         | 作用     |
| ------------ | -------- |
| Vue3         | 前端框架 |
| TypeScript   | 类型约束 |
| Vite         | 构建工具 |
| Element Plus | UI组件库 |
| Axios        | HTTP请求 |

---

## 部署

| 技术  | 作用              |
| ----- | ----------------- |
| Nginx | 反向代理/静态托管 |
| Linux | 运行环境          |

---

# 系统架构

整体流程如下：

```text
Vue前端
   ↓
Axios请求
   ↓
Nginx反向代理
   ↓
SpringMVC Controller
   ↓
Service业务层
   ↓
MyBatisPlus
   ↓
MySQL
```

典型的前后端分离架构。

---

# 数据库设计

项目中第一次完整接触关系型建模。

---

## 学生表

```sql
CREATE TABLE student (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    student_no VARCHAR(20),
    username VARCHAR(50),
    password VARCHAR(100),
    major VARCHAR(100)
);
```

---

## 课程表

```sql
CREATE TABLE course (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    course_name VARCHAR(100),
    teacher_name VARCHAR(100),
    max_count INT,
    selected_count INT
);
```

---

## 学生选课关系表

多对多关系必须拆分：

```sql
CREATE TABLE student_course (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    student_id BIGINT,
    course_id BIGINT,
    create_time DATETIME
);
```

这里是第一次明确理解：

# 多对多必须依赖中间表

---

# 后端结构设计

标准分层结构：

```text
controller
service
mapper
entity
dto
config
utils
common
```

---

# Controller层

负责请求接入：

```java
@RestController
@RequestMapping("/course")
public class CourseController {

    @Autowired
    private CourseService courseService;

    @PostMapping("/select")
    public Result selectCourse(Long courseId){
        return courseService.select(courseId);
    }
}
```

---

# Service层（业务核心）

```java
public Result select(Long courseId){

    Course course = courseMapper.selectById(courseId);

    if(course.getSelectedCount() >= course.getMaxCount()){
        return Result.fail("课程人数已满");
    }

    course.setSelectedCount(course.getSelectedCount() + 1);

    courseMapper.updateById(course);

    return Result.ok();
}
```

---

# 并发问题（核心缺陷）

这里存在典型问题：

```text
同时多个请求读取同一库存
```

示例：

```text
A读到99
B读到99

A写100
B写100
```

结果：

```text
实际变成101（超卖）
```

---

# 并发问题本质

这是典型：

```text
竞态条件（Race Condition）
```

---

# 可选优化方案（后续扩展）

## Redis预扣减

```text
先扣Redis
再写数据库
```

---

## MQ削峰

```text
请求 → MQ → 消费
```

---

## 乐观锁

```sql
WHERE version = ?
```

防止覆盖写。

---

# MyBatisPlus使用体验

初期最大的变化：

```text
从 JDBC → ORM
```

例如：

```java
courseMapper.selectById(id);
```

但也暴露问题：

- 复杂SQL仍需手写
- JOIN能力有限
- 性能需控制

---

# 前端实现

## Vue3 + Element Plus

快速构建管理界面：

```vue
<el-table :data="courseList">
  <el-table-column prop="courseName" label="课程名" />
</el-table>
```

---

## Vite构建体验

相比Webpack：

```text
启动速度明显提升
```

---

## Axios请求封装

```ts
export function selectCourse(courseId: number) {
  return axios.post("/api/course/select", {
    courseId,
  });
}
```

---

# 登录与认证

初期实现较简单：

- 基础登录
- 简单Token

缺乏：

- 权限体系
- RBAC模型
- 会话管理

---

# JWT设计问题

理想结构：

| 数据        | 存储  |
| ----------- | ----- |
| JWT         | 前端  |
| Session状态 | Redis |

用于：

- 登录失效控制
- 踢人机制
- 分布式支持

---

# Nginx部署

第一次完整接触上线流程。

---

## 核心配置

```nginx
server {

    listen 80;

    location / {
        root html;
        index index.html;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8080/;
    }
}
```

---

# 系统上线流程理解

第一次形成完整认知：

```text
代码运行 ≠ 系统上线
```

上线还包括：

- Linux环境
- Nginx代理
- 数据库部署
- 静态资源托管
- 端口与域名配置

---

# 项目主要问题

## 1. 并发控制缺失

选课核心问题未解决。

---

## 2. 数据库设计不完善

- 索引不足
- 约束不严格
- 冗余字段存在

---

## 3. 缓存缺失

所有请求直接打数据库。

---

## 4. 前端状态管理弱

未使用成熟状态管理方案。

---

## 5. 异常处理分散

缺少统一错误体系。

---

# 项目价值

虽然存在明显不足，但完成了完整闭环：

```text
数据库 → 后端 → 前端 → 部署
```

这是第一次真正意义上的：

# 全栈系统实践

---

# 后续可扩展方向

| 模块   | 优化方向    |
| ------ | ----------- |
| 登录   | Redis + JWT |
| 选课   | MQ + Redis  |
| 数据库 | 索引优化    |
| API    | 限流机制    |
| 前端   | 状态管理    |
| 部署   | Docker化    |
| 架构   | 微服务化    |

---

# 总结

该项目的意义不在复杂度，而在于完整性。

第一次把一个系统从：

```text
设计 → 实现 → 部署
```

完整跑通，建立了对工程系统的基础认知。
````
