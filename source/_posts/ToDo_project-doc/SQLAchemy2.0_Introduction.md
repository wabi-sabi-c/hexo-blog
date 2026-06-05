---
title: SQLAchemy 2.0简介
date: 2026-06-05 13:02:20
tags: [Python Project]
categories: [ToDo]
permalink: /todo-doc/SQLAchemy2.0/
---


# SQLAlchemy 2.0


**SQLAlchemy 2.0** 是 Python 中最流行、功能最强大的 **ORM（对象关系映射）** 库。

在你的 ToDo-App 项目中，它的主要作用是 **充当 Python 代码与 PostgreSQL 数据库之间的“翻译官”**。


### 1. 核心作用：ORM (Object-Relational Mapping)


*   **不用写 SQL 语句**：你不需要手动编写复杂的 `SELECT * FROM tasks WHERE ...` 这样的 SQL 字符串。
*   **操作 Python 对象**：你只需要操作 Python 的类（Class）和对象（Object）。SQLAlchemy 会自动把这些操作“翻译”成数据库能听懂的 SQL 语句并执行。

<!--more-->


### 2. 为什么项目强调 “2.0” 和 “异步”？


*   **版本 2.0**：这是 SQLAlchemy 的最新重大版本，统一了 API 风格，推荐使用更现代、更明确的写法（如 `select()` 构造查询），代码可读性和维护性更好。
*   
*   **异步支持 (Async)**：
    *   你的后端框架是 **FastAPI**，它是一个高性能的 **异步** Web 框架。
    *   传统的 SQLAlchemy 是同步的（阻塞的），如果数据库查询慢，整个服务器都会卡住等待。
    *   **SQLAlchemy 2.0 + [asyncpg](/todo-doc/asyncpg/)** 允许数据库操作也变成 **异步非阻塞** 的。当后端在等待数据库返回结果时，它可以先去处理其他用户的请求，从而极大地提高了高并发下的性能。


### 3. 举个简单的例子对比


#### ❌ 不使用 ORM (原生 SQL)


你需要手动拼接字符串，容易出错且不安全（SQL注入风险）：
```python

# 伪代码

db.execute("INSERT INTO tasks (title, description) VALUES ('买牛奶', '记得买低脂的')")

```


#### ✅ 使用 SQLAlchemy ORM


你操作的是 Python 对象，更安全、更直观：

```python

# 1. 定义模型 (Model)

class Task(Base):
    __tablename__ = "tasks"
    id = Column(Integer, primary_key=True)
    title = Column(String)
    description = Column(String)

# 2. 创建任务对象

new_task = Task(title="买牛奶", description="记得买低脂的")

# 3. 添加到会话并提交 (SQLAlchemy 自动转换成 INSERT SQL)

async_session.add(new_task)
await async_session.commit()

```


### 总结


在你的项目中，**SQLAlchemy 2.0 (异步)** 负责：

1.  **定义数据结构**：通过 Python 类定义数据库表结构。
   
2.  **执行 CRUD 操作**：帮你完成增删改查，无需手写 SQL。
   
3.  **保持高性能**：配合 FastAPI 实现异步非阻塞，提升后端响应速度。