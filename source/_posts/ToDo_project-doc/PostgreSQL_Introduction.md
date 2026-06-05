---
title: PostgreSQL简介
date: 2026-06-05 13:04:20
tags: [Python Project]
categories: [ToDo]
permalink: /todo-doc/PostgreSQL/
---


# PostgreSQL简介


**PostgreSQL 16** 是一个功能强大、开源的 **对象-关系型数据库管理系统 (ORDBMS)**。

在你的 ToDo-App 项目中，它扮演着 **“永久仓库”** 的角色，负责安全、可靠地存储所有任务数据。

<!--more-->


### 1. 核心作用


*   **数据持久化**：当你新建一个任务时，数据不仅显示在页面上，还会被写入 PostgreSQL。即使你关闭电脑或重启服务器，数据也不会丢失。
*   **结构化存储**：它以“表”（Table）的形式组织数据。例如，你的 `tasks` 表可能包含 `id`, `title`, `description`, `is_completed`, `created_at` 等列。
*   **数据查询与筛选**：当你点击“查看进行中任务”时，PostgreSQL 会快速从成千上万条记录中找出符合条件的数据并返回给后端。


### 2. 为什么选择 PostgreSQL？


相比于其他数据库（如 SQLite, MySQL），PostgreSQL 有以下优势：
*   **高可靠性与稳定性**：它以“不丢数据”著称，支持复杂的事务处理（ACID），确保数据的一致性。
*   **强大的功能**：支持 JSONB（可以存非结构化数据）、全文搜索、地理信息等高级特性。
*   **开源免费**：社区活跃，文档丰富，适合各种规模的项目。
*   **版本 16**：这是 PostgreSQL 的较新版本，提供了更好的性能优化、查询速度和安全性更新。


### 3. 在项目中的工作流程


1.  **前端 (React)**：用户点击“保存任务”。
2.  **后端 (FastAPI)**：接收到数据，通过 **SQLAlchemy** 生成 SQL 指令。
3.  **驱动 (asyncpg)**：将指令发送给 **PostgreSQL**。
4.  **数据库 (PostgreSQL)**：执行 `INSERT` 操作，将数据写入硬盘，并返回“成功”信号。


### 4. 关于“用 Docker 运行”


在你的技术栈中提到“用 Docker 运行”，这意味着：
*   你不需要在本地电脑上手动安装复杂的 PostgreSQL 软件。
*   你只需要一个 `docker-compose.yml` 文件，一键启动一个隔离的、干净的 PostgreSQL 16 容器。
*   这保证了开发环境和生产环境的一致性，也方便清理和重置数据库。


### 总结


*   **FastAPI** 是处理逻辑的 **大脑**。
*   **SQLAlchemy** 是翻译指令的 **嘴巴**。
*   **PostgreSQL** 是记忆数据的 **大脑皮层（硬盘）**。

没有 PostgreSQL，你的 ToDo 应用就只是一个“一次性”的页面，刷新后数据就会消失。它是整个应用 **数据根基** 所在。