---
title: ToDo-WebProject
date: 2026-06-05 13:30:50
tags: [Python Project]
categories: [ToDo]
permalink: /todo-doc/Todo-WebProject/
---


# ToDo-App项目


> 本文面向初学者的从零搭建并运行的ToDo全栈web项目

---


# 目录


## 1.项目简介与技术栈


### 1.1 项目是什么？


ToDo-App是一个待办事项（Todo）全栈web应用：

- 在页面上**新建任务**（标题 + 描述）

- 在页面上**查看所有任务列表**，支持按「全部/进行中/已完成」筛选

- **勾选复选框**，将任务标记为已完成（调用后端更新接口）

- **删除任务**（调用后端删除接口）

数据保存在PostgreSQL数据库中，前后端分离部署。


### 1.2 技术栈一览


|     层级     |                      技术                       |         说明         |
| :----------: | :---------------------------------------------: | :------------------: |
|     后端     |                  python 3.11+                   |       编程语言       |
|   后端框架   | <a href="/todo-doc/FastAPI/" target="_blank" rel="noopener noreferrer">FastAPI</a>          |     异步Web API      |
|     ORM      | <a href="/todo-doc/SQLAchemy2.0/" target="_blank" rel="noopener noreferrer">SQLALchemy2.0(异步)</a> |      操作数据库      |
|  数据库驱动  |   <a href="/todo-doc/asyncpg/" target="_blank" rel="noopener noreferrer">asyncpg</a>    | 连接PostgreSQL数据库 |
|    数据库    | <a href="/todo-doc/PostgreSQL/" target="_blank" rel="noopener noreferrer">PostgreSQL 16</a>     |     用Docker运行     |
|     前端     | <a href="/todo-doc/React/" target="_blank" rel="noopener noreferrer">React 18</a> |        UI框架        |
|   构建工具   |<a href="/todo-doc/Vite/" target="_blank" rel="noopener noreferrer">Vite 6</a> |    开发与打包工具    |
| 前端样式框架 |<a href="/todo-doc/Tailwind_CSS/" target="_blank" rel="noopener noreferrer">Tailwind CSS 3</a>|      原子化CSS       |

<!--more-->


### 1.3 项目目录结构


```text

todo_app/
├── docker-compose.yml      # PostgreSQL 容器配置
├── backend/                # Python 后端
│   ├── .env                # 数据库连接（勿提交到 Git）
│   ├── .env.example
│   ├── requirements.txt
│   ├── .gitignore
|   └── app/
│       ├── main.py
│       ├── config.py
│       ├── database.py
│       ├── models.py
│       ├── schemas.py
│       ├── crud.py
│       └── routers/
│           └── tasks.py
└── frontend/               # React 前端
    ├── package.json
    ├── vite.config.js
    ├── tailwind.config.js
    ├── index.html
    └── src/
        ├── main.jsx
        ├── App.jsx
        ├── index.css
        ├── api/
        │   └── client.js   # 统一 API 请求封装
        ├── hooks/
        │   └── useApi.js
        └── components/
            ├── TaskList.jsx
            ├── TaskForm.jsx
            ├── TaskItem.jsx
            ├── TaskFilter.jsx
            └── ui/
                ├── Loading.jsx
                └── ErrorAlert.jsx

```

<details>

<summary>点击展开（目录结构解析）</summary>

这段目录结构清晰地展示了 **前后端分离** 的项目架构。为了帮助初学者更好地理解，我将对关键文件和文件夹的作用进行详细解读：


### 📂 根目录 (`todo_app/`)


*   `docker-compose.yml`: **一键启动数据库**。它定义了 PostgreSQL 容器的配置（端口、密码、数据卷）。运行 `docker-compose up -d` 即可在本地启动数据库，无需手动安装 PostgreSQL 软件。

---


### 🐍 后端目录 (`backend/`)


这是基于 **FastAPI** 的 Python 项目，遵循了常见的分层架构：

#### 1. 配置与依赖


*   `.env`: **环境变量文件**。存储敏感信息（如 `DATABASE_URL`），**严禁提交到 Git**。
*   `.env.example`: **模板文件**。告诉其他开发者需要配置哪些变量，但不包含真实密码。
*   `requirements.txt`: **Python 依赖列表**。运行 `pip install -r requirements.txt` 即可安装所有需要的库（FastAPI, SQLAlchemy, asyncpg 等）。


#### 2. 核心代码 (`app/`)


*   `main.py`: **入口文件**。创建 FastAPI 实例 `app`，并注册路由（Routers）和中间件。
*   `config.py`: **配置加载**。使用 `pydantic-settings` 读取 `.env` 中的配置，确保类型安全。
*   `database.py`: **数据库连接**。初始化 SQLAlchemy 的异步引擎 (`create_async_engine`) 和会话工厂 (`async_sessionmaker`)。


#### 3. 数据层 (Model & Schema)


*   `models.py`: **数据库模型**。定义 SQLAlchemy 的表结构（ORM 类），对应 PostgreSQL 中的 `tasks` 表。
*   `schemas.py`: **数据验证模型**。定义 Pydantic 类，用于 API 请求体的验证（如新建任务时标题不能为空）和响应数据的格式化。


#### 4. 业务逻辑层


*   `crud.py`: **数据库操作封装**。CRUD (Create, Read, Update, Delete) 的具体实现。将数据库查询逻辑与路由分离，保持代码整洁。
    *   例如：`get_tasks(db, skip, limit)`，`create_task(db, task)`。


#### 5. 接口层 (API Routes)


*   `routers/tasks.py`: **API 端点定义**。定义具体的 HTTP 接口（如 `GET /tasks/`, `POST /tasks/`）。
    *   它接收请求 -> 调用 `crud.py` 处理数据 -> 返回结果给前端。

---


### ⚛️ 前端目录 (`frontend/`)


这是基于 **React + Vite** 的项目：


#### 1. 配置文件


*   `package.json`: **Node.js 依赖管理**。记录了 React, Tailwind, Axios 等库的版本。
*   `vite.config.js`: **Vite 配置**。可以配置代理（Proxy），解决开发时的跨域问题（将 `/api` 请求转发到后端 `http://localhost:8000`）。
*   `tailwind.config.js`: **Tailwind 配置**。自定义主题颜色、字体或扩展工具类。


#### 2. 源代码 (`src/`)


*   `main.jsx`: **应用入口**。渲染根组件 `<App />` 到 HTML 中。
*   `App.jsx`: **根组件**。通常包含路由配置（如果使用 React Router）或全局布局。
*   `index.css`: **全局样式**。引入 Tailwind 的基础指令 (`@tailwind base;` 等)。


#### 3. 功能模块


*   `api/client.js`: **HTTP 客户端封装**。通常使用 `axios` 创建实例，统一设置 baseURL 和拦截器（如自动携带 Token）。
*   `hooks/useApi.js`: **自定义 Hook**。封装通用的 API 请求逻辑，处理 loading 状态和错误捕获，让组件代码更简洁。


#### 4. 组件 (`components/`)


*   `TaskList.jsx`: **任务列表容器**。负责获取数据并渲染多个 `TaskItem`。
*   `TaskForm.jsx`: **新建/编辑表单**。包含输入框和提交按钮。
*   `TaskItem.jsx`: **单个任务项**。显示标题、描述、复选框和删除按钮。
*   `TaskFilter.jsx`: **筛选器**。提供“全部/进行中/已完成”的切换按钮。
*   `ui/`: **通用 UI 组件**。如加载动画 `Loading.jsx` 和错误提示 `ErrorAlert.jsx`，可在多处复用。

---


### 💡 给初学者的建议


1.  **从后端开始**：先理解 `models.py` 和 `schemas.py` 的区别，然后看 `crud.py` 如何操作数据库，最后看 `routers/tasks.py` 如何暴露接口。
2.  **再学前端**：先看 `api/client.js` 如何请求后端，然后看 `TaskList.jsx` 如何展示数据。
3.  **联调**：确保后端运行在 `8000` 端口，前端运行在 `5173` 端口，并通过 Vite 代理或 CORS 解决跨域问题。

</details>

---


## 2.环境准备


### 2.1 需要安装的软件


| 软件           | 推荐版本     | 用途           |
| -------------- | ------------ | -------------- |
| python         | 3.11或者更高 | 后端开发       |
| Node.js        | 18.x或者更高 | 运行前端       |
| Docker Desktop | 最新稳定版   | 运行PostgreSQL |
| Git            | 最新稳定版   | 代码管理       |


### 2.2 验证安装


Windows (PowerShell) 或 macOS/Linux (bash)：

```bash

# 验证 Python版本 3.11或者更高
python --version

# 验证 Node.js版本 18.x或者更高
node -v

npm -version

# 验证 Docker Desktop版本
docker --version

docker compose version

```

确保以上命令都返回正确的版本信息，否则请按照官方文档进行安装。


### 2.3 PostgreSQL Docker容器说明


本项目不在本机直接安装PostgreSQL，而是使用Docker容器（Docker容器是一个轻量级的虚拟环境，可以快速启动、运行和停止，并且不会影响本地环境）`docker-compose.yml`一键启动。

| 配置项   | 值                   |
| -------- | -------------------- |
| 容器名   | `todo_postgres`      |
| 镜像     | `postgres:16-alpine` |
| 主机端口 | `5432`               |
| 数据库名 | `todo_db`            |
| 用户名   | `postgres`           |
| 密码     | `your_password`      |

后端连接数据库的配置在 `backend/.env` 文件中，请根据实际情况修改。

```dotenv

DATABASE_URL=postgresql+asyncpg://postgres:your_password@localhost:5432/todo_db

```

---


## 3. 后端项目结构与核心代码


### 3.1 主要文件说明


| 文件               | 说明               | 用途                                                  |
| ------------------ | ------------------ | ----------------------------------------------------- |
| `config.py`        | 配置文件           | 从`.env`文件中读取配置(如：数据库连接信息，API密钥等) |
| `database.py`      | 数据库连接文件     | 创建异步数据库引擎、会话等                            |
| `models.py`        | 数据库模型文件     | 定义数据库表结构 SQLAlchemy表模型                     |
| `schemas.py`       | 数据验证模型文件   | 定义数据验证模型 Pydantic请求/响应                    |
| `crud.py`          | 数据库操作封装文件 | 数据库操作业务逻辑(增删改查)                          |
| `routers/tasks.py` | API接口定义文件    | 定义API接口路由，如Get获取任务列表、Post创建任务等    |
| `main.py`          | FastAPI入口文件    | 定义FastAPI应用实例，并注册路由                       |


### 3.2 主要代码片段


#### 3.2.1 python项目依赖清单文件`requirements.txt`


本项目的依赖包清单[requirements.txt](/todo-doc/requirements.txt/)：
```txt

fastapi>=0.115.0
uvicorn[standard]>=0.32.0
sqlalchemy[asyncio]>=2.0.36
asyncpg>=0.30.0
pydantic-settings>=2.6.0
python-dotenv>=1.0.1

```

#### 3.2.2 配置文件`config.py`


本项目的配置文件[config.py](/todo-doc/config.py/)：
```python

from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8")

    database_url: str = (
        "postgresql+asyncpg://postgres:your_password@localhost:5432/todo_db"
    )


settings = Settings()

```


#### 3.2.3 数据库连接文件`database.py`


本项目的数据库连接文件[database.py](/todo-doc/database.py/):

<details>

<summary>点击展开（详细代码）</summary>

```python

from collections.abc import AsyncGenerator

from sqlalchemy import text
from sqlalchemy.ext.asyncio import AsyncSession, async_sessionmaker, create_async_engine
from sqlalchemy.orm import DeclarativeBase

from app.config import settings

engine = create_async_engine(settings.database_url, echo=False)
async_session_maker = async_sessionmaker(
    engine, class_=AsyncSession, expire_on_commit=False
)


class Base(DeclarativeBase):
    pass


async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with async_session_maker() as session:
        yield session


async def check_db_connection() -> bool:
    try:
        async with engine.connect() as conn:
            await conn.execute(text("SELECT 1"))
        return True
    except Exception:
        return False

```

</details>


#### 3.2.4 数据库模型文件`models.py`

本项目的数据库模型文件[models.py](/todo-doc/models.py/)：

<details>

<summary>点击展开（详细代码）</summary>

```python

from datetime import datetime

from sqlalchemy import Boolean, DateTime, String, Text, func
from sqlalchemy.orm import Mapped, mapped_column

from app.database import Base


class Task(Base):
    __tablename__ = "tasks"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    title: Mapped[str] = mapped_column(String(255), nullable=False)
    description: Mapped[str | None] = mapped_column(Text, nullable=True)
    completed: Mapped[bool] = mapped_column(Boolean, default=False, nullable=False)
    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), server_default=func.now(), nullable=False
    )

```

</details>


#### 3.2.5 数据验证校验模型文件`schemas.py`


本项目的数据验证校验模型文件为[schemas.py](/todo-doc/schemas.py/)，代码如下：

<details>

<summary>点击展开（详细代码）</summary>

```python

from datetime import datetime

from pydantic import BaseModel, ConfigDict, Field


class TaskBase(BaseModel):
    title: str = Field(..., min_length=1, max_length=255)
    description: str | None = None
    completed: bool = False


class TaskCreate(TaskBase):
    pass


class TaskUpdate(BaseModel):
    title: str | None = Field(None, min_length=1, max_length=255)
    description: str | None = None
    completed: bool | None = None


class TaskRead(TaskBase):
    model_config = ConfigDict(from_attributes=True)

    id: int
    created_at: datetime

```

</details>


#### 3.2.6 数据库操作封装文件`crud.py`


本项目的数据库操作封装文件[crud.py](/todo-doc/crud.py/)代码如下：

<details>

<summary>点击展开（详细代码）</summary>

```python

from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.models import Task
from app.schemas import TaskCreate, TaskUpdate


async def get_tasks(db: AsyncSession) -> list[Task]:
    result = await db.execute(select(Task).order_by(Task.created_at.desc()))
    return list(result.scalars().all())


async def get_task(db: AsyncSession, task_id: int) -> Task | None:
    return await db.get(Task, task_id)


async def create_task(db: AsyncSession, task_in: TaskCreate) -> Task:
    task = Task(**task_in.model_dump())
    db.add(task)
    await db.commit()
    await db.refresh(task)
    return task


async def update_task(db: AsyncSession, task: Task, task_in: TaskUpdate) -> Task:
    data = task_in.model_dump(exclude_unset=True)
    for key, value in data.items():
        setattr(task, key, value)
    await db.commit()
    await db.refresh(task)
    return task


async def delete_task(db: AsyncSession, task: Task) -> None:
    await db.delete(task)
    await db.commit()

```

</details>


#### 3.2.7 API接口定义文件`routers/tasks.py`


本项目的API接口定义文件为[routers/tasks.py](/todo-doc/routers.tasks.py/)，该文件定义了项目的所有接口，包括获取所有任务、获取单个任务、创建任务、更新任务和删除任务。

<details>

<summary>点击展开（详细代码）</summary>

```python

from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession

from app import crud
from app.database import get_db
from app.schemas import TaskCreate, TaskRead, TaskUpdate

router = APIRouter(prefix="/tasks", tags=["tasks"])


@router.get("", response_model=list[TaskRead])
async def list_tasks(db: AsyncSession = Depends(get_db)) -> list[TaskRead]:
    return await crud.get_tasks(db)


@router.post("", response_model=TaskRead, status_code=status.HTTP_201_CREATED)
async def create_task(
    task_in: TaskCreate, db: AsyncSession = Depends(get_db)
) -> TaskRead:
    return await crud.create_task(db, task_in)


@router.put("/{task_id}", response_model=TaskRead)
async def update_task(
    task_id: int, task_in: TaskUpdate, db: AsyncSession = Depends(get_db)
) -> TaskRead:
    task = await crud.get_task(db, task_id)
    if task is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND, detail="Task not found"
        )
    return await crud.update_task(db, task, task_in)


@router.delete("/{task_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_task(task_id: int, db: AsyncSession = Depends(get_db)) -> None:
    task = await crud.get_task(db, task_id)
    if task is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND, detail="Task not found"
        )
    await crud.delete_task(db, task)

```

</details>


#### 3.2.8 应用入口文件`main.py`


[main.py](/todo-doc/main.py/)是项目的入口文件，它负责启动整个应用程序。以下是`main.py`文件的内容：

<details>

<summary>点击展开（详细代码）</summary>

```python

from contextlib import asynccontextmanager

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from app.database import Base, check_db_connection, engine
from app.routers import tasks


@asynccontextmanager
async def lifespan(app: FastAPI):
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield
    await engine.dispose()


app = FastAPI(title="Todo API", lifespan=lifespan)

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:5173",
        "http://127.0.0.1:5173",
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(tasks.router)


@app.get("/health")
async def health():
    return {"status": "ok", "message": "API is running"}


@app.get("/db-check")
async def db_check():
    connected = await check_db_connection()
    return {
        "database": "connected" if connected else "disconnected",
        "ok": connected,
    }

```

</details>


#### 3.2.9 后端环境变量模板`backend/.env.example`


本项目的后端环境变量模板[.env.example](/todo-doc/.env.example/)。
```dotenv

# 数据库连接配置
# 格式: postgresql+asyncpg://用户名:密码@主机:端口/数据库名
DATABASE_URL=postgresql+asyncpg://postgres:your_password@localhost:5432/todo_db

# 可选：应用调试模式 (True/False)
DEBUG=True

# 可选：秘密密钥 (用于 JWT 签名等，生产环境务必修改)
SECRET_KEY=change_this_to_a_random_string_in_production

```

> Windows 注意：请用 UTF-8 无 BOM 保存 .env，否则 DATABASE_URL 可能无法被正确读取。

---


## 4.前端项目与核心代码


### 4.1 主要文件说明


| 文件 | 作用 |
|------|------|
| `main.jsx` | React 入口，挂载根组件 |
| `App.jsx` | 页面布局，渲染 `TaskList` |
| `api/client.js` | 封装 `fetch`，统一错误处理 |
| `components/TaskList.jsx` | 列表加载、筛选、删除、完成切换 |
| `components/TaskForm.jsx` | 新建任务表单 |
| `components/TaskItem.jsx` | 单条任务卡片 |
| `components/TaskFilter.jsx` | 全部 / 进行中 / 已完成 筛选 |
| `hooks/useApi.js` | 请求 loading / error 状态 |
| `components/ui/Loading.jsx` | 加载动画 |
| `components/ui/ErrorAlert.jsx` | 错误提示 |


#### 4.1.1 `App.jsx` — 根组件


[`App.jsx`](/todo-doc/App.jsx/) 是项目的根组件，它负责渲染 `TaskList` 组件，并监听 URL 参数变化，以切换任务列表的筛选条件。

<details>

<summary>点击展开（详细代码）</summary>

```javascript

import TaskList from "./components/TaskList.jsx";

export default function App() {
  return (
    <div className="min-h-screen bg-slate-900 text-slate-100 flex items-start justify-center p-6 py-10">
      <div className="w-full max-w-2xl rounded-2xl border border-slate-700 bg-slate-800/80 shadow-xl p-8">
        <header className="text-center mb-8">
          <h1 className="text-3xl font-bold">Todo App</h1>
        </header>

        <TaskList />
      </div>
    </div>
  );
}

```

</details>


#### 4.1.2 `api/client.js` — 统一 API 客户端


> 文档中常称此文件为「api 层」；项目路径为 `src/api/client.js`。

[`api/client.js`](/todo-doc/client.js/) 是项目的统一 API 客户端，它封装了 `fetch`，统一错误处理，并支持请求 loading / error 状态。

<details>

<summary>点击展开（详细代码）</summary>

```javascript

export const API_BASE = import.meta.env.VITE_API_URL || "http://localhost:8000";

export async function parseApiError(res) {
  try {
    const data = await res.json();
    if (Array.isArray(data.detail)) {
      return data.detail.map((d) => d.msg || String(d)).join("；");
    }
    if (typeof data.detail === "string") {
      return data.detail;
    }
    if (data.detail) {
      return JSON.stringify(data.detail);
    }
  } catch {
    // ignore parse errors
  }
  return null;
}

export async function apiRequest(path, { method = "GET", body, headers = {} } = {}) {
  const url = `${API_BASE}${path}`;
  const init = { method, headers: { ...headers } };

  if (body !== undefined) {
    init.headers["Content-Type"] = "application/json";
    init.body = JSON.stringify(body);
  }

  let res;
  try {
    res = await fetch(url, init);
  } catch {
    throw new Error("网络错误，请确认后端已启动且地址正确");
  }

  if (!res.ok) {
    const detail = await parseApiError(res);
    throw new Error(detail || `请求失败 (${res.status})`);
  }

  if (res.status === 204) {
    return null;
  }

  const text = await res.text();
  if (!text) {
    return null;
  }
  return JSON.parse(text);
}

````

</details>


#### 4.1.3 `components/TaskList.jsx` — 任务列表组件

[`components/TaskList.jsx`](/todo-doc/TaskList.jsx/) 是任务列表组件，它负责加载、筛选、删除、完成切换任务。

<details>

<summary>点击展开（详细代码）</summary>

```javascript JSX

import { useCallback, useEffect, useMemo, useState } from "react";
import { apiRequest } from "../api/client.js";
import TaskFilter, { filterTasks } from "./TaskFilter.jsx";
import TaskForm from "./TaskForm.jsx";
import TaskItem from "./TaskItem.jsx";
import ErrorAlert from "./ui/ErrorAlert.jsx";
import Loading from "./ui/Loading.jsx";

export default function TaskList() {
  const [tasks, setTasks] = useState([]);
  const [filter, setFilter] = useState("all");
  const [loading, setLoading] = useState(true);
  const [refreshing, setRefreshing] = useState(false);
  const [error, setError] = useState(null);
  const [actionError, setActionError] = useState(null);
  const [togglingId, setTogglingId] = useState(null);
  const [deletingId, setDeletingId] = useState(null);

  const filterCounts = useMemo(
    () => ({
      all: tasks.length,
      active: tasks.filter((t) => !t.completed).length,
      completed: tasks.filter((t) => t.completed).length,
    }),
    [tasks]
  );

  const filteredTasks = useMemo(
    () => filterTasks(tasks, filter),
    [tasks, filter]
  );

  const fetchTasks = useCallback(async (silent = false) => {
    if (silent) {
      setRefreshing(true);
    } else {
      setLoading(true);
    }
    setError(null);

    try {
      const data = await apiRequest("/tasks");
      setTasks(Array.isArray(data) ? data : []);
    } catch (err) {
      setError(err.message || "无法加载任务列表");
      if (!silent) {
        setTasks([]);
      }
    } finally {
      if (silent) {
        setRefreshing(false);
      } else {
        setLoading(false);
      }
    }
  }, []);

  useEffect(() => {
    fetchTasks();
  }, [fetchTasks]);

  async function toggleCompleted(task) {
    const nextCompleted = !task.completed;
    setActionError(null);
    setTogglingId(task.id);
    setTasks((prev) =>
      prev.map((t) =>
        t.id === task.id ? { ...t, completed: nextCompleted } : t
      )
    );

    try {
      const updated = await apiRequest(`/tasks/${task.id}`, {
        method: "PUT",
        body: { completed: nextCompleted },
      });
      setTasks((prev) =>
        prev.map((t) => (t.id === task.id ? updated : t))
      );
    } catch (err) {
      setTasks((prev) =>
        prev.map((t) =>
          t.id === task.id ? { ...t, completed: task.completed } : t
        )
      );
      setActionError(err.message || "更新任务状态失败");
    } finally {
      setTogglingId(null);
    }
  }

  async function deleteTask(taskId) {
    setActionError(null);
    setDeletingId(taskId);

    try {
      await apiRequest(`/tasks/${taskId}`, { method: "DELETE" });
      setTasks((prev) => prev.filter((t) => t.id !== taskId));
    } catch (err) {
      setActionError(err.message || "删除任务失败");
    } finally {
      setDeletingId(null);
    }
  }

  const listBusy = loading || refreshing;

  return (
    <div className="space-y-6">
      <TaskForm onSuccess={() => fetchTasks(true)} />

      <div className="space-y-4">
        <div className="flex items-center justify-between gap-3">
          <h2 className="text-lg font-semibold text-slate-200">任务列表</h2>
          <button
            type="button"
            onClick={() => fetchTasks(true)}
            disabled={listBusy}
            className="text-sm text-sky-400 hover:text-sky-300 transition-colors disabled:opacity-50 flex items-center gap-2"
          >
            {refreshing && (
              <span className="inline-block h-3 w-3 animate-spin rounded-full border-2 border-slate-500 border-t-sky-400" />
            )}
            {refreshing ? "刷新中…" : "刷新"}
          </button>
        </div>

        <ErrorAlert
          title="操作失败"
          message={actionError}
          onRetry={() => setActionError(null)}
          retryLabel="关闭"
        />

        {!loading && !error && tasks.length > 0 && (
          <TaskFilter
            value={filter}
            onChange={setFilter}
            counts={filterCounts}
          />
        )}

        {loading && (
          <div className="rounded-xl border border-slate-600 bg-slate-700/30 p-8">
            <Loading message="正在加载任务…" />
          </div>
        )}

        {!loading && error && (
          <ErrorAlert
            title="加载失败"
            message={error}
            onRetry={() => fetchTasks()}
          />
        )}

        {!loading && !error && refreshing && (
          <Loading message="正在刷新列表…" className="py-2" />
        )}

        {!loading && !error && tasks.length === 0 && (
          <div className="rounded-xl border border-dashed border-slate-600 bg-slate-700/20 p-10 text-center">
            <p className="text-slate-400">暂无任务</p>
            <p className="text-xs text-slate-500 mt-2">在上方表单添加第一条任务</p>
          </div>
        )}

        {!loading && !error && tasks.length > 0 && filteredTasks.length === 0 && (
          <div className="rounded-xl border border-dashed border-slate-600 bg-slate-700/20 p-8 text-center">
            <p className="text-slate-400">当前筛选下暂无任务</p>
          </div>
        )}

        {!loading && !error && filteredTasks.length > 0 && (
          <ul
            className={`grid gap-4 sm:grid-cols-1 ${
              refreshing ? "opacity-60 pointer-events-none" : ""
            }`}
          >
            {filteredTasks.map((task) => (
              <TaskItem
                key={task.id}
                task={task}
                onToggle={toggleCompleted}
                onDelete={deleteTask}
                toggling={togglingId === task.id}
                deleting={deletingId === task.id}
              />
            ))}
          </ul>
        )}
      </div>
    </div>
  );
}

````

</details>


#### 4.1.4 `hooks/useApi.js` - 请求 loading / error 状态


[hooks/useApi.is](/todo-doc/useApi.js/) 是一个 自定义 React Hook，它的核心职责是封装异步请求的通用状态管理逻辑（Loading 和 Error）。

<details>

<summary>点击展开（详细代码）</summary>

```javascript

import { useCallback, useState } from "react";

/**
 * Wraps an async API call with loading + error state.
 */
export function useApi(defaultErrorMessage = "请求失败") {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const clearError = useCallback(() => setError(null), []);

  const run = useCallback(
    async (fn, { fallbackMessage } = {}) => {
      setLoading(true);
      setError(null);
      try {
        return await fn();
      } catch (err) {
        const msg = err.message || fallbackMessage || defaultErrorMessage;
        setError(msg);
        throw err;
      } finally {
        setLoading(false);
      }
    },
    [defaultErrorMessage]
  );

  return { loading, error, run, clearError, setError };
}

```

</details>


#### 4.1.5 `components/TaskItem.jsx` - 列表项组件


[components/TaskItem.jsx](/todo-doc/TaskItem.jsx/) 是一个 React 组件，用于展示一个任务项。

<details>

<summary>点击展开（详细代码）</summary>

```javascript

function formatDate(iso) {
  try {
    return new Date(iso).toLocaleString("zh-CN", {
      dateStyle: "medium",
      timeStyle: "short",
    });
  } catch {
    return iso;
  }
}

export default function TaskItem({
  task,
  onToggle,
  onDelete,
  toggling,
  deleting,
}) {
  const busy = toggling || deleting;

  return (
    <li
      className={`rounded-xl border p-5 shadow-lg transition-shadow hover:shadow-xl ${
        task.completed
          ? "border-emerald-800/50 bg-emerald-950/30"
          : "border-slate-600 bg-slate-700/40"
      } ${busy ? "opacity-70" : ""}`}
    >
      <div className="flex items-start gap-3">
        <div className="mt-0.5 flex flex-col items-center gap-1">
          <input
            type="checkbox"
            checked={task.completed}
            onChange={() => onToggle(task)}
            disabled={busy}
            aria-label={`标记「${task.title}」为${task.completed ? "未完成" : "已完成"}`}
            className="h-5 w-5 shrink-0 cursor-pointer rounded border-slate-500 bg-slate-800 text-sky-500 focus:ring-2 focus:ring-sky-500 focus:ring-offset-0 disabled:cursor-not-allowed"
          />
          {toggling && (
            <span
              className="inline-block h-3 w-3 animate-spin rounded-full border-2 border-slate-500 border-t-sky-400"
              aria-hidden
            />
          )}
        </div>

        <div className="min-w-0 flex-1">
          <div className="flex items-start justify-between gap-3">
            <h3
              className={`text-lg font-semibold ${
                task.completed
                  ? "text-slate-400 line-through"
                  : "text-slate-100"
              }`}
            >
              {task.title}
            </h3>
            <span
              className={`shrink-0 rounded-full px-2.5 py-0.5 text-xs font-medium ${
                task.completed
                  ? "bg-emerald-900/60 text-emerald-300 border border-emerald-700"
                  : "bg-amber-900/50 text-amber-200 border border-amber-700/60"
              }`}
            >
              {task.completed ? "已完成" : "进行中"}
            </span>
          </div>

          {task.description && (
            <p className="mt-2 text-sm text-slate-400 leading-relaxed">
              {task.description}
            </p>
          )}

          <div className="mt-4 flex flex-wrap items-center gap-x-4 gap-y-1 text-xs text-slate-500">
            <span>ID: {task.id}</span>
            <span>创建于 {formatDate(task.created_at)}</span>
          </div>
        </div>

        <button
          type="button"
          onClick={() => onDelete(task.id)}
          disabled={busy}
          className="shrink-0 rounded-lg border border-red-800/60 bg-red-950/40 px-3 py-1.5 text-sm text-red-300 hover:bg-red-900/50 hover:text-red-200 focus:outline-none focus:ring-2 focus:ring-red-500 disabled:cursor-not-allowed disabled:opacity-50 transition-colors flex items-center gap-1.5"
        >
          {deleting && (
            <span className="inline-block h-3 w-3 animate-spin rounded-full border-2 border-red-400/50 border-t-red-300" />
          )}
          {deleting ? "删除中…" : "删除"}
        </button>
      </div>
    </li>
  );
}

```

</details>


### 4.1.6 `components/TaskForm.jsx` - 任务表单组件


[components/TaskForm.jsx](/todo-doc/TaskForm.jsx/) 是一个 React 组件，用于创建一个任务项。

<details>

<summary>点击展开（详细代码）</summary>

```javascript

import { useState } from "react";
import { apiRequest } from "../api/client.js";
import { useApi } from "../hooks/useApi.js";
import ErrorAlert from "./ui/ErrorAlert.jsx";
import Loading from "./ui/Loading.jsx";

export default function TaskForm({ onSuccess }) {
  const [title, setTitle] = useState("");
  const [description, setDescription] = useState("");
  const [validationError, setValidationError] = useState(null);
  const { loading, error, run, clearError } = useApi("创建任务失败");

  async function handleSubmit(e) {
    e.preventDefault();
    const trimmedTitle = title.trim();
    if (!trimmedTitle) {
      setValidationError("标题不能为空");
      return;
    }
    setValidationError(null);

    try {
      await run(
        () =>
          apiRequest("/tasks", {
            method: "POST",
            body: {
              title: trimmedTitle,
              description: description.trim() || null,
              completed: false,
            },
          }),
        { fallbackMessage: "创建任务失败" }
      );
      setTitle("");
      setDescription("");
      onSuccess?.();
    } catch {
      // error handled by useApi
    }
  }

  const displayError = validationError || error;

  return (
    <form
      onSubmit={handleSubmit}
      className="rounded-xl border border-slate-600 bg-slate-700/40 p-5 shadow-lg space-y-4"
    >
      <h2 className="text-lg font-semibold text-slate-200">新建任务</h2>

      <div>
        <label
          htmlFor="task-title"
          className="block text-sm font-medium text-slate-400 mb-1.5"
        >
          标题 <span className="text-red-400">*</span>
        </label>
        <input
          id="task-title"
          type="text"
          value={title}
          onChange={(e) => {
            setValidationError(null);
            clearError();
            setTitle(e.target.value);
          }}
          placeholder="输入任务标题"
          maxLength={255}
          disabled={loading}
          className="w-full rounded-lg border border-slate-600 bg-slate-800 px-3 py-2 text-slate-100 placeholder:text-slate-500 focus:border-sky-500 focus:outline-none focus:ring-1 focus:ring-sky-500 disabled:opacity-60"
        />
      </div>

      <div>
        <label
          htmlFor="task-description"
          className="block text-sm font-medium text-slate-400 mb-1.5"
        >
          描述
        </label>
        <textarea
          id="task-description"
          value={description}
          onChange={(e) => {
            clearError();
            setDescription(e.target.value);
          }}
          placeholder="可选：任务说明"
          rows={3}
          disabled={loading}
          className="w-full resize-y rounded-lg border border-slate-600 bg-slate-800 px-3 py-2 text-slate-100 placeholder:text-slate-500 focus:border-sky-500 focus:outline-none focus:ring-1 focus:ring-sky-500 disabled:opacity-60"
        />
      </div>

      <ErrorAlert title="创建失败" message={displayError} />

      {loading && <Loading message="正在创建任务…" />}

      <button
        type="submit"
        disabled={loading}
        className="w-full rounded-lg bg-sky-600 px-4 py-2.5 font-medium text-white hover:bg-sky-500 focus:outline-none focus:ring-2 focus:ring-sky-400 focus:ring-offset-2 focus:ring-offset-slate-800 disabled:cursor-not-allowed disabled:opacity-60 transition-colors"
      >
        {loading ? "提交中…" : "添加任务"}
      </button>
    </form>
  );
}

```

</details>

### 4.1.7 `ui/ErrorAlert.jsx` - 错误提示组件


[ui/ErrorAlert.jsx](/todo-doc/ui.ErrorAlert.jsx/) 是一个 React 组件，用于显示错误提示。

<details>

<summary>点击展开（详细代码）</summary>

```javascript

export default function ErrorAlert({
  title = "请求失败",
  message,
  hint,
  onRetry,
  retryLabel = "重试",
  className = "",
}) {
  if (!message && !hint) {
    return null;
  }

  return (
    <div
      className={`rounded-lg border border-red-700/60 bg-red-900/30 p-4 text-red-200 ${className}`}
      role="alert"
    >
      <p className="font-semibold">{title}</p>
      {message && <p className="text-sm mt-1">{message}</p>}
      {hint && <p className="text-xs mt-3 text-red-300/80">{hint}</p>}
      {onRetry && (
        <button
          type="button"
          onClick={onRetry}
          className="mt-3 text-sm text-red-300 hover:text-red-100 underline"
        >
          {retryLabel}
        </button>
      )}
    </div>
  );
}

```

</details>


### 4.1.8 `main.jsx` - 应用入口文件


[main.jsx](/todo-doc/main.jsx/) 是一个 React 应用的入口文件，用于启动应用程序。

```jsx

import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App.jsx";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

```


### 4.1.9 `ui/Loading.jsx` - 加载组件

[ui/Loading.jsx](/todo-doc/ui.Loading.jsx/) 是一个 React 组件，用于显示加载状态。

<details>

<summary>点击展开（详细代码）</summary>

```jsx

export default function Loading({ message = "加载中…", className = "" }) {
  return (
    <div
      className={`flex items-center justify-center gap-2 text-slate-400 ${className}`}
      role="status"
      aria-live="polite"
    >
      <span
        className="inline-block h-4 w-4 animate-spin rounded-full border-2 border-slate-500 border-t-sky-400"
        aria-hidden
      />
      <span className="text-sm">{message}</span>
    </div>
  );
}

```

</details>


### 4.2 前端环境变量（可选）`frontend/.env`


[frontend/.env](/todo-doc/frontend.env/) 是一个用于配置前端应用程序的配置文件。

```env

VITE_API_URL=http://localhost:8000

```

不配置时默认请求 `http://localhost:8000`。


### 4.3 启动前端服务`frontend/vite.config.js`


[frontend/vite.config.js](/todo-doc/frontend.vite.config.js/) 是一个 Vite 配置文件，用于启动前端应用程序。

```javascript

import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    proxy: {
      "/api": {
        target: "http://localhost:8000",
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ""),
      },
    },
  },
});

```

### 4.4 `docker-compose.yml` - 数据库容器

[docker-compose.yml](/todo-doc/docker-compose.yml/) 是一个 Docker Compose 文件，用于启动后端和前端应用程序。

<details>

<summary>点击展开（详细代码）</summary>

```yaml

services:
  postgres:
    image: postgres:16-alpine
    container_name: todo_postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: your_password
      POSTGRES_DB: todo_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d todo_db"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s

volumes:
  postgres_data:

```

</details>


### 4.5 [跨域（CORS）说明](/todo-doc/CORS/)


- 开发时前端地址：`http://localhost:5173`
- 后端在 `main.py` 中已允许该来源跨域
- `vite.config.js` 还配置了 `/api` 代理（可选），将请求转发到 `8000` 端口

---


## 5. [数据库模型设计与 API 端点列表](/todo-doc/ModelandAPI/)

### 5.1 表结构：`tasks`

| 字段 | 类型 | 约束 | 说明 |
|------|------|------|------|
| `id` | INTEGER | 主键、自增 | 任务唯一 ID |
| `title` | VARCHAR(255) | NOT NULL | 标题 |
| `description` | TEXT | 可空 | 描述 |
| `completed` | BOOLEAN | NOT NULL，默认 `false` | 是否完成 |
| `created_at` | TIMESTAMPTZ | NOT NULL，默认当前时间 | 创建时间 |

### 5.2 API 端点列表

| 方法 | 路径 | 说明 | 请求体示例 | 成功响应 |
|------|------|------|------------|----------|
| GET | `/health` | 服务健康检查 | 无 | `{"status":"ok","message":"API is running"}` |
| GET | `/db-check` | 数据库连接检查 | 无 | `{"database":"connected","ok":true}` |
| GET | `/tasks` | 获取全部任务 | 无 | 任务数组 JSON |
| POST | `/tasks` | 创建任务 | 见下 | 单条任务 JSON |
| PUT | `/tasks/{id}` | 更新任务 | 见下 | 单条任务 JSON |
| DELETE | `/tasks/{id}` | 删除任务 | 无 | `204 No Content` |

**POST `/tasks` 请求体：**

```json
{
  "title": "学习 FastAPI",
  "description": "完成第一章",
  "completed": false
}
```

**PUT `/tasks/{id}` 请求体（字段均可选，只更新传入的字段）：**

```json
{
  "title": "新标题",
  "description": "新描述",
  "completed": true
}
```

**任务对象响应示例：**

```json
{
  "id": 1,
  "title": "学习 FastAPI",
  "description": "完成第一章",
  "completed": false,
  "created_at": "2026-06-03T08:00:00.000000Z"
}
```

### 5.3 交互式 API 文档

后端启动后访问：

- Swagger UI：<http://localhost:8000/docs>
- ReDoc：<http://localhost:8000/redoc>

---


## 6. 完整安装与运行步骤


以下步骤假设你把项目放在 `todo_app` 文件夹中。请根据你的实际路径替换。


### 步骤 0：获取项目代码


**方式 A：Git 克隆（若有远程仓库）**

```bash

git clone <你的仓库地址> todo_app
cd todo_app

```

**方式 B：已有本地文件夹**

```bash

cd todo_app

```

确认目录中存在 `backend/`、`frontend/`、`docker-compose.yml`。

---


### 步骤 1：启动 PostgreSQL（Docker）


在项目根目录（含 `docker-compose.yml` 的目录）执行：

```bash

docker compose up -d

```

查看容器状态（应显示 `healthy`）：

```bash

docker compose ps

```

查看日志（排错时用）：

```bash

docker compose logs -f postgres

```

---


### 步骤 2：配置并启动后端


#### 2.1 进入后端目录


```bash

cd backend

```


#### 2.2 创建 Python 虚拟环境


**Windows（PowerShell）：**

```powershell

python -m venv .venv
.\.venv\Scripts\Activate.ps1

```

**macOS / Linux：**

```bash

python3 -m venv .venv
source .venv/bin/activate

```

激活成功后，命令行前会出现 `(.venv)`。


#### 2.3 安装依赖


```bash

pip install -r requirements.txt

```


#### 2.4 创建 `.env` 文件

**Windows（PowerShell，无 BOM）：**

```powershell

Copy-Item .env.example .env
# 或手动新建 .env，内容为：
# DATABASE_URL=postgresql+asyncpg://postgres:your_password@localhost:5432/todo_db

```

**macOS / Linux：**

```bash

cp .env.example .env

```

确认 `DATABASE_URL` 中的用户名、密码、库名与 `docker-compose.yml` 一致。


#### 2.5 启动 FastAPI 服务


仍在 `backend` 目录、虚拟环境已激活：

```bash

uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

```

看到 `Uvicorn running on http://0.0.0.0:8000` 即表示成功。

**保持此终端窗口打开。**


#### 2.6 验证后端（新开一个终端）


```bash

curl http://localhost:8000/health
curl http://localhost:8000/db-check
curl http://localhost:8000/tasks

```

浏览器也可打开：<http://localhost:8000/docs>

---


### 步骤 3：配置并启动前端


**新开一个终端**（不要关闭后端终端）。


#### 3.1 进入前端目录


```bash

cd frontend

```

（若从项目根目录：`cd todo_app/frontend`）


#### 3.2 安装 npm 依赖


```bash

npm install

```


#### 3.3（可选）配置 API 地址


创建 `frontend/.env`：


```env

VITE_API_URL=http://localhost:8000

```


#### 3.4 启动开发服务器


```bash

npm run dev

```

终端会显示类似：

```text

  Local:   http://localhost:5173/
  
```


#### 3.5 打开浏览器


访问：<http://localhost:5173>

你应能看到：

1. **新建任务** 表单（标题、描述、添加按钮）
2. **任务列表**（筛选按钮、任务卡片、复选框、删除按钮）

---


### 步骤 4：功能自测清单


| 操作 | 预期结果 |
|------|----------|
| 填写标题并提交 | 列表出现新任务 |
| 点击「进行中 / 已完成」筛选 | 列表按 `completed` 过滤 |
| 勾选复选框 | 任务变为已完成样式 |
| 点击删除 | 任务从列表消失 |
| 点击刷新 | 重新从服务器拉取列表 |

---


### 步骤 5：停止服务


| 服务 | 操作 |
|------|------|
| 前端 | 在前端终端按 `Ctrl + C` |
| 后端 | 在后端终端按 `Ctrl + C` |
| 数据库 | 在项目根目录执行 `docker compose down` |

若要**删除数据库数据**（清空所有任务）：

```bash

docker compose down -v

```

---



## 7. 常见问题与解决方法

### 7.1 数据库连接失败

**现象：**

- 后端启动报错，含 `Connection refused`、`password authentication failed` 等
- `GET /db-check` 返回 `"ok": false`
- 前端提示「无法加载任务列表」或网络错误

**排查步骤：**

1. **确认 Docker 容器在运行**

   ```bash
   docker compose ps
   ```

   状态应为 `Up` 且 `(healthy)`。

2. **确认 `.env` 与 `docker-compose.yml` 一致**

   - 用户：`postgres`
   - 密码：`your_password`（若你改过，两处必须同步）
   - 库名：`todo_db`
   - 端口：`5432`

3. **确认连接串使用异步驱动**

   ```env
   DATABASE_URL=postgresql+asyncpg://postgres:your_password@localhost:5432/todo_db
   ```

   注意是 `postgresql+asyncpg`，不是 `postgresql://` 配同步驱动。

4. **Windows：`.env` 文件不要有 UTF-8 BOM**

   用 VS Code 保存时选择「UTF-8」（无 BOM），或用记事本另存时注意编码。

5. **5432 端口是否被占用**

   若本机已有 PostgreSQL 占用 5432，可修改 `docker-compose.yml`：

   ```yaml
   ports:
     - "5433:5432"
   ```

   同时将 `.env` 改为 `...@localhost:5433/todo_db`。

---

### 7.2 跨域错误（CORS）

**现象：**

浏览器控制台出现类似：

```text
Access to fetch at 'http://localhost:8000/tasks' from origin 'http://localhost:5173'
has been blocked by CORS policy
```

**解决方法：**

1. 确认 `backend/app/main.py` 中 `CORSMiddleware` 包含你的前端地址：

   ```python
   allow_origins=[
       "http://localhost:5173",
       "http://127.0.0.1:5173",
   ],
   ```

2. 若你用 `127.0.0.1:5173` 访问页面，不要用 `localhost:5173` 混用（或反之），保持与 `allow_origins` 一致。

3. 修改 CORS 配置后**重启 uvicorn**。

4. 可选：使用 Vite 代理，在 `frontend/.env` 中设置：

   ```env
   VITE_API_URL=/api
   ```

   请求会经 Vite 转发到后端，减少跨域问题（需 `vite.config.js` 中已配置 proxy）。

---

### 7.3 端口占用

**现象：**

- `Address already in use`（8000 或 5173）
- 浏览器无法访问页面

**解决方法：**

**查占用（Windows）：**

```powershell
netstat -ano | findstr :8000
netstat -ano | findstr :5173
```

**查占用（macOS / Linux）：**

```bash
lsof -i :8000
lsof -i :5173
```

**处理方式：**

- 结束占用进程，或
- 换端口启动，例如：

  ```bash
  uvicorn app.main:app --reload --port 8001
  ```

  并同步修改 `frontend/.env` 中的 `VITE_API_URL`。

---

### 7.4 删除任务后 ID 跳号

**现象：**

创建了 id=1、id=2 的任务，删除 id=1 后，下一条新任务 id 变成 3 而不是 1。

**原因：**

这是 **正常现象**。PostgreSQL 的主键 `id` 使用 **序列（SEQUENCE）** 自增：每插入一条记录，序列加 1，**删除记录不会回收已用过的 id**。

**是否需要修复？**

- **不需要**。生产环境中 id 只要求唯一，不要求连续。
- 若仅用于学习展示，可接受「跳号」；不要手动改序列，除非你知道自己在做什么。

**若必须清空并从 1 重新开始（仅开发环境）：**

```bash
docker compose down -v
docker compose up -d
```

这会删除 Docker 数据卷，**清空所有任务数据**，新插入的任务会从 id=1 开始。

---

### 7.5 其他常见问题速查

| 问题 | 可能原因 | 处理 |
|------|----------|------|
| `ModuleNotFoundError: No module named 'app'` | 未在 `backend` 目录启动 | 先 `cd backend` 再运行 uvicorn |
| 前端页面空白 | 未 `npm install` 或构建报错 | 查看终端报错，重新 `npm install` |
| POST 返回 422 | 标题为空或超长 | 标题 1–255 字符 |
| DELETE 后前端仍显示 | 未刷新状态 | 检查 `deleteTask` 是否从 state 中移除 |
| `pip` 很慢 | 国内网络 | 使用清华等镜像源安装依赖 |

---

## 附录：一键命令速查

```bash
# 1. 数据库
docker compose up -d

# 2. 后端（backend 目录，已激活 venv）
pip install -r requirements.txt
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# 3. 前端（frontend 目录，新终端）
npm install
npm run dev
```

**访问地址：**

- 前端应用：<http://localhost:5173>
- 后端 API 文档：<http://localhost:8000/docs>

---

*文档版本与项目 todo-app 当前实现一致。祝搭建顺利！*