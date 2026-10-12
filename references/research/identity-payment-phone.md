# R19/R20 身份维度调研：支付、银行卡与手机号方案（站点方案 + web 补齐）

调研日期：2026-10-12。范围：支付/银行卡/手机号等注册与身份维度的**可执行方案**。Claude 订阅的支付流程本身按站点规则属于审计范围（R19/R20）；本文件分层标注证据等级：**官方一手**（Anthropic/Apple/运营商官方页）> **社区实测**（第一手使用记录）> **厂商/推广文**（取机制分析，折扣其推销）。

## 0. 硬结论（先读，适用于本文件全部方案）

以上任何方案都只解决**注册前置与信号一致性**（卡种对、账单地址对、真号且地区对、支付环境与日常一致），**都不改变账号资格**：Anthropic 按人身所在地判定（一手原文 "Any country or region not listed is unsupported… Use by persons while physically located in an unsupported region…"，2026-10-12 逐字核实，https://www.anthropic.com/supported-countries ）；2026-10-01 起香港用户经受支持地区 VPN 出口使用数月仍被停用（https://aistify.com/claude-hong-kong-vpn-account-bans/ ）。

## 1. 站点方案（fuck-claude.vercel.app）

### 1.1 支付/银行卡（官方规则 vs 社区观察）

来源：https://fuck-claude.vercel.app/zh/guides/payment-methods-comparison/ 与 https://fuck-claude.vercel.app/zh/guides/account-registration-and-payment-antiban/ （官方信息核对于 2026-10-08）。

**官方规则侧：**
- 网页端 Pro/Max 只收信用卡/借记卡（不收 PayPal/Venmo）；Team 另收预付卡；App Store/Google Play 内购由商店代扣（官方帮助中心确认：内购无 Anthropic 发票，Apple/Google 直接扣款 — https://support.claude.com/en/articles/16607638-understanding-your-pro-or-max-plan-invoices ）。
- 账单地址必须与银行记录一致、与发卡国一致、属受支持账单地区；少一个重音符号都可能拒付。
- 官方扣款排查 7 步：账单地区 → 地址逐字符 → 3D Secure → 卡类型（信用卡/借记卡）→ 余额 → 稍后重试 → 问发卡行。
- 退款先走 Get help > Claude Refund Request；银行拒付（chargeback）可能让账号被关联支付欺诈标记，慎用。

**社区观察侧（站点明示"是观察不是规则"）：**
- 卡风险排序：美国银行发行 Visa/MC 信用卡（低）> 金融科技借记卡（低-中）> 支持国国际借记卡（中）> 大众预付卡/加密货币充值预付卡（高）。
- **第三方出售的卡不建议**：多人共用、余额与冻结不可控、无追索。
- 网传"高成功率 BIN 名单"无来源（Anthropic 从未公布拦截哪些 BIN），站点已删除。
- 同一张卡勿绑定超过 2 个 Claude 账号（最快的连带封号路径）；勿频繁换卡；勿在机房 IP 下绑与账单国不一致的卡。
- 顺序：先配环境 → 同一地区同一会话内完成注册/验证/绑卡/2FA，中途不切地区。

### 1.2 手机号（官方触发逻辑 + 号码质量排序）

来源同上 + https://fuck-claude.vercel.app/zh/guides/regional-access-strategy/ 。

- Anthropic 在首次注册、异地登录、设备/IP 变动时触发手机验证。
- 通过率排序（社区观察）：**个人实体 SIM（美/英/新/日运营商：AT&T/T-Mobile/EE/Singtel/SoftBank）最高** > 付费独占接码（中高）> 公共共享接码池（低、极高关联风险，勿用）> **VoIP（Google Voice/TextNow/Skype）≈ 0，明令勿用**。
- 注册后立即绑 TOTP 2FA（Google Authenticator/1Password/Authy），可显著减少反复短信验证；号码须在整个订阅周期内保持有效。
- 号码国家须与账号地区一致（美号配美区、英号配英区、新号配新区）。

## 2. 手机号具体方案（web 补齐，2026-10 现状）

| 方案 | 类型/网络 | 成本 | 关键机制 | 风险/注意 | 证据 |
|---|---|---|---|---|---|
| **Ultra Mobile PayGo**（美号首选） | T-Mobile 真无线 MVNO（非 VoIP） | $3/30 天（100 分钟/100 短信/100MB） | 开 Wi-Fi Calling 后全球经 Wi-Fi 收发短信（含大陆，社区普遍可用但建议自测）；后台支持信用卡/PayPal 续费 | **60 天欠费即销号**（务必开 Auto Renew）；实体卡需官方零售包（eBay 官方渠道），人在境外激活偶发地理拦截需美国 IP 配合；收码走 Wi-Fi Calling，不开则漫游收短信 $0.1/条 | 官方 https://www.ultramobile.com/paygo/ ；社区实测 https://site2us.com/en/phone/ultra-mobile-paygo 、中文教程 https://nomadinit.com/p/19.html 、https://www.howardforums.com/threads/ultramobile-paygo-3-mo.1931444/ |
| **Tello / US Mobile**（美号备选） | T-Mobile / Verizon+AT&T+T-Mobile 真无线 MVNO | $5/月起 | eSIM + Wi-Fi Calling 收 2FA/银行验证码 | Tello 对境外激活在收紧（部分用户需 VPN）；US Mobile 号称稳定收短码 | 对比 https://nomadadvisers.com/guides/us-phone-number-for-banking-abroad |
| **Giffgaff**（英号，配英区账号） | O2 真实号段 | 0 月租，£10 起充；保号≈£0.3/180 天 | 国际漫游下**接收短信免费**，大陆锁定 China Mobile 后稳定收码（社区近一年实测）；免实名 | ⚠️ **2026-07-28 起官方按公平使用政策批量断开长期在英国境外使用的号码**（任何 4 个月在英外用满 63 天即可停号、余额不退；被停后 30 天内可要 PAC 码携号转网止损）——只适合人常驻英国或明确接受风险者；号码复用风险（前任号主的 Telegram 等可能仍绑此号） | 实测 https://kannetflix.com/esim/giffgaff/ ；断号分析 https://www.digitalnomadlc.com/how-to-buy-giffgaff-esim/ 、https://www.xiaoming.io/giffgaff/ |
| **Lebara**（英号替代） | 英国 MVNO | £2.5-5/月起 | 条款未明确禁止长期漫游；90 天需一次余额变动（漫游短信 £0.49） | 条款可随时收紧；"谁也不敢绝对保证"（社区原文） | 社区方案 https://github.com/zzmy1917-svg/giffgaff-Lebara-eSIM |
| （不推荐）公共接码平台/Google Voice | VoIP/共享 | — | — | 站点与社区一致结论：对 Claude ≈ 0 通过率 + 极高关联风险 | 站点见1.2 |

设备侧备注：国行手机多不支持 eSIM — 社区用"小白卡/eUICC 转接卡"（Xesim、EasyEUICC+小白卡）把 eSIM 写成实体卡；Ultra Mobile 亦可实体卡转 eSIM（官方 App）。

## 3. 支付/银行卡具体方案（web 补齐，风险从低到高）

### 3.1 首选：iOS App Store 内购 + 美区 Apple ID + 正规渠道礼品卡（社区实测共识最稳）

- 机制：Apple 代扣款，不经 Stripe 网页风控；Anthropic 官方确认内购由 Apple 出具收据（官方一手，见1.1链接）；App Store 官方页列出 Claude Pro 内购 $20/月、Max 5x $124.99、Max 20x $249.99（官方一手：https://apps.apple.com/ai/app/claude-by-anthropic/id6473753684 ）。
- 礼品卡必须正规渠道（苹果美国官网/Amazon 美亚/支付宝"惠出境"Pockyt），**低于市价 20% 的卡大概率黑卡**；美区 Apple ID 需"预养号"（社区实测新号直购拦截率高，养 3-7 天明显改善 — 社区实测 https://svip.chat/blog/post/ClaudePro订阅失败 ，该站偏推广，数字自行打折）。
- 代价：内购溢价 — 有用户实测 Max 5x 官网 $100/月经礼品卡≈$125（社区实战记录 https://www.jxxy.net/ai/articles/cxjwin-2051628853878296582/ ）。
- 港区 Apple ID 可下 Claude App 但订阅入口灰色（Anthropic 未在港区开放订阅）— 社区实测。

### 3.2 网页端：本人实名 + 发卡地与账单/IP 一致的卡

社区盘点（2026-07，U卡雷达 https://ucardradar.com/insights/claude-pro-virtual-card-payment — 卡机构评测站，证据分级清晰）：
- **官网活动直接点名 Claude Pro**（第一档）：Bybit Card、Wirex One Card。
- 具备循环扣款/3DS/账单地址能力、可试（第二档）：MetaMask Card、KAST Card、RedotPay Card（⚠️ RedotPay 已不对大陆开放申请）。
- 预付属性可能被拦（第三档）：Crypto.com Prepaid；一次性卡（One-Time）**不适合**任何自动续费订阅。
- Wise 美国卡头虚拟卡：有实测高通过率报告（$25 押金开户；社区实测，见 svip.chat，推广折扣）。
- 选卡四问：能长期用卡号？支持循环扣款（Recurring/Card-on-File）？支持 3DS？账单地址用发卡平台登记的真实地址（勿抄"通用模板"）？
- 网页端绑卡前先验 IP 欺诈分（whoer.net/ipdata.co），余额留出税费与 $1 预授权空间。

### 3.3 绝对避开

- **低质虚拟卡 BIN**：已被 Stripe 批量拉黑（WildCard 2025-07 停服；Depay/Nobepay 不稳）— 社区实战记录（jxxy.net，见3.1）。
- **第三方代充**：盗刷黑卡连带永久封号 + 钱号两失 + 削弱申诉可信度（风控机制分析 https://ipbook.net/prevent/after-recharge-ban/ — 指南站，机制描述与站点一致）。
- **卡被拒后连续重试**：3 次以上可能被标记恶意盗刷（同上 + https://www.photonpay.com/hk/blog/article/claude-pro-subscription-failed?lang=zh-cn — 厂商文，取机制不取推销）。

### 3.4 充值/绑卡纪律（汇总）

1. 先养环境再充值：新号/刚换 IP 的环境里勿立刻大额支付；用免费额度正常用几天再订。
2. 首充从低档起（Max 类高价套餐是风控重点区）。
3. 充值时刻的 IP/设备与日常一致，勿临时切陌生环境。
4. 卡被拒 → 停 24h+、清缓存、换干净环境与从没被拒过的卡再试。
5. 一卡一用途（专卡订阅）、卡内只留 1-2 个月余额（防平台跑路）。
6. 续费日前 3 天查余额；连续扣款失败会掉订阅。
7. 充值后几天保持低调（勿立刻挂脚本/API 高频）。

## 4. 与审计细则的挂钩

- 本文件整体服务 **R19（支付资料与账号地区一致）** 与 **R20（注册手机号可信）**；判定标准见 SKILL.md 差距细则表。
- 任何方案通过后仍需满足 R1-R18 的环境一致性（卡发行国 ≈ 账单国 ≈ 号码国 ≈ 登录 IP 国 ≈ 时区/语言声称地区）。
- R15 资格结论不变：以上均不改变"人在不支持地区"的官方判定（见第 0 节）。

## 5. 来源清单（按证据等级）

**官方一手**：anthropic.com/supported-countries；support.claude.com（8241253 封禁与申诉、16607638 发票说明、付费计划账单 FAQ）；apps.apple.com（Claude 内购价目）；ultramobile.com/paygo。
**站点转述（官方核对于 2026-10-08）**：fuck-claude.vercel.app 的 payment-methods-comparison、account-registration-and-payment-antiban、regional-access-strategy、antiban-essentials。
**社区实测**：site2us.com（Ultra Mobile）、nomadinit.com（Ultra Mobile 中文）、howardforums（Ultra Mobile 激活政策）、kannetflix.com 与 digitalnomadlc.com 与 xiaoming.io（Giffgaff，含 2026-07 断号潮）、aizoos.com（小白卡/eSIM 转接卡）、jxxy.net（App Store 礼品卡实战）、nomadadvisers.com（MVNO 对比）。
**评测/指南站（中等）**：ucardradar.com（虚拟卡盘点）、ipbook.net（充值风控分析）、site2us/esimtao 等。
**厂商/推广文（仅取机制分析）**：photonpay.com、svip.chat、xmpick.com、globalesimcards/mollysim（卡商）。
