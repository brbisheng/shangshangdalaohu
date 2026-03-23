# USING CHROOT to HAVE A SPECIFIC INSTANCE OF CC.

## 第一步：安装创建 chroot 要用的工具
```
apt update
apt install -y debootstrap
```
debootstrap 就是官方常用的“把一套最小 Debian 系统装进一个目录里”的工具。

## 第二步：创建两套最小 Debian
```
mkdir -p /opt/chroots/cc1 /opt/chroots/cc2
debootstrap --variant=minbase bookworm /opt/chroots/cc1 http://deb.debian.org/debian/
debootstrap --variant=minbase bookworm /opt/chroots/cc2 http://deb.debian.org/debian/
```
这里的 bookworm 是 Debian 12。--variant=minbase 的意思是只做一套比较瘦的最小系统。
执行完以后，这两个目录里会各自有一套最小 Debian 文件系统。

## 3. 宿主机上要再做一个进入脚本

这一步很重要。不要每次手打一长串 mount 和 chroot。

创建脚本：
```
cat >/usr/local/bin/chroot-enter <<'EOF'
#!/usr/bin/env bash
set -e

ROOT="$1"

if [ -z "$ROOT" ]; then
  echo "Usage: chroot-enter /opt/chroots/cc1"
  exit 1
fi

if [ ! -d "$ROOT" ]; then
  echo "No such directory: $ROOT"
  exit 1
fi

mkdir -p "$ROOT/dev" "$ROOT/dev/pts" "$ROOT/proc" "$ROOT/sys"

mountpoint -q "$ROOT/dev"     || mount --bind /dev "$ROOT/dev"
mountpoint -q "$ROOT/dev/pts" || mount --bind /dev/pts "$ROOT/dev/pts"
mountpoint -q "$ROOT/proc"    || mount -t proc /proc "$ROOT/proc"
mountpoint -q "$ROOT/sys"     || mount -t sysfs /sys "$ROOT/sys"

cp /etc/resolv.conf "$ROOT/etc/resolv.conf"

exec chroot "$ROOT" /bin/bash
EOF
```

`chmod +x /usr/local/bin/chroot-enter`

这个脚本做了四件事：

把宿主机的 /dev 绑进去
把 /proc 和 /sys 绑进去
把 DNS 配置复制进去
最后执行 chroot

这样你以后只需要输入一条命令。



## 4. 先给两个 chroot 放标记，防止你搞混

在宿主机执行：
```
echo cc1 > /opt/chroots/cc1/CHROOT_NAME
echo cc2 > /opt/chroots/cc2/CHROOT_NAME
```
这个文件非常有用。

因为你一旦进去以后，直接：

`cat /CHROOT_NAME`

看到 `cc1` 或 `cc2`，就知道自己现在到底在哪个环境里。




## 5. 第一次进入 cc1，并初始化

在宿主机执行：
```
chroot-enter /opt/chroots/cc1
```
如果命令成功，你会直接进入 cc1 里面的 shell。

这时候提示符可能看起来和宿主机一模一样，所以先别慌。立刻执行下面三条命令确认：
```
cat /CHROOT_NAME
pwd
ls /
```
如果 `cat /CHROOT_NAME` 输出：

`cc1`

那你就已经在 `cc1` 里了。

然后在 `cc1` 里面 执行下面这些初始化命令：

```
export HOME=/root
apt update
apt install -y curl git ca-certificates nano bash-completion
mkdir -p /root/.local/bin
echo 'export PATH=/root/.local/bin:$PATH' >> /root/.bashrc
echo 'export PS1="(cc1) \u@\h:\w# "' >> /root/.bashrc
source /root/.bashrc
```

这几步的作用分别是：

HOME=/root：确保你后面安装 Claude Code 时用的是 chroot 里的 /root
安装 curl git ca-certificates nano：这是后面安装和日常使用最基本的东西
建立 /root/.local/bin：Claude Code 原生安装推荐位置就是这里
把它加进 PATH
改提示符，让你一眼看到自己在 (cc1)

到这里，cc1 的 Debian 小环境已经准备好了。

## 6. 在 cc1 里安装 Claude Code

现在你已经在 cc1 里面了。

执行 Claude Code 官方当前推荐的 Linux 原生安装命令：

curl -fsSL https://claude.ai/install.sh | bash

这是官方 Quickstart 和 Overview 目前给出的 Linux 安装方式。

装完以后，检查：

which claude
ls -la /root/.local/bin/claude

你应该看到 Claude Code 在：

/root/.local/bin/claude

这个路径是 cc1 里面的路径，不是宿主机的 /root/.local/bin/claude。

再看用户设置目录：

ls -la /root/.claude

Claude Code 的用户级设置就是在这里。官方文档写的是 ~/.claude/settings.json。

如果你要在这个 chroot 里专门做一个项目目录，可以顺手建：

mkdir -p /root/projects/project-a
cd /root/projects/project-a
mkdir -p .claude

以后这个项目自己的项目级设置可以放在：

/root/projects/project-a/.claude/settings.json
/root/projects/project-a/.claude/settings.local.json

官方文档明确区分了这两个项目级设置文件。

7. 退出 cc1，回到宿主机

在 chroot 里面，执行：
```
exit
```
或者按：

Ctrl + D

这就回到了宿主机。

判断自己已经回到宿主机，执行：
```
ls /opt/chroots
```
如果能看到：

`cc1`  `cc2`

说明你已经在宿主机外面了。

为什么这个判断有效？因为在 chroot 里面，当前 / 已经是 /opt/chroots/cc1 了，所以你在里面再看 /opt/chroots/cc1，通常是找不到的；而回到宿主机以后，这条路径当然存在。

8. 再做 cc2，流程和 cc1 一模一样

在宿主机执行：
```
chroot-enter /opt/chroots/cc2
```
进去后先确认：
```
cat /CHROOT_NAME
```
如果输出：
```
cc2
```
说明你已经在第二套环境里。

然后在 cc2 里面 执行：
```
export HOME=/root
apt update
apt install -y curl git ca-certificates nano bash-completion
mkdir -p /root/.local/bin
echo 'export PATH=/root/.local/bin:$PATH' >> /root/.bashrc
echo 'export PS1="(cc2) \u@\h:\w# "' >> /root/.bashrc
source /root/.bashrc
curl -fsSL https://claude.ai/install.sh | bash
which claude
ls -la /root/.local/bin/claude
```
到这里，第二套 Claude Code 也装好了。

9. 现在这两套为什么不会打架

因为：

第一套的 Claude Code 二进制路径其实在宿主机上是：

`/opt/chroots/cc1/root/.local/bin/claude`

第二套的是：

`/opt/chroots/cc2/root/.local/bin/claude`

第一套用户设置目录其实在宿主机上是：

`/opt/chroots/cc1/root/.claude`

第二套的是：

`/opt/chroots/cc2/root/.claude`

物理目录完全不同，所以安装和配置不会撞在同一个真实路径上。

这正好绕开了 Claude Code 官方文档里提到的“多安装冲突”和默认推荐安装到 ~/.local/bin/claude 的问题。

10. 以后你平时怎么用

如果你要进第一套：

`chroot-enter /opt/chroots/cc1`

进去后：
```
cat /CHROOT_NAME
cd /root/projects/project-a
claude
```
如果你要进第二套：
```
chroot-enter /opt/chroots/cc2
```
进去后：
```
cat /CHROOT_NAME
cd /root/projects/project-b
claude
```
你会看到提示符分别像这样：
```
(cc1) root@...
(cc2) root@...
```
所以不容易混。

https://chatgpt.com/share/69c15a6d-8284-800a-a10f-c26b06c86c4d
