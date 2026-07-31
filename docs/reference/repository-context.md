---
title: 仓库上下文
scope: repository
status: active
---

# 仓库上下文

## 项目定位

LoopEvo 是一个开源的目标到工作流平台，目标是把自然语言意图转化为可治理、可复用、可验证并可安全进化的持久工作流。

平台计划通过调研、规划、执行、评估、进化和治理形成闭环，并以 Skills、MCP Server、API、浏览器操作器和 Coding Agent 等能力作为可组合模块。主题情报与信息获取是第一个规划中的端到端场景，但不是平台的最终边界。

## 当前真实状态

项目当前处于 **Pre-alpha / 设计阶段**。

仓库当前已经具备：

- 英文开源首页 `README.md`；
- 完整中文首页 `README.zh-CN.md`；
- Apache License 2.0；
- 仓库级与 docs 级协作入口；
- 已采纳的产品定义、架构边界、Pi / Temporal 候选技术调研、参考项目和 UI 设计；
- 带阶段范围与退出条件的产品和工程路线图；
- 协作、验证、安全和数据治理基线。

仓库当前尚未具备：

- 可运行的应用或命令行工具；
- 工作流 Schema、编译器、调度器或执行引擎；
- 数据库、队列、对象存储或可观测性实现；
- X、Reddit、Facebook、Instagram、Bluesky、RSS 等生产连接器；
- 用户界面、部署配置、CI 或正式版本；
- 已验证的自动评估、灰度发布或工作流进化实现。

任何文档、Issue 或对话中出现的目标能力，都不能仅凭描述视为已经支持。

## 已采纳的长期方向

- 用户从自然语言目标开始，而不是手工搭建空白流程图。
- 有效过程沉淀为持久、可版本化、可调度、可观测的工作流资产。
- 工作流运行保留来源、状态、证据、成本和评估结果。
- 能力通过显式契约接入，不让具体供应商或工具污染核心模型。
- “自进化”是基于证据提出受治理的版本变更，不是无约束自修改。
- 高风险变更需要测试、策略、审批、灰度和回滚。
- 数据访问必须满足授权、平台条款、隐私和地区法律要求。
- 无论选择哪种 Agent Runtime，它都只承担推理与工具循环，不承担产品工作流、权限和持久状态。
- Pi 与 Temporal 是需要由 Phase 0 Spike 验证的暂定技术选择；PostgreSQL 是计划中的产品事实源，S3 兼容对象存储用于大体积原始证据。
- 系统采用 TypeScript 模块化单体与独立 Worker 起步，不提前引入双 Agent Runtime、Kafka、Kubernetes 或完整流程画布。

详细内容见 `docs/design/core/product-and-architecture.md` 与 `docs/design/core/system-architecture.md`。

## 暂定技术基线

以下选择属于暂定目标技术基线，不代表仓库已有对应代码；Pi 与 Temporal 通过路线图的 Spike 门槛后才升级为实现事实：

- TypeScript、React / Next.js Web 与独立 Worker；
- 通过 Adapter 隔离的 `@earendil-works/pi-ai` 和 `@earendil-works/pi-agent-core`；
- Temporal TypeScript SDK 解释固定的 WorkflowVersion；
- PostgreSQL、Postgres Outbox 和 S3 兼容对象存储；
- Playwright、MCP TypeScript SDK、Skills 和 Native Connector；
- OpenTelemetry 作为可观测协议。

## 仍需 Foundation 固化的决策

- 依赖的精确版本、workspace 工具、包名和代码目录；
- WorkflowSpec JSON Schema、状态机、数据库表和迁移策略；
- Identity、租户隔离、Policy、Secret Provider 和生产部署；
- X 数据 Provider、连接器字段、许可、费用和删除模型；
- UI 组件实现、流式传输和可观测分析后端；
- 评估数据集、模型供应商、数值 SLO 和自动审批阈值。

这些决策必须经过 Spike、设计、实施和验证后才能更新为实现事实。

## 关键路径

- 仓库协作入口：`CLAUDE.md`
- 知识库治理：`docs/CLAUDE.md`
- 产品定义与核心模型：`docs/design/core/product-and-architecture.md`
- 系统架构与技术基线：`docs/design/core/system-architecture.md`
- 参考项目与差异化：`docs/design/core/reference-landscape.md`
- UI 设计体系：`docs/design/core/ui-design-system.md`
- 当前路线图：`docs/plans/roadmap.md`
- 协作与交付：`docs/reference/collaboration-and-delivery.md`
- 测试与验证：`docs/reference/testing-and-validation.md`
- 安全与数据治理：`docs/reference/security-and-data-governance.md`
- 其他执行中设计与计划：`docs/plans/*`

## 维护要求

- 技术栈、目录或运行入口真正落地后，在同一交付中更新本文件。
- 已实现能力必须同时具备代码位置和验证证据；只完成设计不能更新为“已支持”。
- 临时分支、当前任务、个人环境路径和一次性命令不写入本文件。
- 外部平台的价格、配额、API 能力和条款具有时效性，引用时必须标注来源并在使用前重新验证。
