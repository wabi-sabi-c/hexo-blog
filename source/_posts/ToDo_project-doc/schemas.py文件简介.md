---
title: schemas.py文件简介
date: 2026-06-05 13:12:20
tags: [Python Project]
categories: [ToDo]
permalink: /todo-doc/schemas.py/
---


# schemas.py 文件简介


`schemas.py` 是后端项目的**数据校验与接口契约层**。它的核心职责是**定义 API 接收什么数据（请求体）以及返回什么数据（响应体）**。

它使用 **Pydantic** 库，充当了前端与后端之间的“翻译官”和“安检员”。

---

<!--more-->


### 1. 代码深度解析


```python

from datetime import datetime
from pydantic import BaseModel, ConfigDict, Field

# 1. 基础模型：提取公共字段，避免重复代码
class TaskBase(BaseModel):
    title: str = Field(..., min_length=1, max_length=255) # ... 表示必填
    description: str | None = None
    completed: bool = False

# 2. 创建模型：继承基础模型，用于 POST 请求
class TaskCreate(TaskBase):
    pass

# 3. 更新模型：所有字段可选，用于 PATCH/PUT 请求
class TaskUpdate(BaseModel):
    title: str | None = Field(None, min_length=1, max_length=255)
    description: str | None = None
    completed: bool | None = None

# 4. 读取模型：用于 GET 请求的响应，包含数据库生成的字段（如 id, created_at）
class TaskRead(TaskBase):
    model_config = ConfigDict(from_attributes=True) # 允许从 ORM 对象读取数据

    id: int
    created_at: datetime

```


#### 关键概念说明：


1.  **`BaseModel`**：
    *   这是 Pydantic 的核心类。所有 Schema 都继承自它。
    *   它会自动根据类型注解（如 `str`, `int`）验证传入的数据。

2.  **`Field(...)`**：
    *   用于提供额外的验证规则。
    *   `...` (Ellipsis) 表示该字段是**必填项**。
    *   `min_length=1, max_length=255`：限制字符串长度，防止用户提交空标题或过长的文本。

3.  **模型继承与复用**：
    *   `TaskCreate` 继承 `TaskBase`：因为新建任务时，用户提供的字段和基础字段一致。
    *   `TaskUpdate` 独立定义：因为更新任务时，用户可能只改标题，不改描述，所以所有字段都设为 `Optional` (`| None`)。

4.  **`model_config = ConfigDict(from_attributes=True)`**：
    *   **非常重要！** 默认情况下，Pydantic 只能从字典（dict）读取数据。
    *   但在 FastAPI 中，我们通常直接从数据库查询得到 SQLAlchemy 的 **ORM 对象**（如 `Task` 实例）。
    *   这个配置告诉 Pydantic：“请尝试从对象的属性（attributes）中读取数据”，从而让我们能直接返回 `TaskRead` 对象给前端。

---


### 2. 为什么需要这么多不同的 Schema？


你可能会问：“为什么不直接用 `TaskBase` 处理所有情况？”

| 场景 | 使用的 Schema | 原因 |
| :--- | :--- | :--- |
| **新建任务 (POST)** | `TaskCreate` | 用户必须提供标题，ID 和创建时间由数据库生成，不需要用户传。 |
| **更新任务 (PATCH)** | `TaskUpdate` | 用户可能只修改“完成状态”，其他字段不传。如果必填，用户就得把所有字段再发一遍，体验很差。 |
| **查看任务 (GET)** | `TaskRead` | 前端需要看到完整的任务信息，包括数据库自动生成的 `id` 和 `created_at`。 |

这种分离设计（Separation of Concerns）使得 API 接口非常清晰且安全。

---


### 3. 它在项目中如何被使用？


#### A. 在路由中接收数据 (`routers/tasks.py`)


```python

@app.post("/tasks/", response_model=TaskRead)
async def create_task(task: TaskCreate, db: AsyncSession = Depends(get_db)):
    # FastAPI 会自动检查请求体是否符合 TaskCreate 的定义
    # 如果 title 为空，它会直接返回 422 错误，不会执行下面的代码
    new_task = await crud.create_task(db, task)
    return new_task

```


#### B. 在 CRUD 中使用

```python

# crud.py
async def create_task(db: AsyncSession, task: TaskCreate):
    # 将 Pydantic 模型转换为 SQLAlchemy 模型
    db_task = Task(title=task.title, description=task.description)
    db.add(db_task)
    await db.commit()
    await db.refresh(db_task)
    return db_task

```


#### C. 自动生成交互式文档
当你打开 `http://localhost:8000/docs` 时，FastAPI 会读取这些 Schema，自动生成漂亮的文档：
*   它会显示 `TaskCreate` 有哪些必填字段。
*   它会显示 `TaskRead` 会返回哪些字段。
*   你可以直接在网页上点击 "Try it out" 进行测试。

---


### 4. 总结


`schemas.py` 是你 API 的**合同**。
*   它保护后端不被非法数据破坏（通过验证）。
*   它帮助前端理解该如何调用接口（通过文档）。
*   它利用 Pydantic 的强大功能，让数据转换变得自动化且类型安全。

**记住一个原则**：
*   `models.py` 关心**数据库怎么存**。
*   `schemas.py` 关心**用户怎么传/怎么看**。