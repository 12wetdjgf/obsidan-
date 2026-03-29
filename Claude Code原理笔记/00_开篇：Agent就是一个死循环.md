# Claude Code 原理笔记 · 第00篇：Agent就是一个死循环

> 💡 **核心洞察**：你以为AI Agent很复杂？其实就是一个 `while True` 加上一个 `if` 语句。真的。

---

## 🎯 问题：为什么需要Agent？

Claude（或任何LLM）很聪明，但有个致命缺陷：**它只能聊天，不能做事**。

```
❌ 你：Claude，帮我读一下 hello.py 的内容
✅ Claude：我可以帮你分析代码，但我看不到你的文件...
```

Claude 看不到你的文件系统、跑不了命令、改不了代码。它就像一个被困在玻璃房间里的天才——再聪明也没用。

**Agent 的工作就是：给Claude一把钥匙，让它能打开玻璃房间的门。**

---

## 🔑 解决方案：最小化Agent循环

```
┌─────────────┐
│   用户提示   │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────────┐
│  LLM 思考：我需要什么工具？          │
└──────┬──────────────────────────────┘
       │
       ▼
    ┌─────────────────────────┐
    │ 调用工具（比如 bash）    │
    │ 执行真实操作            │
    └──────┬──────────────────┘
           │
           ▼
    ┌─────────────────────────┐
    │ 把结果返回给 LLM        │
    └──────┬──────────────────┘
           │
           ▼
    ┌─────────────────────────┐
    │ LLM 继续思考            │
    │ 需要更多工具吗？        │
    └──────┬──────────────────┘
           │
      ┌────┴────┐
      │          │
     是          否
      │          │
      ▼          ▼
   循环      返回答案
```

**核心逻辑**：
- LLM 说"我要用工具X"
- Agent 执行工具X
- 把结果给LLM
- LLM 说"我还要用工具Y"或"我完成了"
- 如果完成了，就停止；否则继续循环

---

## 💻 代码：30行的Agent

```python
def agent_loop(query):
    messages = [{"role": "user", "content": query}]
    
    while True:
        # 第1步：问LLM想用什么工具
        response = client.messages.create(
            model="claude-3-5-sonnet",
            system="你是一个编程助手",
            messages=messages,
            tools=TOOLS,  # 告诉LLM有哪些工具可用
            max_tokens=8000,
        )
        
        # 第2步：把LLM的回复加到对话历史
        messages.append({"role": "assistant", "content": response.content})
        
        # 第3步：检查LLM是否还要用工具
        if response.stop_reason != "tool_use":
            # 不用工具了，说明完成了
            return
        
        # 第4步：执行LLM要求的工具
        results = []
        for block in response.content:
            if block.type == "tool_use":
                # 根据工具名字找到对应的处理函数
                output = TOOL_HANDLERS[block.name](**block.input)
                results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": output,
                })
        
        # 第5步：把工具结果加到对话历史，继续循环
        messages.append({"role": "user", "content": results})
```

**就这么简单！** 没有什么黑魔法，就是：
1. 问LLM
2. 执行它要求的工具
3. 把结果告诉它
4. 重复直到它说"完成了"

---

## 🎮 关键概念

| 概念 | 解释 | 类比 |
|------|------|------|
| **messages** | 对话历史 | 你和Claude的聊天记录 |
| **tools** | 可用工具列表 | 菜单上有什么菜 |
| **stop_reason** | 为什么停止 | Claude说"我完成了"还是"我要用工具" |
| **tool_use** | 工具调用 | Claude说"我要用bash运行这个命令" |
| **tool_result** | 工具结果 | bash运行的输出 |

---

## 🧠 思维模型

想象你和Claude在做一个项目：

```
你：帮我创建一个Python文件，内容是打印"Hello"

Claude：好的，我需要用 write_file 工具
        我要创建 hello.py，内容是 print("Hello")

你（Agent）：好的，我帮你执行了，文件已创建

Claude：太好了！现在我要用 bash 工具验证一下
        运行 python hello.py

你（Agent）：好的，输出是：
            Hello

Claude：完美！任务完成了
```

**你的角色**就是那个"你（Agent）"——执行Claude的指令，把结果反馈给它。

---

## 📊 对比：有Agent vs 没Agent

| 场景 | 没有Agent | 有Agent |
|------|----------|--------|
| 创建文件 | 你手动创建 | Claude自动创建 |
| 运行代码 | 你手动运行 | Claude自动运行 |
| 读取文件 | 你复制粘贴 | Claude自动读取 |
| 调试错误 | 你告诉Claude错误 | Claude自动看到错误 |
| 效率 | 🐢 慢 | 🚀 快 |

---

## 🎓 你来试试！

现在你理解了Agent的核心。下一步我们会学：

1. **s01**：最小化Agent（只有bash工具）
2. **s02**：添加更多工具（read、write、edit）
3. **s03**：让Agent有计划（TodoList）
4. **s04**：分解大任务（Subagent）
5. ... 以及更多高级特性

---

## 💬 口诀

> **"一个循环，一个条件，一个工具，一个Agent。"**
>
> `while True: 问LLM → 执行工具 → 反馈结果 → 直到完成`

---

## 🚀 下一步

准备好了吗？让我们进入 **s01：最小化Agent循环** 吧！

你会看到：
- 如何用真实代码运行一个Agent
- 如何让Claude读文件、写文件、运行命令
- 如何一步步调试

**准备好敲代码了吗？** 😎
