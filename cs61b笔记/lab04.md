# 🌟 嘿！欢迎来到 Git 的奇妙世界！让我来带你飞～

哈喽！我是你的Git小导师！说实话，Git刚开始学的时候我也觉得好迷糊，但搞懂之后就觉得——天哪这也太好用了吧！来来来，我带你把这个Lab 04拆解得明明白白的！✨

---

## 📚 首先，搞清楚大背景

这个Lab的核心就是：**学会用Git管理代码**。整个Lab分成几个部分：

| 部分 | 内容 |
|------|------|
| 概念讲解 | Git vs GitHub、基础命令 |
| Exercise 1.1 | 自己建仓库，练习基础操作 |
| Exercise 1.2 | 回答Beacon上的表单问题 |
| Exercise 2 | 克隆一个仓库，用Git技能找密码（超酷的寻宝游戏！） |

最终你要把4个密码填到 `magic_word.txt` 里提交。

---

## 🧠 核心概念速通

### Git 和 GitHub 的区别

这个超多人搞混的！简单说：

- **Git** = 一个装在你电脑上的**版本控制工具**（就像游戏里的存档系统！每次commit就是存一次档）
- **GitHub** = 一个**网站/云端服务器**，用来存放你的Git仓库（就像把游戏存档上传到云端）

> 💡 类比：Git是你手机里的相机app，GitHub是Google Photos云相册。一个负责拍照（记录），一个负责云端存储。

### Git 的数据模型

Git的提交历史本质上像一个**链表**（linked list）！每个commit指向前一个commit：

```
[commit 1] <--- [commit 2] <--- [commit 3] (最新)
```

这样你就能回到任何一个历史版本——是不是像时间旅行！⏰

---

## 🔧 命令详解（这些你得刻进DNA里）

### 1. `git init` — 创建仓库

```bash
git init
```
在一个文件夹里运行，它就变成了一个Git仓库。就像在一个房间里装了监控摄像头，开始记录一切变化。

### 2. `git add` — 选择要保存的东西

```bash
git add some_file.txt   # 添加单个文件
git add .               # 添加所有改动
```

这一步是把文件放到 **暂存区（staging area）**。可以理解为：你在打包行李箱，先把要带的东西放进去，但还没有锁上箱子。

### 3. `git commit -m "消息"` — 真正保存！

```bash
git commit -m "描述你做了什么改动"
```

这才是真正的"存档"！`-m` 后面的消息一定要写得有意义，未来的你会感谢现在的你的！

> ⚠️ **重要流程：** `add`（选文件）→ `commit`（存档）。缺一不可！

### 4. `git status` — 看看当前状态

```bash
git status
```

告诉你：
- 哪些文件改了但还没 `add`（Changes not staged for commit）
- 哪些文件已经 `add` 了等着 `commit`（Changes to be committed）
- 哪些文件是全新的还没被追踪（Untracked files）

### 5. `git log` — 查看历史

```bash
git log
```

显示所有commit的历史记录。每个commit都有一个超长的**commit ID**（一串字母数字），这个ID在`restore`的时候超级有用！

按 `q` 退出日志界面哦！

### 6. `git restore` — 时间旅行！

```bash
git restore [文件名]                          # 恢复到最近一次commit的版本
git restore --source=[commitID] [文件名]       # 恢复到指定commit的版本
```

---

## 🎯 Exercise 1.1 详解（手把手教你思路）

好了重头戏来了！这个练习要你自己建一个仓库做一系列操作。我**不会直接给你命令**（你说了要独立完成嘛！），但我会告诉你每一步该怎么想：

### Step 1：创建目录 `lab04-checkoff`

> 💭 想想：怎么在终端创建文件夹？提示：`mkdir` 命令。**不要在你的 sp24-s*** 仓库里面创建！**

### Step 2：进入目录，初始化Git仓库

> 💭 进目录用什么？`cd`！初始化用什么？上面刚讲过的 `git init`！

### Step 3：创建 `61b.txt`，写入 "Created 61b.txt"

> 💭 创建文件的方法很多。可以用 `echo "Created 61b.txt" > 61b.txt`，也可以用文本编辑器。

### Step 4：创建 `61boba.txt`，写入 "Created 61boba.txt"

> 💭 跟Step 3一样的方法！

### Step 5：只追踪 `61b.txt`，commit

> 💭 关键词是 **"只"**！所以你不能用 `git add .`（那会把两个文件都加进去）。你应该只 `add` 那一个文件，然后commit，消息是 `Add 61b.txt`。

### Step 6：修改 `61b.txt`

> 💭 把内容改成 "61b.txt changed to version 2"。用 `echo` 配合 `>` 重定向，或者用编辑器。

### Step 7：两个文件一起commit

> 💭 这次要 `add` 两个文件（或者用 `git add .`），然后commit，消息是 `Updated 61b.txt and added 61boba.txt`。

### Step 8：再次修改 `61b.txt`，但不要commit！

> 💭 改成 "61b.txt changed to final version"。然后**停下来**！不要add，不要commit！用 `git status` 和 `git log` 看看当前状态。

### Step 9：用git恢复到最近commit的版本

> 💭 用哪个命令能把文件恢复到最近一次commit？`git restore`！不需要指定commit ID就行了～

### Step 10：用git恢复到第一次commit的版本

> 💭 这次你需要：
> 1. 先用 `git log` 找到第一次commit的 **commit ID**
> 2. 然后用 `git restore --source=[那个ID] 61b.txt`

---

## 🔍 Exercise 2 详解（Git寻宝游戏！）

先运行这个命令克隆仓库（**确保不在你的sp24-s\*\*\*里面**）：

```bash
curl -sS https://sp24.datastructur.es/labs/lab04/lab04.sh | bash
```

然后 `cd git-exercise-sp24`。

### Part 2.1 — 在历史中找密码

打开 `password.txt`，发现里面没密码？别慌！

> 💭 **思路：** 密码在某个**之前的commit**里！你该怎么做？
> 1. 用 `git log` 查看所有commit历史（用方向键滚动，`q`退出）
> 2. 找到那个看起来有密码的commit
> 3. 用 `git restore --source=[commitID] password.txt` 恢复它
> 4. 打开文件看密码！
> 5. **记得之后恢复回最新版本：** `git restore password.txt`

### Part 2.2 — 在另一个分支找密码

> 💭 **思路：** 密码藏在 `erik` 分支里！
> 1. 用 `git switch erik` 切换到那个分支
> 2. 看看 `password.txt` 里的内容
> 3. 验证commit消息是否提示这是密码
> 4. 记下密码，然后 **`git switch main` 切回主分支！**

### Part 2.3 — 从远程仓库拉取密码

> 💭 **思路：**
> 1. 添加远程仓库：`git remote add [你起个名字] git@github.com:Berkeley-CS61B/git-exercise-remote.git`
> 2. 拉取：`git pull [你起的名字] main --allow-unrelated-histories --no-rebase`
> 3. 可能会弹出合并界面，完成合并后密码就出现了！

---

## 📝 最终提交

把4个密码按顺序写进 `magic_word.txt`：

```
第1个密码（Beacon表单得到的）
第2个密码（Part 2.1）
第3个密码（Part 2.2）
第4个密码（Part 2.3）
```

每行一个，结尾不要有多余空格！然后提交到Gradescope～

---

## 💡 我的小Tips

1. **commit之前永远先 `git status`**，确认你add了该add的东西
2. **commit消息认真写**，别写 "aaa" 或 "asdfg"，未来的你会看不懂的😂
3. **遇到问题别慌**，`git status` 是你最好的朋友，它会告诉你当前发生了什么
4. **永远不要在不确定的情况下复制网上的git命令**，这个Lab也特别强调了这一点！

---

加油呀！Git一开始确实有点抽象，但一旦你理解了 `add → commit → push` 这个流程，就会觉得很自然了。有什么不懂的随时问我！你可以的！💪✨