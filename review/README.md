# Review Standards Index

> 起草：质检员 @ 2026-05-05
> 更新：程序员先生 @ 2026-05-05
> 状态：草案
> 用途：作为 GitHub 项目提交与验收标准的统一首页，说明适用场景、使用顺序、输出要求与文件分工

## 一、使用方式

使用顺序：

1. 先读本文件，确定当前任务属于首轮验收、深度验收还是更新验收
2. 再按需要进入标准附件或执行附件
3. 输出时统一遵循本目录内标准文档定义的结论口径

本目录固定为三层结构：

1. **主入口 SOP**：说明适用场景、使用顺序、输出要求与附件导航
2. **标准附件**：定义统一标准、结论口径、提交规范、分层规则
3. **执行附件**：把标准落到清单与模板，不重复定义结论口径

## 二、分工原则

统一规则：
- **SOP 定义标准与结论口径**
- **checklist 定义执行动作、证据输出、引用来源**
- **模板定义提测输入质量**

禁止在 checklist 中重复定义一套独立判定标准。

## 三、术语约定

全目录统一使用以下术语：
- **提交**：代码提交、分支、PR 发起与合并前准备
- **提测**：工程师提交验收前的自检与材料准备
- **验收**：对项目当前交付进行正式判断
- **更新验收**：对已验收项目的增量改动进行判断
- **复核深度**：更新验收下的轻量 / 标准 / 局部深验 / 完整深验

补充：
- 更新验收分级的唯一判断口径，统一看 `tiered-review-standard.md` 第三节“变更类型 → 复核深度表”

## 四、建议阅读顺序与输出要求

### 1. 先看标准附件
1. `tiered-review-standard.md`
2. `acceptance-standard.md`
3. `submission-standard.md`

### 2. 再看执行附件
4. `reviewer-checklist.md`
5. `engineer-pre-handoff-self-check-template.md`

### 3. 按场景进入对应清单
6. `first-pass-review-checklist.md`
7. `deep-review-checklist.md`
8. `update-review-checklist.md`

### 4. 输出要求

- 验收结论统一引用 `acceptance-standard.md`
- 提交与 PR 规范统一引用 `submission-standard.md`
- 更新验收分级统一引用 `tiered-review-standard.md` 第三节
- 执行时优先使用对应 checklist / template，不在主入口重复抄写标准

## 五、文件分工说明

### 1. `tiered-review-standard.md`
**定位：** 分层总纲  
**作用：** 规定什么时候做首轮验收、深度验收、更新验收，以及更新后如何按改动级别决定复核深度。  
**适合谁看：** 负责人、reviewer、流程设计者  
**优先级：高**

### 2. `acceptance-standard.md`
**定位：** 总体验收 SOP  
**作用：** 定义统一验收标准、硬门槛、问题分级、结论阈值、证据要求、企业/个人差异。  
**适合谁看：** reviewer、工程师、外部 reviewer  
**优先级：高**

### 3. `submission-standard.md`
**定位：** GitHub 提交规范 SOP  
**作用：** 定义 branch、commit、PR、关联需求、验证证据、评审意见 closing rule。  
**适合谁看：** 工程师、reviewer、项目负责人  
**优先级：高**

### 4. `reviewer-checklist.md`
**定位：** 统一执行入口  
**作用：** 汇总reviewer 在任一场景下需要做的动作、收集的证据、引用的标准来源；不单独定义判定口径。  
**适合谁看：** reviewer、review 执行人  
**优先级：中高**

### 5. `engineer-pre-handoff-self-check-template.md`
**定位：** 工程师提测前自检模板  
**作用：** 要求工程师在提交验收前写清范围、验证证据、风险与限制，保证输入质量。  
**适合谁看：** 工程师、项目负责人  
**优先级：中高**

### 6. `first-pass-review-checklist.md`
**定位：** 首轮验收清单  
**作用：** 用于新项目、未成熟项目、批量项目的风险筛查与是否进入深验判断。  
**适合谁看：** reviewer、项目负责人  
**优先级：高**

### 7. `deep-review-checklist.md`
**定位：** 深度验收清单  
**作用：** 用于成熟、高风险、准备上线或交付项目的完整质量审查。  
**适合谁看：** 深度 reviewer、项目负责人  
**优先级：高**

### 8. `update-review-checklist.md`
**定位：** 更新验收清单  
**作用：** 用于已验收项目的增量复核，结合变更类型决定轻验、标准复核、局部深验或完整深验。  
**适合谁看：** reviewer、维护者、项目负责人  
**优先级：高**

## 六、完整文件清单

1. `review/tiered-review-standard.md`
2. `review/acceptance-standard.md`
3. `review/submission-standard.md`
4. `review/reviewer-checklist.md`
5. `review/engineer-pre-handoff-self-check-template.md`
6. `review/first-pass-review-checklist.md`
7. `review/deep-review-checklist.md`
8. `review/update-review-checklist.md`
