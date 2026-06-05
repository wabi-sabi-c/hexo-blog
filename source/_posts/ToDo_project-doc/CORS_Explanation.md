---
title: 跨域（CORS）说明
date: 2026-06-05 13:28:20
tags: [Python Project]
categories: [ToDo]
permalink: /todo-doc/CORS/
---


# 跨域（CORS）说明


## 📂 技术概念深度解析：开发环境跨域（CORS）解决方案


### 1. 核心定位 (The "One-Liner")


> **跨域（CORS）配置是项目中用于**解决浏览器“同源策略”限制**的机制**，它允许前端（`localhost:5173`）安全地请求后端（`localhost:8000`）的 API。  
> 在整个开发环境中，这套方案扮演着 **“海关盖章 + 快递中转”** 的双重角色：
> - **海关盖章（后端 CORS）**：后端主动声明“我信任来自 `5173` 的请求”，给请求放行。
> - **快递中转（Vite 代理）**：前端开发服务器“偷偷”把请求转发给后端，绕过了浏览器的跨域检查。

---

<!--more-->

### 2. 技术深度解析


#### A. 为什么会出现跨域问题？

浏览器的 **同源策略**（Same-Origin Policy）规定：**协议、域名、端口**三者必须完全相同，才能直接发送请求并读取响应。

- 前端地址：`http://localhost:5173`
- 后端地址：`http://localhost:8000`
- **端口不同**（5173 vs 8000） → 浏览器的跨域限制生效。

没有跨域配置时，前端 `fetch('/api/tasks')` 会报错：
```

Access to fetch at 'http://localhost:8000/api/tasks' from origin 'http://localhost:5173' 
has been blocked by CORS policy

```


#### B. 解决方案一：后端 CORS（海关盖章）


在 `main.py`（假设使用 FastAPI）中通常这样配置：

```python

from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],   # 只允许前端开发地址
    allow_methods=["*"],                        # 允许所有 HTTP 方法
    allow_headers=["*"],                        # 允许所有请求头
)

```

**解释**：
- **`allow_origins`**：明确告诉浏览器“我信任这些来源的请求”。生产环境应改为实际域名，不能写 `"*"`（通配符）与凭证一起用。
- **`allow_methods`**：允许 `GET`、`POST`、`PUT`、`DELETE` 等。
- **`allow_headers`**：允许前端携带自定义头（如 `Authorization`）。

这种方式是**最正统的 CORS 解决方案**，但要求后端开发者主动配置。


#### C. 解决方案二：Vite 代理（快递中转）


在 `vite.config.js` 中配置代理后，前端请求不再直接发往后端，而是：

1. 前端代码写 `fetch('/api/tasks')`（相对路径）。
2. 请求被 Vite 开发服务器拦截。
3. Vite 将请求**转发**到 `http://localhost:8000/api/tasks`。
4. 浏览器实际访问的是 `http://localhost:5173/api/tasks`，但 Vite 从后端取回数据后返回给浏览器。

**关键点**：浏览器认为它始终在和 **同一个源**（`localhost:5173`）通信，因此跨域检查**完全不会触发**。

```javascript

// vite.config.js 核心片段
proxy: {
  "/api": {
    target: "http://localhost:8000",
    changeOrigin: true,
    rewrite: (path) => path.replace(/^\/api/, ""),
  }
}

```

- **`changeOrigin: true`**：修改请求头中的 `Host` 为 `localhost:8000`，防止后端做 Host 校验。
- **`rewrite`**：去除 `/api` 前缀，让后端路由保持干净（如 `/tasks`）。


#### D. 两种方式的对比与互补


| 维度 | 后端 CORS | Vite 代理 |
|------|-----------|-----------|
| 原理 | 后端响应头告诉浏览器“允许跨域” | 前端服务器中转，浏览器无感知 |
| 配置位置 | `main.py`（后端代码） | `vite.config.js`（前端配置） |
| 是否影响生产 | 是（生产环境也需要 CORS 或反向代理） | 否（代理仅用于开发服务器） |
| 适用场景 | 任何环境（开发、测试、生产） | 仅本地开发（`npm run dev`） |
| 复杂性 | 需理解 CORS 规范、预检请求等 | 简单，但需维护代理规则 |

**项目中的做法**：**两者并存**，但目的不同：
- 后端 CORS 是**生产环境必备**（当后端和前端部署在不同域名/端口时）。
- Vite 代理是**开发体验优化**：避免后端每次改动都重新配置 CORS，也避免前端代码写死 `localhost:8000`。

---


### 3. 为什么需要它们？ (Why & Comparison)


#### ❌ 没有 CORS 配置的混乱做法


- 开发时，前端直接 `fetch('http://localhost:8000/tasks')`，遇到跨域错误，无法调试。
- 临时方案：禁用浏览器安全策略（如 Chrome `--disable-web-security`），极度不安全且仅能一人使用。
- 或者让后端接受 `allow_origins=["*"]`，生产环境也开放所有来源 → **严重安全漏洞**。


#### ✅ 有了上述配置的优雅做法


- **开发阶段**：
  - 后端配置一次 CORS（仅允许 `5173`），稳定可靠。
  - 前端使用代理，代码中只写相对路径 `/api/tasks`，切换环境（生产/开发）无需修改代码。
- **生产阶段**：
  - 移除或限制 CORS 允许的源（例如只允许前端实际域名 `https://myapp.com`）。
  - 或者采用 Nginx 反向代理（类似于 Vite 代理的生产版本），前端代码依然使用相对路径。

**结果**：开发者可以顺畅联调，无需纠结跨域问题；代码干净、可移植。

---


### 4. 在项目中的工作流 (Workflow Context)


下图展示了两种配置如何协同工作：

```text

浏览器  http://localhost:5173
   │
   ├─ 请求静态页面（HTML/JS/CSS）
   │       │
   │       ▼
   │   Vite Dev Server (端口 5173)
   │       │
   │       └─ 返回前端代码
   │
   ├─ 前端 JS 执行 fetch('/api/tasks')
   │       │
   │       │  由于请求路径以 /api 开头，Vite 代理拦截
   │       ▼
   │   [Vite 代理] 转发到 http://localhost:8000/tasks
   │       │
   │       │  此时请求来自 Vite 服务器（不是浏览器），不存在跨域
   │       ▼
   │   后端服务器 (端口 8000)
   │       │
   │       ├─ 返回数据（例如 JSON）
   │       │
   │       ▼
   │   Vite 将数据原样返回给浏览器
   │
   └─ 浏览器成功拿到数据，没有任何跨域错误

另外，如果后端需要被其他前端（例如手机 App、Postman）直接调用，
后端 CORS 配置会允许来自 localhost:5173 的预检请求和实际请求。

```

**与项目文件的关系**：
- `vite.config.js`：包含代理规则。
- `main.py`（后端入口）：包含 CORS 中间件配置。
- 前端 API 调用代码（如 `api/client.js`）：只需写 `/api/...` 路径，不关心实际后端地址。

---


### 5. 总结


- **这是一套面向开发环境的“跨域双保险”方案**：后端 CORS 是标准解决路径，Vite 代理是辅助工具，两者共同确保开发时 API 调用顺畅。
- **它解决了本地前后端分离开发时最头疼的跨域问题**：无需安装浏览器插件、无需关闭安全策略，代码整洁统一。
- **它利用了浏览器同源策略、HTTP 代理、CORS 规范等 Web 基础技术**：理解它们对掌握现代前端工程化至关重要。
- **对于初学者，理解“代理只在开发环境生效”和“CORS 是后端责任”是核心**：
  - 生产环境部署时，必须配置生产环境下的反向代理（如 Nginx）或调整 CORS 允许的来源。
  - 永远不要在生产环境设置 `allow_origins=["*"]`（除非 API 完全公开且无用户认证）。

---


### 🎓 最后的小贴士


- 如果你使用 **Axios** 或 **fetch**，在开发环境下可以自动判断：  
  `baseURL: import.meta.env.DEV ? '/api' : 'https://your-production-api.com'`
- 如果后端使用 **Django**，CORS 配置可通过 `django-cors-headers` 库实现，原理相同。
- **Chrome 开发者工具** → Network 标签页 → 查看请求的 `Remote Address`，可以确认请求是否真的被代理（如果是 `127.0.0.1:5173` 且路径为 `/api/...`，说明代理生效）。
- 当代理和后端 CORS **同时存在**时，也不会冲突——代理请求先被 Vite 拦截，不经过浏览器 CORS 检查；但如果绕过代理直接请求后端（例如用 Postman），仍会受后端 CORS 规则限制。
