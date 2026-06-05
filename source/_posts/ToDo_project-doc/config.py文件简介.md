---
title: config.py文件简介
date: 2026-06-05 13:09:20
tags: [Python Project]
categories: [ToDo]
permalink: /todo-doc/config.py/
---


# config.py文件简介


`config.py`是后端项目的**配置中心**。它的核心职责是**安全、统一地管理项目所需的所有环境变量和配置项**。

在你的 ToDo-App 项目中，它主要解决了“硬编码”问题，让代码更灵活、更安全。

<!--more-->


### 1. 核心作用


*   **读取环境变量**：从 `backend/.env`文件中读取敏感信息（如数据库密码、API 密钥）。
*   **类型安全**：利用 [`pydantic`](/todo-doc/Pydantic/) 的特性，确保读取到的配置数据类型正确（例如，确保端口号是整数而不是字符串）。
*   **提供默认值**：如果 `.env` 文件中没有某个配置，它可以提供一个安全的默认值，防止程序崩溃。
*   **全局共享**：其他模块（如 `database.py`）只需导入这个文件中的 `settings` 对象，即可获取配置，无需重复读取文件。

---


### 2. 代码深度解析


```python

from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    # 1. 配置 Pydantic 的行为
    model_config = SettingsConfigDict(
        env_file=".env",          # 指定读取哪个文件
        env_file_encoding="utf-8" # 指定编码格式，防止中文乱码
    )

    # 2. 定义具体的配置项
    # 变量名通常大写，对应 .env 文件中的键名（不区分大小写）
    database_url: str = (
        "postgresql+asyncpg://postgres:your_password@localhost:5432/todo_db"
    )
    
    # 你可以继续添加其他配置，例如：
    # app_name: str = "ToDo App"
    # debug: bool = True

# 3. 实例化对象
# 整个项目只需要这一个实例，其他文件直接导入使用即可
settings = Settings()

```

#### 关键点说明：
1.  **`BaseSettings`**：这是 `pydantic-settings` 提供的基类。它会自动查找环境变量。优先级通常是：**系统环境变量 > `.env` 文件 > 默认值**。
2.  **`model_config`**：这是 Pydantic V2 的新写法。它告诉程序：“去当前目录下的 `.env` 文件里找配置”。
3.  **`database_url`**：
    *   如果在 `.env` 文件中写了 `DATABASE_URL=postgresql+asyncpg://user:pass@host/db`，那么 `settings.database_url` 就会是这个值。
    *   如果 `.env` 文件不存在或没写这一行，它就会使用代码中等号后面的**默认值**（即 `"postgresql+asyncpg://..."`）。

---


### 3. 为什么需要它？（对比硬编码）


#### ❌ 不好的做法（硬编码）


在 `database.py` 中直接写死连接字符串：
```python

# 危险！密码泄露，且换环境要改代码
engine = create_async_engine("postgresql+asyncpg://postgres:123456@localhost/todo_db")

```


#### ✅ 好的做法（使用 config.py）


在 `database.py` 中动态读取：
```python

from app.config import settings

# 安全！密码在 .env 中，且 .env 不会被上传到 Git
engine = create_async_engine(settings.database_url)

```

---

### 4. 如何扩展 config.py？
随着项目变大，你可能需要更多配置。只需在 `Settings` 类中添加字段即可：

```python

class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file=".env", 
        env_file_encoding="utf-8"
        )

    # 数据库
    database_url: str
    
    # 应用信息
    app_name: str = "ToDo App"
    app_version: str = "1.0.0"
    
    # 安全密钥 (用于 JWT 签名等)
    secret_key: str = "change_me_in_production"
    
    # 调试模式
    debug: bool = False

```

对应的 `.env` 文件内容：
```dotenv

DATABASE_URL=postgresql+asyncpg://postgres:your_password@localhost:5432/todo_db
SECRET_KEY=my_super_secret_key_123
DEBUG=True

```

### 5. 总结


`config.py` 是你后端的 **“保险箱”** 和 **“控制台”**。
*   它保护了你的密码（通过分离 `.env`）。
*   它让你的应用更容易部署（在不同服务器只需修改 `.env`，无需改代码）。
*   它是学习现代 Python Web 开发（FastAPI + Pydantic）的标准实践。