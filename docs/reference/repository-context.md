---
title: 仓库上下文
scope: repository
status: active
---

# 仓库上下文

## 项目定位

LoopEvo 是一个开源的通用 Agent 平台：用户说明目标，Agent 自主调研、执行，并在需要重复、等待或恢复时沉淀为可复用的 Loop。自动信息流是第一个规划中的端到端场景，不是平台边界。

产品同时规划：

- **本地私有桌面端：** 业务数据不进入 LoopEvo 云端，设备直连用户选择的模型和数据 Provider；
- **可选云端：** 基于 Cloudflare 的全天候运行宿主；
- **共享内核：** 两端共享 Agent、Revision、Activation、Run、Artifact、Capability、Memory 和 Policy 语义。

## 当前真实状态

项目当前处于 **Pre-alpha / 设计阶段**。

仓库当前已经具备：

- 英文与中文 README；
- Apache License 2.0；
- 仓库级与 docs 级协作入口；
- 已采纳的产品定义、双运行宿主架构、外部参考和 UI 设计；
- 带阶段范围与退出条件的产品和工程路线图；
- 协作、验证、安全与数据治理基线。

仓库当前尚未具备：

- 可运行的 Web、桌面端、API 或 CLI；
- Agent / Workflow Schema、Pi Adapter、Policy 或持久执行；
- SQLite、PostgreSQL、R2、Cloudflare 部署或本地 Artifact Store；
- Codex、Claude、X、RSS、Web、Reddit 或通知生产 Adapter；
- 已验证的评估、自动进化、CI 或正式版本。

任何设计、Issue、README 或对话中出现的目标能力，都不能仅凭描述视为已经支持。

## 已采纳的长期方向

- 平台边界是通用 Agent，不是舆情监控或单一 Workflow 工具；
- 普通用户只需要 Agent、Loop、Activity、Result 和必要 Connection；
- 一次简单任务可以直接 Run，只有需要重复、等待或恢复时才形成 WorkflowRevision；
- AgentRevision 与 WorkflowRevision 是可移植定义；WorkflowRevision 固定 `agentRevisionId`，Activation 只将 WorkflowRevision 绑定到 local 或 cloud；
- 本地与云端默认独立，不自动同步 Secret、文件、Memory、Run 和 Artifact；
- 本地私有不等于断网或本地模型，模型请求可以从设备直达 Provider；
- 自动化以 PolicyGrant 为边界，Grant 内低风险动作自动执行，边界扩张才打断；
- 自动进化产生新 Revision，必须可评估、可观察、可停止和可回滚；
- SourceStrategy 属于信息流 case，不是通用 Kernel 对象；
- Pi 是唯一原生 Agent Loop，Codex / Claude 等完整 Agent 通过 Adapter 委派；
- 云端优先 Cloudflare，但共享 Kernel 不依赖 Cloudflare 专有类型；
- 非必要不增加模块、包、服务、工作流引擎或用户概念。

## 暂定技术基线

以下选择是已采纳目标或待 Spike 选择，不代表仓库已有对应代码：

| 关注点 | 方向 | 状态 |
| --- | --- | --- |
| 共享语言 | TypeScript | 已采纳，版本待脚手架锁定 |
| UI | React + Vite | 已采纳 |
| 本地宿主 | Electron + Node + SQLite WAL + 本地文件 + OS Keychain | 已采纳，待实现验证 |
| 云端宿主 | Workers + Static Assets + Workflows + PostgreSQL / Hyperdrive + R2 | 已采纳，待实现验证 |
| 原生 Agent | Pi Adapter | 待 Local 与 Workers Spike |
| 外部 Agent | Codex App Server、Claude API / Companion | 待 Spike；合规边界已记录 |
| 能力协议 | Native Capability、MCP、Skills | 已采纳边界，SDK 暂缓 |
| 观测 | Domain Event，后续映射 OpenTelemetry | 已采纳边界 |

Cloudflare Agents SDK、Durable Objects、Queues、AI Gateway、Vectorize、Browser Run 和 Sandbox 按真实触发条件加入，不作为首版默认依赖。本地模型只记录为远期方向。

## 近期需要固化的实现事实

- Node、pnpm、Electron、React、Pi 和测试工具精确版本；
- Kernel Schema 与向前迁移规则；
- Electron 进程、preload API 和本地目录布局；
- SQLite Schema、Run Ledger、Lease、Checkpoint 和 effect 幂等；
- Pi Adapter 与 Workers 兼容结论；
- Codex App Server 版本、Schema、认证和降级路径；
- Cloudflare Worker / Workflows / Hyperdrive / R2 的部署与本地开发入口；
- X 数据 Provider 的授权、字段、费用、删除和增量语义；
- 模型 Provider、身份方案、数值预算和自动进化阈值。

这些内容只有在代码、测试或真实环境验证后才能更新为“已支持”。

## 关键路径

- 仓库协作入口：`CLAUDE.md`
- 知识库治理：`docs/CLAUDE.md`
- 产品定义与核心模型：`docs/design/core/product-and-architecture.md`
- 系统架构与技术基线：`docs/design/core/system-architecture.md`
- 参考项目与差异化：`docs/design/core/reference-landscape.md`
- UI 设计体系：`docs/design/core/ui-design-system.md`
- 当前路线图：`docs/plans/roadmap.md`
- Foundation 实施计划：`docs/plans/foundation-implementation-plan.md`
- 协作与交付：`docs/reference/collaboration-and-delivery.md`
- 测试与验证：`docs/reference/testing-and-validation.md`
- 安全与数据治理：`docs/reference/security-and-data-governance.md`

## 维护要求

- 技术栈、目录和运行入口真正落地后，在同一交付更新本文件；
- 已实现能力必须同时具备代码位置和验证证据；
- 临时分支、个人路径、聊天过程和一次性命令不写入本文件；
- 外部平台能力、价格、配额和条款具有时效性，使用前必须重新验证；
- 设计事实与实现冲突时，先区分“当前实现”和“已采纳目标”，不能静默覆盖。
