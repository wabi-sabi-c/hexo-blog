---
title: main.py文件简介
date: 2026-06-05 13:15:20
tags: [Python Project]
categories: [ToDo]
permalink: /todo-doc/main.py/
---


# main.py文件简介


`main.py` 是后端项目的**总入口**和**指挥中心**。它的核心职责是**初始化 FastAPI 应用、配置全局中间件、注册路由以及管理应用的生命周期**。

当你运行 `uvicorn app.main:app --reload` 时，Python 首先执行的就是这个文件。

---

<!--more-->


### 1. 代码深度解析


#### A. 应用生命周期管理 (`lifespan`)

```python

@asynccontextmanager
async def lifespan(app: FastAPI):
    # 1. 启动时执行：自动创建数据库表
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    
    # 2. yield 表示应用正式运行，处理请求
    yield
    
    # 3. 关闭时执行：释放数据库连接资源
    await engine.dispose()

```

*   **作用**：这是 FastAPI 推荐的管理启动和关闭逻辑的方式。
*   **`Base.metadata.create_all`**：这是一个对初学者非常友好的功能。每次启动后端时，它会自动检查 PostgreSQL 中是否存在 `tasks` 表。如果不存在，它就根据 `models.py` 的定义自动创建。**这意味着你不需要手动去数据库写 `CREATE TABLE` 语句。**
*   **`engine.dispose()`**：确保服务器停止时，所有数据库连接都被干净地关闭，防止资源泄露。


#### B. 创建应用实例

```python

app = FastAPI(title="Todo API", lifespan=lifespan)

```

*   **`title`**：设置 API 的标题，会显示在 Swagger 文档页面的左上角。
*   **`lifespan`**：将上面定义的生命周期函数挂载到应用上。


#### C. 跨域资源共享 (CORS) 配置

```python

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:5173",  # Vite 默认端口
        "http://127.0.0.1:5173",
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

```

*   **为什么需要它？**：浏览器出于安全考虑，禁止前端（运行在 `localhost:5173`）直接访问后端（运行在 `localhost:8000`），因为它们的**端口不同**，被视为“跨域”。
*   **作用**：这个中间件告诉浏览器：“我允许来自 `localhost:5173` 的请求访问我。”
*   **注意**：在生产环境中，你应该把 `allow_origins` 改为你的真实域名（如 `https://www.yourtodoapp.com`），而不是使用 `["*"]`。


#### D. 注册路由


```python

app.include_router(tasks.router)

```
*   **作用**：将 `routers/tasks.py` 中定义的所有接口（如 `/tasks`）挂载到主应用上。
*   如果不写这一行，你访问 `/tasks` 就会报 404 错误。


#### E. 健康检查接口


```python

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

*   **作用**：这两个接口用于快速测试后端是否正常运行。
*   **`/health`**：只要服务器启动了，就能访问。
*   **`/db-check`**：不仅检查服务器，还尝试连接数据库。如果数据库没启动或密码错了，这里会返回 `disconnected`，方便调试。

---


### 2. 它在项目中的工作流程


1.  **启动**：你运行启动命令。
2.  **初始化**：`main.py` 被加载，`lifespan` 中的启动代码执行，数据库表被创建。
3.  **监听**：Uvicorn 服务器开始监听 `8000` 端口。
4.  **接收请求**：
    *   前端发来请求。
    *   **CORS 中间件**先检查来源是否合法。
    *   **Router** 根据 URL 找到对应的处理函数（在 `routers/tasks.py` 中）。
    *   处理函数调用 `crud.py` 操作数据库。
    *   结果返回给前端。
5.  **关闭**：你按下 `Ctrl+C`，`lifespan` 中的关闭代码执行，断开数据库连接。

---


### 3. 常见修改场景


*   **添加新的功能模块**：如果你新建了一个 `users.py` 路由器，你需要在这里添加 `app.include_router(users.router)`。
*   **部署到服务器**：你需要修改 `allow_origins`，加入你的服务器公网 IP 或域名。
*   **添加全局异常处理**：可以在这里定义 `@app.exception_handler` 来统一处理报错。

---


### 4. 总结


`main.py` 是你后端的**骨架**。
*   它把分散的模块（路由、数据库、配置）**组装**成一个完整的应用。
*   它解决了前后端分离最头疼的 **跨域 (CORS)** 问题。
*   它通过 **Lifespan** 自动化了数据库表的初始化，让项目“开箱即用”。

对于初学者来说，理解 `lifespan` 和 `CORSMiddleware` 是掌握 FastAPI 项目结构的关键一步。