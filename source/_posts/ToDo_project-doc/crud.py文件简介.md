---
title: crud.py文件简介
date: 2026-06-05 13:13:20
tags: [Python Project]
categories: [ToDo]
permalink: /todo-doc/crud.py/
---


# crud.py文件简介


`crud.py` 是后端项目的**业务逻辑层**（也称为数据访问层）。它的核心职责是**封装所有与数据库交互的具体操作**（增删改查，即 CRUD）。

在软件工程中，这种设计模式被称为 **分离关注点（Separation of Concerns）**：
*   **路由层 (`routers/`)** 只负责接收 HTTP 请求和返回响应。
*   **CRUD 层 (`crud.py`)** 负责具体的数据库查询和事务处理。

这样做的好处是代码整洁、易于测试，且如果未来需要更换数据库或修改查询逻辑，只需改动 `crud.py`，而不需要动路由代码。

---

<!--more-->


### 1. 代码深度解析


#### A. 查询所有任务 (`get_tasks`)


```python

async def get_tasks(db: AsyncSession) -> list[Task]:
    # 1. 构建查询语句：选择 Task 表，并按创建时间倒序排列
    result = await db.execute(select(Task).order_by(Task.created_at.desc()))
    # 2. 提取结果：scalars() 获取对象本身，all() 获取列表
    return list(result.scalars().all())

```

*   **`select(Task)`**: SQLAlchemy 2.0 的标准查询写法。
*   **`order_by(...desc())`**: 确保最新的任务排在最前面。
*   **`result.scalars().all()`**: `execute` 返回的是一个 Result 对象，我们需要通过 `scalars()` 拿到具体的 `Task` 实例列表。


#### B. 查询单个任务 (`get_task`)


```python

async def get_task(db: AsyncSession, task_id: int) -> Task | None:
    # 使用主键直接查询，最简单高效的方式
    return await db.get(Task, task_id)

```

*   **`db.get()`**: 这是 SQLAlchemy 提供的快捷方法，专门用于根据主键（Primary Key）获取对象。如果找不到，返回 `None`。


#### C. 创建任务 (`create_task`)


```python

async def create_task(db: AsyncSession, task_in: TaskCreate) -> Task:
    # 1. 解包数据：将 Pydantic 模型转换为字典，并 unpack 到 Task 构造函数中
    task = Task(**task_in.model_dump())
    
    # 2. 添加到会话
    db.add(task)
    
    # 3. 提交事务：真正写入数据库
    await db.commit()
    
    # 4. 刷新对象：从数据库重新加载最新数据（如自动生成的 id 和 created_at）
    await db.refresh(task)
    
    return task

```

*   **`task_in.model_dump()`**: 这是 Pydantic V2 的方法，将验证过的数据转为普通字典 `{'title': '...', 'description': '...'}`。
*   **`await db.refresh(task)`**: 非常重要！因为 `id` 和 `created_at` 是数据库自动生成的，如果不 refresh，返回给前端的对象里这两个字段可能是空的。


#### D. 更新任务 (`update_task`)


```python

async def update_task(db: AsyncSession, task: Task, task_in: TaskUpdate) -> Task:
    # 1. 提取非空字段：exclude_unset=True 确保只更新用户传了的字段
    data = task_in.model_dump(exclude_unset=True)
    
    # 2. 动态赋值：遍历字典，修改 task 对象的属性
    for key, value in data.items():
        setattr(task, key, value)
        
    # 3. 提交并刷新
    await db.commit()
    await db.refresh(task)
    return task

```

*   **`exclude_unset=True`**: 这是处理 `PATCH` 请求的关键。如果用户只传了 `{"completed": true}`，那么 `data` 里就只有 `completed`，其他字段（如 `title`）不会被覆盖为 `None`。


#### E. 删除任务 (`delete_task`)


```python

async def delete_task(db: AsyncSession, task: Task) -> None:
    await db.delete(task)
    await db.commit()

```

*   简单直接：标记删除并提交事务。

---


### 2. 它在项目中如何被调用？


在 `routers/tasks.py` 中，你会看到这样的结构：

```python

from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from app.database import get_db
from app import crud, schemas

router = APIRouter()

@router.get("/tasks/", response_model=list[schemas.TaskRead])
async def read_tasks(db: AsyncSession = Depends(get_db)):
    # 路由层只负责调用 crud，不写任何 SQL
    tasks = await crud.get_tasks(db)
    return tasks

@router.post("/tasks/", response_model=schemas.TaskRead)
async def create_new_task(
    task: schemas.TaskCreate, 
    db: AsyncSession = Depends(get_db)
):
    # 路由层只负责传递参数
    return await crud.create_task(db, task)

```

---


### 3. 为什么要把 CRUD 单独抽离出来？


1.  **复用性**：如果你需要在多个接口（比如“获取任务列表”和“统计任务数量”）中查询数据库，你可以直接调用 `crud.get_tasks`，而不需要重复写 SQL。
2.  **可测试性**：你可以单独为 `crud.py` 编写单元测试，模拟数据库会话，验证逻辑是否正确，而不需要启动整个 Web 服务器。
3.  **清晰度**：`routers` 文件会变得非常短小精悍，只关注 API 的定义（URL、HTTP 方法、权限校验），而复杂的数据库逻辑隐藏在 `crud` 中。


### 4. 总结


`crud.py` 是你后端的**实干家**。
*   它不关心 HTTP 协议，只关心怎么把数据存进 PostgreSQL。
*   它利用了 SQLAlchemy 的异步特性，确保高并发下的性能。
*   它是连接 **Schema（数据规范）** 和 **Model（数据库结构）** 的桥梁。

对于初学者来说，理解 `create_task` 中的 `commit` 和 `refresh`，以及 `update_task` 中的 `exclude_unset`，是掌握 FastAPI 后端逻辑的关键。