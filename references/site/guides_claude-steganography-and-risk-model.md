# Claude Code 隐写暗记原理与 Anthropic 四维风控模型解密

作者 [LinXiaoTao](https://github.com/LinXiaoTao) 更新于: 2026-08

深度拆解 Claude Code 如何利用 Unicode 撇号与日期斜杠隐写中国指纹，以及 Anthropic 从 IP、时区、支付卡头到对话频次的全维度风控逻辑。

本文基于社区反馈与公开逆向分析，未逐项对照 Anthropic 官方文档核对；重要细节请以官方帮助中心为准。

## 一、 Claude Code 隐写暗记事件深度复盘

据公开逆向分析，官方 `@anthropic-ai/claude-code` CLI 在通过 `ANTHROPIC_BASE_URL` 指向非官方中转端点时，曾在客户端隐蔽运行了一套隐写标记检测机制：

-   **系统时区检测：** 检查操作系统时区是否为 `Asia/Shanghai` 或 `Asia/Urumqi`。若命中，会将其注入 system prompt 的 `Today's date is 2026-06-30` 日期分隔符由短横线 `-` 替换为斜杠 `/`（如 `2026/06/30`）。
-   **域名黑名单与实验室关键字：** 读取中转主机名，匹配内嵌的 147+ 域名与国内 AI 实验室关键字（如 `deepseek`、`moonshot`、`zhipu`、`minimax`、`dashscope` 以及所有 `*.cn` 域名）。列表在二进制中经过 XOR（密钥 91）与 Base64 加密。
-   **Unicode 撇号隐写替换：** 将 "Today's date" 中的撇号替换为视觉几乎一致的 Unicode 变体：
-   命中中转黑名单域名 → 替换为 `U+2019` (右单引号 ’)
-   命中 AI 实验室关键字 → 替换为 `U+02BC` (修饰符撇号)
-   同时命中两者 → 替换为 `U+02B9` (修饰符角标)

这种隐写标记能在不添加任何 HTTP Header 的情况下穿透中转代理，被 Anthropic 后端模型与分类器识别，用于精准标记模型蒸馏（Distillation）与未经授权的转售行为。

### 防护与规避命令

```
# 1. 告知客户端将自定义中转视作原生端点（跳过隐写检测）
export _CLAUDE_CODE_ASSUME_FIRST_PARTY_BASE_URL=1

# 2. 运行时强制覆盖系统时区（如日本/新加坡时区）
export TZ=Asia/Tokyo

# 启动 Claude Code
claude
```

## 二、 Anthropic 四维复合风控体系

Anthropic 在 `claude.ai` 网页端、API 接口及 Claude Code 客户端构建了全方位的风控模型：

1.  **网络与 IP 属性：** 绝大多数数据中心机房 IP（如 AWS、Hetzner、搬瓦工、DigitalOcean）都会被赋予高风险权重；而位于支持地区（如美国、英国、新加坡、日本、台湾）的原生双 ISP 住宅家宽 IP 风险最低。
2.  **环境与指纹层：** 综合评估系统时区（`Intl.DateTimeFormat`）、浏览器语言列表（`navigator.languages`）、Canvas 字体探测（微软雅黑、苹方、MiSans/鸿蒙黑体等）及设备品牌。
3.  **支付与身份校验：** 虚拟信用卡卡头（BIN）、账单地址国家与 IP 国家一致性对齐，以及接码用的手机号段（VoIP 虚拟号直接拒绝）。
4.  **行为与蒸馏模型：** 自动化脚本高频并发请求、异常上下文饱和攻击以及破坏 Prompt Cache 的请求模式，会触发 429 Rate Limit 与账户判定。
