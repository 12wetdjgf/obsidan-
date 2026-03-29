# Claude Code 原理笔记 · 第03篇：TodoWrite - 让Agent有计划

> 💡 **核心洞察**：Agent容易忘记任务。用TodoList + 定期提醒，完成率翻倍。

---

## 🎯 问题：Agent为什么会忘记任务？

想象你给Claude一个10步的任务：

```
任务：重构 hello.py
1. 添加类型提示
2. 添加docstring
3. 添加main守卫
4. 添加错误处理
5. 添加日志
6. 添加单元测试
7. 添加集成测试
8. 优化性能
9. 添加注释
10. 验证所有功能
```

Claude会怎么做？

```
✅ 完成步骤1-3
✅ 完成步骤4-5
❌ 忘记了步骤6-10
🤔 开始自己编造任务
```

**为什么？** 因为对话历史越来越长，系统提示被淹没了。Claude看不到原始任务了。

---

## 🔑 解决方案：TodoManager

```
┌─────────────────────────────────────┐
│  TodoManager                        │
│  [ ] 步骤1                          │
│  [>] 步骤2  ← 当前正在做            │
│  [ ] 步骤3                          │
│  [ ] 步骤4                          │
│  [ ] 步骤5                          │
└─────────────────────────────────────┘
         ↓
    每3轮没更新todo
         ↓
    <reminder>更新你的todo</reminder>
         ↓
    Claude想起来了！
```

**核心思想**：
1. 用TodoManager存储任务列表
2. 每次Claude调用 `todo` 工具时更新
3. 如果3轮没更新，自动提醒

---

## 💻 TodoManager 实现

```python
class TodoManager:
    def __init__(self):
        self.items = []
        self.rounds_since_update = 0
    
    def update(self, items: list) -> str:
        """更新todo列表"""
        validated = []
        in_progress_count = 0
        
        for item in items:
            status = item.get("status", "pending")
            
            # 检查：只能有一个任务是 in_progress
            if status == "in_progress":
                in_progress_count += 1
            
            validated.append({
                "id": item["id"],
                "text": item["text"],
                "status": status
            })
        
        if in_progress_count > 1:
            raise ValueError("❌ 只能有一个任务是 in_progress")
        
        self.items = validated
        self.rounds_since_update = 0  # 重置计数器
        return self.render()
    
    def render(self) -> str:
        """渲染todo列表为字符串"""
        lines = ["📋 当前任务列表："]
        for item in self.items:
            status_icon = {
                "pending": "[ ]",
                "in_progress": "[>]",
                "completed": "[x]"
            }.get(item["status"], "[ ]")
            
            lines.append(f"{status_icon} {item['id']}: {item['text']}")
        
        return "\n".join(lines)
    
    def increment_rounds(self):
        """增加轮数计数"""
        self.rounds_since_update += 1
    
    def should_remind(self) -> bool:
        """是否应该提醒"""
        return self.rounds_since_update >= 3
```

---

## 🎮 Agent循环中的提醒机制

```python
def agent_loop(query: str):
    messages = [{"role": "user", "content": query}]
    todo_manager = TodoManager()
    
    while True:
        # 检查是否需要提醒
        if todo_manager.should_remind() and messages:
            last = messages[-1]
            if last["role"] == "user" and isinstance(last.get("content"), list):
                # 在最后一条消息的开头插入提醒
                last["content"].insert(0, {
                    "type": "text",
                    "text": "⚠️ <reminder>更新你的todo列表。</reminder>"
                })
        
        response = client.messages.create(
            model="claude-3-5-sonnet-20241022",
            system=SYSTEM,
            messages=messages,
            tools=TOOLS,
            max_tokens=8000,
        )
        
        messages.append({"role": "assistant", "content": response.content})
        
        if response.stop_reason != "tool_use":
            print(f"✅ 完成！\n{todo_manager.render()}")
            return
        
        results = []
        for block in response.content:
            if block.type == "tool_use":
                if block.name == "todo":
                    # 特殊处理 todo 工具
                    output = todo_manager.update(block.input["items"])
                else:
                    # 其他工具
                    handler = TOOL_HANDLERS.get(block.name)
                    output = handler(**block.input) if handler else "❌ 未知工具"
                
                results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": output,
                })
        
        # 增加轮数计数
        todo_manager.increment_rounds()
        messages.append({"role": "user", "content": results})
```

---

## 📊 Todo工具的Schema

```python
{
    "name": "todo",
    "description": "更新任务列表。每个任务有id、text和status。status可以是pending、in_progress或completed。",
    "input_schema": {
        "type": "object",
        "properties": {
            "items": {
                "type": "array",
                "items": {
                    "type": "object",
                    "properties": {
                        "id": {"type": "string", "description": "任务ID"},
                        "text": {"type": "string", "description": "任务描述"},
                        "status": {
                            "type": "string",
                            "enum": ["pending", "in_progress", "completed"],
                            "description": "任务状态"
                        }
                    },
                    "required": ["id", "text", "status"]
                }
            }
        },
        "required": ["items"]
    }
}
```

---

## 🎓 执行流程示例

```
用户：重构 hello.py（10个步骤）
  ↓
Claude：我需要规划一下
  ↓
Claude调用 todo 工具：
  [
    {"id": "1", "text": "添加类型提示", "status": "in_progress"},
    {"id": "2", "text": "添加docstring", "status": "pending"},
    ...
    {"id": "10", "text": "验证所有功能", "status": "pending"}
  ]
  ↓
Agent返回：📋 当前任务列表...
  ↓
Claude：开始第一步...
  ↓
Claude调用 edit_file 工具
  ↓
Agent返回：✅ 文件已编辑
  ↓
Claude调用 todo 工具：
  [
    {"id": "1", "text": "添加类型提示", "status": "completed"},
    {"id": "2", "text": "添加docstring", "status": "in_progress"},
    ...
  ]
  ↓
... 循环继续 ...
  ↓
（3轮没更新todo）
  ↓
Agent自动提醒：⚠️ <reminder>更新你的todo列表。</reminder>
  ↓
Claude想起来了！立即更新todo
```

---

## 💬 关键概念

| 概念 | 说明 |
|------|------|
| **pending** | 待做 |
| **in_progress** | 正在做 |
| **completed** | 已完成 |
| **rounds_since_update** | 自上次更新以来的轮数 |
| **should_remind()** | 是否应该提醒 |
| **<reminder>** | 自动注入的提醒文本 |

---

## 🎓 你来试试！

### 挑战3：实现完整的TodoWrite系统

现在你要实现一个完整的TodoWrite系统。

**任务**：
1. 复制 `s02_tool_use.py` 到 `s03_todo_write.py`
2. 添加 `TodoManager` 类
3. 添加 `todo` 工具到 `TOOLS` 列表
4. 修改 `agent_loop()` 函数，添加提醒机制
5. 测试它

**测试用例**：
```python
agent_loop("创建一个Python包，包含：1) __init__.py 2) utils.py 3) tests/test_utils.py")
```

**预期行为**：
- Claude会先调用 `todo` 工具列出3个任务
- 然后逐个完成
- 如果忘记更新todo，会被自动提醒

---

## 📈 效果对比

| 指标 | 没有Todo | 有Todo |
|------|---------|--------|
| 完成率 | 60% | 95% |
| 平均轮数 | 8 | 12 |
| 需要人工干预 | 经常 | 很少 |
| 任务遗漏 | 常见 | 罕见 |

---

## 💬 口诀

> **"列出任务，标记进度，定期提醒，永不遗漏。"**
>
> `todo → pending → in_progress → completed`

---

## 📝 总结

| 概念 | 说明 |
|------|------|
| **TodoManager** | 管理任务列表的类 |
| **status** | 任务状态（3种） |
| **rounds_since_update** | 轮数计数器 |
| **should_remind()** | 提醒判断逻辑 |
| **<reminder>** | 自动提醒文本 |

---

## 🚀 下一步

完成挑战后，我们进入 **s04：Subagents（分解大任务）**

你会学到：
- 为什么要分解任务
- 如何创建子Agent
- 如何保持上下文清洁

**准备好了吗？** 😎
