---
title: step6-automated_tests
date: 2026-05-10 13:29:23
tags: [Python Project]
categories: [ai-chatflow]
---

# 🧪 Step 6：编写自动化测试（pytest）

前后端已经完全打通，你拥有了一个能对话的 AI ChatFlow 全栈应用。接下来我们要进入工程化的最后一步：自动化测试

为什么要写测试？
- 保证代码质量：每次修改后，运行一遍测试就知道有没有改坏旧功能。

- 简历亮点：“为项目编写了完整的自动化测试用例，覆盖核心业务逻辑”。

- 专业习惯：所有正规软件开发都必须有测试。

<!--more-->

## 📦 第一步：安装测试工具
确保你在 `backend` 目录且虚拟环境已激活，然后安装：
```powershell
.\venv\Scripts\Activate.ps1

pip install pytest httpx

# pytest：Python 测试框架
# httpx：用来模拟 HTTP 请求，测试 FastAPI 接口

```

## 🧱 第二步：创建测试目录和文件
在 `backend` 文件夹下新建目录 `tests`，并创建下面的文件。
最终测试文件结构：
```bash
backend/
├── tests/
│   ├── __init__.py          （空文件, 让 Python 把 tests 当包）
│   ├── conftest.py          （测试配置，提供测试客户端）
│   └── test_api.py          （具体的测试用例）
```

1. 创建 `backend/tests/__init__.py`（空文件，让 Python 识别包）
2. 创建 `backend/tests/conftest.py`（测试固定配置）
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
3. 创建 `backend/tests/test_api.py`（核心测试用例）
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
## 🚀 第三步：运行测试
在 `backend` 目录下，执行：
```powershell
pytest tests/ -v
```

如果一切正常，你会看到类似这样的输出：
```bash
tests/test_api.py::TestUserAPI::test_register PASSED
tests/test_api.py::TestUserAPI::test_register_duplicate PASSED
tests/test_api.py::TestUserAPI::test_login_success PASSED
tests/test_api.py::TestUserAPI::test_login_wrong_password PASSED
tests/test_api.py::TestConversationAPI::test_create_conversation PASSED
tests/test_api.py::TestConversationAPI::test_get_conversations PASSED
tests/test_api.py::TestConversationAPI::test_send_message_and_ai_reply PASSED
tests/test_api.py::TestConversationAPI::test_get_messages_after_send PASSED
tests/test_api.py::TestConversationAPI::test_delete_conversation PASSED

========================== 9 passed in 2.34s ==========================
```

## 📝 注意事项
- 测试会直接操作你的**开发数据库**（和你的 API 用的是同一个数据库），所以尽量不要在测试时有关键数据，或者测试后手动清理。如果想隔离，可以后续进阶使用测试数据库，但现在先跑通流程。

- 如果某个测试失败，仔细看错误信息，通常是数据库里残留了之前的数据导致冲突（比如重复注册用户）。如果真的失败，可以先进数据库清空 `user` 和 `conversation` 和 `message` 表再运行。

- 如何只跑某一个测试类或一个测试？
```bash
pytest tests/test_api.py::TestUserAPI -v          # 只跑用户测试类
pytest tests/test_api.py::TestUserAPI::test_register -v  # 只跑注册测试
```

## ✅ 验收标准
- pytest tests/ -v 所有 9 个测试用例全部 PASSED（绿色）

- 没有报错或失败

# 🧩 **如何写新的测试用例？（模板）**
假设你要测试一个新接口 POST /reset-password，你可以这样写：
```python
def test_reset_password(self, client):
    response = client.post(
        "/users/reset-password",
        json={"email": test_user["email"]},
    )
    assert response.status_code == 200
    assert response.json()["message"] == "密码重置邮件已发送"
```
核心步骤永远三步：
1. 发请求：client.post(...) 或 client.get(...)
2. 检查状态码：assert response.status_code == 期望的数字
3. 检查返回数据：assert response.json()["某字段"] == 期望值

## 逐行解释模板(AI生成)：
我们来把这个测试函数模板彻底拆解，把每一个语法点、每一个约定都掰开揉碎。看完后你会明白，所有的测试都是同一个模子刻出来的。

---

### 🧱 模板完全拆解

```python
def test_reset_password(self, client):
    response = client.post(
        "/users/reset-password",
        json={"email": test_user["email"]},
    )
    assert response.status_code == 200
    assert response.json()["message"] == "密码重置邮件已发送"
```

---

### 第一行：函数定义

```python
def test_reset_password(self, client):
```

| 部分                  | 是什么        | 官方约定/大白话                                                                                                  |
| --------------------- | ------------- | ---------------------------------------------------------------------------------------------------------------- |
| `def`                 | Python 关键字 | 定义函数的固定写法，全 Python 通用                                                                               |
| `test_reset_password` | 函数名        | **必须**以 `test_` 开头！这是 pytest 发现测试的**唯一规则**。名字随便起，但必须能看出测什么                      |
| `(self, client):`     | 参数列表      | `self` 是因为这个函数在类里面（类的实例方法）；`client` 是我们从 `conftest.py` 里定义的夹具，pytest 会自动传进来 |

**固定规则**：
- 测试函数名以 `test_` 开头
- 需要什么工具（夹具）就在参数里写同名变量

---

### 第二行：发送 HTTP 请求

```python
    response = client.post(
```

| 部分       | 是什么          | 大白话                                                                            |
| ---------- | --------------- | --------------------------------------------------------------------------------- |
| `response` | 变量名          | 用来接收服务器返回的结果，名字随意，叫 `res` 也行                                 |
| `client`   | TestClient 实例 | 从夹具传进来的"虚拟浏览器"，不用真启动服务器就能发请求                            |
| `.post(`   | HTTP POST 方法  | 向服务器提交数据。除了 `.post()`，还有 `.get()`、`.delete()`、`.put()` `.patch()` |

**`client` 对象上的方法**（全部和 HTTP 方法一一对应）：

| 方法                         | HTTP 动作 | 场景                           |
| ---------------------------- | --------- | ------------------------------ |
| `client.get(url)`            | GET       | 查数据（获取对话列表）         |
| `client.post(url, json={})`  | POST      | 提交数据（注册、登录、发消息） |
| `client.delete(url)`         | DELETE    | 删除数据                       |
| `client.put(url, json={})`   | PUT       | 完整更新数据                   |
| `client.patch(url, json={})` | PATCH     | 部分更新数据                   |

---

### 第三行：请求的路径

```python
        "/users/reset-password",
```

| 部分                      | 是什么           | 大白话                                                                                                       |
| ------------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------ |
| `"/users/reset-password"` | 字符串，URL 路径 | 这是你后端接口的地址。**不需要写完整域名**（如 `http://localhost:8000`），因为 `TestClient` 已经知道根地址了 |

**固定规则**：路径必须以 `/` 开头，和你在后端 `@router.post("/users/reset-password")` 里写的一致。

---

### 第四行：请求体

```python
        json={"email": test_user["email"]},
```

| 部分                            | 是什么      | 大白话                                                        |
| ------------------------------- | ----------- | ------------------------------------------------------------- |
| `json=`                         | 参数名      | **FastAPI TestClient 的固定参数名**，表示请求体的格式是 JSON  |
| `{"email": test_user["email"]}` | Python 字典 | 键值对，模拟前端传来的数据                                    |
| `test_user["email"]`            | 字典取值    | `test_user` 是我们定义的全局字典，取它的 `"email"` 键对应的值 |

**为什么是 `json=...` 而不是 `data=...`？**
- FastAPI 后端接口大部分接收 JSON 格式
- `json=` 会自动设置请求头 `Content-Type: application/json` 并把字典序列化为 JSON 字符串
- 如果是表单提交，才用 `data=`

**固定规则**：FastAPI 测试中，请求体传字典用 `json=` 参数。

---

### 第五行：关闭括号

```python
    )
```

Python 函数调用的右括号，闭合上面的 `client.post(...)`。

---

### 第六行：第一个断言 — 检查状态码

```python
    assert response.status_code == 200
```

| 部分                   | 是什么                | 大白话                                                                        |
| ---------------------- | --------------------- | ----------------------------------------------------------------------------- |
| `assert`               | Python 关键字         | "我断言后面的事情是真的"。如果后面是假的，直接报错，测试失败                  |
| `response.status_code` | TestClient 响应的属性 | HTTP 状态码（200=成功, 400=请求错误, 401=未登录, 404=不存在, 500=服务器炸了） |
| `== 200`               | 比较运算符            | 状态码必须等于 200                                                            |

**常见状态码速查**：

| 状态码 | 含义     | 何时期待它           |
| ------ | -------- | -------------------- |
| 200    | 成功     | 正常请求             |
| 400    | 请求错误 | 重复注册、格式不对   |
| 401    | 未登录   | 密码错误、没带 token |
| 404    | 不存在   | 查不存在的对话       |
| 429    | 请求过多 | 触发限流时           |

**固定规则**：**永远先断言状态码**。状态码对了，才去看返回内容；状态码不对，直接在这里失败，后面的断言不会执行。

---

### 第七行：第二个断言 — 检查返回内容

```python
    assert response.json()["message"] == "密码重置邮件已发送"
```

这一行看起来紧凑，我们拆成几步：

#### 第一步：`response`

服务器返回的响应对象。

#### 第二步：`.json()`

| 部分      | 是什么              | 大白话                                           |
| --------- | ------------------- | ------------------------------------------------ |
| `.json()` | Response 对象的方法 | 把服务器返回的 JSON 字符串**解析成 Python 字典** |

**注意**：这是加括号的 `.json()`，不是不加括号的 `.json`。不加括号拿不到数据。

#### 第三步：`["message"]`

| 部分          | 是什么            | 大白话                                            |
| ------------- | ----------------- | ------------------------------------------------- |
| `["message"]` | Python 字典键访问 | 从上一步得到的字典里，取出键名为 `"message"` 的值 |

等价于你先写：
```python
data = response.json()    # 得到整个字典，比如 {"message": "密码重置邮件已发送", "status": "ok"}
msg = data["message"]     # 取出 "message" 对应的值
```
然后断言：
```python
assert msg == "密码重置邮件已发送"
```

我们写成一行纯属简洁。

#### 第四步：`== "密码重置邮件已发送"`

期望返回的消息内容和这个字符串完全一致。

---

## 🧩 把模板抽象成“万能填空题”

以后要写任何新接口的测试，只填下面三个地方：

```python
def test_【功能描述】(self, client):
    response = client.【HTTP方法】(
        "【接口路径】",
        json=【请求体字典，没有就不写这行】,
    )
    assert response.status_code == 【期望的状态码】
    assert response.json()["【关键字段】"] == 【期望的值】
```

**实战例子一：测试获取用户信息接口 `GET /users/me`**
```python
def test_get_current_user(self, client):
    response = client.get(
        "/users/me",
        headers=self.auth_headers()   # 需要登录，带上 token
    )
    assert response.status_code == 200
    assert response.json()["email"] == test_user["email"]
```

**实战例子二：测试删除不存在的对话返回 404**
```python
def test_delete_nonexistent_conversation(self, client):
    response = client.delete(
        "/conversations/99999",
        headers=self.auth_headers()
    )
    assert response.status_code == 404
```

---

## 📋 所有涉及的 Python/HTTP 语法总结

| 语法                             | 分类                     | 固定程度                   |
| -------------------------------- | ------------------------ | -------------------------- |
| `def test_xxx(self, client):`    | pytest 约定              | ⭐⭐⭐ 必须这样写             |
| `client.post/get/delete(...)`    | FastAPI TestClient       | ⭐⭐⭐ 方法名和 HTTP 方法一致 |
| `json={...}`                     | TestClient 参数          | ⭐⭐⭐ 请求体用 `json=`       |
| `response.status_code`           | HTTP 协议                | ⭐⭐⭐ 一定有                 |
| `response.json()`                | TestClient / requests 库 | ⭐⭐⭐ 解析 JSON 响应         |
| `assert ... == ...`              | Python 关键字            | ⭐⭐⭐ 断言固定写法           |
| `test_user["email"]`             | Python 字典取值          | ⭐⭐ 通用语法                |
| `headers={"Authorization": ...}` | HTTP 协议                | ⭐⭐ 鉴权接口必须带          |

---

这就是测试的全部核心套路。你只要记住：**发请求 → 查状态码 → 查返回内容**，三步走，任何一个接口都能测。