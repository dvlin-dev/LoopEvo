---
title: UI 设计体系
scope: repository
status: active
---

# UI 设计体系

## 文档定位

本文定义 LoopEvo 官网、Web 和桌面端的视觉、信息架构与交互基线。参考对象仅为 [Wegic 官网](https://wegic.ai/)及其工作区的视觉语言和交互层级，**与 Wegic 的建站功能无关**。

调研于 2026-08-01 覆盖 Wegic 官网、创建页、项目列表、聊天工作区、项目设置和 Memory 设置。研究只用于提炼原则，不复制其 Logo、吉祥物、插画、文案、业务流程或专有资产。

## 体验目标

LoopEvo 面向不熟悉 Agent、API 和流程编排的普通用户。界面始终帮助用户回答：

1. Agent 理解的目标是什么；
2. 它现在做什么，下一次什么时候做；
3. 结果是什么，来自哪里；
4. 哪些动作超出了我已授予的边界。

设计气质是：**冷静、轻量、可信、自动但不神秘。** 对话是入口，结果是主角；内部工作流、Run Trace 和 Evaluation 只在需要解释或排错时展开。

## 从 Wegic 提取的原则

### 借鉴

- 近白画布、强黑段落和大留白形成节奏；
- 首页只有一个主意图输入和少量示例，不先展示复杂设置；
- 工作区以对话为中心，计划、状态和结果以内联卡片出现；
- 导航弱化，当前 Agent、运行位置、状态和主动作保持可见；
- 设置使用左侧分类、右侧内容的大面板；
- 移动端保持单列对话和底部输入，不缩小桌面画布。

### 不复制

- 不使用 Wegic 的品牌色、Logo、吉祥物、插画、营销文案和业务结构；
- 不复制其建站功能或页面内容；
- 不用装饰动画掩盖运行、费用和权限状态；
- 不把所有事实藏在聊天里，Loop、Activity、Result 和 Connection 必须可再次找到。

## 用户概念

界面只把以下概念作为一等入口：

| 概念 | 用户文案 | 展示内容 |
| --- | --- | --- |
| Agent | “帮我长期完成目标的 AI” | 目标、状态、下一动作、Memory 摘要 |
| Loop | “会重复或持续运行的任务” | 做什么、触发方式、下次运行、运行位置 |
| Activity | “Agent 做过什么” | 当前进度、历史、费用、失败与恢复 |
| Result | “Agent 交付的东西” | 摘要、文件、来源、覆盖和可用动作 |
| Connection | “Agent 获得的模型或数据授权” | Provider、范围、费用、健康与撤销 |

WorkflowRevision、Run、Artifact、Evaluation、PolicyDecision 和 ChangeSet 只在高级详情或开发者模式出现。Approval 不建设全局一级页面；真正需要用户决定时，在当前对话和通知中出现一次清楚请求。

## 信息架构

```text
Public
├── Home
├── Docs
├── Examples
└── GitHub

App / Desktop
├── Home
│   ├── Agent list
│   └── Intent composer
├── Agent workspace
│   ├── Conversation
│   ├── Loops
│   ├── Activity
│   └── Results
├── Connections
└── Settings
    ├── Execution & privacy
    ├── Model providers
    ├── Notifications
    ├── Usage
    └── Appearance
```

首版不建立 Projects、Workflow、Runs、Evidence、Evaluations、Approvals、Memory 和 Security Center 等一级导航。团队空间与组织管理也不进入首版。

### 阶段范围

| 阶段 | 页面与入口 |
| --- | --- |
| Phase 0 | 单一 Agent Workspace、Loop / Activity / Result 卡片、Connection 与本地隐私设置 |
| Phase 1 | Agent Home、云端执行选择、Sources / Notifications / Usage、来源 Inspector |
| Phase 2+ | 多 Agent 管理、渐进式 Workflow / Evaluation / Version 详情 |

尚未实现的功能不放空壳入口。

## 页面规范

### 官网

首屏结构：

1. Logo、Docs、Examples、GitHub、Open app；
2. 一句话价值：“Describe the goal. Your Agent builds the loop.”；
3. 大型 Intent Composer；
4. 竞争情报、研究跟踪、学习和重复分析等真实示例；
5. 从 Goal → Agent → Loop → Result → Evolve 的可视闭环；
6. “Local private or cloud” 的清楚选择；
7. 开源、Provider 授权和 GitHub CTA。

页面用浅色段落与一到两个深色全宽段落交替，不堆叠功能墙。动画只解释状态流，并尊重 `prefers-reduced-motion`。

### Home

- 首次打开只显示一句引导、示例和 Intent Composer；
- 已有 Agent 时显示名称、目标摘要、健康、下一个 Loop 和最近 Result；
- 顶部可切换或查看 `Local private` / `Cloud`，但不要求用户先选技术部署；
- 空状态直接让用户描述目标，不展示空白 Dashboard；
- 不用 Credits、模型参数和工作流节点抢占首要任务。

### Agent Workspace

桌面布局：

```text
┌──────────────┬──────────────────────────────────┬────────────────────┐
│ Agent rail   │ Conversation + inline cards      │ Context inspector  │
│              │                                  │ Result / Source    │
│ Agents       │ Goal / Loop / Activity / Result  │ Version / Cost     │
│ Connections  │                                  │ Advanced details   │
│ Settings     │ Sticky composer                  │                    │
└──────────────┴──────────────────────────────────┴────────────────────┘
```

- 左栏可收起，只显示 Agent、Connections 和 Settings；
- 中栏最大宽度约 800 px，保证对话与结果可读；
- 右栏按选中对象显示来源、覆盖、版本、费用或 Trace；默认收起；
- 顶部固定 Agent 名称、运行位置、健康和 Stop / Run 主动作；
- `Loops / Activity / Results` 是稳定短标签，可从对话卡片到达；
- Composer 显示当前作用域，例如“告诉 Agent 新目标”或“改进这个 Loop”。

### Connections

- 分为 Model、Source、Delivery 三类，不按技术协议拆页面；
- 每项显示 Provider、授权范围、运行位置、最近使用、费用和健康；
- 本地 Provider 标记“设备直连”，云端 Provider 标记“由 LoopEvo Cloud 调用”；
- Secret 从不回显，只提供测试、缩小范围、轮换和撤销；
- Connection 失效说明受影响 Loop，不静默降级。

### Execution & Privacy

必须使用准确、可验证的文案：

| 选项 | 必须说明 |
| --- | --- |
| Local private | Agent、Run、Memory 和 Artifact 留在本机；模型和 Source 请求由设备直达 Provider；电脑或本地 Host 停止后 Loop 暂停 |
| LoopEvo Cloud | 数据和 Run 存在 LoopEvo 云；设备关闭后继续运行；费用与保留策略可查看 |

不使用“完全离线”“数据绝不离开电脑”等不准确表述。切换 Loop 的运行位置实际创建新 Activation；默认不搬迁 Secret、Run、Memory 和 Artifact。界面在迁移前展示重复副作用风险和停止旧 Activation 的状态。独立多用户 Self-hosted 宿主仍是未来方向，完成运行与运维设计前不显示为可选模式。

## 对话卡片

Agent 消息优先使用简洁结构化卡片，不输出大段系统术语：

| 卡片 | 用途 | 默认显示 |
| --- | --- | --- |
| `GoalCard` | 复述目标 | 结果、范围、周期、未知项 |
| `PlanCard` | 解释准备做什么 | 关键步骤、预计时间、需要的 Connection |
| `ConnectionRequestCard` | 一次授权 | Provider、数据、范围、费用、有效期、拒绝影响 |
| `LoopCard` | 沉淀自动任务 | 做什么、触发方式、下次运行、local / cloud |
| `ActivityCard` | 长任务进度 | 当前阶段、等待 / 重试、耗时、费用、Stop |
| `ResultCard` | 展示结果 | 输出、验证依据、完整性、不确定性、导出 |
| `BoundaryRequestCard` | 超出已有 Grant | 新增权限 / 预算 / 目标、原因、可选最小范围 |
| `ChangeCard` | 自动改进说明 | 改了什么、依据、效果、回滚入口 |
| `CoverageGapCard` | 明示缺口 | 未覆盖来源、时间范围、授权或费用影响 |

Workflow、Run、Evaluation 和版本 ID 放在“技术详情”，不能成为理解卡片的前提。

## 授权与自动化交互

- 低风险、可逆、只读且在 PolicyGrant 内的动作直接执行并留痕；
- 首次 Connection 或新的敏感范围使用一次清楚授权，后续同类 Tool 不重复询问；
- 权限、预算、数据用途、外部写入、删除和代码发布扩大时使用 Boundary Request；
- 用户可以随时查看、缩小、暂停或撤销 Grant；
- 自动进化通过简短 Change Card 告知，不要求逐次批准；
- Change Card 必须提供依据、观察结果和回滚入口；
- 长任务立即出现 Activity，允许 Stop，不锁死对话；
- 错误说明影响、已自动尝试什么和用户是否必须介入，不直接显示堆栈或 Secret。

## 视觉 Token

以下 Token 追求 Wegic 的克制感，不逐值复制其设计：

### 颜色

| Token | 值 | 用途 |
| --- | --- | --- |
| `--canvas` | `#F7F6F2` | 页面背景 |
| `--surface` | `#FFFFFF` | 卡片与面板 |
| `--surface-subtle` | `#F0EFEB` | 选中与次级面 |
| `--ink` | `#121212` | 主文字与深色面 |
| `--ink-muted` | `#6D6B66` | 次级信息 |
| `--line` | `#E4E2DC` | 边框与分隔 |
| `--accent` | `#7557FF` | Agent、主动作、运行中 |
| `--verified` | `#1F9D68` | 已验证与成功 |
| `--warning` | `#B87418` | 覆盖缺口与等待 |
| `--danger` | `#D14343` | 失败、撤销与破坏性动作 |

状态色只用于图标、边框和非文本强调；普通文字使用满足对比度的深色变体。颜色不能单独表达状态，必须结合图标和文字。

### 字体与尺度

- Display：`Instrument Sans`；
- UI / Body：`Inter`；
- Mono：`JetBrains Mono`，只用于 ID、版本、日志和 Diff；
- 字重以 400 / 500 / 600 为主；
- Display 64/70、H1 44/50、H2 32/38、H3 22/28、Body 16/24、Small 14/20；
- 移动端 Display 42/46，H1 34/40。

### 间距、圆角与阴影

- 4 px 基础网格，常用 8 / 12 / 16 / 24 / 32 / 48 / 64；
- 控件 10 px、卡片 16 px、大面板 20 px、胶囊 999 px；
- 默认 1 px `--line` 边框，阴影只用于浮层与 Composer；
- 正文行宽 65–80 字符，应用中栏约 800 px。

## 状态语言

| 用户状态 | 含义 |
| --- | --- |
| Ready | Agent 可以接受目标 |
| Running | 正在执行且有最近进度 |
| Waiting | 等待时间、Provider 或已知外部条件 |
| Needs you | 只有边界扩大或无法自动恢复时使用 |
| Paused | Loop 被用户或 Policy 暂停 |
| Degraded | 仍有结果，但覆盖、质量或延迟下降 |
| Failed | 自动恢复耗尽，需要查看影响 |
| Completed | 本次 Activity 已完成 |

Draft、Revision、Checkpoint 等内部状态只在高级详情出现。

## 响应式

### 小于 768 px

- 单列 Conversation 默认；
- 左栏使用 Drawer，Inspector 使用 Bottom Sheet；
- 顶部只保留返回、Agent、local / cloud 状态和 Stop；
- Loops / Activity / Results 使用可横向滚动短标签；
- Composer 吸底，键盘弹出后仍可发送与停止；
- 表格转换为分组列表。

### 768–1199 px

- 左栏默认收起；
- Inspector 与主内容互斥打开；
- 高级 Workflow 详情使用只读摘要，不显示拥挤画布。

### 1200 px 及以上

- 支持三栏，右栏宽 320–400 px；
- 关闭 Inspector 后中栏保持居中，不无意义铺满。

## 无障碍与动效

- 文本与交互满足 WCAG 2.2 AA；
- 所有核心路径可键盘和读屏完成；
- 触控目标至少 44 × 44 px；
- Dialog / Drawer / Sheet 正确管理焦点、Escape 和背景滚动；
- Activity 更新使用节制的 `aria-live`，不连续朗读日志；
- 超过预期等待时间时说明原因，不使用无限骨架屏；
- `prefers-reduced-motion` 下关闭视差、自动滚动和非必要过渡；
- Diff、图表和状态提供文本等价信息。

## 文案风格

- 先说结果或状态，再解释步骤；
- 区分“来源事实”“Agent 判断”“覆盖缺口”和“需要授权”；
- 不使用“完全理解”“全网覆盖”“绝对安全”“完全离线”等表述；
- 按钮描述动作和范围，例如“允许读取这个文件夹”，不用泛化“确认”；
- 成本同时显示预计、已用和预算，不只显示 Credits；
- 技术名词默认翻译为用户任务，详情中再提供原始术语。

## 验收清单

- 新用户不理解 Workflow、Run 和 Tool 也能创建第一个 Agent 与 Loop；
- Grant 内自动运行不产生重复审批，边界扩大能被准确拦截；
- local / cloud 数据路径、Provider 发送和设备在线限制表达准确；
- 每个长期 Loop、Activity、Result 和 Connection 可以再次找到；
- 结果可以展开验证依据、完整性、费用和技术 Trace；信息流 Result 再展示来源与覆盖，但默认不制造认知负担；
- 视觉保持近白画布、大留白、强层级和克制强调色；
- 不包含 Wegic 的功能模型、品牌资产、专有内容或逐页复刻；
- 桌面和移动端均能完成键盘、读屏、高缩放和 reduced-motion 核心路径。
