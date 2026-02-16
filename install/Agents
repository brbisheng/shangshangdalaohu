# 


代理运行时  
OpenClaw 运行一个源自 pi-mono 的单一嵌入式代理运行时。  

​  
工作区（必需）  
OpenClaw 使用一个单一的代理工作区目录（agents.defaults.workspace）作为代理用于工具和上下文的唯一工作目录（cwd）。  
建议：使用 openclaw setup 创建 ~/.openclaw/openclaw.json（如果缺失），并初始化工作区文件。  
完整的工作区布局 + 备份指南：Agent workspace  
如果启用了 agents.defaults.sandbox，非主会话可以使用 agents.defaults.sandbox.workspaceRoot 下的每会话工作区来覆盖此设置（参见 Gateway 配置）。  

​  
引导文件（注入）  
在 agents.defaults.workspace 内，OpenClaw 期望以下用户可编辑文件：  
AGENTS.md — 操作说明 + “记忆”  
SOUL.md — 人设、边界、语气  
TOOLS.md — 用户维护的工具说明（例如 imsg、sag、约定）  
BOOTSTRAP.md — 一次性首次运行仪式（完成后删除）  
IDENTITY.md — 代理名称/风格/表情符号  
USER.md — 用户档案 + 偏好的称呼方式  
在新会话的第一轮中，OpenClaw 会将这些文件的内容直接注入到代理上下文中。  
空白文件会被跳过。大型文件会被裁剪并以标记截断，以保持提示精简（读取文件可查看完整内容）。  
如果文件缺失，OpenClaw 会注入一行“缺失文件”标记（并且 openclaw setup 会创建一个安全的默认模板）。  
BOOTSTRAP.md 仅会在全新的工作区中创建（不存在其他引导文件时）。如果你在完成仪式后将其删除，它不应在后续重启时重新创建。  
要完全禁用引导文件创建（用于预置工作区），请设置：  
{ agent: { skipBootstrap: true } }  

​  
内置工具  
核心工具（read/exec/edit/write 及相关系统工具）始终可用，但受工具策略限制。apply_patch 是可选的，并由 tools.exec.applyPatch 控制。TOOLS.md 不控制哪些工具存在；它只是关于你希望如何使用这些工具的指导。  

​  
技能  
OpenClaw 从三个位置加载技能（名称冲突时以工作区为准）：  
Bundled（随安装包提供）  
Managed/local：~/.openclaw/skills  
Workspace：<workspace>/skills  
技能可以通过配置/环境进行限制（参见 Gateway 配置中的 skills）。  

​  
pi-mono 集成  
OpenClaw 重用 pi-mono 代码库中的部分内容（模型/工具），但会话管理、发现和工具连接由 OpenClaw 自行负责。  
没有 pi-coding 代理运行时。  
不会读取 ~/.pi/agent 或 <workspace>/.pi 设置。  

​  
会话  
会话记录以 JSONL 格式存储在：  
~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl  
会话 ID 是稳定的，并由 OpenClaw 选择。不会读取旧版 Pi/Tau 会话文件夹。  

​  
流式传输时的引导  
当队列模式为 steer 时，传入消息会被注入到当前运行中。队列会在每次工具调用后检查；如果存在排队消息，则会跳过当前助手消息中剩余的工具调用（工具结果报错为“Skipped due to queued user message.”），然后在下一个助手响应前注入排队的用户消息。  
当队列模式为 followup 或 collect 时，传入消息会被保留直到当前回合结束，然后以排队的负载启动新的代理回合。有关模式 + 去抖/上限行为，请参见 Queue。  
块式流传输会在助手块完成后立即发送；默认关闭（agents.defaults.blockStreamingDefault: "off"）。通过 agents.defaults.blockStreamingBreak（text_end 或 message_end；默认为 text_end）调整边界。使用 agents.defaults.blockStreamingChunk 控制软块分块（默认 800–1200 个字符；优先段落分隔，其次换行；句子最后）。通过 agents.defaults.blockStreamingCoalesce 合并流式块以减少单行刷屏（发送前基于空闲时间合并）。非 Telegram 渠道需要显式设置 *.blockStreaming: true 才能启用块式回复。详细工具摘要会在工具启动时发出（无去抖）；Control UI 在可用时通过代理事件流式传输工具输出。更多详情：Streaming + chunking。  

​  
模型引用  
配置中的模型引用（例如 agents.defaults.model 和 agents.defaults.models）通过在第一个 / 处分割来解析。  
配置模型时请使用 provider/model。  
如果模型 ID 本身包含 /（OpenRouter 风格），请包含提供方前缀（例如：openrouter/moonshotai/kimi-k2）。  
如果省略提供方，OpenClaw 会将输入视为默认提供方的别名或模型（仅当模型 ID 中不包含 / 时有效）。

