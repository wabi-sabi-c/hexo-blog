---
title: asyncpg简介
date: 2026-06-05 13:03:20
tags: [Python Project]
categories: [ToDo]
permalink: /todo-doc/asyncpg/
---


# asyncpg简介


**asyncpg** 是一个用于 PostgreSQL 数据库的 **高性能异步 Python 驱动程序**。

在你的 ToDo-App 项目技术栈中，它扮演着 **“底层通信员”** 的角色。

<!--more-->


### 1. 核心作用


*   **建立连接**：它负责让你的 Python 程序（FastAPI后端）与 PostgreSQL 数据库之间建立网络连接。

*   **传输数据**：它将 [SQLAlchemy](/todo-doc/SQLAchemy2.0/) 生成的 SQL 指令发送给数据库，并将数据库返回的结果传回给 Python。
 
*   **异步支持**：这是它最关键的特性。它支持 `async/await` 语法，意味着它在等待数据库响应时**不会阻塞**服务器的其他任务。


### 2. 为什么需要它？（它与 SQLAlchemy 的关系）


你可以把它们的关系理解为 **“经理”与“快递员”**：

*   **SQLAlchemy (ORM)** 是 **经理**：
    *   它负责逻辑层面的工作，比如把 Python 对象转换成 SQL 语句。
    *   但它自己**不直接**去连数据库，它需要一个干苦力的司机。
  
*   **asyncpg (Driver)** 是 **快递员/司机**：
    *   它负责实际的跑腿工作，拿着 SQL 语句去数据库取数据。
    *   因为它是 **async**（异步）的，所以它跑得很快，而且可以同时处理多个快递任务而不堵车。


### 3. 为什么选择 asyncpg 而不是其他驱动？


在 FastAPI + 异步 ORM 的场景下，asyncpg 是首选，原因如下：

1.  **速度极快**：asyncpg 是用 C 语言编写的部分核心，比传统的纯 Python 驱动（如 `psycopg2`）快得多。

2.  **原生异步**：它专为 `asyncio` 设计，完美契合 FastAPI 的异步特性。如果使用同步驱动（如 `psycopg2`），在 FastAPI 中需要额外的线程池来处理，会降低性能。

3.  **资源占用低**：在高并发下，它能以更少的内存处理更多的连接。


### 4. 代码中的体现


在你的项目中，你通常不会直接调用 `asyncpg` 的 API，而是通过 SQLAlchemy 配置它。例如在创建数据库引擎时：

```python

from sqlalchemy.ext.asyncio import create_async_engine

# 注意 URL 前缀是 postgresql+asyncpg://
# 这告诉 SQLAlchemy：“请使用 asyncpg 这个驱动来连接数据库”
engine = create_async_engine(
    "postgresql+asyncpg://user:password@localhost/dbname",
    echo=True,
)

```

### 总结
*   **PostgreSQL**：仓库（存数据的地方）。
*   **SQLAlchemy**：翻译官（把 Python 代码翻译成 SQL）。
*   **asyncpg**：高速快递员（异步地把 SQL 送到仓库并取回结果）。

三者配合，实现了你项目中 **高性能、非阻塞** 的数据存取功能。