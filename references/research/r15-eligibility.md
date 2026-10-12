# R15 — Eligibility vs. Signal: Can Third-Party Software Make a Mainland-China Claude Account Compliant?

Scope note: this file covers **eligibility and registration-time identity prerequisites only**. Claude payments, subscriptions, cards, billing addresses and virtual cards are out of scope and are not researched or discussed here.

## 1. The gap, restated

The site itself states the gap that no tooling closes:

- Mainland China is **not** in Anthropic's supported-countries list. The antiban-essentials page says: "中国大陆、香港、澳门不在[支持地区名单](https://www.anthropic.com/supported-countries)内，台湾、新加坡、日本、印度在名单内" (mainland China, Hong Kong and Macao are not in the supported-countries list; Taiwan, Singapore, Japan and India are). Source: https://fuck-claude.vercel.app/zh/guides/antiban-essentials/
- "Creating an account from an unsupported region" is an **official ban reason**: the same page lists Anthropic's three published ban reasons as "反复违反使用政策、从不受支持的地区创建账户、违反服务条款" (repeated Usage Policy violations, creating an account from an unsupported region, violating the Terms of Service), citing the help-center article. Source: https://fuck-claude.vercel.app/zh/guides/antiban-essentials/
- **No browser score makes an ineligible account compliant.** The antiban page's self-check states: "从不受支持的地区创建的账号，无论浏览器得分多少都可能被停用" (an account created from an unsupported region may be suspended regardless of its browser score). Source: https://fuck-claude.vercel.app/zh/guides/antiban-essentials/
- The regional-access strategy page states the same limit directly: "支持地区的出口 IP 不会让账号变得合规，2026 年 10 月 1 日起已有香港用户反馈被停用" (a supported-region exit IP will not make the account compliant; since October 1, 2026 Hong Kong users have reported suspensions). Source: https://fuck-claude.vercel.app/zh/guides/regional-access-strategy/

## 2. Primary question: does a VPN / third-party tool change eligibility? — answered from Anthropic primary sources

### 2.1 https://www.anthropic.com/supported-countries (fetched 2026-10-12)

Exact supporting language:

> "Anthropic's products and services — including our apps (such as Claude.ai, Claude Code, and Cowork), our API and developer platform, and Claude models offered through cloud platforms and authorized resellers — are available only in the countries and regions listed below."

> "Any country or region not listed is unsupported. This includes:
> - Use by persons while physically located in an unsupported region, including users and personnel of otherwise supported entities.
> - Use by entities incorporated or headquartered in an unsupported region and their users or personnel, regardless of whether such individuals are physically located in a supported region.
> - Use by entities that are majority-owned or controlled, directly or indirectly, by persons or entities in unsupported regions…"

Mainland China (and Hong Kong) do not appear in the list; Taiwan, Singapore and Japan do.

**Limiting language — what this means for VPNs:** the page defines support by **physical location** ("Use by persons while physically located in an unsupported region"), not by the network route. The page contains **no provision** under which a VPN, proxy, or any third-party tool changes that test — there is no language on the page granting eligibility based on IP address or exit country. (Honest caveat: neither primary page contains the literal word "VPN"; the conclusion follows from the physical-location definition of the rule, not from an explicit anti-VPN sentence.)

### 2.2 https://support.claude.com/en/articles/8241253-safeguards-warnings-and-appeals (fetched 2026-10-12)

Exact ban-reason language:

> "As part of our safety process, we may ban an account for a variety of reasons:
> - Repeated violations of our Usage Policy
> - **Account creation from an unsupported location**
> - Terms of Service violations"

The ban reason attaches to **where the account was created**, not to the IP address used on any given day — so a tool that changes the observed IP at login does not alter the creation-location fact the rule keys on. (Note: the site renders this as "从不受支持的地区创建账户 / creating an account from an unsupported region"; the article's exact English wording is "Account creation from an unsupported location.")

### 2.3 October 2026 Hong Kong suspension reporting (VPN exits not curing eligibility)

The site cites https://aistify.com/claude-hong-kong-vpn-account-bans/ (fetched 2026-10-12), which reports:

> "Hong Kong users report Claude account suspensions after accessing the service through VPNs."

> "A connection routed through a supported country does not, by itself, establish that the underlying use complies with Anthropic's access rules."

The article describes one public Reddit account of a Hong Kong user who "had accessed Claude through NordVPN for six months before receiving a suspension email," with the notice attributing the decision to "suspicious account signals and a Usage Policy violation, without identifying a specific detection method." It relays SCMP reporting (https://www.scmp.com/tech/tech-war/article/3369526/hong-kong-users-caught-offguard-anthropic-tightens-vpn-access-claude) of users routing through supported locations such as Singapore, while honestly noting: "The available evidence does not establish that Anthropic has introduced a worldwide ban on VPN use or disclose how it identified the affected accounts." In short: months of working through a supported-country VPN exit did not keep those accounts available — the VPN exit did not cure the underlying eligibility question.

## 3. Honest distinction: SIGNAL-level risk vs. ELIGIBILITY-level risk

| Level | What it is | Cleanable by third-party tools/settings? | Sources |
|---|---|---|---|
| **SIGNAL-level** (reduces false positives / friction) | System timezone vs. IP mismatch; browser language list; WebRTC/IP leaks; datacenter-ASN vs. residential IP reputation; device-fingerprint consistency; SMS re-verification frequency (reduced by binding TOTP 2FA); VoIP phone-number ranges being auto-rejected | **Yes — these are exactly what settings and tools can clean up.** The site frames these as reducing "信号矛盾带来的 false positive," explicitly "而非追求完全隐匿身份" (not complete identity concealment). | https://fuck-claude.vercel.app/zh/guides/antiban-essentials/ , https://fuck-claude.vercel.app/zh/guides/regional-access-strategy/ |
| **ELIGIBILITY-level** (binary, not tool-fixable) | (a) Being physically located in an unsupported region while using the service; (b) account creation from an unsupported location; (c) entity incorporation/headquarters/majority ownership in an unsupported region | **No.** Anthropic's own rules define these by physical location and creation location; no browser score, IP, VPN, or helper software changes where the person physically is or where the account was created. | https://www.anthropic.com/supported-countries , https://support.claude.com/en/articles/8241253-safeguards-warnings-and-appeals |

The site says this itself: its practices "不能让不符合资格的账号变得符合资格" (cannot make an ineligible account compliant) — https://fuck-claude.vercel.app/zh/guides/antiban-essentials/

## 4. Registration-time identity prerequisites the site names (outside payment scope)

Payment/card/billing prerequisites are deliberately excluded here; the three identity items below are what the site names for **registration itself**:

1. **Phone verification.** "Anthropic 在首次注册、异地登录或设备/IP 变动时都会触发手机验证…Google Voice、TextNow、Skype 以及多数免费接码平台的 VoIP 号段已被大规模拦截…风控模型会直接拒绝" (Anthropic triggers phone verification at first registration, unfamiliar logins, and device/IP changes; Google Voice, TextNow, Skype and most free receiving-platform VoIP ranges are blocked at scale and the risk model rejects them directly). The page's comparison table rates "个人实体 SIM（美/英/新/日）" (personal carrier SIM) highest for pass rate and "VoIP / Google Voice" as "接近零 / 请勿使用" (near zero / do not use) — i.e., **carrier SIM preferred over VoIP pools**. Source: https://fuck-claude.vercel.app/zh/guides/account-registration-and-payment-antiban/ (same guidance summarized at https://fuck-claude.vercel.app/zh/guides/antiban-essentials/: "避开 VoIP 号段：优先实体 SIM 或口碑接码平台").
2. **Email.** The site's news digest treats the registration email as a weighted identity factor: "针对近期 Claude Code 封号潮的稳定经验：采用几个月前注册的干净 Gmail、绑定真实美区手机号" and "注册环境与邮箱权重至关重要，避免临时虚拟号接码" (stable practice: an established clean Gmail registered months earlier, bound to a real US-region phone number; registration environment and email weight matter — avoid temp receiving-number services). Source: https://fuck-claude.vercel.app/zh/news/
3. **Two-factor authentication (2FA).** "注册后立即绑定 TOTP 2FA：使用 Google Authenticator、1Password 或 Authy，在首次更换 IP 或设备前完成绑定，可显著减少反复短信验证" (bind TOTP 2FA immediately after registration, before the first IP/device change, to markedly reduce repeated SMS verification); also "启用 2FA 并下载备份码后再结束会话" (enable 2FA and download backup codes before ending the session). Source: https://fuck-claude.vercel.app/zh/guides/account-registration-and-payment-antiban/ (also https://fuck-claude.vercel.app/zh/guides/regional-access-strategy/, travel checklist item 1: "在 Claude 账号启用 2FA（Google Authenticator）").

**If a supported-region phone number is needed** (identity research, not payment): legitimate options exist — a prepaid carrier SIM bought in person in the supported country (US prepaid requires no identity documents at purchase) or a carrier/MVNO eSIM activated remotely before arrival with Wi-Fi calling: https://sim-finder.com/en/guides/local-contract/ and https://settlekit.com/guides/phone-number . Obtaining such a number satisfies a registration prerequisite; it does not change physical-location eligibility (Section 2).

## 5. Verdict

**Verdict: NO** — third-party software does not solve R15. Anthropic's primary sources define eligibility by the user's *physical location* ("Use by persons while physically located in an unsupported region") and by *account creation location* ("Account creation from an unsupported location"); no VPN, proxy, fingerprint tool, or browser score alters either fact, the site itself says a supported-region exit IP "不会让账号变得合规," and the October 2026 Hong Kong suspensions show supported-country VPN exits did not keep accounts eligible — tools can only clean SIGNAL-level risk, never ELIGIBILITY-level risk.

---

### Source URLs cited (10)

1. https://fuck-claude.vercel.app/zh/guides/antiban-essentials/
2. https://fuck-claude.vercel.app/zh/guides/regional-access-strategy/
3. https://www.anthropic.com/supported-countries
4. https://support.claude.com/en/articles/8241253-safeguards-warnings-and-appeals
5. https://aistify.com/claude-hong-kong-vpn-account-bans/
6. https://www.scmp.com/tech/tech-war/article/3369526/hong-kong-users-caught-offguard-anthropic-tightens-vpn-access-claude
7. https://fuck-claude.vercel.app/zh/guides/account-registration-and-payment-antiban/
8. https://fuck-claude.vercel.app/zh/news/
9. https://sim-finder.com/en/guides/local-contract/
10. https://settlekit.com/guides/phone-number
