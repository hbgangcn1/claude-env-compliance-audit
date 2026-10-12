# Claude Code 安全配置与第三方 API 中转平滑降级规范

*Claude Code 与 API 规范 · 5 分钟阅读*

作者 [LinXiaoTao](https://github.com/LinXiaoTao)·更新于: 2026-08

安全配置 ANTHROPIC_BASE_URL 与隐藏主机名，通过 TZ 与环境变量防护，以及如何选择不破坏 Prompt Cache 的优质 API 中转。

> 本文基于社区反馈与公开逆向分析，未逐项对照 Anthropic 官方文档核对；重要细节请以官方帮助中心为准。

Claude Code 与自定义 API 集成上传的不只是 Prompt——还包括环境元数据、路由选择与重试行为，上游系统可能将其与账号健康度关联分析。本文聚焦**运维安全**：一致的 Shell 配置、网关选型，以及在配额收紧时的平滑降级。目标是可靠的工程工作流，而非规避 Anthropic 政策。

请先完成干净的基础环境：[环境清理与 IP 配置](/zh/guides/environment-cleanup-and-ip-setup/) 与 [VPN 与代理选型](/zh/guides/vpn-and-proxy-selection/)，再调整 CLI 变量。若运营中转或多模型路由，请交叉阅读 [国产与开源平替](/zh/guides/domestic-and-open-source-alternatives/) 中的降级路径。

## 一、Claude Code 开发者安全环境配置

Claude Code 在启动时读取 Shell 环境变量。时区、代理或 Base URL 不一致会导致莫名报错，或使 CLI 行为与浏览器会话脱节。应把终端视为与浏览器同等重要的运行环境。

### 核心环境变量

```
# ~/.zshrc 或 ~/.bashrc — 每个交互式 Shell 加载

# 1. 显式声明 Anthropic 兼容端点
export ANTHROPIC_BASE_URL="https://your-gateway.example.com"
export ANTHROPIC_API_KEY="sk-your-key"

# 2. 使用可信、形态接近官方的网关时，减少客户端不匹配告警
export _CLAUDE_CODE_ASSUME_FIRST_PARTY_BASE_URL=1

# 3. 终端时区与文档化的运营地区一致
export TZ=Asia/Tokyo   # 或 America/Los_Angeles — 与代理出口地理匹配

# 4. CLI HTTPS 走与浏览器相同的干净代理（如需要）
export HTTP_PROXY="http://127.0.0.1:7890"
export HTTPS_PROXY="http://127.0.0.1:7890"
export NO_PROXY="localhost,127.0.0.1,.internal"
```

### 配置检查清单

1. **单一事实来源：**将变量写入 `~/.zshrc`（macOS）或专用 `~/.claude/env` 并在 Shell 中 source——避免只在某个终端 tab 临时 export。
2. **启动前验证：**在启动 Claude Code 的同一窗口执行 `env | grep -E 'ANTHROPIC|TZ|PROXY'`。
3. **浏览器与 CLI 地理一致：**若浏览器 Profile 通过防指纹工具设为东京时区，CLI 不应仍报告 `Asia/Shanghai`。
4. **Document Base URL 变更：**从直连 Anthropic 切到中转时，同步更新内部 Runbook，避免团队成员混用 endpoint。
5. **密钥卫生：**勿将 API Key 提交仓库；env 文件 `chmod 600`，中转 Key 建议季度轮换。

### `_CLAUDE_CODE_ASSUME_FIRST_PARTY_BASE_URL` 做什么、不做什么

该标志让 Claude Code 在某些客户端检查中，将非默认 `ANTHROPIC_BASE_URL` 视作标准 Anthropic 端点，便于企业内网已审批网关接入。它**不会**改变服务端政策、计费或账号资格。关于客户端元数据仍可能被如何评估，见 [Claude 隐写与风险模型](/zh/guides/claude-steganography-and-risk-model/)。

### 常见误配故障

- **代理分裂：**浏览器走 VPN、CLI 不走——请求来自不同 ASN。现象：网页聊天正常，CLI 403。
- **`NO_PROXY` 过期：**本机网关 `127.0.0.1` 须绕过公司代理，否则连接被重置。
- **Key 类型错误：**组织 Key 与项目 Key 在限流下行为不同，请在 Anthropic 控制台确认类型。
- **交互式 vs 非交互 Shell：**CI 调用 Claude Code 须 source 同一 env 文件；笔记本 login shell 常掩盖缺失 export。

## 二、第三方 API 中转网关选型规范

当直连 `api.anthropic.com` 受路由或采购限制时，优质中转可提升可用性——前提是协议保真。劣质网关会静默剥离 Header、改写 Prompt 或引入延迟尖刺，表现像「模型变笨了」。

### 网关评估矩阵

| 维度 | 合格 | 不合格（应更换） |
| --- | --- | --- |
| Prompt Cache Header | 原样转发 `anthropic-beta: prompt-caching-2024-07-15`（或当前 beta） | 剥离 beta；cache\_read 恒为 0 |
| System Prompt 完整性 | 字节级一致 relay | 注入广告、水印或「帮助性」前缀 |
| 流式 SSE | 保留事件边界与 tool delta | 缓冲完整响应再输出；破坏 tool UI |
| 错误透明 | 透传 Anthropic 状态码与 body | 一切映射为泛化 502 |
| 主机名中性 | 中性域名（如 `api.yourcorp.net`） | 含无关 AI 厂商关键字或纯 Claude 流量走 `.cn` 域名 |
| TLS | 有效公网 CA、HSTS、无 MITM 拆链 | 企业 SSL inspection 未更新客户端信任库 |

### Prompt Cache 保留（成本影响）

Anthropic Prompt Caching 可将重复输入成本降低约 90%。丢弃 Cache Header 的中转会在每一轮按全价 input 计费——长 system prompt 场景常见 4–10 倍账单惊喜。用两次请求验证：

```
# 请求 1：创建缓存
curl -s "$ANTHROPIC_BASE_URL/v1/messages" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: prompt-caching-2024-07-15" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-20250514","max_tokens":64,
       "system":[{"type":"text","text":"'$(python3 -c "print('x'*5000)")'",
         "cache_control":{"type":"ephemeral"}}],
       "messages":[{"role":"user","content":"ping"}]}'

# 请求 2：usage 中应出现 cache_read_input_tokens > 0
```

深度优化见 [API 高级优化](/zh/guides/api-advanced-optimization/)。

### 上线前尽职调查

- 24 小时合成探针（每 15 分钟一条消息），测 p95 延迟与错误率。
- 采样 request ID，确认应用可将网关日志与上游 ID 关联排障。
- 审阅数据处理协议：部分中转为滥用检测记录 Prompt—— regulated workload 不可接受。
- 处理源码时，优先 VPC 自建 LiteLLM、One-API，而非来源不明的公开 reseller。

账号级稳定性仍取决于注册卫生——若接入中转 coincides 新建组织，请读 [账号注册与支付防封](/zh/guides/account-registration-and-payment-antiban/)。

## 三、配额超限（429）与平滑降级机制

Anthropic 速率限制在组织与模型档位层面生效，而非单个 Key 孤立计算。Agent 突发、CI 并行与无界重试会耗尽共享配额，阻塞同 org 内无关服务。

### 限制类型（简化）

| 信号 | HTTP | 含义 | 安全响应 |
| --- | --- | --- | --- |
| 速率限制 | 429 | 每分钟请求/Token 过多 | 指数退避 + 抖动；降并发 |
| 上游过载 | 529 | 容量不足 | 短退避；可选降模型档 |
| 鉴权/政策 | 403 | Key 无效或拒绝访问 | 勿盲目重试；查账号状态 |
| 错误请求 | 400 | Schema 或 Token 上限 | 修 payload；裁上下文 |

### 重试与降级模式

```
// TypeScript — 429/529 重试，持续压力时 failover
async function callClaudeWithFallback(prompt: string, opts: CallOpts) {
  const maxRetries = 4;
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await callClaudeAPI(prompt, opts);
    } catch (error: any) {
      const status = error?.status;
      if (status === 429 || status === 529) {
        const delay = Math.min(30_000, 500 * 2 ** attempt + Math.random() * 250);
        await sleep(delay);
        continue;
      }
      if (status === 403) throw error;
      throw error;
    }
  }
  console.warn('Claude 配额饱和，切至备份上游');
  return await callFallbackAPI(prompt, opts); // DeepSeek / GLM / 本地 — 见平替指南
}
```

### 并发控制

- **CI 令牌桶：**每 org 限制并行 Agent 任务（如最多 3 个）。
- **请求合并：**可能时将多文件 lint 修复合并为单次调用。
- **熔断：**连续 N 次 429 后开路 60 秒并自动 failover——避免重试风暴。
- **可观测性：**导出 `claude_requests_total{status}`、`fallback_invocations_total`、p95 延迟；fallback 率超 15% 持续 10 分钟则告警。

### 403 与 429：勿混为一谈

429 是时间维度的压力——退避后恢复。403 常意味凭证吊销、地理限制或账号状态变化。未诊断就轮换 Key 盲目重试可能加速 enforcement。请用 [故障排查指南](/zh/guides/troubleshooting-guide/) 分类错误后再动基础设施。

## 常见问题

### 系统时区已正确，还要设 `TZ` 吗？

若 OS、代理出口与浏览器 Profile 已对齐，显式 `TZ` 可能冗余——但 CI 容器常默认 UTC。显式 export 让笔记本与服务器行为可预期。

### 公开 API reseller 处理公司源码「安全」吗？

视同任何第三方子处理器：审日志、保留期与子处理器清单。专有代码场景下，VPC 自建中转严格优于匿名 reseller。

### 换网关后 Cache 省钱效果为何消失？

最常见是中转剥离 `anthropic-beta` 或重排 system 块，导致 cache key 失效。任何网关变更后重做两次请求的 cache 测试。

### 浏览器与 Claude Code 能否用不同代理？

技术上可以，但 divergence 风险上升。除非实验账号隔离 Profile，否则 Claude 相关流量宜走同一条已文档化 egress。

### 429 时 Claude Code 应降级到哪个模型？

Claude Code 本身无内置多模型 failover——在网关（One-API/LiteLLM）实现，或暂时停用 Claude Code、改用指向备份 API 的 IDE 插件。路由表见 [国产与开源平替](/zh/guides/domestic-and-open-source-alternatives/)。

### curl 成功、Claude Code 失败如何 debug？

1. 对比 Header：用 mitmproxy 或网关 access log 抓 curl vs CLI。
2. 确认启动 Shell 的 env：IDE 集成终端有时不加载 login rc。
3. 检查 Node/fetch 代理：部分版本须设 `GLOBAL_AGENT_HTTP_PROXY` 才认 `HTTPS_PROXY`。
4. 排查企业网 TLS interception。

### assume first-party base URL 会影响 Anthropic 计费吗？

不会。计费跟随实际服务请求的 API Key 与组织。网关转发 Anthropic 则付 Anthropic；网关换模型则付网关所调 Provider。
