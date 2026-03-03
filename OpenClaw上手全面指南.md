**OpenClaw上手全面指南**

本指南基于官方文档和昆仑巢OpenClaw群社区贡献，提供结构化的教程，帮助用户从基础认知到生产级部署。内容涵盖安装、配置、核心机制和高级架构，旨在解决常见痛点如隐私泄露、集成复杂性和资源管理。

**一、入门：安装与初始化**

这一部分聚焦于 OpenClaw 的基本理解和快速上手，适合新手用户。目标是让您在 10 分钟内启动并运行第一个互动。

- 1.  **OpenClaw 是什么**
        1.  **定位**：自托管的个人 AI 助手框架，运行在本地机器或云端
        2.  **核心特征**：
- 将 LLM 接入日常消息平台（WhatsApp、Telegram、Discord 等）。
- 长期记忆
- 自我进化
- 主动唤醒+心跳
- 自我学习技能
    - 1.  **与 ChatGPT 的区别：**
- OpenClaw 是主动型 Agent（Proactive），可监控、定时执行任务（如心跳检查、提醒）；而 ChatGPT 是被动响应型，仅限于查询回复。
    1.  **环境准备与安装**
        1.  **系统要求**

Node.js 22+，支持系统： macOS、Linux、Windows（推荐 WSL2）。

- - 1.  **安装方式（三选一）：**
- **一键安装脚本**：

curl -fsSL https://openclaw.ai/install.sh | bash

- **npm 全局安装**：

npm install -g openclaw@latest。

- **源码构建**

git clone https://github.com/openclaw/openclaw，然后 pnpm install && pnpm build。

- - 1.  **初始化脚本**

a)运行：openclaw onboard，配置认证、Gateway 和频道。

b)再运行：openclaw doctor 检查环境。

- 1.  **核心概念速览：**

| **概念** | **说明** |
| --- | --- |
| Gateway | 核心进程，所有消息的路由中心，管理会话、工具和事件。 |
| Agent | AI 运行实例，每个 Agent 有独立工作区和会话，支持多模型集成。 |
| Channel | 消息平台接入插件（如 Telegram、WhatsApp），支持 20+ 平台。 |
| Skill | 可复用的 Markdown 格式提示模板，扩展 Agent 能力（如邮件、日历）。 |
| Node | 移动端/桌面端设备客户端，提供摄像头、屏幕等外设访问。 |

- 1.  **第一次启动与使用**
- **启动 Gateway**：openclaw gateway start --port 18789。
- **访问控制面板**：浏览器打开 http://127.0.0.1:18789/。
- **健康检查**：openclaw doctor（检测配置错误和运行问题）。
- **首次聊天**：通过已配置的消息平台（如 Telegram）向 Agent 发送消息，例如 "/status" 检查状态。

- 1.  **基础配置文件**

**配置路径**：

~/.openclaw/openclaw.json（JSON5 格式，支持注释）。

**配置方式（四选一）**：

- 交互式向导：openclaw onboard 或 openclaw configure。
- CLI 命令：openclaw config get/set/unset &lt;key&gt; &lt;value&gt;。
- 控制台 UI：通过网页可视化修改。
- 直接编辑文件（支持热重载，无需重启）。

**二、功能配置**

构建在基础之上，这一层探讨如何自定义 OpenClaw 以适应日常工作流。重点包括集成扩展和内存管理，帮助解决如上下文丢失或多平台切换的问题。

1.  1.  **频道(Channel)配置**

**内置支持平台**：WhatsApp（Web 协议）、Telegram（Bot API）、Discord（Bot API）、iMessage（macOS via BlueBubbles）、Slack、Signal、Google Chat、Matrix、Mattermost、MS Teams 等 20+。

- **频道接入**：openclaw channels login &lt;channel&gt;（例如设置 Telegram 的 botToken）。
- **访问策略配置**：
    
    - dmPolicy：pairing（默认审批）、allowlist、open、disabled。
    - groupPolicy：群聊触发模式（@mention 或关键词匹配）。
    - allowFrom：基于发件人的白名单。
    
    1.  **Agent 配置**
- **默认 Agent (Nova)**：内置，随 Gateway 启动。
- **多 Agent 设置**：定义多个命名 Agent，每个有独立会话。
- **模型配置**：格式 provider/model（如 anthropic/claude-opus-4-6）。支持 GPT、Claude、Gemini、DeepSeek、Llama 等。
- **回退模型**：配置 fallback chain 以处理失败。
- **本地模型**：通过 Ollama 集成（推荐 64k+ 上下文模型）。
    1.  **Skills(技能)系统**
- **技能格式**：带 YAML frontmatter 的 Markdown 文件（SKILL.md）。
- **技能类型**：邮件（Email）、日历（Calendar）、网页浏览（Web browsing）、文件操作（File operations）、自定义工作流。
- **ClawHub**：官方技能市场，Agent 可自动搜索安装（例如从 GitHub 仓库）。
- **安装命令**：openclaw skills install &lt;url&gt;，只安装 verified 标记的技能以确保安全。

#### OpenClaw 技能架构的理解

在核心中，OpenClaw 通过 Gateway 进程运行，该进程协调代理、内存和技能。技能不是单纯的提示；它们封装可重用工作流，通常需要工具（如 shell 访问）来执行。捆绑技能随框架提供，即时可用，而 ClawHub——一个拥有超过 13,000 条目的公共注册表——托管社区贡献。安装简单：使用 ClawHub CLI（npx clawhub@latest install &lt;skill-slug&gt;）或手动放置到 ~/.openclaw/skills/ 文件夹。然而，正如安全审计所指出的，始终优先选择 ClawHub 上带有徽章的已验证技能，以缓解风险，如 2026 年早期报告中影响约 1,200 个技能的恶意软件。性能提升来自链式技能——例如，将 'github' 与 'coding-agent' 结合用于自动代码审查——根据社区基准，在开发工作流中可减少 50-70% 的手动工作。

- 1.  **内存(Memory)系统**
- **持久化存储**：纯文本文件，跨会话保留上下文。
- **核心文件**：
    - agents.md：Agent 行为偏好与角色定义。
    - soul.md：用户个人偏好与风格。
    - 会话对话文件：历史记录。
- **GitHub 同步**：支持将内存文件同步到私有 GitHub 仓库备份。
    1.  **会话(Session)管理**
- **会话作用域 (dmScope)**：
    - main：所有 DM 共用一个会话。
    - per-peer：每个发件人独立会话。
    - per-channel-peer：每个频道 + 发件人独立。
    - per-account-channel-peer：最细粒度隔离。
- **会话重置**：支持 daily 模式（按时每日重置）。
- **线程绑定**：配置 idleHours 和 maxAgeHours 控制线程生命周期。
    1.  **工具（Tools）与媒体支持**
- **内置工具**：网页搜索（Web search）、浏览器自动化（Browser automation）、Canvas 画布操作、Node 设备调用、Cron 定时任务。
- **媒体类型**：支持图片、音频、文档。
- **图片处理**：imageMaxDimensionPx（默认 1200px 压缩）。

**三、架构深度与生产部署**

针对开发者和服务运维，这一节深入探讨底层机制和规模化部署。包括 RPC、路由和安全最佳实践，帮助解决复杂场景如多实例协调或远程访问。

1.  1.  **Gateway 深度架构**

- **Gateway 本质**：单一长期运行的 Node.js 进程，同时承担：
    - WebSocket RPC 服务（ws://127.0.0.1:18789）。
    - HTTP 服务（Control UI、/tools/invoke、OpenAI 兼容 API）。
    - 会话持久化与 transcript 存储。
    - Cron 调度与内存索引。
- **PI Agent 运行时**：基于 @mariozechner/pi-coding-agent，LLM 可直接创建、编辑、运行、删除文件。
- **LLM 角色**：作为编排者（Orchestrator），而非仅知识提供者。
    1.  **RPC 协议**
- **协议层**：WebSocket 结构化帧，包括 RequestFrame（客户端→服务端）、ResponseFrame（服务端→客户端）、EventFrame（广播）。
- **方法命名空间**：agent._、chat._、sessions._、cron._、config._、nodes._、mesh.\*。
- **工具调用 API**：/tools/invoke HTTP 端点（OpenAI 兼容接口）。
- **速率限制**：config.\* 编程更新限制 3 次/60 秒/设备。
    1.  **多 Agent 路由架构**
- **隔离策略**：每个 Agent 拥有独立工作区目录（~/.openclaw/workspace）。
- **路由规则**：按发件人、频道、工作区维度路由到不同 Agent。
- **跨 Agent 通信**：通过 Mesh 命名空间（mesh.\*）实现。
- **工作流示例**：主 Agent 接收消息 → 路由至专属 Agent（如“市场分析 Agent”） → 结果汇总回主频道。
    1.  **自动化与调度系统**
- **Heartbeat (心跳)**：默认每 30 分钟触发一次主动检查，配置 interval 和 deliveryTargets。
- **Cron 定时任务**：完整 cron 表达式支持，配置 concurrency（并发数）和 retention（保留记录数）。
- **Webhook Hooks**：Token 认证，支持将外部事件映射到指定 Session。
    1.  **安全与沙箱**
- **单操作员模型**：默认仅 loopback 绑定（127.0.0.1）。
- **认证机制**：Token 或密码认证。
- **Docker 沙箱**：模式 off/non-main/all，作用域 session/agent/shared。
- **Exec 批准级别**：控制 shell 命令执行权限。
- **安全审计**：openclaw security audit。
- **最佳实践**：最小权限原则、不可逆操作需二次确认、日志全量记录（推荐接入 Supabase）、部署在 VPS 而非个人机器。
    1.  **生产部署与守护进程**
- **Daemon 安装**：openclaw onboard --install-daemon，macOS 生成 launchd 单元，Linux/WSL2 生成 systemd 单元。
- **Gateway 生命周期管理**：openclaw gateway \[start|stop|restart|status\]。
- **远程访问**：SSH 隧道、Tailscale（tailnet 访问）、TLS 配置（HTTPS）。
- **成本优化策略**：按任务类型选择模型（Gemini Flash 做摘要，Claude Opus 做复杂推理）、设置月度消费上限、直接 SSH 编辑配置避免额外 API token 消耗。
    1.  **配置高级特性**
- **环境变量注入**：来源父进程、.env、~/.openclaw/.env，语法 ${VAR_NAME}，支持 env/file/exec 类型的 secret 引用。
- **配置模块化 ($include)**：将大型配置拆分为多个文件，支持数组深度合并，最多 10 层嵌套。
- **热重载策略**：hybrid（默认：安全变更即时生效，关键变更自动重启）、hot（仅安全变更）、restart（任何变更均重启）、off（禁用文件监听）。
- **编程式配置更新**：通过 config.\* RPC 方法，支持 schema 验证。

**附件1：OpenClaw25 个核心工具：技能执行的基础**

工具是技能调用的低级原语，按层分类以管理风险。第 1 层关注基本 I/O，第 2 层关注协调，扩展层关注工作流。下表总结所有 25 个工具，并基于风险和实用性给出启用推荐。基本工具如 'read'、'write' 和 'web_search' 对于代理的基本认知不可或缺，而高风险工具如 'exec' 提升自动化，但需要审批（例如，通过配置标志）以防止意外系统更改。

| **层级** | **工具名称** | **描述** | **风险级别** | **是否启用？** | **备注/提升** |
| --- | --- | --- | --- | --- | --- |
| 1   | read | 从磁盘读取文件 | 低   | 是   | 上下文加载必备；启用内存持久化。 |
| 1   | write | 写入/创建文件 | 中   | 是   | 输出生成核心；与技能配对用于日志记录。 |
| 1   | edit | 结构化文件编辑 | 中   | 是   | 提高代码/文件修改精度；减少错误。 |
| 1   | apply_patch | 应用代码补丁 | 中   | 是   | 开发工作流关键；自动化更新。 |
| 1   | exec | 执行 shell 命令 | 极高  | 是（需审批） | 解锁完整自动化；显著提升功能，但需沙箱。 |
| 1   | process | 管理系统进程 | 中   | 是   | 增强多任务；用于长时间任务。 |
| 1   | web_search | 执行网络搜索 | 低   | 是   | 实时信息必备；与研究技能集成。 |
| 1   | web_fetch | 获取原始网络内容 | 中   | 是   | 补充浏览；改善数据抓取效率。 |
| 2   | browser | 完整浏览器自动化 | 高   | 是   | 推荐用于交互网络任务；提升代理自治。 |
| 2   | canvas | 视觉绘图工作区 | 低   | 否   | 可选用于创意任务；性能影响最小。 |
| 2   | image | 分析/处理图像 | 低   | 是   | 提升媒体处理；与创意技能配对。 |
| 2   | memory_search | 搜索代理内存 | 中   | 是   | 改善上下文回忆；长期性能关键。 |
| 2   | memory_get | 检索内存条目 | 中   | 是   | 持久状态必备；减少冗余查询。 |
| 2   | sessions_list | 列出会话 | 低   | 是   | 辅助管理；开销低。 |
| 2   | sessions_history | 访问会话历史 | 中   | 是   | 增强调试；用于反思技能。 |
| 2   | sessions_send | 向会话发送消息 | 高   | 是（仅自身） | 提升多代理通信；限制以避免垃圾信息。 |
| 2   | sessions_spawn | 创建子代理 | 高   | 是   | 可选用于扩展；显著扩展复杂工作流。 |
| 2   | session_status | 检查会话健康 | 低   | 是   | 推荐用于监控；防止停机。 |
| 2   | message | 跨频道消息 | 极高  | 是（仅自身） | 集成核心；提升连接性。 |
| 2   | nodes | 控制硬件节点 | 极高  | 否   | 安全禁用；仅用于 IoT 时启用。 |
| 2   | cron | 调度重复任务 | 高   | 是   | 主动性必备；自动化提醒/检查。 |
| 2   | gateway | 管理 OpenClaw 网关 | 高   | 是   | 系统控制关键；改善稳定性。 |
| 2   | agents_list | 列出运行代理 | 低   | 是   | 可选但用于监督有用。 |
| Ext | llm_task | 基于 LLM 的工作流步骤 | 中   | 否   | 禁用，除非用于高级链式。 |
| Ext | lobster | 工作流协调引擎 | 中   | 否   | 可选用于企业级自动化。 |

这些工具直接放大技能——例如，'exec' 启用 'tmux' 用于终端管理，导致更健壮的开发环境。

**附件2：OpenClaw 53 个官方技能：内置必备项**

官方技能已捆绑且无风险，按类别组织。它们提供基线能力，其中优先项如 'gog'（Google 集成）和 'github' 启用生产力跃升。高优先级技能（如 'clawhub' 用于技能管理）推荐所有用户，因为它们无需外部依赖即可促进扩展。下表详细列出所有技能，并基于典型用例给出启用建议。

| **类别** | **技能名称** | **功能** | **风险/是否启用？** | **备注/提升** | **高度推荐？** |
| --- | --- | --- | --- | --- | --- |
| 笔记  | obsidian | Obsidian 仓库集成 | 低/否 | 本地笔记管理 | PKM 用户可选 |
| 笔记  | notion | Notion API 访问 | 中/是 | 云端文档 | 团队推荐 |
| 笔记  | apple-notes | Apple Notes (macOS) | 低/否 | 平台特定 | 非 macOS 禁用 |
| 笔记  | bear-notes | Bear 应用 (macOS) | 低/否 | Markdown 笔记 | 禁用  |
| 任务  | things-mac | Things 3 (macOS) | 低/否 | 任务列表 | 禁用  |
| 任务  | apple-reminders | Apple 提醒 | 低/否 | 简单提醒 | 禁用  |
| 任务  | trello | Trello 面板 | 中/否 | 项目跟踪 | 可选  |
| 工作  | gog | Google Workspace (邮件/日历) | 中/是 | OAuth 基于；可撤销 | 高度；提升生产力 |
| 工作  | himalaya | IMAP/SMTP 邮件 | 中/否 | 基本邮件 | 禁用  |
| 聊天  | slack | Slack 消息 | 高/否 | 完整频道访问 | 隐私禁用 |
| 聊天  | discord | Discord 机器人 | 高/否 | 服务器访问 | 禁用  |
| 聊天  | wacli | WhatsApp CLI | 高/否 | 消息  | 禁用  |
| 聊天  | imsg | iMessage (macOS) | 高/否 | 短信  | 禁用  |
| 聊天  | bluebubbles | 外部 iMessage | 高/否 | 跨平台 | 禁用  |
| 社交  | bird | X/Twitter API | 高/否 | 发布/阅读 | 禁用  |
| 开发  | github | GitHub CLI | 中/是 | 问题/PR；OAuth | 高度；开发者必备 |
| 开发  | coding-agent | AI 代码生成 | 中/否 | 调用外部 LLM | 编码推荐 |
| 开发  | tmux | 终端多路复用 | 低/是 | 会话管理 | 推荐  |
| 开发  | session-logs | 日志搜索 | 低/是 | 转录分析 | 推荐  |
| 音乐  | spotify-player | Spotify 控制 | 中/否 | 播放  | 禁用  |
| 音乐  | sonoscli | Sonos 扬声器 | 中/否 | 音频流 | 禁用  |
| 音乐  | blucli | BluOS 系统 | 中/否 | 音乐硬件 | 禁用  |
| 家居  | openhue | Philips Hue 灯 | 中/否 | IoT 控制 | 禁用  |
| 家居  | eightctl | Eight Sleep 床 | 中/否 | 睡眠跟踪 | 禁用  |
| 食物  | food-order | 外卖平台 | 中/否 | 订购  | 禁用  |
| 食物  | ordercli | Foodora 特定 | 中/否 | 区域外卖 | 禁用  |
| 创意  | openai-image-gen | DALL-E 图像 | 中/否 | 生成  | 禁用  |
| 创意  | nano-banana-pro | Gemini 图像 | 中/否 | 替代生成 | 禁用  |
| 创意  | video-frames | 视频提取 | 中/否 | 媒体处理 | 禁用  |
| 创意  | gifgrep | GIF 搜索 | 低/否 | 趣味媒体 | 禁用  |
| 语音  | sag | ElevenLabs TTS | 中/否 | 语音合成 | 禁用  |
| 语音  | openai-whisper | 本地 STT | 中/否 | 转录  | 禁用  |
| 语音  | openai-whisper-api | 云 STT | 中/否 | 准确转录 | 禁用  |
| 语音  | sherpa-onnx-tts | 离线 TTS | 低/否 | 本地语音 | 禁用  |
| 安全  | 1password | 密码管理器 | 高/否 | 仓库访问 | 禁用；使用替代 |
| AI  | gemini | Gemini 模型集成 | 中/否 | LLM 调用 | 禁用  |
| AI  | oracle | Oracle DB | 中/否 | 数据库查询 | 禁用  |
| AI  | mcporter | MCP 工具 | 中/否 | 自定义集成 | 禁用  |
| 系统  | clawhub | 技能市场 | 低/是 | 安装/管理技能 | 推荐  |
| 系统  | skill-creator | 创建自定义技能 | 低/是 | DIY 扩展 | 可选  |
| 系统  | healthcheck | 系统诊断 | 低/是 | 监控  | 推荐  |
| 系统  | summarize | 文本摘要 | 低/是 | 内容消化 | 推荐  |
| 系统  | weather | 天气 API | 低/是 | 预报  | 推荐  |
| 地点  | goplaces | Google Places | 中/否 | 位置搜索 | 禁用  |
| 地点  | local-places | 代理地点 | 中/否 | 近似离线 | 禁用  |
| 媒体  | camsnap | RTSP 摄像头 | 高/否 | 视频捕获 | 禁用  |
| 新闻  | blogwatcher | RSS 监控 | 低/否 | 源跟踪 | 禁用  |
| 文档  | nano-pdf | PDF 编辑 | 中/否 | 文档修改 | 禁用  |
| 监控  | model-usage | LLM 跟踪 | 低/否 | 成本监控 | 禁用  |
| 系统  | peekaboo | macOS UI 交互 | 高/否 | 屏幕控制 | 禁用  |
| 通信  | voice-call | 语音通话 | 高/否 | 电话  | 禁用  |
| 创意  | canvas | 绘图操作 | 低/否 | 视觉  | 禁用  |
| 音乐  | songsee | 音频可视化 | 中/否 | 波形  | 禁用  |

这些技能确立了 OpenClaw 的多功能性；例如，'gog' 主动处理邮件/日历，减少上下文切换。

**附件3：你必须要看看的OpenClaw社区技能：来自 Awesome-OpenClaw-Skills 的高影响力增强器**

来自拥有 5,400+ 条目的精选仓库，这些技能填补官方技能的空白，强调自治、安全和专业化。类别如 DevOps 和安全对于生产使用至关重要，技能如 'agent-sovereign-stack' 通过单命令启用自托管基础设施部署——显著改善可扩展性。下表突出每个类别的顶级技能，聚焦于 GitHub 活跃度高的（如下载量超过 10k）。

| **类别** | **技能名称** | **描述** | **提升** | **是否推荐？** |
| --- | --- | --- | --- | --- |
| 编码代理 | agent-evolver | 通过错误学习自我进化 | 提升适应性；任务解析快 2 倍 | 高度  |
| 编码代理 | agent-reflect | 分析对话以改进 | 增强推理；减少幻觉 | 高度  |
| DevOps | agent-sovereign-stack | 单命令主权基础设施部署 | 提升自治；减少设置时间 80% | 必备  |
| DevOps | agentic-devops | Docker/监控工具包 | 稳定操作；添加日志 | 高度  |
| 浏览器 | agent-browser | 无头 Rust 浏览器 | 启用反机器人抓取；提升网络任务 | 必备  |
| 浏览器 | playwright-mcp | 完整浏览器自动化 | 自动化表单/截图；效率 3 倍 | 高度  |
| 安全  | arc-trust-verifier | 验证技能来源 | 缓解恶意软件；构建信任分数 | 必备  |
| 安全  | credential-manager | 凭证安全仓库 | 保护 API；生产必备 | 必备  |
| AI/LLMs | agent-memory | 加密云内存 | 扩展上下文；改善长期回忆 | 高度  |
| Git/GitHub | arc-skill-gitops | 自动化技能部署 | 简化版本控制；减少错误 | 推荐  |
| 通信  | agent-mail | 代理邮箱 | 启用外部通信；集成工作流 | 高度  |
| 智能家居 | google-home | Nest 设备控制 | 自动化 IoT；增强家居集成 | 推荐  |
| 健康  | calorie-counter | 跟踪摄入/目标 | 个性化健康；添加数据分析 | 推荐  |
| 代理协议 | claw-to-claw | 多代理协调 | 扩展协作；提升复杂任务 | 高度  |

安装 5-10 个这些技能（例如，从安全技能开始）可将 OpenClaw 从基本助手转变为健壮协调器，尽管建议通过 'healthcheck' 监控冲突。