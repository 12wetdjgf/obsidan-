# 🎮 Lab 10: Tetris 全解析！

嘿嘿～欢迎来到 CS 61B 的 Tetris 实验！我是你的小导师～让我用最通俗的方式带你搞懂这个 lab 到底要干嘛，然后你自己动手把它写出来，okay？💪

---

## 🌟 首先，搞懂大局观！

这个 lab 本质上就是让你**写一个俄罗斯方块游戏**！听起来很酷对吧？但别慌，他们已经帮你写了一大堆辅助代码了，你只需要填 **5 个方法**：

| 方法 | 干嘛用的 |
|------|----------|
| `updateBoard` | 读取键盘输入，移动/旋转方块 |
| `incrementScore` | 根据消除行数加分 |
| `clearLines` | 检查并清除填满的行 |
| `runGame` | 主游戏循环（把上面的都串起来） |
| `renderScore` | 在屏幕上显示分数 |

游戏的整体流程是这样的：

```
生成方块 → 玩家操作 → 方块落地 → 检查消行 → 加分 → 生成新方块 → 重复...直到游戏结束
```

---

## 📂 先读懂已有的文件！

这一步**超级重要**，很多人直接跳过然后卡半天 😤

你有这些文件：

- **`Tetris.java`** — 你要写代码的地方
- **`Tetromino.java`** — 定义了方块的形状（T、L、S、Z 那些形状）
- **`Movement.java`** — 已经帮你写好了移动和旋转的逻辑！！
- **`BagRandomizer.java`** — 随机生成方块的工具
- **`StdDraw`** — 一个绘图+输入的库

> ⚠️ **关键提醒**：坐标 `(0, 0)` 是**左下角**，不是左上角哦！

---

## 🔧 逐个方法攻破！

### 1️⃣ `updateBoard` — 处理键盘输入

**目标**：读取玩家按了什么键，然后执行对应的操作。

**思路引导**：

首先，你需要用 `StdDraw` 的两个方法：
- `StdDraw.hasNextKeyTyped()` — 检查玩家有没有按键
- `StdDraw.nextKeyTyped()` — 获取玩家按的是哪个键

然后根据按键执行不同操作：

```
按键映射：
  'a' → 左移一格
  's' → 下移一格  
  'd' → 右移一格
  'q' → 左旋转 90°
  'w' → 右旋转 90°
```

**💡 提示**：
- 你**不需要自己写移动逻辑**！去 `Movement.java` 里看看有什么现成的方法可以调用
- 这个类里已经有一个 `Movement` 实例给你用了
- 想想用什么结构处理不同的按键？**`if-else` 或 `switch`** 就行啦

**伪代码给你参考**：
```
如果有键盘输入:
    获取输入的字符
    如果是 'a':
        调用Movement的左移方法
    如果是 's':
        调用Movement的下移方法
    如果是 'd':
        调用Movement的右移方法
    如果是 'q':
        调用Movement的左旋方法
    如果是 'w':
        调用Movement的右旋方法
    // 其他键？什么都不做就对了！
```

---

### 2️⃣ `incrementScore` — 加分逻辑

**目标**：根据一次消除了多少行来加分。

这个是最简单的！规则如下：

| 消除行数 | 加分 |
|----------|------|
| 1 行 | +100 |
| 2 行 | +300 |
| 3 行 | +500 |
| 4 行 | +800 |

**💡 提示**：
- 这就是一个简单的条件判断
- 用 `if-else` 或 `switch` 都可以
- 注意：参数是消除的行数，你要把对应的分数加到 score 变量上

**伪代码**：
```
如果 linesCleared == 1: score += 100
如果 linesCleared == 2: score += 300
如果 linesCleared == 3: score += 500
如果 linesCleared == 4: score += 800
```

简单到哭对不对？😂

---

### 3️⃣ `clearLines` — 消除行（这个有点难度哦！）

**目标**：检查整个棋盘，找到被填满的行，消除它们，并让上面的行下落。

这个是这个 lab 里**最有思考量的方法**了，来，深呼吸～

**思路引导**：

**Step 1：遍历每一行，判断是否填满**

```
对于每一行 (从下往上遍历):
    检查这一行的每个格子是不是都有方块
    如果有一个格子是空的 → 这行没满，跳过
    如果全满了 → 需要消除！
```

**Step 2：消除一行后，上面的行要下移**

想象一下你在书架上抽掉了一本书，上面的书都会往下掉一格，对吧？就是这个道理！

```
当发现第 i 行填满了:
    把第 i+1 行的内容复制到第 i 行
    把第 i+2 行的内容复制到第 i+1 行
    ...一直到最顶行
    最顶行清空
    linesCleared++
```

**Step 3：用你写好的 `incrementScore` 更新分数**

**💡 关键提示**：
- ⚠️ 一定要用参数 `tiles` 来操作棋盘，不要用其他变量（autograder 会检查）
- 消除一行后，**当前行需要重新检查**！因为上面掉下来的可能也是满的
- 记得跟踪消除了多少行

**伪代码**：
```
linesCleared = 0

对于 row 从 0 到 board高度:
    isFull = true
    对于 col 从 0 到 board宽度:
        如果 tiles[col][row] 是空的:
            isFull = false
            break
    
    如果 isFull:
        // 把上面所有行往下移一行
        对于 r 从 row 到 board高度-2:
            对于 c 从 0 到 board宽度:
                tiles[c][r] = tiles[c][r+1]
        // 清空最顶行
        对于 c 从 0 到 board宽度:
            tiles[c][最顶行] = 空
        
        linesCleared++
        row--  // ⚡ 重新检查当前行！因为新的行掉下来了

调用 incrementScore(linesCleared)
```

---

### 4️⃣ `runGame` — 游戏主循环

**目标**：把所有东西串起来，让游戏跑起来！

**思路引导**：

游戏的核心就是一个**循环**，不停地执行，直到游戏结束：

```
while (游戏没有结束):
    如果当前没有方块 (currentTetromino == null):
        生成新方块 (spawnPiece)
        检查游戏是否结束 (isGameOver)
    
    更新棋盘 (updateBoard) — 处理玩家输入
    渲染棋盘 (renderBoard) — 画出来
    
    如果当前方块变成了 null (说明已经落地了):
        消除行 (clearLines)
        渲染棋盘 (renderBoard) — 消除后重新画
```

**💡 提示**：
- 用 `while` 循环让游戏持续运行
- 方块落地后会被自动设为 `null`，你不需要管这个逻辑
- 方块变成 `null` 后要先 `clearLines`，再生成新方块
- 别忘了传参！`clearLines` 需要传入棋盘

**伪代码**：

```java
while (!isGameOver()) {
    if (currentTetromino == null) {
        spawnPiece();  // 生成新方块
    }
    
    updateBoard();     // 处理输入
    renderBoard();     // 渲染画面
    
    if (currentTetromino == null) {
        clearLines(board);  // 检查消行
        renderBoard();      // 重新渲染
    }
}
```

> 注意：这里的逻辑顺序可能需要根据你读代码后的理解微调！关键是理解：**先处理输入 → 再渲染 → 方块落地后消行 → 循环**

---

### 5️⃣ `renderScore` — 显示分数

**目标**：在屏幕上画出当前分数。

**步骤**：

1. 设置文字颜色为白色：`StdDraw.setPenColor(new Color(255, 255, 255))`
2. 在位置 `(7, 19)` 画出分数文字：`StdDraw.text(7, 19, "Score: " + score)`
3. 调用 `StdDraw.show()` 渲染出来

**💡 提示**：
- 你可能需要 `import java.awt.Color`
- 分数转成字符串可以用 `"Score: " + score` 或者 `String.valueOf(score)`
- 别忘了 `show()`！不然画了也看不见

---

## 🧠 总结一些关键思维

| 概念 | 为什么重要 |
|------|-----------|
| **读辅助代码** | Movement.java 里的方法直接拿来用，别自己重写！ |
| **StdDraw API** | 处理输入和渲染的核心，Project 3 也会用到 |
| **分解问题** | 大问题拆成小方法，每个方法干一件事 |
| **游戏循环** | `while` 循环是所有游戏的核心模式 |

---

## 🚀 做题顺序建议

```
incrementScore (最简单，热热身)
    ↓
updateBoard (熟悉 StdDraw 和 Movement)
    ↓
clearLines (核心逻辑，需要仔细想)
    ↓
runGame (串联一切)
    ↓
renderScore (收尾！)
```

---

好啦！我给你的都是思路和伪代码，具体的 Java 代码就交给你自己写啦！毕竟你说了要**独立完成**嘛～遇到 bug 别怕，`System.out.println` 是你最好的朋友（debug 大法好）！

有什么不懂的随时问我哦！加油加油，你一定可以的！🎉✨