---
title: docker-compose.yml 文件简介
date: 2026-06-05 13:27:20
tags: [Python Project]
categories: [ToDo]
permalink: /todo-doc/docker-compose.yml/
---


# docker-compose.yml 文件简介


## 📂 技术文件深度解析：`docker-compose.yml`（数据库容器版）


### 1. 核心定位 (The "One-Liner")


> **`docker-compose.yml` 是 Docker Compose 工具的**项目定义文件**。它的核心职责是**用一个命令启动整套服务（这里是一个 PostgreSQL 数据库容器）**，并配置它的运行参数、数据持久化和健康检查。  
> 在项目中，它扮演着 **“一键式智能家电安装包”** 的角色：
> - 你不需要手动去官网下载 PostgreSQL、安装、配置用户名密码、初始化数据库、设置开机自启……  
> - 你只需要运行 `docker-compose up -d`，它就会**自动拉取镜像、创建容器、设置环境变量、映射端口、挂载数据卷**，几秒钟后一个完整的数据库就 ready 了。

---

<!--more-->

### 2. 代码深度解析


逐块拆解这个 YAML 文件。

```yaml

services:
  postgres:
    image: postgres:16-alpine
    container_name: todo_postgres
    restart: unless-stopped

```


#### A. 服务定义与基础配置


*   **`services:`**：Docker Compose 的顶层关键字，下面定义你要启动的**所有容器**（可以有一个或多个）。这里只定义了一个叫 `postgres` 的服务。
*   **`image: postgres:16-alpine`**：指定使用哪个 **Docker 镜像**。  
    *   `postgres` 是官方 PostgreSQL 镜像。  
    *   `16-alpine` 表示基于 **Alpine Linux** 的 PostgreSQL 16 版本。Alpine 镜像非常小（约 50MB vs 标准版 300MB），启动快、省资源。  
*   **`container_name: todo_postgres`**：给容器起一个**固定的名字**。  
    *   如果不指定，Docker 会自动生成一个随机的名字（如 `awesome_hopper`）。固定名字方便你在命令行操作：`docker logs todo_postgres`、`docker exec -it todo_postgres bash`。  
*   **`restart: unless-stopped`**：**容器退出后的重启策略**。  
    *   `unless-stopped` 含义：除非你手动 `docker stop`，否则容器**崩溃或服务器重启后都会自动重新启动**。非常适合数据库这种需要长期运行的服务。

```yaml

    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: your_password
      POSTGRES_DB: todo_db

```


#### B. 环境变量 —— 数据库的“初始化说明书”


这些变量会被 PostgreSQL 镜像的 **启动脚本** 读取，用来**自动创建用户和数据库**。

*   **`POSTGRES_USER: postgres`**：超级管理员用户名。默认是 `postgres`，这里显式设置。
*   **`POSTGRES_PASSWORD: your_password`**：超级管理员的密码。**⚠️ 安全提示**：实际项目中请使用更复杂的密码，或者通过 Docker 的 `env_file` 从外部文件读取，避免硬编码在 YAML 里。
*   **`POSTGRES_DB: todo_db`**：容器启动时**自动创建的一个普通数据库**。  
    *   你的应用（待办事项后端）会连接这个 `todo_db`，而不是连接默认的 `postgres` 数据库。  
    *   **注意**：如果这个数据库已经存在，它不会重复创建或报错。

> 💡 为什么不用手动执行 `CREATE DATABASE todo_db;`？因为 PostgreSQL 官方镜像提供了这种“初始化钩子”，极大简化了开发环境搭建。

```yaml

    ports:
      - "5432:5432"

```


#### C. 端口映射 —— 让外界能访问容器内的数据库


*   **格式**：`"宿主机端口:容器内部端口"`  
    *   左边 `5432` 是你电脑（宿主机）上的端口。  
    *   右边 `5432` 是容器内 PostgreSQL 监听的端口（默认）。  
*   **作用**：当你电脑上的程序（例如后端的 FastAPI、Django，或者 GUI 工具如 DBeaver、pgAdmin）连接 `localhost:5432` 时，Docker 会自动把流量转发到容器内的 `5432` 端口。  
*   **如果没有端口映射**：容器内的数据库只能被 Docker 内部网络访问，你的宿主机（macOS/Windows/Linux）无法直接连接它（但其他容器可以通过服务名 `postgres` 访问，见工作流部分）。

```yaml

    volumes:
      - postgres_data:/var/lib/postgresql/data

```


#### D. 数据卷 —— 防止数据库“失忆”


*   **问题**：容器是“一次性”的。如果容器被删除，里面所有数据都会消失（包括你存储的待办事项）。  
*   **解决方案**：**数据卷** 将容器内的目录 `/var/lib/postgresql/data`（PostgreSQL 实际存储数据的地方）**持久化到宿主机**。  
*   **`postgres_data:`**：这是一个**命名卷**，由 Docker 管理，具体存在你电脑的 `/var/lib/docker/volumes/` 下。  
*   **效果**：即使你 `docker-compose down` 删除了容器，下次 `docker-compose up` 时，旧数据依然存在。只有在运行 `docker-compose down -v` 时才会删除卷。

```yaml

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d todo_db"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s

```


#### E. 健康检查 —— 知道数据库什么时候“真正准备好了”


*   **为什么需要**：数据库容器启动需要几秒钟（初始化、加载配置）。如果其他容器（如后端应用）在数据库还没准备好时就去连接，会报错。  
*   **`test`**：执行的命令。  
    *   `pg_isready` 是 PostgreSQL 自带的工具，用于检测数据库是否接受连接。  
    *   `-U postgres` 指定用户名，`-d todo_db` 指定数据库名。  
*   **`interval: 5s`**：每 5 秒检查一次。  
*   **`timeout: 5s`**：每次检查最多等待 5 秒。  
*   **`retries: 5`**：连续失败 5 次才标记为 `unhealthy`。  
*   **`start_period: 10s`**：容器启动后等待 10 秒才开始第一次健康检查，避免启动瞬间的资源竞争。

> 你可以用 `docker inspect --format='{{json .State.Health}}' todo_postgres` 查看健康状态。

```yaml

volumes:
  postgres_data:

```


#### F. 顶层 volumes —— 声明命名卷


*   在文件最底部定义 `volumes:`，列出你之前引用的所有命名卷。  
*   如果不声明，Docker Compose 会报错。  
*   这里只有一个 `postgres_data`，没有额外配置（比如外部驱动、卷选项），所以 Docker 会创建一个**默认的本地卷**。

---


### 3. 为什么需要它？ (Why & Comparison)


#### ❌ 没有 Docker Compose 的混乱做法（传统手动安装数据库）


1.  **去 PostgreSQL 官网下载安装包**（Windows 下载 exe，macOS 下载 dmg，Linux 用 apt-get 但版本可能旧）。
2.  **运行安装向导**，一路下一步，还要记住安装路径。
3.  **初始化数据库集群**（`initdb`），设置环境变量 `PGDATA`。
4.  **修改配置文件** `postgresql.conf` 和 `pg_hba.conf` 允许外部连接。
5.  **启动服务**（`pg_ctl start`）或注册为系统服务。
6.  **创建用户和数据库**：  
    ```sql

    CREATE USER postgres WITH PASSWORD 'your_password';
    CREATE DATABASE todo_db OWNER postgres;

    ```
7.  **每次换电脑 / 队友加入 / 重装系统**，重复以上所有步骤 → 痛苦不堪。

**弊端**：
- 环境不一致（你的 PostgreSQL 版本是 14，同事可能是 15）。
- 卸载不干净，残留数据文件和配置。
- 多个项目共用同一个数据库实例，容易产生冲突。


#### ✅ 有了 `docker-compose.yml` 的优雅做法


- **第一次使用**：  
  ```bash
  docker-compose up -d
  ```
  所有步骤自动完成，**10 秒后数据库即可用**。

- **换一台电脑**：只需安装 Docker Desktop，然后 `git clone` 项目，再 `docker-compose up -d` → 完全一致的环境。

- **多个项目**：每个项目有自己的 `docker-compose.yml` 和独立的**数据卷**，互不干扰。

- **停止并删除容器（保留数据）**：  
  ```bash
  docker-compose down
  ```
- **彻底清除所有数据**：  
  ```bash
  docker-compose down -v
  ```

> 用表格总结对比：

| 操作 | 传统手动安装 | Docker Compose |
|------|-------------|----------------|
| 安装数据库 | 下载 200MB+ 安装包，运行向导 | `docker-compose up` 自动拉取 50MB 镜像 |
| 配置用户名密码 | 手动执行 SQL | 写在 `environment` 里 |
| 持久化数据 | 手动指定数据目录 | `volumes` 自动管理 |
| 跨项目隔离 | 难，需要改端口或使用不同实例 | 每个项目有自己的容器+卷 |
| 团队协作 | 每个人都要折腾一遍 | 所有人用同一个 YAML 文件 |

---


### 4. 在项目中的工作流 (Workflow Context)


这个 `docker-compose.yml` 文件是**整个项目的数据库启动入口**。下图展示它如何融入开发流程：

```text
开发者操作：
    cd 项目目录
    docker-compose up -d
          │
          ▼
Docker Compose 读取 docker-compose.yml
          │
          ├─ 检查镜像 postgres:16-alpine 是否存在
          │      └─ 若不存在，从 Docker Hub 拉取
          ├─ 创建命名卷 postgres_data（若不存在）
          ├─ 启动容器 todo_postgres
          │      ├─ 设置环境变量 (用户名、密码、数据库名)
          │      ├─ 映射端口 5432:5432
          │      ├─ 挂载数据卷到 /var/lib/postgresql/data
          │      └─ 执行 PostgreSQL 初始化脚本
          │             └─ 创建用户 postgres 和数据库 todo_db
          └─ 开始健康检查 (每 5 秒)

数据库就绪后，其他服务可以连接：
    ┌─────────────────────┐
    │  后端应用（待启动）    │
    │  连接字符串：         │
    │  postgresql://postgres:your_password@localhost:5432/todo_db
    └─────────────────────┘
              │
              ▼
        查询 / 插入待办事项

```

**与其他文件/容器的关联**：
- **后端容器（未在本文件中定义）**：通常会有一个 `backend` 服务，其 `depends_on` 会指向 `postgres`，并且会读取本数据库的地址（通过服务名 `postgres` 或宿主机 `localhost`）。
- **`.env` 文件**（常见实践）：把密码等敏感信息放在 `.env` 里，然后在 `docker-compose.yml` 中通过 `env_file` 引用，避免硬编码。
- **前端容器**：不直接连接数据库，而是通过后端 API 间接访问。

> 注意：这里的 `docker-compose.yml` **只定义了数据库**。一个完整的项目通常还会有后端和前端服务定义在同一个文件里。但即使只有数据库，它也是不可或缺的基础设施。

---


### 5. 总结


- **它是一个“声明式”的环境定义文件**：你告诉 Docker 想要什么样的数据库（版本、用户、持久化、健康检查），Docker 负责实现它，无需手动配置。
- **它解决了开发环境一致性和“环境地狱”问题**：让所有开发者、测试环境、甚至生产环境用完全相同的容器配置，杜绝“在我电脑上能跑”的尴尬。
- **它利用了 Docker 的镜像分层、数据卷、健康检查等特性**：`image` 保证版本固定，`volumes` 保证数据安全，`healthcheck` 保证服务就绪顺序。
- **对于初学者，理解 `volumes` 和 `ports` 是掌握它的核心**：
  - `ports` 让外部能访问容器（连接数据库）。
  - `volumes` 让数据在容器销毁后依然存在（否则重启后所有待办事项都会消失！）。

---


### 🎓 最后的小贴士


- 如果你想**连接这个数据库**（比如用命令行或 GUI 工具），连接参数是：
  - Host: `localhost`
  - Port: `5432`
  - User: `postgres`
  - Password: `your_password`
  - Database: `todo_db`
- 如果想**重置所有数据**（清空所有表），运行 `docker-compose down -v` 然后 `docker-compose up -d`。
- 生产环境请不要使用 `alpine` 镜像的 `latest` 标签，应指定具体版本如 `16-alpine`（本文件已做到）。
- 若后端也跑在 Docker 容器中，可以通过**容器名** `todo_postgres` 或服务名 `postgres` 来连接，而不是 `localhost`（因为每个容器有自己的网络栈）。

