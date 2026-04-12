# 🎀 嘿！欢迎来到 CS 61B Project 1C 的世界！

哈喽～我来带你拆解这个作业！别怕，这个项目看起来长，其实逻辑超清晰的！我们一步步来～

---

## 📋 整体概览

这个项目叫 **Project 1C: Deque61B Enhancements**，就是给你之前在 1A 和 1B 写的双端队列（Deque）加点"超能力"✨。总共要做 **四件事**：

| 任务 | 占分 | 难度 |
|------|------|------|
| LinkedListDeque61B 的三个方法 | 20% | ⭐⭐ |
| ArrayDeque61B 的三个方法 | 20% | ⭐⭐ |
| MaxArrayDeque61B | 40% | ⭐⭐⭐ |
| GuitarString（吉他英雄！） | 20% | ⭐⭐⭐ |

---

## 🧩 第一部分：Object Methods（三个方法）

你需要在 `LinkedListDeque61B` 和 `ArrayDeque61B` 里都实现三个方法。先把你 1A 和 1B 的代码复制进来，记得保留 `package deque;`！

---

### 1️⃣ `iterator()` —— 让你的 Deque 可以被 for-each 遍历

**问题是什么？**

现在你的 Deque 不能这样用：

```java
for (String s : lld1) {
    System.out.println(s);
}
```

编译器会报错："foreach not applicable to type"，因为 Java 不知道怎么遍历你的 Deque。

**怎么解决？**

两步走：

**第一步**：修改 `Deque61B` 接口，让它继承 `Iterable<T>`：

```java
public interface Deque61B<T> extends Iterable<T> {
```

这就相当于跟 Java 说："嘿，我这个东西是可以遍历的！"

**第二步**：在两个实现类里写 `iterator()` 方法。

> 💡 **思路提示**（不直接给代码哦，你要自己写！）：
> 
> - 你需要创建一个**内部类**，实现 `Iterator<T>` 接口
> - 这个内部类需要：
>   - 一个实例变量来追踪"我现在遍历到哪了"（比如一个 `int pos`）
>   - `hasNext()` 方法：还有没有下一个元素？
>   - `next()` 方法：返回当前元素，然后往后移一位
> - `iterator()` 方法就是 `return new 你的内部类()`
> 
> ⚠️ **不能调用 `toList`！**

想象一下，迭代器就像一个"指针小人"🏃‍♀️，从第一个元素开始，一个一个往后走，走到末尾就停。

---

### 2️⃣ `equals()` —— 判断两个 Deque 是否"内容相等"

**问题是什么？**

默认的 `equals` 只比较内存地址（就是看两个变量是不是指向同一个对象）。但我们想要的是：**内容一样就算相等**！

```java
// 我们希望这个测试能通过！
assertThat(lld1).isEqualTo(lld2);
```

**怎么解决？**

> 💡 **思路提示**：
> 
> 你的 `equals` 方法应该按这个逻辑走：
> 
> ```
> 1. 如果 this == obj，直接返回 true（同一个对象肯定相等嘛）
> 2. 如果 obj 是 Deque61B<?> 的实例（用 instanceof 检查）：
>    a. 检查 size 是否相同，不同直接 false
>    b. 逐个元素比较，有不同的就 false
>    c. 都一样就 true
> 3. 否则返回 false
> ```

**几个超重要的注意点**⚠️：

- **不要用 `getClass`**，用 `instanceof`！
- 泛型类型要用 `?` 通配符：`obj instanceof Deque61B<?>` ✅，不是 `Deque61B<T>` ❌
- 方法签名必须是 `equals(Object obj)`，不是 `equals(Deque61B<T> other)`！
- 记得加 `@Override` 注解！这是安全网，写错了编译器会告诉你

**关键设计思想**：一个 `LinkedListDeque61B` 和一个 `ArrayDeque61B`，如果内容一样，应该 **相等**！所以你 instanceof 检查的是 `Deque61B<?>`，不是具体的实现类。这就是多态的魅力啊～ ✨

---

### 3️⃣ `toString()` —— 让打印输出好看

**问题是什么？**

```java
System.out.println(lld1);
// 输出: deque.LinkedListDeque61B@1a04f701  😱 丑死了
// 我们想要: [front, middle, back]  😍
```

**怎么解决？**

> 💡 **超级提示**：题目说了有**一行解法**！而且两个类的实现**完全一样**！
> 
> 再提示：Java 的 `List` 接口有个 `toString` 方法，输出格式刚好是 `[元素1, 元素2, 元素3]`……
> 
> 再再提示：你有一个 `toList()` 方法对吧？🤭

这个是最简单的，想到了就是一行代码的事！

---

## 🏆 第二部分：MaxArrayDeque61B

这个超酷！你要做一个能找到"最大元素"的 ArrayDeque！

**关键点：它继承自 ArrayDeque61B！**

> 题目特别说了：如果你发现自己在复制粘贴整个 ArrayDeque61B 的代码，那你就**做错了**！用继承（`extends`）！

```java
public class MaxArrayDeque61B<T> extends ArrayDeque61B<T> {
    // 不需要重写 Deque 的方法，继承来的就够了！
}
```

**你需要实现：**

| 方法 | 说明 |
|------|------|
| `MaxArrayDeque61B(Comparator<T> c)` | 构造函数，保存这个 Comparator |
| `T max()` | 用构造函数给的 Comparator 找最大值 |
| `T max(Comparator<T> c)` | 用传入的 Comparator 找最大值 |

> 💡 **思路提示**：
> 
> - 构造函数里，把那个 `Comparator` 存成实例变量
> - `max()` 方法就是遍历所有元素，用 Comparator 比较，找最大的那个
> - 空的就返回 `null`
> - `max(Comparator<T> c)` 同理，只是用参数里的 Comparator
> 
> **Comparator 怎么用？**
> ```java
> // c.compare(a, b) 
> // 返回正数：a > b
> // 返回负数：a < b  
> // 返回 0：a == b
> ```

**为什么需要 Comparator？**

因为同样的数据，"最大"的定义可能不同！比如字符串，你可以按字母顺序比，也可以按长度比。Comparator 让使用者自己决定怎么比较，超灵活～

---

## 🎸 第三部分：Guitar Hero（吉他英雄！）

这部分超好玩！你要用你的 Deque 来**模拟吉他弦的声音**！用的是 **Karplus-Strong 算法**。

### 算法原理（其实超简单）

想象你有一根吉他弦被拨动了：

1. **初始化**：创建一个 Deque，装满 0.0（长度 = 采样率 ÷ 频率）
2. **拨弦 (pluck)**：把 Deque 里所有值替换成 -0.5 到 0.5 之间的随机数
3. **推进一步 (tic)**：
   - 取出前端元素（`removeFirst`）
   - 取出新的前端元素（`get(0)`）
   - 把这两个数取平均，乘以 0.996（能量衰减）
   - 把结果加到 Deque 末尾（`addLast`）

```
视觉化：
Deque: [0.2, 0.4, 0.5, 0.3, -0.2, 0.4]
         ↑ 取出
新值 = (0.2 + 0.4) / 2 × 0.996 = 0.2988
结果: [0.4, 0.5, 0.3, -0.2, 0.4, 0.2988]
                                    ↑ 加到末尾
```

> 💡 **实现提示**：
> 
> - **构造函数**：capacity = (int)(SR / frequency)，SR 是采样率（题目里有常量）。用循环把 capacity 个 0.0 加进 Deque
> - **`pluck()`**：把所有元素替换成 `Math.random() - 0.5`
> - **`tic()`**：就是上面算法第 3 步
> - **`sample()`**：返回 Deque 前端的值（不移除！用 `get(0)`）
> - **⚠️ 不要在 GuitarString 里调用 `StdAudio.play`！**

### 为什么这能产生吉他声？

- **Deque 的长度**决定了音高（频率）
- **随机噪音**模拟拨弦的瞬间
- **取平均 + 衰减**相当于低通滤波器，高频会逐渐消失，低频保留
- 这就像真实吉他弦上的能量慢慢减弱一样！

物理 + CS = 🎵 太浪漫了！

---

## 🗺️ 建议的做题顺序

```
1. 复制 1A、1B 代码到 proj1c
2. 修改 Deque61B 接口（加 extends Iterable<T>）
3. 实现 iterator()（两个类都要）
4. 实现 toString()（一行搞定！）
5. 实现 equals()（稍微复杂点）
6. 写测试验证上面三个方法
7. 实现 MaxArrayDeque61B（用继承！）
8. 实现 GuitarString
9. 运行 GuitarHeroLite 听听你的吉他 🎸
```

---

## 🔥 最后的忠告

1. **写测试！写测试！写测试！** 虽然不交，但能帮你发现 bug
2. **不要 force push**！Git 出问题去 OH 或 Ed 求助
3. **检查 style**！会扣分的哦
4. 每写完一个方法就测一下，别堆到最后一起 debug（那真的会哭的 😭）

---

你有什么具体的部分想深入讨论的吗？比如 iterator 的内部类怎么写、equals 的细节、或者 Karplus-Strong 算法的某一步？随时问我呀！我超喜欢讲这些的～ 💫