# 关于本站

## 网站运营与作者团队

**LinXiaoTao** — 独立开发者,基于 Claude Code 指纹检测的公开逆向分析构建了 Fuck Claude。本站是参考工具与教育型知识库,与 Anthropic 无任何关联。

## Fuck Claude 项目概述

**Fuck Claude** 是一套面向开发者与技术研究者的开源诊断工具与深度知识库。项目旨在帮助全球开发者与科研人员深入理解现代 AI 客户端（特别是 Anthropic 的 Claude Code 与 Web 界面）如何对运行环境进行区域指纹识别与风控检测。

通过检测系统时区、浏览器语言优先级、中文字体集、国产厂商专属字体、WebRTC 候选泄漏、国产浏览器引擎、硬件设备属性、Intl 区域解析、UTC 偏移量以及 Emoji 渲染等十项核心指标，本站提供透明的 0–100 加权风控分值及针对性的纯化配置建议。

## 编辑准则与研究方法论

Fuck Claude 知识库与动态资讯遵循严格的软件工程与技术验证准则：

- **实证逆向分析：** 所有防封指南与指纹权重均基于公开可复现的二进制逆向分析、Prompt 注入测试及抓包协议分析，杜绝道听途说与虚假信息。
- **持续跟踪与校验：** 每当 Anthropic 调整地区政策、上线新模型架构（如 Claude 3.7 Sonnet 混合推理）、推行文本水印或更新 API 速率限制，团队均会第一时间在 [防封指南](https://fuck-claude.vercel.app/zh/guides/) 与 [最新动态](https://fuck-claude.vercel.app/zh/news/) 中同步实测结论与应对策略。
- **开源透明与同行评审：** 本站的全部代码、评分算法及指南文稿均在 [GitHub](https://github.com/LinXiaoTao/FuckClaude) 上基于 MIT 协议完全开源，接受全球开发者的共同审查与改进。

## 技术范围与能力边界说明

本站检测器衡量的是常规 Web 页面所能合法读取的客户端与操作系统特征。开发者需知悉其技术边界：

- Anthropic 完整的后端风控体系还包括 IP 自治系统（ASN）评级、数据中心与住宅 IP 信誉库、支付卡发卡行 BIN 码与欺诈风控模型、账号注册生命周期及 API 异常调用模式等。
- 获得低风险评分意味着你的客户端本地环境不存在明显的指纹泄露；但这不能替代合规的 IP 质量与正规支付手段，亦不构成对任何第三方账号状态的绝对保证。

## 开源协议与社区协作

我们坚信开源透明的力量。项目仓库地址：[LinXiaoTao/FuckClaude](https://github.com/LinXiaoTao/FuckClaude)。欢迎通过 [联系我们](https://fuck-claude.vercel.app/zh/contact/) 页面或 GitHub Issues 提交技术探讨、算法优化建议及勘误。

## 独立声明

Fuck Claude 为独立的开源技术与教育研究项目，**与 Anthropic, PBC 无任何隶属、赞助、授权或官方关联**。Claude、Claude Code、Anthropic 及其相关标识均为 Anthropic, PBC 的注册商标或许可财产。
