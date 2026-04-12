# 🌟 嘿！欢迎来到 CS 61B 的世界！

哈喽哈喽～我是你的非官方小导师！你居然要学 CS 61B 了，太酷了吧！这门课超级有意思的，虽然 Lab 01 看起来巨长巨吓人，但其实就是——**装软件**。对，没骗你，第一个 lab 的核心就是把环境搭好。别慌，我带你一步步拆解！🎀

---

## 📋 Lab 01 到底要你做什么？

我帮你把这个 lab 拆成 **8 个大任务**，你一个一个来就好：

| 步骤 | 任务 | 难度 |
|------|------|------|
| 1 | 安装 Git | ⭐ |
| 2 | 熟悉终端(Terminal) | ⭐ |
| 3 | 注册 GitHub + Beacon 账号 | ⭐ |
| 4 | 配置 Git | ⭐⭐ |
| 5 | 克隆 Java 库 + 配置个人仓库 | ⭐⭐⭐ |
| 6 | 安装 IntelliJ + 插件 + JDK | ⭐⭐ |
| 7 | **修 bug（唯一要写代码的部分！）** | ⭐⭐ |
| 8 | 用 Git 提交 + Gradescope 交作业 | ⭐⭐ |

看到没？**只有第 7 步需要写代码**，其余全是配置环境。所以深呼吸～

---

## 🔧 Step 1：安装 Git

Git 是一个**版本控制工具**，简单说就是——它能帮你保存代码的每一个"快照"。你改坏了代码？没事，回滚！就像游戏里的存档点。

- **Windows**：去他们给的链接装 Git for Windows
- **Mac**：终端里输入 `git --version`，如果没装过系统会提示你装
- **Linux**：`sudo apt-get install git`（大概率你已经有了）

装完之后打开终端输入：
```bash
git --version
```
如果出来版本号，恭喜，过关！✨

---

## 💻 Step 2：熟悉终端 (Terminal)

终端就是那个黑乎乎（或者白乎乎）的命令行窗口。你需要会这几个**核心命令**：

```bash
cd <目录名>      # 进入某个文件夹（Change Directory）
cd ..            # 返回上一级
ls               # 列出当前文件夹里的东西（Mac/Linux）
dir              # 列出当前文件夹里的东西（Windows）
mkdir <名字>     # 创建新文件夹
pwd              # 显示你现在在哪（Print Working Directory）
```

> 💡 **小天才提示**：你可以把终端想象成"没有图形界面的文件管理器"。你用 `cd` 就像双击文件夹，用 `ls` 就像打开文件夹看里面有啥。

记住这几个命令，后面全程要用！

---

## 🔑 Step 3：GitHub + Beacon 账号

1. 去 [github.com](https://github.com) 注册一个账号（如果有了就跳过）
2. 去 Beacon（CS 61B 自己的成绩系统）完成注册
3. 你会收到一封邮件，邀请你加入你的**课程私人仓库**（类似 `sp24-s123`）

> ⚠️ **注意**：邮件会发到你注册 GitHub 用的邮箱，不一定是学校邮箱哦！

---

## ⚙️ Step 4：配置 Git

打开终端，依次输入这些命令：

```bash
# 设置你的名字和邮箱（Git 会用这个标记你的提交）
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"

# 设置默认分支名为 main
git config --global init.defaultBranch main

# 设置合并策略
git config --global pull.rebase false
```

然后把默认编辑器改成 `nano`（因为默认的 `vim` 对新手来说简直是噩梦级别的存在😂）：

按照课程给的链接去配置编辑器就好。

---

## 📦 Step 5：克隆仓库（这步最关键！）

这步有**三个小任务**，我帮你理清楚：

### 5a. 下载课程 Java 库

先创建一个工作文件夹：
```bash
mkdir cs61b
cd cs61b
```

然后克隆库：
```bash
git clone https://github.com/Berkeley-CS61B/library-sp24
```

用 `ls library-sp24` 看看里面有没有 `.jar` 文件，有的话就对了！

### 5b. SSH 认证

这步是让你的电脑和 GitHub "握手"，这样以后 push 代码就不用每次输密码了。

```bash
# 生成 SSH 密钥
curl -sS https://sp24.datastructur.es/labs/lab01/get-ssh-key.sh | bash

# 查看你的公钥（注意 .pub 后缀！）
cat <路径>.pub
```

把输出的那一长串东西复制下来，去 GitHub → Settings → SSH and GPG Keys → New SSH Key，粘贴进去。

然后测试：
```bash
ssh -T git@github.com
```

看到 `Hi USERNAME! You've successfully authenticated` 就成功了！🎉

### 5c. 克隆你的个人仓库

```bash
# 把 *** 换成你的仓库编号！
git clone git@github.com:Berkeley-CS61B-Student/sp24-s***.git
cd sp24-s***
git branch -M main

# 添加 skeleton（骨架代码）远程仓库
git remote add skeleton https://github.com/Berkeley-CS61B/skeleton-sp24.git

# 拉取 Lab 01 的代码
git pull skeleton main
```

> 💡 **小天才提示**：你的文件夹结构应该长这样：
> ```
> cs61b/
> ├── library-sp24/     ← Java 库
> └── sp24-s***/        ← 你的个人仓库
>     └── lab01/
>         ├── src/Arithmetic.java
>         └── tests/ArithmeticTest.java
> ```
> **千万别把个人仓库放在 library-sp24 里面！** 它们是并列关系！

---

## 🖥️ Step 6：安装 IntelliJ

1. 去 JetBrains 官网下载 **Community Edition**（免费版，往下滚才能看到，别下成 Ultimate 了！）
2. 安装好之后打开，装两个插件：
   - **CS 61B** 插件
   - **Java Visualizer** 插件
3. 重启 IntelliJ
4. 安装 JDK：File → Project Structure → 选一个 **17 或更高版本**的 JDK

然后按照课程的 Assignment Workflow 指南打开 `lab01` 项目，记得配置 `library-sp24`！

---

## 🐛 Step 7：修 Bug！！！（唯一要写代码的部分）

好了好了，终于到正题了！这是整个 lab **唯一需要你动脑子写代码**的地方！

### 发现问题

先运行 `Arithmetic.java` 的 main 方法（点绿色三角形），跟着提示输入数字，你会发现——**结果不对！**

然后运行 `ArithmeticTest.java`，你会看到有些测试 ✅ 通过了，有些 ❌ 失败了。

### 怎么修？

打开 `Arithmetic.java`，仔细看代码。这个文件做的事情很简单——**基本的算术运算**。

> 🧠 **思考提示**（我不直接告诉你答案哦，你要独立完成嘛～）：
> 
> 1. 看看测试失败的是哪个方法？
> 2. 找到那个方法的实现代码
> 3. **仔细看运算符**——是不是用错了？或者**整数除法**的问题？
> 
> 在 Java 中有个经典坑：
> ```java
> int a = 3;
> int b = 4;
> System.out.println(a / b);  // 输出 0，不是 0.75！
> ```
> **Java 里两个 `int` 相除，结果还是 `int`，小数部分直接被砍掉了！** 如果你想要小数结果，至少有一个操作数需要是 `double` 或者 `float`。
> 
> 这个知识点够你解决这个 bug 了！去看看代码里有没有类似的问题？🔍

修完之后再跑一次 `ArithmeticTest`，看到全部 ✅ 就大功告成了！

---

## 📤 Step 8：提交作业

### 用 Git 保存你的工作

```bash
# 查看改了什么
git status

# 暂存你的修改
git add lab01/src/Arithmetic.java

# 提交（写一个有意义的提交信息）
git commit -m "lab01: Fixed Arithmetic.java bug"

# 推送到 GitHub
git push origin main
```

### 在 Gradescope 提交

1. 登录 Gradescope
2. 选择 Lab 01
3. 选 GitHub 提交方式
4. 选你的 `sp24-s***` 仓库和 `main` 分支
5. 提交！

自动评分器跑完，全绿就满分了！🎊

---

## 🌈 总结一下

| 你学到了什么 | 为什么重要 |
|-------------|-----------|
| Terminal 基本命令 | 以后天天用 |
| Git 工作流 (add → commit → push) | 工业界标准，必须会 |
| SSH 认证 | 安全地和 GitHub 通信 |
| IntelliJ 使用 | 这学期的主力 IDE |
| Java 整数除法的坑 | Java 基础知识点 |

---

> 💪 **最后的鼓励**：这个 lab 看着巨长，但本质就是环境搭建 + 修一个小 bug。你搞定这些之后，后面的 lab 才是真正有趣的数据结构部分！加油鸭～有什么不懂的随时问我！✨

你现在卡在哪一步了呀？还是还没开始？告诉我你的进度，我可以更有针对性地帮你！😊