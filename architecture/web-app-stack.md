# Web 应用标准架构

> 适用于个人 Web App 项目，前端 + API + 数据库 + AI

## 架构图

```
用户（中国）
  ↓
Cloudflare（CDN + 自定义域名）
  ↓
Vercel（前端 + API 路由）
  ↓
Firebase Firestore（数据库）
  ↓
Vertex AI Express Mode（AI 能力）
```

## 各组件职责

| 组件 | 职责 | 为什么选它 |
|---|---|---|
| Cloudflare | DNS + CDN + 自定义域名 | 中国可访问，免费 |
| Vercel | 前端渲染 + API 路由 | git push 自动部署，对新手最友好 |
| Firebase | Firestore 数据库 | 免费额度够用，实时同步 |
| Vertex AI | Gemini API（Express Mode） | Google Cloud 赠金可用 |

## 环境变量模板

```env
# AI（Vertex Express Mode）
GOOGLE_GENAI_USE_VERTEXAI=true
GEMINI_API_KEY=your_google_cloud_api_key
GOOGLE_CLOUD_LOCATION=global

# Firebase
FIREBASE_PROJECT_ID=your_project_id

# API 安全
API_SECRET_TOKEN=your_random_token
VITE_API_TOKEN=same_as_above
```

## 新项目 Agent 指令模板

开始新项目时，直接告诉 AI Agent：

```
架构：Vercel + Cloudflare + Firebase
- 前端 + API：Vercel（React/Vite）
- 数据库：Firebase Firestore
- AI：Vertex AI Express Mode（@google/genai, vertexai: true, apiKey）
- CDN：Cloudflare（自定义域名）
- 环境变量：GOOGLE_GENAI_USE_VERTEXAI=true, GEMINI_API_KEY, GOOGLE_CLOUD_LOCATION=global
- API 鉴权：x-api-token header + query param 双模式
```

## 不要用什么

- ❌ Firebase Hosting + Cloud Functions（部署复杂，冷启动慢）
- ❌ Vertex ADC 模式在 Vercel 上（需要配服务账号 JSON，容易出错）
- ❌ BigQuery 日志（对个人项目过重，代码层面加 debug 端点更实用）
