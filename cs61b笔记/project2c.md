# 嗨！欢迎来到 CS 61B 的 Project 2C 解析课堂~ 🎀✨

哈喽哈喽！我是你的非官方小助教！先说好哦，我不会直接给你代码的——因为你说了要**独立完成**，这点超级棒的！那我就帮你把这个项目**拆碎了嚼烂了**喂给你理解，好不好？走起！

---

## 📌 首先，这个项目到底在干嘛？

这个项目叫 **NGordnet**，简单来说就是一个**文字历史探索器**。它结合了两个东西：

1. **Google NGrams 数据集**：记录了英语单词在历史上每一年出现的次数
2. **WordNet**：一个描述单词之间语义关系的图（有向图！）

在 2A 和 2B 里你已经搞定了基础部分。现在 2C 要你做**两件新事情**：

---

## 🎯 任务一：处理 `k != 0` 的情况

### 这是什么意思？

之前在 2B，你处理的是 `k == 0`，也就是返回**所有**的下义词（hyponyms）。

现在用户可以说：*"嘿，我只想要最热门的 k 个下义词！"*

### 举个例子你就懂了 🍰

> **输入**: `words = ["food", "cake"]`, `startYear = 1950`, `endYear = 1990`, `k = 5`
>
> **意思是**: 找出所有既是 "food" 又是 "cake" 的下义词，然后看看在 1950-1990 年间，哪 5 个词出现次数**最多**，按字母序返回。
>
> **输出**: `[cake, cookie, kiss, snap, wafer]`

### 🧠 你需要理解的核心逻辑：

```
1. 先用 2B 的方法找到所有共同 hyponyms（这你已经会了！）
2. 对这些词，查询它们在 [startYear, endYear] 时间段的总出现次数
3. 按出现次数排序，取前 k 个
4. 把这 k 个词按字母序排好返回
```

### ⚠️ 注意几个坑（超重要！）：

| 坑 | 说明 |
|---|---|
| **用 count 不是 weight** | 要用出现总次数，不是频率/权重！ |
| **出现次数为 0 的不要** | 如果一个词在那段时间从没出现过，别返回它 |
| **不够 k 个怎么办？** | 有多少返回多少就好 |
| **全是 0 怎么办？** | 返回空列表 `[]` |
| **不要用全局静态变量！** | ❌ `public static NGramMap`——这是坏习惯！要通过构造函数或方法参数传递 |

### 💡 设计提示

你需要想清楚：**`HyponymsHandler` 怎么访问到 `NGramMap`？**

答案很简单——通过**构造函数注入**！就像这样（伪代码）：

```
// 伪代码，不是直接答案哦~
class HyponymsHandler {
    private WordNet wn;
    private NGramMap ngm;  // 通过构造函数传进来！

    HyponymsHandler(WordNet wn, NGramMap ngm) {
        this.wn = wn;
        this.ngm = ngm;
    }
}
```

这样你在处理请求时就能同时用到 WordNet（找 hyponyms）和 NGramMap（查频率）了！

---

## 🎯 任务二：找共同祖先（Common Ancestors）

### 这又是啥？

之前你找的是**下义词**（往下找），现在要你找**上义词/祖先**（往上找）！

> **Hyponyms = 往下走** 🔽
> **Ancestors = 往上走** 🔼

### 举个例子 🌳

在 WordNet 图里：

```
         occurrence
            |
         event
        /      \
   change    adjustment
```

- `"adjustment"` 的祖先 = `[adjustment, alteration, event, happening, modification, natural_event, occurrence, occurrent]`
- `"change"` 的祖先 = `[act, action, alteration, change, event, happening, human_action, human_activity, modification, natural_event, occurrence, occurrent]`

### 共同祖先呢？

> `words = ["change", "adjustment"]` 的共同祖先 = 那些**同时**包含 "change" 和 "adjustment" 作为下义词的词。
>
> 结果：`[alteration, event, happening, modification, natural_event, occurrence, occurrent]`

### 🧠 核心思路

```
对于每个输入单词：
    1. 找到它在 WordNet 中对应的所有节点（synset IDs）
    2. 从这些节点出发，沿着边"往上走"，收集所有祖先节点
    3. 把这些祖先节点对应的"词"收集起来

对所有输入单词的祖先词集合取交集 → 就是共同祖先！
```

### 💡 超级重要的设计提示

> 你在 2B 里找 hyponyms 用了**BFS/DFS 往下遍历**，对吧？
>
> 那找 ancestors 你只需要……**往上遍历**就好了！😎

想想看，你的图结构（用的是什么？邻接表？）能不能支持**反向查找**？

有两个思路：

1. **建一个反向图（reverse graph）**：原图的边是 parent→child，反向图就是 child→parent。找祖先时在反向图上做 BFS/DFS
2. **直接在构建图的时候同时维护两个方向**

第一种更干净，我个人推荐~ ✨

### 关于 `NgordnetQueryType`

你的 handler 现在需要判断请求类型：

```
if (queryType == NgordnetQueryType.HYPONYMS) {
    // 走你的 2B 逻辑（往下找）
} else if (queryType == NgordnetQueryType.ANCESTORS) {
    // 走新的逻辑（往上找）
}
```

而且！`k != 0` 对 ancestors 也适用哦，所以你的 popularity 过滤逻辑应该是**通用的**——不管是 hyponyms 还是 ancestors，最后都可能需要按频率筛选。

---

## 🏗️ 整体架构建议（伪代码级别）

```
handleRequest(query):
    words = query.words()
    startYear = query.startYear()
    endYear = query.endYear()
    k = query.k()
    queryType = query.type()

    // Step 1: 找到候选词集合
    if queryType == HYPONYMS:
        candidates = findCommonHyponyms(words)   // 你 2B 已经写好的
    else:
        candidates = findCommonAncestors(words)   // 新写的，反向遍历

    // Step 2: 处理 k
    if k == 0:
        return sort(candidates)    // 字母序排好直接返回
    else:
        // 按 [startYear, endYear] 的总出现次数排序
        // 取前 k 个（过滤掉出现次数为 0 的）
        // 按字母序返回
        return filterByPopularity(candidates, startYear, endYear, k)
```

看到没？**hyponyms 和 ancestors 的 k 过滤逻辑完全一样！** 这就是为什么 spec 说"不需要复制粘贴代码"——你可以写一个通用的 helper method！

---

## 🧪 测试建议

1. **先用小文件测！** `synsets16.txt`, `hyponyms16.txt` 这种小文件
2. 用 [Staff Solution 网页](https://ngordnet.datastructur.es/) 生成期望输出
3. 用 [WordNet Visualizer](https://www.qxbytes.com/wordnet/) 可视化理解图结构
4. 自己写的 test case 比什么都重要！项目给的只有各一个 sanity check

---

## 📝 总结：你要做的事清单

- [ ] 1. 让 `HyponymsHandler` 能接收并使用 `NGramMap`（构造函数注入）
- [ ] 2. 实现 `k != 0`：按热度筛选 + 字母序返回
- [ ] 3. 建反向图或支持反向遍历
- [ ] 4. 实现 `findCommonAncestors`（和 hyponyms 对称的逻辑）
- [ ] 5. 根据 `NgordnetQueryType` 分发处理
- [ ] 6. 写充足的单元测试
- [ ] 7. 填好 `AutograderBuddy.java` 用于提交

---

加油鸭！这个项目看着复杂，但其实核心就是**图遍历 + 集合交集 + 排序**。你已经在 2B 里搞定了最难的部分，2C 只是在那个基础上加两个 feature 而已~

有什么具体的地方卡住了随时来问我！不过记住——代码要自己写哦！我只负责让你**理解** 😉💪