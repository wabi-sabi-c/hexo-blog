---
title: step2-login&registration
date: 2026-05-08 10:30:08
tags: [Python Project]
categories: [ai-chatflow]
---

# 🚀 Step 2：用户认证模块（注册 & 登录）
这一阶段你将产出：

- 用户数据表（SQLModel 定义）

- 两个核心 API：POST /register 和 POST /login

- JWT 令牌生成与验证逻辑

<!--more-->

## 第一步：创建数据库基类和用户模型
在 `backend/app` 下新建文件夹 `models`，然后新建文件 `backend/app/models/user.py`,内容：
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

## 第二步：创建认证工具模块
在 `backend/app` 下新建文件夹 `utils`，然后新建文件 `backend/app/utils/auth.py`,内容：
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
from jose import JWTError, jwt
from datetime import datetime, timedelta, timezone
# 导入配置文件 拿密钥、过期时间、加密算法
from app.core.config import settings

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
    except JWTError:
        return None

```

## 第三步：创建用户路由
在 `backend/app` 下新建文件夹 `routers`，然后新建文件 `backend/app/routers/users.py`,内容：
```python
'''
创建用户路由
FastAPI 接口
实现两个功能：
- /register 注册（创建用户，密码加密）
- /login 登录（验证账号密码，返回 Token）
'''

# 导入接口工具：创建路由、抛错、依赖注入
from fastapi import APIRouter, HTTPException, Depends
# 导入数据库工具：查询、会话
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
- 前缀(prefix)：所有这个文件里的接口，URL 前面自动加上 /users
- 比如：
    你写 /register
    实际访问地址是 /users/register
    你写 /login
    实际访问地址是 /users/login
3. tags=["users"]
- 给接口分组起名
- 只作用在 API 文档（/docs） 里
- 让所有用户接口都放在 users 分类下，看着整齐
'''
router = APIRouter(prefix="/users", tags=["users"])

# 注册(register)
'''
1. @
- 装饰器标记
- 作用：把后面的东西，绑定下面的函数
- @ 绑定接口！让函数变成可被前端调用的 API！否则就是普通函数
2. router
- 你创建的用户路由分组
3. post
- 接口类型：POST
- （用来提交数据：注册、登录都用 POST）
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

## 第四步：创建数据库连接与表初始化
在 `backend/app` 下新建文件 `backend/app/db.py`,内容：
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

## 第五步：更新 main.py，挂载路由和初始化数据库
在 `backend/app` 下更新 `main.py`文件,内容：
```python
from fastapi import FastAPI
from contextlib import asynccontextmanager
from fastapi.middleware.cors import CORSMiddleware
from app.routers import users
from app.db import create_db_and_tables


# 1. 定义 lifespan 开机自启（启动 + 关闭 事件）
# 为什么要用@asynccontextmanager因为@app.on_event("startup")被新版的FastAPI所弃用
@asynccontextmanager
# 固定写法
async def lifespan(app: FastAPI):
    # ======================
    # 【项目启动时执行】
    # ======================
    # 创建表
    create_db_and_tables()
    print("数据库表创建成功！")
    
    yield  # 这里会把控制权交给 FastAPI 运行
    
    # ======================
    # 【项目关闭时执行】（可选）
    # ======================
    print("项目关闭！")
# 2. 创建 FastAPI 时，直接传入 lifespan
'''
1. app = FastAPI( )
- 创建一个 FastAPI 应用实例
- 就是你整个后端项目的总入口
2. lifespan=
- FastAPI 提供的一个参数
- 作用：设置项目的生命周期函数
3. lifespan（第二个）
- 就是你自己写的那个生命周期函数
4. 为什么要写app = FastAPI(lifespan=lifespan)
- 因为FastAPI 默认是不执行生命周期函数的，所以要手动传给FastAPI
- 因为 FastAPI 默认是单进程的，所以要设置生命周期函数，才能实现多进程
'''
app = FastAPI(title="AI ChatFlow API", lifespan=lifespan)

# 跨域配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


# 挂载路由
# 把你写在别的文件里的接口（比如 users 路由），挂到主程序 app 上，让 FastAPI 知道这些接口存在
app.include_router(users.router)

@app.get("/")
def read_root():
    return {"message": "Hello from AI ChatFlow API"}

@app.get("/health")
def health_check():
    return {"status": "ok"}

```

## 第六步：启动项目
确保**虚拟环境已激活**，并且你在 `backend` 目录下:
终端输入：
```bash
uvicorn app.main:app --reload --port 8000
```
访问：`http://localhost:8000/docs` 会看到新增的 `POST /users/register` 和 `POST /users/login`。

### 测试：注册&登录
1. 注册：
- 点击 `POST /users/register → "Try it out"`
- 请求体改为：
    ```json
    {
        "email": "test@example.com",
        "password": "123456"
    }
    ```
- 点击 `Execute`
- 响应结果：
    **往下滚动看 Responses**
    >    Server response 下面有一个 Responses 部分。
    >
    >    你会看到 "Code" 为 200 的栏，下面 "Details" 里的 "Response body" 就是服务器返回的 JSON 数据。      
    >   成功的返回应该类似：
    >```json  
     {
     "email": "test@example.com",
     "id": 1,
     "created_at": "2026-05-07T09:30:00"
     }
     ```
    >   恭喜！你已成功注册！

2. 登录： 跟注册一样，点击 `POST /users/login → "Try it out"`


# 总结：
1. 明确该模块需要哪些功能？注册和登录
2. 确定功能实现逻辑：
   1. **注册需要什么？登录需要什么？** 如注册需要用户账户和密码 -> 
   2. 用户账户和密码都保存在数据库中 -> 
   3. 那我们就要写用户模型来创建用户数据库表 -> 
   4. 创建`backend/app/models`文件夹, 再新建文件 `backend/app/models/user.py` -> 
   5. **存储数据的地方有了那还需要什么？**-> 
   6. 安全性就要考虑一下，密码要加密存储 -> 
   7. 创建一个加密工具 -> 
   8. 创建`backend/app/utils`文件夹来放各种工具，再新建 `backend/app/utils/auth.py`创建这个加密工具
   9. **账户密码都保存的地方有了且可以加密处理，那现在呢？** ->
   10. 前端接收用户账户和密码,后端获取用户账户和密码 ->
   11. 创建用户路由 接口 ->
   12. 在 `backend/app` 下新建文件夹 `routers`，然后新建文件 `backend/app/routers/users.py` ->
   13. **接收到用户账户和密码后，需要做哪些事情？当然是实实在在的存储到数据库中** ->
   14. 在 `backend/app` 下新建文件 `backend/app/db.py` ->
   15. 创建数据库连接引擎，开始操作数据库 ->
   16. **最后**
   17. 更新 `backend/app/main.py` 文件，挂载路由和初始化数据库 ->
   18. 启动项目
## 总得来说就是：1.在 models/ 下为每个功能建一个模型文件  2.在 routers/ 下为每个功能建一个路由文件  3.在 main.py 里挂载这些路由
