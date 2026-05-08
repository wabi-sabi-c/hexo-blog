---
title: step3-conversation
date: 2026-05-08 17:59:20
tags: [Python Project]
categories: [ai-chatflow]
---

# 🚀 Step 3：对话聊天模块
下一步，用户登录后要能开始聊天。一个聊天应用的最小核心是：

- `Conversation`（对话）：用户说“我要开一个新话题”，比如“学习 Python”。

- `Message`（消息）：在这个话题下，用户问一句话，AI 回答一句话。

对应到代码里，我们需要：

- `models/conversation.py` → 定义 `Conversation` 和 `Message` 这两个表的“设计图”

- `routers/conversations.py` → 写增删改查接口（创建对话、查看消息、发消息等）

在 `main.py` 里挂载这个新路由，让 FastAPI 知道“多了一扇门”

<!--more-->

## 第一步：创建对话和消息模型
新建文件 `backend/app/models/conversation.py`,内容如下：
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

## 第二步：创建对话和消息的 API 路由 接口
新建文件 `backend/app/routers/conversations.py`,内容如下：
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
'''

# APIRouter: 创建一组接口、HTTPException: 异常处理、Depends: 自动连接数据库(yield session)
from fastapi import APIRouter, HTTPException, Depends
# Session: 数据库会话操作数据库、select：查询语句
from sqlmodel import Session, select
from app.models.conversation import Conversation, Message
from app.db import get_session
# tags: 接口分组名：conversations  prefix: 接口前缀 -> 所有接口路径前面自动加 /conversations
router = APIRouter(prefix="/conversations", tags=["conversations"])

# ---------- 对话相关 ----------

# response(响应)
@router.post("/", response_model=Conversation)
def create_conversation(conversation: Conversation, session: Session = Depends(get_session)):
    # 现在先不验证用户身份，硬性要求必须传 user_id，后续会加上 JWT 自动获取
    if not conversation.user_id:
        raise HTTPException(status_code=400, detail="user_id is required")
    session.add(conversation)
    session.commit()
    session.refresh(conversation)
    return conversation

@router.get("/", response_model=list[Conversation])
def get_conversations(user_id: int, session: Session = Depends(get_session)):
    # order_by: 排序  desc: 倒序、从大到小   最新创建的排在最前面
    statement = select(Conversation).where(Conversation.user_id == user_id).order_by(Conversation.created_at.desc())
    # exec(): 执行语句 .all(): 获取所有结果
    return session.exec(statement).all()

# {conversation_id}这是路径参数, FastAPI自带的功能, 自动帮你做匹配
# {conversation_id} = 占位符 = FastAPI 自动识别 = 自动匹配同名函数参数 
# 如: /conversations/123 -> 那么conversation_id = 123
@router.get("/{conversation_id}", response_model=Conversation)
def get_conversation(conversation_id: int, session: Session = Depends(get_session)):
    # get(): 查询单条数据
    conv = session.get(Conversation, conversation_id)
    if not conv:
        raise HTTPException(status_code=404, detail="Conversation not found(发现)")
    return conv

@router.delete("/{conversation_id}")
def delete_conversation(conversation_id: int, session: Session = Depends(get_session)):
    conv = session.get(Conversation, conversation_id)
    if not conv:
        raise HTTPException(status_code=404, detail="Conversation not found")
    # 删除对话时，它下面的消息（message）还绑着这个对话的 conversation_id。
    # 数据库试图把它改成 NULL，但不允许，于是就报错了。一句话：我们得先删掉“子消息”，再删“父对话”。
    # ✅ 先删除该对话下的所有消息
    messages = session.exec(
        select(Message).where(Message.conversation_id == conversation_id)
    ).all()
    for msg in messages:
        session.delete(msg)
    
    # ✅ 再删除对话本身
    session.delete(conv)
    session.commit()
    return {"ok": True}

# ---------- 消息相关 ----------

@router.get("/{conversation_id}/messages", response_model=list[Message])
def get_messages(conversation_id: int, session: Session = Depends(get_session)):
    conv = session.get(Conversation, conversation_id)
    if not conv:
        raise HTTPException(status_code=404, detail="Conversation not found")
    statement = select(Message).where(Message.conversation_id == conversation_id).order_by(Message.created_at)
    return session.exec(statement).all()

@router.post("/{conversation_id}/messages", response_model=Message)
def send_message(conversation_id: int, message: Message, session: Session = Depends(get_session)):
    # 简单校验：消息必须属于该对话
    message.conversation_id = conversation_id
    # 暂时只保存用户消息，AI 回复后期集成
    session.add(message)
    session.commit()
    session.refresh(message)
    return message

```

## 第三步：挂载路由
修改 `backend/app/main.py`, 在现有的 `app.include_router(users.router)` 下面加上一行：
```python
from app.routers import conversations   # 新增导入
app.include_router(conversations.router)
```

## 第四步：测试
确保你的 FastAPI 在运行（如果没启动，重新 `uvicorn app.main:app --reload --port 8000`）,
然后打开 `http://localhost:8000/docs`你会看到新出现的 conversations 标签, 里面有 6 个接口。我们按顺序测试：
1. 创建对话：
- 点击 `POST /conversations/ → "Try it out"`
- 请求体改为：
    ```json
    {
        "title": "学习 Python",
        "user_id": 1
    }
    ```
- 点击 `Execute`
- 响应结果：
    **往下滚动看 Responses**
    >    Server response 下面有一个 Responses 部分。
    >
    >    你会看到 "Code" 为 200 的栏，下面 "Details" 里的 "Response body" 就是服务器返回的 JSON 数据。      
    >   成功的返回应该类似：
    ```json
    {
        "title": "学习 Python",
        "user_id": 1,
        "id": 1,
        "created_at": "xxxx-xx-xxTxx:xx:xx.xxxxxx"  
    }
    ```

2. 获取一下这个用户的所有对话
- 点击 `GET /conversations/ → "Try it out"`
- 在 `user_id` 参数框输入 `1` → `Execute`
- 响应结果：是上面那个创建的对话

3. 发一条消息
- 点击 `POST /conversations/{conversation_id}/messages/ → "Try it out"`
- `conversation_id` 填刚创建对话的 id（比如 1）
- 请求体改为：
    ```json
    {
        "role": "user",
        "content": "Python 里怎么读取文件？"
    }
    ```
- 点击 `Execute`
- 响应结果：返回 200，消息保存成功

4. 获取对话里的所有消息
- 点击 `GET /conversations/{conversation_id}/messages/ → "Try it out"`
- `conversation_id` 填刚创建对话的 id（比如 1）
- 点击 `Execute`
- 响应结果:
    ```json
    
    [
        {
          "content": "Python 里怎么读取文件？",
          "id": 1,
          "role": "user",
          "conversation_id": 1,
          "created_at": "xxxx-xx-xxTxx:xx:xx.xxxxxx"
        }
    ]
    ```

# 总结
- 恭喜你，你已经成功创建了一个对话系统。
- 创建对话、获取对话列表、发送消息、获取对话里的所有消息、删除对话
- 流程很相似, 1.在 models/ 下为每个功能建一个模型文件  2.在 routers/ 下为每个功能建一个路由文件  3.在 main.py 里挂载这些路由
