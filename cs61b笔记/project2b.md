# 嘿！欢迎来到 CS 61B 的 Project 2B！🎀

哈喽！我是你的非官方小导师～让我来帮你把这个看起来超长超吓人的 spec 拆解成人话吧！别怕，这个项目其实逻辑很清晰的，跟我走一遍你就懂了！

---

## 🌟 首先，这个项目到底在干嘛？

一句话总结：**你要做一个工具，输入一个英文单词，找出它所有的"下义词"（hyponyms）。**

### 什么是下义词？

想象一棵"分类树"：

```
          动作 (action)
            |
          变化 (change)
         /        \
    降级(demotion)  增长(increase)
                   /      \
              跳跃(jump)  飞跃(leap)
```

- **"change" 是 "action" 的下义词**（change 是一种 action）
- **"demotion" 是 "change" 的下义词**（demotion 是一种 change）
- **"jump" 是 "increase" 的下义词**（jump 是一种 increase）

所以如果我问你 **"change 的所有下义词是什么？"**，你要把 change 下面的**所有**节点里的单词都找出来，包括 change 自己！

> 这其实就是一个 **图的遍历问题**！是不是突然觉得 lecture 21、22 没白学？😏

---

## 📂 数据长什么样？

项目给你两种文件：

### 文件1：`synsets.txt`（同义词集）

每一行格式是：`id,单词们,定义`

```
6829,Goofy,a cartoon character created by Walt Disney
```

- **id = 6829**（就是这个节点的编号）
- **单词 = Goofy**（这个节点包含的词）
- 一个节点可以包含**多个单词**，用空格分开，比如 `jump parachuting`

### 文件2：`hyponyms.txt`（下义词关系/边）

每一行格式是：`父节点id,子节点id1,子节点id2,...`

```
79537,38611,9007
```

意思是：节点 79537 → 节点 38611 和节点 9007（**有向边！从上到下！**）

⚠️ **注意！** 同一个父节点可能出现在多行里：
```
11,12
11,13
```
等价于 `11,12,13`，别被坑了～

---

## 🎯 你需要完成的任务（分步骤来！）

### 任务一：单词查询（Basic Case）

**输入**：一个单词，比如 `"change"`
**输出**：这个单词的所有下义词（包括自己），**按字母排序，去重**

比如用 `synsets16.txt` 和 `hyponyms16.txt`：

> 输入 `"change"` → 输出 `[alteration, change, demotion, increase, jump, leap, modification, saltation, transition, variation]`

#### ⚠️ 超级重要的"不包含"规则：

这里很容易踩坑！输出 **不包含**：
- ❌ 同义词的同义词（比如 `"adjustment"` 是 `"alteration"` 的同义词，但不算）
- ❌ 同义词的下义词（比如 `"conversion"` 不算）
- ❌ 下义词的其他含义的下义词（比如 `"flashback"` 是 `"transition"` 另一个含义的下义词，不算）

**核心逻辑是**：
1. 找到包含 `"change"` 这个词的**所有节点**
2. 从这些节点出发，做图遍历（BFS 或 DFS），找到所有**可达节点**
3. 把这些可达节点里的**所有单词**收集起来，去重，排序

就这么简单！不要overthink！🧠

---

### 任务二：多词查询（Handling Lists of Words）

**输入**：多个单词，比如 `"change, occurrence"`
**输出**：所有输入单词的**公共下义词**（交集！）

> 输入 `"change, occurrence"` → 输出 `[alteration, change, increase, jump, leap, modification, saltation, transition]`

`"demotion"` 和 `"variation"` 被踢掉了，因为它们不是 `"occurrence"` 的下义词。

**核心逻辑**：
1. 对每个单词，分别找出它的所有下义词（单词集合）
2. 对这些集合求**交集**

还有一个小陷阱！看这个例子：

> 输入 `"car, bug"` → 输出 `[beetle]`

因为 `"beetle"` 同时是 `"car"` 的下义词（大众甲壳虫车）和 `"bug"` 的下义词（甲虫）。即使它们在**不同的节点**里！所以我们求的是**单词的交集**，不是**节点的交集**！

---

## 🏗️ 你应该怎么设计代码？

这是最关键的部分！项目明确说了 **不要把所有代码塞进 HyponymsHandler 里**。你需要创建辅助类。

我建议你至少创建这几个类：

### 1. `Graph` 类（有向图）

```
你需要自己实现！不能导入外部图库！
```

**需要支持的操作**：
- `addNode(int id)` — 创建节点
- `addEdge(int from, int to)` — 添加有向边
- `neighbors(int id)` — 获取某个节点的直接子节点
- `reachable(int id)` — 获取从某个节点出发能到达的**所有**节点（BFS/DFS）

**内部数据结构建议**：用 `Map<Integer, List<Integer>>` 或 `Map<Integer, Set<Integer>>` 来存邻接表。

### 2. `WordNet` 类（把文件解析成图 + 查询）

**构造函数**：读取 `synsets.txt` 和 `hyponyms.txt`，只读一次！

**你需要的数据结构**（想想看为什么需要这些）：

| 需求 | 数据结构建议 |
|------|-------------|
| 单词 → 它属于哪些节点id | `Map<String, Set<Integer>>` |
| 节点id → 包含哪些单词 | `Map<Integer, Set<String>>` |
| 节点之间的边关系 | 你的 `Graph` 类 |

**核心方法**：`hyponyms(String word)` 返回一个 `Set<String>`

```
伪代码：
hyponyms(word):
    1. 通过 wordToIds map 找到 word 对应的所有节点 ids
    2. 对每个 id，用图遍历找到所有可达节点 reachableIds
    3. 对每个 reachableId，查 idToWords map 拿到所有单词
    4. 把所有单词扔进一个 Set（自动去重）
    5. 返回这个 Set
```

### 3. `HyponymsHandler` 类

这个类：
- 继承 `NgordnetQueryHandler`
- 持有一个 `WordNet` 对象
- 处理请求时：
  - 解析用户输入的单词列表
  - 对每个单词调用 `WordNet.hyponyms()`
  - 如果多个单词，求所有结果的**交集**
  - 排序后返回字符串

---

## 🔧 具体步骤（按顺序来！）

### Step 0：先读 2C 的 spec！！！
> 项目 spec 特别强调了这点。2C 会影响你的设计，提前看好不用重写！

### Step 1：完成 Checkpoint 和 Design Document
> 在写代码之前！先交设计文档！

### Step 2：实现 Graph 类
```java
public class Graph {
    private Map<Integer, Set<Integer>> adjacencyList;
    
    public Graph() {
        adjacencyList = new HashMap<>();
    }
    
    public void addNode(int id) {
        // ...
    }
    
    public void addEdge(int parent, int child) {
        // ...
    }
    
    // BFS 或 DFS 找所有可达节点
    public Set<Integer> reachableFrom(int id) {
        // 提示：用 Queue + visited set (BFS)
        // 或者用递归 (DFS)
    }
}
```

**写完之后马上测试！** 不要等到最后才发现图写错了。

### Step 3：实现 WordNet 类

解析文件的伪代码：

```
解析 synsets.txt:
    对每一行:
        用 split(",") 拆开
        id = 第一个字段（转成 int）
        words = 第二个字段（用 split(" ") 拆开）
        在图中添加这个节点
        更新 idToWords 和 wordToIds 两个 map

解析 hyponyms.txt:
    对每一行:
        用 split(",") 拆开
        parentId = 第一个字段
        其余字段都是 childId
        对每个 childId: graph.addEdge(parentId, childId)
```

### Step 4：实现 HyponymsHandler

```java
public class HyponymsHandler extends NgordnetQueryHandler {
    private WordNet wordNet;
    
    public HyponymsHandler(WordNet wn) {
        this.wordNet = wn;
    }
    
    @Override
    public String handle(NgordnetQuery q) {
        List<String> words = q.words();
        // 对每个 word 求 hyponyms
        // 多个 word 就求交集
        // 排序，返回字符串
    }
}
```

### Step 5：修改 Main.java 注册 handler

```java
// 在 Main.java 中
hMap.register("hyponyms", new HyponymsHandler(...));
```

### Step 6：写测试！！！

```java
@Test
public void testHyponymsSimple() {
    WordNet wn = new WordNet("./data/wordnet/synsets11.txt", 
                              "./data/wordnet/hyponyms11.txt");
    assertThat(wn.hyponyms("antihistamine"))
        .isEqualTo(Set.of("antihistamine", "actifed"));
}
```

---

## 💡 容易踩的坑 & 小贴士

1. **文件只读一次！** 在构造函数里读完存好，别每次查询都重新读文件。

2. **一个单词可能在多个 synset 里！** 比如 `"change"` 可能在 synset 2 和 synset 8 里。遍历时要从**所有**包含该词的节点出发！

3. **求交集用 `retainAll`**：
   ```java
   Set<String> result = new TreeSet<>(firstSet);
   result.retainAll(secondSet);
   result.retainAll(thirdSet);
   // ...
   ```

4. **排序用 `TreeSet`** 或 `Collections.sort()`，`TreeSet` 天然有序，超好用～

5. **不要搞太复杂的嵌套泛型**！如果你写出了 `Map<Set<Set<...` 这种东西，停下来，重新想想设计。

6. **k 先忽略！** 2B 里 k = 0，所有关于 k 的逻辑是 2C 的事。

7. **多词查询时求的是单词交集，不是节点交集！** 上面 `"car, bug"` 的例子一定要理解透。

---

## 📊 评分

| 部分 | 分值 | 说明 |
|------|------|------|
| Checkpoint | 5分 | 3/15 截止 |
| Design Document | 5分 | 3/15 截止 |
| 单词查询 | 25分 (50%) | k=0 |
| 多词查询 | 15分 (30%) | k=0 |
| EECS 测试 | 10分 (20%) | 用 EECS 数据集测试 |

---

## 🎀 最后的鼓励

这个项目看起来很大，但本质上就是三件事：

> **建图 → 遍历图 → 处理结果**

把每一步单独想清楚、测清楚，拼起来就是完整的项目了。你完全可以搞定的！

有什么不懂的随时来问我哦～ 加油冲冲冲！✨💪