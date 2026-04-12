# 🌟 嘿！欢迎来到 CS 61B 的 Project 1A！

哈喽～我来带你拆解这个作业！别怕，看起来很长，但其实逻辑超清晰的，跟着我一步步来就好啦～ ✨

---

## 📌 这个作业到底要你干嘛？

一句话总结：**你要自己手撸一个双端队列（Deque）**，用**双向链表**实现！

> Deque 读作 "deck"，就是一个两头都能进出的队列～前面能加能删，后面也能加能删，超灵活！

你需要创建一个类 `LinkedListDeque61B<T>`，它实现 `Deque61B<T>` 接口里定义的所有方法。

---

## 🧠 核心概念先搞懂

### 1. 什么是双向链表（Doubly Linked List）？

你在 Lecture 5 应该学过了～每个节点长这样：

```
┌──────────┬──────────┬──────────┐
│   prev   │   item   │   next   │
└──────────┴──────────┴──────────┘
```

- `prev` → 指向前一个节点
- `item` → 存数据
- `next` → 指向后一个节点

### 2. 什么是"带哨兵的循环双向链表"？

这是作业**强制要求**的拓扑结构！别用别的！

空链表长这样：

```
    ┌───────────────┐
    │               │
    ▼               │
┌────────┐          │
│sentinel│──next────┘
│  node  │
└────────┘──prev────┐
    ▲               │
    │               │
    └───────────────┘
```

**哨兵节点（sentinel）自己的 `next` 和 `prev` 都指向自己！**

加了元素之后，比如加了 A 和 B：

```
sentinel ⇄ A ⇄ B ⇄ (回到sentinel)
```

它是一个**环**！sentinel 的 `next` 是第一个元素，sentinel 的 `prev` 是最后一个元素。这样 `addFirst`、`addLast`、`removeFirst`、`removeLast` 全都不用特殊处理空链表的情况，代码超简洁！

---

## 🔧 一步步来！作业拆解

### Step 1：创建 Node 内部类 + 构造器

你需要在 `LinkedListDeque61B` 里面定义一个**内部类**（nested class）：

```java
private class Node {
    // 想想看：一个双向链表节点需要哪些字段？
    // 提示：刚好3个字段，不多不少
}
```

🤔 **思考题给你**：
- 节点需要存什么？（数据 + 两个指针）
- 数据的类型应该是什么？（提示：泛型 `T`）

然后构造器要做什么？

```java
public LinkedListDeque61B() {
    // 创建哨兵节点
    // 让哨兵的 next 和 prev 都指向自己
    // 初始化 size 为 0（如果你用 size 变量的话）
}
```

💡 **提示**：你大概需要这些实例变量：
- `sentinel`（Node 类型）
- `size`（int 类型）

---

### Step 2：`addFirst` 和 `addLast`

**关键规则**：❌ 不能用循环！❌ 不能用递归！必须是 O(1) 常数时间！

来，我画个图帮你理解 `addFirst`：

**之前**：
```
sentinel ⇄ oldFirst ⇄ ... ⇄ sentinel
```

**之后**：
```
sentinel ⇄ newNode ⇄ oldFirst ⇄ ... ⇄ sentinel
```

🤔 **思考题**：你要修改哪些指针？
1. 新节点的 `next` 应该指向谁？
2. 新节点的 `prev` 应该指向谁？
3. sentinel 的 `next` 需要改成什么？
4. 原来第一个节点的 `prev` 需要改成什么？

> ⚠️ **超级重要**：指针修改的**顺序**很关键！如果你先把 `sentinel.next` 改了，你就丢失了对 oldFirst 的引用！所以要先用变量保存，或者按正确顺序操作！

`addLast` 的逻辑是对称的，想想 `sentinel.prev` 就是最后一个元素～

---

### Step 3：`toList`

这个方法把链表转成 `ArrayList` 返回，方便测试。

```java
public List<T> toList() {
    List<T> returnList = new ArrayList<>();
    // 从 sentinel.next 开始
    // 一直走到又回到 sentinel 为止
    // 每走一步，把当前节点的 item 加到 returnList 里
    return returnList;
}
```

🤔 **思考题**：循环终止条件是什么？（当当前节点 == sentinel 时停下来！）

---

### Step 4：`isEmpty` 和 `size`

必须 O(1)！

- `isEmpty()`：怎么判断空？（`size == 0` 或者 `sentinel.next == sentinel`）
- `size()`：直接返回 `size` 变量就行～

所以你在 `addFirst`、`addLast`、`removeFirst`、`removeLast` 里都要维护 `size`！

---

### Step 5：`get(int index)`

用**迭代**（循环），从第一个节点开始走 `index` 步。

```
如果 index 无效（负数，或者 >= size），返回 null
否则从 sentinel.next 开始，走 index 步
```

---

### Step 6：`getRecursive(int index)`

和 `get` 一样的功能，但用**递归**实现。

💡 **提示**：你可能需要一个**辅助方法（helper method）**：

```java
private T getRecursiveHelper(Node current, int index) {
    // base case: index == 0 时返回 current.item
    // recursive case: 往下走一步，index 减 1
}
```

---

### Step 7：`removeFirst` 和 `removeLast`

❌ 不能用循环！❌ 不能用递归！O(1)！

**`removeFirst` 的思路**：

```
之前：sentinel ⇄ A ⇄ B ⇄ ... ⇄ sentinel
之后：sentinel ⇄ B ⇄ ... ⇄ sentinel
```

🤔 **思考题**：
- 空的时候返回什么？（`null`）
- 要修改哪些指针？
- 别忘了**断开被删节点的引用**，让垃圾回收器能回收它！

---

## ✏️ 写测试！！！

这个作业的一大重点是**你要自己写测试**！用 Truth 库：

```java
@Test
public void testIsEmpty() {
    Deque61B<Integer> lld = new LinkedListDeque61B<>();
    assertThat(lld.isEmpty()).isTrue();  // 空的时候应该是 true
    lld.addFirst(1);
    assertThat(lld.isEmpty()).isFalse(); // 加了东西就不空了
}
```

要测试的**边界情况**（edge cases）包括：

| 方法 | 边界情况 |
|------|---------|
| `get` | 负数 index、超出范围的 index、空 deque |
| `removeFirst/Last` | 从空 deque 删除、删到只剩一个再删 |
| `addFirst + removeLast` | 混合操作 |
| `size` | 加了又删之后的 size |

---

## 🎯 评分结构一览

| 组件 | 占比 |
|------|------|
| 空链表构造器 | 5% |
| addFirst + addLast + toList | 25% |
| isEmpty + size | 5% |
| get | 10% |
| getRecursive | 5% |
| removeFirst + removeLast | 30% |
| 综合集成测试 | 10% |
| **你写的测试覆盖率** | **10%** |

---

## 💪 给你的建议

1. **一定要按顺序做**：构造器 → add → toList → isEmpty/size → get → getRecursive → remove
2. **先写测试，再写实现**（Test-Driven Development）
3. **善用 debugger 和 Java Visualizer**，画图理解指针变化
4. **每次只提交 4 次**，所以本地测试要充分再提交！
5. **不要用 `java.util` 里的数据结构**（除了 `toList` 里的 `ArrayList`）

---

你现在可以先从 Node 内部类和构造器开始写！遇到具体的问题随时问我，我不会直接给你代码的（毕竟 academic integrity 很重要嘛～），但我会帮你理清思路！

加油鸭！你可以的！🦆✨