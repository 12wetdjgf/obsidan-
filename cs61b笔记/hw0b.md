# 欢迎踏上 CS61B 的旅程！🎓

很好！你已经安装了 IntelliJ IDEA，这是第一步。让我们先搞定工具，再引导你完成作业。

---

## 第一部分：IntelliJ IDEA 基础入门

### 🔧 打开项目

1. 启动 IntelliJ IDEA
2. 选择 **Open**（不是 New Project）
3. 找到你从 GitHub 克隆下来的 `hw0b` 文件夹，点击打开

> 💡 如果你还没有克隆作业仓库，你需要先按照 [Assignment Workflow Guide](https://sp24.datastructur.es/resources/guides/assignment-workflow/) 的步骤来获取作业文件。

---

### 🖥️ IntelliJ 界面认识

```
┌─────────────────────────────────────────────┐
│  菜单栏 (File / Edit / View / Run...)        │
├──────────────┬──────────────────────────────┤
│              │                              │
│  项目文件树  │      代码编辑区              │
│  (左侧)      │      (中间/右侧)             │
│              │                              │
│  📁 src/     │   public class Point {       │
│    📄 *.java │       ...                    │
│  📁 tests/   │   }                          │
│              │                              │
├──────────────┴──────────────────────────────┤
│  终端 / 运行输出 / 错误信息 (底部)           │
└─────────────────────────────────────────────┘
```

### ▶️ 如何运行代码

- 找到含有 `main` 方法的类
- 看到左侧的 **绿色三角形 ▶️** 按钮
- 点击它，选择 **Run**
- 输出会显示在底部的 **Run** 面板

### 🧪 如何运行测试

- 打开 `tests/` 文件夹中的测试文件
- 同样点击绿色三角形运行
- ✅ 绿色 = 通过，❌ 红色 = 失败

### ⌨️ 常用快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+S` / `Cmd+S` | 保存文件 |
| `Ctrl+/` / `Cmd+/` | 注释/取消注释 |
| `Alt+Enter` | 快速修复（导入包等）|
| `Shift+F10` | 运行当前程序 |
| `Ctrl+Z` / `Cmd+Z` | 撤销 |

> 💡 **最重要的技巧**：当你使用 `ArrayList`、`TreeMap` 等类时，把鼠标悬停在红色波浪线上，IntelliJ 会提示你 **自动导入**，按 `Alt+Enter` 即可！

---

## 第二部分：作业解析 + 苏格拉底式引导 🏛️

作业 HW0B 共有 **4个任务**，我们逐一来看。

---

## 🏛️ Task 1: JavaExercises.java

### 方法一：`makeDice()`

> ：**"一个骰子有哪些面？它们是固定的还是可以增减的？"**

骰子的点数是 `[1, 2, 3, 4, 5, 6]`，共 6 个，**数量固定不变**。

那么，在 Java 中，存储**固定数量**元素，用什么数据结构？

```java
// 提示：回忆作业里的 Arrays 部分
// 如何初始化一个已知元素的数组？
int[] array = {?, ?, ?};
```

> 🤔 **思考**：`return` 一个数组，语法是什么？

---

### 方法二：`takeOrder(String customer)`

> **"你如何判断一个人是不是 'Ergun'？在 Java 中，比较字符串和比较数字有什么不同？"**

注意作业里有一个重要警告：

> ⚠️ **不要用 `==` 比较字符串！要用 `.equals()` 方法！**

```java
// 错误方式 ❌
if (customer == "Ergun") { ... }

// 正确方式 ✅
if (customer.equals("Ergun")) { ... }
```

> 🤔 **思考**：这个方法需要处理几种情况？如果都不匹配，返回什么？

---

### 方法三：`findMinMax(int[] array)`

> **"如果我给你一串数字，你怎么找到最大值和最小值？你会怎么一步步思考？"**

**算法思路引导**：

```
初始状态：假设第一个元素既是最大值也是最小值
然后：遍历数组中的每一个元素
  - 如果当前元素 > 当前最大值 → 更新最大值
  - 如果当前元素 < 当前最小值 → 更新最小值
最后：返回 最大值 - 最小值
```

> 🤔 **思考**：用 `for` 循环遍历数组，怎么写？返回值是什么类型？

---

### 方法四：`hailstone(int n)`

> 苏格拉底问你：**"什么是递归？递归必须有什么条件才能终止？"**

冰雹序列规则：
- 如果 `n` 是偶数 → 下一个是 `n / 2`
- 如果 `n` 是奇数 → 下一个是 `n * 3 + 1`
- 直到 `n == 1` 停止

```
例如 n=3：
3 → 10 → 5 → 16 → 8 → 4 → 2 → 1
```

**结构提示**：

```java
// 返回类型是 List<Integer>
public static List<Integer> hailstone(int n) {
    // 创建一个新的 ArrayList
    // 调用 hailstoneHelper(n, list)
    // 返回 list
}

// 辅助方法（递归）
private static void hailstoneHelper(int n, List<Integer> list) {
    // 先把 n 加入 list
    // 基础情况（base case）：n == 1 时停止
    // 递归情况：根据奇偶性，调用自身
}
```

> 🤔 **思考**：递归的 "base case" 是什么？

---

## 🏛️ Task 2: ListExercises.java

### `sum(List<Integer> L)`

> **"如何把一个列表里所有数字加起来？你需要一个什么样的'容器'来积累总和？"**

```java
// 伪代码思路：
int total = 0;
for (每一个元素 elem : L) {
    total += elem;
}
return total;
```

---

### `evens(List<Integer> L)`

> **"怎么判断一个数是偶数？你需要返回一个新的列表，如何创建它？"**

- 判断偶数：`n % 2 == 0`
- 创建新列表：`new ArrayList<>()`

---

### `common(List<Integer> L1, List<Integer> L2)`

> **"如何知道某个元素在另一个列表里存在？List 有什么方法可以帮你？"**

💡 提示：`list.contains(element)` 返回 `true` 或 `false`

---

### `countOccurrencesOfC(List<String> words, char c)`

> **"如何遍历一个字符串里的每一个字符？String 有什么方法？"**

💡 提示：
```java
String word = "hello";
for (int i = 0; i < word.length(); i++) {
    char ch = word.charAt(i);
    // 比较 ch 和 c
}
```

---

## 🏛️ Task 3: MapExercises.java

### `letterToNum()`

> **"'a' 对应 1，'b' 对应 2... 如果我告诉你字符可以做加减法运算，你能算出每个字母对应的数字吗？"**

💡 关键提示：
```java
char letter = 'a';
int num = letter - 'a' + 1; // 'a'-'a'+1 = 1, 'b'-'a'+1 = 2 ...

// 遍历所有小写字母：
for (char c = 'a'; c <= 'z'; c++) { ... }
```

---

### `squares(List<Integer> nums)`

> **"Map 的 put 方法需要什么参数？key 和 value 分别是什么？"**

```java
map.put(key, value);
// 这里 key = 数字本身，value = 数字的平方
```

---

### `countWords(List<String> words)`

> **"如果一个单词已经在 map 里了，你应该怎么更新它的计数？如果还没有呢？"**

```java
// 伪代码：
if (map中已有这个word) {
    map.put(word, 原来的值 + 1);
} else {
    map.put(word, 1);
}
// 提示：map.getOrDefault(key, defaultValue) 很有用！
```

---

## 🏛️ Task 4: Dessert.java（从零创建）

>**"实例变量和静态变量有什么区别？如果我说 '静态变量被所有对象共享'，这对 numDesserts 意味着什么？"**

**创建步骤**：
1. 右键 `src/` 文件夹 → New → Java Class → 命名 `Dessert`
2. 实现以下结构：

```java
public class Dessert {
    // 实例变量（每个对象独有）
    int flavor;
    int price;
    
    // 静态变量（所有对象共享）⭐
    static int numDesserts = 0;
    
    // 构造器
    public Dessert(int flavor, int price) {
        // 设置实例变量
        // 别忘了让 numDesserts 增加！
    }
    
    // 打印方法：格式是 "flavor price numDesserts"
    public void printDessert() { ... }
    
    // main 方法
    public static void main(String[] args) {
        System.out.println("I love dessert!");
    }
}
```

> 🤔 **最终思考**：`numDesserts++` 应该放在哪里，才能保证每次创建新 Dessert 对象时都会增加？

---

## 📋 总结：你的行动清单

- [x] 在 IntelliJ 中打开 hw0b 项目
- [x] 尝试运行已有的测试，看看初始状态
- [x] 完成 `JavaExercises.java` 的4个方法
- [x] 完成 `ListExercises.java` 的4个方法  
- [x] 完成 `MapExercises.java` 的3个方法
- [x] 创建 `Dessert.java` 并实现所有要求
- [x] 运行所有测试，确保通过

---
