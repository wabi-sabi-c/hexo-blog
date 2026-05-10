---
title: step8-beautifyUI
date: 2026-05-10 15:54:38
tags: [Python Project]
categories: [ai-chatflow]
---

# 🐳 step8  美化界面
我觉的DeepSeek 的界面确实好看——左侧深色侧边栏、右侧极简白、输入区居中悬浮。

所以我们直接照这个风格来改,而且我知道美化要单独一个个改太麻烦了，直接全选 -> 复制 -> 粘贴。

<!--more-->

## 🎨 仿 DeepSeek 风格聊天界面美化方案
| 文件                 | 改动                 | 效果                         |
| -------------------- | -------------------- | ---------------------------- |
| ConversationList.vue | 深色侧边栏           | 和 DeepSeek 一样的左侧深色栏 |
| ChatWindow.vue       | 气泡风格、输入区居中 | 右侧极简白，输入框居中悬浮   |
| ChatView.vue         | 整体背景             | 纯白背景，左右分明           |

1. `ConversationList.vue` — 深色侧边栏
修改 `ConversationList.vue`
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

2. `ChatWindow.vue` — 气泡风格、输入区居中悬浮
修改 `ChatWindow.vue`
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
3. `ChatView.vue` — 整体背景、左右分明
修改`ChatView.vue`
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
4. `LoginView.vue` - 登录页面居中显示
修改`LOginView.vue`
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

# 结果展示
<!--严格限制单张图宽度，电脑一行排多张，手机自动换行不乱跑
gap:16px 控制图片间距
width 统一尺寸，强制限制大小-->

<div style="display:flex;justify-content:center;flex-wrap:wrap;gap:16px;">
<img src="/images/ai_chatflow_Login.png" width="260">
<img src="/images/ai_chatflow_chat.png" width="260">
<img src="/images/ai_chatflow_chat1.png" width="260">
</div>