---
title: step5-Vue3_FrontendChatInterface
date: 2026-05-09 16:06:06
tags: [Python Project]
categories: [ai-chatflow]
---

# 🎯 Step 5：Vue 3 前端聊天界面
在核心后端已经具备“智能回复”能力，接下来将把网页聊天界面搭建起来，让你可以在浏览器里直接进行对话。
1. 创建一个 `Vue 3 + Vite` 的前端项目（脚手架）
2. 安装必要的库：HTTP 请求工具（axios）、页面路由（vue-router）
3. 构建两个核心页面：
   - `ConversationList`：左侧对话列表
   - `ChatWindow`：右侧聊天区域
4. 与后端 API 联调：发送消息、获取历史、自动显示 AI 回复
5. 验证效果：在浏览器 `http://localhost:5173` 看到完整的聊天应用

<!--more-->

## 📦 第一步：创建前端的工程骨架
**提示：当然如果提前创建了 `frontend` 文件夹可以直接删除这个空文件夹再执行命令**
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
`vue-router` 用于页面间的导航，`axios` 用于调用后端 API `vue-router@4` -> 指定使用的 Vue Router 版本为 4.x 版本

## ✏️ 第二步：删除脚手架默认文件，准备写我们自己的代码
Vite 会生成一些示例文件，我们全部清空，从头写。

1. 在 VS Code 中打开 frontend/src 文件夹。

2. **删除** `src/components/HelloWorld.vue`

3. **删除** `src/assets/vue.svg`

4. **清空** `src/App.vue` 的内容（保留空文件）

5. **清空** `src/style.css` 的内容

## 🧱 第三步：创建核心聊天组件
我们需要建立这样的文件结构：
```bash
frontend/src/
├── views/
│   └── ChatView.vue        ← 主聊天页面（左列表 + 右窗口）
├── components/
│   ├── ConversationList.vue
│   └── ChatWindow.vue
├── api/
│   └── index.js            ← 封装 axios 请求
├── router/
│   └── index.js            ← 路由配置
├── App.vue                 ← 根组件
└── main.js                 ← 入口文件
```

### 1. 配置 API 请求封装
新建 `frontend/src/api/index.js`:
```javascript
// 文件位置：frontend/src/api/index.js
// 作用：统一封装后端请求，方便全局管理

// 引入axios(发送请求的工具)
import axios from "axios";

// 创建axios实例(以后所有请求都会自动拼上前缀：http://localhost:8000)
const apiClient = axios.create({
  baseURL: "http://localhost:8000", // 后端地址
  timeout: 10000, // 请求超时时间10秒
  headers: { "Content-Type": "application/json" }, // 请求头 数据格式为json
});

//下面就都是接口方法export default {}可以将这些接口暴露出去，vue组件中import { } from "@/api"就可以使用这些接口了
export default {
  // 创建一个新对话
  createConversation(conversation) {
    return apiClient.post("/conversations/", conversation);
  },
  // 获取某个用户的所有对话列表
  // 访问地址：/conversations/?user_id=xxx  params 是 axios 固定写法，用来生成？后面的参数
  getConversations(userId) {
    return apiClient.get("/conversations/", { params: { user_id: userId } });
  },
  // 删除一个对话  ${id} = 把变量插进网址
  deleteConversation(id) {
    return apiClient.delete(`/conversations/${id}`);
  },
  // 获取某个对话里的所有消息
  getMessages(conversationId) {
    return apiClient.get(`/conversations/${conversationId}/messages`);
  },
  // 发送消息 → 就是调用你最开始写的那个后端接口
  sendMessage(conversationId, message) {
    return apiClient.post(`/conversations/${conversationId}/messages`, message);
  },
};

```

### 2. 配置路由
新建 `frontend/src/router/index.js`:
```javascript
// 用户访问网址 / → 显示 ChatView.vue 这个聊天页面
/*
- import：导入 
- createRouter：创建路由（导航器）
- createWebHistory：使用正常网址模式（不带 #）
- 
*/
import { createRouter, createWebHistory } from "vue-router";
import ChatView from "../views/ChatView.vue";

// 路由规则   path：网址  name：路由名称  component：组件
// 用户访问 http://localhost:5173/ → 显示聊天页面ChatView.vue
const routes = [{ path: "/", name: "Chat", component: ChatView }];
/*
创建路由实例
history: createWebHistory()
用正常网址，比如 http://xxx/
不是老式的 http://xxx/#/

routes
把上面写的规则放进去

*/
const router = createRouter({
  history: createWebHistory(),
  routes,
});

export default router;

```

### 3. 修改程序入口
打开 `frontend/src/main.js`, 完全替换为以下内容：
```javascript
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

### 4. 修改根组件 App.vue
**清空原有内容**, 粘贴：
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
### 5.整个项目结构就是 main.js 项目启动入口 → router/index.js 配置路由(导航地图) → App.vue 总外壳 + 屏幕 → ChatView.vue 真正的聊天页面 → api/index.js 发请求给后端

## 💬 第四步：编写聊天界面组件
### 1. 对话列表组件 `ConversationList.vue`
新建 `frontend/src/components/ConversationList.vue`:
```vue
<template>
  <div class="conversation-list">
    <div class="header">
      <h3>对话列表</h3>
      <button @click="$emit('new')">+ 新建</button>
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
  </div>
</template>

<script setup>
defineProps({
  conversations: Array,
  activeId: Number,
});

defineEmits(["new", "select", "delete"]);
</script>

<style scoped>
.conversation-list {
  width: 260px;
  background: #f5f5f5;
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.header {
  padding: 1rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #ddd;
}

ul {
  list-style: none;
  padding: 0;
  overflow-y: auto;
}

li {
  padding: 0.8rem 1rem;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

li.active {
  background: #e0e0e0;
  font-weight: bold;
}

.del-btn {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 0.9rem;
}
</style>

```

### 2. 聊天窗口组件 `ChatWindow.vue`
新建 `frontend/src/components/ChatWindow.vue`:
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
}

.messages {
    flex: 1;
    overflow-y: auto;
    padding: 1rem;
}

.message {
    margin-bottom: 0.8rem;
}

.message.user .content {
    background: #dcf8c6;
    align-self: flex-end;
    padding: 0.6rem 1rem;
    border-radius: 12px 12px 0 12px;
    display: inline-block;
    max-width: 70%;
}

.message.assistant .content {
    background: #ffffff;
    border: 1px solid #ddd;
    padding: 0.6rem 1rem;
    border-radius: 12px 12px 12px 0;
    display: inline-block;
    max-width: 70%;
}

.input-area {
    display: flex;
    padding: 0.8rem;
    border-top: 1px solid #ddd;
}

.input-area input {
    flex: 1;
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 6px;
    margin-right: 0.5rem;
}

.input-area button {
    padding: 0.5rem 1rem;
    background: #42b983;
    color: white;
    border: none;
    border-radius: 6px;
    cursor: pointer;
}
</style>

```

### 3. 主聊天视图 `ChatView.vue`
新建 `frontend/src/views/ChatView.vue`:
```vue
<template>
    <div class="chat-layout">
        <ConversationList :conversations="conversations" :activeId="activeConvId" @new="createConversation"
            @select="selectConversation" @delete="handleDelete" />
        <ChatWindow v-if="activeConvId" :messages="currentMessages" :conversationId="activeConvId"
            @send="sendMessage" />
        <div v-else class="no-chat">
            请选择或创建一个对话
        </div>
    </div>
</template>

<!--这是 Vue3 的语法，表示这里写逻辑代码 
1. 从 Vue 官方库里导入 3 个工具：
- ref：创建响应式变量 **数据一变，页面自动更新**
- computed：创建计算属性 **依赖变，我就自动重新算**
- onMounted：挂载完成生命周期 **组件刚显示出来，立刻做某事**
-->
<script setup>  
import { ref, computed, onMounted } from 'vue'
import ConversationList from '../components/ConversationList.vue'
import ChatWindow from '../components/ChatWindow.vue'
import api from '../api/index.js'

const userId = 1  // 因为还未做登录，先硬编码为注册的用户 ID

const conversations = ref([])
const activeConvId = ref(null)
const messagesMap = ref({})  // { convId: [msg, ...] }

const currentMessages = computed(() => {
    return messagesMap.value[activeConvId.value] || []
})

// 初始化：加载用户对话列表
onMounted(async () => {
    try {
        const res = await api.getConversations(userId)
        conversations.value = res.data
    } catch (e) {
        console.error(e)
    }
})

// 创建新对话
async function createConversation() {
    const title = prompt('请输入对话标题：')
    if (!title) return
    try {
        const res = await api.createConversation({ title, user_id: userId })
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
async function handleDelete(id) {
    if (!confirm('确定删除这个对话吗？')) return
    try {
        await api.deleteConversation(id)
        conversations.value = conversations.value.filter(c => c.id !== id)
        delete messagesMap.value[id]
        if (activeConvId.value === id) activeConvId.value = null
    } catch (e) {
        console.error(e)
    }
}
</script>

<style scoped>
.chat-layout {
    display: flex;
    height: 100vh;
    width: 100%;
}

.no-chat {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #999;
    font-size: 1.2rem;
}
</style>


```
<details>

<summary> 关于ChatView.vue的template部分逐行解释,AI生成：</summary>

这是一个Vue 单文件组件（.vue），用来做聊天页面：左边是会话列表，右边是聊天窗口，没选会话就显示提示文字。

    <ConversationList 
        :conversations="conversations" 
        :activeId="activeConvId" 
        @new="createConversation"
        @select="selectConversation" 
        @delete="handleDelete" 
    />

这一行是核心业务组件，我拆成 6 小段讲：
1. < ConversationList
- 大白话：调用一个封装好的左侧会话列表组件，就像用一个现成的 UI 零件。
- 原理：Vue 是组件化开发，把页面拆成小零件（会话列表、聊天窗口），复用 + 好维护。
- 官方格式：组件名用大驼峰（PascalCase），这是 Vue 官方推荐写法。
1. :conversations="conversations"
- 大白话：把父组件的对话列表数据，传给子组件。
- 左边列表要显示哪些对话，全靠这个数据。
- 原理：: 是 v-bind 缩写 → 父传子 数据绑定。
- 官方格式：属性名用小驼峰，值用双引号包变量。
1. :activeId="activeConvId"
- 大白话：告诉子组件当前哪个对话是选中状态，高亮显示。
- 原理：还是父传子，把选中的对话 ID 传给列表组件。
1. @new="createConversation"
- 大白话：子组件触发 **“新建对话”** 时，父组件执行 createConversation 方法。
- 原理：@ 是 v-on 缩写 → 子传父 事件通信。
- 官方格式：事件名用短横线 / 小驼峰都行，方法名小驼峰。
1. @select="selectConversation"
- 大白话：点击列表里的对话，触发选中，父组件切换聊天内容。
1. @delete="handleDelete"
- 大白话：删除对话时，子组件通知父组件执行删除逻辑。
1. />
- 大白话：自闭和标签，组件没有子内容就这么写，简洁。


</details>

## 🚀 第五步：启动前端并验收
1. 确保后端在运行`http://localhost:8000`

2. 在 `frontend` 目录下启动前端开发服务器：
`npm run dev`
3. 浏览器打开 `http://localhost:5173`, 应该看到一个空的聊天页面


## 🎉 验收：
✅ 验收清单
- 左侧出现对话列表，点击“+ 新建”可以创建一个对话

- 点击对话可以进入聊天界面，输入文字并回车发送

- 发送后能够收到一条 AI 回复（模拟内容）

- 删除对话功能正常

- 刷新页面后对话列表依然存在（数据在后端持久化）


# 最后的最后我深刻反思Vue那部分代码太烦人了，一点儿都不想看，注释又不好直接注释，template里要用`<!--内容-->`注释,script里要用`/*内容*/`注释,真的烦人。说这么多是想告诉你们，自己研究vue部分的代码吧，我这里就不多写，不解释了。

