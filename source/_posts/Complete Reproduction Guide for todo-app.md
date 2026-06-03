---
title: Complete Reproduction Guide for todo-app
date: 2026-06-03 17:50:20
tags: [Python Project]
categories: [自学记录]
---

# todo项目复现指南

> 本文面向有一点 Python 和 React 基础的初学者。按顺序完成每一步，即可在本地从零搭建并运行完整的 Todo 全栈应用。

---

<!-- more -->

## 目录

1. [项目简介与技术栈](#1-项目简介与技术栈)
2. [环境准备](#2-环境准备)
3. [后端项目结构与核心代码](#3-后端项目结构与核心代码)
4. [前端项目结构与核心代码](#4-前端项目结构与核心代码)
5. [数据库模型设计与 API 端点列表](#5-数据库模型设计与-api-端点列表)
6. [完整安装与运行步骤](#6-完整安装与运行步骤)
7. [常见问题与解决方法](#7-常见问题与解决方法)

---

## 1. 项目简介与技术栈

### 1.1 项目是什么？

**todo-app** 是一个待办事项（Todo）全栈 Web 应用：

- 在页面上**新建任务**（标题 + 描述）
- **查看任务列表**，支持按「全部 / 进行中 / 已完成」筛选
- **勾选复选框**标记完成（调用后端更新接口）
- **删除任务**

数据保存在 **PostgreSQL** 数据库中，前后端分离部署。

### 1.2 技术栈一览

| 层级 | 技术 | 说明 |
|------|------|------|
| 后端 | Python 3.11+ | 编程语言 |
| 后端框架 | FastAPI | 异步 Web API |
| ORM | SQLAlchemy 2.0（异步） | 操作数据库 |
| 数据库驱动 | asyncpg | 连接 PostgreSQL |
| 数据库 | PostgreSQL 16 | 用 Docker 运行 |
| 前端 | React 18 | UI 框架 |
| 构建工具 | Vite 6 | 开发与打包 |
| 样式 | TailwindCSS 3 | 原子化 CSS |

### 1.3 项目目录结构

```text
todo_app/
├── docker-compose.yml      # PostgreSQL 容器配置
├── backend/                # Python 后端
│   ├── .env                # 数据库连接（勿提交到 Git）
│   ├── .env.example
│   ├── requirements.txt
│   └── app/
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

---

## 2. 环境准备

### 2.1 需要安装的软件

| 软件 | 推荐版本 | 用途 | 下载 |
|------|----------|------|------|
| Python | **3.11 或更高** | 运行后端 | https://www.python.org/downloads/ |
| Node.js | **18 LTS 或更高** | 运行前端 | https://nodejs.org/ |
| Docker Desktop | 最新稳定版 | 运行 PostgreSQL | https://www.docker.com/products/docker-desktop/ |
| Git（可选） | 最新版 | 克隆代码 | https://git-scm.com/ |

### 2.2 验证安装

**Windows（PowerShell）或 macOS/Linux（终端）：**

```bash
python --version
# 应显示 Python 3.11.x 或更高

node --version
# 应显示 v18.x.x 或更高

npm --version

docker --version
docker compose version
```

### 2.3 PostgreSQL Docker 容器说明

本项目**不在本机直接安装 PostgreSQL**，而是用 `docker-compose.yml` 启动容器：

| 配置项 | 值 |
|--------|-----|
| 容器名 | `todo_postgres` |
| 镜像 | `postgres:16-alpine` |
| 主机端口 | `5432` |
| 数据库名 | `todo_db` |
| 用户名 | `postgres` |
| 密码 | `your_password` |

后端连接串格式（写在 `backend/.env` 中）：

```text
postgresql+asyncpg://postgres:your_password@localhost:5432/todo_db
```

---

## 3. 后端项目结构与核心代码

### 3.1 主要文件说明

| 文件 | 作用 |
|------|------|
| `config.py` | 从 `.env` 读取 `DATABASE_URL` |
| `database.py` | 创建异步数据库引擎、会话、健康检查 |
| `models.py` | SQLAlchemy 表模型 `Task` |
| `schemas.py` | Pydantic 请求/响应模型 |
| `crud.py` | 增删改查业务逻辑 |
| `routers/tasks.py` | REST 路由 `/tasks` |
| `main.py` | FastAPI 入口、CORS、健康检查 |

### 3.2 `requirements.txt`

```text
fastapi>=0.115.0
uvicorn[standard]>=0.32.0
sqlalchemy[asyncio]>=2.0.36
asyncpg>=0.30.0
pydantic-settings>=2.6.0
python-dotenv>=1.0.1
```

### 3.3 `config.py` — 读取环境变量

```python
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8")

    database_url: str = (
        "postgresql+asyncpg://postgres:your_password@localhost:5432/todo_db"
    )


settings = Settings()
```

### 3.4 `database.py` — 异步数据库连接

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

### 3.5 `models.py` — 任务表模型

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

### 3.6 `schemas.py` — API 数据校验

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

### 3.7 `routers/tasks.py` — 任务 REST 接口

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

### 3.8 `main.py` — 应用入口

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

> **说明**：应用启动时会自动执行 `create_all`，在数据库中创建 `tasks` 表（若不存在）。

### 3.9 后端环境变量模板 `backend/.env.example`

```env
DATABASE_URL=postgresql+asyncpg://postgres:your_password@localhost:5432/todo_db
```

复制为 `backend/.env` 后按需修改密码或端口。

> **Windows 注意**：请用 UTF-8 **无 BOM** 保存 `.env`，否则 `DATABASE_URL` 可能无法被正确读取。

---

## 4. 前端项目结构与核心代码

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

### 4.2 `App.jsx` — 根组件

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

### 4.3 `api/client.js` — 统一 API 客户端

> 文档中常称此文件为「api 层」；项目路径为 `src/api/client.js`。

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
```

### 4.4 `components/TaskList.jsx` — 核心片段

```javascript
import { useCallback, useEffect, useMemo, useState } from "react";
import { apiRequest } from "../api/client.js";
import TaskFilter, { filterTasks } from "./TaskFilter.jsx";
import TaskForm from "./TaskForm.jsx";
import TaskItem from "./TaskItem.jsx";
// ... Loading、ErrorAlert 等 UI 组件

export default function TaskList() {
  const [tasks, setTasks] = useState([]);
  const [filter, setFilter] = useState("all");
  // loading、error、操作中等状态 ...

  const filteredTasks = useMemo(
    () => filterTasks(tasks, filter),
    [tasks, filter]
  );

  const fetchTasks = useCallback(async (silent = false) => {
    // GET /tasks，更新 tasks 状态
    const data = await apiRequest("/tasks");
    setTasks(Array.isArray(data) ? data : []);
  }, []);

  useEffect(() => {
    fetchTasks();
  }, [fetchTasks]);

  async function toggleCompleted(task) {
    const nextCompleted = !task.completed;
    const updated = await apiRequest(`/tasks/${task.id}`, {
      method: "PUT",
      body: { completed: nextCompleted },
    });
    setTasks((prev) => prev.map((t) => (t.id === task.id ? updated : t)));
  }

  async function deleteTask(taskId) {
    await apiRequest(`/tasks/${taskId}`, { method: "DELETE" });
    setTasks((prev) => prev.filter((t) => t.id !== taskId));
  }

  return (
    <div className="space-y-6">
      <TaskForm onSuccess={() => fetchTasks(true)} />
      {/* 筛选按钮 TaskFilter、任务卡片 TaskItem 列表 */}
    </div>
  );
}
```

### 4.5 前端环境变量（可选）`frontend/.env`

```env
VITE_API_URL=http://localhost:8000
```

不配置时默认请求 `http://localhost:8000`。

### 4.6 `docker-compose.yml` — 数据库容器

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

### 4.7 跨域（CORS）说明

- 开发时前端地址：`http://localhost:5173`
- 后端在 `main.py` 中已允许该来源跨域
- `vite.config.js` 还配置了 `/api` 代理（可选），将请求转发到 `8000` 端口

---

## 5. 数据库模型设计与 API 端点列表

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
