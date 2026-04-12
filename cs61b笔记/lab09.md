# 🎀 嗨！欢迎来到 CS 61B Lab 09 大冒险！

哈喽哈喽～我来帮你拆解这个 Lab！别慌，虽然看起来很长，但其实逻辑超级清晰的！我们一步一步来～

---

## 📌 这个 Lab 到底在干嘛？

这个 Lab 分**两大部分**：

| 部分 | 内容 | 目的 |
|------|------|------|
| Part I | 认识瓦片渲染引擎 (Tile Rendering Engine) | 学会怎么在屏幕上画东西 |
| Part II | 实现**康威生命游戏** (Conway's Game of Life) | 练习二维数组操作 + 文件读写 |

你真正需要**写代码**的只有 **3 个方法**：
1. `nextGeneration` — 生命游戏的核心逻辑
2. `saveBoard` — 把棋盘保存到文件
3. `loadBoard` — 从文件加载棋盘

---

## 🧩 Part I：先搞懂渲染引擎（不用写代码，但必须看懂！）

这部分其实就是让你**读代码**，理解三个核心类怎么用：

### 核心概念速览

```
TERenderer  → 渲染器，负责把东西画到屏幕上
TETile      → 单个瓦片（一个格子）
Tileset     → 预定义的瓦片集合（墙、草、水、空...）
```

### 渲染流程就三步，超简单：

```java
// 第1步：初始化渲染器
TERenderer ter = new TERenderer();
ter.initialize(WIDTH, HEIGHT);

// 第2步：创建二维数组并填充
TETile[][] world = new TETile[WIDTH][HEIGHT];
for (int x = 0; x < WIDTH; x++) {
    for (int y = 0; y < HEIGHT; y++) {
        world[x][y] = Tileset.NOTHING;  // 必须初始化！不然会 NullPointerException
    }
}

// 第3步：渲染！
ter.renderFrame(world);
```

> ⚠️ **超级重要的坐标系**：`(0, 0)` 是**左下角**！不是左上角！往右是 x 增大，往上是 y 增大。记住这个，后面会用到！

### 关于 Random（伪随机数）

```java
Random r = new Random(12345);  // 12345 是种子(seed)
r.nextInt();  // 生成一个"随机"整数
```

**关键点**：**相同的种子 → 相同的序列**！这就是"确定性随机"，Project 3 要靠这个来重现世界。

---

## 🎮 Part II：康威生命游戏 — 重头戏来了！

### 生命游戏是啥？

想象一个无限大的棋盘，每个格子要么**活着** (`Tileset.CELL`)，要么**死了** (`Tileset.NOTHING`)。每一个时间步，所有格子**同时**根据规则更新状态。

---

## ✏️ 任务 1：`nextGeneration`

### 规则（背下来！只有4条）：

| # | 条件 | 结果 |
|---|------|------|
| 1 | 活细胞，邻居 **< 2** 个活的 | **死**（孤独死 😢） |
| 2 | 活细胞，邻居 **2 或 3** 个活的 | **继续活**（刚刚好 😊） |
| 3 | 活细胞，邻居 **> 3** 个活的 | **死**（太挤了 😵） |
| 4 | 死细胞，邻居**恰好 3** 个活的 | **复活**（繁殖 🌱） |

### 每个细胞有 8 个邻居：

```
[左上] [上] [右上]
[左]   [我] [右]
[左下] [下] [右下]
```

### 思路引导（我不直接给你代码哦，你要自己写！）：

**Step 1：写一个辅助方法，数某个格子周围有多少活邻居**

```
想想看：对于位置 (x, y)，它的8个邻居坐标是什么？
→ (x-1,y-1), (x,y-1), (x+1,y-1)
   (x-1,y),            (x+1,y)
   (x-1,y+1), (x,y+1), (x+1,y+1)
```

> 💡 **小心边界！** 如果 x=0，那 x-1 就越界了。题目说超出边界的都当作**死细胞**处理，所以你需要做**边界检查**。

**Step 2：遍历每个格子，根据规则决定 `newGen[x][y]` 是什么**

```
伪代码：
for 每个位置 (x, y):
    count = 数活邻居数量
    如果当前格子是活的:
        如果 count == 2 或 count == 3:
            newGen[x][y] = CELL  (活)
        否则:
            newGen[x][y] = NOTHING  (死)
    如果当前格子是死的:
        如果 count == 3:
            newGen[x][y] = CELL  (复活！)
        否则:
            newGen[x][y] = NOTHING  (还是死的)
```

> 🌟 **关键陷阱**：`newGen` 初始已经全是 `NOTHING` 了！所以你其实只需要管"什么时候放 `CELL`"就好了，可以简化逻辑～

---

## ✏️ 任务 2：`saveBoard`

### 要把棋盘保存成这种格式：

```
宽度 高度
000110...
001100...
...
```

### 思路引导：

**Step 1：** 先写第一行 —— `width + " " + height + "\n"`

**Step 2：** 逐行写棋盘内容

这里有个**超级容易踩的坑**！！！注意：

```
文件里从上往下写，但 (0,0) 在左下角！
```

所以你写行的时候，**y 要从大到小遍历**！

```
想象一下：
y=4  → 这是文件的第一行（最上面）
y=3  → 第二行
y=2  → 第三行
y=1  → 第四行  
y=0  → 最后一行（最下面）
```

对于每一行 y，遍历 x 从 0 到 width-1：
- 如果 `currentState[x][y]` 是 `CELL` → 写 `"1"`
- 如果是 `NOTHING` → 写 `"0"`

每行末尾加 `"\n"`。

**Step 3：** 用 `FileUtils` 的方法把字符串写入文件 `"src/save.txt"`。

> 💡 **提示**：可以用 `StringBuilder` 来拼接字符串，最后一次性写入文件，又快又优雅～

---

## ✏️ 任务 3：`loadBoard`

### 从文件读取棋盘并返回 `TETile[][]`

### 思路引导：

**Step 1：** 用 `FileUtils` 读取文件内容为字符串

**Step 2：** 用 `split("\n")` 按行分割

**Step 3：** 第一行解析宽度和高度

```java
String[] dimensions = lines[0].split(" ");
width = Integer.parseInt(dimensions[0]);  // 别忘了！题目要求你初始化实例变量
height = Integer.parseInt(dimensions[1]);
```

**Step 4：** 创建 `TETile[width][height]` 数组

**Step 5：** 逐行读取棋盘数据

跟 `saveBoard` 一样的道理，**文件的第一行对应 y 最大值**：

```
lines[1] → y = height - 1
lines[2] → y = height - 2
...
lines[height] → y = 0
```

对于每行的每个字符：
- `'1'` → `Tileset.CELL`
- `'0'` → `Tileset.NOTHING`

> 💡 用 `lines[i].charAt(j)` 来获取第 j 个字符！

---

## 🧪 测试

运行提供的本地测试来验证。记住：**本地测试全过 ≠ autograder 满分**，所以自己也要多想想边界情况！

如果想可视化运行生命游戏：
```
Run → Edit Configurations → Program arguments: -l patterns/hammerhead.txt
```

---

## 🎯 做题顺序建议

```
1️⃣ 先做 nextGeneration（最核心，理解了规则就不难）
2️⃣ 再做 saveBoard（格式化输出）
3️⃣ 最后做 loadBoard（saveBoard 的逆过程）
4️⃣ 跑测试，debug～
```

---

## 💬 最后的小 Tips

- **一定要先读懂已有代码！** 特别是构造函数和变量
- 写 `nextGeneration` 的时候，**不要直接修改 `tiles`！** 要把结果放到 `newGen` 里。因为如果你边算边改原数组，后面格子的邻居计数就会出错（这是经典坑！）
- `TETile` 是**不可变的**，不能直接改属性
- 善用辅助方法，不要一个方法写100行

加油呀！有具体哪一步卡住了随时来问我～这个 Lab 做完之后你对 Project 3 就有底了！✨