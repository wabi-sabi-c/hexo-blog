---
title: step1-lightweightFastAPI
date: 2026-05-07 11:40:00
tags: [Python Project]
categories: [ai-chatflow]
---

# 🚀 Step 1：创建后端代码，启动 FastAPI

<!--more-->

## 第一步：创建项目子文件夹与必要文件
用 VS Code 打开项目文件夹： `ai-chatflow`文件夹
在 `ai-chat-flow` 根目录下，新建文件夹 `backend`，然后在 `backend` 里再新建文件夹 `app`，在 `app` 里再新建文件夹 `core`
**目录结构：**         
```bash
ai-chat-flow/
├── backend/                      --> Python FastAPI 后端，相当于"房子的地基和骨架"
│   └── app/                      --> 核心代码，相当于"骨架内部"
│       └── core/                 --> 配置相关，相当于"电表箱"
├── docker-compose.yml
└── ...
```
## 第二步：创建配置文件和环境变量
1. 在 `backend` 文件夹中，新建文件 `requirements.txt`,内容：
```bash
# requirements必要条件 包版本管理文件
# pip install -r requirements.txt


# FastAPI框架 写接口的
fastapi==0.115.6
# 启动项目的服务器 启动接口 主包[附加功能组]==版本 uvicorn[standard]：满血完整版、带加速、带全套配件
# uvicorn main:app
uvicorn[standard]==0.34.0  # standard 标准的
# 操作数据库的工具包
sqlmodel==0.0.22
# Python 连接 PostgreSQL 数据库的驱动
# binary是二进制编译包（已经提前编译好，拿来就能用）
# 意思是：开箱即用、不报错、现成编译好的驱动
psycopg2-binary==2.9.10
# ** 生成登录令牌（Token）** 的工具（做登录功能、用户验证）相当于给登录用户发一张 “临时通行证”
python-jose[cryptography]==3.3.0  # cryptography 密码术 密码
# 密码加密工具包
passlib==1.7.4
bcrypt==4.0.1
# 读取配置文件的工具 （比如数据库密码、端口、密钥这些敏感信息，存在 .env 文件里，靠它读取。让项目更安全、更规范）
python-dotenv==1.0.1
pydantic-settings==2.7.1

```

2. 在 `backend` 文件夹中，新建文件 `.env`,内容：
```bash
#  .env 配置文件，作用就是：把敏感信息、账号密码、配置参数，单独放这里，不写在代码里。

# 数据库(database)连接信息  postgresql:// → 用的数据库是 PostgreSQL
# chatflow_user → 数据库用户名   chatflow_password → 数据库密码 
# localhost → 数据库在你本机   5432 → 数据库端口号   chatflow → 数据库名
DATABASE_URL=postgresql://chatflow_user:chatflow_password@localhost:5432/chatflow
# 密钥（secret秘密 key）（change-me-to-a-random-string-in-production意思是：开发时先用这个，正式上线必须换掉！）
SECRET_KEY=change-me-to-a-random-string-in-production
# 登录令牌加密算法（algorithm算法）HS256（HmacSHA256）
ALGORITHM=HS256
# 访问令牌有效期时间（access_token_expire_minutes）30分钟
ACCESS_TOKEN_EXPIRE_MINUTES=30

```

3. 在 `backend/app/core` 文件夹中，新建文件 `config.py`,内容：
```python
'''
专门用来读取 .env 配置文件里的数据库、密钥、过期时间等信息，统一管理项目配置。
以后你要改数据库、改登录时效，只改 .env 文件，不用动代码！
想拿数据库地址：settings.database_url
想拿密钥：settings.secret_key
想拿登录时效：settings.access_token_expire_minutes

'''

# 导入一个专门读取配置文件的工具。作用：帮你自动读 .env 文件，自动校验格式
from pydantic_settings import BaseSettings

# 创建一个配置类，把所有项目配置（数据库、密钥等）放这里统一管理
class Settings(BaseSettings):
    # 定义一个配置项：数据库连接地址; 如果 .env 里有，就用 .env 的,如果没有，就用这里的默认值
    database_url: str = "postgresql://chatflow_user:chatflow_password@localhost:5432/chatflow"
    # 密钥
    secret_key: str = "change-me-to-a-random-string-in-production"
    # 加密算法
    algorithm: str = "HS256"
    # 登录时效
    access_token_expire_minutes: int = 30
    # 去读取项目里的 .env 文件，用里面的配置覆盖这里的默认值。
    class Config:
        env_file = ".env"
# 创建一个配置实例 
settings = Settings()

```

## 第三步：创建主应用文件
在 `backend/app` 文件夹中，新建文件 `main.py`,内容：
```python
# FastAPI 简单项目的入口文件（main.py） = 启动后端服务、配置跨域、写接口


from fastapi import FastAPI
# 引入跨域中间件(middleware) 作用：允许前端（Vue / 网页）调用你的后端接口
from fastapi.middleware.cors import CORSMiddleware

# 创建 FastAPI 实例
app = FastAPI(title="AI ChatFlow API")

# 允许前端跨域请求（开发阶段宽松配置）
# 给后端服务添加一个功能插件(CORSMiddleware,指定插件类型：跨域处理)
app.add_middleware(
    CORSMiddleware,
    # 只允许这个地址访问后端（origin起源）
    allow_origins=["http://localhost:5173"],  # Vue 开发服务器地址 5173 是 Vue 默认端口
    # 允许带登录凭证（cookie/token）做登录必须开（credentials资格证书）
    allow_credentials=True,
    # 允许的请求方法（允许所有请求方式GET / POST / PUT / DELETE 全都允许）(method方法)
    allow_methods=["*"],
    # 允许的请求头（允许所有请求头,前端传什么参数都放行）（header数据头）
    allow_headers=["*"],
)

# 创建一个接口 （接口名称：read_root）
# 访问方式：http://localhost:8000/
@app.get("/")
def read_root():
    return {"message": "Hello from AI ChatFlow API"}

# 创建一个接口 （接口名称：health_check）
# 访问方式：http://localhost:8000/health
@app.get("/health")
def health_check():
    return {"status": "ok"}

```
## 第四步：创建 Python 虚拟环境并安装依赖
1. 创建虚拟环境：
2. 进入项目文件夹`ai-chatflow/backend`，打开终端`cmd / powershell`，输入：
```bash
python -m venv .venv  # 第二个venv 是虚拟环境文件夹名，可自定义但一般为.venv

```
3. 激活虚拟环境（Windows） --> 终端会显示(.venv)
```bash
# 如果提示权限问题，先执行
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

.venv\Scripts\Activate.ps1
# Windows PowerShell   脚本文件：Activate.ps1	
# Windows Cmd          脚本文件：activate.bat
# Mac / Linux          脚本文件：activate
# 退出虚拟环境  
deactivate
```
4. 安装依赖：
```bash
pip install -r requirements.txt
```

## 第五步：启动 FastAPI 并验收
`uvicorn app.main:app --reload --port 8000`
启动成功后，
访问 `http://localhost:8000` → 如果能看到 `{"message":"Hello from AI ChatFlow API"}`，说明项目启动成功。
访问`http://localhost:8000/health` ，如果返回 `{"status": "ok"}`，说明项目启动成功。
访问 `http://localhost:8000/docs` → 能看到自动生成的 Swagger 文档界面



# 总结：
## 创建此 Python 项目(不包括前端)的步骤：
1. 创建项目文件夹
2. 创建前后端分离文件夹`backend`和`frontend`
3. 创建`.gitignore`文件(git管理代码，忽略一些不必要文件)
4. 创建`docker-compose.yml`文件(因为要使用docker容器，所以需要配置docker-compose.yml文件)
5. 在后端文件夹`backend`下创建requirements.txt文件(安装依赖)
6. 创建虚拟环境--> 在后端文件夹`backend`下创建虚拟环境 `python -m venv .venv`
7. 激活虚拟环境 --> 终端会显示(.venv)
8. 安装依赖 `pip install -r requirements.txt`
9. 写文件`backend/.env` 配置密钥等敏感信息
10. 创建文件`backend/app/core/config.py` 读取配置文件
11. 创建文件`backend/app/main.py` 创建主应用文件
12. 启动 FastAPI 并验收

**注意：提前设置编译格式为UTF-8**

## 🎉 恭喜！最难缠的环境搭建关已经过了，现在你的电脑上已经完整拥有：

     ✅ Docker + PostgreSQL 数据库

     ✅ Python 虚拟环境里的 FastAPI 项目骨架

     ✅ 可运行的 API 端点

