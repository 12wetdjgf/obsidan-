# Claude Code 原理笔记 · 第01篇：最小化Agent循环

> 💡 **核心洞察**：一个工具 + 一个循环 = 一个完整的Agent。Bash就够了。

---

## 🎯 问题回顾

从上一篇我们知道Agent就是一个循环。但怎么**真正实现**它呢？

最小化的Agent需要什么？
- ✅ 一个LLM（Claude）
- ✅ 一个工具（Bash）
- ✅ 一个循环（while True）
- ✅ 一个停止条件（stop_reason）

**就这么多。** 其他都是优化。

---

## 🔧 工具：Bash

为什么选Bash？因为它是**万能的**。

```bash
# 读文件
cat hello.py

# 写文件
echo "print('Hello')" > hello.py

# 运行代码
python hello.py

# 查看目录
ls -la

# 一切皆可bash
```

Bash可以做任何事。所以一个Agent只需要Bash工具就能工作。

---

## 💻 完整代码：s01_agent_loop.py

```python
from anthropic import Anthropic

# 初始化客户端
client = Anthropic()

# 定义系统提示
SYSTEM = """你是一个编程助手。
用户会给你任务，你需要用bash工具完成它们。
思考清楚，然后调用工具。"""

# 定义Bash工具的schema
TOOLS = [
    {
        "name": "bash",
        "description": "在终端运行bash命令",
        "input_schema": {
            "type": "object",
            "properties": {
                "command": {
                    "type": "string",
                    "description": "要运行的bash命令"
                }
            },
            "required": ["command"]
        }
    }
]

def run_bash(command: str) -> str:
    """执行bash命令，返回输出"""
    import subprocess
    try:
        result = subprocess.run(
            command,
            shell=True,
            capture_output=True,
            text=True,
            timeout=10
        )
        return result.stdout + result.stderr
    except subprocess.TimeoutExpired:
        return "命令超时"
    except Exception as e:
        return f"错误: {e}"

def agent_loop(query: str):
    """Agent主循环"""
    messages = [{"role": "user", "content": query}]
    
    print(f"\n🤖 用户: {query}\n")
    
    while True:
        # 第1步：调用Claude
        response = client.messages.create(
            model="claude-3-5-sonnet-20241022",
            system=SYSTEM,
            messages=messages,
            tools=TOOLS,
            max_tokens=8000,
        )
        
        # 第2步：把Claude的回复加到历史
        messages.append({"role": "assistant", "content": response.content})
        
        # 第3步：检查是否需要用工具
        if response.stop_reason != "tool_use":
            # Claude完成了，打印最终答案
            for block in response.content:
                if hasattr(block, "text"):
                    print(f"✅ Claude: {block.text}\n")
            return
        
        # 第4步：执行Claude要求的工具
        results = []
        for block in response.content:
            if block.type == "tool_use":
                print(f"🔧 Claude要求: {block.name}(command='{block.input['command']}')")
                
                # 执行bash命令
                output = run_bash(block.input["command"])
                print(f"📤 输出: {output[:200]}...\n" if len(output) > 200 else f"📤 输出: {output}\n")
                
                results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": output,
                })
        
        # 第5步：把工具结果加到历史，继续循环
        messages.append({"role": "user", "content": results})

# 主程序
if __name__ == "__main__":
    # 试试这些任务：
    tasks = [
        "创建一个文件叫 hello.py，内容是打印 'Hello, World!'",
        "列出当前目录的所有Python文件",
        "当前git分支是什么？",
        "创建一个目录叫 test_output，然后在里面创建3个文件",
    ]
    
    for task in tasks:
        agent_loop(task)
        print("=" * 60)
```

---

## 🎮 运行步骤

### 第1步：设置环境

```bash
# 进入项目目录
cd C:\Users\14936\learn-claude-code

# 安装依赖
pip install -r requirements.txt

# 设置API密钥
# 编辑 .env 文件，添加你的 ANTHROPIC_API_KEY
```

### 第2步：运行Agent

```bash
python agents/s01_agent_loop.py
```

你会看到：
```
🤖 用户: 创建一个文件叫 hello.py，内容是打印 'Hello, World!'

🔧 Claude要求: bash(command='echo "print('Hello, World!')" > hello.py')
📤 输出: 

🔧 Claude要求: bash(command='cat hello.py')
📤 输出: print('Hello, World!')

✅ Claude: 完成了！我已经创建了 hello.py 文件...
```

---

## 🧠 执行流程详解

```
用户: "创建 hello.py"
  ↓
Claude: "我需要用bash创建这个文件"
  ↓
Agent执行: bash echo "..." > hello.py
  ↓
Agent返回: "文件已创建"
  ↓
Claude: "我再验证一下"
  ↓
Agent执行: bash cat hello.py
  ↓
Agent返回: "print('Hello, World!')"
  ↓
Claude: "完成了！"
  ↓
Agent停止循环，返回答案
```

---

## 📊 关键代码解析

### messages 数组的演变

```python
# 初始状态
messages = [
    {"role": "user", "content": "创建 hello.py"}
]

# Claude回复后
messages = [
    {"role": "user", "content": "创建 hello.py"},
    {"role": "assistant", "content": [
        {"type": "tool_use", "name": "bash", "input": {"command": "..."}}
    ]}
]

# 工具执行后
messages = [
    {"role": "user", "content": "创建 hello.py"},
    {"role": "assistant", "content": [...]},
    {"role": "user", "content": [
        {"type": "tool_result", "tool_use_id": "...", "content": "输出"}
    ]}
]

# 循环继续...
```

### stop_reason 的含义

```python
if response.stop_reason == "tool_use":
    # Claude说："我要用工具"
    # → 继续循环，执行工具
    
elif response.stop_reason == "end_turn":
    # Claude说："我完成了"
    # → 停止循环，返回答案
```

---

## 🎓 你来试试！

现在轮到你敲代码了。我给你一个**挑战**：

### 挑战1：添加一个新工具 `read_file`

目前Agent只有bash。现在你要添加一个专门的 `read_file` 工具。

**任务**：
1. 在 `TOOLS` 列表中添加 `read_file` 工具的schema
2. 创建 `run_read_file()` 函数
3. 在工具执行部分添加处理逻辑
4. 测试它能否读取文件

**提示**：
```python
# read_file 的 schema 应该是这样的：
{
    "name": "read_file",
    "description": "读取文件内容",
    "input_schema": {
        "type": "object",
        "properties": {
            "path": {
                "type": "string",
                "description": "文件路径"
            }
        },
        "required": ["path"]
    }
}

# run_read_file 函数应该这样写：
def run_read_file(path: str) -> str:
    try:
        with open(path, 'r', encoding='utf-8') as f:
            return f.read()
    except Exception as e:
        return f"错误: {e}"
```

**你的任务**：
- 把这两个东西加到代码里
- 修改工具执行部分，让它能处理 `read_file` 工具
- 运行测试：`agent_loop("读取 hello.py 的内容")`

---

## 💬 口诀

> **"一个bash，一个循环，一个stop_reason，一个Agent。"**
>
> `while stop_reason == "tool_use": 执行工具 → 继续循环`

---

## 📝 总结

| 概念 | 说明 |
|------|------|
| **TOOLS** | 告诉Claude有哪些工具可用 |
| **messages** | 对话历史，越来越长 |
| **stop_reason** | Claude的停止原因 |
| **tool_use** | Claude要求使用工具 |
| **tool_result** | 工具执行的结果 |

---

## 🚀 下一步

完成挑战后，我们进入 **s02：多工具系统** 

你会学到：
- 如何优雅地添加多个工具
- 工具分发（dispatch）的设计模式
- 路径安全（防止Agent逃出沙箱）

**准备好了吗？** 😎
