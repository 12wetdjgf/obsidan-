# 🌟 嘿！欢迎来到 CS 61B 的 Lab 06！让我来带你飞~ 🌟

哈喽！我是你的学习搭档！这个 lab 超有意思的，我们要从零开始写一个 **二叉搜索树（BST）** 版本的 Map！别慌，我一步一步带你搞懂它！

---

## 📖 首先，搞清楚我们到底要干嘛

这个 lab 的核心任务就是：

> **创建一个 `BSTMap` 类，用二叉搜索树（BST）作为底层数据结构，实现 `Map61B` 接口。**

你可以把 Map 想象成一个**字典**📚：
- **Key（键）**：你要查的"单词"
- **Value（值）**：那个单词的"释义"

而我们要用 **BST（二叉搜索树）** 来组织这个字典，让查找变得更快！

---

## 🌳 先搞懂 BST 是什么鬼

二叉搜索树的规则超级简单，就三条：

```
        8          ← 根节点
       / \
      3   10       ← 左边比父节点小，右边比父节点大
     / \    \
    1   6    14
       / \   /
      4   7 13
```

1. **左子树**的所有节点 **< 父节点**
2. **右子树**的所有节点 **> 父节点**
3. 左右子树本身也是 BST（递归定义！）

所以如果我要找 `6`：
- 从 `8` 开始 → 6 < 8 → 往左走
- 到 `3` → 6 > 3 → 往右走
- 到 `6` → 找到了！🎉

**这就是 BST 快的原因：每次比较都能排除一半的节点！**

---

## 🏗️ 现在来设计 BSTMap 的结构

### 第一步：类的声明（最容易踩坑的地方！）

```java
public class BSTMap<K extends Comparable<K>, V> implements Map61B<K, V> {
    // ...
}
```

⚠️ **注意这个 `K extends Comparable<K>`！** 

这叫**有界类型参数（bounded type parameter）**。为什么需要它呢？因为 BST 需要**比较** key 的大小来决定往左走还是往右走！如果 K 不能比较，那 BST 就没法工作了。

> 小知识：虽然 `Comparable` 是个接口，按理说应该用 `implements`，但在泛型里统一用 `extends`。Java 就是这么任性啦~ 🤷‍♀️

### 第二步：内部节点类

BST 是由一个一个节点组成的嘛，每个节点要存什么呢？

```java
private class BSTNode {
    K key;        // 键
    V value;      // 值
    BSTNode left;  // 左子节点
    BSTNode right; // 右子节点

    BSTNode(K key, V value) {
        this.key = key;
        this.value = value;
    }
}
```

### 第三步：成员变量

```java
private BSTNode root;  // 根节点
private int size;      // 当前有多少个键值对
```

---

## 🔧 逐个实现必需的方法

这是重头戏！我们需要实现 **5 个必需方法** + 1 个调试方法。

我**强烈建议**按这个顺序来实现（因为测试有依赖关系）：

### 1️⃣ `size()` — 最简单的，热热身

```java
@Override
public int size() {
    return size;
}
```

就这么简单！因为我们用一个变量 `size` 来追踪嘛~ 

---

### 2️⃣ `put(K key, V value)` — 核心方法！

**思路：** 从根节点开始，比较 key：
- key **小于**当前节点 → 往**左**走
- key **大于**当前节点 → 往**右**走
- key **等于**当前节点 → **更新** value
- 走到 `null` 了 → 在这里**插入**新节点！

**关键提示：用递归辅助方法！**

```java
@Override
public void put(K key, V value) {
    root = putHelper(root, key, value);
}

private BSTNode putHelper(BSTNode node, K key, V value) {
    if (node == null) {
        size++;
        return new BSTNode(key, value);  // 找到空位，插入！
    }

    int cmp = key.compareTo(node.key);

    if (cmp < 0) {
        // key 比当前节点小，往左走
        node.left = putHelper(node.left, key, value);
    } else if (cmp > 0) {
        // key 比当前节点大，往右走
        node.right = putHelper(node.right, key, value);
    } else {
        // key 相等！更新 value
        node.value = value;
    }

    return node;
}
```

💡 **这个递归模式超级经典！** 注意看：
- `node.left = putHelper(node.left, ...)` 这种写法让我们能够在递归返回时**自动链接**新节点
- 这就是所谓的 **"返回型递归"** 模式，在树的操作中超级常用！

---

### 3️⃣ `get(K key)` — 查找

**思路跟 put 几乎一样，只是不插入，而是返回找到的值：**

```java
@Override
public V get(K key) {
    return getHelper(root, key);
}

private V getHelper(BSTNode node, K key) {
    if (node == null) {
        return null;  // 没找到
    }

    int cmp = key.compareTo(node.key);

    if (cmp < 0) {
        return getHelper(node.left, key);
    } else if (cmp > 0) {
        return getHelper(node.right, key);
    } else {
        return node.value;  // 找到了！
    }
}
```

看到没？跟 `put` 的结构几乎一毛一样！BST 的美就在于**所有操作都遵循同样的"比较→选方向"的模式**。✨

---

### 4️⃣ `containsKey(K key)` — 判断 key 是否存在

有了 `get`，这个就是一行的事儿：

```java
@Override
public boolean containsKey(K key) {
    return get(key) != null;
}
```

等等！⚠️ 这里有个小陷阱：**如果 value 本身就是 `null` 呢？** 

不过对于这个 lab 来说，通常我们假设 value 不会是 null（看看 `Map61B` 的注释确认一下）。如果你想更严谨，可以写一个专门的 `containsKeyHelper`，像 `getHelper` 那样但返回 `boolean`。

---

### 5️⃣ `clear()` — 清空整棵树

```java
@Override
public void clear() {
    root = null;
    size = 0;
}
```

是的，就这么简单！Java 的**垃圾回收**会帮我们清理那些没有引用的节点~ 

---

### 6️⃣ `printInOrder()` — 中序遍历（调试用）

中序遍历 BST 会按**从小到大**的顺序输出所有 key！

```java
public void printInOrder() {
    printInOrderHelper(root);
    System.out.println();
}

private void printInOrderHelper(BSTNode node) {
    if (node == null) {
        return;
    }
    printInOrderHelper(node.left);      // 先左
    System.out.print(node.key + " ");   // 再自己
    printInOrderHelper(node.right);     // 后右
}
```

**为什么中序遍历是有序的？** 因为 BST 的性质保证了：左 < 根 < 右，递归展开就是从小到大！

---

### 7️⃣ 不实现的方法（抛异常）

对于 `iterator()`、`remove()` 和 `keySet()`，暂时这样写：

```java
@Override
public Iterator<K> iterator() {
    throw new UnsupportedOperationException();
}

@Override
public V remove(K key) {
    throw new UnsupportedOperationException();
}

@Override
public Set<K> keySet() {
    throw new UnsupportedOperationException();
}
```

---

## ⚡ 速度测试部分

完成 BSTMap 后，运行 `InsertRandomSpeedTest.java`，然后把结果写到 `speedTestResults.txt` 里。

你会观察到大概这样的趋势：

| 数据结构 | 随机插入 N 个元素的时间复杂度 |
|---------|-------------------------|
| **ULLMap**（无序链表）| O(N²) 😱 |
| **BSTMap**（你写的！）| O(N log N) 😎 |
| **TreeMap**（红黑树）| O(N log N) 🏆 |
| **HashMap** | O(N) 🚀 |

> **为什么 ULLMap 这么慢？** 因为每次 put 都要遍历整个链表检查 key 是否存在，这是 O(N) 的，N 次插入就是 O(N²)。
>
> **为什么 BSTMap 快很多？** 因为随机数据下，BST 大概是平衡的，每次操作 O(log N)，N 次就是 O(N log N)。

在 `speedTestResults.txt` 里写上你测试了什么、输入大小、观察到的时间就行！

---

## 🧠 可选的渐进分析题（加深理解）

快速过一下那些 True/False 题，帮你建立直觉：

| 题号 | 命题 | 答案 | 解释 |
|------|------|------|------|
| 1 | `put` ∈ O(log N) | **False** | 最坏情况下 BST 退化成链表，O(N) |
| 2 | `put` ∈ Θ(log N) | **False** | 同上，不是每次都 log N |
| 3 | `put` ∈ Θ(N) | **False** | 最好情况只需 O(1)（根就是要找的） |
| 4 | `put` ∈ O(N) | **True** ✅ | 最坏 O(N)，所以 O(N) 是上界 |
| 5 | `put` ∈ O(N²) | **True** ✅ | O(N) ⊂ O(N²)，大O是上界 |
| 6 | 两个 containsKey 都是 Ω(log N) | **False** | 如果 C 是根节点就是 O(1) |

第 7 题的 `mystery` 函数其实是在找 BST 中**第 z 小的元素**（select 操作）：
- **最好情况：** O(N) — 每次递归都正好找到
- **最坏情况：** O(N) — `numberOfNodes` 本身就是 Θ(M)，总共还是 O(N)

---

## 🎯 总结和建议

1. **先写节点类和成员变量**
2. **按 size → put → get → containsKey → clear 的顺序实现**
3. **大量使用递归辅助方法**（这是 BST 的精髓！）
4. **用 `TestBSTMap.java` 边写边测**
5. **最后跑速度测试，写结果文件**

递归思维是这个 lab 的精髓。如果你能理解 `putHelper` 那个"递归下去，返回时链接"的模式，其他所有方法都是同样的套路！

加油！你一定可以搞定的！有什么不懂的随时来问我哦~ 💪✨