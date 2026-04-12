# 🌟 Lab 08: HashMap 超详细解析！跟我一起搞定它吧～

嗨嗨！欢迎来到 CS 61B 的 HashMap 实验！我是你的小导师～虽然这个 lab 看起来很长很吓人，但其实拆开来一步步做超级有成就感的！来，我们一起把它拆解掉！✨

---

## 📌 首先，搞清楚我们到底要做什么

简单来说，这个 lab 要你 **从零实现一个 HashMap**，叫 `MyHashMap`。

你可能会问：HashMap 是啥？

> HashMap 就是一个 **键值对（key-value）** 的存储结构，你给它一个 key，它能 **超快** 地帮你找到对应的 value。就像一本字典📖，你查一个单词（key），马上得到释义（value）。

它的底层原理是 **哈希表（Hash Table）**，核心思想是：

```
bucket_index = hashCode(key) % 桶的数量M
```

把 key 通过一个哈希函数算出一个数字，然后取模，决定这个 key-value 放到哪个"桶（bucket）"里。

---

## 🧠 核心概念先搞懂！（这个超重要）

### 1. 哈希表的结构

想象你有一个 **数组**，数组的每个格子就是一个"桶"：

```
buckets: [ 桶0, 桶1, 桶2, 桶3, ..., 桶M-1 ]
```

每个桶里可以放 **多个元素**（因为不同的 key 可能算出相同的 index，这叫 **哈希冲突/collision**）。

### 2. 处理冲突：Separate Chaining（分离链接法）

冲突了怎么办？每个桶不是只放一个东西，而是放一个 **链表**（或其他集合）！

```
buckets[3] → [("apple", 5), ("banana", 7), ("cherry", 2)]
```

所以底层数据结构就是：**一个数组，每个元素是一个 Collection（集合）**。

### 3. Load Factor（负载因子）

```
loadFactor = N / M
```

- **N** = 已存入的键值对数量
- **M** = 桶的数量

当 `loadFactor` 超过阈值（默认 0.75），就要 **扩容（resize）**——把桶的数量翻倍，然后把所有元素重新哈希分配（rehash）。

### 4. Node 类

lab 已经给你了：

```java
protected class Node {
    K key;
    V value;
    Node(K k, V v) {
        key = k;
        value = v;
    }
}
```

每个 Node 就是一个键值对。桶里装的就是一堆 Node。

---

## 🔧 实现步骤拆解（手把手带你走一遍思路）

### Step 0: 实例变量

你需要这些：

```java
private Collection<Node>[] buckets;  // 桶数组（已给）
private int size;                     // 当前键值对数量 N
private int initialCapacity;          // 桶的数量 M
private double loadFactor;            // 最大负载因子
```

### Step 1: 构造函数（3个）

```java
public MyHashMap();                              // 默认: capacity=16, loadFactor=0.75
public MyHashMap(int initialCapacity);           // 自定义capacity, loadFactor=0.75  
public MyHashMap(int initialCapacity, double loadFactor); // 都自定义
```

**思路：**
- 保存参数
- `size = 0`
- 创建桶数组：`buckets = new Collection[initialCapacity]`（注意⚠️不能写泛型！）
- 给每个桶初始化：用 `createBucket()` 方法！

```java
// 伪代码示意（你自己写哦～）
for (int i = 0; i < initialCapacity; i++) {
    buckets[i] = createBucket();  // 不要直接 new LinkedList！用工厂方法！
}
```

> 💡 **为什么用 `createBucket()`？** 因为后面要测试不同的桶类型（LinkedList、ArrayList、HashSet等），子类会重写这个方法。这就是多态的魅力呀～

### Step 2: `createBucket()` 方法

超简单：

```java
protected Collection<Node> createBucket() {
    return new LinkedList<>();  // 或 ArrayList，随你喜欢
}
```

### Step 3: 一个私有辅助方法——根据 key 找到对应的桶的 index

```java
private int getBucketIndex(K key) {
    return Math.floorMod(key.hashCode(), buckets.length);
}
```

> ⚠️ **超级重要！** 用 `Math.floorMod()` 而不是 `%`！因为 `hashCode()` 可能返回负数，`%` 遇到负数会返回负数，而 `Math.floorMod()` 永远返回非负值，就像 Python 的 `%` 一样～

### Step 4: 再来一个辅助方法——在桶里找 Node

```java
// 在某个桶里找到 key 对应的 Node，找不到返回 null
private Node getNode(K key) {
    int index = getBucketIndex(key);
    for (Node node : buckets[index]) {
        if (node.key.equals(key)) {
            return node;
        }
    }
    return null;
}
```

> 用 `.equals()` 比较 key！不要用 `==`！这是 Java 的经典坑～

### Step 5: 实现各个方法

#### `size()`
```java
public int size() {
    return size;
}
```

就这么简单哈哈😆

#### `get(K key)`

```java
public V get(K key) {
    Node node = getNode(key);
    if (node == null) return null;
    return node.value;
}
```

#### `containsKey(K key)`

```java
public boolean containsKey(K key) {
    return getNode(key) != null;
}
```

#### `put(K key, V value)` ⭐ 这是重头戏！

**思路：**
1. 先看 key 是否已经存在 → 如果存在，**更新** value
2. 如果不存在，**新增** Node
3. 新增后检查是否需要 resize

```
put(key, value):
    node = getNode(key)
    if node 存在:
        node.value = value   // 更新！不增加 size
        return
    
    // 不存在，新增
    index = getBucketIndex(key)
    buckets[index].add(new Node(key, value))
    size += 1
    
    // 检查是否需要扩容
    if ((double) size / buckets.length > loadFactor):
        resize()
```

#### `resize()` ⭐ 也很重要！

**思路：几何扩容（乘以2），然后 rehash 所有元素**

```
resize():
    newBuckets = new Collection[buckets.length * 2]
    // 初始化每个新桶
    for i in range(newBuckets.length):
        newBuckets[i] = createBucket()
    
    // 把旧桶里所有 Node 重新分配到新桶
    for 每个旧桶 in buckets:
        for 每个 node in 旧桶:
            newIndex = Math.floorMod(node.key.hashCode(), newBuckets.length)
            newBuckets[newIndex].add(node)
    
    buckets = newBuckets
```

> 💡 注意：resize 时要用 **新数组的长度** 重新算 index！不是用旧的！

#### `clear()`

```java
public void clear() {
    size = 0;
    buckets = new Collection[buckets.length]; // 或用 initialCapacity
    for (int i = 0; i < buckets.length; i++) {
        buckets[i] = createBucket();
    }
}
```

#### 不需要实现的方法

```java
public V remove(K key) {
    throw new UnsupportedOperationException();
}

public Set<K> keySet() {
    throw new UnsupportedOperationException();
}

public Iterator<K> iterator() {
    throw new UnsupportedOperationException();
}
```

---

## 🎯 完整实现的检查清单

| 方法 | 状态 |
|------|------|
| 3个构造函数 | ☐ |
| `createBucket()` | ☐ |
| `getBucketIndex()` (辅助) | ☐ |
| `getNode()` (辅助) | ☐ |
| `size()` | ☐ |
| `get()` | ☐ |
| `containsKey()` | ☐ |
| `put()` | ☐ |
| `resize()` (辅助) | ☐ |
| `clear()` | ☐ |
| `remove/keySet/iterator` 抛异常 | ☐ |

---

## ⚡ 易错点总结（划重点啦！）

### 1. 泛型数组不能直接创建
```java
// ❌ 错误！
buckets = new Collection<Node>[size];

// ✅ 正确！
buckets = new Collection[size];
```

### 2. 一定要用 `Math.floorMod()`
```java
// ❌ 可能返回负数！
int index = key.hashCode() % buckets.length;

// ✅ 永远非负！
int index = Math.floorMod(key.hashCode(), buckets.length);
```

### 3. 用 `createBucket()` 而不是直接 new
```java
// ❌ 这样子类无法替换桶类型
buckets[i] = new LinkedList<>();

// ✅ 多态的力量！
buckets[i] = createBucket();
```

### 4. put 的时候如果 key 已存在要更新，不是再 add 一个！

### 5. resize 是 **乘法**（×2），不是加法（+16 之类的）！

---

## 📊 Speed Test 部分（results.txt）

做完代码后，运行 speed test，你会发现：

1. **`MyHashMap` vs `ULLMap`**：MyHashMap 快得多！ULLMap 是链表实现，每次查找都是 O(N)，而 HashMap 平均 O(1)。
2. **`MyHashMap` vs Java `HashMap`**：差距不大，可能在 10 倍以内。说明 HashMap 相比 TreeMap 更容易实现到接近工业水平！
3. **不同 bucket 类型**：LinkedList、ArrayList、HashSet 等作为桶，在实际表现上差异不大，因为如果负载因子控制得好，每个桶里的元素很少。

在 `results.txt` 里记录你的观测结果就行～

---

## 🌈 最后的鼓励

这个 lab 看起来很长，但核心就是 **put、get、resize** 三个方法。其他的都是小菜一碟！

你只要理解了：
1. **哈希 → 取模 → 找桶**
2. **桶里遍历找 key**
3. **超过负载因子就扩容重哈希**

这三个核心思想，整个 lab 就是 **把思想翻译成代码** 而已！

加油！你一定可以搞定的！💪 有什么不懂的随时来问我哦～我超级乐意帮忙的嘻嘻 ✨