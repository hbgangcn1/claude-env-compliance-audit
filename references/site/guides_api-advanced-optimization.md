# Claude API 进阶优化：Token 控制与 Prompt Cache 实战

*Claude Code 与 API 规范 · 1 分钟阅读*

作者 [LinXiaoTao](https://github.com/LinXiaoTao)·更新于: 2026-08

深度解析 Prompt Cache 最大化利用策略、Token 计费优化技巧、长上下文窗口管理、并发控制与流式输出最佳实践。

> 本文基于社区反馈与公开逆向分析，未逐项对照 Anthropic 官方文档核对；重要细节请以官方帮助中心为准。

## 一、 Prompt Cache 原理与最大化利用策略

Anthropic 的 Prompt Caching 功能可将重复上下文的成本降低 90%，延迟降低 85%。理解缓存行为对 API 优化至关重要：

### 缓存行为规则

- **最小缓存大小：** 缓存块必须 ≥ 1024 tokens。更小的块不会被缓存。
- **缓存 TTL：** 缓存内容在 5 分钟不活动后过期。在 5 分钟内重复使用同一缓存会延长 TTL。
- **缓存键：** 缓存通过精确内容匹配作为键。更改一个字符会使缓存失效。
- **缓存位置：** 仅 `system` 角色消息和最后的 `user` 消息支持缓存。中间消息无法缓存。

### 最优缓存策略

```
const systemPrompt = `你是一位专家软件架构师...
[很少变化的大型 5000 token 上下文]`;

// 标记系统提示词进行缓存
const response = await anthropic.messages.create({
  model: "claude-sonnet-4.5-high",
  max_tokens: 2048,
  system: [
    {
      type: "text",
      text: systemPrompt,
      cache_control: { type: "ephemeral" }, // 缓存此块
    },
  ],
  messages: [
    { role: "user", content: "重构这个函数..." },
  ],
});
```

## 二、 Token 计费优化与长上下文窗口管理

Claude API 定价不对称：输入 token 成本低于输出 token，缓存 token 成本比输入 token 低 90%。构建对话以最大化缓存利用：

### 成本对比（Claude Sonnet 4.5 High）

| Token 类型 | 每百万 Token 成本 | 相对成本 |
| --- | --- | --- |
| 输出 Tokens | $15.00 | 100x |
| 输入 Tokens | $3.00 | 20x |
| 缓存输入 Tokens | $0.30 | 2x |
| 缓存写入 Tokens | $3.75 | 25x |

### 长上下文窗口管理

```
// 多轮对话的滑动窗口方法
function maintainContextWindow(history: Message[], maxTokens: number = 180000) {
  let totalTokens = estimateTokenCount(history);

  while (totalTokens > maxTokens && history.length > 2) {
    // 首先移除最旧的非系统消息
    history.splice(1, 2); // 移除一对用户-助手消息
    totalTokens = estimateTokenCount(history);
  }

  return history;
}
```

## 三、 并发请求控制与速率限制应对

Anthropic 实施组织级速率限制。实现客户端并发控制以避免 429 错误：

### 令牌桶速率限制器

```
class RateLimiter {
  private tokens: number;
  private lastRefill: number;
  private refillRate: number; // 每秒令牌数
  private capacity: number;

  constructor(requestsPerMinute: number) {
    this.capacity = requestsPerMinute;
    this.tokens = requestsPerMinute;
    this.lastRefill = Date.now();
    this.refillRate = requestsPerMinute / 60;
  }

  async acquire(): Promise {
    this.refill();

    while (this.tokens < 1) {
      const waitTime = (1 - this.tokens) / this.refillRate * 1000;
      await new Promise(resolve => setTimeout(resolve, waitTime));
      this.refill();
    }

    this.tokens -= 1;
  }

  private refill(): void {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(this.capacity, this.tokens + elapsed * this.refillRate);
    this.lastRefill = now;
  }
}

const limiter = new RateLimiter(50); // 每分钟 50 个请求

async function callClaude(prompt: string) {
  await limiter.acquire();
  return await anthropic.messages.create({...});
}
```

## 四、 流式输出与超时重试最佳实践

流式响应减少首 token 时间并支持渐进式渲染。与指数退避结合实现稳健的错误处理：

### 带重试逻辑的流式输出

```
async function* streamWithRetry(prompt: string, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const stream = await anthropic.messages.stream({
        model: "claude-sonnet-4.5-high",
        max_tokens: 4096,
        messages: [{ role: "user", content: prompt }],
      });

      for await (const chunk of stream) {
        if (chunk.type === "content_block_delta") {
          yield chunk.delta.text;
        }
      }

      return; // 成功，退出重试循环

    } catch (error: any) {
      if (error.status === 529 && attempt < maxRetries - 1) {
        // 过载错误，带退避重试
        const delay = Math.min(1000 * Math.pow(2, attempt), 30000);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
}
```
