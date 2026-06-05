---
title: database.py文件简介
date: 2026-06-05 13:10:20
tags: [Python Project]
categories: [ToDo]
permalink: /todo-doc/database.py/
---


# database.py文件简介


`database.py` 是后端项目的**数据库基础设施层**。它的核心职责是**建立与 PostgreSQL 数据库的连接通道，并管理数据库会话（Session）**。

如果说 `config.py` 提供了“地址和钥匙”，那么 `database.py` 就是负责“开车去仓库”并“管理搬运过程”的司机和调度员。

---

<!--more-->


### 1. 核心组件解析


#### A. 异步引擎 (`engine`)


```python

engine = create_async_engine(settings.database_url, echo=False)

```

*   **作用**：这是 SQLAlchemy 的核心入口，负责维护与数据库的连接池。
*   **`create_async_engine`**：因为我们要配合 FastAPI 使用异步模式，所以必须用这个函数（而不是同步的 `create_engine`）。
*   **`settings.database_url`**：从 `config.py` 读取连接字符串（如 `postgresql+asyncpg://...`）。
*   **`echo=False`**：是否打印 SQL 日志。开发时可以设为 `True` 方便调试，生产环境务必设为 `False` 以提升性能。


#### B. 会话工厂 (`async_session_maker`)


```python

async_session_maker = async_sessionmaker(
    engine, class_=AsyncSession, expire_on_commit=False
)

```

*   **作用**：它是一个“工厂”，每次需要操作数据库时，我们就调用它来创建一个独立的**会话（Session）**。
*   **`class_=AsyncSession`**：指定创建的是异步会话对象，支持 `await` 操作。
*   **`expire_on_commit=False`**：这是一个重要的优化配置。默认情况下，事务提交后，对象属性会过期，再次访问会重新查询数据库。设为 `False` 可以避免在异步环境中出现“GreenletSpawnError”等常见错误，并保持数据在内存中的可用性。


#### C. 基类 (`Base`)


```python

class Base(DeclarativeBase):
    pass

```
*   **作用**：这是所有数据库模型（Model）的父类。
*   **用法**：在 `models.py` 中，你的 `Task` 类会继承这个 `Base`。SQLAlchemy 通过它来识别哪些类需要映射到数据库表。


#### D. 依赖注入函数 (`get_db`)


```python

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with async_session_maker() as session:
        yield session

```

*   **作用**：这是 FastAPI **依赖注入（Dependency Injection）** 的核心。
*   **工作流程**：
    1.  当 API 请求进来时，FastAPI 调用此函数。
    2.  `async with` 打开一个会话。
    3.  `yield session` 将会话传递给路由函数（如 `tasks.py` 中的接口）。
    4.  路由函数执行完毕（无论成功或报错）后，代码回到 `yield` 之后，`async with` 块结束，**自动关闭会话并提交/回滚事务**。
*   **好处**：你不需要在每个接口里手动写 `session.close()`，防止连接泄露。


#### E. 健康检查 (`check_db_connection`)


```python

async def check_db_connection() -> bool:
    try:
        async with engine.connect() as conn:
            await conn.execute(text("SELECT 1"))
        return True
    except Exception:
        return False

```

*   **作用**：用于测试数据库是否连通。通常在项目启动时或编写单元测试时使用。

---


### 2. 它在项目中如何被使用？


#### 第一步：在 `models.py` 中继承 `Base`


```python

from app.database import Base
from sqlalchemy import Column, Integer, String

class Task(Base):
    __tablename__ = "tasks"
    id = Column(Integer, primary_key=True)
    title = Column(String)
    # ...

```


#### 第二步：在 `routers/tasks.py` 中注入 `get_db`


```python

from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession
from app.database import get_db

@app.post("/tasks/")
async def create_task(task: TaskCreate, db: AsyncSession = Depends(get_db)):
    # FastAPI 会自动调用 get_db() 并把 session 赋值给 db
    # 你可以在这里直接使用 db 进行查询或保存
    new_task = Task(title=task.title)
    db.add(new_task)
    await db.commit()
    # 无需手动关闭 db，get_db 会自动处理
    return new_task

```

---


### 3. 为什么需要异步会话？


在传统的同步框架（如 Flask + SQLAlchemy）中，数据库操作会阻塞线程。而在 **FastAPI + asyncpg** 架构中：
1.  **非阻塞**：当 `await db.execute(...)` 等待数据库返回时，Python 事件循环可以去处理其他用户的请求。
2.  **高并发**：这使得你的 ToDo-App 能够同时处理成百上千个请求，而不会因为数据库等待而卡死。


### 4. 总结


`database.py` 是后端与数据库交互的**桥梁**。
*   **`engine`** 是连接池（水管总闸）。
*   **`session_maker`** 是会话工厂（水龙头）。
*   **`get_db`** 是自动化管理器（确保用完水后自动关紧水龙头，不浪费资源）。

理解了这个文件，你就掌握了 **FastAPI 异步数据库操作** 的精髓。