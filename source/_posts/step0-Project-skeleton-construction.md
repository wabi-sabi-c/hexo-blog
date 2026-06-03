---
title: step0-Project_skeleton_construction
date: 2026-05-11 16:03:45
tags: [Python Project]
categories: [ai-ledgerAssistant]
---

# 🚀 step0  环境与项目骨架搭建
这个项目(FastAPI)是AI识别自动录入的台账工具：AI识别 → 页面展示 → 自动匹配 + 手动修正 → 确认写入

<!--more-->

## 一、环境搭建
1. 创建项目目录并初始化
```powershell
cd Project存放目录
mkdir ledger-assistant
code .   # 打开 VSCode
```
2. 创建虚拟环境
```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```
3. 安装依赖包
创建`requirements.txt`, 粘贴以下内容:
```bash
fastapi==0.115.6
uvicorn[standard]==0.34.0
sqlmodel==0.0.22
psycopg2-binary==2.9.10
python-dotenv==1.0.1
pydantic-settings==2.7.1
python-multipart==0.0.20
paddleocr==2.10.0
paddlepaddle==3.0.0
openpyxl==3.1.5
jinja2==3.1.6
```

<details>

<summary>点击展开依赖包说明</summary>

1. fastapi: 用于创建FastAPI
2. uvicorn: 用于运行FastAPI
3. sqlmodel: 用于数据库操作
4. psycopg2-binary: 用于PostgreSQL数据库操作
5. python-dotenv: 用于加载环境变量(.env文件)
6. pydantic-settings: 用于配置环境变量(.env文件)
7. python-multipart: 用于处理图片文件上传
8. openpyxl: 用于处理Excel文件
9. jinja2: 用于生成HTML文件 网页模板渲染引擎
10. paddeocr: baidu OCR识别引擎
11. paddlepaddle: baidu paddleOCR的底层深度学习框架

</details>

在安装依赖包
```powershell
pip install -r requirements.txt
```

## 二、项目骨架搭建
###  项目结构
```bash
ledger-assistant/               ← 项目根目录
├── .env                        ← 数据库密码、AI Key 等
├── .gitignore
├── requirements.txt            ← Python 依赖清单
├── docker-compose.yml          ← 数据库 + 后端
├── app/
│   ├── main.py                 ← FastAPI 入口
│   ├── db.py                   ← 数据库连接
│   ├── models/
│   │   └── ledger.py           ← 台账数据表
│   ├── routers/
│   │   └── ledger.py           ← 台账 API 接口
│   ├── utils/
│   │   ├── ocr.py              ← PaddleOCR 封装
│   │   └── matcher.py          ← 品名模糊匹配逻辑
│   └── templates/
│       └── index.html          ← 手机端 H5 页面（拍照+复核）
└── data/
    └── template.xlsx           ← 你的 Excel 台账模板
```

### 创建项目骨架
#### 1. `.env`
创建 `.env` 文件，并添加如下内容：
```bash
DATABASE_URL=postgresql://ledger_user:ledger_password@localhost:5432/ledger_db
```

#### 2. `.gitignore`
创建 `.gitignore` 文件，并添加如下内容：
```bash
__pycache__/
*.pyc
.env
venv/
.venv/
node_modules/
dist/
.pytest_cache/
*.db
postgres_data/
```

#### 3. `docker-compose.yml`
创建 `docker-compose.yml` 文件，并添加如下内容：
```yaml
version: '3.8'
services:
  db:
    image: postgres:16-alpine
    container_name: ledger_db
    restart: unless-stopped
    environment:
      POSTGRES_USER: ledger_user
      POSTGRES_PASSWORD: ledger_password
      POSTGRES_DB: ledger_db
    ports:
      - "5433:5432"  # 注意：避免和你之前的 chatflow 冲突，用 5433
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

#### 4. `app/__init__.py`、`app/models/__init__.py`、`app/routers/__init__.py`、`app/utils/__init__.py`
这些全是空文件，直接创建即可。(可自行搜索命令行一次性创建不同文件目录下同一文件)

#### 5. `app/db.py`
创建`app`目录并创建`app/db.py`文件, 粘贴如下内容：
```python
from sqlmodel import SQLModel, create_engine, Session
import os

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://ledger_user:ledger_password@localhost:5433/ledger_db")

engine = create_engine(DATABASE_URL)

def create_db_and_tables():
    SQLModel.metadata.create_all(engine)

def get_session():
    with Session(engine) as session:
        yield session
```

#### 6. `app/models/ledger.py`
创建`app/models`目录并创建`app/models/ledger.py`文件, 粘贴如下内容：
```python
from sqlmodel import SQLModel, Field
# 导入一个类型注解标记：Optional
# 这个变量 / 返回值 可以是指定类型，也可以是 None
from typing import Optional
from datetime import datetime, timezone

class LedgerRecord(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    product_name: str          # 产品名称
    specification: str         # 规格
    quantity: str              # 数量（保留字符串，因为可能有“斤、个”等单位）
    supplier: str = "xxxx物流配送有限公司"  # 默认供货商
    contact: str = "15xxxxxxxxx"
    purchase_date: datetime = Field(default_factory=lambda: datetime.now(timezone.utc))
    created_at: datetime = Field(default_factory=lambda: datetime.now(timezone.utc))
```

#### 7. `app/utils/orc.py`（占位版，后面再实现真正OCR）
创建`app/utils`目录并创建`app/utils/orc.py`文件，内容如下：
```python
def recognize_image(image_bytes: bytes) -> list[dict]:
    """
    模拟 OCR 识别，返回示例数据。
    后续将替换为 PaddleOCR 真实调用。
    """
    return [
        {"product_name": "土豆", "specification": "斤", "quantity": "100"},
        {"product_name": "白菜", "specification": "斤", "quantity": "60"},
    ]
```

#### 8. `app/utils/matcher.py`（占位版）
创建`app/utils/matcher.py`文件，并添加以下内容：
```python
def match_products(ocr_items: list[dict], known_products: list[str]) -> list[dict]:
    """
    简单模糊匹配占位，后续实现真正的匹配逻辑。
    当前直接把 OCR 结果原样返回。
    """
    return ocr_items
```

#### 9. `app/routers/ledger.py`
创建`app/routers`目录并创建`app/routers/ledger.py`文件, 粘贴以下内容：
```python
from fastapi import APIRouter, UploadFile, File, Depends
from sqlmodel import Session
from app.db import get_session
from app.models.ledger import LedgerRecord
from app.utils.ocr import recognize_image

router = APIRouter(prefix="/ledger", tags=["ledger"])

@router.post("/upload")
async def upload_image(file: UploadFile = File(...)):
    """
    接收手机拍照上传的图片，调用 OCR 识别，
    返回识别结果 JSON，供前端页面展示复核。
    """
    contents = await file.read()
    items = recognize_image(contents)
    return {"items": items, "count": len(items)}

@router.get("/records")
def get_records(session: Session = Depends(get_session)):
    """获取所有台账记录（后续实现）"""
    return []
```

<details>

<summary>点击代码解释</summary>

1. 导入部分
`from fastapi import APIRouter, UploadFile, File, Depends`
- 大白话：从 FastAPI 框架里导入 4 个必备工具
- 作用：
    1. APIRouter：创建接口分组
    2. UploadFile：接收上传的文件（图片 / 文档）
    3. File(...)：标记参数是上传文件
    4. Depends：依赖注入（自动帮你调用函数）
- 官方注解：FastAPI 核心工具，用于构建 API 接口
`from sqlmodel import Session`
- 大白话：导入数据库会话
- 作用：用来操作数据库（增删改查）
- 官方用法：Session 是数据库连接的 “工作窗口”
`from app.db import get_session`
- 大白话：导入获取数据库连接的工具函数
- 作用：自动创建 / 关闭数据库连接
- 原理：依赖注入，自动给接口提供数据库会话
`from app.models.ledger import LedgerRecord`
- 大白话：导入台账表的模型（数据库表结构）
- 作用：对应数据库里的 ledger_record 表
- 原理：ORM 模型，把数据库表变成 Python 类
`from app.utils.ocr import recognize_image`
- 大白话：导入图片文字识别工具
- 作用：输入图片 → 输出文字（OCR）
- 原理：调用 AI 识别图片中的文字内容

2. 创建路由
`router = APIRouter(prefix="/ledger", tags=["ledger"])`
- 大白话：创建一组以 /ledger 开头的接口
- 参数解释：
    - prefix="/ledger"：所有接口前面自动加 /ledger
    - tags=["ledger"]：接口文档里分组显示
- 官方用法：用于接口模块化、分文件管理
- 原理：把一堆接口归为一组，方便维护
3. 上传图片接口
`@router.post("/upload")`
- 大白话：定义一个 上传图片的接口
- 请求方式：POST（适合上传数据）
- 最终地址：/ledger/upload
- 官方用法：@router.post(路径) 注册接口
`async def upload_image(file: UploadFile = File(...))`
- 大白话：异步函数，接收前端上传的图片
- async：异步，不卡住程序（适合文件 IO）
- UploadFile：FastAPI 专门接收上传文件的类型
- File(...)：表示这个参数必须上传，不能为空
- 官方用法：`async def 函数名(参数: UploadFile = File(...))`
`contents = await file.read()`
- 大白话：把上传的图片读成二进制数据
- await：等待读取完成（异步操作）
- 原理：文件读取是 IO 操作，必须异步等待
`items = recognize_image(contents)`
- 大白话：把图片丢给 OCR，返回识别出来的清单
- 输入：图片二进制
- 输出：识别后的文字列表（商品、数量等）
- 原理：调用你写的 OCR 工具函数
`return {"items": items, "count": len(items)}`
- 大白话：返回 JSON 给前端
- 格式：
```json
{
  "items": [识别结果],
  "count": 多少条
}
```
- 原理：FastAPI 自动把字典转 JSON

</details>

#### 10. `app/main.py`
创建`app/main.py`
```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from fastapi.responses import HTMLResponse
from app.db import create_db_and_tables
from app.routers import ledger

@asynccontextmanager
async def lifespan(app: FastAPI):
    create_db_and_tables()
    yield

app = FastAPI(title="智能台账助手", lifespan=lifespan)

app.include_router(ledger.router)

@app.get("/", response_class=HTMLResponse)
async def index():
    """手机端 H5 页面（占位）"""
    return """
    <!DOCTYPE html>
    <html>
    <head><meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
    <title>智能台账助手</title></head>
    <body>
    <h1>智能台账助手</h1>
    <p>项目启动成功！</p>
    </body>
    </html>
    """
```

#### 11. `templates/index.html`（暂时留空，后续会做完整界面）
在根目录下创建`templates`目录，并创建`templates/index.html`文件

## 三、启动数据库和项目
1. 启动数据库
在项目根目录`ledger-assistant`终端下执行命令：
```powershell
docker-compose up -d
```
2. 启动FastAPI
```powershell
uvicorn main:app --reload --port 8001
```
3. 验证启动成功
访问`http://localhost:8001/`, 看到“智能台账助手”
访问`http://localhost:8001/docs`, 查看接口文档, 可以看到ledger接口(POST GET)


✅ 完成这章后该有的状态
- 项目骨架已建立

- 一个最简单的 FastAPI 应用已启动(`CTRL + C`停止后端、`docker-compose down`停止docker容器)

- 占位的 OCR 和匹配逻辑，方便后续替换


