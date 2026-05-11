---
title: 从 C 到 Java：我对面向对象的理解变化
date: 2022-11-18
tags:
  - OOP
  - 面向对象
  - C语言
  - 软件设计
categories:
  - 后端开发
  - Java
description: 从 C 语言过程式开发到 Java 面向对象开发，对封装、继承、多态、接口、设计模式以及工程化开发思维的一次系统性理解整理。
---

# 从 C 到 Java：我对面向对象的理解变化

很多人第一次学 Java 时，会觉得：

> “不就是把函数塞进 class 里吗？”

但真正开始做项目之后会发现：

Java 和 C 的区别，从来不只是语法。

而是：

- **代码组织方式**
- **模块协作方式**
- **大型项目维护方式**
- **团队开发思维**
- **软件工程体系**

这些东西加在一起，才是“面向对象”的真正意义。

---

# 为什么很多人学不会 OOP

很多教程的问题是：

- 一上来讲 `class`
- 然后讲 `extends`
- 然后讲“万物皆对象”

结果新人完全不知道这些东西到底解决了什么问题。

实际上：

> OOP 本质是在解决“大型程序复杂度失控”的问题。

小项目：

```c
main()
```

当然能跑。

但项目越来越大后：

```txt
用户模块
订单模块
权限模块
支付模块
消息模块
缓存模块
数据库模块
日志模块
```

如果还是：

```c
xxx_user()
xxx_order()
xxx_pay()
```

最后会变成：

```txt
全局变量到处飞
函数互相调用
模块互相依赖
改一个地方全崩
```

这也是很多 C 项目后期极其痛苦的原因。

---

# C 语言的核心思维

C 的核心是：

- 数据结构
- 内存
- 指针
- 过程控制

例如：

```c
struct User {
    char name[20];
    int age;
};

void print_user(struct User* user) {
    printf("%s", user->name);
}
```

这里：

- 数据是 `struct`
- 行为是函数
- 两者分离

这种方式：

## 优点

- 性能高
- 内存可控
- 非常贴近底层
- 非常适合系统开发

所以：

- Linux
- Redis
- Nginx
- SQLite

本质上都是 C 系生态。

---

# Java 的核心变化

Java 开始强调：

```txt
数据 + 行为
应该绑定在一起
```

于是：

```java
public class User {

    private String name;

    public void printUser() {
        System.out.println(name);
    }
}
```

核心思想：

## 对象 = 状态 + 行为

这件事非常关键。

因为它开始：

- 限制数据访问
- 控制模块边界
- 减少外部破坏

这就是：

# 封装（Encapsulation）

---

# 封装到底解决了什么

很多人觉得：

```java
private
getter
setter
```

很烦。

但它实际上是在：

## 防止系统失控

例如：

```java
user.age = -100;
```

显然非法。

于是：

```java
public void setAge(int age){
    if(age < 0){
        throw new RuntimeException();
    }
    this.age = age;
}
```

这样：

- 数据合法性可控
- 外部无法乱改
- 模块更稳定

大型系统里非常重要。

---

# 继承为什么后期越来越少

初学 Java 时：

```java
class Cat extends Animal
```

感觉很帅。

但工程实践后会发现：

> 继承其实耦合度非常高。

例如：

```txt
父类改动
↓
所有子类受影响
```

尤其：

```txt
多层继承
```

会变成灾难。

所以现代 Java 越来越强调：

# 组合优于继承

例如：

```java
class UserService {
    private RedisService redisService;
}
```

而不是：

```java
class UserService extends RedisService
```

---

# 多态为什么是 Java 的核心

真正让 Java 工程能力提升的：

其实是：

# 抽象接口

例如：

```java
interface Storage {
    void save();
}
```

然后：

```java
class MysqlStorage implements Storage
class RedisStorage implements Storage
class OssStorage implements Storage
```

业务层：

```java
Storage storage
```

根本不关心：

```txt
到底是谁实现的
```

这就是：

# 解耦

也是：

- Spring
- MyBatis
- Tomcat
- JDBC

整个 Java 生态最核心的思想。

---

# 为什么 Java 特别强调接口

因为大型项目真正重要的是：

## 规范

而不是：

## 某个具体实现

例如：

```txt
支付接口
日志接口
缓存接口
消息接口
```

大家只需要：

```txt
遵守同一个协议
```

就能协作。

于是：

团队开发能力会大幅提高。

---

# Java 为什么适合企业开发

因为 Java 天生强调：

## 工程化

包括：

| 能力   | Java生态          |
| ------ | ----------------- |
| 包管理 | Maven             |
| IOC    | Spring            |
| ORM    | MyBatis/Hibernate |
| Web    | SpringMVC         |
| 日志   | Logback           |
| 监控   | Actuator          |
| 构建   | Maven/Gradle      |
| 测试   | Junit             |

它不是：

```txt
单个语言强
```

而是：

```txt
整个企业开发生态成熟
```

---

# Java 与 Python 的区别

后来再回头看：

两者思维差异其实很明显。

| 对比         | Java   | Python   |
| ------------ | ------ | -------- |
| 类型         | 强类型 | 动态类型 |
| 工程约束     | 强     | 弱       |
| 开发速度     | 中等   | 快       |
| 大型协作     | 强     | 中等     |
| 灵活性       | 一般   | 极强     |
| 企业传统生态 | 极强   | 中等     |

Python 更像：

```txt
快速实现
```

Java 更像：

```txt
大型工业系统
```

---

# 我后面才真正理解的东西

以前觉得：

```txt
面向对象 = class
```

后来才发现：

真正核心的是：

# 软件复杂度管理

包括：

- 分层
- 解耦
- 依赖控制
- 模块边界
- 接口规范
- 团队协作

这些才是 OOP 真正重要的东西。

---

# 为什么很多人学 Java 很痛苦

因为：

## 一开始接触的就是“工业体系”

新人甚至：

```txt
main 都没理解
```

就开始：

- Spring
- Maven
- IOC
- AOP
- Bean
- 注解

自然会懵。

Java 的难点：

从来不是语法。

而是：

# 工程体系

---

# 后来我对 Java 的理解

Java 最强的地方不是：

```txt
语言本身
```

而是：

# 一整套成熟的软件工程规范

包括：

- 怎么组织项目
- 怎么管理依赖
- 怎么分层
- 怎么解耦
- 怎么部署
- 怎么团队协作

这些东西结合起来：

才构成了真正的：

# Java 企业开发体系
