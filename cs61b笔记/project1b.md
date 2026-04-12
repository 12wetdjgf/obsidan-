# 🌟 Project 1B: ArrayDeque61B 超详细解析！

嗨嗨！欢迎来到CS 61B的世界～我来帮你把这个作业拆解得明明白白的！放心，我不会直接给你代码（毕竟你要独立完成嘛），但我会让你**彻底理解**每一步在做什么！✨

---

## 📌 首先，这个作业到底要干嘛？

简单来说：你在 Project 1A 里用**链表**实现了一个双端队列（Deque），现在要用**数组**再实现一遍！

> **双端队列（Deque）** = 两头都能插入和删除的队列。前面能加、后面能加、前面能删、后面能删。

区别就是底层数据结构变了：

| | Project 1A | Project 1B |
|---|---|---|
| 底层结构 | 链表（LinkedList） | 数组（Array） |
| 类名 | `LinkedListDeque61B` | `ArrayDeque61B` |

---

## 🧠 核心概念：循环数组（Circular Array）

这是整个作业**最最最关键**的思想！不理解这个，后面全废。让我好好讲讲～

### 普通数组的问题

假设你有一个数组 `[_, _, _, a, b, c, _, _]`，`a` 是队头（front），`c` 是队尾（back）。

- `addLast` 很简单：往 `c` 后面放就行
- 但 `addFirst` 呢？往 `a` 前面放？那前面空间用完了咋办？

如果每次 `addFirst` 都把所有元素往后挪……那就是 O(n) 了，题目要求**常数时间**！💀

### 循环数组的妙处

想象把数组**首尾相连**，变成一个圆环🔵：

```
索引:    0   1   2   3   4   5   6   7
数组:   [e,  _,  _,  _,  _,  a,  b,  c]
                              ↑front    ↑back
```

如果我要 `addFirst`，新元素放在 `a` 前面一个位置，也就是索引 4。
如果我要 `addLast`，新元素放在 `c` 后面一个位置，也就是索引 0（绕回来了！）。

**这就是"循环"的意思！到了数组末尾就绕回开头，到了开头就绕回末尾。**

### Math.floorMod 是你的好朋友！

怎么实现"绕回来"？用 `Math.floorMod`！

```java
// 往前移一位（比如 addFirst）
newIndex = Math.floorMod(currentIndex - 1, array.length);

// 往后移一位（比如 addLast）
newIndex = Math.floorMod(currentIndex + 1, array.length);
```

普通的 `%` 遇到负数会出问题，比如 `-1 % 8 = -1`（错！），但 `Math.floorMod(-1, 8) = 7`（对！绕到最后去了）。

> 🎯 **记住这个**：`floorMod` 永远返回非负数，完美处理循环！

---

## 🏗️ 需要哪些实例变量？

作业不告诉你用什么变量，你得自己想。给你提示，你至少需要：

```
想想看，你需要追踪什么信息？

1. ________  → 存数据的数组本身
2. ________  → 当前有多少个元素
3. ________  → 队头在数组中的位置（索引）
4. ________  → 队尾在数组中的位置（索引）
```

> 💡 **提示**：`nextFirst` 和 `nextLast` 这种命名方式很常见——它们指向**下一个要插入的位置**，而不是当前元素的位置。这样设计会让 `addFirst` 和 `addLast` 的逻辑更简洁。

---

## 🔧 逐个方法解析

### 1. Constructor（构造器）

```java
public ArrayDeque61B() {
    // 题目要求：初始数组大小必须是 8
    // 初始化你的所有实例变量
    // nextFirst 和 nextLast 的初始值？随你选，但要合理
}
```

> 💡 比如你可以让 `nextFirst = 3`，`nextLast = 4`，这样它们从中间开始往两边扩展。或者 `nextFirst = 7`，`nextLast = 0` 也行。只要逻辑自洽就OK！

### 2. addFirst 和 addLast

思路非常直白：

```
addFirst(item):
    1. 把 item 放到 nextFirst 的位置
    2. nextFirst 往前移一位（用 floorMod 绕回来！）
    3. size 加 1
    4. 如果数组满了？→ 扩容！（先跳过，后面说）

addLast(item):
    1. 把 item 放到 nextLast 的位置
    2. nextLast 往后移一位（用 floorMod 绕回来！）
    3. size 加 1
    4. 如果数组满了？→ 扩容！
```

**关键**：不能用循环！这两个方法必须是 O(1) 的（扩容除外）。

### 3. get(int index)

这里 `index` 是**逻辑索引**（第0个元素、第1个元素……），不是数组的物理索引！

你需要把逻辑索引转换成物理索引：

```
想一想：
- 第0个元素（也就是 front）在数组中的物理位置是哪里？
- 如果 nextFirst 指向"下一个要插入的前端位置"，
  那当前 front 元素在哪？
- 知道 front 之后，第 i 个元素呢？

物理索引 = Math.floorMod(front的物理位置 + index, 数组长度)
```

必须是 O(1)！数组天然支持随机访问，一步到位～这就是数组比链表爽的地方😎

### 4. isEmpty 和 size

超简单，看你的 `size` 变量就行：

```
isEmpty → size == 0?
size → 直接返回 size
```

O(1)，没啥好说的。

### 5. toList

把 deque 里的元素按顺序放进一个 `ArrayList` 返回：

```
创建一个 ArrayList
从 front 开始，遍历 size 个元素，依次加进去
返回 ArrayList
```

> 💡 提示：你可以用你写好的 `get` 方法！

### 6. removeFirst 和 removeLast

和 add 是反过来的操作：

```
removeFirst:
    1. nextFirst 往后移一位（和 add 时移的方向相反！）
    2. 取出该位置的元素
    3. 把该位置设为 null（释放引用！题目要求的）
    4. size 减 1
    5. 检查是否需要缩容
    6. 返回取出的元素

removeLast:
    类似的，方向相反
```

**不能用循环**，O(1)！

### 7. getRecursive

这个是送的😂直接抛异常就行：

```java
@Override
public T getRecursive(int index) {
    throw new UnsupportedOperationException("No need to implement getRecursive for proj 1b");
}
```

---

## 📐 扩容和缩容（Resizing）—— 最难的部分！

### 扩容（Resize Up）

当数组**满了**（`size == array.length`），你需要：

1. 创建一个**更大的新数组**（通常是原来的 **2倍**，这就是"geometric factor"）
2. 把旧数组的元素**按正确顺序**复制到新数组
3. 更新 `nextFirst`、`nextLast` 等变量

```
旧数组（循环的，乱序的）:
  [d, e, _, _, _, a, b, c]
           ↑ 满了！

新数组（重新排列好）:
  [a, b, c, d, e, _, _, _, _, _, _, _, _, _, _, _]
                  ↑nextLast
```

> ⚠️ 作业**不推荐**用 `arraycopy`！建议用 `for` 循环 + `get` 方法（或者类似逻辑）按顺序复制。

### 缩容（Resize Down）

当元素太少（使用率 ≤ 25%，且数组长度 ≥ 16）时，要**缩小**数组：

```java
// 伪代码：在 remove 操作中
if (array.length >= 16 && size < array.length / 4) {
    // 缩小为原来的一半（或其他合理大小）
    resize(array.length / 2);
}
```

> 💡 缩容和扩容的逻辑几乎一样！建议写一个 `resize(int capacity)` 的辅助方法，扩容缩容都调用它。

---

## ✅ 测试驱动开发（TDD）

作业非常强调：**先写测试，再写实现！**

```java
@Test
public void addFirstAndLastTest() {
    Deque61B<Integer> ad = new ArrayDeque61B<>();
    ad.addFirst(10);   // [10]
    ad.addLast(20);    // [10, 20]
    ad.addFirst(5);    // [5, 10, 20]
    
    assertThat(ad.toList()).containsExactly(5, 10, 20).inOrder();
}
```

要测的场景包括但不限于：
- 🔲 空 deque 的 `isEmpty`、`size`
- ➕ 各种 add 组合
- 🔢 `get` 的正常情况和边界情况（负数索引、超大索引）
- ➖ remove 后检查正确性
- 📏 **触发扩容和缩容**后还能正常工作
- 🔄 add 和 remove 交替操作

---

## 🗺️ 推荐的完成顺序

1. **先不考虑扩容**，假设8个空间够用
2. 实现 `addFirst`、`addLast`
3. 实现 `toList`（用来验证）
4. 实现 `get`、`isEmpty`、`size`
5. 实现 `removeFirst`、`removeLast`
6. 确认一切在固定大小下正常工作 ✅
7. **最后加上扩容和缩容逻辑**
8. 疯狂测试！

---

## 🎯 评分分布

| 部分 | 占比 |
|---|---|
| Adding（addFirst, addLast, toList） | 25% |
| isEmpty, size | 5% |
| get | 10% |
| Removing（removeFirst, removeLast） | 30% |
| Memory（扩容缩容） | 20% |
| 测试覆盖率 | 10% |

---

## 💬 最后几句

这个作业的精髓就是**循环数组**。我第一次做的时候也画了一堆图才想明白的！建议你：

1. 📝 **拿纸画出来**！画一个数组，标出 `nextFirst` 和 `nextLast`，手动模拟 `addFirst`、`addLast`、`removeFirst`、`removeLast`，看看指针怎么动
2. 🖼️ 看看作业给的 [demo slides](https://docs.google.com/presentation/d/1kjbO8X7-i63NwQ_9wIt4HXr6APp2qc9PkghD-GO7_is/)，超有帮助
3. 🐛 一步一步来，别急着写扩容

你完全可以搞定的！有什么不懂的随时问我哦～ 加油！💪🌸