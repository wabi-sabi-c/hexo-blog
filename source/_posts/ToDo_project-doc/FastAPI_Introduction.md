---
title: FastAPI简介
date: 2026-06-05 13:01:20
tags: [Python Project]
categories: [ToDo]
permalink: /todo-doc/FastAPI/
---


# FastAPI简介


**FastAPI** 是一个用于构建 API 的现代、快速（高性能）的 **Web 框架**，基于标准 Python 类型提示。

在你的 ToDo-App 项目中，它扮演着 **“后端大脑”** 和 **“交通指挥中心”** 的角色。

<!--more-->


### 1. 核心作用


*   **接收请求**：当你在前端页面点击“新建任务”或“删除任务”时，浏览器会发送一个 HTTP 请求。FastAPI 负责接收这些请求。
*   **处理逻辑**：它根据请求的类型（如 `GET` 获取列表，`POST` 新建任务），调用相应的 Python 函数进行处理。
*   **连接数据库**：在函数内部，它会指挥 **[SQLAlchemy](/todo-doc/SQLAchemy2.0/)** 去数据库里存取数据。
*   **返回响应**：处理完成后，它将结果（比如任务列表的 JSON 数据）打包发回给前端 React 页面。


### 2. 为什么选择 FastAPI？


在你的技术栈中，FastAPI 有以下几个显著优势：

*   **极速高性能**：它的性能堪比 NodeJS 和 Go，是 Python 中最快的框架之一。这得益于它底层使用的 **Starlette**（用于 Web 部分）和 **Pydantic**（用于数据验证）。
*   **原生异步支持 (Async/Await)**：
    *   这是它与老牌框架（如 Django, Flask）最大的区别。
    *   它可以轻松处理高并发请求。当它在等待 **[asyncpg](/todo-doc/asyncpg/)** 从数据库取数据时，它不会“傻等”，而是可以去处理其他用户的请求。这对于全栈应用的用户体验至关重要。
*   **自动生成交互式文档**：
    *   你不需要额外写文档，FastAPI 会根据你的代码自动生成 **Swagger UI** (通常在 `/docs` 路径)。
    *   你可以在浏览器里直接看到所有可用的接口（如 `/tasks`, `/tasks/{id}`），并可以直接在线测试接口，这对前后端分离开发非常友好。
*   **基于类型提示 (Type Hints)**：
    *   它利用 Python 3.6+ 的类型提示来自动验证数据。如果前端传过来的数据格式不对（比如标题应该是字符串却传了数字），FastAPI 会自动返回清晰的错误信息，无需你手动写大量验证代码。


### 3. 代码中的体现


在你的后端代码中，你会看到类似这样的结构：

```python

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# 定义数据模型

class TaskCreate(BaseModel):
    title: str
    description: str | None = None

# 定义接口：POST /tasks/

@app.post("/tasks/")
async def create_task(task: TaskCreate):
    # 这里会调用 SQLAlchemy 保存数据
    # ...
    return {"message": "任务创建成功", "task": task}

```


### 4. 总结它在项目中的位置


*   **前端 (React)**：负责“长相”和交互，向后端发指令。
*   **FastAPI (后端)**：负责“逻辑”和“调度”，接收指令，指挥数据库干活，然后反馈结果。
*   **SQLAlchemy + asyncpg**：负责“存储”，真正操作 PostgreSQL 数据库。

简单来说，**FastAPI 是你后端服务的入口和控制器**，它让 Python 也能写出高性能、易维护的现代化 Web 接口。