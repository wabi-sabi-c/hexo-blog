---
title: models.py文件简介
date: 2026-06-05 13:11:20
tags: [Python Project]
categories: [ToDo]
permalink: /todo-doc/models.py/
---


# models.py文件简介


`models.py` 是后端项目的**数据模型层**。它的核心职责是**定义数据库表的结构**，将 Python 类映射到 PostgreSQL 中的具体表格。

在 SQLAlchemy 2.0 的异步模式下，它使用了现代化的 **Mapped Types** 语法，使得代码更加简洁且具备强大的类型提示支持。

---

<!--more-->


### 1. 代码深度解析


```python

from datetime import datetime
from sqlalchemy import Boolean, DateTime, String, Text, func
from sqlalchemy.orm import Mapped, mapped_column
from app.database import Base  # 继承自 database.py 中定义的基类

class Task(Base):
    __tablename__ = "tasks"  # 对应数据库中的表名

    # 1. 主键 ID：整数，自动递增
    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    
    # 2. 标题：字符串，最大长度 255，不允许为空
    title: Mapped[str] = mapped_column(String(255), nullable=False)
    
    # 3. 描述：长文本，允许为空 (None)
    description: Mapped[str | None] = mapped_column(Text, nullable=True)
    
    # 4. 完成状态：布尔值，默认为 False，不允许为空
    completed: Mapped[bool] = mapped_column(Boolean, default=False, nullable=False)
    
    # 5. 创建时间：带时区的日期时间，默认值为数据库当前时间
    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), 
        server_default=func.now(), 
        nullable=False
    )

```


#### 关键概念说明：


1.  **`Base` 类**：
    *   所有模型类都必须继承自 `database.py` 中定义的 `Base`。SQLAlchemy 通过它来追踪哪些类需要被创建为数据库表。

2.  **`Mapped[...]` (类型注解)**：
    *   这是 SQLAlchemy 2.0 的核心特性。它告诉 Python 解释器和 IDE：这个属性在 Python 层面是什么类型（如 `int`, `str`）。
    *   好处：当你写 `task.title` 时，IDE 会知道它是一个字符串，并提供智能补全。

3.  **`mapped_column(...)` (列配置)**：
    *   这里定义的是**数据库层面**的属性。
    *   `String(255)`：对应数据库的 `VARCHAR(255)`。
    *   `Text`：对应数据库的 `TEXT` 类型，适合存长描述。
    *   `nullable=False`：约束该字段在数据库中不能为 `NULL`。
    *   `server_default=func.now()`：这是一个非常实用的技巧。它让 **PostgreSQL 数据库**在插入数据时自动填入当前时间，而不是依赖 Python 代码生成时间。这样能确保时间的准确性（以服务器时间为准）。

---


### 2. 它与 `schemas.py` 的区别（初学者易混淆点）


很多初学者会问：“既然有了 `models.py`，为什么还需要 `schemas.py`？”

| 特性 | `models.py` (ORM 模型) | `schemas.py` (Pydantic 模型) |
| :--- | :--- | :--- |
| **用途** | **与数据库对话**。定义表结构，用于存取数据。 | **与前端对话**。定义 API 接口接收和返回的数据格式。 |
| **库** | 使用 `sqlalchemy` | 使用 `pydantic` |
| **关注点** | 数据库类型 (String, Integer, DateTime) | 数据验证 (min_length, email 格式, 必填项) |
| **生命周期** | 存在于整个后端逻辑中，代表数据库里的一行记录。 | 仅存在于 API 请求/响应的边界，用于验证和序列化。 |

**简单比喻**：
*   `models.py` 是**仓库的货架标签**（规定了这个格子能放多大的箱子）。
*   `schemas.py` 是**快递单的填写规范**（规定了客户寄件时必须填哪些信息，格式是什么）。

---


### 3. 如何在项目中使用它？


#### A. 创建表


在项目初始化时（通常在 `main.py` 或启动脚本中），SQLAlchemy 会根据 `Task` 类的定义，在 PostgreSQL 中自动生成 `tasks` 表：
```python

# 伪代码
async with engine.begin() as conn:
    await conn.run_sync(Base.metadata.create_all)

```

#### B. 查询数据


在 `crud.py` 中，你会这样使用它：
```python

from app.models import Task

# 查询所有任务
result = await db.execute(select(Task))
tasks = result.scalars().all() # 返回的是 Task 对象列表

```


#### C. 新增数据


```python

# 创建一个 Task 实例
new_task = Task(title="学习 FastAPI", description="阅读文档")
db.add(new_task)
await db.commit()

```

---


### 4. 总结


`models.py` 是你后端数据的**蓝图**。
*   它决定了你的数据库长什么样。
*   它利用了 SQLAlchemy 2.0 的现代语法，提供了完美的类型安全。
*   它是连接 Python 代码与 PostgreSQL 数据库的桥梁。

当你需要增加新的功能（比如给任务加一个“截止日期” `due_date`）时，你首先就要修改这个文件，添加一个新的 `mapped_column`。