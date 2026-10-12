# 你是「Claude 中国用户」吗

*此网站使用 Claude Fable 5 开发*

**本地指纹检测器：** 初始状态 `0/ 100` · 待检测。操作按钮：开始检测、保存结果图片。卖点标注：纯本地检测 / 结果零上传 / 开源代码。

## 检测哪些信号

十项区域与网络指纹，加权得出 0–100 风险分。

1. **系统时区** 权重 24 — Claude 同款。Intl.DateTimeFormat 读到的就是 Claude Code 读取的同一个系统时区，与 Asia/Shanghai、Asia/Urumqi 等中国时区比对。
2. **浏览器语言** 权重 18 — 检查 navigator.languages；首选 zh-CN / 简体中文得分最高。
3. **已安装中文字体** 权重 14 — 用 canvas 宽度探测微软雅黑、苹方等简繁中文字体。
4. **国产厂商字体** 权重 10 — 用 canvas 探测 MiSans、鸿蒙黑体、OPPO Sans、WPS 方正字体等国产厂商 / 软件字体，命中即为强信号。
5. **WebRTC IP 泄露** 权重 10 — 探测浏览器 RTCPeerConnection 是否通过 STUN 服务泄露真实内网或公网 IP。
6. **国产浏览器 / WebView** 权重 8 — 用 UA 与 UA-CH brands 匹配微信、QQ、夸克、UC、百度等国产浏览器或应用内 WebView。
7. **国产品牌设备** 权重 6 — 用 UA-CH 设备型号(navigator.userAgentData)与 UA 匹配鸿蒙、华为、小米、OPPO、vivo 等国产品牌。
8. **Intl 区域设置** 权重 4 — 浏览器用于日期 / 数字格式化的 locale。
9. **时区偏移** 权重 3 — getTimezoneOffset() 是否为 UTC+8。
10. **Emoji 渲染风格** 权重 3 — 由 UA 推断操作系统厂商，弱相关信号。

## 检测原理

当 Claude Code 通过 ANTHROPIC_BASE_URL 指向中转端点时，据公开逆向分析，它会读取操作系统时区与中转 hostname，再把结果用 Unicode 隐写术藏进 system prompt:「Today’s date」那一行的日期分隔符和 4 种几乎一样的撇号变体，编码了你是否像中国用户。

网页读不到 Claude Code 能读的全部信息，但关键信号完全一致：本工具读取同一个系统时区，再叠加浏览器语言、中文字体、国产厂商字体、WebRTC IP 泄露、国产浏览器、设备品牌、Intl locale、UTC+8 偏移与 emoji 风格九项指纹，加权得分。得分 ≥0.25 计为命中；分档：低 0–30、中 31–60、高 61–100。

## 为什么会被 Claude 判为「中国用户」

Anthropic 并不在中国大陆销售 Claude，而 Claude Code 被曝会把「看起来像中国」的流量视为更高风险。这个判定并不是一个开关，而是把多个弱信号叠加成一个概率。系统时区是单项里最重的一环 —— 它伴随每一次请求，又很难被无意伪装；但语言、字体、设备品牌与网络都会在它之上继续加权。

整个浏览器环境之所以会泄露这么多，是因为区域设置会渗透到方方面面：操作系统时区决定了 Intl 与 Date，语言偏好决定了 navigator.languages 与 Accept-Language 请求头，而操作系统自带的字体可以被 canvas 逐像素探测。如果再让 Claude Code 走一层中转，中转的 hostname 本身又成了一个信号 —— 逆向分析发现，判定结果被隐写进了 system prompt 里「Today’s date」那一行。

这些信号单看没有一个是决定性的，而这恰恰是风险会累积的原因：一个美国时区，配上一整套中文字体和一个微信 WebView，照样会被读成可疑。隐写机制与完整风控模型拆解见 [隐写暗记与四维风控模型指南](https://fuck-claude.vercel.app/zh/guides/claude-steganography-and-risk-model/)，而具体的解决办法收在 [环境纯化与住宅 IP 配置指南](https://fuck-claude.vercel.app/zh/guides/environment-cleanup-and-ip-setup/)。

## 如何降低你的风险分

一份简明的有序清单。每一步都直接对应上方某一项加权信号 —— 请从上往下做，因为时区和语言的权重最高。

1. 把系统时区改成非中国时区(例如 America/New_York 或 Europe/London)，并确认 getTimezoneOffset() 不再是 UTC+8。逆向分析披露 Claude Code 直接读取的信号只有这一项，所以它在本页评分中权重最高。
2. 把 zh-CN / zh-Hans 从浏览器与系统语言列表的首位移除，让 navigator.languages 和 Accept-Language 以英文或其他非中文 locale 打头。
3. 隔离中文与厂商字体(微软雅黑、PingFang SC、MiSans、鸿蒙黑体)—— 使用干净的浏览器 Profile 或防指纹浏览器，让 canvas 字体探测找不到任何明显的中文痕迹。
4. 堵住 WebRTC 的 IP 泄露，并走原生住宅 IP 而非数据中心 IP，避免真实网络与地理位置盖过一个本来已经很干净的环境。
5. 通过中转使用 Claude Code 时，保持 ANTHROPIC_BASE_URL 的 hostname 中性 —— 避开含敏感关键词或 AI 实验室名称的域名，若宿主机在中国还需覆盖 TZ 环境变量。

完整的逐步操作收录在 [防封指南资料库](https://fuck-claude.vercel.app/zh/guides/).

## 也支持 curl 命令行

喜欢终端？请求下面的接口 —— 它会根据你的 IP 归属地 + 请求头估算风险，并按你的 Accept-Language 请求头返回对应语言。

```bash
# 文本报告 —— 跟随 Accept-Language
curl https://fuck-claude.vercel.app/api/check

# 通过请求头指定语言
curl -H "Accept-Language: zh" https://fuck-claude.vercel.app/api/check

# JSON 输出
curl https://fuck-claude.vercel.app/api/check?format=json
```

## Claude 最新动态

精选 Claude 产品更新、API 变更、地区政策与社区风控趋势，助你及时调整使用策略。

[查看全部动态 →](https://fuck-claude.vercel.app/zh/news/)

1. **产品动态** · 2026年10月8日

### [Claude Code 2.1.292–2.1.294：权限与沙箱修复、stdio MCP 协议协商、prompt hook 修复](https://code.claude.com/docs/en/changelog)

Claude Code 2.1.292 修复多处权限绕过：PreToolUse hook 的批准与 auto mode 会跳过网络（UNC）路径文件读取的权限确认；沙箱内命令可读取 `/ultrareview` 上传的暂存副本；服务端托管设置在拉取失败时，被篡改的本地缓存可关闭内置策略插件。该版本还让本地（stdio）MCP 服务器在所有安装环境（含 Bedrock、Vertex、Foundry）默认协商 2026-07-28 协议版本，可用 `MCP_PROTOCOL_NEGOTIATION=legacy` 退回旧行为。2.1.293 将 Claude Haiku 5.5 设为默认 Haiku 模型。2.1.294 修复以指令形式书写的 `prompt` 与 `agent` hook（如「Block commands that…」）未能拦截目标命令的问题，并改进 Stop 与 SubagentStop prompt hook 的判定。

实战要点：若依赖 `prompt` 或 `agent` hook 拦截命令，请升级到 2.1.294 并重新验证；旧版本在 hook 以指令形式书写时可能放行本应拦截的命令。

2. **产品动态** · 2026年10月7日

### [Claude Haiku 5.5 发布，定价 $0.10/$0.50；Sonnet 5.5 缓存读取降至 $0.10；Max 与 Team 新增每月 API 额度](https://www.anthropic.com/claude-haiku-5-5)

Anthropic 发布 Claude Haiku 5.5（`claude-haiku-5-5`）：1M 上下文、128k 最大输出，支持带 `effort` 参数的 adaptive thinking；100k token 以内的 prompt 定价 $0.10 输入 / $0.50 输出 per MTok，超过 100k 为 $0.50 / $2.50。官方称其平均成本比 Haiku 4.5 低约 75%，面向 subagent、摘要、compaction 与 browser use；复杂 agentic 编程仍推荐 Sonnet 5.5 与 Opus 5.5。为 Haiku 4.5 编写的代码可能失效：手动 extended thinking（`budget_tokens`）返回 400，adaptive thinking 默认开启，新 tokenizer 对相同文本计更多 token。同日，Sonnet 5.5 缓存读取由 $0.20 降至 $0.10 per MTok；Claude Max（5x 每月 $100，20x 每月 $200）与 Team（每月最高 $500，团队共享）开始发放每月 Claude Platform API 额度。Python 与 TypeScript SDK 同时新增 browser use 与 computer use 工具的 beta 类。

实战要点：将低成本 subagent 与摘要调用迁到 Haiku 5.5 前，先移除 `budget_tokens`，并按新 tokenizer 重新估算 token 用量。Max 与 Team 管理员可按帮助中心文章领取 API 额度。

3. **安全与风控** · 2026年10月7日

### [Managed Agents 的 `web_fetch` 现仅抓取会话中已出现过的 URL](https://platform.claude.com/docs/en/release-notes/overview)

为降低数据外泄风险，Claude Managed Agents 的 `web_fetch` 只抓取会话中已出现过的 URL：来自用户消息、`web_search` 结果，或此前已抓取的页面。仅出现在 Claude 自身输出、system prompt、附件文档，或 `bash`、`read`、MCP 等工具输出中的 URL，会返回 `url_not_in_prior_context` 错误。另外，在 `limited` 网络的云环境中，`allowed_hosts` 现同时约束 `web_search` 与 `web_fetch`；若已启用的网页工具的 `allowed_domains` 含不在 `allowed_hosts` 内的条目，创建或更新会话将返回 400。`allowed_hosts` 条目只匹配一个确切主机，除非以 `*.` 开头。

实战要点：Agent 需要抓取的 URL 请放进 `user.message` 事件发送，并在 `allowed_hosts` 中逐个列出主机——`example.com` 不覆盖 `docs.example.com`。

## 社区防封实战精选

来自 X (Twitter) 的 Claude 封号、申诉、支付与 Claude Code 第一手经验，附原文直达链接与实战避坑提炼。

[查看全部实战帖 →](https://fuck-claude.vercel.app/zh/news/#community-posts)

- **AYi** ([@AYi_AInotes](https://x.com/AYi_AInotes)) · 2026年10月4日

### 国庆封号反馈后的万字防封指南：环境、支付、使用行为与本机设置

一篇按四层组织的 X 长文。环境：每个账号固定一个出口节点、开启 TUN 模式、系统时区与出口地区一致，并检查 WebRTC、DNS 与代理残留。账号与支付：优先走 App Store 内购以保留退款退路，按 Free → Pro → Max 的顺序升级。使用行为：不要把周额度刷空。Claude Code：只用官方客户端，并在 `settings.json` 中设置遥测开关。文末给出 11 步最低稳法，以及申诉与退款的处理顺序。原帖含第三方 IP 与银行卡服务的推广链接，本站不做背书。

实战要点：每个账号固定一个出口节点和一台设备，每次登录前检查 WebRTC、DNS 与残留代理变量（`ANTHROPIC_BASE_URL`、`HTTPS_PROXY`）。代码保存在本地 Git，避免封号时连项目状态一起丢失。

标签：基础设施 · 支付 · Claude Code　|　相关指南：[Claude 防封速查手册：账号、API 与频率控制要点](https://fuck-claude.vercel.app/zh/guides/antiban-essentials/)　|　[在 X (Twitter) 查看原文 ↗](https://x.com/AYi_AInotes/status/2106639522586829094)

- **实践哥 Li** ([@MinLiBuilds](https://x.com/MinLiBuilds)) · 2026年10月2日

### 两个号同日被封且提示不同：只盯网络不够

作者向朋友传授防封经验后，自己的两个账号同日被封。一个提示登录国家不对，另一个提示违反 Usage Policy。作者表示自己常让 Claude 处理反编译、成人内容调研等边缘任务，怀疑封号不只与网络有关，内容也是原因之一。

实战要点：封号有两条独立通道：访问信号（地区、登录国家）与内容（Usage Policy）。网络环境再干净，也挡不住提示词触发策略分类器。

标签：使用行为　|　相关指南：[Claude 封号案例复盘与避坑经验](https://fuck-claude.vercel.app/zh/guides/ban-case-studies/)　|　[在 X (Twitter) 查看原文 ↗](https://x.com/MinLiBuilds/status/2105890192108536029)

- **Ray Wang** ([@wangray](https://x.com/wangray)) · 2026年10月1日

### 国庆封号反馈期间分享：半年稳定使用的配置

分享过去半年未被封的配置：独立 Mac 主机、美西时区、单账号从未切换、Surge 规则模式常开、夏威夷独立家宽 IP、美国信用卡、每周用量 50%–80%，外出时远程访问主机发起任务。作者强调不存在 100% 防封的方法，稳定 IP 加非高频使用大概率能降低风险。

实战要点：保持稳定、少变化：一个账号、一台设备、一个出口 IP，周额度留出余量。

标签：基础设施 · 支付 · 使用行为　|　相关指南：[操作系统/浏览器环境纯化与原生住宅 IP 配置指南](https://fuck-claude.vercel.app/zh/guides/environment-cleanup-and-ip-setup/)　|　[在 X (Twitter) 查看原文 ↗](https://x.com/wangray/status/2105658146685608265)

## Claude 防封避坑与环境纯化资料库

先汇总 Anthropic 公开的规则，再深入讲解风控原理、环境纯化、注册支付避坑、API 中转规范与英文申诉 SOP。

[阅读全文 →](https://fuck-claude.vercel.app/zh/guides/)

模型还是中国的好：[Kimi](https://www.kimi.com/code?utm_source=fuck-claude) · [DeepSeek](https://www.deepseek.com/?utm_source=fuck-claude) · [GLM](https://bigmodel.cn/?utm_source=fuck-claude)

## 常见问题

### Claude 真的会检查我的时区吗？

据公开逆向分析，Claude Code 连接非官方端点时会读取系统时区与中转 hostname，并把结果隐写进 system prompt。本页通过 Intl.DateTimeFormat 读到的，正是同一个系统时区。

### 这个分数就是 Claude 的真实判定吗？

不是。只有系统时区能与 Claude 被披露的机制一一对应，其余九项是与之相关的「中文环境 / 网络指纹」。分数是估计，不是定论。

### 怎么降低分数？

把系统时区改出 Asia/Shanghai 等中国时区，把 zh-CN 从浏览器语言列表首位移除，禁用 WebRTC 泄露，并避免让 Claude Code 走 hostname 含敏感域名 / AI 实验室关键词的中转。上方「如何降低你的风险分」清单按顺序完整讲了一遍。

### 会上传我的数据吗？

不会。所有检测都在浏览器本地完成，检测到的任何信号都不会被发送。网站加载 Google Analytics 统计匿名访问量，并通过 Google AdSense 展示广告(广告可能使用 Cookie)。WebRTC 泄露检测可能短暂连接公共 STUN 服务器以收集 ICE 候选。

### 分数低就一定不会被封吗？

不能保证。本工具只衡量浏览器可见的指纹。Anthropic 真实的风控模型还会权衡你的 IP 信誉、支付卡 BIN、账号资历与使用模式 —— 这些网页都看不到。低分能消除明显的本地破绽，但一个数据中心 IP 或高风险虚拟卡照样可能让账号被标记。

### 用这个工具或看这些指南会违反 Anthropic 条款吗？

检测器只是读取你自己的浏览器并如实报告 —— 这本身不违反任何条款。指南是关于指纹检测原理的教育性参考资料。你需要为自己如何使用 Claude、以及是否遵守 Anthropic 政策与当地法律负责。另请注意：Anthropic 帮助中心把「从不受支持的地区创建账户」列为封禁原因，任何浏览器设置都不会让不符合资格的账号变得合规。

### 为什么时区比我的 IP 地址还重要？

因为系统时区是在客户端读取的，并以逆向报告直接关联到「被编码的判定结果」的方式随请求一起传出。VPN 换的是 IP，通常不会改动系统时区 —— 这正是很多「干净」的 VPN 配置在这里依然高分的最常见原因。

### API 结果和浏览器检测不一样，为什么？

/api/check 的 curl 接口跑在服务端，只能看到请求暴露的信息:Vercel 归属地时区、国家、Accept-Language 与 User-Agent。它无法探测字体、Intl locale 或 WebRTC 泄露，因此只覆盖约 62/100 的权重再归一化到 0–100。浏览器端检测读取的是你真实的操作系统，是更完整的画像。

### 我的账号已经被封了，现在怎么办？

先判定封号类型(IP 拦截、被迫退款，还是账号被完全禁用)，因为每一种的处理路径都不一样。「封号类型判定与英文申诉 SOP」指南讲了如何区分并附有申诉信模板；如果恢复不现实，「国产 / 开源灾备」指南讲了如何以最小损失切换到平替模型。官方申诉入口是用被封账号登录 claude.ai 后进入的申诉表单。

## 隐私说明

所有检测都在你的浏览器本地完成，扫描结果不会离开你的设备。网站加载 Google Analytics 统计匿名页面访问量，并通过 Google AdSense 展示广告；检测到的信号不会被发送。WebRTC 泄露检测可能短暂连接公共 STUN 服务器。

[完整隐私政策 →](https://fuck-claude.vercel.app/zh/privacy/)
