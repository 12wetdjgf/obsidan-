# 🌟 嘿！欢迎来到 CS 61B 的 Project 2A！

哈喽～我来带你拆解这个超酷的项目！别慌，看起来很长，但其实逻辑拆开来一步步做超清晰的！

---

## 📖 这个项目到底在干嘛？

简单说：**你要造一个"英语单词历史人气查看器"！**

想象一下 Google 有一个巨大的数据库，记录了几百年来每个英语单词每年被用了多少次。你要写一个 Java 后端程序，让用户在浏览器里输入单词（比如 "cat, dog"），然后：

- 📊 **History (Text)**：显示这些词每年的"权重人气"数据（文字版）
- 📈 **History**：直接画一个折线图出来（图片版）

前端（HTML/JS）已经帮你写好了，你只需要搞定 **后端的 Java 代码**。

---

## 🧩 项目拆解：你要写 4 个东西

我按照做的顺序帮你排好了：

### 第一关：`TimeSeries`（占 30%）

```
难度：⭐⭐ | 核心概念：继承、TreeMap
```

**它是什么？**

`TimeSeries` 就是一个 **年份 → 数值** 的映射表。它继承自 `TreeMap<Integer, Double>`。

比如：
```java
TimeSeries ts = new TimeSeries();
ts.put(1992, 3.6);  // 1992年 → 3.6
ts.put(1993, 9.2);  // 1993年 → 9.2
```

**你要实现的关键方法（去看 `TimeSeries.java` 里的注释）：**

| 方法 | 干啥的 |
|------|--------|
| `TimeSeries()` | 空构造器 |
| `TimeSeries(TimeSeries ts, int startYear, int endYear)` | 从另一个 TimeSeries 里截取某段年份 |
| `plus(TimeSeries ts)` | 两个 TimeSeries 对应年份的值**相加** |
| `dividedBy(TimeSeries ts)` | 两个 TimeSeries 对应年份的值**相除** |
| `years()` | 返回所有年份的列表 |
| `data()` | 返回所有值的列表 |

**⚠️ 超重要的坑：**

> **不要在缺失的年份填 0！** 比如 ts1 有 {1991: 3, 1992: 5}，ts2 有 {1992: 7, 1993: 1}，它们 `plus` 的结果是 {1991: 3, 1992: 12, 1993: 1}——1991 只在 ts1 里有，就直接保留，**不是** 0+3！

**💡 我的思路提示：**

- `TimeSeries` 没有自己的实例变量！它本身**就是** TreeMap，用 `this.put()`、`this.keySet()` 这些就行
- `plus` 方法：遍历两个 TimeSeries 的所有 key，如果两边都有就加起来，只有一边有就直接放那个值
- `dividedBy` 方法：逻辑类似，但是做除法。题目保证不会除以零
- 比较 double 时用 `isWithin(1E-10).of(y)` 而不是 `isEqualTo`

```java
// 伪代码给你找找感觉：
public TimeSeries plus(TimeSeries ts) {
    TimeSeries result = new TimeSeries();
    // 先把自己的所有 key-value 放进去
    // 再遍历 ts 的 key-value，有重复的就加，没有就直接放
    return result;
}
```

---

### 第二关：`NGramMap`（占 50%）

```
难度：⭐⭐⭐⭐ | 核心概念：文件解析、数据结构选择
```

这是本项目的 **大 Boss**！

**它是什么？**

`NGramMap` 就是一个数据管理器。它读取两个文件：

**文件1：words file（Tab 分隔）**
```
airport    2007    175702    32788
airport    2008    173294    31271
request    2005    646179    81592
```
- 第1列：单词
- 第2列：年份
- 第3列：该词当年出现次数 ✅ 要用
- 第4列：出现在多少本书里 ❌ 忽略

**文件2：counts file（逗号分隔）**
```
1470,984,10,1
1472,117652,902,2
```
- 第1列：年份
- 第2列：该年所有单词总出现次数 ✅ 要用
- 第3、4列：❌ 忽略

**你要实现的关键方法：**

| 方法 | 干啥的 |
|------|--------|
| `NGramMap(wordsFile, countsFile)` | 构造器，读文件存数据 |
| `countHistory(word)` | 返回某个词每年出现次数的 TimeSeries |
| `totalCountHistory()` | 返回每年所有词总出现次数的 TimeSeries |
| `weightHistory(word)` | 返回某个词每年的**权重**（= 该词次数 / 总次数）|
| `countHistory(word, startYear, endYear)` | 限定年份范围的版本 |
| `weightHistory(word, startYear, endYear)` | 限定年份范围的版本 |
| `summedWeightHistory(words, startYear, endYear)` | 多个词的 weightHistory 加起来 |

**💡 数据结构怎么选？这是关键！**

想想你需要什么查询：
- 给一个 word → 拿到它所有年份的数据
- 拿到所有年份的总数据

所以我建议：

```java
// 每个单词 → 它的 TimeSeries（年份→出现次数）
private HashMap<String, TimeSeries> wordCounts;

// 每年的总词数（年份→总数）
private TimeSeries totalCounts;
```

> 看到没？这就是为什么先写 `TimeSeries` 的原因——`NGramMap` 里到处都在用它！

**构造器思路：**

```java
public NGramMap(String wordsFilename, String countsFilename) {
    wordCounts = new HashMap<>();
    totalCounts = new TimeSeries();
    
    // 1. 读 words file
    //    对每一行：解析出 word, year, count
    //    把 count 放到 wordCounts.get(word) 对应的 TimeSeries 里
    //    如果这个 word 第一次见，先 new 一个 TimeSeries
    
    // 2. 读 counts file
    //    对每一行：解析出 year, totalCount
    //    放到 totalCounts 里
}
```

**方法实现思路（超简单一旦数据结构选对了）：**

```java
public TimeSeries countHistory(String word) {
    // 直接从 wordCounts 里拿！
    // 如果 word 不存在，返回 new TimeSeries()
}

public TimeSeries weightHistory(String word) {
    // 拿到 countHistory(word)
    // 用 dividedBy(totalCountHistory()) 
    // 就是你 TimeSeries 里写的那个方法！
}
```

看到了吗？**选对数据结构，方法就是一两行的事！** 这就是数据结构课的精髓啊！✨

**⚠️ 读文件的注意事项：**

```java
// 用 In 类读文件（别用 readAllLines，太慢）
In in = new In(wordsFilename);
while (in.hasNextLine()) {
    String nextLine = in.readLine();
    String[] splitLine = nextLine.split("\t");  // words file 用 tab
    // splitLine[0] 是 word
    // splitLine[1] 是 year（要 Integer.parseInt）
    // splitLine[2] 是 count（要 Double.parseDouble）
}
```

counts file 用 `split(",")` 因为是逗号分隔。

---

### 第三关：`HistoryTextHandler`（占 10%）

```
难度：⭐⭐ | 核心概念：把数据变成文字输出
```

**它是什么？**

用户在浏览器点 **"History (Text)"** 按钮时，调用这个处理器，返回纯文字结果。

**输出格式（必须严格匹配）：**
```
cat: {2000=1.715E-5, 2001=1.612E-5, ...}
dog: {2000=3.127E-5, 2001=2.995E-5, ...}
```

**💡 实现思路：**

照着 `DummyHistoryTextHandler.java` 改！

```java
public class HistoryTextHandler extends NgordnetQueryHandler {
    private NGramMap map;  // 存起来！
    
    public HistoryTextHandler(NGramMap map) {
        this.map = map;
    }
    
    @Override
    public String handle(NgordnetQuery q) {
        List<String> words = q.words();
        int startYear = q.startYear();
        int endYear = q.endYear();
        
        StringBuilder response = new StringBuilder();
        for (String word : words) {
            TimeSeries ts = map.weightHistory(word, startYear, endYear);
            response.append(word + ": " + ts.toString() + "\n");
        }
        return response.toString();
    }
}
```

> `TimeSeries` 继承自 `TreeMap`，而 `TreeMap` 的 `toString()` 自动就是 `{key=value, key=value, ...}` 这种格式，完美！

然后去 `Main.java` 把：
```java
hns.register("historytext", new DummyHistoryTextHandler(ngm));
```
改成：
```java
hns.register("historytext", new HistoryTextHandler(ngm));
```

---

### 第四关：`HistoryHandler`（占 10%）

```
难度：⭐⭐ | 核心概念：调用绘图库
```

**它是什么？**

和上面类似，但这次返回的不是文字，而是一张**图片的 base64 编码字符串**。

**💡 实现思路：**

照着 `DummyHistoryHandler.java` 改！核心就是用 `Plotter` 类：

```java
public class HistoryHandler extends NgordnetQueryHandler {
    private NGramMap map;
    
    public HistoryHandler(NGramMap map) {
        this.map = map;
    }
    
    @Override
    public String handle(NgordnetQuery q) {
        List<String> words = q.words();
        int startYear = q.startYear();
        int endYear = q.endYear();
        
        ArrayList<TimeSeries> lts = new ArrayList<>();
        ArrayList<String> labels = new ArrayList<>();
        
        for (String word : words) {
            labels.add(word);
            lts.add(map.weightHistory(word, startYear, endYear));
        }
        
        XYChart chart = Plotter.generateTimeSeriesChart(labels, lts);
        return Plotter.encodeChartAsString(chart);
    }
}
```

同样去 `Main.java` 注册你的 handler。

---

## 🗺️ 总结：做题顺序路线图

```
Step 1: TimeSeries ← 先搞定基础数据结构
   ↓
Step 2: NGramMap ← 核心！解析文件 + 存数据
   ↓
Step 3: HistoryTextHandler ← 文字输出（照葫芦画瓢）
   ↓
Step 4: HistoryHandler ← 图片输出（照葫芦画瓢）
   ↓
🎉 完成！打开浏览器测试！
```

## 🔥 给你的忠告

1. **先写测试！** 用小文件（`very_short.csv`）测试，别一上来就用大文件
2. **TimeSeries 是地基**，它不对后面全崩。多写几个测试确认
3. **NGramMap 的核心在构造器**，把数据读对了存好了，后面的方法就是调 TimeSeries 的方法
4. **不要填零！不要填零！不要填零！** 重要的事情说三遍
5. 读文件用 `In` 类 + `hasNextLine()` + `readLine()`，别用 `readAllLines`

你完全可以搞定的！一步一步来，先让 TimeSeries 的测试全绿，再往下走。有什么具体代码问题随时来问我～ 💪✨