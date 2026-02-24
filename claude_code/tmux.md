这个报错的含义是：`tmux` 客户端想去连接一个已经存在的 tmux 服务器 socket（路径 `/tmp/tmux-0/default`），但那个 socket 文件不存在了。最常见的原因是：**tmux 服务端没在跑**，或者 **/tmp 被清空了**（VPS 重启、系统定时清理 tmp、tmpfs 挂载等都会导致 `/tmp` 里的 socket 消失）。

按下面顺序排查/修复（每步都给出能看到的“具体对象/文件/进程”）：

---

## 1) 先确认有没有 tmux 进程（具体对象：进程）

```bash
ps aux | grep '[t]mux'
```

* 如果没有任何 `tmux` 进程：说明 tmux server 没在跑，直接第 3 步新建会话即可。
* 如果看到了 `tmux` 进程：可能是 socket 路径不对或权限/残留问题，继续第 2 步。

---

## 2) 看 /tmp 里有没有 tmux 目录和 socket（具体对象：目录、socket 文件）

```bash
ls -ld /tmp /tmp/tmux-0
ls -l /tmp/tmux-0 2>/dev/null
```

你报错指向 `/tmp/tmux-0/default`，正常情况下这里会有一个叫 `default` 的 socket 文件（类型一般是 `s`）。

* 如果 `/tmp/tmux-0` 根本不存在：`/tmp` 被清空或 tmux 没启动过/没成功启动。
* 如果存在但里面没有 `default`：socket 没生成或被删了。
* 如果存在但权限不对（比如属于别的用户）：会导致连接失败。

---

## 3) 直接新建一个 tmux 会话（最常用的恢复）

```bash
tmux new -s work
```

成功后你会进入 tmux。再开一个 SSH 窗口可以验证：

```bash
tmux ls
```

能列出 `work` 就说明恢复了。

---

## 4) 如果提示已有 server 但还是连不上：清理残留 socket/目录（具体对象：残留目录）

有时 tmux 异常退出会留下坏的目录结构。先确保没有重要会话（通常连不上就基本没有可救的会话了）。

1. 再次确认没有 tmux 进程：

```bash
ps aux | grep '[t]mux'
```

2. 如果确实没有 tmux 进程，再删掉残留目录：

```bash
rm -rf /tmp/tmux-0
```

3. 然后重新创建：

```bash
tmux new -s work
```

---

## 5) 你是 root 还是普通用户：检查 socket 目录名是否该是 tmux-<uid>（具体对象：UID、路径）

tmux 默认用 **UID** 命名目录：`/tmp/tmux-<uid>/default`。你这里是 `tmux-0`，意味着你在用 **root(UID=0)** 跑 tmux。

确认当前用户 UID：

```bash
id -u
```

* 输出 `0`：root，路径 `tmux-0` 合理
* 输出不是 `0`：那正常路径应是 `/tmp/tmux-<你的uid>/default`，你却在连 `/tmp/tmux-0/default`，说明你可能在不同用户之间切换导致混乱。

这时用当前用户明确启动并列出：

```bash
tmux -S /tmp/tmux-$(id -u)/default ls
```

或者直接用当前用户新建：

```bash
tmux new -s work
```

---

## 6) 为什么“我明明装了 tmux”还会这样（具体对象：/tmp 挂载、系统重启）

安装只是把二进制放到磁盘（例如 `/usr/bin/tmux`），但 **socket 在 `/tmp`**，而 `/tmp` 常见情况会丢失：

* VPS 重启后 `/tmp` 清空
* `/tmp` 是 tmpfs（内存盘），重启即没
* systemd-tmpfiles 定期清理 `/tmp`

你可以看一下 `/tmp` 是否是 tmpfs：

```bash
df -h /tmp
mount | grep ' /tmp '
```

---

如果你把下面三条命令的输出贴出来（无需敏感信息），我可以直接定位是哪一种情况并给最短修复路径：

```bash
id
ps aux | grep '[t]mux'
ls -l /tmp /tmp/tmux-* 2>/dev/null
```
