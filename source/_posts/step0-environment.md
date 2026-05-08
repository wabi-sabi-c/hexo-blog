---
title: step0-environment
date: 2026-05-06 10:38:06
tags: [Python Project]
categories: [ai-chatflow]
---

```bash
1. ai-chatflow项目介绍：
    一个支持多轮对话、历史记录管理的智能问答平台。
    用户可通过网页 与 AI 自然对话，所有对话历史都会保存在你自己的数据库中。
    此项目是一个完整的前后端项目，主要是练习前后端分离开发的知识。
2. 技能点：
    - 后端：FastAPI 路由设计、SQLModel 数据库操作、JWT 用户认证

    - 数据库：PostgreSQL 表设计、一对多关系、索引优化

    - 前端：Vue3 组合式 API、现代化聊天界面 UI、与后端实时交互

    - AI 集成：调用 OpenAI 兼容接口（如 DeepSeek、通义千问等）实现智能问答

    - 工程化：Docker 容器化部署、pytest 自动化测试、Git 版本控制

    - 文档沉淀：每个步骤都会生成 Markdown 教程
3.核心功能（MVP）：
    - 用户注册 / 登录（JWT 令牌鉴权）

    - 创建、查看、删除对话

    - 在对话中发送、接收消息，并调用 LLM 获得智能回复

    - 前端基础聊天界面
```

<!--more-->


# 🚀 Step 0：开发环境搭建 & 项目根目录骨架创建
**提示：此项目是windows环境开发**

## 一、开发环境搭建

### 第一步：安装基础软件
1.1 安装Python(3.11+)
   - 安装完成后，打开 PowerShell（在开始菜单右键选择“Windows PowerShell”或“终端”），输入：`python --version`查看Python版本
    
1.2 安装Node.js（LTS 长期支持版）
   - 安装完成后，打开 PowerShell（在开始菜单右键选择“Windows PowerShell”或“终端”），输入：`node --version` `npm --version`查看Node.js版本 和 npm 版本

1.3 安装Docker Desktop
   - 安装完成后，打开 PowerShell（在开始菜单右键选择“Windows PowerShell”或“终端”），输入：`docker --version`查看Docker版本

1.4 安装IDE （此项目使用VS Code）

1.5 安装Git
   - 安装完成后，打开 PowerShell（在开始菜单右键选择“Windows PowerShell”或“终端”），输入：`git --version`查看Git版本

1.6 上述所有软件可以自行搜索最新的安装教程 如安装python 直接用必应搜索“python安装” 选择最新的即可

### 第二步：创建项目根目录并初始化 Git
1.1 第一次安装Git（必须做的配置）
   - `git config --global user.name "你的名字"` `git config --global user.email "你的邮箱"` 设置Git 的用户名和邮箱
   - 这是 Git 要求的身份标识，和 GitHub 无关，随便填都行

1.2 创建项目根目录并初始化 Git
```bash
cd ~                # 进入你的用户目录
mkdir ai-chatflow   # 创建项目文件夹
cd ai-chatflow
git init            # 初始化 Git 仓库（用于版本管理，也方便博客记录提交历史）
```

## 二、创建项目根目录骨架
### 1.1 根目录文件结构
```bash
ai-chatflow/
├── backend/                  ← Python FastAPI 后端
├── frontend/                 ← Vue3 前端
├── docker-compose.yml        ← 一键启动 PostgreSQL 数据库
├── docs/                     ← Markdown文章素材
└── .gitignore                ← git忽略文件配置
```

### 1.2 创建.gitignore文件

1. 为什么要使用Git？
   - Git 是一个分布式版本控制软件，用于管理代码的版本，并记录每次修改的内容。**相当于“游戏存档，可以回退到任何时间点，也可以查看历史记录”**这样就不怕修改代码后无法回到正常状态。
   - 具体使用教程可以搜索“Git使用教程”，vs code中使用Git就是修改 -> 暂存 -> 记录提交
2. 为什么要使用.gitignore？
   - .gitignore 是一个文件，用于告诉 Git 忽略某些文件。因为某些文件是**敏感信息**，比如密码、密钥、配置文件等等，我们不希望将它们提交,可能有隐患,因此使用.gitignore文件来告诉Git忽略这些文件。


创建一个文件名叫 `.gitignore`,粘贴下面内容：

<details>

<summary>点击查看</summary>

```bash
# 这些内容按需使用或注释 Ctrl + / 快速注释
# ==============================================
# 基础：GitHub 官方 Python.gitignore 核心部分
# 来源：https://github.com/github/gitignore/blob/main/Python.gitignore
# ==============================================

# 1. Python 字节码与编译文件
__pycache__/
*.py[cod]
*$py.class
*.so

# 2. 虚拟环境（绝对不能传）
.venv/
venv/
ENV/
env/
.Python

# 3. 分发与打包产物
build/
dist/
downloads/
eggs/
.eggs/
wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# 4. 测试与覆盖率报告
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
.hypothesis/
.pytest_cache/
cover/

# 5. 文档构建
docs/_build/
site/

# 6. Jupyter Notebook
.ipynb_checkpoints/

# 7. Python 版本管理
.python-version

# 8. PyInstaller
*.manifest
*.spec

# ==============================================
# 补充：FastAPI 官方仓库自用配置
# 来源：https://github.com/fastapi/fastapi/blob/master/.gitignore
# ==============================================

# 9. 数据库文件（SQLite）
*.db
*.sqlite
*.sqlite3
test.db

# 10. 日志文件
*.log
log.txt
logs/

# 11. 类型检查缓存
.mypy_cache/
.dmypy.json
dmypy.json

# 12. 编辑器与 IDE 配置
.idea/
.vscode/
*.swp
*.swo
*~
.*.sw?

# 13. 操作系统临时文件
.DS_Store
Thumbs.db
.codspeed

# ==============================================
# 安全：敏感信息（绝对禁止上传）
# ==============================================
.env
.env.*
!.env.example

# ==============================================
# 可选：根据你的项目情况添加
# ==============================================
# 如果有前端代码在同目录
# node_modules/
# dist/

# 如果使用 Docker
# .dockerignore
# docker-compose.override.yml

# 如果使用 Celery
# celerybeat-schedule
```

</details>

#### 1.2.1 验证和常见问题
```bash
# 检查某个文件是否会被忽略
git check-ignore -v .venv
# 输出：.gitignore:12:.venv/ .venv → 表示生效

# 检查所有未跟踪文件
git status
# 应该只显示你真正需要提交的代码文件
```

```bash
# 如果已经不小心提交了不该传的文件
# 1. 从 Git 中删除（保留本地文件）
git rm -r --cached .venv
git rm --cached .env

# 2. 提交变更
git add .gitignore
git commit -m "fix: add proper .gitignore and remove sensitive files"

# 3. 推送到远程
git push
```

#### 1.2.2 完整流程
```bash
# 终端
1. 安装 Git → 必须
2. 打开项目目录
3. git init → 初始化
4. 创建 .gitignore
5. git add .
6. git commit -m "初始提交"
7. git push
# vscode 直接在源代码管理中暂存更改 -> 提交（在本地存储,git push 推送到远程仓库）
```

### 2. 创建docker-compose.yml文件
1. 为什么要创建docker-compose.yml文件？
   - 当一个项目需要多个容器一个个启动太麻烦，并且这些docker容器都是分散独立的,也不方便镜像管理
**提示：本项目目前只使用PostgreSQL数据库容器**
```bash
# 固定写法：声明这个配置文件用的语法版本 3.8，通用兼容所有新版 Docker Desktop
version: '3.8'
# 配置服务（可以放多种服务如前端、后端、数据库、缓存、消息队列等）
services:
  # 数据库服务,给数据库服务起的小名：db
  db:
    # 镜像  指定是PostgreSQL 16 版本数据库，alpine 极简精简版，体积小、不占电脑资源，开发用刚刚好
    image: postgres:16-alpine
    # 容器名 给数据库起一个名字：chatflow_db，方便管理
    container_name: chatflow_db
    # 重启策略：只要你不手动关掉它，电脑开机自动启动、数据库意外崩了也自动重启，一直在线
    restart: unless-stopped
    # 配置环境变量，数据库用户名、密码、数据库名
    environment:
      POSTGRES_USER: chatflow_user
      POSTGRES_PASSWORD: chatflow_123456
      # 数据库名  Docker 会自动帮你创建一个名叫 chatflow 的数据库，不用你手动进数据库建库
      POSTGRES_DB: chatflow
    # 端口映射，把本机的 5432 端口映射到容器的 5432 端口，这样数据库就可以被本机访问了
    ports:
      - "5432:5432"
    # 数据持久化 数据卷挂载到容器的 /var/lib/postgresql/data 目录下，数据持久化
    volumes:
      - postgres_data:/var/lib/postgresql/data
# 声明 用一个名叫 postgres_data 的存储空间，交给 Docker 自动管理，不用你自己找文件夹、不用手动配置路径
volumes:
  postgres_data:
```


## 三、总结
> 1. 其实项目初期还是有许多的内容的：各种软件的安装、Git、docker-compose.yml 这些的内容还是有些困难的、耗费时间的
>     - 有时安装过程中会有网络的问题，权限的问题，尤其是docker Desktop安装会很繁琐也有问题但是不需要深入探究，只要会安装就行
>     - Git 目前也不需要深入学习，只有基础用法就行；docker-compose.yml也是同样但有兴趣可以自学docker compose
>