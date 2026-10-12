# Claude 最新动态

精选 Claude 产品更新、API 变更、地区政策与社区风控趋势，助你及时调整使用策略。

条目由本站人工整理，来源为 Anthropic 官方公告与社区反馈。点击标题将在新标签页打开外部链接。

## Claude 最新动态

精选 Claude 产品更新、API 变更、地区政策与社区风控趋势，助你及时调整使用策略。

产品动态 2026年10月8日

### [Claude Code 2.1.292–2.1.294：权限与沙箱修复、stdio MCP 协议协商、prompt hook 修复](https://code.claude.com/docs/en/changelog)

Claude Code 2.1.292 修复多处权限绕过：PreToolUse hook 的批准与 auto mode 会跳过网络（UNC）路径文件读取的权限确认；沙箱内命令可读取 `/ultrareview` 上传的暂存副本；服务端托管设置在拉取失败时，被篡改的本地缓存可关闭内置策略插件。该版本还让本地（stdio）MCP 服务器在所有安装环境（含 Bedrock、Vertex、Foundry）默认协商 2026-07-28 协议版本，可用 `MCP_PROTOCOL_NEGOTIATION=legacy` 退回旧行为。2.1.293 将 Claude Haiku 5.5 设为默认 Haiku 模型。2.1.294 修复以指令形式书写的 `prompt` 与 `agent` hook（如「Block commands that…」）未能拦截目标命令的问题，并改进 Stop 与 SubagentStop prompt hook 的判定。

实战要点： 若依赖 `prompt` 或 `agent` hook 拦截命令，请升级到 2.1.294 并重新验证；旧版本在 hook 以指令形式书写时可能放行本应拦截的命令。

来源: [Anthropic ↗](https://code.claude.com/docs/en/changelog) [相关实战指南 →](/zh/guides/claude-code-and-api-safety/)

产品动态 2026年10月7日

### [Claude Haiku 5.5 发布，定价 $0.10/$0.50；Sonnet 5.5 缓存读取降至 $0.10；Max 与 Team 新增每月 API 额度](https://www.anthropic.com/claude-haiku-5-5)

Anthropic 发布 Claude Haiku 5.5（`claude-haiku-5-5`）：1M 上下文、128k 最大输出，支持带 `effort` 参数的 adaptive thinking；100k token 以内的 prompt 定价 $0.10 输入 / $0.50 输出 per MTok，超过 100k 为 $0.50 / $2.50。官方称其平均成本比 Haiku 4.5 低约 75%，面向 subagent、摘要、compaction 与 browser use；复杂 agentic 编程仍推荐 Sonnet 5.5 与 Opus 5.5。为 Haiku 4.5 编写的代码可能失效：手动 extended thinking（`budget_tokens`）返回 400，adaptive thinking 默认开启，新 tokenizer 对相同文本计更多 token。同日，Sonnet 5.5 缓存读取由 $0.20 降至 $0.10 per MTok；Claude Max（5x 每月 $100，20x 每月 $200）与 Team（每月最高 $500，团队共享）开始发放每月 Claude Platform API 额度。Python 与 TypeScript SDK 同时新增 browser use 与 computer use 工具的 beta 类。

实战要点： 将低成本 subagent 与摘要调用迁到 Haiku 5.5 前，先移除 `budget_tokens`，并按新 tokenizer 重新估算 token 用量。Max 与 Team 管理员可按帮助中心文章领取 API 额度。

来源: [Anthropic ↗](https://www.anthropic.com/claude-haiku-5-5) [相关实战指南 →](/zh/guides/api-advanced-optimization/)

安全与风控 2026年10月7日

### [Managed Agents 的 `web_fetch` 现仅抓取会话中已出现过的 URL](https://platform.claude.com/docs/en/release-notes/overview)

为降低数据外泄风险，Claude Managed Agents 的 `web_fetch` 只抓取会话中已出现过的 URL：来自用户消息、`web_search` 结果，或此前已抓取的页面。仅出现在 Claude 自身输出、system prompt、附件文档，或 `bash`、`read`、MCP 等工具输出中的 URL，会返回 `url_not_in_prior_context` 错误。另外，在 `limited` 网络的云环境中，`allowed_hosts` 现同时约束 `web_search` 与 `web_fetch`；若已启用的网页工具的 `allowed_domains` 含不在 `allowed_hosts` 内的条目，创建或更新会话将返回 400。`allowed_hosts` 条目只匹配一个确切主机，除非以 `*.` 开头。

实战要点： Agent 需要抓取的 URL 请放进 `user.message` 事件发送，并在 `allowed_hosts` 中逐个列出主机——`example.com` 不覆盖 `docs.example.com`。

来源: [Anthropic Docs ↗](https://platform.claude.com/docs/en/release-notes/overview) [相关实战指南 →](/zh/guides/automation-safety-practices/)

安全与风控 2026年10月6日

### [Cyber Verification Program 扩展为三档访问权限，覆盖 Opus 5.5、Sonnet 5.5 与 Mythos 5.1](https://www.anthropic.com/news/cyber-verification-program)

Anthropic 将 Project Glasswing 与 Cyber Verification Program（CVP）合并为一个三档计划。Defense Access 覆盖 SOC 与事件响应、恶意软件逆向和漏洞分析，官方目标是数日内答复。Red Team Access 在此基础上增加授权渗透测试与红队演练，仅限组织申请，审核需数周，期间先开通 Defense Access。Specialized Access 拦截最少，仅限经验证、获授权测试电网、银行间转账等关键系统的组织，并与美国政府共同审核。入组组织必须允许数据留存；计划于今秋推出的 Enterprise Frontier Safeguards 将支持存放到客户自控云环境。普通可用模型保持保守的网络安全护栏，仍可用于代码审查、修复已知问题与告警分诊。现有 CVP 成员保留当前设置，并自动评估新模型的访问资格。

实战要点： 普通模型上被拦截的防御性安全工作属于资格问题，而非措辞问题：应申请对应档位，而不是反复改写请求。

来源: [Anthropic ↗](https://www.anthropic.com/news/cyber-verification-program) [相关实战指南 →](/zh/guides/claude-steganography-and-risk-model/)

政策与地区 2026年10月1日

### [10 月 1 日起香港用户集中反馈 Claude 账号被停用，Anthropic 尚未回应](https://aistify.com/claude-hong-kong-vpn-account-bans/)

自 2026 年 10 月 1 日起，香港用户反馈 claude.ai 出现「Account Suspended」或「Access Denied」，其中包括通过新加坡等受支持地区 VPN 出口访问的 Pro 付费用户。香港、中国内地与澳门不在 Anthropic 的支持地区名单内，台湾、新加坡、日本在名单内。帮助中心将「从不受支持的地区创建账户」列为封禁原因，VPN 出口位于受支持国家并不改变使用资格。一份公开的停用通知仅提到「可疑信号」与违反使用政策，未说明具体触发条件。媒体与社区分析指向共享机房 VPN IP、虚拟卡或非支持地区支付，以及频繁切换登录国家，但 Anthropic 未确认其中任何一项，受影响账号数量亦未知。有用户反馈数据导出只有账号信息而无对话记录；帮助中心说明，可导出的数据范围可能因违规类型而受限。

实战要点： 受支持国家的 VPN 出口不能让不支持地区的账号变得合规。被停用后：先导出数据，再到 claude.ai/restricted 如实申诉，申诉期间不要另注册新账号。

来源: [AIstify (citing SCMP) ↗](https://aistify.com/claude-hong-kong-vpn-account-bans/) [相关实战指南 →](/zh/guides/regional-access-strategy/)

API 变更 2026年9月30日

### [Claude Sonnet 4.5 进入弃用期，11 月 30 日退役；Admin API 在 SDK 与 ant CLI 中转正](https://platform.claude.com/docs/en/release-notes/overview)

Anthropic 宣布弃用 Claude Sonnet 4.5（`claude-sonnet-4-5-20250929`），Claude API 上的退役日期定为 2026 年 11 月 30 日，推荐迁移至 Claude Sonnet 5.5。在 Python SDK 1.10.0、TypeScript SDK 0.130.0、C# SDK 12.52.0、Go SDK 1.77.0、Java SDK 2.67.0、PHP SDK 0.53.0、Ruby SDK 1.75.0 与 `ant` CLI 1.37.0 中，组织信息、成员、邀请、工作区、API 密钥、速率限制、服务账号、workload identity federation、客户自管加密密钥与合规设置相关的 Admin API 方法已转正，可通过 `client.organization` 与 `ant organization` 调用；beta 路径继续保留。

实战要点： 请在 11 月 30 日前替换硬编码的 `claude-sonnet-4-5-20250929`，并先阅读 Sonnet 5.5 迁移指南：`thinking` 与强制 `tool_choice` 的行为与旧模型不同。

来源: [Anthropic Docs ↗](https://platform.claude.com/docs/en/release-notes/overview) [相关实战指南 →](/zh/guides/api-advanced-optimization/)

产品动态 2026年9月28日

### [Claude Sonnet 5.5 发布，价格仍为 $2/$10，thinking block 与账号绑定](https://www.anthropic.com/claude-sonnet-5-5)

Anthropic 发布 Claude Sonnet 5.5（`claude-sonnet-5-5`），为继 Opus 5.5 之后的第二款 Claude 5.5 模型：1M 上下文、128k 最大输出，速度比 Sonnet 5 快 30% 以上；价格仍为 $2 / $10 per MTok，但同样工作消耗的 token 更少，多数场景成本最高降低约 30%。从 Sonnet 5 迁移有五处不兼容：关闭前置 thinking 需改用 `thinking: {"type": "between_tools"}`（`high` 及以下 effort），不再使用 `"disabled"`；`tool_choice` 的 `any` 与 `tool` 返回 400；thinking block 与模型和会话绑定；旧版 `computer_20251124` 工具在 Claude API 与 Google Cloud 上被拒绝；advisor 工具不再接受 Opus 4.8、Opus 4.7 与 Sonnet 5 作为 advisor。Sonnet 5.5 生成的 thinking block 仅在生成它的账号或与之关联的账号中有效；其他账号发送时，API 会在模型看到之前丢弃该 block，请求本身仍然成功。Claude Code 2.1.284 起，Anthropic API 上的 `sonnet` 别名指向 Sonnet 5.5。

实战要点： 在同一会话中轮换多个账号的 API 中转或网关，可能会悄悄丢失 Sonnet 5.5 的 thinking block。请让每个会话固定使用同一个账号。

来源: [Anthropic ↗](https://www.anthropic.com/claude-sonnet-5-5) [相关实战指南 →](/zh/guides/multi-account-management/)

产品动态 2026年9月25日

### [Claude Code 2.1.283：Enterprise 模型白/黑名单与第三方默认 auto mode](https://github.com/anthropics/claude-code/releases/tag/v2.1.283)

Claude Code 2.1.283 新增托管设置 `availableModelsMatch`（`exact` 仅允许列表中的具体模型版本，直至管理员更新）与 `deniedModels`（可单独封禁某模型）。在第三方 API、Vertex、Bedrock 或 Foundry 且关闭 telemetry 时，若未配置 `permissions.defaultMode`，新会话默认进入 auto mode。其他要点：`/doctor prompt-audit` 审计 CLAUDE.md 与 skill 是否沿用旧模型写法；MCP 返回的图片会落盘供后续工具读取；`claude plugin validate` 会拒绝无法安装的 marketplace 命名。

实战要点： Enterprise 建议在敏感仓库同时配置 `deniedModels` 与明确的 `permissions.defaultMode`——auto mode 已不仅限于 claude.ai 订阅环境。

来源: [Anthropic ↗](https://github.com/anthropics/claude-code/releases/tag/v2.1.283) [相关实战指南 →](/zh/guides/claude-code-and-api-safety/)

API 变更 2026年9月24日

### [Anthropic 恢复对生物、前沿 LLM 与推理提取类「零输出拒答」的 API 计费](https://platform.claude.com/docs/en/release-notes/overview)

自 2026 年 9 月 24 日起，当 `stop_details.category` 为 `bio`、`frontier_llm` 或 `reasoning_extraction` 时，Messages API 在尚未输出任何内容即拒答的请求将重新计入账单——Anthropic 称这三类误报率最低。中途拒答此前已计费；其余类别的「零输出拒答」仍免费，fallback 额度规则不变。费用按实际执行模型的常规定价，全平台生效。

实战要点： 涉及生物或能力探测的 prompt 若触发分类器硬拒，即使 assistant 为空也可能按完整输入计费，请在成本模型里预留这部分。

来源: [Anthropic Docs ↗](https://platform.claude.com/docs/en/release-notes/overview) [相关实战指南 →](/zh/guides/api-advanced-optimization/)

API 变更 2026年9月24日

### [Compliance API Activity Feed 不再返回文件名与 artifact 标题](https://platform.claude.com/docs/en/release-notes/overview)

Compliance API Activity Feed 已停止在事件中暴露文件名、项目文档名与 artifact 标题。文件、项目文档与 artifact 类活动的 `filename`、`title` 字段现恒为空或省略，历史记录亦同。Enterprise 若需可读名称，须用带 `read:compliance_user_data` 范围的 Compliance Access Key 按 ID 另行查询。

实战要点： 若 SIEM 或审计面板曾依赖 Activity Feed 内嵌标题，请改为只存 activity ID，或额外调用 compliance 用户数据端点解析。

来源: [Anthropic Docs ↗](https://platform.claude.com/docs/en/release-notes/overview) [相关实战指南 →](/zh/guides/automation-safety-practices/)

API 变更 2026年9月23日

### [Messages API 缓存诊断（cache diagnostics）转正 GA](https://platform.claude.com/docs/en/release-notes/overview)

Claude API 上的缓存诊断已 GA，不再需要 `cache-diagnosis-2026-04-07` beta 头。在 Messages 请求中加入 `diagnostics` 对象即可启用；所有 `POST /v1/messages` 响应现均含 `diagnostics` 字段（未启用时为 `null`）。仍发送旧 beta 头的请求保持原有行为。

实战要点： 多轮 Agent 请在每一轮都带上 `diagnostics`；仅发 beta 头不再写入 fingerprint，后续 `previous_message_id` 诊断会失效。

来源: [Anthropic Docs ↗](https://platform.claude.com/docs/en/release-notes/overview) [相关实战指南 →](/zh/guides/api-advanced-optimization/)

产品动态 2026年9月22日

### [Claude Opus 5.5 发布：$4/$20 定价，典型 workload 较 Opus 5 约省 40%](https://www.anthropic.com/news/claude-opus-5-5)

Anthropic 发布 Claude Opus 5.5（`claude-opus-5-5`），为 Claude 5.5 系列首款：1M 上下文、128k 最大输出、始终开启 adaptive thinking，定价 $4 / $20 per MTok，缓存读取 $0.20 / MTok（Opus 5 为 $5 / $25 与 $0.50）。官方称其多数 agentic 编程与知识工作接近 Fable 5.1，且单任务 token 更少。API 限制与 Fable 5.1 类似：不可关闭 `thinking`，`tool_choice` 的 `any`/`tool` 返回 400。Fast mode 为 research preview。Pro / Max / Team 与按席位 Enterprise 提升五小时额度，并提供可自选时机的 rate limit 重置。

实战要点： 长周期编程 Agent 可从 Opus 5 迁来降本；切换 model ID 前请移除关闭 thinking 与强制 `tool_choice` 的写法。

来源: [Anthropic ↗](https://www.anthropic.com/news/claude-opus-5-5) [相关实战指南 →](/zh/guides/api-advanced-optimization/)

产品动态 2026年9月18日

### [Claude Code 2.1.277 支持 AGENTS.md、移除 TaskOutput，并修复无头会话](https://github.com/anthropics/claude-code/releases/tag/v2.1.277)

Claude Code 2.1.277 在无 `CLAUDE.md` 的项目中改读 `AGENTS.md`（可在「Project instructions」配置；Bedrock / Vertex / Foundry 尚未支持）。已弃用的 TaskOutput 工具被移除，请用 Read 读取后台任务输出文件。网关 egress 可设 `CLAUDE_GATEWAY_PROXY_IS_EGRESS_BOUNDARY=1`，让代理收到主机名而非本地解析结果。另含大量修复：`claude -p` 挂起、空 text block 导致 resume 失败、插件重装竞态，以及无头 resume 后费用统计归零等。

实战要点： 每个仓库固定选用 CLAUDE.md 或 AGENTS.md 之一；混用或留空会改变会话启动时加载的项目指令。

来源: [Anthropic ↗](https://github.com/anthropics/claude-code/releases/tag/v2.1.277) [相关实战指南 →](/zh/guides/automation-safety-practices/)

安全与风控 2026年9月17日

### [Life Sciences Verification Program 开放，向合规生命科学团队放宽生物类护栏](https://www.anthropic.com/news/life-sciences-verification-program)

Anthropic 正式开放 Life Sciences Verification Program（LSVP），面向通过资质、安全与科研监督审查的团队。Standard Use 授权覆盖 API、Claude Science、claude.ai 与 Claude Code 上多数研发流程，可在 Mythos 5.1、Opus 5、Sonnet 5 及后续模型上使用针对合规生物任务调优的分类器。High-risk Use 附加授权针对单个项目移除生命科学拦截，每半年续期。执法更多转向离线行为监测，LSVP 流量强制 30 天留存；网络安全类分类器仍生效。

实战要点： 消费级 Fable 上被拦的常规生物问答并非 bypass 漏洞；合规实验室应走 LSVP 审核，而非 prompt 越狱。

来源: [Anthropic ↗](https://www.anthropic.com/news/life-sciences-verification-program)

API 变更 2026年9月14日

### [Messages API 按需会话压缩（compaction）进入 beta](https://platform.claude.com/docs/en/release-notes/overview)

在 `compact-2026-09-04` beta 头下，Messages API 可通过顶层 `compaction` 参数按需压缩会话。API 返回带签名的 `compaction` 块以概括已发送消息；后续轮次用该块替代完整历史。调用方自行决定压缩时机，可后台执行，并可在摘要之后保留最近若干轮原文——在支持的模型上，这些轮次中的 preserved thinking 仍可保持有效。

实战要点： 长 Agent 循环优先用官方 compaction，勿手写未签名摘要——在 8 月 31 日后新建账号上，自制摘要仍会破坏 thinking block 绑定。

来源: [Anthropic Docs ↗](https://platform.claude.com/docs/en/release-notes/overview) [相关实战指南 →](/zh/guides/api-advanced-optimization/)

安全与风控 2026年9月10日

### [2026 年 9 月威胁情报：Agent 化网络行动、蒸馏滥用与 Claude Code 参与国家级行动](https://www.anthropic.com/threat-intelligence-report-september-2026)

Anthropic 2026 年 9 月威胁情报报告汇总 2025 年 12 月至 2026 年 8 月间被处置的滥用，涵盖网络行动、影响操作、诈骗、监控、生物、武器与蒸馏等七类。Haiku、Sonnet、Opus 曾被滥用；Fable 与 Mythos 级模型未卷入（仅一起非法蒸馏例外）。案例包括多 Agent 网络杀伤链——有行动者通过调整 Claude Code skill 在检测触发后自动重构恶意软件——并再次表明高并发自动化能力提取仍是执法重点。

实战要点： Claude Code 的 auto mode 与泄露 API Key 均属高风险面——官方已将 skill 驱动工作流与国家级间谍行动关联披露。

来源: [Anthropic Threat Intelligence ↗](https://www.anthropic.com/threat-intelligence-report-september-2026) [相关实战指南 →](/zh/guides/claude-code-and-api-safety/)

产品动态 2026年9月10日

### [Claude Managed Agents 支持服务端 `auto` 权限评估；`ant beta:sessions connect` 上线](https://platform.claude.com/docs/en/release-notes/overview)

Managed Agents 权限策略新增 `auto`：服务端逐条评估 agent 或 MCP 工具调用并执行、拒绝或暂停待审批。`agent.tool_use` 与 `agent.mcp_tool_use` 事件在 `evaluated_permission` 旁新增 `evaluation` 字段。Anthropic CLI 1.32.0 提供 `ant beta:sessions connect`，可在终端接入 Managed Agents 会话、发送消息并审批挂起工具调用；加 `--web` 可在本地打开 Console 会话查看器。

实战要点： 在托管沙箱中运行不可信 MCP 时，优先用服务端 `auto` 评估，而非仅依赖客户端 allowlist。

来源: [Anthropic Docs ↗](https://platform.claude.com/docs/en/release-notes/overview) [相关实战指南 →](/zh/guides/automation-safety-practices/)

政策与地区 2026年9月3日

### [官方封号申诉专用入口 claude.ai/restricted 已启用](https://claude.ai/restricted)

Anthropic 现已将通过 claude.ai/restricted 专用表单处理被封账号与组织禁用申诉，不再依赖通用 support ticket。表单收集账号邮箱、封禁背景与用途说明。2026 年 9 月社区反馈显示：申诉中明确陈述合法软件开发或学术用途并提供可验证细节时，1–3 个工作日内有成功解封案例。

实战要点： Disabled 状态走 claude.ai/restricted；API 层级警示与误判走 usersafety@anthropic.com。

来源: [Anthropic ↗](https://claude.ai/restricted) [相关实战指南 →](/zh/guides/account-appeal-and-recovery-sop/)

产品动态 2026年9月2日

### [Claude Code 2.1.259 上线组织托管 MCP 与无头权限控制](https://github.com/anthropics/claude-code/releases/tag/v2.1.259)

Claude Code 2.1.259 新增 `managedMcpServers`：组织可向所有用户下发 HTTP/SSE MCP 服务器（格式同 `.mcp.json`，需执行命令的条目会被跳过）。`allowedMcpServers` 现仅约束用户自行添加的服务器。CI 与无头环境可用 `--permission-prompts none` 自动拒绝一切需弹窗确认的操作，当前权限模式（含 auto mode）仍生效。工具摘要亦识别 GitLab MR 命令（`glab mr create/merge/close/reopen/note/update`）。

实战要点： Enterprise 管理员应通过托管设置集中下发受信 MCP 端点，避免依赖用户各自维护 `.mcp.json`。

来源: [Anthropic ↗](https://github.com/anthropics/claude-code/releases/tag/v2.1.259) [相关实战指南 →](/zh/guides/automation-safety-practices/)

安全与风控 2026年9月1日

### [2026-08-31 后新建账号限制 Thinking block 重放与上下文篡改](https://platform.claude.com/docs/en/release-notes/overview)

自 2026 年 8 月 31 日起，该日期及之后创建的账号在多轮 API 会话中对 adaptive thinking block 执行更严格的绑定校验。重放、重排或篡改已存储上下文中的 thinking block 会被视为蒸馏信号，可能触发组织级禁用。这与 Anthropic 持续打击模型蒸馏与自动化能力提取的风控方向一致。

实战要点： Agent 流水线切勿缓存并重放原始 thinking block；仅保存最终 assistant 文本，每轮重新推理。

来源: [Anthropic Docs ↗](https://platform.claude.com/docs/en/release-notes/overview) [相关实战指南 →](/zh/guides/automation-safety-practices/)

产品动态 2026年9月1日

### [Claude Fable 5.1 与 Mythos 5.1 发布，缓存读取降价](https://platform.claude.com/docs/en/release-notes/overview)

Anthropic 发布 Claude Fable 5.1（`claude-fable-5-1`）与 Claude Mythos 5.1（`claude-mythos-5-1`）：1M 上下文、128k 最大输出、始终开启 adaptive thinking，定价 $10/$50 per MTok，缓存读取降至 $0.25/MTok（为输入价的 0.025×，其他模型为 0.1×）。Fable 5.1 在 API 与云伙伴平台 GA；Mythos 5.1 仍限可信访问计划。API 变更包括不支持 `tool_choice` 的 `any`/`tool`、2026-08-31 后新建账号的更严格 thinking block 重放绑定，以及 beta 的逐消息 effort 与 turn-scoped system message。Claude Code 2.1.257 将 Fable 5.1 设为默认 Fable 模型。

实战要点： 缓存命中率高的 Agent 工作负载从 Fable 5.1 获益最大；生产迁移前请检查 `tool_choice` 与多轮 thinking block 重放逻辑。

来源: [Anthropic Docs ↗](https://platform.claude.com/docs/en/release-notes/overview) [相关实战指南 →](/zh/guides/api-advanced-optimization/)

产品动态 2026年8月27日

### [Model Hardware Standard 研究预览向实验室与制造商开放](https://www.anthropic.com/news/model-hardware-standard-research-preview)

Anthropic 开放 Model Hardware Standard（MHS）研究预览——一套供 AI Agent 操作显微镜、移液工作站、机械臂等物理设备的共享规范。MHS 与模型无关，可通过 MCP、CLI 或代码 API 接入；在开源发布前，仅向科研实验室与先进制造商邀请开放。早期伙伴反馈设备集成更快、实验迭代更敏捷，并支持在线故障检测。

实战要点： MHS 尚未开源；实验室与制造团队应通过 modelhardwarestandard.com 申请接入，而非各自编写临时设备驱动。

来源: [Anthropic ↗](https://www.anthropic.com/news/model-hardware-standard-research-preview)

API 变更 2026年8月27日

### [Files / Skills SDK 路径转正 GA，个人密钥与服务账号密钥上线](https://platform.claude.com/docs/en/release-notes/overview)

Python SDK 1.2.0、TypeScript SDK 0.122.0 及对应 Go/Java/Ruby/C# 版本中，`client.beta.files` 与 `client.beta.skills` 不再发送 beta 请求头，行为与 `client.files`、`client.skills` 一致。Beta 类型 `BetaSkill` 重命名为 `BetaContainerSkill`，`client.beta.skills.delete()` 会删除 Skill 及其全部版本。Claude Console 同时支持个人密钥与服务账号密钥，可按工作区或管理员端点授权。

实战要点： 从生产 SDK 调用中移除 `files-api-2025-04-14` 与 `skills-2025-10-02` beta 头，并迁移至稳定 client 路径。

来源: [Anthropic Docs ↗](https://platform.claude.com/docs/en/release-notes/overview) [相关实战指南 →](/zh/guides/api-advanced-optimization/)

产品动态 2026年8月26日

### [Cowork 内置浏览器上线；Claude in Chrome GA 并支持自主操作](https://claude.com/blog/cowork-built-in-browser)

Claude Cowork 桌面版现可在侧栏打开独立浏览器处理网页任务，无需扩展；向 Pro / Max / Team 逐步推送（macOS / Windows / Linux beta），Enterprise 管理员可在组织设置中管控。同期 Claude in Chrome 向全部付费套餐 GA：Claude 可在用户 Chrome 会话中读取页面、导航并自主操作，安全分类器逐条校验动作。Enterprise 扩展默认将于 2026-09-10 改为开启（若此前未手动关闭）。

实战要点： 隔离式网页任务用内置浏览器，需复用已登录会话时用 Claude in Chrome；请在 9 月 10 日前确认组织的浏览器策略。

来源: [Anthropic ↗](https://claude.com/blog/cowork-built-in-browser) [相关实战指南 →](/zh/guides/automation-safety-practices/)

API 变更 2026年8月26日

### [Compliance API 会话端点转正；Admin API 登陆 SDK 与 ant CLI](https://platform.claude.com/docs/en/release-notes/overview)

Cowork 与 Claude Code 的 Compliance API 会话端点已 GA。本地会话端点亦返回 Claude Science 与 Claude for Microsoft 365 会话记录（beta，Enterprise）。Admin API 已在 `ant` CLI 及 Python、TypeScript、C#、Go、Java、PHP、Ruby SDK 的 `client.beta.organization` 下提供，涵盖成员、工作区、API 密钥、速率限制与服务账号管理。

实战要点： Enterprise 管理员可通过 SDK Admin API 自动化组织治理，无需再依赖纯 curl 流程。

来源: [Anthropic Docs ↗](https://platform.claude.com/docs/en/release-notes/overview) [相关实战指南 →](/zh/guides/automation-safety-practices/)

产品动态 2026年8月25日

### [Claude Cowork 记忆跨 Chat 同步，Topics 可编辑并支持敏感主题控制](https://docs.anthropic.com/en/release-notes/claude-apps)

记忆功能现已在 Chat 与云端 Cowork 间同步。所有记忆条目可在 Settings > Memory 的 Topics 中查看、编辑或删除。健康、信仰等敏感主题默认不写入记忆，需开启「Include sensitive topics」才会收录。Free / Pro / Max 默认开启记忆；Team / Enterprise 默认关闭。

实战要点： 向 Team / Enterprise 用户推广 Cowork 前，请先确认组织的记忆功能默认策略。

来源: [Anthropic Help Center ↗](https://docs.anthropic.com/en/release-notes/claude-apps)

API 变更 2026年8月20日

### [Python SDK v1.0 发布（迁移 httpx2，移除旧 API）](https://platform.claude.com/docs/en/release-notes/overview)

Anthropic 发布 Python SDK v1.0：HTTP 层从 httpx 迁至维护中的 httpx2 分支（需 Python 3.10+）。移除长期弃用接口，含旧 Text Completions API、Messages 上的 `temperature`/`top_p`/`top_k`，以及 tool runner 客户端 `compaction_control`。异步 `.with_raw_response` 现需 `await response.parse()`。完整破坏性变更见 v1 迁移指南。

实战要点： 在生产环境升级 SDK 前，请确保所有旧调用已彻底从 /v1/complete 迁移至 /v1/messages。

来源: [Anthropic Docs ↗](https://platform.claude.com/docs/en/release-notes/overview) [相关实战指南 →](/zh/guides/claude-code-and-api-safety/)

API 变更 2026年8月19日

### [Computer use、browser use、Files API 与 Agent Skills 正式 GA](https://platform.claude.com/docs/en/release-notes/overview)

Anthropic 将四项平台能力正式 GA：computer use（`computer_toolset_20260801`）与 browser use（`browser_toolset_20260801`）不再需要 beta header；Files API 与 Agent Skills（`/v1/skills`）亦同步转正。四项能力均已在 Fable 5、Mythos 5、Opus 5、Sonnet 5 与 Opus 4.8 上提供。

实战要点： 在调用 Computer Use 与工具集时，可以从生产代码中安全移除相关的 anthropic-beta 请求头。

来源: [Anthropic Docs ↗](https://platform.claude.com/docs/en/release-notes/overview) [相关实战指南 →](/zh/guides/automation-safety-practices/)

政策与地区 2026年8月14日

### [Claude 文本水印全球上线，配合 EU AI Act 透明度要求](https://www.anthropic.com/news/claude-text-watermark)

新 Claude 模型在生成文本中嵌入不可见的 SynthID-Text 统计水印，全球生效（非仅限欧盟），以符合欧盟 AI 法案第 50 条透明度规范。水印不含用户或组织标识、不增加 token 消耗，常规同义词替换难以破坏其统计特征。

实战要点： 如需识别和合规处理生成的文本水印，可查阅本站的 AI 内容水印深度指南。

来源: [Anthropic ↗](https://www.anthropic.com/news/claude-text-watermark) [相关实战指南 →](/zh/guides/claude-ai-content-watermarking/)

产品动态 2026年8月14日

### [Claude Code auto mode 成为 Pro / Max / Team 默认权限模式](https://claude.com/blog/auto-mode-default-in-claude-code)

自 8 月 14 日起，Pro / Max / Team 的新 Claude Code 会话默认进入 auto mode，由后台分类器自动放行常规工具调用而非逐条弹窗确认。上述套餐不再收取分类器 token 开销；Enterprise 与 API 接入仍为 opt-in。

实战要点： 若在敏感仓库需要人工审批，请在 `~/.claude/settings.json` 中固定 `permissions.defaultMode`。

来源: [Anthropic ↗](https://claude.com/blog/auto-mode-default-in-claude-code) [相关实战指南 →](/zh/guides/claude-code-and-api-safety/)

API 变更 2026年8月10日

### [Claude Sonnet 优惠价 $2/$10 定为永久标准定价](https://platform.claude.com/docs/en/about-claude/pricing)

Anthropic 取消原定涨价计划。Sonnet API 永久维持 $2 input / $10 output per MTok 的标准价，为高频自动化开发提供极具性价比的基础模型选择。

实战要点： 搭配 Prompt Caching，Sonnet 命中缓存时的实际输入成本可低至 $0.20 / MTok。

来源: [Anthropic Docs ↗](https://platform.claude.com/docs/en/about-claude/pricing) [相关实战指南 →](/zh/guides/api-advanced-optimization/)

安全与风控 2026年8月7日

### [Fable 5 生物安全护栏优化，误报 fallback 减少约 85%](https://www.anthropic.com/news/improving-fable-5-s-biology-safeguards)

Anthropic 重调 Fable 5 生物安全分类器，日常健康与教育类问题更少误触发 fallback 至 Opus 5——测试中生物相关 fallback 减少约 85%。双重用途请求（病毒学、毒理学、分子设计等）仍会路由至能力较低的模型；面向前沿生物能力的可信访问通道仍在建设中。

实战要点： 常规生物医学学习与问答不再容易触发异常降级，保障了学术研究的连贯性。

来源: [Anthropic ↗](https://www.anthropic.com/news/improving-fable-5-s-biology-safeguards) [相关实战指南 →](/zh/guides/claude-steganography-and-risk-model/)

安全与风控 2026年8月6日

### [Enterprise 技能与插件安全扫描（beta）上线](https://docs.anthropic.com/en/release-notes/claude-apps)

Enterprise 套餐可开启技能与插件安全扫描，在用户上传或编辑第三方 Skill / Plugin 时自动检测恶意内容。

实战要点： 在允许成员安装社区 Skill 前，先在组织设置中启用扫描策略。

来源: [Anthropic Help Center ↗](https://docs.anthropic.com/en/release-notes/claude-apps) [相关实战指南 →](/zh/guides/automation-safety-practices/)

API 变更 2026年8月5日

### [Claude Opus 4.1 已退役，需迁移至 Opus 4.8 或 Opus 5](https://platform.claude.com/docs/en/about-claude/model-deprecations)

Anthropic 在 60 天弃用期后停止支持 claude-opus-4-1-20250805。旧 model ID 的 API 调用现已报错；推荐迁移至 Opus 4.8 或 Opus 5。

实战要点： 请检查工程配置文件，及时替换硬编码的旧模型名称，避免运行时 400 报错。

来源: [Anthropic Docs ↗](https://platform.claude.com/docs/en/about-claude/model-deprecations) [相关实战指南 →](/zh/guides/troubleshooting-guide/)

产品动态 2026年7月24日

### [Anthropic 发布 Claude Opus 5，成为 Pro 与 Max 默认模型](https://www.anthropic.com/news/claude-opus-5)

Claude Opus 5（`claude-opus-5`）以约为 Fable 5 一半的价格提供接近前沿的智能，具备 1M 上下文、128k 最大输出，并默认开启 adaptive thinking。其为 Claude Max 默认模型、Pro 最强模型，定价 $5/$25 per MTok。Beta 功能包括对话中途切换工具集，以及安全分类器拒答时的 API 自动 fallback。

实战要点： 可启用 `fallbacks` 的 `"default"` 模式（beta），使被标记的 Opus 5 请求自动路由至备用模型而非硬阻断。

来源: [Anthropic ↗](https://www.anthropic.com/news/claude-opus-5) [相关实战指南 →](/zh/guides/domestic-and-open-source-alternatives/)

安全与风控 2026年7月23日

### [Anthropic 2026 上半年透明度报告：封禁 1140 万账号，申诉解封率约 10.5%](https://www.anthropic.com/transparency/system-trust-reporting)

Anthropic Transparency Hub 2026 上半年报告显示：半年内封禁 1140 万违规账号，收到 39.8 万起申诉，成功解封 4.2 万起（解封率约 10.5%）。主要风控针对：未支持地区访问、滥用检测、多账号批量注册与模型蒸馏攻击。被封账号（组织被禁用 / Disabled）需访问 claude.ai/restricted 填写官方申诉表单；API 违规警示与误判请联系 usersafety@anthropic.com。

实战要点： 提供合规学术/开发用途证明的申诉有明确解封概率 — 申诉期间勿慌乱重复建号。

来源: [Anthropic Transparency Hub ↗](https://www.anthropic.com/transparency/system-trust-reporting) [相关实战指南 →](/zh/guides/account-appeal-and-recovery-sop/)

产品动态 2026年6月30日

### [Anthropic 发布 Claude Sonnet 5（默认自适应思考，1M 上下文）](https://www.anthropic.com/news/claude-sonnet-5)

Claude Sonnet 5（`claude-sonnet-5`）是 Sonnet 4.6 的直接升级，支持 1M 上下文、128k 最大输出，并默认开启 adaptive thinking。手动 extended thinking 与非默认 `temperature`/`top_p`/`top_k` 现返回 400 错误。新 tokenizer 对相同文本约多计 30% token。首发定价 $2/$10 per MTok，8 月 10 日定为永久标准价。

实战要点： 从 Sonnet 4.6 迁移时移除 `thinking: {type: "enabled"}` 与采样参数，改用 `effort` 控制推理深度。

来源: [Anthropic ↗](https://www.anthropic.com/news/claude-sonnet-5) [相关实战指南 →](/zh/guides/api-advanced-optimization/)

产品动态 2026年6月9日

### [Anthropic 发布 Claude Fable 5 与受限版 Claude Mythos 5](https://www.anthropic.com/news/claude-fable-5-mythos-5)

Claude Fable 5（`claude-fable-5`）是 Anthropic 目前面向公众的最强模型，具备 1M 上下文、始终开启的 adaptive thinking，以及可能返回 `stop_reason: "refusal"` 的安全分类器。Claude Mythos 5 为同一基座、面向 Project Glasswing 伙伴放宽部分护栏的版本。6 月 12 日至 7 月 1 日曾因出口管制短暂停服，随后全球恢复。

实战要点： 可通过 opt-in 的 `fallbacks` 参数（beta）将被拒 Fable 5 请求路由至备用模型，避免硬阻断。

来源: [Anthropic ↗](https://www.anthropic.com/news/claude-fable-5-mythos-5) [相关实战指南 →](/zh/guides/claude-steganography-and-risk-model/)

## 社区防封实战精选

来自 X (Twitter) 的 Claude 封号、申诉、支付与 Claude Code 第一手经验，附原文直达链接与实战避坑提炼。

AYi

@AYi_AInotes

2026年10月4日

### 国庆封号反馈后的万字防封指南：环境、支付、使用行为与本机设置

一篇按四层组织的 X 长文。环境：每个账号固定一个出口节点、开启 TUN 模式、系统时区与出口地区一致，并检查 WebRTC、DNS 与代理残留。账号与支付：优先走 App Store 内购以保留退款退路，按 Free → Pro → Max 的顺序升级。使用行为：不要把周额度刷空。Claude Code：只用官方客户端，并在 `settings.json` 中设置遥测开关。文末给出 11 步最低稳法，以及申诉与退款的处理顺序。原帖含第三方 IP 与银行卡服务的推广链接，本站不做背书。

实战要点： 每个账号固定一个出口节点和一台设备，每次登录前检查 WebRTC、DNS 与残留代理变量（`ANTHROPIC_BASE_URL`、`HTTPS_PROXY`）。代码保存在本地 Git，避免封号时连项目状态一起丢失。

基础设施 支付 Claude Code

相关指南: [Claude 防封速查手册：账号、API 与频率控制要点 →](/zh/guides/antiban-essentials/)

[在 X (Twitter) 查看原文 ↗](https://x.com/AYi_AInotes/status/2106639522586829094) 将在新标签页打开外部链接

实践哥 Li

@MinLiBuilds

2026年10月2日

### 两个号同日被封且提示不同：只盯网络不够

作者向朋友传授防封经验后，自己的两个账号同日被封。一个提示登录国家不对，另一个提示违反 Usage Policy。作者表示自己常让 Claude 处理反编译、成人内容调研等边缘任务，怀疑封号不只与网络有关，内容也是原因之一。

实战要点： 封号有两条独立通道：访问信号（地区、登录国家）与内容（Usage Policy）。网络环境再干净，也挡不住提示词触发策略分类器。

使用行为

相关指南: [Claude 封号案例复盘与避坑经验 →](/zh/guides/ban-case-studies/)

[在 X (Twitter) 查看原文 ↗](https://x.com/MinLiBuilds/status/2105890192108536029) 将在新标签页打开外部链接

Ray Wang

@wangray

2026年10月1日

### 国庆封号反馈期间分享：半年稳定使用的配置

分享过去半年未被封的配置：独立 Mac 主机、美西时区、单账号从未切换、Surge 规则模式常开、夏威夷独立家宽 IP、美国信用卡、每周用量 50%–80%，外出时远程访问主机发起任务。作者强调不存在 100% 防封的方法，稳定 IP 加非高频使用大概率能降低风险。

实战要点： 保持稳定、少变化：一个账号、一台设备、一个出口 IP，周额度留出余量。

基础设施 支付 使用行为

相关指南: [操作系统/浏览器环境纯化与原生住宅 IP 配置指南 →](/zh/guides/environment-cleanup-and-ip-setup/)

[在 X (Twitter) 查看原文 ↗](https://x.com/wangray/status/2105658146685608265) 将在新标签页打开外部链接

陈大黄

@realchendahuang

2026年9月28日

### 几十位用户反馈整理：网络泄漏、账单信用、调用节奏三层防线

整理几十位用户的反馈。不少封号的诱因是 DNS 或 WebRTC 暴露了真实位置，而不是 IP 类型。账单信用的权重不低于网络：网页端绑定国内双币卡或虚拟卡是封号高发场景，而用美区 Apple ID 走 App Store 内购可以隔离发卡行信息。新号需要缓冲期，周额度也不要每次耗尽。作者认为语言不是判定指标，网络指纹与调用频率才是。

实战要点： 每次登录前做泄漏自查（WebRTC、DNS、系统语言、时区），避开网页端虚拟卡支付，新号别在几天内把额度刷干，留出余量。

基础设施 支付 使用行为

相关指南: [Claude 防封速查手册：账号、API 与频率控制要点 →](/zh/guides/antiban-essentials/)

[在 X (Twitter) 查看原文 ↗](https://x.com/realchendahuang/status/2104455272614035947) 将在新标签页打开外部链接

Kieran Ji

@kieranji404

2026年9月5日

### Claude 第二次封号 — 排查原因为虚拟 U 卡支付

Claude 第二次被封号。排除 IP 与注册问题后，大概率因使用虚拟 U 卡支付触发风控。建议尽量使用 Wise 入金或真实借记卡。

实战要点： 避免高风险虚拟卡/U 卡卡头，推荐 Wise 境外账户或真实 Visa/Mastercard 借记卡。

支付

相关指南: [Claude 支付方式、银行卡风险与扣款失败排查 →](/zh/guides/payment-methods-comparison/)

[在 X (Twitter) 查看原文 ↗](https://x.com/kieranji404/status/2096384314992910370) 将在新标签页打开外部链接

林微明

@OrientLinden

2026年9月5日

### 多账号容灾策略 — 独立隔离 IP 养号

分享多账号容灾策略：使用独立隔离 IP 分别养 Claude 与 ChatGPT 备用号，防止主号遭遇风控阻断生产业务。

实战要点： 生产环境严禁单点依赖，建立隔离环境的主备账号矩阵。

多账号 基础设施

相关指南: [多账号管理最佳实践：环境隔离与安全切换 →](/zh/guides/multi-account-management/)

[在 X (Twitter) 查看原文 ↗](https://x.com/OrientLinden/status/2096270753968628132) 将在新标签页打开外部链接

Yifan Xu

@yifanxu_ephai

2026年9月5日

### Claude Code 频次与 Token 消耗风控体验

讨论高频工具调用与自动化请求触发模型拒答或组织禁用的边界。短时间超高频 tool_use 比持续中等负载更危险。

实战要点： 控制 Claude Code 自动化频率与单次并发，避免短时间超高频 tool_use。

Claude Code

相关指南: [Claude 自动化与批量调用安全规范 →](/zh/guides/automation-safety-practices/)

[在 X (Twitter) 查看原文 ↗](https://x.com/yifanxu_ephai/status/2096315059240894701) 将在新标签页打开外部链接

芊羽Wing

@qianyuwing

2026年9月4日

### Claude Code 封号潮下的稳定注册经验

针对近期 Claude Code 封号潮的稳定经验：采用几个月前注册的干净 Gmail、绑定真实美区手机号，辅助邮箱绑定老 Gmail。

实战要点： 注册环境与邮箱权重至关重要，避免临时虚拟号接码。

注册 Claude Code

相关指南: [Claude 账号注册避坑、虚拟卡 BIN 风险与订阅支付指南 →](/zh/guides/account-registration-and-payment-antiban/)

[在 X (Twitter) 查看原文 ↗](https://x.com/qianyuwing/status/2096175929911296432) 将在新标签页打开外部链接

Raymond Zhu

@raymondzhu

2026年9月4日

### 官方申诉渠道提交后成功解封，分享解封通知邮件

在官方申诉渠道提交后成功解封 Claude 账号，分享解封通知邮件。申诉中提供了合规学术/开发用途证明。

实战要点： 申诉时附上可验证的学术或开发用途证明，让审核方有可核对的内容。

申诉解封

相关指南: [Claude 封号类型判定与英文申诉自救 SOP →](/zh/guides/account-appeal-and-recovery-sop/)

[在 X (Twitter) 查看原文 ↗](https://x.com/raymondzhu/status/2096210657184207105) 将在新标签页打开外部链接

Neo

@NeoAiCompiler

2026年9月3日

### 封号第 2 天通过官方申诉顺利恢复

Claude 封号第 2 天通过官方申诉顺利恢复账号。申诉期间不要重复建号。

实战要点： 封号后不要慌乱重复建号，通过 claude.ai/restricted 标准英文申诉流程有明确成功解封概率。

申诉解封

相关指南: [Claude 封号类型判定与英文申诉自救 SOP →](/zh/guides/account-appeal-and-recovery-sop/)

[在 X (Twitter) 查看原文 ↗](https://x.com/NeoAiCompiler/status/2096081821771669534) 将在新标签页打开外部链接

DaoDaodl

@JichuanT45327

2026年9月3日

### Claude Code 全局部署防封指南

Claude Code 全局部署防封指南：配置纯净美国双 ISP 静态住宅 IP + Windows/Mac 系统时区与 WebRTC 防泄漏。数据中心机房 IP 是封号重灾区。

实战要点： 数据中心机房 IP 是封号重灾区，Claude Code 需搭配住宅家宽 IP 与本地系统时区完全对齐。

基础设施 Claude Code

相关指南: [Claude Code 安全配置与第三方 API 中转平滑降级规范 →](/zh/guides/claude-code-and-api-safety/)

[在 X (Twitter) 查看原文 ↗](https://x.com/JichuanT45327/status/2096151684602904651) 将在新标签页打开外部链接
