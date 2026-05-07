# Vercel + Cloudflare + Firebase 部署执行手册

> 本文档合并了项目实战手册和通用部署 SOP。踩坑记录（第 10 节）来自真实项目经验，是最有价值的部分。

## 1. 适用范围

适用于这类项目：

- 前端部署在 `Vercel`
- 域名入口放在 `Cloudflare`
- 数据层和部分后端能力依赖 `Firebase / Firestore`
- AI 能力依赖 `Gemini / OpenAI / Claude` 等服务端密钥
- 目标用户是自己和少量朋友，不是大规模商业化产品

不适用于：

- 仍然强依赖 `SQLite + WebSocket + 常驻 Express` 的旧项目
- 需要中国大陆生产级稳定保障的大规模公众产品

---

## 2. 推荐架构

```text
浏览器
  -> Cloudflare
  -> Vercel
  -> Vercel Functions / API
  -> Firebase / Gemini / 其他第三方服务
```

规则：

1. 前端页面放 `Vercel`
2. 域名放 `Cloudflare`
3. 数据放 `Firestore`
4. 所有关键写入和密钥调用都走服务端
5. 不把 `SQLite` 当生产数据层
6. 不在 Vercel 上依赖 `WebSocket` 作为核心同步机制

---

## 3. 部署前改造检查

在点部署前，先确认项目已经满足这些条件：

### 3.1 数据层

- 已从 `SQLite` 迁到 `Firestore`，或至少已支持 Firestore 主路径
- 服务端和前端都能读写 Firestore
- 已有 seed 脚本初始化 Firestore

### 3.2 实时能力

- 不再依赖 Vercel 上的 `WebSocket`
- 前端实时刷新优先使用 `Firestore listener`

### 3.3 后端结构

- 不能只保留一个本地 `Express + express.static()` 的假设
- 需要能跑在 `Vercel Functions / api/*` 结构里
- 需要尽量避免把核心能力压在深层多段动态路由上

### 3.4 本地验证

至少本地验证通过这些功能：

- 首页可打开
- todo 勾选后刷新可保留
- AI chat 正常
- 导出正常
- 基础编辑写入后刷新可保留

### 3.5 Vercel 路由检查

部署前先确认 `api/` 目录没有路由冲突。

高风险形态：

- `api/trip/[...route].js`
- 同时再放一批 `api/trip/[tripId]/...`、`api/trip/[tripId]/day/[dayNum]/...`

这类组合在 Vercel 上很容易直接构建失败，错误形态通常是：

```text
Two or more files have conflicting paths or names.
```

原则：

1. `catch-all` 和同层深度显式动态路由不要混搭
2. 能用顶层单段 API 解决的问题，优先不要做深层路径
3. 真要保留多段路径，先本地梳理 `api/` 目录，确保无冲突

---

## 4. Firebase 配置步骤

### 4.1 创建项目

在 [Firebase Console](https://console.firebase.google.com/)：

1. 创建项目
2. 记录 `Project ID`

### 4.2 开启 Firestore

1. 进入 `Firestore Database`
2. 创建数据库
3. 选择区域

建议区域：

- `asia-southeast1`
- 或 `asia-east1`

### 4.3 创建 Web App

1. 添加 `Web App`
2. 拿到前端配置：

```env
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_AUTH_DOMAIN=
VITE_FIREBASE_PROJECT_ID=
VITE_FIREBASE_STORAGE_BUCKET=
VITE_FIREBASE_MESSAGING_SENDER_ID=
VITE_FIREBASE_APP_ID=
```

### 4.4 创建服务端私钥

在 `Project Settings -> Service Accounts`：

1. 选 `Node.js`
2. 点击 `Generate new private key`
3. 下载 JSON

从 JSON 中取：

```env
FIREBASE_PROJECT_ID=
FIREBASE_CLIENT_EMAIL=
FIREBASE_PRIVATE_KEY=
```

---

## 5. 本地环境变量写法

本项目当前约定是：

- 前端会读 `.env`
- 服务端也只读 `.env`

所以这一步不要只写 `.env.local`。

应写入项目根目录：

`your-project-path/.env`

模板如下：

```env
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_AUTH_DOMAIN=
VITE_FIREBASE_PROJECT_ID=
VITE_FIREBASE_STORAGE_BUCKET=
VITE_FIREBASE_MESSAGING_SENDER_ID=
VITE_FIREBASE_APP_ID=

FIREBASE_PROJECT_ID=
FIREBASE_CLIENT_EMAIL=
FIREBASE_PRIVATE_KEY=

GEMINI_API_KEY=
AMAP_WEB_KEY=
AMAP_KEY=
QWEATHER_KEY=
```

### private key 写法

`FIREBASE_PRIVATE_KEY` 必须写成单行，例如：

```env
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
```

要求：

1. 整行放在一行里
2. 用双引号包住
3. 换行写成 `\n`
4. 不要直接粘多行原始私钥

---

## 6. Firestore 初始化

本项目的初始化命令：

```bash
cd "/Users/zaoluli/Documents/AI Tools/川西旅行手帐"
npm run seed:firestore
```

预期结果：

- trip 文档写入成功
- days 子集合写入成功
- todos 子集合写入成功

如果失败，先看：

1. `FIREBASE_PROJECT_ID`
2. `FIREBASE_CLIENT_EMAIL`
3. `FIREBASE_PRIVATE_KEY`
4. Firestore 是否已经开启

---

## 7. 本地验证步骤

### 7.1 生产模式本地启动

这个项目本地开发时，`npm run dev` 可能受本机 watch / 端口监听影响。

更稳的验证方式是：

```bash
cd "/Users/zaoluli/Documents/AI Tools/川西旅行手帐"
npm run build
npm start
```

然后打开：

[http://localhost:8080](http://localhost:8080)

### 7.2 验证清单

按这个顺序测：

1. 首页是否正常打开
2. todo 勾选后刷新是否保留
3. AI chat 是否正常
4. chat 刷新后是否保留
5. 导出 PDF 是否正常
6. 可编辑字段刷新后是否保留

---

## 8. Vercel 部署步骤

### 8.1 GitHub

先把项目推到独立 GitHub 仓库。

### 8.2 导入项目

在 [Vercel Dashboard](https://vercel.com/dashboard)：

1. `Add New`
2. `Project`
3. 选择 GitHub 仓库

### 8.3 环境变量

推荐准备一个专门给 Vercel 导入的文件，例如：

`your-project-path/.env.vercel`

只保留部署所需变量：

```env
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_AUTH_DOMAIN=
VITE_FIREBASE_PROJECT_ID=
VITE_FIREBASE_STORAGE_BUCKET=
VITE_FIREBASE_MESSAGING_SENDER_ID=
VITE_FIREBASE_APP_ID=

FIREBASE_PROJECT_ID=
FIREBASE_CLIENT_EMAIL=
FIREBASE_PRIVATE_KEY=

GEMINI_API_KEY=
AMAP_WEB_KEY=
AMAP_KEY=
QWEATHER_KEY=
```

导入方式：

1. 在 Vercel 项目导入页进入 `Environment Variables`
2. 使用 `Import .env`
3. 把 `.env.vercel` 内容整体粘进去
4. 目标环境选 `Production and Preview`

### 8.4 部署

点 `Deploy`。

默认成功后会得到一个 `.vercel.app` 域名。

### 8.5 部署后先看什么

不要只看“Deploy 成功”的字样，优先确认三件事：

1. 当前生产域名绑定到的是最新成功部署
2. 失败的旧 deployment 没有被误当成当前线上版本
3. 关键 API 真实可访问

建议直接测：

```bash
curl -sSI https://your-domain/api/health
curl -sSI https://your-domain/api/chat-history
curl -sSI https://your-domain/api/export-trip
```

注意：

- `HEAD` 返回 `405` 不一定是坏事，很多函数只允许 `GET/DELETE`
- 这时要继续用真实 `GET` 测一次，不要误判

### 8.6 推荐的 API 路径策略

如果项目里有“聊天历史”“导出”“刷新天气/路线”这类功能，优先使用**顶层单段 API**：

- `/api/chat-history`
- `/api/export-trip`
- `/api/refresh-weather`

不优先推荐这种形态作为第一版上线策略：

- `/api/trip/:tripId/chat`
- `/api/trip/:tripId/export`
- `/api/trip/:tripId/day/:dayNum/recalculate`

原因：

1. 路径更深，Vercel 路由问题更难排查
2. 一旦失败，表现通常是线上 404，但本地完全正常
3. 顶层 API 更容易独立验证

### 8.7 页面更新了，不代表数据就对

这次真实踩过：

- 页面结构已经是新版本
- 但标题仍显示 `test`

根因不是部署失败，而是数据层里真的有 `trip.title = test`。

经验：

1. 先确认线上 bundle 是否更新
2. 再确认真实业务数据是否合理
3. 对通用字段优先在统一数据层做兜底

---

## 9. Cloudflare 绑定步骤

### 9.1 在 Vercel 添加域名

在 Vercel 项目里添加：

```text
your-domain.example.com
```

### 9.2 在 Cloudflare 加 DNS

到 Cloudflare 的 `your-domain.example.com -> DNS Records`：

新增：

- Type: `CNAME`
- Name: `trip`
- Target: `cname.vercel-dns.com`
- Proxy status: `DNS only`

注意：

- 第一轮验证时先用 `DNS only`
- 不要先开 `Proxied`

### 9.3 验证域名

最终访问：

[https://your-domain.example.com](https://your-domain.example.com)

---

## 10. 这次踩过的坑

### 10.1 `.env.local` 不够

问题：

- 前端能读 `.env.local`
- 但服务端当前只读 `.env`

结果：

- 前端走 Firestore
- 服务端静默退回 SQLite

结论：

- 当前项目必须把 Firebase 服务端变量写进 `.env`

### 10.2 `FIREBASE_PRIVATE_KEY` 格式错误

问题：

- 直接把多行私钥原样粘进 `.env`

结果：

- 服务端无法正确初始化 Firebase Admin

正确写法：

```env
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
```

### 10.3 旧项目不能直接按 Vercel 静态站部署

问题：

- 旧代码还带 `SQLite + WebSocket + express.static`

结果：

- 不能直接作为纯 `Vite` 静态项目部署

处理方式：

- 先改为 Firestore 主路径
- 去掉 Vercel 对 `WebSocket` 的依赖
- API 改成 Vercel 兼容结构

### 10.4 Vercel 的 `DNS Change Recommended` 不一定是阻塞

这次实际情况：

- 域名已经能正常打开
- Vercel 仍显示 `DNS Change Recommended`

原因：

- 它更像优化建议，不一定代表站点不可访问

处理原则：

- 先验证 `https://your-domain.example.com` 是否能正常打开
- 能正常访问，就先继续验收

### 10.5 Vercel 显示“部署成功”，不等于生产真的更新了

这次真实踩到的坑：

- 新代码已经 push
- Vercel 里也出现了新 deployment
- 但它其实是 `Error / Stale`
- 生产域名仍然指向更早的成功提交

结果：

- 你以为线上已经是新代码
- 实际访问还是旧版本

结论：

部署后一定要看：

1. 最新 deployment 是否 `Ready`
2. 当前 production 指向的是不是这次提交
3. 线上 API 返回是否符合新改动

### 10.6 深层 `api/trip/*` 路由在线上可能比本地更脆

这次实际表现是：

- 本地 `npm test`、`npm run build` 都通过
- 某些深层 API 本地正常
- 线上却出现 404 或构建冲突

结论：

- 不要把“本地路由可用”直接等价成“Vercel 线上可用”
- 先从顶层单段 API 起步，成功率更高

### 10.7 `npm run dev` 不一定稳

这次本机出现过：

- watch 相关问题
- 监听异常

更稳的办法：

```bash
npm run build
npm start
```

### 10.8 Firestore 写 `undefined` 会崩

问题：

- Firestore 不接受 `undefined` 字段

结果：

- 更新 todo 时服务端崩溃

结论：

- 所有 Firestore 写入前都要过滤 `undefined`

### 10.9 浏览器只做最后一跳，别把它当主调试面板

这次一个明确经验是：

- 浏览器适合登录、点按钮、配 DNS、看最终页面
- 不适合当主要调试界面

更省时的顺序：

1. 先用本地代码、`curl`、日志、命令行确认问题

### 10.10 自动计算的真正脆弱点是地点解析层

路线、海拔、天气本身不是写死值，实际都依赖外部 API。

当前真正写死的是：

- 地点名 -> 经纬度

如果地点不在本地坐标表里，就会导致：

- 路线无法重算
- 海拔无法重算
- 如果处理不好，还会保留旧的脏数据

短期修法：

1. 补常见地点坐标
2. 找不到地点时清空衍生字段，不保留旧值

长期修法：

1. 接高德 geocoding
2. fallback 本地表
3. 增加缓存

### 10.11 不要把关键副作用放在“响应后异步执行”

这次真实踩到的坑：

- `day-update` 先返回 200
- 然后在函数尾部异步触发路线 / 海拔 / 天气重算

本地通常看起来正常，但在 Vercel 上不可靠。函数返回后，后续异步逻辑可能被平台截断。

真实后果：

- 用户保存地点成功
- 但路线 / 海拔 / 天气有时更新，有时不更新，或者在下一次操作时才更新

结论：

- 不要把关键副作用放在 response 已返回后的异步代码里
- 正确做法是：
  1. 要么在请求生命周期内同步完成
  2. 要么由前端显式发第二个请求，例如 `/api/day-recalculate`

### 10.12 当用户截图和当前源码不一致时，先验 bundle 再改 UI

这次还踩到一个隐蔽问题：

- 用户线上截图里看到 `test`、`已完成`、票单详情样式
- 但当前仓库源码和线上 bundle 检查结果并不包含这套首页结构

如果这时继续直接改 UI，很容易越改越偏。

标准排查顺序：

1. 确认当前 Git HEAD
2. 抓线上 HTML，看它引用了哪个 bundle
3. grep 线上 bundle 里是否存在截图中的关键文案
4. 再判断是：
   - 缓存问题
   - 历史页 / 导出页残留
   - 另一条渲染路径
   - 还是当前源码真的有问题

结论：

- 当“用户截图”和“当前源码”明显不一致时，先证伪渲染路径，再改界面
2. 只在必须点击的地方再开浏览器
3. 不要反复用截图做状态同步

这样更省 token，也更稳定

---

## 11. 线上验收清单

正式域名接通后，按这个顺序测：

1. 首页是否正常打开
2. Todo 勾选后刷新是否保留
3. AI 对话是否正常
4. AI 对话刷新后记录是否保留
5. 导出是否正常
6. 手机访问是否正常
7. 核心编辑写入后刷新是否保留

如果只剩功能层细节问题，例如：

- 编辑入口不明显
- 某些按钮点击反馈差
- 乐观更新延迟

这时说明**部署主链已经通了**，不要再把它当成架构阻塞。

## 12. 下次新项目建议模板

如果下次是新项目，建议从一开始就按这套结构做：

1. 前端：React / Vite 或 Next.js
2. 部署：Vercel
3. 域名：Cloudflare
4. 数据：Firestore
5. AI：服务端函数调用
6. 实时：Firestore listener
7. 不上 SQLite
8. 不上自建 WebSocket

这样会比“先写旧结构，再迁移”省很多时间。

---

## 13. 复用时的最短执行顺序

1. 建 Firebase 项目
2. 开 Firestore
3. 建 Web App
4. 生成 Service Account JSON
5. 写 `.env`
6. 跑 `npm run seed:firestore`
7. 本地 `npm run build && npm start`
8. 验证核心功能
9. 推 GitHub
10. Vercel 导入仓库
11. 导入 `.env.vercel`
12. 检查 `api/` 路由是否有冲突
13. 部署
14. 先看 deployment 是否 `Ready`
15. Cloudflare 加 `CNAME`
16. 打开正式域名验证
17. 逐项做线上验收
