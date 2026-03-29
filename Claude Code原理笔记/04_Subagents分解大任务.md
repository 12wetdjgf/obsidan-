# Claude Code 原理笔记 · 第04篇：Subagents - 分解大任务

> 💡 **核心洞察**：大任务会污染上下文。给每个子任务一个干净的Agent，只返回摘要。

---

## 🎯 问题：上下文污染

想象你给Claude一个复杂任务：

```
任务：分析这个项目
1. 读取所有Python文件
2. 总结每个文件的功能
3. 找出设计模式
4. 提出改进建议
```

会发生什么？

```
第1步：读取 file1.py (500行)
  → messages 增加 500行

第2步：读取 file2.py (800行)
  → messages 增加 800行

第3步：读取 file3.py (1200行)
  → messages 增加 1200行

...

第10步：Claude想提出改进建议
  → 但上下文已经被文件内容填满了
  → 系统提示被淹没了
  → Claude开始胡说八道
```

**问题**：所有的中间结果都留在messages里，永远不会被清理。

---

## 🔑 解决方案：Subagent

```
主Agent                          子Agent
┌──────────────────┐             ┌──────────────────┐
│ messages=[...]   │             │ messages=[]      │ ← 干净！
│                  │  dispatch   │                  │
│ tool: task       │ ────────→   │ while tool_use:  │
│   prompt="..."   │             │   call tools     │
│                  │  summary    │   append results │
│   result="..."   │ ←────────   │ return last text │
└──────────────────┘             └──────────────────┘

主Agent的messages保持干净
子Agent的messages被丢弃
```

**核心思想**：
1. 主Agent遇到复杂子任务
2. 创建一个子Agent，给它一个干净的 `messages=[]`
3. 子Agent完成任务，只返回最终摘要
4. 子Agent的所有中间结果被丢弃
5. 主Agent继续工作，上下文保持干净

---

## 💻 Subagent 实现

```python
def run_subagent(prompt: str) -> str:
    """
    创建一个子Agent来完成任务
    返回最终摘要（中间结果被丢弃）
    """
    # 子Agent从干净的messages开始
    sub_messages = [{"role": "user", "content": prompt}]
    
    # 子Agent的循环（最多30轮）
    for round_num in range(30):
        response = client.messages.create(
            model="claude-3-5-sonnet-20241022",
            system=SUBAGENT_SYSTEM,  # 子Agent的系统提示
            messages=sub_messages,
            tools=CHILD_TOOLS,  # 子Agent没有 task 工具（防止递归）
            max_tokens=8000,
        )
        
        # 添加Claude的回复
        sub_messages.append({"role": "assistant", "content": response.content})
        
        # 检查是否完成
        if response.stop_reason != "tool_use":
            # 提取最后的文本作为摘要
            summary = ""
            for block in response.content:
                if hasattr(block, "text"):
                    summary += block.text
            return summary or "(无摘要)"
        
        # 执行工具
        results = []
        for block in response.content:
            if block.type == "tool_use":
                handler = TOOL_HANDLERS.get(block.name)
                output = handler(**block.input) if handler else "❌ 未知工具"
                results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": str(output)[:50000]  # 限制输出大小
                })
        
        # 添加工具结果
        sub_messages.append({"role": "user", "content": results})
    
    return "❌ 子Agent超过最大轮数"
```

---

## 🎮 主Agent中的Task工具

```python
# 主Agent的工具列表（包含 task）
PARENT_TOOLS = CHILD_TOOLS + [
    {
        "name": "task",
        "description": "创建一个子Agent来完成复杂子任务。子Agent有干净的上下文。",
        "input_schema": {
            "type": "object",
            "properties": {
                "prompt": {
                    "type": "string",
                    "description": "给子Agent的任务描述"
                }
            },
            "required": ["prompt"]
        }
    }
]

# 在主Agent的工具分发中
TOOL_HANDLERS = {
    "bash": lambda **kw: run_bash(kw["command"]),
    "read_file": lambda **kw: run_read_file(kw["path"]),
    "write_file": lambda **kw: run_write_file(kw["path"], kw["content"]),
    "edit_file": lambda **kw: run_edit_file(kw["path"], kw["old_text"], kw["new_text"]),
    "task": lambda **kw: run_subagent(kw["prompt"]),  # ← 新增
}
```

---

## 📊 执行流程示例

```
主Agent：分析这个项目的所有Python文件

主Agent思考：这太复杂了，我需要子Agent帮忙

主Agent调用 task 工具：
  prompt="读取 src/ 目录下的所有Python文件，
          总结每个文件的功能"

  ↓ 创建子Agent ↓

子Agent：我需要读取所有Python文件
  ↓
子Agent调用 bash：ls src/*.py
  ↓
子Agent调用 read_file：src/main.py
  ↓
子Agent调用 read_file：src/utils.py
  ↓
子Agent调用 read_file：src/config.py
  ↓
子Agent总结：
  "main.py: 程序入口，处理命令行参数
   utils.py: 工具函数，包括文件操作和数据处理
   config.py: 配置管理，读取和验证配置文件"

  ↓ 返回摘要 ↓

主Agent收到摘要：
  "main.py: 程序入口...
   utils.py: 工具函数...
   config.py: 配置管理..."

主Agent的messages保持干净！
  ↓
主Agent继续工作...
```

---

## 🔒 为什么子Agent没有 task 工具？

```python
# ❌ 错误：子Agent也有 task 工具
CHILD_TOOLS = PARENT_TOOLS  # 包含 task

# 会发生什么？
主Agent → 子Agent1 → 子Agent2 → 子Agent3 → ...
                                    ↓
                            无限递归！

# ✅ 正确：子Agent没有 task 工具
CHILD_TOOLS = [bash, read_file, write_file, edit_file]  # 不包含 task

# 子Agent只能用基础工具，不能创建更多子Agent
```

---

## 💻 完整的主Agent循环

```python
def agent_loop(query: str):
    """主Agent循环"""
    messages = [{"role": "user", "content": query}]
    
    print(f"\n🤖 主Agent: {query}\n")
    
    while True:
        response = client.messages.create(
            model="claude-3-5-sonnet-20241022",
            system=SYSTEM,
            messages=messages,
            tools=PARENT_TOOLS,  # 包含 task 工具
            max_tokens=8000,
        )
        
        messages.append({"role": "assistant", "content": response.content})
        
        if response.stop_reason != "tool_use":
            for block in response.content:
                if hasattr(block, "text"):
                    print(f"✅ 主Agent: {block.text}\n")
            return
        
        results = []
        for block in response.content:
            if block.type == "tool_use":
                tool_name = block.name
                
                if tool_name == "task":
                    # 创建子Agent
                    print(f"🔄 创建子Agent: {block.input['prompt'][:50]}...")
                    output = run_subagent(block.input["prompt"])
                    print(f"📤 子Agent返回: {output[:100]}...\n")
                else:
                    # 其他工具
                    handler = TOOL_HANDLERS.get(tool_name)
                    output = handler(**block.input) if handler else "❌ 未知工具"
                
                results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": output,
                })
        
        messages.append({"role": "user", "content": results})
```

---

## 📊 上下文对比

### 没有Subagent

```
主Agent messages:
  用户: "分析项目"
  Claude: "我需要读取文件"
  工具结果: file1.py (500行)
  Claude: "我需要读取更多文件"
  工具结果: file2.py (800行)
  工具结果: file3.py (1200行)
  ...
  总大小: 5000+ 行
  ❌ 系统提示被淹没
```

### 有Subagent

```
主Agent messages:
  用户: "分析项目"
  Claude: "我需要子Agent帮忙"
  工具结果: "main.py: 入口点\nutils.py: 工具函数\n..."
  Claude: "好的，现在我可以提出改进建议"
  ...
  总大小: 100 行
  ✅ 系统提示保持清晰
```

---

## 🎓 你来试试！

### 挑战4：实现Subagent系统

现在你要实现一个完整的Subagent系统。

**任务**：
1. 复制 `s03_todo_write.py` 到 `s04_subagent.py`
2. 实现 `run_subagent()` 函数
3. 添加 `task` 工具到 `PARENT_TOOLS`
4. 修改 `TOOL_HANDLERS`，添加 `task` 的处理
5. 测试它

**测试用例**：
```python
agent_loop("使用子任务来：1) 找出项目中有多少个Python文件 2) 总结每个文件的功能")
```

**预期行为**：
- 主Agent会调用 `task` 工具
- 子Agent会读取文件并总结
- 主Agent收到摘要后继续工作
- 主Agent的messages保持干净

---

## 💬 关键概念

| 概念 | 说明 |
|------|------|
| **主Agent** | 有 task 工具，管理大任务 |
| **子Agent** | 没有 task 工具，完成子任务 |
| **messages** | 主Agent的保持干净，子Agent的被丢弃 |
| **摘要** | 子Agent返回的最终结果 |
| **递归防护** | 子Agent没有 task 工具 |

---

## 💬 口诀

> **"大任务分解，子Agent独立，摘要返回，上下文清洁。"**
>
> `主Agent → task → 子Agent → 摘要 → 主Agent`

---

## 📝 总结

| 概念 | 说明 |
|------|------|
| **run_subagent()** | 创建子Agent的函数 |
| **PARENT_TOOLS** | 主Agent的工具（包含task） |
| **CHILD_TOOLS** | 子Agent的工具（不包含task） |
| **sub_messages** | 子Agent的干净消息列表 |
| **摘要提取** | 从子Agent的最后回复中提取 |

---

## 🚀 下一步

完成挑战后，我们进入 **s05：Skills（知识加载）**

你会学到：
- 为什么不能把所有知识放在系统提示里
- 如何动态加载知识
- 如何用工具结果注入知识

**准备好了吗？** 😎
