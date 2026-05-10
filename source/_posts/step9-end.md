---
title: step9-end
date: 2026-05-10 18:06:48
tags: [Python Project]
categories: [ai-chatflow]
---

# 🎉 step9：项目正式结束
因为我又优化了一下这个Demo项目，增加了一些功能，变得更像实际项目，不想再一一介绍了所以写了这最后一篇文章，展示所有的代码内容
并且此篇就只介绍代码，不介绍原理。**文件内容通过测试，运行没有问题**
- FastAPI：Python 里写接口最快、最现代、自带文档。

- SQLModel：把数据库表和 Python 类结合，读写数据库就像操作普通对象。

- PostgreSQL：功能最全的关系型数据库。

- Vue 3：前端框架，学习曲线最平缓，把页面拆成组件。

- Docker：把整个系统装进容器，一键启动，环境永不打架。
<!--more-->

增加功能有：
1. backend/app/utils/auth.py — 新增 get_current_user_id 来避免前端改IP获得别人对话数据
2. backend/app/utils/ai_service.py — 升级到真实 AI + 降级 + 安全限制
3. backend/app/routers/conversations.py — 加上鉴权 + 限流
4. backend/app/main.py — 加全局限流器 + 补跨域 
5. frontend/src/api/index.js — 补全登录/注册 + JWT 拦截器 + 适配无参 GET /conversations/
6. frontend/src/router/index.js — 添加登录页路由 + 路由守卫
7. 新建 frontend/src/views/LoginView.vue — 登录/注册页面
8. frontend/src/views/ChatView.vue — 适配新 API（去掉硬编码 userId）
9. 项目根目录 .env — 统一管理所有敏感配置
- 本地 uvicorn 启动→ 读 backend/.env
- Docker 启动 → 读根目录 .env（通过 env_file 指定）
- 这样只需要将ai_KEY填入.env文件更加安全了, 避免了敏感信息泄漏
1.  docker-compose.yml — 一键启动所有服务
- backend/Dockerfile — 后端镜像构建
- frontend/Dockerfile — 前端镜像构建
1.  backend/tests/test_api.py — 适配 JWT 鉴权

其实就是：
1. 升级后端 AI 服务
2. 增加鉴权功能
3. 增加前端登录/注册
4.  升级 Docker Compose
5.  敏感信息永远不进代码 → 全部进 .env，并且 .gitignore 里要有 .env
6.  所有涉及网络、数据库的操作，外面必须包 try → 不崩是底线
7.  API Key 安全化、服务降级（模拟回复）、加上限流和长度保护


# **本项目所有文件编码为UTF-8** **下面内容只是按目录顺序，并不是工程开发顺序**
# 根目录ai-chatflow
## `.env`
```bash
# ===== Database =====
POSTGRES_USER=chatflow_user
POSTGRES_PASSWORD=chatflow_123456
POSTGRES_DB=chatflow


DATABASE_URL=postgresql://chatflow_user:chatflow_123456@db:5432/chatflow

# ===== JWT =====
SECRET_KEY=change-me-to-a-real-random-secret
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# ===== AI  =====
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxyour-api-key
DEEPSEEK_BASE_URL=https://api.deepseek.com/v1
```

## `.gitignore`
```bash
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
node_modules/
dist/

# 如果使用 Docker
.dockerignore
docker-compose.override.yml


# 如果使用 Celery
# celerybeat-schedule
```

## `docker-compose.yml`
```yml
version: '3.8'
services:
  db:
    image: postgres:16-alpine
    container_name: chatflow_db
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 5s
      timeout: 5s
      retries: 5

  backend:
    build: ./backend
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000
    env_file:
      - .env
    ports:
      - "8000:8000"
    depends_on:
      db:
        condition: service_healthy

  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - backend

volumes:
  postgres_data:
```

# 后端代码backend
## `backend/app/core/config.py`
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
    # AI
    deepseek_api_key: str = "yourkey"              # 新增
    deepseek_base_url: str = "https://api.deepseek.com/v1"  # 新增
    # 去读取项目里的 .env 文件，用里面的配置覆盖这里的默认值。
    model_config = {"env_file": ".env",
                    "env_file_encoding": "utf-8"}
# 创建一个配置实例 
settings = Settings()
```

## `backend/app/models/conversation.py`
```python
'''
创建两个数据表：
1. Conversation 对话表：存 “一个聊天窗口”
2. Message 消息表：存这个窗口里的每一条消息
一对多关系：
    1 个对话 ↔ 多条消息

'''

# SQLModel：用来定义 “数据库表” Field：用来定义 “字段 / 列” Relationship：用来定义 “表和表之间的关系”
from sqlmodel import SQLModel, Field, Relationship
# 类型提示工具，告诉代码：某个值可以为空、是列表
from typing import Optional, List
from datetime import datetime, timezone

class Conversation(SQLModel, table=True):
    # 数据库会自动从 1、2、3… 自动递增给你生成 id
    id: Optional[int] = Field(default=None, primary_key=True)
    # 对话标题：规定字符串类型
    title: str
    # foreign_key = 外键
    user_id: int = Field(foreign_key="user.id")   # 关联用户表
    # default_factory 是 “创建时自动调用函数” 注意：必须传一个可调用的函数
    created_at: datetime = Field(default_factory=lambda: datetime.now(timezone.utc))

    # 一个对话下面可以有多条消息，这里定义关系（可选，方便以后直接用 conversation.messages）
    '''
    1. messages:
    - 给对话类加一个**属性**，名字叫 messages。
    2. List["Message"]
    - 意思是：
        这个对话下面，装着一 “列表” 的消息
        列表 = 多条消息。
        加引号 "Message" 是因为：
        Message 类写在下面, Python 还没读到, 先占个位置。
    3. Relationship(...)
    - 关系声明: 告诉 SQLModel: 对话 和 消息 是有关系的！
    4. back_populates="conversation"
    - 最关键的一句！
    - 意思：你去(Message 类)找一个叫 conversation 的属性，咱俩互相关联！
    5. 对话.messages      -> 直接返回这个对话的全部消息列表直接返回这个对话的全部消息列表
       消息.conversation  -> 直接拿到这条消息属于哪个对话  
    '''
    messages: List["Message"] = Relationship(back_populates="conversation")

class Message(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    conversation_id: int = Field(foreign_key="conversation.id")
    # 角色：规定字符串类型 这条消息是谁发的  两种值："user" = 用户发的 "assistant" = AI 回复的
    role: str = Field(default="user")   # 'user' 或 'assistant'
    # 内容：规定字符串类型
    content: str
    created_at: datetime = Field(default_factory=lambda: datetime.now(timezone.utc))

    # 属于哪个对话
    conversation: Conversation = Relationship(back_populates="messages")
```

## `backend/app/models/user.py`
```python
'''
SQLModel 数据库模型  作用：定义数据库里的「用户表」结构 + 接口接收 / 返回的数据格式
'''

# 导入数据库工具： SQLModel = 建表、定义数据格式的基类  Field = 给字段加设置（主键、索引、默认值等）
from sqlmodel import SQLModel, Field
# 导入数据类型： Optional = 允许字段为空
from typing import Optional
# 导入时间模块
from datetime import datetime, timezone
# 用户基础信息（公共字段）
# 创建数据库模型  UserBase = 用户基础类（公共模板） 
# index=True = 加速查询（搜邮箱更快）  unique=True = 邮箱不能重复
class UserBase(SQLModel):
    email: str = Field(index=True, unique=True)
# 真正的数据库表
# 创建数据库模型  User = 用户类（继承 UserBase → 自带 email）
# table=True → 这是数据库真实表，会自动创建
class User(UserBase, table=True):
    # 用户编号, optional[int] → 允许为空的数字
    # Field设置规则，default=None → 默认值是None 
    # primary_key=True → 主键
    id: Optional[int] = Field(default=None, primary_key=True)
    # 密码加密存储, 类型是str 字符串
    hashed_password: str
    # 创建时间 自动填写当前时间
    # Field设置规则，default_factory=lambda（临时的函数）: datetime.now(timezone.utc) → 默认值是当前UTC时间（世界标准时间（UTC））
    created_at: datetime = Field(default_factory=lambda: datetime.now(timezone.utc))
# 用户注册用的表 （继承 UserBase → 自带 email）
# 填写密码 + 邮箱
class UserCreate(UserBase):
    password: str
# 返回给前端看的用户信息 （继承 UserBase → 自带 email）
# id + 创建时间
class UserRead(UserBase):
    id: int
    created_at: datetime
# 用户登录用的表 （继承 SQLModel → 自带 email）
# 填写邮箱 + 密码
class UserLogin(SQLModel):
    email: str
    password: str

```

## `backend/app/routers/conversations.py`
```python
'''
提供对话和消息的增删改查接口
- 被谁引用: main.py 会把这个文件挂载
- 引用了谁: models/conversation.py(操作表), db.py(获得数据库连接)
1. @router.get 是什么
-  HTTP 的 GET 请求  (前端主动找后端**要数据**, 只读数据，不修改、不新增、不保存) 
- GET -> 数据一般放 URL 地址里(路径参数、查询参数), 不能传大内容、不能传复杂 JSON
2. @router.post 是什么
-  HTTP 的 POST 请求 (前端把数据发给后端, 让后端保存 / 处理, 会改数据库：新增、创建、提交)
- POST -> 数据一般放请求体里(JSON), 可以传复杂对象、长文本、表单

现在所有接口都要求登录，自动从 token 获取 user_id。
发送消息接口额外加了速率限制。
'''

from fastapi import APIRouter, HTTPException, Depends, Request
from sqlmodel import Session, select
from app.models.conversation import Conversation, Message
from app.db import get_session
from app.utils.auth import get_current_user_id
from app.utils.ai_service import generate_ai_reply
from slowapi import Limiter
from slowapi.util import get_remote_address

router = APIRouter(prefix="/conversations", tags=["conversations"])
# 速率限制,按IP 限制
limiter = Limiter(key_func=get_remote_address)


# ---------- 对话相关 ----------

@router.post("/", response_model=Conversation)
# 不从前端获取 user_id, 直接从后端拿
def create_conversation(
    conversation: Conversation,
    session: Session = Depends(get_session),
    user_id: int = Depends(get_current_user_id)   # 自动获取当前用户
):
    """创建对话，自动绑定当前登录用户"""
    conversation.user_id = user_id  # 强制覆盖
    session.add(conversation)
    session.commit()
    session.refresh(conversation)
    return conversation


@router.get("/", response_model=list[Conversation])
def get_conversations(
    session: Session = Depends(get_session),
    user_id: int = Depends(get_current_user_id)   # 只返回自己的对话
):
    statement = (
        select(Conversation)
        .where(Conversation.user_id == user_id)
        .order_by(Conversation.created_at.desc())
    )
    return session.exec(statement).all()


@router.get("/{conversation_id}", response_model=Conversation)
def get_conversation(
    conversation_id: int,
    session: Session = Depends(get_session),
    user_id: int = Depends(get_current_user_id)   # 只能看自己的
):
    conv = session.get(Conversation, conversation_id)
    if not conv or conv.user_id != user_id:
        raise HTTPException(status_code=404, detail="Conversation not found")
    return conv


@router.delete("/{conversation_id}")
def delete_conversation(
    conversation_id: int,
    session: Session = Depends(get_session),
    user_id: int = Depends(get_current_user_id)   # 只能删自己的
):
    conv = session.get(Conversation, conversation_id)
    if not conv or conv.user_id != user_id:
        raise HTTPException(status_code=404, detail="Conversation not found")

    # 先删除子消息
    messages = session.exec(
        select(Message).where(Message.conversation_id == conversation_id)
    ).all()
    for msg in messages:
        session.delete(msg)

    session.delete(conv)
    session.commit()
    return {"ok": True}


# ---------- 消息相关 ----------

@router.get("/{conversation_id}/messages", response_model=list[Message])
def get_messages(
    conversation_id: int,
    session: Session = Depends(get_session),
    user_id: int = Depends(get_current_user_id)   # 只能看自己对话的消息
):
    conv = session.get(Conversation, conversation_id)
    # 对话不存在或者不是当前用户
    if not conv or conv.user_id != user_id:
        raise HTTPException(status_code=404, detail="Conversation not found")

    statement = (
        select(Message)
        .where(Message.conversation_id == conversation_id)
        .order_by(Message.created_at)
    )
    return session.exec(statement).all()


@router.post("/{conversation_id}/messages", response_model=Message)
@limiter.limit("1/second;20/minute")               # 速率限制(同IP最多1次/秒，最多20次/分钟)
def send_message(
    request: Request,                              # 限流需要 Request 对象
    conversation_id: int,
    message: Message,
    session: Session = Depends(get_session),
    user_id: int = Depends(get_current_user_id)    # 自动获取当前用户
):
    """发送消息，保存后自动调用 AI 生成回复"""
    conv = session.get(Conversation, conversation_id)
    if not conv or conv.user_id != user_id:
        raise HTTPException(status_code=404, detail="Conversation not found")

    # 保存用户消息
    message.conversation_id = conversation_id
    session.add(message)
    session.commit()
    session.refresh(message)

    # 生成并保存 AI 回复
    ai_reply = generate_ai_reply(message.content)
    ai_msg = Message(
        conversation_id=conversation_id,
        role="assistant",
        content=ai_reply
    )
    session.add(ai_msg)
    session.commit()
    session.refresh(ai_msg)

    return ai_msg
```

## `backend/app/routers/users.py`
```python
'''
创建用户路由
FastAPI 接口
实现两个功能：
- /register 注册（创建用户，密码加密）
- /login 登录(验证账号密码，返回 Token)
'''

# 导入接口工具：创建路由、抛错、连接数据库
from fastapi import APIRouter, HTTPException, Depends
# 导入数据库工具：会话、查询
from sqlmodel import Session, select
# 导入用户模型：数据库表、注册、返回格式、登录
from app.models.user import User, UserCreate, UserRead, UserLogin
# 导入密码加密工具：密码加密、验证密码、生成令牌
from app.utils.auth import hash_password, verify_password, create_access_token
# 导入数据库会话
from app.db import get_session

# 创建路由
'''
1. APIRouter()
- FastAPI 里用来分组接口的工具
- 作用：把用户相关的接口都放一起，不乱
2. prefix="/users"
- 前缀(prefix): 所有这个文件里的接口, URL 前面自动加上 /users
- 比如：
    你写 /register
    实际访问地址是 /users/register
    你写 /login
    实际访问地址是 /users/login
3. tags=["users"]
- 给接口分组起名
- 只作用在 API 文档(/docs) 里
- 让所有用户接口都放在 users 分类下，看着整齐
'''
router = APIRouter(prefix="/users", tags=["users"])

# 注册(register)
'''
1. @
- 装饰器标记
- 作用：把后面的东西，绑定下面的函数
- @ 绑定接口！让函数变成可被前端调用的 API!否则就是普通函数
2. router
- 你创建的用户路由分组
3. post
- 接口类型: POST
- (用来提交数据：注册、登录都用 POST)
4. "/register"
- 接口路径
- 完整地址：/users/register
5. response(响应)_model=UserRead
- 规定返回格式
- 只返回安全字段（不返回密码）
'''
@router.post("/register", response_model=UserRead)
# 接收前端传来的用户信息（UserCreate表的格式），并且自动拿到数据库连接（session）
def register(user: UserCreate, session: Session = Depends(get_session)):
    # 检查邮箱是否已注册
    # 创建查询语句 去User表里找邮箱等于前端传来的邮箱
    statement = select(User).where(User.email == user.email)
    # 执行查询语句statement并只取第一条
    existing_user = session.exec(statement).first()
    # if 存在existing_user用户 → 抛出错误raise HTTPException → 400 = 前端收到 “请求错误 → detail（细节） 错误提示信息
    # raise 就是 Python 里的 “主动抛出异常” 强制停止，报错
    if existing_user:
        raise HTTPException(status_code=400, detail="Email already registered")
    
    # 创建用户
    db_user = User(
        email=user.email,
        hashed_password=hash_password(user.password)
    )
    session.add(db_user)
    session.commit()
    session.refresh(db_user)  # 刷新 从数据库重新读取最新数据
    return db_user

@router.post("/login")
def login(user: UserLogin, session: Session = Depends(get_session)):
    statement = select(User).where(User.email == user.email)
    db_user = session.exec(statement).first()
    if not db_user or not verify_password(user.password, db_user.hashed_password):
        raise HTTPException(status_code=401, detail="Incorrect email or password")
    
    token = create_access_token(data={"sub": str(db_user.id)})
    # 告诉前端：这个 Token 的使用规则是 Bearer 规范，前端必须按这个格式带在请求头里！
    # 行业通用标准，就这么写
    # 前端调用需要登录的接口时，请求头里必须写： Authorization: Bearer 这里放你的Token
    return {"access_token": token, "token_type": "bearer"}

```

## `backend/app/utils/ai_service.py`
```python
"""
AI 回复服务模块

当前流程：
1. 先检查消息长度
2. 如果配置了 DEEPSEEK_API_KEY, 调用真实 DeepSeek API
3. 如果调用失败或没有 Key, 降级为模拟回复
4. 打印本次调用消耗的 Token 数
"""

import os
from openai import OpenAI
from pathlib import Path
from dotenv import load_dotenv

# 显式加载 backend 文件夹下的 .env
# Path(__file__)当前文件位置.resolve()转化为绝对路径.parent向上一级 * 3 即向上3级到backend目录
env_path = Path(__file__).resolve().parent.parent.parent / '.env'   # 指向 backend/.env
load_dotenv(env_path)

# ==================== 环境变量读取 ====================
# 从 .env 文件读取 Key 和接口地址，没有 Key 时 api_key 为 None
api_key = os.getenv("DEEPSEEK_API_KEY")
base_url = os.getenv("DEEPSEEK_BASE_URL", "https://api.deepseek.com/v1")

# 最大允许的消息长度（字符数）
MAX_MESSAGE_LENGTH = 1000


def generate_ai_reply(user_message: str) -> str:
    """
    根据用户消息生成 AI 回复。
    如果有 Key 则调用真实 API，否则降级为模拟回复。
    """
    # # 临时添加，调试用
    # print(f"🔑 读取到的 Key 前8位: {api_key[:8] if api_key else 'None'}")

    # ---------- 1. 长度限制 ----------
    if len(user_message) > MAX_MESSAGE_LENGTH:
        return "⚠️ 你的消息太长了，请控制在 1000 个字符以内。"

    # ---------- 2. 尝试调用真实 API ----------
    if api_key:
        try:
            # 创建 OpenAI 客户端（指向 DeepSeek 地址）
            client = OpenAI(api_key=api_key, base_url=base_url)

            # 发送请求，要求 AI 回答
            response = client.chat.completions.create(
                model="deepseek-chat",
                messages=[{"role": "user", "content": user_message[:MAX_MESSAGE_LENGTH]}],
                temperature=0.7,               # 控制随机性，0 最确定，1 最发散
            )

            # 打印本次消耗的 Token 数，方便监控额度
            usage = response.usage
            if usage:
                print(
                    f"📊 Token 消耗：总计 {usage.total_tokens}"
                    f"（输入 {usage.prompt_tokens}，输出 {usage.completion_tokens}）"
                )

            # 提取 AI 回复的文本
            return response.choices[0].message.content

        except Exception as e:
            # 网络错误、Key 无效、欠费等都会到这里
            print(f"❌ AI 调用失败：{e}")

    # ---------- 3. 降级回复 ----------
    return f"🤖 [模拟回复] 你说：“{user_message}”。（要获得真实 AI 回答，请在 .env 中配置 DEEPSEEK_API_KEY）"
```

## `backend/app/utils/auth.py`
```python
'''
项目里 ** 专门管 “密码加密 + 登录令牌”** 的工具
- 给密码加密（存数据库）
- 验证密码对不对（登录时用）
- 生成登录令牌（Token）（登录成功发一张通行证）
- 验证令牌是否有效（判断用户是否已登录）
'''


# 导入密码加密工具 用来给密码加密、验证密码
# 从「passlib 密码加密库」的「上下文工具」中，导入「CryptContext 加密器」
from passlib.context import CryptContext
# 导入生成 / 验证登录令牌（Token）的工具
# 从「jose工具库」中，导入「JWTError 错误」和「jwt 登录令牌」
# from jose import JWTError, jwt
import jwt
from datetime import datetime, timedelta, timezone
# 导入配置文件 拿密钥、过期时间、加密算法
from app.core.config import settings

# 新增
'''
Depends	                            FastAPI 依赖注入的核心，让函数自动执行
HTTPException	                    主动抛出错误(比如 401 未授权)
status	                            HTTP 状态码常量(401, 403 等)
HTTPBearer	                        告诉 FastAPI: “去请求头里找 Authorization: Bearer xxx”
HTTPAuthorizationCredentials	    取出 Bearer 后面的 token 内容
'''
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials


# 创建一个密码加密器 固定写法
# schemes=["bcrypt"] (schemes方案计划设计) 指定加密方式
# deprecated="auto"  自动处理旧加密方式，自动升级、自动兼容、自动安全
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# 密码加密
'''
1. def hash_password(...)
- def = 定义一个函数
- hash_password = 函数名字，意思是 给密码加密
2. password: str
- 这个函数需要你传一个参数
- password = 明文密码（比如用户输入的 123456）
- : str = 这个参数必须是文本 / 字符串
3. -> str
- 这个函数执行完会返回什么
- 意思是：返回一个字符串（加密后的乱码）
4. return
- 把结果返回出去
5. pwd_context.hash(password)
- pwd_context = 我们刚才创建的加密机
- .hash() = 加密动作
- password = 把你传进来的明文密码丢进去

定义一个叫 hash_password 的函数，接收一个明文密码，
用加密机把它加密，然后把加密后的乱码返回。
'''
def hash_password(password: str) -> str:
    return pwd_context.hash(password)

# 验证密码对不对 (verify检验)
# .verify() 是 CryptContext 加密机里自带的验密功能
def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

# 生成登录令牌
# (time delta 时间间隔/时间差)（expire期满）
'''
1. def create_access_token(data: dict, expires_delta: timedelta | None = None) -> str:
- dict = 数据类型为字典（装用户信息）
- timedelta | None = 可选的时间长度 过期时间可传可不传
- = None = 默认不传就是空，用配置里的 30 分钟
'''
def create_access_token(data: dict, expires_delta: timedelta | None = None) -> str:
    # 创建一个字典，把用户信息复制过来(避免改坏原来的数据)
    to_encode = data.copy()
    # 创建一个过期时间  (当前时间 + 过期时间or配置文件里默认30分钟)
    expire = datetime.now(timezone.utc) + (expires_delta or timedelta(minutes=settings.access_token_expire_minutes))
    # 把过期时间添加到字典里
    to_encode.update({"exp": expire})
    # 把字典里的数据用密钥加密成令牌
    # jwt.encode(要加密的内容, 密钥, 加密方式)
    return jwt.encode(to_encode, settings.secret_key, algorithm=settings.algorithm)

# 验证令牌是否有效
def verify_token(token: str) -> dict | None:
    try:
        # jwt.decode(要解密的内容, 密钥, 加密方式)
        payload = jwt.decode(token, settings.secret_key, algorithms=[settings.algorithm])
        return payload
    except jwt.PyJWTError:
        return None
    
# 新增get_current_user_id
'''
1. 目前你项目里，对话接口还不知道是谁在操作。比如：

- 创建对话时，前端要手动传 user_id: 1

- 任何人只要把 user_id 改成别人的数字，就能看到别人的对话

- 这非常不安全。我们要让后端自动从请求头里识别当前登录用户。

2. 它是如何工作的？
    1. 用户登录 → 拿到 JWT 令牌(token)
    2. 用户每次发请求 → 前端把 token 放在请求头 Authorization: Bearer <token>
    3. 后端 get_current_user_id 函数：
        a. 从请求头取出 token
        b. 调用 verify_token 解密，拿到用户的 id
        c. 如果 token 过期或无效 → 返回 401(请重新登录)
        d. 如果有效 → 返回用户 id
    4. 对话接口拿到这个 id, 就知道是谁在操作
'''
# 创建一个 "Bearer 令牌提取器"
# 它告诉 FastAPI：这个接口需要登录，请从请求头取出 Authorization: Bearer <token>
security = HTTPBearer()


def get_current_user_id(
    credentials: HTTPAuthorizationCredentials = Depends(security)
) -> int:
    """
    从请求头中取出 JWT 令牌，解密后返回当前登录用户的 id。
    如果令牌无效或过期，直接返回 401。
    """
    # 1. 取出 token 字符串
    token = credentials.credentials

    # 2. 调用我们之前写好的 verify_token 函数来解密
    payload = verify_token(token)

    # 3. 如果解密失败（过期/伪造/错误），payload 会是 None
    if payload is None:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token 无效或已过期，请重新登录",
        )

    # 4. 从解密后的数据里取出用户 id（注册时我们存在 "sub" 字段里）
    user_id = payload.get("sub")

    # 5. 返回整数类型的用户 id
    return int(user_id)

```

## `backend/app/db.py`
```python
'''
整个项目的数据库核心文件
1. SQLModel
- 所有数据表模型的爸爸类
- 你写的 User 类都要继承它
- 作用：自动把类变成数据库表
2. create_engine
- 创建数据库连接引擎
- 相当于打开数据库大门
- 没有它，连不上数据库
3. Session
- 数据库会话
- 相当于操作数据库的临时通道
- 增删改查全靠它
'''

# 导入 SQLModel 三件套：模型基类、数据库引擎、数据库会话  
#  建表 + 用来连接数据库 + 读写数据
from sqlmodel import SQLModel, create_engine, Session
from app.core.config import settings
# 创建数据库连接引擎
engine = create_engine(settings.database_url)

# 创建数据库表
def create_db_and_tables():
    # 写的所有模型（User 等），只要继承了 SQLModel(class User(SQLModel, table=True):) 
    # 就会自动被 “登记” 到 SQLModel.metadata 里
    SQLModel.metadata.create_all(engine)

# 创建数据库会话(给接口提供一个安全、自动关闭的数据库连接 session)
'''
with Session(engine) as session
- 创建一个数据库会话
- with = 自动管理
    → 用完自动关闭连接
    → 不会泄露、不会卡死
yield session
- 把 session 抛出去给接口用
- 这是一个 生成器函数
- FastAPI 的 Depends 专门用它
'''
def get_session():
    with Session(engine) as session:
        yield session
```

## `backend/app/main.py`
```python
"""
AI ChatFlow 后端应用入口
- 启动时自动创建数据库表
- 配置跨域、限流、路由挂载
"""

from contextlib import asynccontextmanager
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import JSONResponse

from app.db import create_db_and_tables
from app.routers import users, conversations

from slowapi import Limiter
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded


# ==================== 应用生命周期 ====================

@asynccontextmanager
async def lifespan(app: FastAPI):
    # 启动时：自动创建所有数据库表
    create_db_and_tables()
    print("✅ 数据库表已就绪")
    yield
    # 关闭时（可选）
    print("👋 应用关闭")


# ==================== 创建 FastAPI 实例 ====================

app = FastAPI(title="AI ChatFlow API", lifespan=lifespan)


# ==================== 全局速率限制器 ====================

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

# 当触发限流时，返回 429 状态码和友好提示
@app.exception_handler(RateLimitExceeded)
async def rate_limit_handler(request, exc):
    return JSONResponse(
        status_code=429,
        content={"detail": "请求太频繁了，请稍后再试。"},
    )


# ==================== 跨域中间件 ====================

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:5173",   # Vue 开发服务器
        "http://localhost",        # Docker 部署后的前端
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


# ==================== 挂载路由 ====================

app.include_router(users.router)
app.include_router(conversations.router)


# ==================== 健康检查接口 ====================

@app.get("/")
def root():
    return {"message": "Hello from AI ChatFlow API"}


@app.get("/health")
def health_check():
    return {"status": "ok"}
```

## `backend/tests/__init__.py`
```python

```

## `backend/tests/conftest.py`
```python
# 文件位置：backend/tests/conftest.py
# 作用：提供测试用的 FastAPI 客户端，每个测试函数都能用它发请求

import pytest
# TestClient 是 FastAPI 官方提供的测试工具。
# 它能把你的 FastAPI 应用“装进一个虚拟浏览器”，
# 你可以对它发请求，然后检查返回结果，完全不需要真正启动 uvicorn 服务器。
from fastapi.testclient import TestClient
from app.main import app

# 这是一个装饰器。
# 意思是：下面这个函数是一个“工具工厂”，
# 哪个测试函数需要 client，
# 就在参数里写 def test_xxx(client):
# pytest 会自动调这个函数，把造好的 client 传给你。
@pytest.fixture
def client():
    # 把我们的 app（FastAPI 应用实例）包装成 TestClient 对象，返回。
    # 每个测试拿到的 client 都可以发 GET/POST/DELETE 请求。
    return TestClient(app)

# 好处：所有测试函数共享同一个 client，不用每个文件都写一遍 TestClient(app)。
```

## `backend/tests/test_api.py`
```python
# 文件位置：backend/tests/test_api.py
# 作用：测试所有核心 API 接口（适配 JWT 鉴权）

import pytest

# 测试用户数据
test_user = {"email": "pytest@example.com", "password": "test123"}
# 全局保存 token 和对话 ID
token = None
conversation_id = None


class TestUserAPI:
    """用户注册与登录测试"""

    def test_register(self, client):
        # 向 /users/register 发一个 POST 请求，请求体是那个测试用户。TestClient 会直接把响应返回。
        response = client.post("/users/register", json=test_user)
        # 检查 HTTP 状态码是不是 200（成功）
        assert response.status_code == 200
        # 把返回的 JSON 解析成 Python 字典
        data = response.json()
        # 检查返回的邮箱是不是我们刚注册的邮箱
        assert data["email"] == test_user["email"]
        # 检查返回数据里有没有 id 字段（说明数据库生成了 ID）
        assert "id" in data

    # 重复注册应返回 400, 如果返回 200,说明后端没拦住, 测试会失败
    def test_register_duplicate(self, client):
        response = client.post("/users/register", json=test_user)
        assert response.status_code == 400

    # 登录成功并保存 token
    def test_login_success(self, client):
        global token
        response = client.post("/users/login", json={
            "email": test_user["email"],
            "password": test_user["password"]
        })
        assert response.status_code == 200
        data = response.json()
        assert "access_token" in data
        token = data["access_token"]   # 保存 token 给后续测试用

    # 密码错误应返回 401
    def test_login_wrong_password(self, client):
        response = client.post("/users/login", json={
            "email": test_user["email"],
            "password": "wrong"
        })
        assert response.status_code == 401


class TestConversationAPI:
    """对话与消息 API 测试（需要认证）"""

    # 辅助函数：生成带 token 的请求头
    def auth_headers(self):
        return {"Authorization": f"Bearer {token}"}

    def test_create_conversation(self, client):
        global conversation_id
        response = client.post(
            "/conversations/",
            json={"title": "测试对话"},
            headers=self.auth_headers()
        )
        assert response.status_code == 200
        data = response.json()
        assert data["title"] == "测试对话"
        conversation_id = data["id"]

    def test_get_conversations(self, client):
        response = client.get(
            "/conversations/",
            headers=self.auth_headers()
        )
        assert response.status_code == 200
        assert isinstance(response.json(), list)

    def test_send_message_and_ai_reply(self, client):
        global conversation_id
        response = client.post(
            f"/conversations/{conversation_id}/messages",
            json={"role": "user", "content": "你好"},
            headers=self.auth_headers()
        )
        assert response.status_code == 200
        ai_msg = response.json()
        assert ai_msg["role"] == "assistant"
        assert "你好" in ai_msg["content"]

    def test_get_messages_after_send(self, client):
        global conversation_id
        response = client.get(
            f"/conversations/{conversation_id}/messages",
            headers=self.auth_headers()
        )
        assert response.status_code == 200
        messages = response.json()
        assert len(messages) == 2
        assert messages[0]["role"] == "user"
        assert messages[1]["role"] == "assistant"

    def test_delete_conversation(self, client):
        global conversation_id
        response = client.delete(
            f"/conversations/{conversation_id}",
            headers=self.auth_headers()
        )
        assert response.status_code == 200
        # 再查应 404
        response = client.get(
            f"/conversations/{conversation_id}",
            headers=self.auth_headers()
        )
        assert response.status_code == 404
```

## `backend/.env`
```bash
DATABASE_URL=postgresql://chatflow_user:chatflow_123456@localhost:5432/chatflow

SECRET_KEY=change-me-to-a-random-string-in-production

ALGORITHM=HS256

ACCESS_TOKEN_EXPIRE_MINUTES=30
# Deepseek API Key
DEEPSEEK_API_KEY=sk-xxxxxyour-api-key
DEEPSEEK_BASE_URL=https://api.deepseek.com/v1
```

## `backend/Dockerfile`
```dockerfile
# 基于轻量 Python 镜像
FROM python:3.12-slim

WORKDIR /app

# 先复制依赖文件，利用 Docker 缓存
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制所有后端代码
COPY . .

# 启动命令
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## `backend/requirements.txt`
```txt
fastapi==0.115.6
uvicorn[standard]==0.34.0 
sqlmodel==0.0.22
psycopg2-binary==2.9.10
PyJWT[crypto]==2.10.1 
passlib==1.7.4
bcrypt==4.0.1
python-dotenv==1.0.1
pydantic-settings==2.7.1
openai==1.68.2
slowapi==0.1.9
httpx==0.27.0
pytest==8.3.4

```

# 前端代码frontend
**前端目录是生成的，不是自建的**
**建议以管理员身份打开 PowerShell**，在项目根目录下执行（确保你的终端路径最后是 `ai-chatflow`，而不是 `backend`）:
```bash
npm create vite@latest frontend -- --template vue
```
运行成功后，会在当前目录下生成一个 `frontend` 文件夹，里面是 Vue 3 的基础模板。

然后进入前端目录`frontend`, 安装所有依赖和需要的库：
```bash
cd frontend
npm install
npm install vue-router@4 axios
```
最后：
**删除** `src/components/HelloWorld.vue`
**删除** `src/assets/vue.svg`
**清空** `src/App.vue` 的内容（保留空文件）
**清空** `src/style.css` 的内容

**之后的没有的目录就自行创建**

## `frontend/src/api/index.js`
```js
// 文件位置：frontend/src/api/index.js
// 作用：统一封装后端请求，自动处理 JWT 和 401 跳转

import axios from "axios";

const apiClient = axios.create({
  baseURL: "http://localhost:8000",
  timeout: 10000,
});

// ==================== 请求拦截器 ====================
// 每次发请求前，自动从 localStorage 取出 token，放到请求头
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// ==================== 响应拦截器 ====================
// 每次收到响应后，如果状态码是 401，自动清除登录信息并跳转到登录页
apiClient.interceptors.response.use(
  (response) => response,                    // 正常响应原样返回
  (error) => {
    if (error.response?.status === 401) {
      const url = error.config?.url;
      // 登录和注册接口的 401 由各自的页面处理，不跳转
      if (url !== "/users/login" && url !== "/users/register") {
        localStorage.removeItem("token");
        localStorage.removeItem("userEmail");
        window.location.href = "/login";
      }
    }
    return Promise.reject(error);
  }
);

// ==================== 导出所有接口方法 ====================
export default {
  // ---- 用户认证 ----
  register(data) {
    return apiClient.post("/users/register", data);
  },
  login(data) {
    return apiClient.post("/users/login", data);
  },

  // ---- 对话管理 ----
  createConversation(data) {
    return apiClient.post("/conversations/", data);
  },
  // 后端自动识别当前用户，不再需要传 userId
  getConversations() {
    return apiClient.get("/conversations/");
  },
  deleteConversation(id) {
    return apiClient.delete(`/conversations/${id}`);
  },

  // ---- 消息管理 ----
  getMessages(conversationId) {
    return apiClient.get(`/conversations/${conversationId}/messages`);
  },
  sendMessage(conversationId, data) {
    return apiClient.post(`/conversations/${conversationId}/messages`, data);
  },
};

```

## `frontend/src/components/ChatWindow.vue`
```vue
<template>
    <div class="chat-window">
        <div class="messages" ref="msgContainer">
            <div v-for="msg in messages" :key="msg.id" :class="['message', msg.role]">
                <div class="content">{{ msg.content }}</div>
            </div>
        </div>
        <div class="input-area">
            <input v-model="inputText" @keyup.enter="send" placeholder="输入消息，按回车发送..." :disabled="sending" />
            <button @click="send" :disabled="sending">发送</button>
        </div>
    </div>
</template>

<script setup>
import { ref, watch, nextTick } from 'vue'

const props = defineProps({
    messages: Array,
    conversationId: Number
})

const emit = defineEmits(['send'])

const inputText = ref('')
const sending = ref(false)
const msgContainer = ref(null)

async function send() {
    const text = inputText.value.trim()
    if (!text || sending.value) return
    sending.value = true
    emit('send', text)
    // 等父组件发出后清空
    inputText.value = ''
    // 发送结束由父组件控制，这里简单延迟取消（或者通过 watch props 来判断，简单做法就是固定延迟后恢复）
    setTimeout(() => { sending.value = false }, 500)
}

// 自动滚到底部
watch(() => props.messages.length, async () => {
    await nextTick()
    if (msgContainer.value) {
        msgContainer.value.scrollTop = msgContainer.value.scrollHeight
    }
})
</script>


<style scoped>
.chat-window {
    flex: 1;
    display: flex;
    flex-direction: column;
    height: 100vh;
    background: #ffffff;
}

.messages {
    flex: 1;
    overflow-y: auto;
    padding: 2rem;
    max-width: 800px;
    margin: 0 auto;
    width: 100%;
}

.messages::-webkit-scrollbar {
    width: 4px;
}

.messages::-webkit-scrollbar-thumb {
    background: #d0d5dd;
    border-radius: 4px;
}

.message {
    margin-bottom: 1.5rem;
}

/* 用户消息 — 右侧对齐，蓝色气泡 */
.message.user {
    display: flex;
    justify-content: flex-end;
}

.message.user .content {
    background: #e8f0fe;
    color: #1a1a2e;
    padding: 0.8rem 1.2rem;
    border-radius: 12px 12px 4px 12px;
    max-width: 70%;
    word-wrap: break-word;
    line-height: 1.6;
    font-size: 0.95rem;
}

/* AI 消息 — 左侧对齐，无背景仅文字 */
.message.assistant {
    display: flex;
    justify-content: flex-start;
}

.message.assistant .content {
    background: transparent;
    padding: 0;
    max-width: 85%;
    word-wrap: break-word;
    line-height: 1.75;
    font-size: 0.95rem;
    color: #333;
}

/* 输入区域 — 居中悬浮 */
.input-area {
    display: flex;
    padding: 1rem 2rem 1.5rem;
    max-width: 800px;
    margin: 0 auto;
    width: 100%;
    gap: 10px;
    background: #ffffff;
    border-top: 1px solid #f0f0f0;
}

.input-area input {
    flex: 1;
    padding: 0.75rem 1.2rem;
    border: 1px solid #e0e0e0;
    border-radius: 30px;
    font-size: 0.95rem;
    outline: none;
    background: #f8f9fa;
    transition: all 0.2s;
}

.input-area input:focus {
    border-color: #1a73e8;
    background: #ffffff;
    box-shadow: 0 0 0 3px rgba(26, 115, 232, 0.1);
}

.input-area button {
    padding: 0.75rem 1.8rem;
    background: #1a73e8;
    color: white;
    border: none;
    border-radius: 30px;
    font-size: 0.9rem;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s;
}

.input-area button:hover {
    background: #1557b0;
}

.input-area button:disabled {
    background: #b0bec5;
    cursor: not-allowed;
}
</style>
```

## `frontend/src/components/ConversationList.vue`
```vue
<template>
  <div class="conversation-list">
    <div class="header">
      <h3>对话列表</h3>
      <button @click="$emit('new')" title="新建对话" class="new-btn">+</button>
    </div>
    <ul>
      <!-- 循环显示对话列表 循环数组 conversations Vue 循环必须加 key，用对话 id 就行 
       如果当前对话 id = 选中的 id  自动加上 active 样式（变灰、加粗） -->
      <li v-for="conv in conversations" :key="conv.id" :class="{ active: conv.id === activeId }"
        @click="$emit('select', conv.id)">
        <span>{{ conv.title }}</span>
        <!-- @click.stop = 阻止事件往上跳（避免点删除时触发切换对话） -->
        <button class="del-btn" @click.stop="$emit('delete', conv.id)">
          🗑️
        </button>
      </li>
    </ul>
    <!-- 底部退出区域 -->
    <div class="sidebar-footer">
      <button class="logout-btn" @click="$emit('logout')">退出登录</button>
    </div>
  </div>
</template>

<script setup>
defineProps({
  conversations: Array,
  activeId: Number,
});

defineEmits(["new", "select", "delete", "logout"]);
</script>

<style scoped>
.conversation-list {
  width: 260px;
  background: #1a1a2e;
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.header {
  padding: 1.2rem 1rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #2a2a4a;
}

.header h3 {
  margin: 0;
  font-size: 1rem;
  color: #e0e0e0;
  font-weight: 500;
}

.header button {
  background: #3a3a5c;
  color: #e0e0e0;
  border: none;
  width: 30px;
  height: 30px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1.2rem;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s;
}

.header button:hover {
  background: #4a4a6a;
}

ul {
  list-style: none;
  padding: 0.5rem 0.6rem;
  overflow-y: auto;
  flex: 1;
}

ul::-webkit-scrollbar {
  width: 4px;
}

ul::-webkit-scrollbar-thumb {
  background: #3a3a5c;
  border-radius: 4px;
}

li {
  padding: 0.7rem 0.8rem;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-radius: 8px;
  margin-bottom: 2px;
  color: #b0b0c0;
  font-size: 0.9rem;
  transition: all 0.15s;
}

li:hover {
  background: #252545;
  color: #e0e0e0;
}

li.active {
  background: #2a2a4a;
  color: #ffffff;
  font-weight: 500;
}

.del-btn {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 0.9rem;
  opacity: 0;
  transition: opacity 0.2s;
  color: #ff6b6b;
}

li:hover .del-btn {
  opacity: 0.8;
}

.del-btn:hover {
  opacity: 1;
}

.new-btn {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  background: #3a3a5c;
  color: #e0e0e0;
  border: none;
  font-size: 1.2rem;
  line-height: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background 0.2s;
}

.new-btn:hover {
  background: #4a4a6a;
}

/* 侧边栏底部 */
.sidebar-footer {
  padding: 0.8rem;
  border-top: 1px solid #2a2a4a;
}

.logout-btn {
  width: 100%;
  padding: 0.6rem 1rem;
  background: transparent;
  border: 1px solid #3a3a5c;
  border-radius: 8px;
  color: #b0b0c0;
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.2s;
}

.logout-btn:hover {
  background: #252545;
  color: #ff6b6b;
  border-color: #ff6b6b;
}

</style>

```

## `frontend/src/router/index.js`
```js
// 文件位置：frontend/src/router/index.js
// 作用：页面路由配置 + 登录守卫

import { createRouter, createWebHistory } from "vue-router";
import LoginView from "../views/LoginView.vue";   // 登录页面（稍后新建）
import ChatView from "../views/ChatView.vue";     // 聊天主页

const routes = [
  // 根路径自动跳转到登录页
  { path: "/", redirect: "/login" },

  // 登录页
  { path: "/login", name: "Login", component: LoginView },

  // 聊天页（需要登录）
  {
    path: "/chat",
    name: "Chat",
    component: ChatView,
    meta: { requiresAuth: true },   // 标记：此页面需要登录
  },
];

const router = createRouter({
  history: createWebHistory(),
  routes,
});

// ==================== 全局导航守卫 ====================
router.beforeEach((to, from, next) => {
  const token = localStorage.getItem("token");

  // 如果要访问的页面需要登录，但没有 token
  if (to.matched.some((record) => record.meta.requiresAuth) && !token) {
    next("/login");   // 强制跳转到登录页
  } else {
    next();           // 放行
  }
});

export default router;
```

## `frontend/src/views/ChatView.vue`
```vue
<template>
    <div class="chat-layout">
        <ConversationList :conversations="conversations" :activeId="activeConvId" @new="createConversation"
            @select="selectConversation" @delete="handleDelete" @logout="handleLogout" />

        <ChatWindow v-if="activeConvId" :messages="currentMessages" :conversationId="activeConvId"
            @send="sendMessage" />

        <!-- 空状态：仿 DeepSeek 的居中引导 -->
        <div v-else class="no-chat">
            <div class="welcome-box">
                <div class="welcome-icon">🤖</div>
                <h2>开始新对话</h2>
                <p>点击左侧「+ 新建」或选择一个已有的对话</p>
            </div>
        </div>

        <!-- 仅保留删除确认弹窗 -->
        <div v-if="showDeleteDialog" class="dialog-overlay" @click.self="showDeleteDialog = false">
            <div class="dialog-card">
                <h3>删除对话</h3>
                <p>确定要删除这个对话吗？删除后无法恢复。</p>
                <div class="dialog-actions">
                    <button class="cancel-btn" @click="showDeleteDialog = false">取消</button>
                    <button class="confirm-btn danger" @click="confirmDelete">确认删除</button>
                </div>
            </div>
        </div>
    </div>
</template>

<!--这是 Vue3 的语法，表示这里写逻辑代码 
1. 从 Vue 官方库里导入 3 个工具：
- ref：用来定义 “会变的变量”（比如输入框文字、是否正在发送）
- computed：用来定义 “会变化的变量”，比如当前对话的标题、当前对话的 ID 等
- onMounted：用来在页面加载完成后执行代码
-->
<script setup>  
import { ref, computed, onMounted } from 'vue'
import { useRouter } from "vue-router";
import ConversationList from '../components/ConversationList.vue'
import ChatWindow from '../components/ChatWindow.vue'
import api from '../api/index.js'


const conversations = ref([])
const activeConvId = ref(null)
const messagesMap = ref({})  // { convId: [msg, ...] }

const currentMessages = computed(() => {
    return messagesMap.value[activeConvId.value] || []
})

const showDeleteDialog = ref(false)
const pendingDeleteId = ref(null)
const router = useRouter();

// 初始化：加载用户对话列表
onMounted(async () => {
    try {
        const res = await api.getConversations()
        conversations.value = res.data
    } catch (e) {
        console.error(e)
    }
})

// 创建新对话
// 直接创建对话（不需要输入框）
async function createConversation() {
    try {
        const res = await api.createConversation({ title: "新对话" })
        conversations.value.unshift(res.data)
        activeConvId.value = res.data.id
        messagesMap.value[res.data.id] = []
    } catch (e) {
        console.error(e)
    }
}

// 选择对话，加载消息
async function selectConversation(id) {
    activeConvId.value = id
    if (!messagesMap.value[id]) {
        try {
            const res = await api.getMessages(id)
            messagesMap.value[id] = res.data
        } catch (e) {
            console.error(e)
        }
    }
}

// 发送消息
async function sendMessage(text) {
    if (!activeConvId.value) return
    try {
        // 1. 先乐观更新用户消息到界面
        const tempUserMsg = { id: Date.now(), role: 'user', content: text, conversation_id: activeConvId.value }
        if (!messagesMap.value[activeConvId.value]) {
            messagesMap.value[activeConvId.value] = []
        }
        messagesMap.value[activeConvId.value].push(tempUserMsg)

        // 2. 发送到后端，等待 AI 回复
        const res = await api.sendMessage(activeConvId.value, { role: 'user', content: text })
        // 后端返回的是 AI 回复的消息对象
        const aiMsg = res.data
        // 用后端返回的真实消息替换掉本地的临时用户消息
        // 简单做法：删除临时消息，把后端返回的用户消息（如果有）和 AI 消息都加进去
        // 但后端 send_message 只返回 AI 消息，所以我们需要再拉取一次完整消息列表以确保同步
        // 为了简单，我们拉取最新全部消息
        const updatedRes = await api.getMessages(activeConvId.value)
        messagesMap.value[activeConvId.value] = updatedRes.data
    } catch (e) {
        console.error(e)
    }
}

// 删除对话
// 点击删除按钮 → 显示确认弹窗
function handleDelete(id) {
    pendingDeleteId.value = id
    showDeleteDialog.value = true
}

// 确认删除
async function confirmDelete() {
    const id = pendingDeleteId.value
    if (!id) return
    try {
        await api.deleteConversation(id)
        conversations.value = conversations.value.filter(c => c.id !== id)
        delete messagesMap.value[id]
        if (activeConvId.value === id) activeConvId.value = null
    } catch (e) {
        console.error(e)
    } finally {
        showDeleteDialog.value = false
        pendingDeleteId.value = null
    }
}

function handleLogout() {
    localStorage.removeItem("token");
    localStorage.removeItem("userEmail");
    router.push("/login");
}

</script>

<style scoped>
.chat-layout {
    display: flex;
    height: 100vh;
    width: 100%;
    background: #ffffff;
}

.no-chat {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #ffffff;
    text-align: center;
    /* 确保子元素内联内容也居中 */
}

.welcome-box {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
    color: #888;
    max-width: 400px;
}

.welcome-icon {
    font-size: 4rem;
    margin-bottom: 1rem;
}

.welcome-box h2 {
    font-size: 1.5rem;
    color: #222;
    margin: 0 0 0.5rem;
    font-weight: 500;
}

.welcome-box p {
    font-size: 0.95rem;
    color: #999;
    margin: 0;
}

/* 删除确认弹窗 */
.dialog-card p {
    color: #666;
    font-size: 0.9rem;
    line-height: 1.6;
    margin: 0.5rem 0 1.2rem;
}

.dialog-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background: rgba(0, 0, 0, 0.35);
    display: flex;
    align-items: flex-start;
    justify-content: center;
    padding-top: 18vh;
    z-index: 1000;
}

.confirm-btn.danger {
    background: #dc3545;
}

.confirm-btn.danger:hover {
    background: #c82333;
}
</style>

```

## `frontend/src/views/LoginView.vue`
```vue
<template>
    <div class="login-container">
        <div class="login-card">
            <!-- Logo / 标题区 -->
            <div class="logo-area">
                <div class="logo-icon">🤖</div>
                <h1>AI ChatFlow</h1>
                <p class="subtitle">登录你的账号，开始智能对话</p>
            </div>

            <!-- 消息提示条 -->
            <div v-if="message.text" :class="['message-tip', message.type]">
                {{ message.text }}
            </div>

            <!-- 表单 -->
            <form @submit.prevent="submit">
                <div class="input-group">
                    <label>邮箱</label>
                    <input v-model="email" type="email" placeholder="name@example.com" required />
                </div>

                <div class="input-group">
                    <label>密码</label>
                    <input v-model="password" type="password" placeholder="请输入密码" required />
                </div>

                <button type="submit" class="submit-btn">
                    {{ isLogin ? "登录" : "创建账号" }}
                </button>
            </form>

            <p class="switch-text">
                {{ isLogin ? "还没有账号？" : "已有账号？" }}
                <a href="#" @click.prevent="switchMode">
                    {{ isLogin ? "立即注册" : "去登录" }}
                </a>
            </p>
        </div>
    </div>
</template>

<script setup>
import { ref, reactive } from "vue";
import { useRouter } from "vue-router";
import api from "../api";

const email = ref("");
const password = ref("");
const isLogin = ref(true);
const router = useRouter();

const message = reactive({ text: "", type: "success" });

function showMessage(text, type = "error") {
    message.text = text;
    message.type = type;
    setTimeout(() => {
        message.text = "";
    }, 5000);
}

function switchMode() {
    isLogin.value = !isLogin.value;
    message.text = "";
}

async function submit() {
    try {
        if (isLogin.value) {
            const res = await api.login({
                email: email.value,
                password: password.value,
            });
            localStorage.setItem("token", res.data.access_token);
            localStorage.setItem("userEmail", email.value);
            router.push("/chat");
        } else {
            await api.register({
                email: email.value,
                password: password.value,
            });
            showMessage("注册成功！请登录", "success");
            isLogin.value = true;
            email.value = "";
            password.value = "";
        }
    } catch (err) {
        const msg = err.response?.data?.detail || "操作失败，请重试";
        showMessage(msg, "error");
    }
}
</script>

<style scoped>
.login-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    background: #f5f6fa;
    padding: 20px;
}

.login-card {
    max-width: 420px;
    width: 100%;
    background: white;
    padding: 3rem 2.5rem;
    border-radius: 20px;
    box-shadow: 0 4px 24px rgba(0, 0, 0, 0.06);
}

.logo-area {
    text-align: center;
    margin-bottom: 2rem;
}

.logo-icon {
    font-size: 3rem;
    margin-bottom: 0.5rem;
}

.logo-area h1 {
    margin: 0;
    font-size: 1.6rem;
    color: #1a1a2e;
    font-weight: 700;
}

.subtitle {
    margin: 0.5rem 0 0;
    color: #888;
    font-size: 0.9rem;
}

/* 消息提示条 */
.message-tip {
    padding: 0.7rem 1rem;
    border-radius: 10px;
    margin-bottom: 1.2rem;
    font-size: 0.9rem;
}

.message-tip.success {
    background: #e8f5e9;
    color: #2e7d32;
    border: 1px solid #c8e6c9;
}

.message-tip.error {
    background: #ffebee;
    color: #c62828;
    border: 1px solid #ffcdd2;
}

/* 输入框组 */
.input-group {
    margin-bottom: 1.2rem;
    text-align: left;
}

.input-group label {
    display: block;
    margin-bottom: 0.4rem;
    font-size: 0.85rem;
    color: #555;
    font-weight: 500;
}

.input-group input {
    display: block;
    width: 100%;
    padding: 0.75rem 1rem;
    border: 1.5px solid #e0e0e0;
    border-radius: 12px;
    font-size: 0.95rem;
    box-sizing: border-box;
    outline: none;
    transition: border-color 0.2s, box-shadow 0.2s;
}

.input-group input:focus {
    border-color: #1a73e8;
    box-shadow: 0 0 0 3px rgba(26, 115, 232, 0.08);
}

.submit-btn {
    width: 100%;
    padding: 0.85rem;
    background: #1a73e8;
    color: white;
    border: none;
    border-radius: 12px;
    font-size: 1rem;
    font-weight: 500;
    cursor: pointer;
    margin-top: 0.5rem;
    transition: background 0.2s;
}

.submit-btn:hover {
    background: #1557b0;
}

.switch-text {
    margin-top: 1.5rem;
    margin-bottom: 0;
    font-size: 0.9rem;
    color: #888;
}

.switch-text a {
    color: #1a73e8;
    text-decoration: none;
    font-weight: 600;
}

.switch-text a:hover {
    text-decoration: underline;
}
</style>
```

## `frontend/src/App.vue`
```vue
<template>
  <!--这是网页结构-->
  <div id="app">
    <!-- 这是整个项目的最外层盒子所有内容都放这里面 -->
    <router-view />
    <!-- 核心中的核心！这就是路由的 “显示窗口”! 你之前配置的:path: '/', component: ChatView -->
  </div>
</template>

<style>
body {
  margin: 0;
  /* 去掉网页默认边距 */
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  /* 统一字体 */
}

#app {
  height: 100vh;
  /* 高度占满整个屏幕 */
  display: flex;
  /* 弹性布局（方便聊天界面排版） */
}
</style>

```

## `frontend/src/main.js`
```js
/*
这是 Vue 项目的入口文件（相当于整个项目的总开关）
创建 Vue 项目 → 安装路由 → 把项目挂载到页面上运行起来
1. import { createApp } from "vue";
    - 从 Vue 本身，导入创建应用的方法
    - createApp = 造一个 Vue 项目
2. import App from "./App.vue";
    - 导入根组件
    - App.vue = 整个项目的大总管页面
    - 所有页面都放在它里面
3. import router from "./router";
    - 导入路由
    - 让 Vue 知道：网址切换要显示哪个页面
4. const app = createApp(App);
    - 创建 Vue 应用实例
    - 相当于：项目启动了
5. app.use(router);
    - 给 Vue 安装路由插件
    - 告诉项目：我要用路由跳转页面！
    - **这一步必须写**，否则路由不生效
6. app.mount("#app");
    - 把 Vue 项目挂载到网页上
    - 找到网页里 id="app" 的标签
    - 把整个项目渲染进去
*/

/* 
1. 用生活比喻（秒懂）
    - createApp = 买一套房子
    - App.vue = 房子主体结构
    - router = 房子里的房间导航
    - app.use(router) = 把导航装到房子里
    - app.mount("#app") = 把房子放到地基上，开始住人
2. 这 5 行代码的执行流程（超级清晰）
    - 造一个 Vue 项目
    - 把根页面 App.vue 装进去
    - 把路由（导航）装进去
    - 把整个项目渲染到网页上
    - 项目成功跑起来！
3. 所有插件（路由、组件库等）都必须用 app.use () 安装
   最后必须 mount，否则页面一片空白
4. 启动项目 + 启用路由 + 渲染页面
*/

import { createApp } from "vue";
import App from "./App.vue";
import router from "./router";

const app = createApp(App);
app.use(router);
app.mount("#app");

```

## `frontend/Dockerfile`
```dockerfile
# 阶段 1：编译前端资源
FROM node:20-alpine as build

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# 阶段 2：用 nginx 提供静态文件
FROM nginx:alpine

# 把我们的 nginx 配置复制进容器，覆盖默认配置
COPY nginx.conf /etc/nginx/conf.d/default.conf

# 从构建阶段复制编译好的静态文件
COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## `frontend/nginx.conf`
```conf
server {
    listen 80;
    server_name localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;  # 关键行：自动回退到 index.html
    }
}
```

# 最后的最后：**两个.env文件里的账号和密码要改,ai_key要填自己申请的并且.gitignore文件里一定要忽略.env文件**
