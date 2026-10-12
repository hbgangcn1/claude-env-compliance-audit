---
name: claude-env-compliance-audit
description: 判断本机是否符合 Claude 账号审核/防封标准（fuck-claude.vercel.app 的 10 信号加权模型 + 官方支持地区规则）。当用户问"本机是否符合 Claude 规定""Claude 防封/环境纯化/账号审核""按 fuck-claude 检测标准查一下""Claude account review environment check"时使用。内置只读审计命令包、R1-R20 差距细则与修复命令、住宅 IP/字体指纹/WebRTC 第三方方案（带来源 URL）、账号资格结论，references/ 内附站点 25 页中文全量摘要与历次审计原始产物。适用于 OmO / DSH / OpenClaw / OpenCode / Codex（共享 ~/.agents/skills）与 Hermes（需放 ~/.hermes/skills）。
---

# Claude 账号环境合规审计（本机 vs fuck-claude.vercel.app 标准）

## 宿主代理与安装位置（AgentSkills 通用格式，name 与目录同名）

| 代理 | 加载位置 | 证据 |
|---|---|---|
| OmO / DSH / OpenClaw / OpenCode / Codex | `~/.agents/skills/claude-env-compliance-audit/SKILL.md`（五家共享同一根） | 各家官方文档/本机实证；Codex 官方文档 USER 级行明列 `$HOME/.agents/skills` |
| Hermes（NousResearch） | `~/.hermes/skills/claude-env-compliance-audit/SKILL.md`（只认自己的根，需复制一份过去） | hermes-agent 官方文档：All skills live in ~/.hermes/skills |

## 何时用

- 用户问：本机（Windows 或 macOS）是否符合 Claude 账号审核/防封标准、按 fuck-claude.vercel.app 检测一下、环境纯化还差什么、为什么被判定为中国用户。
- 交付物永远是**一份差距报告**：每条要求 → PASS/FAIL/PARTIAL/N/A + 证据（命令原文 + 输出）+ 修复动作（本机命令 / 第三方方案 / 无解）。

## 何时不用

- 用户账号已被封、要申诉 → 直接读 `references/site/guides_account-appeal-and-recovery-sop.md`。
- 用户主动限定范围（如"只看本机设置，支付先不管"）→ 按用户指定裁剪，不要自行扩大或缩小。

## 站点与模型背景（先读懂再审计）

来源：https://fuck-claude.vercel.app/zh/ （首页）与 /zh/guides/ （指南目录）。站点是**社区逆向整理，非 Anthropic 官方文档**——报告里必须保留这个限定。全站 25 页中文摘要在 `references/site/`（覆盖校验 coverage-report.md = 25/25 PASS，抓取于 2026-10-12）。

**10 项加权信号（0-100 分；低 0-30 / 中 31-60 / 高 61-100）：**

| 权重 | 信号 | 站点要求 |
|---|---|---|
| 24 | 系统时区 | 非 Asia/Shanghai、Asia/Urumqi（其声称 Claude Code 经 Intl 读 OS 时区） |
| 18 | 浏览器语言 | zh-CN 不得在 navigator.languages / Accept-Language 首位 |
| 14 | 已装中文字体 | canvas 宽度探测不到雅黑/宋体/苹方等（用干净 Profile 或防指纹浏览器隔离） |
| 10 | 国产厂商字体 | 无 MiSans/鸿蒙黑体/OPPO Sans/方正等 |
| 10 | WebRTC IP 泄露 | 堵住 STUN 泄露真实 IP |
| 8 | 国产浏览器/WebView | UA/UA-CH 无微信/QQ/夸克/UC/百度等 |
| 6 | 国产品牌设备 | UA-CH 无鸿蒙/华为/小米/OPPO/vivo |
| 4 | Intl 区域设置 | 目标地区 locale（如 en-US） |
| 3 | 时区偏移 | getTimezoneOffset() != UTC+8 |
| 3 | Emoji 渲染 | 弱信号 |

**官方规则（站点引用，核对于 2026-10-08）：** 中国大陆/香港/澳门**不在** Anthropic 支持地区（台/新/日/印在）；三类封禁原因（反复违反使用政策、从不受支持地区创建账户、违反 ToS）；**支持地区的 VPN 出口不改变账号资格**。支付/身份侧官方规则：网页端 Pro/Max 只收信用卡/借记卡（Team 另收预付卡），账单地址必须与发卡国家一致且属受支持账单地区（站点引自帮助中心付费计划账单 FAQ）。一手核验：https://www.anthropic.com/supported-countries （原文 "Any country or region not listed is unsupported… Use by persons while physically located in an unsupported region…"，2026-10-12 已逐字核实）。

## 审计命令包（先判宿主 OS：Windows 用 A-F；macOS 用下方"macOS 命令包"专节。全部只读；逐条记录命令原文+完整输出为证据）

### A. 系统区域与时区（对应 R1/R2/R4）

```powershell
(Get-TimeZone).Id; (Get-Date).ToString('yyyy-MM-dd HH:mm:ss zzz')
Get-Culture | Select-Object Name,DisplayName | Format-List
Get-WinSystemLocale | Select-Object Name | Format-List
Get-WinUserLanguageList | ForEach-Object { $_.LanguageTag + ' | ' + $_.Autonym }
[System.Globalization.RegionInfo]::CurrentRegion | Select-Object Name,EnglishName | Format-List
reg query "HKCU\Control Panel\International" /v sLanguage
Get-TimeZone -ListAvailable | Select-Object -ExpandProperty Id   # 确认非中国时区已安装
```

期望（达标态）：时区为 Pacific/Eastern/GMT/Singapore/Tokyo 等非中国时区；显示语言列表 en-US 打头；RegionInfo 非 CN。修复：`Set-TimeZone -Id "Pacific Standard Time"`（管理员）；`Set-WinUserLanguageList -LanguageList en-US -Force`；`Set-Culture -CultureName en-US`；`Set-WinSystemLocale -Name en-US`（后两项需注销重登）。只想保护 Claude 场景时可不改系统级，只改浏览器/Profile 语言。

### B. 字体（R5/R6）

```powershell
Get-ChildItem C:\Windows\Fonts, "$env:LOCALAPPDATA\Microsoft\Windows\Fonts" -ErrorAction SilentlyContinue |
  Where-Object {$_.Name -match '(?i)msyh|simhei|simsun|pingfang|MiSans|HarmonyOS|OPPO|SourceHan|NotoSansCJK|Deng|KaiTi|FangSong|Alibaba'} |
  Select-Object Name | Format-Table -AutoSize
```

判定：命中 msyh/simhei/simsun/Deng/STKAITI = 中文字体信号 FAIL（OS 级，浏览器设置无解）；命中 MiSans/HarmonyOS/OPPO 等厂商字体 = 额外 FAIL。**不要建议删除系统字体**（毁中文显示且部分受保护删不掉），站点也只建议 Profile 隔离 → 见 R5 第三方方案。

### C. 网络与出口（R8/R9/R10/R14/R18）

```powershell
curl -s --max-time 20 https://ipinfo.io/json
curl -s --max-time 20 "http://ip-api.com/json/?fields=query,country,regionName,city,isp,org,as"
netsh winhttp show proxy
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"   # ProxyEnable/ProxyServer
Get-ChildItem env: | Where-Object {$_.Name -match '(?i)proxy'} | Format-Table -AutoSize
Get-NetAdapter | Select-Object Name,InterfaceDescription,Status | Format-Table -AutoSize
nslookup ipinfo.io    # DNS 解析器
```

判定：`org` 为消费级 ISP（Comcast/AT&T/Verizon 等）= 住宅 PASS；云厂商/托管 ASN（AWS/Hetzner/24 SHELLS 等）= FAIL。DNS 必须非国内（1.1.1.1/8.8.8.8）且与出口同路径。**若出现 Wintun/TUN 适配器**，揪出归属进程：

```powershell
Get-CimInstance Win32_Process | Where-Object { $_.CommandLine -match '(?i)wintun|clash|mihomo|v2ray|sing-box|openvpn|astrill' } |
  Select-Object ProcessId,Name,@{N='Cmd';E={$_.CommandLine.Substring(0,[Math]::Min(300,$_.CommandLine.Length))}} | Format-List
```

（2026-10-12 实测本机为 Astrill VPN：`asovpnc.exe --remote … --adapter wintun`，服务 ASOVPNHelper；fake-IP 网段 198.18.x = Clash 系 TUN 特征。）

### D. Claude Code 环境（R11/R12/R13）

```powershell
Get-Command claude -ErrorAction SilentlyContinue | Select-Object Source; claude --version
Get-ChildItem env: | Where-Object {$_.Name -match '(?i)anthropic|claude|^TZ$'} | Format-Table -AutoSize
Test-Path "$HOME\.claude\settings.json"
reg query "HKCU\Environment"    # 查 TZ / ANTHROPIC_* 残留
```

判定：无 claude CLI / 无 ANTHROPIC_BASE_URL → R11-R13 = N/A（不适用，非 PASS）。若用了中转：hostname 须中性（无 deepseek/moonshot/zhipu/minimax/dashscope 等实验室词、非 .cn）、shell 里 `TZ` 与出口一致、非官方端点加 `_CLAUDE_CODE_ASSUME_FIRST_PARTY_BASE_URL=1`（站点称隐写机制读 OS 时区+中转 hostname）。

### E. 服务端快测（可选）

```powershell
curl -s --max-time 20 -H "Accept-Language: en-US,en;q=0.9" "https://fuck-claude.vercel.app/api/check?format=json"
```

注意：这是**服务端**按 IP+请求头估算，只覆盖 62/100 权重；Accept-Language 由你自报，**测出的 language 分数不代表浏览器真实值**——浏览器语言结论必须以 A 步的系统显示语言推导。

### F. 浏览器端（无法命令行取证，交用户自测并如实标注）

- WebRTC：装 WebRTC Network Limiter / WebRTC Control 后去 https://browserleaks.com/webrtc 自测（状态未知就报 UNKNOWN，不许猜 PASS）。
- navigator.languages / Intl：DevTools 控制台看首位与 locale。
- 打开 https://fuck-claude.vercel.app/zh/ 跑站内检测器看 0-100 分。

## macOS 命令包（darwin 宿主用本节替代 A-D；判定标准与修复顺序同下文细则）

### A′. 系统时区 / 语言 / 区域（R1/R2/R3/R4）

```bash
readlink /etc/localtime                                   # 时区，如 /usr/share/zoneinfo/America/Los_Angeles
defaults read -g AppleLanguages                           # 显示语言顺序，首位决定 navigator.languages
defaults read -g AppleLocale                              # 区域，如 en_US / zh_CN
node -e "const o=Intl.DateTimeFormat().resolvedOptions();console.log(o.timeZone,o.locale)"
```

期望：localtime 指向非中国时区；AppleLanguages 首位 en-US；AppleLocale 非 zh_CN。修复（站点同款命令）：`sudo systemsetup -settimezone "America/Los_Angeles"`；语言/地区走 系统设置 → 通用 → 语言与地区（首选 English (US)，移除中文项）。

### B′. 字体（R5/R6）

```bash
ls /System/Library/Fonts /Library/Fonts ~/Library/Fonts 2>/dev/null | grep -iE "PingFang|STHeiti|STSong|Songti|Hiragino|Kaiti|Heiti|MiSans|HarmonyOS|OPPO|SourceHan|NotoSans|Alibaba"
```

判定：PingFang/STHeiti/Songti/Hiragino/Kaiti = 中文字体信号（macOS 系统自带且受保护，无法卸载 → 只能 Profile 隔离）；MiSans 等厂商字体为额外命中。

### C′. 网络与出口（R8/R9/R10/R14/R18）

```bash
curl -s https://ipinfo.io/json                            # 出口 IP/ASN（判定同下文 R8）
ifconfig | grep -A2 -E "^utun|^tun"                       # VPN 隧道接口
scutil --proxy                                            # 系统代理状态
scutil --dns | grep "nameserver" | head                   # DNS 解析器
ps aux | grep -iE "clash|surge|stash|v2ray|sing-box|xray|tailscale|wireguard|astrill" | grep -v grep   # 代理客户端
```

### D′. Claude Code 环境（R11/R12/R13）

```bash
command -v claude && claude --version
env | grep -iE "anthropic|claude|^TZ="
ls ~/.claude/settings.json 2>/dev/null
```

E′/F′ 与 Windows 版相同（服务端 api/check 用 curl；浏览器端 WebRTC/Lang/Intl 自测步骤不变）。修复命令汇总（macOS）：时区 `sudo systemsetup -settimezone "<目标时区>"`；语言/地区走系统设置；其余修复均为浏览器/Profile 层操作，与平台无关。

## 差距细则 R1-R20（判定标准 + 修复归类）

| # | 要求 | PASS 条件 | 修复归类 |
|---|---|---|---|
| R1 | OS 时区非中国 | 时区 ID 非中国时区（Win：`(Get-TimeZone).Id`；mac：`/etc/localtime` 指向非中国时区） | LOCAL：Win `Set-TimeZone` / mac `systemsetup -settimezone` |
| R2 | 偏移非 UTC+8 | getTimezoneOffset() != -480 | LOCAL：随 R1 |
| R3 | 语言 zh 不打头 | 显示语言列表/浏览器 en-US 首位 | LOCAL：Win `Set-WinUserLanguageList` / mac 系统设置语言与地区 + 浏览器语言设置 |
| R4 | 区域/Intl 对齐 | 区域非 CN（Win：RegionInfo/Culture；mac：AppleLocale） | LOCAL：Win `Set-Culture`/`Set-WinSystemLocale` / mac 系统设置地区 |
| R5 | 中文字体 canvas 不可探 | Claude 用 Profile 探测不到中文字体 | THIRD-PARTY：干净 Profile + 掩蔽扩展 / 防指纹浏览器 |
| R6 | 无厂商字体 | 字体扫描不命中 MiSans 等 | 已装即 FAIL，卸载/隔离 |
| R7 | WebRTC 不泄露 | browserleaks 自测干净 | LOCAL：扩展 + 自测 |
| R8 | 住宅/移动 ISP 出口 | ipinfo org = 消费级 ISP | THIRD-PARTY：换住宅出口 |
| R9 | 出口在支持地区 | country ∈ US/UK/SG/JP/TW… | 已达标即 PASS（与 R15 无关） |
| R10 | DNS 一致 | 非国内 DNS 且同路径 | LOCAL：保持远程 DNS + dnsleaktest |
| R11 | 中转 hostname 中性 | 无实验室词/.cn（用中转时） | N/A 或 LOCAL：换中性域名 |
| R12 | CLI TZ 对齐 | 每个 shell TZ=出口地区（用 CC 时） | N/A 或 LOCAL：Win `setx TZ` / mac 在 shell 配置里 `export TZ` |
| R13 | first-party 开关 | 用非官方端点时置 1 | N/A 或 LOCAL |
| R14 | 无国产浏览器/设备信号 | UA/UA-CH 干净 | LOCAL：用原版 Edge/Chrome |
| R15 | 账号资格（支持地区） | 人身/注册地在支持地区 | **STRUCTURAL：无软件可解**（见下） |
| R16 | 使用模式（养号/一机一号/非蒸馏流量） | 观察项 | ADVISORY：非机器可测 |
| R17 | IPv6 不泄露 | 适配器 v6 处理或代理覆盖 | LOCAL：禁 v6 绑定或验证隧道 |
| R18 | 浏览器与 CLI 同出口 | 无代理分裂 | LOCAL：统一 TZ/代理/端点 |
| R19 | 支付资料与账号地区一致 | 卡种被官方接受（Pro/Max=信用卡/借记卡）、账单地址与发卡国一致且属支持地区、无已知 BIN/预付卡风险、不频繁换卡 | IDENTITY：换卡种/对齐账单地址；官方扣款排查清单与卡种风险表见 `references/site/guides_payment-methods-comparison.md`；具体推荐与风险分级（App Store 礼品卡路径/卡盘点/避坑）见 `references/research/identity-payment-phone.md` |
| R20 | 注册手机号可信 | 实体运营商 SIM（非 VoIP/公共接码池）、号段与账号地区一致、已绑 TOTP 2FA | IDENTITY：实体 SIM/eSIM；通过率对比表见 `references/site/guides_account-registration-and-payment-antiban.md`；具体办卡方案（Ultra Mobile PayGo/Tello/Giffgaff/Lebara 含风险）见 `references/research/identity-payment-phone.md` |

**R15 权威结论（不得软化）：** Anthropic 按**人身所在地**与**账号创建地**判定资格，任何 VPN/指纹工具/评分都不能改变；2026-10-01 起香港用户经受支持地区 VPN 出口使用数月仍被停用（https://aistify.com/claude-hong-kong-vpn-account-bans/ ）。报告必须写明：**信号级风险可清理，资格级风险无解**。

## 第三方方案速查（本机设置无解时；均经 web 调研、带来源）

**R8 住宅出口**（目标：US/SG/JP/TW 消费级 ISP ASN、粘性会话、远程 DNS）：
- Oxylabs 住宅代理（四地专节点、按 ASN 钉选、24h 粘性）https://oxylabs.io/products/residential-proxy-pool — 综合最优
- Bright Data https://brightdata.com/proxy-types/residential-proxies — 住宅网络需企业 KYC，个人基本不可用
- CometVPN Native Residential https://cometvpn.com/features/residential-vpn/native/ — VPN 形态最省事，宣称住宅但须连上后 ipinfo 验 ASN
- 中文服务商：RESIP.NET https://resip.net/use-cases/ai-tools 、MaskProxy、IPRoyal（有大陆接入教程 https://help.iproyal.com/en/articles/10667956-how-to-use-iproyal-proxies-with-shadowrocket ）
- **Astrill Private IP 不够**：是 Astrill 自家机房服务器 IP，不改 ASN 类别（https://www.astrill.com/network/dedicated-ips ）

**R5 字体指纹**（个人单账号按站点建议优先免费路线）：
- 首选：独立 Profile + 掩蔽扩展 —— Firefox 用 CanvasBlocker（保护 TextMetrics，正中宽度探测，https://canvasblocker.kkapsner.de/ ）或 Chameleon 00x127；Chrome/Edge 用 Fingerprint Protector（https://chromewebstore.google.com/detail/fingerprint-protector/lhnhapbalajnchknjmjlcpoiehjegbdm ）。⚠️ 勿用 ghostwords 的 Chameleon（已归档自认不防 canvas）
- 防指纹浏览器：AdsPower（免费 2 Profile，字体列表掩蔽+canvas 噪声默认开）、GoLogin（免费 3，唯一官方声称掩蔽覆盖渲染测量）、比特浏览器/MoreLogin（国产人民币渠道）
- 完整对比与文档 URL：`references/research/r5-font-fingerprint.md`

**R7 WebRTC**：WebRTC Network Limiter（https://chromewebstore.google.com/detail/webrtc-network-limiter/npeicpdbkakmehahjeeohfdhnlpdklia ）或 WebRTC Control，设为"走代理"，browserleaks 自测。

## 执行纪律（血泪教训）

1. **全部只读**：不改时区/语言/系统区域/代理等系统设置——给命令让用户自己跑，或明确征得同意后再改。
2. **逐条证据**：每条判定必须附命令原文 + 完整输出；未知（如浏览器端信号）如实报 UNKNOWN。
3. **CJK 字数坑**：检查摘要文件长度用**非空白字符数**，空白分词对中文失效。
4. **服务端分数 ≠ 浏览器分数**：api/check 只覆盖 62% 权重，且 Accept-Language 自报。
5. **别把测试头当真**：用 curl 测语言信号时若自加了 Accept-Language，结论必须按系统显示语言推导并注明。
6. **支付/手机号一并审计**（R19/R20，属注册/身份维度）；仅当用户明确说不看时才裁剪并在报告中注明。
7. 报告结构参照 `references/example-report-20261012.md`（历次交付样例）。

## 语料说明（分享版）

本文件已自足：审计命令包（Windows + macOS 双平台）、R1-R20 细则、修复命令、第三方方案与来源 URL 全部在上文。可选的离线语料包（站点 25 页中文全量摘要 + 第三方方案全文调研含支付/手机号具体方案，均为通用内容、不含任何个人信息）可从以下地址现场获取：

- 站点全量摘要：直接抓取 https://fuck-claude.vercel.app/zh/guides/ 及其全部子页（robots.txt → sitemap-index.xml 可枚举）。
- 第三方方案全文：见上文各方案的官方/来源 URL。

参考基线（2026-10-12 某 Windows 11 机器实测样例，已脱敏；Mac 用户勿套用，重跑对应命令包）：时区 China Standard Time FAIL；显示语言仅 zh-Hans-CN FAIL；区域 CN FAIL；中文字体在列 FAIL、无厂商字体 PASS；出口为美国洛杉矶托管商机房 ASN（经商业 VPN 的 TUN 隧道）FAIL；DNS 为 Cloudflare（1.1.1.x）PASS；系统代理关闭、无代理分裂 PASS；无 claude CLI → R11-13 N/A。复审时重跑命令包更新，不要直接引用旧结论。