---
title: step7-runproject
date: 2026-05-10 13:30:19
tags: [Python Project]
categories: [ai-chatflow]
---

# 🔧 step7 ：启动项目
把整个项目从零跑起来，确保一切正常，然后提交到 Git。
我们按用户使用的两种场景分别说明：
1. 本地开发模式（你用 VS Code 和终端跑）
2. Docker 一键部署模式（验证封装效果）
你先按本地模式验证，然后再试 Docker。

<!--more-->

## 🔧 场景一：本地开发模式验证
### 第一步：启动 PostgreSQL 数据库
如果你还没有启动数据库，在项目根目录打开终端执行：
```powershell
docker compose up -d db
```

这只会启动数据库容器。验证数据库已运行：
```powershell
docker ps   # 应看到 chatflow_db 状态为 Up
```

### 第二步：启动后端 (FastAPI)
1. 打开新的终端，进入 backend 目录：
```powershell
cd ai-chatflow\backend
```

2. 激活虚拟环境并安装依赖（确保依赖完整）：
```powershell
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

3. 启动后端服务：
```powershell
uvicorn app.main:app --reload --port 8000
```

4. 验证后端：浏览器访问 `http://localhost:8000` 看到 `{"message":"Hello from AI ChatFlow API"}`，访问 `http://localhost:8000/docs` 看到 API 文档。

### 第三步：启动前端 (Vue)
1. 再开一个新终端，进入 frontend 目录：
```powershell
cd ai-chatflow\frontend
```

2. 安装依赖（如果之前没装）并启动：
```powershell
npm install
npm run dev
```

3. 验证前端：浏览器访问 `http://localhost:5173`, 会自动跳转到登录页面 `http://localhost:5173/login`。

### 第四步：完整功能测试
1. 注册新用户：在登录页点击“没有账号？去注册”，输入邮箱和密码，注册成功后会提示，并自动切回登录页。
2. 登录：输入刚注册的邮箱密码，点击登录，页面跳转到聊天界面。
3. 创建对话：点击左侧“+ 新建”，输入标题，创建后选中该对话。
4. 发送消息：在右侧输入框输入“你好”，回车发送，稍等片刻应看到 AI 回复（真实 AI 或模拟降级）。
5. 删除对话：在左侧对话列表点击删除按钮，确认删除，对话消失。

### 第五步：运行自动化测试
在启用后端的情况下（测试直接调用 `TestClient`，不需要启动服务器，但数据库必须能连接），进入 `backend` 目录并激活虚拟环境，执行：
```powershell
.\venv\Scripts\Activate.ps1
pytest tests/ -v
```

应该看到 9 个测试全部通过，绿色 `PASSED`。
**如果某个测试失败**，检查：
• 数据库是否正在运行
• 之前是否注册过 `pytest@example.com`，如果注册过，先手动清空 `user`、`conversation`、`message` 表再跑测试。

#### 🧹 如何手动清空数据库
只需要把 `user`、`conversation`、`message` 这三张表里的数据全部清空，但保留表结构。用 Docker 执行一条 SQL 命令就行，不需要安装任何额外工具。
**前提**
• 你的数据库容器正在运行。在项目根目录打开终端（PowerShell），执行 `docker ps` 应该能看到 `chatflow_db` 容器，状态是 `Up`。

**操作步骤**（直接复制运行）
1. **在项目根目录的终端中**，复制下面一整条命令（一整行），回车执行：
```powershell
docker exec -it chatflow_db psql -U chatflow_user -d chatflow -c "TRUNCATE TABLE message, conversation, \`"user\`" CASCADE;"
```

2. 你会看到输出：
```text
TRUNCATE TABLE
```

这就表示三张表的记录已经全部清空，干干净净。

**这条命令在干什么**？（**拆开看**）

1. `docker exec -it chatflow_db`
   - 进入正在运行的数据库容器 `chatflow_db` 里面，执行命令
2. `psql -U chatflow_user -d chatflow`
   - 用 PostgreSQL 自带的命令行客户端，以用户 `chatflow_user` 身份连接到数据库 `chatflow`
3. `-c "TRUNCATE TABLE ..."`
   - 执行后面双引号里的 SQL 语句
4. `TRUNCATE TABLE message, conversation, "user" CASCADE`
   - 清空 `message`、`conversation`、`user` 三张表的数据，并且因为加了 `CASCADE`，会自动处理表之间的外键关系，不会报错
5. `"user"` 加双引号
   - 因为 `user` 是 PostgreSQL 的保留关键字，必须用双引号括起来，否则会语法错误
**别担心**：`TRUNCATE` 只删数据，不会删表结构，也不会删你的数据库。跑完后表还在，只是变成了空表。

**什么时候需要清空？**
- 重新运行 pytest 测试前（避免上次测试留下的数据干扰）。
- 想清空所有测试用户、对话、消息，从头开始手动测试时。
- 放心，你可以随时执行这条命令，不会搞坏任何东西。

**备用方法（如果你有 pgAdmin）**
如果你装了 pgAdmin 图形界面，也可以：
1. 连接到 `localhost:5432` 数据库。
2. 找到 `chatflow` 数据库 → Schemas → public → Tables。
3. 右键点击 `message` 表 → Truncate → 选择 `CASCADE` → 确定。
4. 对 `conversation` 和 `user` 表同样操作。
不过命令行更快，建议用上面那条 Docker 命令就够了


## 🐳 场景二：Docker 一键部署验证
如果你想验证 Docker 封装效果：
1. 确保系统环境变量（或根目录 `.env`）里已配置好 `DEEPSEEK_API_KEY`。
2. 在项目根目录执行：
```powershell
docker compose up -d --build
```

3. 等待构建完成，访问：
   - 前端：`http://localhost`（因为端口映射 80）
   - 后端文档：`http://localhost:8000/docs`
4. 功能测试同上。
```bash
# 清空测试数据（如果需要）
docker exec -it chatflow_db psql -U chatflow_user -d chatflow -c "TRUNCATE TABLE message, conversation, \`"user\`" CASCADE;"
# 跑测试
cd backend
pytest tests/ -v
```
5. 停止并清理：
```powershell
docker compose down
```

# ✅ 收尾确认清单
- `uvicorn` 后端已停止（`Ctrl+C` 了）
- `npm run dev` 前端已停止（`Ctrl+C` 了）
- Docker 数据库容器已关闭（`docker compose down` 执行完毕）
- Git 仓库提交（`git add .`、`git commit -m "commit message(自定义 注释作用)"`、`git push`）


# Docker 一键部署后有个问题？刷新浏览器后，页面会报错：`404 (Not Found)`
这个问题是单页应用（SPA）在 Docker/Nginx 部署时最经典的坑。
当你刷新页面（比如 /login），浏览器向 Nginx 发起请求，Nginx 去服务器上找 /login 这个文件，找不到就返回了 404。
解决办法：让 Nginx 在找不到文件时，始终返回 index.html，由 Vue 的路由接管。
## 🛠️ 修复步骤（两步搞定）
1. 在 `frontend/` 目录下新建 `nginx.conf` 文件
复制下面的内容，保存为 `frontend/nginx.conf`（和 `Dockerfile` 同级，放在 `frontend/` 文件夹里）：
```nginx
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

2. 修改 `frontend/Dockerfile`
在 nginx 阶段增加一行复制配置文件的命令。
打开 `frontend/Dockerfile`，把 nginx 部分改成这样：
```dockerfile
FROM nginx:alpine

# 把我们的 nginx 配置复制进容器，覆盖默认配置
COPY nginx.conf /etc/nginx/conf.d/default.conf

# 从构建阶段复制编译好的静态文件
COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

```

## 🚀 重新构建并启动
在项目根目录执行：
```powershell
docker compose down          # 停止并删除旧的容器
docker compose up -d --build # 重新构建并后台启动
```

这次再打开 `http://localhost`，登录后刷新页面，就不会再 404 了。

## 💡 为什么这样就可以了？
• `try_files $uri $uri/ /index.html`; 的意思是：
Nginx 先找 `$uri`（用户请求的路径），找不到就找 `$uri/`（目录），再找不到就返回 `index.html`。
返回 `index.html` 后，Vue 路由通过 URL 恢复对应页面。
• 这行配置是**所有 SPA 部署到 Nginx 的标准写法。**
