---
title: routers/tasks.py文件简介
date: 2026-06-05 13:14:20
tags: [Python Project]
categories: [ToDo]
permalink: /todo-doc/routers.tasks.py/
---


# routers/tasks.py文件简介


`routers/tasks.py` 是后端项目的 **API 接口层**（路由层）。它的核心职责是**定义 URL 路径、HTTP 方法，并将接收到的请求分发给业务逻辑层处理**。

它是前端与后端交互的“大门”。当前端发起请求时，首先到达的就是这里。

---

<!--more-->


### 1. 代码深度解析


#### A. 路由器初始化


```python

router = APIRouter(prefix="/tasks", tags=["tasks"])

```

*   **`prefix="/tasks"`**: 给这个路由器下的所有接口自动加上 `/tasks` 前缀。
    *   例如：`@router.get("")` 实际上对应的是 `GET /tasks`。
*   **`tags=["tasks"]`**: 在自动生成的 Swagger 文档中，将这些接口归类到 "tasks" 分组下，方便查看。


#### B. 获取所有任务 (`GET /tasks`)


```python

@router.get("", response_model=list[TaskRead])
async def list_tasks(db: AsyncSession = Depends(get_db)) -> list[TaskRead]:
    return await crud.get_tasks(db)

```

*   **`@router.get("")`**: 定义一个 GET 请求接口。
*   **`response_model=list[TaskRead]`**: 告诉 FastAPI 返回的数据格式。FastAPI 会自动将数据库返回的 `Task` 对象列表转换为 `TaskRead` 格式的 JSON，并过滤掉不需要返回的字段。
*   **`Depends(get_db)`**: 依赖注入。FastAPI 会自动调用 `database.py` 中的 `get_db` 函数，获取一个数据库会话 `db`，并在请求结束后自动关闭它。


#### C. 创建新任务 (`POST /tasks`)


```python

@router.post("", response_model=TaskRead, status_code=status.HTTP_201_CREATED)
async def create_task(
    task_in: TaskCreate, db: AsyncSession = Depends(get_db)
) -> TaskRead:
    return await crud.create_task(db, task_in)

```

*   **`task_in: TaskCreate`**: FastAPI 会自动从 HTTP 请求体（Body）中读取 JSON 数据，并使用 `schemas.py` 中的 `TaskCreate` 进行验证。如果验证失败（如标题为空），它会直接返回 422 错误，不会执行函数内部代码。
*   **`status_code=201`**: 创建成功通常返回 201 (Created)，而不是默认的 200。


#### D. 更新任务 (`PUT /tasks/{task_id}`)


```python

@router.put("/{task_id}", response_model=TaskRead)
async def update_task(
    task_id: int, task_in: TaskUpdate, db: AsyncSession = Depends(get_db)
) -> TaskRead:
    # 1. 先检查任务是否存在
    task = await crud.get_task(db, task_id)
    if task is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND, detail="Task not found"
        )
    # 2. 存在则更新
    return await crud.update_task(db, task, task_in)

```

*   **`{task_id}`**: 路径参数。URL 类似 `/tasks/1`，FastAPI 会自动将 `1` 转换为整数传给 `task_id`。
*   **`HTTPException`**: 如果数据库里找不到这个 ID，手动抛出一个 404 错误，前端会收到清晰的报错信息。


#### E. 删除任务 (`DELETE /tasks/{task_id}`)


```python

@router.delete("/{task_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_task(task_id: int, db: AsyncSession = Depends(get_db)) -> None:
    task = await crud.get_task(db, task_id)
    if task is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND, detail="Task not found"
        )
    await crud.delete_task(db, task)

```

*   **`status_code=204`**: 删除成功通常返回 204 (No Content)，表示操作成功但没有内容返回给前端。
*   **`-> None`**: 函数没有返回值。

---


### 2. 它在项目中的位置


你可以把后端看作一个餐厅：
1.  **`routers/tasks.py` (服务员)**：接待顾客（前端），记下顾客要点什么菜（接收请求参数），检查菜单是否合法（数据验证），然后把单子交给厨房。
2.  **`crud.py` (厨师)**：在厨房裡真正做菜（操作数据库），做好后把菜递给服务员。
3.  **`models.py` (食材)**：定义菜的原材料结构。
4.  **`schemas.py` (菜单)**：定义顾客能点什么，以及端上去的菜长什么样。

**`routers` 不负责具体的数据库操作，它只负责“调度”和“校验”。**

---


### 3. 如何注册路由？


定义了 `router` 后，还需要在 `main.py` 中把它挂载到主应用上：

```python

# main.py
from fastapi import FastAPI
from app.routers import tasks

app = FastAPI()

# 将 tasks 路由器注册到 app
app.include_router(tasks.router)

```

这样，访问 `http://localhost:8000/tasks` 就会触发 `routers/tasks.py` 中的逻辑。

---


### 4. 总结


`routers/tasks.py` 是你后端 API 的 **入口点**。
*   它定义了 **URL** 和 **HTTP 方法** (GET, POST, PUT, DELETE)。
*   它利用 **Pydantic Schema** 自动验证输入数据。
*   它利用 **Dependency Injection** (`Depends`) 管理数据库会话。
*   它通过调用 **CRUD** 函数实现业务逻辑，保持代码解耦。

对于前端开发者来说，这个文件决定了他们该调用哪个 URL，以及该传什么参数。