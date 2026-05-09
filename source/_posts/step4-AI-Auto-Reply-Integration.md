---
title: step4-AI_Auto-Reply_Integration
date: 2026-05-09 11:23:28
tags: [Python Project]
categories: [ai-chatflow]
---

# 🚀 Step 4：AI 自动回复集成
- 对话和消息的增删改查已经全部跑通。现在我们要让它“活”起来——当用户发送一条消息后，系统能自动生成 AI 回复并存储。
- 如果你还没有 API Key，我会先用一个聪明的模拟方案（假回复），但把真实接口的位置留好。将来你拿到 Key，只需要改一行代码就能切换成真正的 AI。

<!--more-->

## 第一步：创建 AI 服务模块
新建文件 `backend/app/utils/ai_service.py`,内容：
```python
'''
作用：生成 AI 回复。目前用模拟回复，将来替换成真实 LLM 调用。

被谁引用：routers/conversations.py 中的 send_message 接口
'''


import time

def generate_ai_reply(user_message: str) -> str:
    """
    根据用户消息 生成(generate) AI 回复。
    当前是模拟版本，返回固定的占位回复。
    未来只需替换此函数体，即可接入真正的 LLM。
    """
    # ---- 模拟回复 ----
    time.sleep(0.5)  # 模拟 API 延迟，让体验更真实
    return f"🤖 你说：“{user_message}”。我是 AI ChatFlow,目前由模拟引擎驱动。等你接入 OpenAI 兼容的 API Key,我就能真正回答啦!"



'''
💡 将来如何切换成真实 AI?(提前说明)
当你拿到 API Key(比如 DeepSeek 的)后，只需要把 generate_ai_reply 改成类似这样(我会在之后的步骤帮你写，但现在你可以看一下思路):
import openai
def generate_ai_reply(user_message: str) -> str:
    client = openai.OpenAI(api_key="你的key", base_url="https://api.deepseek.com/v1")
    response = client.chat.completions.create(
        model="deepseek-chat",
        messages=[{"role": "user", "content": user_message}]
    )
    return response.choices[0].message.content
'''

```

## 第二步：修改发送消息接口，实现自动回复
1. 打开 `backend/app/routers/conversations.py`,在文件顶部导入新的 AI 服务：
    `from app.utils.ai_service import generate_ai_reply`
2. 修改 `send_message` 接口：
   ```python
    @router.post("/{conversation_id}/messages", response_model=Message)
    def send_message(conversation_id: int, message: Message, session: Session = Depends(get_session)):
    # 1. 检查对话是否存在
    conv = session.get(Conversation, conversation_id)
    if not conv:
        raise HTTPException(status_code=404, detail="Conversation not found")
    
    # 2. 保存用户的真实消息
    message.conversation_id = conversation_id
    session.add(message)
    session.commit()
    session.refresh(message)
    
    # 3. 生成 AI 回复并保存
    ai_reply_content = generate_ai_reply(message.content)
    ai_message = Message(
        conversation_id=conversation_id,
        role="assistant",
        content=ai_reply_content
    )
    session.add(ai_message)
    session.commit()
    session.refresh(ai_message)
    
    # 4. 只返回用户消息，还是都返回？我们返回 AI 消息也可以，但为了前端方便，
    #    这里直接返回 AI 消息，前端就知道对话有新内容了。
    return ai_message

   ```

## 第三步：测试自动回复
1. 虚拟环境(.venv)启动后端：`uvicorn app.main:app --reload --port 8000`
2. 确保服务器在运行，打开 `http://localhost:8000/docs`
3. 创建对话（如果还没有）：用 `POST /conversations/` 创建一个新对话,获取它的 id
4. 发送一条用户消息：用 `POST /conversations/{conversation_id}/messages`
   - `conversation_id` 填你创建的那个对话 ID。
    ```bash
    {
        "role": "user",
        "content": "你好，你是谁？"
    }
    ```
5. 点击 `Execute`，稍等片刻，应该返回 200，响应体是 AI 的回复，并且 `role` 为 "assistant",content 是模拟的那段话。
6. 查看对话消息：再调用 `GET /conversations/{conversation_id}/messages`,你会看到两条消息：
    - 第一条：`role: "user"`, content: "你好，你是谁？"
    - 第二条：`role: "assistant"`, content: 🤖 你说：“{user_message}”。我是 AI ChatFlow,目前由模拟引擎驱动。等你接入 OpenAI 兼容的 API Key,我就能真正回答啦!



# 总结：
恭喜你，你成功实现了自动回复功能。过程也很简单在`backend/app/utils`里增加一个工具`ai_service.py`,修改路由/接口
✅ Step 4 验收标准
- 调用 send_message 后，接口不再只返回自己刚发的消息，而是返回一条 AI 回复。

- GET 查询该对话的消息，能看到连续的 user 和 assistant 消息。

- 整个过程无报错，状态码 200。
