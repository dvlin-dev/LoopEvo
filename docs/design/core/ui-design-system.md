---
title: UI 设计体系
scope: repository
status: active
---

# UI 设计体系

## 文档定位

本文定义 LoopEvo 官网和 Web 应用的视觉、信息架构与交互基线。参考对象仅为 [Wegic 官网](https://wegic.ai/)及其项目工作区的视觉语言和交互层级，**与 Wegic 的网站生成功能无关**。

调研于 2026-08-01 在桌面和 390 × 844 移动视口实机完成，覆盖官网、创建页、项目列表、聊天工作区、项目设置和 Memory 设置。研究截图仅作内部证据，不进入仓库，也不复制 Wegic 的 Logo、吉祥物、插画、文案、业务流程或其他专有资产。

## 设计目标

LoopEvo 的界面应让用户始终回答四个问题：

1. 系统理解的目标是什么；
2. 当前准备做什么、正在做什么；
3. 结论来自哪里，覆盖和不确定性如何；
4. 哪些变更或动作正在等待我的决定。

设计气质是：**冷静、清晰、可信、轻量但不玩具化。** 对话是入口，状态、证据、版本和审批是主角。

## 从 Wegic 提取的原则

### 借鉴

- 近白画布、强黑色段落和大量留白形成清晰节奏；
- 首页只保留一个主意图输入和少量示例，不先展示复杂设置；
- 工作区以对话为中心，计划、任务状态和结果卡片直接嵌入时间线；
- 导航弱化，项目身份、当前状态和主动作保持可见；
- 设置采用左侧分类、右侧内容的大面板，项目级与账号级边界清楚；
- 移动端保持单列对话和底部输入，不缩小成不可用的桌面画布。

### 不复制

- 不使用 Wegic 的品牌色值、Logo、吉祥物、插画、卡片内容或营销文案；
- 不复制其建站业务的信息结构；
- 不用装饰性动画掩盖运行状态；
- 不把所有能力藏在对话中，Workflow、Run、Evidence 和 Approval 必须有稳定入口。

## 目标态产品信息架构

以下结构描述完整产品目标，不代表 Phase 0 / 1 全部实现。

```text
Public
├── Home
├── Docs
├── Examples
└── GitHub

App
├── Projects
├── Create Project
├── Project Workspace
│   ├── Conversation
│   ├── Workflow
│   ├── Runs
│   ├── Evidence
│   └── Evaluations
├── Approvals
└── Settings
    ├── Project
    │   ├── General
    │   ├── Sources
    │   ├── Capabilities
    │   ├── Schedules
    │   ├── Memory
    │   ├── Notifications
    │   └── Security
    └── Account
        ├── Profile
        ├── Model Providers
        ├── Secrets
        ├── Usage
        └── Appearance
```

阶段最小范围：

| 阶段 | 页面与入口 |
| --- | --- |
| Phase 0 | 单一 Project Workspace：Conversation、Run、Evidence、最小 Provider 设置 |
| Phase 1 | Projects、Create、Conversation、Workflow、Runs、Evidence、Sources、Notifications、Usage |
| Phase 3+ | 独立 Evaluations、全局 Approvals、Memory 管理和完整 Security Center |

首版不建立独立“Agent 商店”或“Workflow 画布”一级入口。尚未进入当前阶段的页面可以通过内联卡片承载必要动作，但不能出现空壳导航。

## 页面规范

### 官网

首屏结构：

1. 简洁导航：Logo、Docs、Examples、GitHub、Open app；
2. 一句话价值：“Describe the outcome. LoopEvo builds and evolves the workflow.”；
3. 大型 Intent Composer，可直接输入长期目标；
4. 3–4 个真实目标示例，如竞争情报、技术雷达、研究跟踪；
5. 从 Intent 到 Evidence 到 Evolution 的可视闭环；
6. 开源、自托管、治理和来源合规说明；
7. GitHub CTA。

页面使用浅色段落与一到两个黑色全宽段落交替，不用密集功能墙。动画只解释状态流转，并尊重 `prefers-reduced-motion`。

### Projects

- 顶部只保留标题、搜索、状态筛选和 New Project；
- 卡片展示名称、目标摘要、当前 Release、最近 Run、健康状态和待审批数量；
- 无结果状态直接提供 Intent Composer，不展示空白仪表盘；
- Credits / Usage 放在侧栏底部或设置内，不抢占主任务空间。

### Create Project

默认是一个居中的自然语言输入，而不是多步骤表单：

- 上方显示简短引导；
- 中间提供可编辑目标示例；
- 底部大型 Composer 支持附件和约束；
- 提交后进入同一个 Project Workspace，由 Agent 输出目标复述、调研计划和需要的授权。

只有会改变成本、权限、数据范围或交付方式的字段才逐步提问。

### Project Workspace

桌面布局：

```text
┌──────────────┬─────────────────────────────────┬──────────────────────┐
│ Project rail │ Conversation / Workflow / Runs  │ Context inspector    │
│              │                                 │ sources / evidence   │
│ projects     │ timeline                        │ version / cost       │
│ approvals    │ inline plan and result cards    │                      │
│ settings     │                                 │                      │
│              │ sticky composer                 │                      │
└──────────────┴─────────────────────────────────┴──────────────────────┘
```

- 左栏可收起，显示项目、全局审批和设置；
- 中栏最大宽度约 800 px，保证对话和证据可读；
- 右栏按选中对象展示 Workflow、Source、Evidence、Diff 或 Trace；无上下文时收起；
- 顶部固定 Project、Release、Run 状态和 Run / Pause 主动作；
- `Conversation / Workflow / Runs / Evidence / Evaluations` 是稳定视图，不依赖 Agent 消息中的临时链接；
- Composer 固定在底部，发送前展示当前作用域，例如“修改 Draft”或“分析 Run #42”。

### Settings

- 桌面使用居中大面板或完整页面，左侧分类、右侧内容；
- 项目设置和账号设置用分组标题分开；
- `Secrets` 只展示名称、作用域、最近使用和轮换状态，不回显值；
- `Memory` 展示作用域、类型、来源、更新时间、冲突和是否参与当前 Workflow；
- `Sources` 展示授权、checkpoint、覆盖、费用、健康和删除入口；
- 危险操作独立区域，并要求清楚说明影响和可恢复性。

## 对话与事实面的协作

Agent 消息不能只输出大段自然语言。涉及长期状态时，使用结构化卡片：

| 组件 | 用途 | 必须显示 |
| --- | --- | --- |
| `GoalSummaryCard` | 复述目标 | 范围、周期、输出、预算、未知项 |
| `ResearchPlanCard` | 首次调研计划 | 问题、步骤、预计耗时、外部访问 |
| `SourceProposalCard` | 推荐信源 | 理由、覆盖、授权、费用、风险、替代项 |
| `WorkflowDraftCard` | 展示候选流程 | 触发器、关键步骤、能力、策略、版本 |
| `ApprovalCard` | 高影响确认 | 动作、影响、权限、费用、有效期、拒绝结果 |
| `RunProgressCard` | 执行进度 | 当前步骤、成功/等待/重试、已用时间和成本 |
| `EvidenceCard` | 结论依据 | 来源、作者、时间、原文摘要、引用与可信状态 |
| `CoverageGapCard` | 明示缺口 | 未覆盖来源、授权失败、时间缺口和影响 |
| `EvolutionDiffCard` | 工作流改进 | 基线与候选 Diff、依据、评测、风险、回滚点 |

卡片中的状态必须同步到对应事实视图；对话内容不能成为唯一记录。

## 视觉 Token

以下是 LoopEvo 自己的初始 Token，目标是接近 Wegic 的克制感，而非逐值复制：

### 颜色

| Token | 值 | 用途 |
| --- | --- | --- |
| `--canvas` | `#F7F6F2` | 页面主背景 |
| `--surface` | `#FFFFFF` | 卡片与面板 |
| `--surface-subtle` | `#F0EFEB` | 选中、分组和次级面 |
| `--ink` | `#121212` | 主文字与深色面 |
| `--ink-muted` | `#6D6B66` | 次级信息 |
| `--line` | `#E4E2DC` | 细边框与分隔 |
| `--accent` | `#7557FF` | Agent、主动作和运行中状态 |
| `--verified` | `#1F9D68` | 已验证、成功和来源可用 |
| `--verified-ink` | `#125E3E` | 浅色背景上的成功状态文字 |
| `--warning` | `#B87418` | 覆盖缺口、预算和等待 |
| `--warning-ink` | `#70400A` | 浅色背景上的警告文字 |
| `--danger` | `#D14343` | 失败、撤销和破坏性动作 |
| `--danger-ink` | `#8A2020` | 浅色背景上的危险状态文字 |

`--verified`、`--warning` 和 `--danger` 只用于图标、边框、大字号或非文本强调；普通文字使用对应 `*-ink` Token。颜色不能单独表达状态，始终结合图标、文字和 `aria-live`。

### 字体

- Display：`Instrument Sans`，用于首页标题和少量大数字；
- UI / Body：`Inter`，用于应用正文、表格和控制项；
- Mono：`JetBrains Mono`，用于 ID、版本、JSON、日志和 Diff；
- 字重以 400 / 500 / 600 为主，不使用大面积 700+。

建议字号：Display 64/70、H1 44/50、H2 32/38、H3 22/28、Body 16/24、Small 14/20、Meta 12/16。移动端 Display 降为 42/46，H1 降为 34/40。

### 间距、圆角与阴影

- 4 px 基础网格；常用间距 8 / 12 / 16 / 24 / 32 / 48 / 64；
- 控件圆角 10 px，卡片 16 px，大面板 20 px，胶囊使用 999 px；
- 默认 1 px `--line` 边框；阴影只用于浮层和 Composer；
- 正文行宽 65–80 字符；官网内容宽度 1200 px，应用中栏约 800 px。

## 状态语言

统一状态，不用含糊的“处理中”：

| 状态 | 含义 |
| --- | --- |
| Draft | 尚未发布，可修改 |
| Awaiting approval | 等待用户或策略审批 |
| Scheduled | 已安排但未开始 |
| Running | 正在执行且有心跳 |
| Waiting | 等待 Timer、Signal、Provider 或预算窗口 |
| Degraded | 仍有结果，但覆盖、质量或延迟下降 |
| Failed | 已停止，需要干预或自动重试已耗尽 |
| Cancelled | 被用户或策略终止 |
| Verified | 输出通过当前版本的验证规则 |

每个失败状态提供错误类别、影响、已经重试什么和下一步；不直接显示原始堆栈或 Secret。

## 交互规则

- 低风险、可逆、只读动作可直接执行并留痕；
- 凭据、费用、外部写入、数据删除和生产发布必须显示 Approval Card；
- 所有长任务立即返回 Run，持续流式更新，不锁死对话；
- 用户可 Pause、Cancel、Retry from step，并能区分“重跑”与“从检查点恢复”；
- Agent 修改 Draft 时实时展示结构化 Diff，不在后台静默覆盖；
- 证据引用支持在右栏预览并回到原链接；受限内容只展示许可范围内的信息；
- 删除操作先展示覆盖范围、保留规则和是否可恢复。

## 响应式

### 小于 768 px

- 单列 Conversation 为默认；
- 左栏使用抽屉，右侧 Inspector 使用底部 Sheet；
- 顶部只保留返回、项目名、状态和主动作；
- 视图切换使用可横向滚动的短标签；
- Composer 吸底，键盘弹出后仍保留发送与停止；
- 表格转换为分组列表，不横向压缩关键字段。

### 768–1199 px

- 左栏默认收起；
- Inspector 与主内容互斥打开；
- Workflow 图以只读缩略图 + 节点详情呈现。

### 1200 px 及以上

- 支持三栏；右栏宽 320–400 px；
- 用户关闭 Inspector 后，中栏保持居中而不是铺满。

## 无障碍与动效

- 文本和交互对比度满足 WCAG 2.2 AA；
- 所有动作可键盘完成，焦点顺序与视觉顺序一致；
- 触控目标至少 44 × 44 px；
- Dialog / Drawer / Sheet 正确管理焦点、Escape 和背景滚动；
- Run 更新使用节制的 `aria-live`，避免连续朗读日志；
- 不用无限骨架屏；超过预期时间时说明等待原因；
- `prefers-reduced-motion` 下关闭视差、自动滚动和非必要过渡；
- Diff、图表和状态均提供文本等价信息。

## 文案风格

- 先说结果或状态，再解释步骤；
- 区分“已验证”“模型判断”“覆盖缺口”和“需要授权”；
- 不使用“AI 已经完全理解”“全网覆盖”“绝对准确”等不可验证表述；
- 按钮描述动作和对象，例如“批准并发布 v3”，不用泛化的“确认”；
- 成本同时展示预计值、已用值和预算，不只展示 Credits。

## 验收清单

- 用户无需打开画布即可创建并理解第一个工作流；
- 每个 Agent 计划、外部动作、结论和变更都能进入稳定事实视图；
- 无论桌面或移动端，Run 状态、停止动作、证据和待审批项不丢失；
- 视觉上保持近白画布、大留白、强层级和克制强调色；
- 不包含 Wegic 的功能模型、品牌资产、专有内容或逐页复刻；
- 页面在键盘、读屏、高缩放和 reduced motion 下可完成核心路径。
