---
title: Spring Boot 与 Java Web 开发体系整理
date: 2024-10-03
tags:
  - SpringBoot
  - SpringMVC
  - MyBatis
  - Maven
  - Tomcat
categories:
  - 后端开发
  - Java
description: 对 Java Web 开发体系的一次完整整理，包括 Spring、SpringBoot、SpringMVC、MyBatis、Maven、Tomcat、Lombok 等核心生态的作用、关系、工作流程与常见踩坑。
---

# Spring Boot 与 Java Web 开发体系整理

很多人第一次接触 Java Web 时都会懵。

因为会同时出现：

- Spring
- SpringBoot
- SpringMVC
- MyBatis
- Maven
- Tomcat
- Lombok

甚至还有：

- Redis
- RabbitMQ
- Nacos
- Gateway
- Dubbo

新人非常容易：

```txt
概念混乱
```

这篇文章尽量把：

# Java Web 整个体系的关系

讲清楚。

---

# Java Web 到底是什么

本质上：

```txt
浏览器
↓
HTTP请求
↓
Java服务器
↓
数据库
```

Java Web 就是在处理：

- 路由
- 参数
- 登录
- 数据库
- 权限
- JSON
- 文件上传
- 接口响应

这些东西。

---

# Java Web 项目的典型结构

现代项目通常：

```txt
src
├── controller
├── service
├── mapper
├── entity
├── dto
├── config
├── utils
└── common
```

对应：

| 层         | 作用       |
| ---------- | ---------- |
| controller | 接口入口   |
| service    | 业务逻辑   |
| mapper     | 数据库操作 |
| entity     | 数据对象   |
| dto        | 参数对象   |
| config     | 配置       |
| utils      | 工具类     |

这套结构：

本质是：

# 分层架构

---

# Maven 到底是什么

很多新人以为：

```txt
Maven = 依赖下载器
```

其实它本质是：

# Java项目构建工具

负责：

- 下载依赖
- 项目构建
- 生命周期管理
- 插件管理
- 打包
- 编译

---

# Maven 最核心的文件

```xml
pom.xml
```

例如：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

它会自动：

- 下载jar
- 管理版本
- 处理依赖关系

---

# Maven 为什么会出现依赖地狱

例如：

```txt
A依赖B1.0
C依赖B2.0
```

最后：

```txt
到底加载谁？
```

于是：

经常出现：

```txt
NoSuchMethodError
ClassNotFoundException
```

这也是 Java 老项目非常常见的问题。

---

# Spring 到底是什么

Spring 本质是：

# IOC容器

即：

# 控制反转

以前：

```java
UserService service = new UserService();
```

现在：

```java
@Autowired
private UserService service;
```

对象不再：

```txt
自己创建
```

而是：

```txt
Spring统一管理
```

---

# IOC 到底解决了什么

核心是：

# 解耦

例如：

```txt
UserService
依赖
RedisService
```

Spring 自动：

- 创建对象
- 注入对象
- 管理生命周期

于是：

业务代码不用：

```java
new
```

整个系统会更容易维护。

---

# SpringBoot 为什么火

以前 Spring 配置：

```xml
一大堆
```

新人直接崩溃。

SpringBoot 出现后：

```java
@SpringBootApplication
```

直接自动配置。

核心理念：

# 约定优于配置

---

# SpringMVC 是什么

SpringMVC：

本质是：

# Web MVC框架

负责：

- 路由
- 参数解析
- JSON响应
- 请求处理

例如：

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @GetMapping("/{id}")
    public User getUser(@PathVariable Integer id){
        return service.getUser(id);
    }
}
```

---

# 请求是怎么流转的

完整流程：

```txt
浏览器
↓
Tomcat
↓
DispatcherServlet
↓
Controller
↓
Service
↓
Mapper
↓
MySQL
```

其中：

```txt
DispatcherServlet
```

是 SpringMVC 核心调度器。

---

# MyBatis 是什么

MyBatis：

本质是：

# ORM/SQL映射框架

用于：

```txt
Java对象
↔
SQL
```

例如：

```java
@Mapper
public interface UserMapper {

    User selectById(Integer id);
}
```

对应：

```xml
<select id="selectById">
    select * from user where id = #{id}
</select>
```

---

# MyBatis 为什么很多人喜欢

因为：

## SQL可控

相比：

```txt
Hibernate全自动ORM
```

MyBatis 更适合：

- 复杂SQL
- 大型业务
- 高频优化

很多企业至今仍主用 MyBatis。

---

# Lombok 为什么几乎成了标配

以前：

```java
getter
setter
toString
constructor
```

全手写。

后来：

```java
@Data
@Builder
@NoArgsConstructor
```

直接生成。

项目代码量会少非常多。

---

# Lombok 的坑

最经典的：

## IDEA能跑

## Maven编译失败

原因通常：

```txt
annotationProcessor
未开启
```

或者：

```txt
JDK版本不兼容
```

---

# Tomcat 到底是什么

Tomcat：

本质是：

# Servlet容器

负责：

- 接收HTTP请求
- 管理线程
- 返回响应

SpringBoot 内嵌了：

```txt
Tomcat
```

所以：

```bash
java -jar xxx.jar
```

就能直接运行。

---

# Java Web 为什么层特别多

因为：

# 企业系统复杂

例如：

```txt
权限
事务
缓存
日志
消息队列
分布式
```

如果不分层：

后期会非常难维护。

---

# Java Web 常见踩坑

---

## 1. Bean循环依赖

例如：

```txt
A依赖B
B依赖A
```

Spring 会直接报错。

解决：

- 重新拆分业务
- 减少循环引用
- 使用事件机制

---

## 2. 数据库连接池耗尽

表现：

```txt
接口卡死
```

原因：

```txt
连接未释放
```

或者：

```txt
SQL执行过慢
```

解决：

- HikariCP
- SQL优化
- 合理连接池大小

---

## 3. JSON序列化问题

经典：

```txt
LocalDateTime
```

格式错误。

解决：

```java
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
```

---

## 4. Maven仓库污染

表现：

```txt
明明依赖正常
就是编译失败
```

解决：

```bash
mvn clean
```

或者：

删除：

```txt
.m2/repository
```

重新下载。

---

# Java Web 为什么现在仍然强势

因为：

## 企业需要稳定

Java 生态：

- 成熟
- 文档多
- 人才多
- 中间件完善
- 微服务生态完整

尤其：

```txt
金融
政企
大型互联网
```

至今仍大量使用。

---

# 我后面对 Java Web 的理解

它真正强的：

从来不是：

```txt
语法
```

而是：

# 一整套成熟的工程体系

包括：

- IOC
- 分层
- ORM
- 构建系统
- 中间件
- 微服务
- 规范化开发

这些组合起来：

才构成了：

# Java 企业级开发生态
