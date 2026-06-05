---
title: useApi.js 文件简介
date: 2026-06-05 13:20:20
tags: [Python Project]
categories: [ToDo]
permalink: /todo-doc/useApi.js/
---


# `useApi.js` 文件简介


`hooks/useApi.js` 是一个 **自定义 React Hook**，它的核心职责是**封装异步请求的通用状态管理逻辑**（Loading 和 Error）。

在 React 开发中，每次发起网络请求通常都需要处理三个状态：
1.  **加载中** (`loading`)：显示转圈动画，禁用按钮。
2.  **出错** (`error`)：显示错误提示。
3.  **成功**：更新数据。

如果没有这个 Hook，你需要在每个组件里重复写 `useState` 和 `try...catch`。有了它，你可以用更简洁、统一的方式处理所有 API 调用。

---

<!--more-->

### 1. 代码深度解析


```javascript

import { useCallback, useState } from "react";

export function useApi(defaultErrorMessage = "请求失败") {
  // 1. 内部状态：管理 loading 和 error
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  // 2. 清除错误：用于用户点击“关闭”或重试时
  const clearError = useCallback(() => setError(null), []);

  // 3. 核心执行函数 run
  const run = useCallback(
    async (fn, { fallbackMessage } = {}) => {
      setLoading(true);   // 开始请求：开启 loading
      setError(null);     // 开始请求：清除旧错误
      
      try {
        return await fn(); // 执行传入的异步函数（如 apiRequest）
      } catch (err) {
        // 捕获错误：设置错误信息
        const msg = err.message || fallbackMessage || defaultErrorMessage;
        setError(msg);
        throw err; // 重新抛出错误，让调用者知道失败了
      } finally {
        setLoading(false); // 结束请求：关闭 loading
      }
    },
    [defaultErrorMessage]
  );

  // 4. 返回给组件使用的接口
  return { loading, error, run, clearError, setError };
}

```


#### 关键点说明：


*   **高阶抽象**：它不关心具体请求的是“任务”还是“用户”，它只关心“这是一个异步操作”。
*   **`run` 函数**：这是最精妙的设计。你只需要把具体的 API 调用函数传给它，它会自动帮你包裹上 `setLoading(true/false)` 和错误捕获逻辑。
*   **`throw err`**：在 `catch` 块中重新抛出错误非常重要。这样，如果组件需要知道请求失败了（比如为了弹窗提示），它可以通过 `await run(...)` 捕获到异常。

---


### 2. 如何使用它？（对比传统写法）


#### ❌ 传统写法（在组件里硬编码状态）


```javascript

const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);

async function handleDelete(id) {
  setLoading(true);
  setError(null);
  try {
    await apiRequest(`/tasks/${id}`, { method: 'DELETE' });
    // 更新列表...
  } catch (err) {
    setError(err.message);
  } finally {
    setLoading(false);
  }
}

```


#### ✅ 使用 `useApi` Hook（简洁、复用）


```javascript

import { useApi } from "../hooks/useApi";

function TaskItem({ task }) {
  // 一行代码搞定状态管理
  const { loading, error, run, clearError } = useApi("删除失败");

  async function handleDelete() {
    try {
      // 把具体的请求逻辑传给 run
      await run(() => apiRequest(`/tasks/${task.id}`, { method: 'DELETE' }));
      // 如果成功，执行后续逻辑（如通知父组件刷新）
      console.log("删除成功");
    } catch {
      // run 已经设置了 error 状态，这里可以做额外处理
    }
  }

  return (
    <div>
      {error && <div className="text-red-500">{error} <button onClick={clearError}>x</button></div>}
      <button onClick={handleDelete} disabled={loading}>
        {loading ? "删除中..." : "删除"}
      </button>
    </div>
  );
}

```

---


### 3. 为什么它在项目中很重要？


虽然在你提供的 `TaskList.jsx` 代码中，开发者选择了手动管理状态（为了演示乐观更新等复杂逻辑），但在以下场景中，`useApi` 是非常有用的：

1.  **简单表单提交**：比如登录、注册、修改密码。这些操作通常只需要简单的 Loading 和 Error 提示，不需要复杂的乐观更新。
2.  **统一用户体验**：确保所有使用它的地方，Loading 状态的切换时机和错误处理方式是一致的。
3.  **减少样板代码**：让组件代码更专注于业务逻辑（UI 怎么变），而不是基础设施逻辑（怎么发请求）。

---


### 4. 总结


`hooks/useApi.js` 是前端的 **异步状态工具箱**。
*   它将 **Loading** 和 **Error** 状态从业务组件中剥离出来。
*   它通过 `run` 函数提供了一种 **声明式** 的异步调用方式。
*   它是 React 组合式思想（Composition）的典型体现：**把逻辑抽离成 Hook，让组件变得更干净。**

对于初学者来说，理解 `useApi` 有助于你掌握如何编写可复用的 React 逻辑，这是从“写页面”进阶到“写应用”的重要一步。