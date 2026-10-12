# 反检测（指纹隔离）浏览器调研 —— Windows 11 下隔离中文字体 / 时区 / 语言 / WebRTC

调研日期：2026-10-12。目标场景：为 Claude 账号做环境隔离（参考 https://fuck-claude.vercel.app/zh/guides/environment-cleanup-and-ip-setup/ 的要求：每配置独立 cookies/存储、每配置独享粘性住宅 IP、Canvas 噪声 / WebGL 掩蔽 / AudioContext 隔离、时区跟随代理、UA 与声称的操作系统一致）。

关键前提：下述所有产品均在**浏览器配置（profile）层面**伪造指纹与网络参数（时区、语言、字体列表、WebRTC、Canvas/WebGL 等按 profile 独立设置），**不修改 Windows 11 系统本身的区域/时区设置**。"中文字体隔离"在本机装有中文字体的情况下只能靠控制 profile 上报的字体列表（伪装/自定义字体枚举）实现，无法按 profile 物理增删系统字体——以下产品均提供该能力或由内核算法生成一致的字体指纹。

---

## 1. AdsPower（指纹浏览器）

- **名称**：AdsPower
- **官方URL**：https://www.adspower.com/
- **能力/适用点**：50+ 可定制指纹参数、25 类配置；可按 profile 设置 IP、语言、时区、字体、Canvas、WebGL、WebGPU、AudioContext（生成与所声明 OS/GPU 一致的成套指纹）；WebRTC 有独家 "Disable UDP" 模式；有字体与插件枚举自定义；支持 Windows/macOS/Linux/Android/iOS 指纹模板（可声称 Windows 11）；Chromium（SunBrowser）+ Firefox（FlowerBrowser）双内核；本地 API、RPA、多窗口同步；SOC 2 Type II / ISO 27001 认证。注意：第三方实测提示其默认时区可能不跟随代理，**每个新 profile 需手动把时区设为匹配代理 IP**。适合按站点指南逐项核对配置。
- **是否面向中国用户可获取**：是。中国团队产品，中文官网/中文客服，国内可直接下载使用。
- **价格档次**：免费版（永久，2 个 profile）/ Base（约 $5.4/月起，10 profiles）/ Pro（约 $30/月，100 profiles）/ Enterprise 定制。
- **证据URL**：
  - https://www.adspower.com/multi-account-management （双内核、OS 模板、20+ 参数）
  - https://www.adspower.com/blog/adspower-antidetect-browser-review （50+ 参数、时区/语言/字体/AudioContext/WebRTC Disable UDP、价格档位）
  - https://gologin.com/blog/anti-fingerprinting-browser/ （第三方实测：默认时区不跟随代理的警告，及价格）

## 2. GoLogin

- **名称**：GoLogin
- **官方URL**：https://gologin.com/
- **能力/适用点**：自研 Chromium 内核 Orbita；53 项指纹参数；第三方实测（Pixelscan/Iphey/CreepJS）显示时区、语言、地理位置**自动跟随代理 IP**，无需手动设置，指纹一致性通过检测；独立 cookies/指纹/代理 per profile；内置住宅/移动代理；REST API、Selenium/Puppeteer/Playwright、MCP server；云端启动。对"时区跟随代理 + 语言匹配"这条要求开箱即用程度最高。
- **是否面向中国用户可获取**：部分。官网可下载 Windows 客户端、有免费档，但界面与支付以英文/国际支付为主，无中文本地化服务的公开证据；国内访问与支付渠道需自行验证。
- **价格档次**：Forever Free（3 profiles）/ Professional（$4.5–19.5/月，10–100 profiles，年付价）/ Business（$59.5/月起）/ Enterprise（$149.5/月）/ Custom（$224.5/月起）。
- **证据URL**：
  - https://gologin.com/pricing/ （档位与包含项）
  - https://gologin.com/docs/general/account-and-billing/pricing （Free/Professional/Business/Enterprise/Custom 价格）
  - https://gologin.com/blog/anti-fingerprinting-browser/ （时区/语言/位置自动匹配代理的实测、推荐用法：每账号一个粘性代理）

## 3. 比特浏览器（BitBrowser）

- **名称**：比特浏览器（BitBrowser）
- **官方URL**：https://www.bitbrowser.cn/
- **能力/适用点**：北京比特互联科技有限公司开发，谷歌/火狐内核深度定制；每窗口模拟独立电脑信息+代理 IP，窗口间环境隔离；20+ 指纹参数含 UserAgent、Language、WebGL、Audio、Timezone、DNS、Cookie、Canvas、Fonts、WebRTC、LocalStorage、分辨率、地理坐标；**时区可自动按 IP 归属地同步**（也支持手动指定）；WebRTC 默认防护：强制走代理 IP 或可直接关闭；支持自定义字体列表。为该站点指南要求覆盖度最全的国产产品。
- **是否面向中国用户可获取**：是。国产、全中文官网与文档、人民币计价，国内支付方式（客户端内充值/开通套餐）。
- **价格档次**：永久免费（10 个环境，1 成员）/ Base 50 窗口 ¥50/月 / 100 窗口 ¥75/月 / 200 窗口 ¥125/月 / 定制版。
- **证据URL**：
  - https://www.bitbrowser.cn/team-work/all-browser-fingerprinting-support （时区按 IP 同步、WebRTC 防护模式、字体等 20+ 指纹）
  - https://www.bitbrowser.cn/price （人民币套餐档位）
  - https://doc.bitbrowser.net/zh （开发公司与产品定位）

## 4. MoreLogin（摩尔指纹浏览器）

- **名称**：MoreLogin
- **官方URL**：https://www.morelogin.com/zh
- **能力/适用点**：Chromium + Firefox 双引擎；每环境独立 Cookie、缓存、登录状态、代理与指纹；可配置 User-Agent、Canvas、WebGL、**字体**、**时区**（"匹配 IP"或自定义）、**语言**、**WebRTC**（转发/隐私/替换/真实/禁止 五种模式）；创建环境时智能匹配代理地区的语言、时区与地理位置；真实画布指纹技术；同步器、本地 API、Selenium/Puppeteer 自动化。中文帮助文档详细，参数粒度与站点要求吻合。
- **是否面向中国用户可获取**：是。中国团队产品，中文官网、中文帮助中心、中文客服。
- **价格档次**：永久免费（2 个环境 + 100 分钟云手机）/ Pro 按环境数与成员数定制（10 环境年付约 $5.4/月起）。
- **证据URL**：
  - https://www.morelogin.com/zh/pricing （免费档与 Pro 档）
  - https://support.morelogin.com/zh-CN/articles/10204297-%E7%8E%AF%E5%A2%83%E8%AE%BE%E7%BD%AE （时区匹配 IP、WebRTC 五模式等环境设置细节）
  - https://www.morelogin.com/zh/antidetect-browser （代理地区语言/时区/地理位置智能匹配）

## 5. NstBrowser（NST 指纹浏览器）

- **名称**：NstBrowser
- **官方URL**：https://www.nstbrowser.io/zh
- **能力/适用点**：Chromium 与 Firefox 引擎；**语言、时区根据环境 IP 自动生成**，也可手动调整；地理位置可禁止/按 IP 生成/手动；字体、WebRTC、Canvas、WebGL 等由算法生成一致值；RPA、API（支持时区与语言指纹参数设置）、Puppeteer/Playwright；Profile 持久化保留 Cookie/会话/指纹/本地数据；PC 端模拟仅支持 Windows（与 Win11 宿主匹配）。免费策略激进（免费订阅 + 按启动次数计费）。
- **是否面向中国用户可获取**：是。中文官网、中文文档，主打国内外跨境用户，免费可用。
- **价格档次**：免费订阅（Professional 功能免费，环境启动约 $0.01/次，按量计费）/ Professional / Enterprise 订阅（页面显示至约 $89.9/月 档）/ 内置代理另计（约 $0.4/GB）。
- **证据URL**：
  - https://docs.nstbrowser.io/zh/guide/fingerprint-browser/settings.html （语言/时区按 IP 自动生成，字体/WebRTC/Canvas/WebGL 参数）
  - https://www.nstbrowser.io/zh/pricing （免费订阅、按次计费与订阅档位）

## 6. Multilogin

- **名称**：Multilogin
- **官方URL**：https://multilogin.com/
- **能力/适用点**：2015 年起的老牌反检测浏览器；55+ 可定制参数，含 User-Agent、屏幕、**时区与语言**、WebGL/Canvas、**已安装字体与插件**；每 profile 独立 cookies/会话/指纹/代理；Mimic（Chromium，推荐）与 Stealthfox（Firefox，已停止内核更新）双内核；内置住宅代理 150+ 国家；加固的 Selenium/Playwright/Puppeteer 自动化。行业标杆但价格最高档，且官方建议代理位置、语言、时区自洽——需人工核对。
- **是否面向中国用户可获取**：有限。官网可下载 Windows 客户端并有免费档，但无中文界面/中文本地化服务的公开证据，以美元计价、国际支付为主；国内可用性需自行验证。
- **价格档次**：Free（5 个云端 profiles，无需信用卡）/ Pro（$11/月起，Pro 10；官网促销页显示 $7.08/月 起）/ Business（$40–89/月）/ Custom。
- **证据URL**：
  - https://multilogin.com/pricing/ （Free/Pro 档位）
  - https://multilogin.com/help/en_US/subscription-plan-comparison （Pro 10/20/50、Business 100/300 具体档位）
  - https://multilogin.com/antidetect/antidetect-browser/ （55+ 参数、时区/语言/字体/Canvas/WebGL 掩蔽）

---

## 结论（排序建议）

**首选比特浏览器或 MoreLogin**：两者都是中国团队产品、人民币/国内渠道可得、中文文档完整，且明确支持"时区/语言跟随代理 IP"与 WebRTC 强制走代理或禁用，正好覆盖站点指南的核心要求且不触碰系统设置。**若更看重第三方实测的指纹一致性，选 GoLogin**（时区/语言/位置自动跟随代理、通过三家指纹检测站），但需接受英文界面与国际支付；AdsPower 参数最深但默认时区需手动校正，Multilogin 适合作为企业级备选、性价比最低。无论选哪款，都应按站点要求为每个 profile 配独享粘性住宅 IP，并在启动后用 Pixelscan/BrowserScan 核对时区、语言、字体与 WebRTC 是否与代理一致。
