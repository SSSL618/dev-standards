# SOP：GitHub 项目提交规范

> 起草：程序员先生 @ 2026-05-05
> 状态：草案
> 触发条件：工程师准备提交代码、发起 PR、进入提测或处理评审意见

## 一、目标

统一 GitHub 项目的提交、提测、PR 与评审处理规范，降低“能合但不可审”“能跑但不可追溯”的问题。

## 二、Branch 命名

推荐格式：
- `feature/<scope>-<short-desc>`
- `fix/<scope>-<short-desc>`
- `refactor/<scope>-<short-desc>`
- `chore/<scope>-<short-desc>`
- `docs/<scope>-<short-desc>`

要求：
- 使用小写字母与短横线
- 名称能看出改动类型与作用域
- 禁止使用 `test`、`tmp`、`new`、`final` 这类无语义命名

## 三、Commit Message 规范

推荐格式：
- `feat: add xxx`
- `fix: correct xxx`
- `refactor: simplify xxx`
- `docs: update xxx`
- `test: cover xxx`
- `chore: adjust xxx`

要求：
- 单个 commit 只表达一个主要动作
- message 必须可读，不写“update”“fix bug”“misc”这类空泛描述
- 禁止把无关改动混入同一个 commit
- 若存在迁移、回滚点或风险改动，需在 commit 或 PR 描述中体现

## 四、PR 标题与描述

### PR 标题
要求：
- 直接说明本次改动意图
- 避免纯内部简称，外部 reviewer 能看懂
- 若有 Issue 编号，建议在标题中带上编号
- 若无 Issue 编号，必须使用可追溯替代标识：`PRD:<name>`、`TASK:<name>` 或 `REQ:<name>` 三选一

示例：
- `Fix login timeout #42`
- `PRD:diet-record add ingredient search`
- `TASK:review-standard align acceptance tables`

### PR 描述最小模板
必须包含：
1. **目标**：为什么改
2. **范围**：改了哪些模块/页面/接口
3. **关联依据**：PRD / Issue / 任务单链接
4. **风险**：可能影响什么
5. **验证证据**：引用 `acceptance-standard.md` 的证据最小格式表
6. **未覆盖范围**：哪些没测、哪些延期

## 五、关联需求强制要求

以下至少满足一种，且必须可追溯：
- PRD
- GitHub Issue
- 任务单
- 需求说明文档

禁止：
- 无任何需求依据直接发起“待验收 PR”
- PR 合入后仍无法追溯本次改动对应哪项需求

## 六、一个 PR 的范围约束

允许：
- 同一目标下的必要代码、测试、文档、配置联动改动

禁止：
- 混入与本次目标无关的重构
- 混入纯格式清理、顺手修 bug、无关依赖升级
- 把多个独立需求塞进同一个 PR

若确有无关问题顺手发现：
- 单独开 Issue，或单独开后续 PR

## 七、提测前必须附的验证证据

提测输入必须至少包含：
- 验证环境
- 验证步骤
- 实际结果
- 截图 / 日志 / CI 链接（企业项目必需；个人工具可降级但应尽量提供）
- 未覆盖范围

引用标准：`acceptance-standard.md` → **证据最小格式表**

## 八、评审意见 Closing Rule

### 处理要求
- 每条评审意见必须有明确状态：
  - 已修复
  - 不采纳（需说明理由）
  - 延后处理（需说明责任人、截止时间、临时缓解）

### Closing Rule
仅在以下条件满足时，评审意见才能视为关闭：
1. 已有代码或文档变更对应处理
2. 或明确说明不处理原因，且 reviewer/负责人接受
3. 或标记为延期，并补齐责任人、截止时间、临时缓解方案

禁止：
- 未说明即直接 resolve
- 用“后面再说”替代正式结论
- P1 及以上问题无跟踪信息直接关闭

## 九、评审处理 SLA（建议）

- 阻断问题（P0/P1）：同工作日内响应
- 普通问题（P2/P3）：1 个工作日内响应
- 若无法按时处理，需主动说明原因与预计时间

## 十、合并前检查

合并前至少确认：
- PR 描述完整
- 关联需求齐全
- 验证证据齐全
- 评审意见已清空或有明确未关闭理由
- 无明显无关改动
- 无临时文件、构建产物、个人配置误提交

## 十一、与验收标准的关系

- 本文档定义 **怎么提交、怎么提测、怎么关闭评审问题**
- 项目是否通过验收，统一以 `acceptance-standard.md` 的结论口径为准
