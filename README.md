# claude-env-compliance-audit

判断**本机是否符合 Claude 账号审核/防封标准**的跨代理 Agent Skill：按 [fuck-claude.vercel.app](https://fuck-claude.vercel.app/zh/) 的 10 信号加权模型 + Anthropic 官方支持地区规则，跑只读审计命令，产出逐条 PASS/FAIL 差距报告与修复方案。

> 社区教育性参考，非 Anthropic 官方文档。站点内容为社区逆向整理；本仓库的站点摘要仅为便于检索的中文整理，原始内容版权归 [fuck-claude.vercel.app](https://fuck-claude.vercel.app/zh/) 及其作者所有，如需最全文本请访问原站。

## 特性

- **双平台命令包**：Windows（PowerShell）与 macOS（bash/systemsetup/scutil）各一套，先判宿主 OS 再执行。
- **R1–R20 差距细则**：时区/语言/区域/字体/WebRTC/IP 类型/DNS/Claude Code 环境/支付与手机号身份维度，每条给 PASS 条件、修复归类（本机命令 / 第三方方案 / 无解）。
- **第三方方案速查（带来源 URL）**：住宅 IP 出口、字体指纹隔离、WebRTC 防泄漏、支持地区手机号（Ultra Mobile PayGo / Tello / Giffgaff 等）、支付路径（App Store 礼品卡 / 虚拟卡盘点与避坑）。
- **资格结论（R15）**：Anthropic 按人身所在地与账号创建地判定资格，附一手原文核验 —— 任何工具都不能把不合资格账号变合规。
- **全量语料**：`references/site/` 为站点 25 页中文摘要（25/25 覆盖校验）；`references/research/` 为 7 份带来源的第三方调研。

## 安装（任选一处，name 与目录同名）

| 代理 | 位置 |
|---|---|
| OmO / DSH / OpenClaw / OpenCode / Codex | `~/.agents/skills/claude-env-compliance-audit/` |
| Hermes（NousResearch） | `~/.hermes/skills/claude-env-compliance-audit/` |

克隆到对应目录即可：

```bash
# 通用（五代理共享根）
git clone https://github.com/hbgangcn1/claude-env-compliance-audit.git ~/.agents/skills/claude-env-compliance-audit

# Hermes
git clone https://github.com/hbgangcn1/claude-env-compliance-audit.git ~/.hermes/skills/claude-env-compliance-audit
```

之后对任意代理说"帮我查本机是否符合 Claude 防封标准"即可触发。

## 目录结构

```
claude-env-compliance-audit/
├── SKILL.md                  # skill 主体（审计命令包 + R1-R20 细则 + 方案速查）
└── references/
    ├── site/                 # 站点 25 页中文摘要 + 覆盖校验 + URL 清单
    └── research/             # 7 份第三方调研（IP/字体/WebRTC/资格/支付手机号）
```

## 声明

- 审计命令**全部只读**，不改动任何系统设置。
- 检测结论是估计，不是 Anthropic 的真实判定；账号长期存活取决于注册地区、支付身份与使用模式。
- 请遵守 Anthropic 使用政策与当地法律，自行判断如何使用本仓库内容。
