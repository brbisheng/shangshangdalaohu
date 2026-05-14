`tmux new -s agent` 的意思是：

```bash
tmux new -s agent
```

拆开看：

```text
tmux = 启动 tmux
new = 新建一个 tmux session
-s agent = 这个 session 的名字叫 agent
```

所以它不是自动保存聊天记录。

它只是创建一个“VPS 内部的长期终端空间”，名字叫：

```text
agent
```

你可以理解成：你在 VPS 里面开了一个独立工作间，名字叫 `agent`。

---

## 1. `tmux new -s agent` 会自动保存信息吗？

不会。

它不会自动保存 Claude Code / Codex 的聊天内容到文件。

它主要保存的是：

```text
这个 terminal session 还活着
里面正在运行的程序还活着
当前窗口/分屏结构还在
屏幕上的一部分历史输出还在
```

但它不是正式日志系统。

如果你想把文字完整保存成文件，需要额外用：

```bash
script -a ~/agent-session.log
```

这个才是保存 terminal 文字日志。

---

## 2. 如果我在 tmux 里 `exit` 了会怎样？

要看你在哪里 `exit`。

### 情况 A：你在 Claude Code / Codex 里面 exit

比如你正在跑：

```bash
claude
```

你输入：

```bash
exit
```

这通常只是退出 Claude Code，回到 shell。

画面可能变成：

```bash
your_user@vps:~/project$
```

tmux session 还在。

---

### 情况 B：你在 shell 里 exit

如果 Claude Code 已经退出了，你又输入：

```bash
exit
```

这会关闭当前 tmux pane。

如果这个 tmux session 里只有一个 pane，那么整个 tmux session 会结束。

也就是：

```text
agent 这个 tmux session 没了
```

下次你再运行：

```bash
tmux attach -t agent
```

可能会看到：

```text
no sessions
```

---

## 3. 如果我只是想离开，但不关闭 tmux，应该怎么做？

不要输入 `exit`。

应该按：

```text
Ctrl+b
然后按 d
```

意思是：

```text
detach = 暂时离开 tmux，但不关闭它
```

之后你会回到普通 VPS shell。

下次重新进去：

```bash
tmux attach -t agent
```

就能回到原来的 Claude Code / Codex 界面。

---

## 4. 最关键区别

```text
exit = 关闭当前程序 / 当前窗口
Ctrl+b d = 离开但保留现场
```

所以你平时应该多用：

```text
Ctrl+b d
```

不要随便用：

```text
exit
```

---

## 5. AGENT_STATE.md 不是“一个屏幕”

`AGENT_STATE.md` 是一个真实文件。

比如它在你的项目文件夹里：

```text
your_project/AGENT_STATE.md
```

你可以用命令看它：

```bash
cat AGENT_STATE.md
```

或者用编辑器打开：

```bash
nano AGENT_STATE.md
```

它不是 tmux 屏幕。

但是你可以用 tmux 分屏，把它显示在右边。

例如：

```text
左边：Claude Code / Codex
右边：AGENT_STATE.md
```

画面像这样：

```text
┌──────────────────────────────┬──────────────────────────────┐
│ claude                       │ cat AGENT_STATE.md            │
│                              │                              │
│ agent 正在运行                │ 当前任务摘要                   │
│ 输出、报错、修改代码           │ 已改文件、当前问题、下一步      │
└──────────────────────────────┴──────────────────────────────┘
```

---

## 6. 可以再开一个屏幕看 log 吗？

可以。

你可以有三个区域：

```text
左边大窗口：Claude Code / Codex
右上：AGENT_STATE.md
右下：agent-session.log
```

大概这样：

```text
┌──────────────────────────────┬──────────────────────────────┐
│ Claude Code / Codex          │ AGENT_STATE.md                │
│                              │ 当前任务摘要                   │
│                              ├──────────────────────────────┤
│                              │ agent-session.log             │
│                              │ 历史文字日志                   │
└──────────────────────────────┴──────────────────────────────┘
```

---

## 7. 如何创建多个屏幕 / 分屏？

进入 tmux 以后：

### 左右分屏

```text
Ctrl+b
然后按 %
```

结果：

```text
左边一个 pane
右边一个 pane
```

### 上下分屏

```text
Ctrl+b
然后按 "
```

结果：

```text
上面一个 pane
下面一个 pane
```

### 在 pane 之间移动

```text
Ctrl+b
然后按方向键
```

例如：

```text
Ctrl+b  → 
```

移动到右边 pane。

---

## 8. 一个实际操作例子：三个屏幕

假设你已经进入项目：

```bash
cd your_project
tmux new -s agent
```

现在你在第一个 pane 里。

### 第一步：左边跑 Claude Code

```bash
claude
```

或者：

```bash
codex
```

---

### 第二步：左右分屏

按：

```text
Ctrl+b
%
```

现在右边出现新 pane。

---

### 第三步：右边看 AGENT_STATE.md

在右边输入：

```bash
watch -n 2 cat AGENT_STATE.md
```

意思是每 2 秒刷新一次显示。

右边就会持续显示任务状态。

---

### 第四步：把右边再上下分屏

确保光标在右边 pane。

按：

```text
Ctrl+b
"
```

右边被切成上下两个。

---

### 第五步：右下角看 log

在右下角输入：

```bash
tail -f ~/agent-session.log
```

意思是持续看日志文件末尾。

---

## 9. 但是要注意：log 文件要先开始记录

如果你要有 `agent-session.log`，启动 Claude Code / Codex 前要这样做：

```bash
script -a ~/agent-session.log
claude
```

或者：

```bash
script -a ~/agent-session.log
codex
```

更准确的流程是：

```bash
cd your_project
tmux new -s agent
script -a ~/agent-session.log
claude
```

这样 Claude Code 输出的文字才会被写进：

```text
~/agent-session.log
```

---

## 10. 推荐你第一次只用两个屏幕

不要一开始搞太复杂。

先这样：

```text
左边：Claude Code / Codex
右边：AGENT_STATE.md
```

操作：

```bash
cd your_project
tmux new -s agent
```

进入以后：

```bash
script -a ~/agent-session.log
claude
```

然后按：

```text
Ctrl+b
%
```

右边输入：

```bash
watch -n 2 cat AGENT_STATE.md
```

如果右边报错说文件不存在，先创建：

```bash
touch AGENT_STATE.md
watch -n 2 cat AGENT_STATE.md
```

---

## 11. 以后怎么回来？

离开但不关闭：

```text
Ctrl+b
d
```

重新进来：

```bash
tmux attach -t agent
```

查看有哪些 tmux session：

```bash
tmux ls
```

如果看到：

```text
agent: 1 windows
```

说明它还活着。

---

## 12. 你真正要记住的几个命令

```bash
tmux new -s agent
```

新建一个叫 `agent` 的工作区。

```bash
tmux attach -t agent
```

回到这个工作区。

```bash
tmux ls
```

查看有哪些工作区。

```text
Ctrl+b d
```

离开但不关闭。

```text
Ctrl+b %
```

左右分屏。

```text
Ctrl+b "
```

上下分屏。

```text
Ctrl+b 方向键
```

切换到另一个屏幕区域。

```bash
script -a ~/agent-session.log
```

开始保存 terminal 文字日志。

```bash
watch -n 2 cat AGENT_STATE.md
```

每 2 秒刷新查看任务摘要。

```bash
tail -f ~/agent-session.log
```

实时查看日志末尾。
