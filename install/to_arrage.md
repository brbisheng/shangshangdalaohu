🈯 常见命令：
* openclaw status

# SKILLS
* https://docs.openclaw.ai/tools/clawhub
 * npm install
* skill sources
 * bundled/managed skill (~/.openclaw/openclaw.json)
 * shared skills: ~/.openclaw/skills
 * per-agent workspace: (<workspace>/skills)



🕐待办：
- search
- skill
- new workspace.
- Memory.md是长期记忆. session_id是短期记忆.

1️⃣ 记忆文件夹的现状
在当前工作区（/root/.openclaw/workspace）里还没有 memory/ 子目录，也就没有每日日志文件。\
如果你想开始记录每天的日志，只要在工作区根目录下创建这个文件夹即可：\
mkdir -p /root/.openclaw/workspace/memory\
以后系统会把每天的原始笔记写到类似 memory/2026-02-17.md、memory/2026-02-18.md 之类的文件里。\

---

2️⃣ 你可以把技能放在哪里？

OpenClaw 本身的技能库位于 /root/.nvm/versions/node/v22.22.0/lib/node_modules/openclaw/skills/。\
这里已经有若干官方技能（如 clawhub、healthcheck、skill-creator、weather），你可以：\
直接使用已有技能：在终端里运行 openclaw skill list 查看全部可用技能。\
自己创建新技能：在上述 skills/ 目录下新建一个文件夹（例如 my‑tools），并在其中放置 SKILL.md + 代码/脚本。\
在工作区引用：如果你不想把自定义技能写进系统目录，也可以在工作区根目录下新建 custom-skills/（任意名称），把技能文件夹放进去，然后在 openclaw config set skillsPath /root/.openclaw/workspace/custom-skills 中告诉 OpenClaw 去那里查找（需要管理员执行一次配置命令）。

---

3️⃣ 工作区命名

每次启动 OpenClaw 时会使用 当前工作区的路径 作为默认空间。默认是 /root/.openclaw/workspace，名字本身没有特殊含义，只是一个文件系统位置。你完全可以：

创建多个工作区：比如 mkdir -p /root/.openclaw/workspace1、/root/.openclaw/workspace2。启动时把 OPENCLAW_WORKSPACE 环境变量指向对应路径，或在命令行加 --workspace /root/.openclaw/workspace1。
在不同工作区之间切换：每个工作区都有独立的 AGENTS.md、MEMORY.md、memory/等文件，互不干扰，适合把项目、实验或个人笔记分离开来。

---

下一步建议
创建 memory/ 文件夹（只需要一次）：

mkdir -p /root/.openclaw/workspace/memory


在工作区根目录下新建 custom-skills/（可选），以后把自己的技能放进去：

mkdir -p /root/.openclaw/workspace/custom-skills


然后让管理员运行：

openclaw config set skillsPath /root/.openclaw/workspace/custom-skills
openclaw gateway restart # 让新配置生效


如果想尝试新技能，在 custom-skills/hello-world/ 里放一个最简 SKILL.md：

# SKILL.md
name: hello-world
description: 简单示例技能，回复“Hello, World!”。


之后运行 openclaw skill list 检查是否能看到 hello-world。

告诉我你想先完成哪一步，我可以帮你执行对应的命令或写好示例文件！
