# 🌟 嘿！欢迎来到 CS 61B 的 Percolation 作业！

哈喽～我来带你拆解这道超经典的作业题！说实话这道题第一次看会觉得有点吓人，但其实理解了核心思想之后，就是一道 **Union-Find 的应用题** 而已啦～别慌，跟着我走！

---

## 📖 一、先搞懂：什么是 Percolation（渗透）？

想象你面前有一个 **N × N 的网格**，每个格子一开始都是 **blocked（封锁的）**。

然后你一个一个地随机打开格子（open）。

**核心问题是：** 水从最顶行往下流，只能流过 open 的格子，而且只能往 **上下左右** 四个方向流。当水能从 **顶行一路流到底行** 的时候，我们就说这个系统 **percolates（渗透了）**！

用个比喻：你可以想象这是一个咖啡滤纸 ☕ ——

> - **blocked 的格子** = 滤纸实心的部分，水过不去
> - **open 的格子** = 滤纸上的小孔
> - **full 的格子** = 水实际流到的那些小孔（从顶部连通的 open 格子）
> - **percolates** = 水从上面滴到了下面！

看这张图理解一下：

```
左边（渗透了）：               右边（没渗透）：
水从顶部一路连通到底部 ✅       水被挡住了，到不了底部 ❌
```

---

## 🧩 二、你要实现什么？

你需要完成 **`Percolation.java`** 这一个文件，实现以下方法：

```java
public class Percolation {
    public Percolation(int N)              // 创建 N×N 网格，全部 blocked
    public void open(int row, int col)     // 打开 (row, col) 这个格子
    public boolean isOpen(int row, int col) // (row, col) 是否已经 open？
    public boolean isFull(int row, int col) // (row, col) 是否是 full？（水能流到这里吗？）
    public int numberOfOpenSites()          // 当前有多少个 open 的格子
    public boolean percolates()             // 整个系统是否渗透了？
}
```

> ⚠️ 注意：`PercolationStats.java` 已经帮你写好了，不用管它！你只需要搞定 `Percolation.java`。

---

## 🧠 三、核心思路：用 Union-Find 来解决！

这里就是这道题最巧妙的地方了！你要把 **"水能不能从顶部流到某个格子"** 这个问题，转化成 **"这两个东西是不是在同一个集合里"** ——这就是 **Union-Find（并查集）** 能干的事！

### 🔑 关键思路

#### Step 1：把 2D 坐标映射到 1D

`WeightedQuickUnionUF` 是一个一维的数据结构，但我们的网格是二维的。所以你需要一个转换：

```
(row, col)  →  row * N + col
```

比如在一个 5×5 的网格里：
- (0, 0) → 0
- (0, 4) → 4
- (1, 0) → 5
- (2, 3) → 13

这个映射超级简单但超级重要！💡

#### Step 2：虚拟顶部节点 & 虚拟底部节点（Virtual Top & Virtual Bottom）

这是这道题 **最精髓** 的优化思想！！

如果你想判断 `percolates()`，暴力方法是：检查底行的每一个格子是不是 full 的——这太慢了！

**聪明的做法：**

```
        [Virtual Top]  ← 一个虚拟节点，连接顶行所有 open 的格子
        /   |    \
      (0,0)(0,1)(0,2) ...  ← 顶行
       ...  ...  ...
      (N-1,0)(N-1,1) ...   ← 底行
        \   |    /
       [Virtual Bottom] ← 一个虚拟节点，连接底行所有 open 的格子
```

- 创建 UF 的时候，大小是 **N*N + 2**（多出来的 2 就是虚拟顶和虚拟底）
- 虚拟顶的 index = `N * N`
- 虚拟底的 index = `N * N + 1`
- 当你 open 顶行的格子时，把它和 **虚拟顶** union
- 当你 open 底行的格子时，把它和 **虚拟底** union
- **`percolates()` 只需要检查虚拟顶和虚拟底是否 connected！** 一步搞定！🎉

#### Step 3：open() 的逻辑

当你打开 `(row, col)` 时：

1. 标记这个格子为 open（用一个 `boolean[][]` 或 `boolean[]` 数组记录）
2. 检查它的 **上下左右** 四个邻居：如果邻居也是 open 的，就把它们 **union** 在一起
3. 如果在顶行 → union 虚拟顶
4. 如果在底行 → union 虚拟底
5. openSites 计数 +1

伪代码长这样：

```
open(row, col):
    if already open → return
    mark as open
    openSites++
    
    if row == 0 → union(this site, virtualTop)
    if row == N-1 → union(this site, virtualBottom)
    
    for each neighbor (up, down, left, right):
        if neighbor is valid AND neighbor is open:
            union(this site, neighbor)
```

#### Step 4：isFull() 的逻辑

一个格子是 **full** 的条件 = 它是 open 的 **并且** 它和虚拟顶 **connected**。

```java
return isOpen(row, col) && uf.connected(xyTo1D(row, col), virtualTop);
```

简单粗暴！

#### Step 5：percolates() 的逻辑

```java
return uf.connected(virtualTop, virtualBottom);
```

一行搞定，是不是很优雅？✨

---

## ⚠️ 四、一个超级大坑：Backwash 问题！！

这是很多人踩的坑，我第一次做也被坑了 😤

**问题是什么？**

当系统 percolates 之后，虚拟顶和虚拟底已经 connected 了。这时候如果底行有一个 open 的格子，它虽然和虚拟底相连，但水其实从顶部流不到它那里——然而因为 **虚拟顶 ↔ 虚拟底 是连通的**，`isFull()` 会错误地返回 true！

```
举个例子（3×3 网格）：
    ■ □ ■         ■ = blocked, □ = open
    ■ □ ■         
    □ □ ■         

水从 (0,1) → (1,1) → (2,1) 渗透了 ✅
但是 (2,0) 虽然 open，水到不了那里
可是因为 (2,0) 连着虚拟底，虚拟底连着虚拟顶...
isFull(2, 0) 会错误返回 true ❌  ← 这就是 backwash！
```

**解决方案：用两个 Union-Find！**

- **UF1**：有虚拟顶 **和** 虚拟底 → 用来判断 `percolates()`
- **UF2**：只有虚拟顶，**没有** 虚拟底 → 用来判断 `isFull()`

每次 open 的时候，两个 UF 都要同步操作。这样 `isFull()` 用 UF2 来判断，就不会受到 backwash 的影响了！

```java
// percolates() 用 uf1（有顶有底）
public boolean percolates() {
    return uf1.connected(virtualTop, virtualBottom);
}

// isFull() 用 uf2（只有顶）
public boolean isFull(int row, int col) {
    return isOpen(row, col) && uf2.connected(xyTo1D(row, col), virtualTop);
}
```

---

## 🚨 五、Corner Cases（边界情况）

别忘了处理这些：

| 情况 | 处理方式 |
|------|---------|
| `N ≤ 0` | 构造函数抛 `IllegalArgumentException` |
| `row` 或 `col` 越界 | `open()`、`isOpen()`、`isFull()` 抛 `IndexOutOfBoundsException` |
| 坐标范围 | `0` 到 `N-1`，(0,0) 是左上角 |
| 重复 open 同一个格子 | 不要重复计数，直接 return |
| 检查邻居时 | 注意不要越出网格边界！ |

---

## 🏗️ 六、整体代码框架（提示，不是答案哦！）

```java
public class Percolation {
    private int N;
    private boolean[][] grid;         // 记录每个格子是否 open
    private int openSites;
    private WeightedQuickUnionUF uf1; // 有虚拟顶 + 虚拟底
    private WeightedQuickUnionUF uf2; // 只有虚拟顶
    private int virtualTop;
    private int virtualBottom;

    public Percolation(int N) {
        // 1. 验证 N > 0
        // 2. 初始化网格、UF
        // 3. 设置虚拟节点的 index
    }

    private int xyTo1D(int row, int col) {
        // 把 (row, col) 转成一维 index
    }

    private void validate(int row, int col) {
        // 检查越界，抛异常
    }

    public void open(int row, int col) {
        // 核心方法！打开格子，union 邻居
    }

    public boolean isOpen(int row, int col) {
        // 查 grid 数组
    }

    public boolean isFull(int row, int col) {
        // 用 uf2 检查是否和虚拟顶连通
    }

    public int numberOfOpenSites() {
        // 返回 openSites
    }

    public boolean percolates() {
        // 用 uf1 检查虚拟顶和虚拟底是否连通
    }
}
```

---

## 🧪 七、怎么测试？

1. **用 `InteractivePercolationVisualizer`**：点击格子，看看颜色变化对不对
   - 蓝色 = full（水流到了）
   - 白色 = open 但没水
   - 黑色 = blocked
   
2. **用 `PercolationPicture`**：跑 `inputFiles/input20.txt` 等测试文件，对比预期输出

3. **写你自己的单元测试** 在 `PercolationTest.java` 里！比如：
   - 1×1 网格，open 唯一格子，应该 percolates
   - 2×2 网格，测试各种情况
   - 测试 backwash 场景！

---

## 💪 八、做题顺序建议

1. ✅ 先写 `xyTo1D()` 和 `validate()`
2. ✅ 写构造函数，初始化一切
3. ✅ 写 `open()`（先不管虚拟节点，只做邻居 union）
4. ✅ 写 `isOpen()` 和 `numberOfOpenSites()`
5. ✅ 加入虚拟节点逻辑
6. ✅ 写 `isFull()` 和 `percolates()`
7. ✅ 处理 backwash（加第二个 UF）
8. ✅ 跑 visualizer 测试
9. ✅ 写单元测试

---

## 🎯 总结一句话

> **这道题的本质 = 把"水能不能从顶流到底"转化成 Union-Find 的连通性问题，再用虚拟节点优化查询效率。**

说真的，这道题做完你会对 Union-Find 的理解上升一个档次！它不只是课本上的抽象数据结构，而是真的能解决实际问题的超强工具 💪

加油！有什么不懂的随时问我～你一定可以搞定的！🌸