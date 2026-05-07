# Vertex AI 迁移指南（AI Studio → Vertex Express Mode）

> 适用场景：Vercel 部署 + `@google/genai` SDK + 从 AI Studio 迁移到 Vertex AI
> 
> 最后更新：2026-05-07
> 基于川西旅行手帐项目的实际迁移经验

---

## 一、迁移前后对比

| | AI Studio（旧） | Vertex Express Mode（新） |
|---|---|---|
| SDK | `@google/genai` | `@google/genai`（同一个） |
| 初始化 | `new GoogleGenAI({ apiKey })` | `new GoogleGenAI({ vertexai: true, apiKey, apiVersion: 'v1' })` |
| API Key 来源 | aistudio.google.com | console.cloud.google.com |
| 计费 | AI Studio 免费额度 / 个人付费 | Google Cloud 项目计费（可用赠金） |
| 端点 | `generativelanguage.googleapis.com` | Vertex 路由（SDK 自动处理） |
| 代码改动 | — | 仅改初始化，其余不变 |

## 二、迁移步骤

### 步骤 1：在 Google Cloud Console 创建 API Key

1. 打开 [Google Cloud Console](https://console.cloud.google.com)
2. 选择你的项目（或创建新项目）
3. 确认已开启 **Vertex AI API**：
   - 导航到「APIs & Services」→「Enabled APIs」
   - 搜索 `Vertex AI API`，如未启用则点击启用
4. 创建 API Key：
   - 导航到「APIs & Services」→「Credentials」
   - 点击「Create Credentials」→「API Key」
   - 建议限制 Key 只能调用 Vertex AI API

### 步骤 2：修改代码

#### 改动 1：初始化逻辑

```javascript
// ============ 旧代码（AI Studio）============
const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

// ============ 新代码（Vertex Express Mode）============
function isVertexEnabled() {
  const v = process.env.GOOGLE_GENAI_USE_VERTEXAI;
  return ['1', 'true', 'yes', 'on'].includes(String(v || '').trim().toLowerCase());
}

function createGeminiClient() {
  const apiKey = process.env.GEMINI_API_KEY || process.env.GOOGLE_API_KEY;
  
  if (isVertexEnabled() && apiKey) {
    // Vertex Express Mode
    return new GoogleGenAI({
      vertexai: true,
      apiKey,
      apiVersion: 'v1',
    });
  }
  
  // 回退到 AI Studio（开发/测试环境）
  return new GoogleGenAI({ apiKey });
}
```

#### ⚠️ 关键约束

```javascript
// ❌ 错误！project/location 和 apiKey 不能同时传
new GoogleGenAI({
  vertexai: true,
  apiKey,
  project: 'my-project',     // 不要加这个
  location: 'us-central1',   // 不要加这个
});

// ✅ 正确！Express Mode 只需要这三个参数
new GoogleGenAI({
  vertexai: true,
  apiKey,
  apiVersion: 'v1',
});
```

#### 改动 2：模型名称

模型名称**不需要改**，Vertex 和 AI Studio 用的模型名完全一致：

```javascript
// 这些模型名在两种模式下通用
'gemini-3-flash-preview'
'gemini-2.5-flash'
'gemini-2.5-pro'
```

#### 改动 3：其他 API 调用

`generateContent`、`generateContentStream`、工具调用（function calling）等 API 用法**完全不变**：

```javascript
// 这些代码不需要任何修改
const response = await ai.models.generateContentStream({
  model: 'gemini-3-flash-preview',
  contents: [...],
  config: {
    systemInstruction: '...',
    tools: [...],
    maxOutputTokens: 2048,
  },
});
```

### 步骤 3：配置环境变量

#### Vercel 环境变量

在 Vercel → Project → Settings → Environment Variables 中设置：

```env
GOOGLE_GENAI_USE_VERTEXAI=true
GEMINI_API_KEY=你的_Google_Cloud_API_Key
```

可选（用于代码中标识项目，非 SDK 必需）：
```env
GOOGLE_CLOUD_PROJECT=你的_GCP_项目ID
GOOGLE_CLOUD_LOCATION=global
```

#### 本地开发 `.env`

```env
# 开发时可以继续用 AI Studio 的 Key（不设 VERTEX 标志）
GEMINI_API_KEY=你的_AI_Studio_Key

# 或者也用 Vertex
GOOGLE_GENAI_USE_VERTEXAI=true
GEMINI_API_KEY=你的_Google_Cloud_Key
```

### 步骤 4：部署验证

1. 推代码到 GitHub，等 Vercel 自动部署
2. 验证 AI 功能正常
3. 确认 Google Cloud Console → Billing 中可以看到 Vertex AI 的用量

## 三、两种 Vertex 模式说明

Vertex AI 在 `@google/genai` SDK 中有两种模式，**选 Express**：

### Express Mode ✅（推荐，当前使用）

```javascript
new GoogleGenAI({
  vertexai: true,
  apiKey: '...',
  apiVersion: 'v1',
});
```

- 适合 Vercel / Cloudflare Workers 等非 GCP 环境
- 只需要 API Key，不依赖 GCP 凭证
- 简单稳定

### ADC Mode ❌（不推荐用于 Vercel）

```javascript
new GoogleGenAI({
  vertexai: true,
  project: 'my-project',
  location: 'us-central1',
});
```

- 需要 Application Default Credentials
- 在 GCP 原生环境（Cloud Functions、Cloud Run）自动可用
- 在 Vercel 上需要额外配置服务账号 JSON，复杂且容易出错

### 什么时候用 ADC？

- 部署在 **Firebase Hosting + Cloud Functions** 时（ADC 自动可用）
- 部署在 **Cloud Run / GKE** 时
- 需要 **请求-响应日志**（BigQuery logging）时（Express Mode 不支持）

## 四、常见问题

### Q: `Project/location and API key are mutually exclusive`

A: 初始化时同时传了 `project/location` 和 `apiKey`。Express Mode 只传 `vertexai: true` + `apiKey` + `apiVersion`。

### Q: `Could not load the default credentials`

A: SDK 进入了 ADC 模式但找不到凭证。检查是否正确传了 `apiKey`。

### Q: 迁移后 AI Studio 的日志看不到了

A: 正常。AI Studio 日志只记录通过 AI Studio API Key 的请求。Vertex 请求不会出现在 AI Studio 日志中。替代方案：
1. 在代码中加 prompt debug 端点（参考本项目的 `/api/prompt-debug-ui`）
2. 如果必须要 GCP 原生日志，需要改用 ADC 模式 + 区域端点（非 global）

### Q: `gemini-3-flash-preview` 支持哪些 location？

A: 目前只支持 `global`。代码中不需要显式传 location，SDK 会自动处理。

### Q: 切换到 Vertex 后需要改 Firestore / 其他服务吗？

A: 不需要。Vertex AI 的迁移只影响 Gemini API 的调用方式，与 Firestore、Auth、Storage 等完全无关。

### Q: 旧的 AI Studio Key 还能用吗？

A: 能用，但会走 AI Studio 计费（免费额度用完后按量付费）。两个 Key 可以共存，通过 `GOOGLE_GENAI_USE_VERTEXAI` 环境变量切换。

## 五、新项目模板

开始新项目时，直接告诉 AI Agent：

```
这个项目的 Gemini 走 Vertex，不走 AI Studio。
部署环境是 Vercel。
请默认使用 Vertex Express Mode，而不是 ADC。

环境变量：
GOOGLE_GENAI_USE_VERTEXAI=true
GEMINI_API_KEY=（Google Cloud Console 创建的 Key）

代码初始化：
new GoogleGenAI({ vertexai: true, apiKey, apiVersion: 'v1' })

不要显式同时传 project/location 和 apiKey。
```

## 六、参考链接

- [Google Cloud Vertex AI 文档](https://cloud.google.com/vertex-ai/generative-ai/docs)
- [Gemini API 迁移指南（官方）](https://cloud.google.com/vertex-ai/generative-ai/docs/migrate/migrate-google-ai)
- [@google/genai SDK](https://www.npmjs.com/package/@google/genai)
- 本项目 Prompt Debug UI: `/api/prompt-debug-ui?token=YOUR_TOKEN`
