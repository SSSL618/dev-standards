# AI Prompt 设计规范

> 基于实际项目经验总结的 Prompt 优化策略

## 分层上下文架构

```
Layer 1: System Prompt（永远发）
  - 角色设定 + 回复规则（~500 字）

Layer 2: 业务数据（按需发）
  - 行程/订单/项目等背景数据
  - 通过关键词匹配判断是否需要

Layer 3: 对话历史（滑动窗口）
  - 最近 10 条消息（5 轮）
  - 不是全部历史

Layer 4: 当前用户输入
```

## 对话窗口大小选择

| 消息数 | 轮数 | 适用场景 |
|---|---|---|
| 6 | 3 轮 | 简单问答、工具类 |
| 10 | 5 轮 | 通用对话（推荐） |
| 20 | 10 轮 | 复杂多步骤任务 |
| 30+ | 15+ 轮 | 不推荐，token 浪费严重 |

## System Prompt 规则

1. **简洁**：核心角色 + 关键规则控制在 500 字以内
2. **业务数据分离**：不要把大段数据写死在 system prompt 里，按需注入
3. **明确约束**：字数限制、语言要求、格式要求写清楚
4. **禁止项比允许项更重要**：明确说"不要做什么"

## 成本估算

```
1 中文字 ≈ 1.5 tokens
1000 字 ≈ 1500 tokens
Gemini 3 Flash: 输入 $0.10/M tokens, 输出 $0.40/M tokens

示例：
- 500 字 system prompt = ~750 tokens = $0.000075
- 10 条对话（~3000 字） = ~4500 tokens = $0.00045
- 每轮总成本 ≈ $0.0005（约 ¥0.004）
- 一天 100 轮 ≈ ¥0.4
```

## Prompt Debug

在每个项目中加入 prompt debug 端点，方便查看实际发送给 API 的内容：

```javascript
app._lastPromptDebug = {
  timestamp: new Date().toISOString(),
  model: MODEL_NAME,
  systemInstruction: contextText,
  messages: effectiveMessages.map(m => ({ role: m.role, content: m.content })),
  totalEstimatedChars: totalChars,
};
```

配合可视化 UI 页面查看（参考川西项目的 `/api/prompt-debug-ui`）。
