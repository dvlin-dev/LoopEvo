---
title: 参考项目与差异化
scope: repository
status: active
---

# 参考项目与差异化

## 文档定位

本文记录 LoopEvo 产品和架构决策的外部依据。事实仅取自截至 2026-08-01 的官方文档和官方 GitHub；“借鉴、避免、决策”属于 LoopEvo 的研究判断。外部能力、版本、部署形态与许可证在采用前必须重新核验。

## 总体结论

在本文核验的项目中，尚未发现一个项目同时把以下全部能力作为开源、可自托管的一等产品模型。这个判断基于有限样本，不代表穷尽市场：

```text
自然语言目标
→ 自动调研信源与能力
→ 版本化 Source Strategy 与 Workflow
→ 可恢复的长期执行
→ 原始 Evidence 与派生链
→ 质量和成本评估
→ Replay、审批、Canary 与 Rollback
```

n8n、Dify、Gumloop 等已经能够通过自然语言生成或改进流程，Gumloop 也已提供评估与受审查的反思。因此 LoopEvo 的差异不能建立在“AI 会搭流程”或“流程会改进”上，而必须落到统一的 `SourceStrategyVersion + WorkflowVersion + Run + Evidence + Evaluation + EvolutionProposal` 模型、运行版本固定和可移植的治理发布链。

## 许可证与部署形态

| 项目 | 许可 / 形态 | 对 LoopEvo 的含义 |
| --- | --- | --- |
| [OpenClaw](https://github.com/openclaw/openclaw/blob/main/LICENSE) | MIT，自托管开源 | 可借鉴常驻 Gateway、渠道和调度，不作为产品模型 |
| [Hermes Agent](https://github.com/NousResearch/hermes-agent/blob/main/LICENSE) | MIT，自托管开源 | 可借鉴学习、Memory 和 Observer，不复制中央巨型 Agent |
| [OpenAI Codex](https://github.com/openai/codex/blob/main/LICENSE) | Apache-2.0，开源 Coding Agent | 可选 Coding Worker 与规则 / Skill 参照 |
| [Claude Code](https://github.com/anthropics/claude-code/blob/main/LICENSE.md) | Anthropic Commercial Terms | 可选商业 Coding Worker，不能成为开源核心依赖 |
| [Dust](https://github.com/dust-tt/dust/blob/main/LICENSE) | MIT 仓库 + 商业托管服务 | 可借鉴团队 Agent Builder，部署与服务范围需分别核验 |
| [Gumloop](https://www.gumloop.com/) | 公共资料展示的托管商业产品；本文未找到完整核心自托管发行 | 最接近的交互参照，不依赖其托管状态 |
| [n8n](https://github.com/n8n-io/n8n/blob/master/LICENSE.md) | Sustainable Use License，`.ee` 另受 Enterprise License | 不是 OSI 开源底座；仅桥接或借鉴契约 |
| [Dify](https://github.com/langgenius/dify/blob/main/LICENSE) | Dify Open Source License，基于 Apache-2.0 且有附加条件 | 不直接作为核心，采用前逐条核验限制 |
| [Airbyte](https://github.com/airbytehq/airbyte/blob/master/LICENSE) | 组件存在 MIT / ELv2 差异 | 只按组件核验复用，不部署整套平台 |
| [Pi](https://github.com/earendil-works/pi/blob/main/LICENSE) / [Temporal](https://github.com/temporalio/temporal/blob/main/LICENSE) | MIT；均可自托管 | 暂定基础依赖，必须先通过 Phase 0 Spike |

## 用户指定的四类 Agent

### OpenClaw

[OpenClaw](https://github.com/openclaw/openclaw) 是本地优先的常驻个人 AI 助手。其 [Gateway 架构](https://docs.openclaw.ai/concepts/architecture)统一接入渠道、Session 和 Agent；官方还提供 [Skills](https://docs.openclaw.ai/tools/skills)、[Plugins](https://docs.openclaw.ai/tools/plugin)、[Subagents](https://docs.openclaw.ai/tools/subagents) 与 [Cron Jobs](https://docs.openclaw.ai/automation/cron-jobs)。

借鉴：Gateway / Channel / Worker 分离，持久 Session，状态变化触发，自适应下次检查，幂等事件和 Skill 信任边界。

不照搬：不以几十个消息渠道、个人助理、语音和桌面控制为 MVP；不以 Agent Session / Memory 代替 Workflow、Run 和 Evidence；不建立同时负责推理、调度、存储与权限的巨型 Gateway。

### Hermes Agent

[Hermes Agent](https://github.com/NousResearch/hermes-agent) 是 NousResearch 的通用自改进 Agent，覆盖多模型、Session 搜索、Skills、MCP、Cron、Delegation、Sandbox 和 Observer Hooks。官方资料包括[架构](https://github.com/NousResearch/hermes-agent/blob/main/website/docs/developer-guide/architecture.md)、[Memory](https://github.com/NousResearch/hermes-agent/blob/main/website/docs/user-guide/features/memory.md)、[Skills](https://github.com/NousResearch/hermes-agent/blob/main/website/docs/user-guide/features/skills.md)、[Delegation](https://github.com/NousResearch/hermes-agent/blob/main/website/docs/user-guide/features/delegation.md)与[可观测 Hooks](https://github.com/NousResearch/hermes-agent/blob/main/docs/observability/README.md)。

借鉴：需要判断的 Agent 任务与机械脚本分开；Session 检索与压缩谱系；无人值守任务固定模型和能力；Observer 只记录不改行为；学习变更展示 Diff。

不照搬：不让中央 Agent 同时掌握工具、会话、调度、记忆和存储；后台复盘不能直接改生产 Skill、Prompt 或代码；个人 Agent 人格不是 LoopEvo 的核心资产。

### OpenAI Codex

[Codex](https://github.com/openai/codex) 面向软件工程，提供 CLI、IDE、Desktop、Cloud、[AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md)、[Skills](https://learn.chatgpt.com/docs/build-skills)、[MCP](https://learn.chatgpt.com/docs/extend/mcp)、[Subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents)、[Automations](https://learn.chatgpt.com/docs/automations)、[App Server](https://learn.chatgpt.com/docs/app-server)以及独立的[审批与沙箱](https://learn.chatgpt.com/docs/agent-approvals-security)边界。

借鉴：规则、Skill、Memory 和任务上下文分层；Plan → Execute → Verify；沙箱与审批正交；typed event protocol 解耦 UI；默认单任务；写操作使用隔离工作区。

不照搬：Codex 的核心对象是代码仓库和开发任务，不是长期业务 Workflow。它只能作为可替换 Coding Worker，仓库规则不能代替运行时 Policy、Credential 和数据授权。

### Claude Code

[Claude Code](https://code.claude.com/docs/en/overview) 提供 [Agent SDK Loop](https://code.claude.com/docs/en/agent-sdk/agent-loop)、[Dynamic Workflows](https://code.claude.com/docs/en/workflows)、[Routines](https://code.claude.com/docs/en/routines)、[Skills](https://code.claude.com/docs/en/skills)、[MCP](https://code.claude.com/docs/en/mcp)、[Subagents](https://code.claude.com/docs/en/sub-agents)、[Permissions](https://code.claude.com/docs/en/permissions)和[Sandboxing](https://code.claude.com/docs/en/sandboxing)。

借鉴：将对话流程升级为可查看、保存和重跑的程序；支持 pipeline、fan-out、交叉验证、暂停、规模边界、token 可见性与成本预警；中间结果留在工作流状态；Skill 演进使用测试集和前后对比。

边界：Dynamic Workflow 的恢复依赖同一 Session，Routines 在调研时仍为 research preview；它们不是跨进程业务运行保证。可重跑脚本也不足以表达 Source Strategy、checkpoint、原始 Evidence、不可变 Release 和完整发布治理。

## Agent 自动化与工作流产品

| 产品 | 已验证能力 | LoopEvo 应借鉴 | 仍然保持的差异 |
| --- | --- | --- | --- |
| [Gumloop Agents](https://docs.gumloop.com/core-concepts/agents) | 对话更新 Instructions、Skills、Triggers；Tool 权限；[AI Trigger Creation](https://docs.gumloop.com/core-concepts/ai_trigger_creation) | 渐进配置、按需发现工具、逐 Tool allow / ask / deny | `SourceStrategyVersion`、原始 Evidence / Derivation、不可变 Workflow Release、运行版本固定、可移植自托管契约 |
| [Gumloop Evaluations](https://docs.gumloop.com/core-concepts/evaluations) / [Reflections](https://docs.gumloop.com/core-concepts/reflections) | 测试用例比较变化；定期分析历史并引用证据提出改进；进入 Review Queue，也可对满足条件的低风险建议自动应用 | 反馈聚类、证据化建议、Review Queue | LoopEvo 要求统一 Replay → Canary → Rollback 发布链，不把“会反思”当独有卖点 |
| [Dust](https://docs.dust.tt/docs/user-documentation/agents/create-your-first-agent) | Agent Builder、工具 / 数据建议、实时预览、团队知识权限 | AI 配置助手、最小知识范围、试运行与团队分发 | 信源理由、checkpoint、覆盖、证据和成本是一等对象 |
| [n8n AI Workflow Builder](https://docs.n8n.io/build/ways-of-building-workflows/ai-workflow-builder/) | 可按自然语言创建和迭代 Workflow；AI Assistant 可编辑、测试和排错，并有[评估](https://docs.n8n.io/build/integrate-ai/test-and-improve-ai-workflows/understand-why-to-test/) | Connector 生命周期、节点 I/O、凭据、逐步调试和变更历史 | 不以画布为入口；不依赖其许可；把 Source Strategy、Evidence 和固定版本 Run 统一建模 |
| [Dify New Agent](https://docs.dify.ai/en/self-host/use-dify/build/new-agent/build) | 可通过描述创建 Agent，支持 Workflow / Trigger / Human Input，并有[版本控制](https://docs.dify.ai/en/self-host/use-dify/build/version-control) | Model Provider、Workflow 预览、RAG、Plugin 和运行日志 | 不作为底座；LoopEvo 聚焦长期来源、checkpoint、证据和受治理演进 |

Gumloop 是产品交互上最接近的商业参照。LoopEvo 不以“Agent 会自动完善”为差异，而以开放可移植的事实模型、来源治理和可恢复发布链为差异。

## Agent 与基础设施项目

| 项目 | 可复用或借鉴 | 决策 |
| --- | --- | --- |
| [Pi](https://github.com/earendil-works/pi) | 模型适配、Agent Loop、Tool Calling、事件与上下文 | 暂定 Agent 内核；Phase 0 证明 Tool 暂停 / 恢复和 Adapter 隔离后确认 |
| [Temporal](https://docs.temporal.io/temporal) | Event History、Durable Timer、Schedule、Signal / Update 和故障恢复 | 首选候选持久执行层；Phase 0 通过 Replay、Versioning 与自托管门槛后确认 |
| [LangGraph](https://github.com/langchain-ai/langgraph) | 状态图、checkpoint、interrupt、确定性与 Agent 节点混合 | 学习契约；若 Pi 验证通过，不增加第二套 Agent Runtime |
| [Airbyte Connector Builder](https://docs.airbyte.com/platform/connector-development/connector-builder-ui/overview) | Discover、认证、分页、增量 checkpoint、partition 和错误模型 | 借鉴 Connector CDK；许可逐组件核验，不部署整套平台 |
| [Letta Code](https://github.com/letta-ai/letta-code) | 长期 Memory、Skill、Schedule 和 Git 式变更历史 | 借鉴分层与历史；演进 Workflow Artifact，而非无边界人格 |
| [OpenHands](https://github.com/OpenHands/OpenHands) | 可替换 Coding Agent、Sandbox 和执行后端 | 通过 Coding Adapter 接入，不放进 Agent 内核 |
| [OpenTelemetry](https://opentelemetry.io/docs/what-is-opentelemetry/) | 开放 Trace / Metric / Log 语义 | 采用协议；观测后端可替换 |

## 主题情报与信息获取直接参照

这一层决定数据授权、覆盖、增量和成本，不能只研究 Agent 框架。

| 类别 / 项目 | 已验证能力 | LoopEvo 的判断 |
| --- | --- | --- |
| 社交监听：[Brandwatch Consumer Research](https://www.brandwatch.com/products/consumer-research/)、[Meltwater Social Listening](https://www.meltwater.com/en/capabilities/social-listening) | 跨渠道品牌 / 竞品监听、实时信号、AI 分析、报告和企业协作 | 成熟的结果与运营 UX 参照；属于商业数据产品，不替代开放 Workflow 与 Provider 可移植性 |
| 研究情报：[Feedly AI](https://feedly.com/ai) | 用 AI Model 过滤噪声并加速持续研究 | 借鉴 Topic / Model / Source 组合与研究体验；不把它误当通用工作流运行时 |
| 数据能力：[Bright Data Social Media Scraper](https://brightdata.com/products/web-scraper/social-media-scrape) | 通过 API 提供社交数据采集、解析、批量请求和按交付计费能力 | 作为付费 Connector 候选；数据来源、条款、删除、字段、增量和费用仍需逐 Provider 验证 |
| Web 能力：[Firecrawl](https://github.com/firecrawl/firecrawl) | 开源 + Cloud 的 search、scrape、crawl、map 和结构化输出 | 可选 Web Connector 参照；不是社交数据授权、调度或 Evidence 治理层 |
| Trigger / Action：[IFTTT Applets](https://ifttt.com/docs/applets) | Trigger、Filter Code 和 Action 组成 Applet，平台或连接服务承担检查 / 推送 | 借鉴统一触发与动作契约；用户仍需选择服务，且缺少完整研究、证据与版本进化模型 |
| 开源 Feed：[RSSHub](https://github.com/DIYgod/RSSHub) | 将大量站点内容转换为 RSS，可自托管 | 适合作为低成本 Source Adapter；每条 Route 仍需核验稳定性、许可和平台条款 |
| 开源事件自动化：[Huginn](https://github.com/huginn/huginn) | 自托管 Agent 监控事件并执行动作 | 借鉴 Event、状态与 Agent 组合；编排主要由用户完成，不提供 LoopEvo 的目标调研与治理模型 |
| 开源页面监控：[changedetection.io](https://github.com/dgtlmoon/changedetection.io) | 页面变化检测、条件、计划、浏览器步骤、通知与 AI 摘要 | 可借鉴变化 Diff 和通知；覆盖网站变化，不覆盖社区讨论和跨来源 Source Strategy |

这些项目说明：LoopEvo 不应自建社交数据网络，也不能承诺全平台覆盖。核心应允许用户在官方 API、Bright Data 等商业 Provider、RSSHub 等开源 Source 和定向 Web Connector 之间选择，并统一处理授权、checkpoint、证据、成本和替换。

## Build / Reuse / Buy

### 必须自研

- Intent / Research Planner；
- WorkflowSpec、Compiler、Version / Release Registry；
- `SourceStrategyVersion` 与 Connector Contract；
- Evidence / Provenance 数据模型；
- Evaluation、EvolutionProposal、Replay、Canary 与 Rollback；
- Chat 控制面与 Workflow / Run / Evidence 事实面。

### 候选复用

- Pi：Agent Loop 与模型工具层，Phase 0 验证后确认；
- Temporal：持久执行与调度，Phase 0 验证后确认；
- PostgreSQL / S3：结构化事实和大对象；
- Playwright、MCP SDK、Agent Skills、Webhook、RSS；
- OpenTelemetry：观测协议；
- Codex、Claude Code、OpenHands：可插拔 Coding Worker；
- RSSHub、Firecrawl 等：按 Connector 边界选择性接入。

### 购买或 BYOK

- LLM、搜索 API、云浏览器；
- X、Reddit 等平台的官方 API 或合同明确的数据 Provider；
- Brandwatch / Meltwater 等成品情报服务的数据或导出集成，仅在许可和价值成立时使用；
- 企业 Identity、Secret、通知和可观测托管服务。

供应商只提供能力，不拥有 LoopEvo 的 WorkflowVersion、Run、Evidence 和 Evaluation 事实模型。

## 最终差异化

```text
OpenClaw / Hermes：让 Agent 持续存在和学习
Codex / Claude Code：让 Agent 完成可验证的软件任务与编码流程
n8n / Dify：用 AI 和画布生成、编辑并运行自动化或 AI 应用
Gumloop：对话式配置、评估并反思 Agent 自动化
LoopEvo：把持续目标、信源策略、原始证据、固定版本运行和治理发布链统一成开放工作流产品
```

LoopEvo 必须坚持：

1. **目标驱动：** 先理解持续目标，再选择来源和能力；
2. **来源驱动：** 选择理由、授权、覆盖、checkpoint 和替代方案独立版本化；
3. **证据驱动：** 原始证据、派生结论和变更依据可以追溯；
4. **版本驱动：** Workflow、Capability、Model 和 Policy 在每次 Run 中固定；
5. **治理驱动：** 进化是 Replay、审批、Canary 和 Rollback，不是在线自改。

## 明确不做

- 不把消息渠道、模型或 Connector 数量当核心竞争力；
- 不声称自然语言生成流程、评估或反思是 LoopEvo 独有；
- 不在 MVP 建设个人助理、语音、桌面控制、Agent Teams 或公共市场；
- 不复制 n8n / Dify 的完整画布与应用构建层；
- 不在运行中执行 Agent 新生成的未评审代码；
- 不将外部内容、Webhook 或模型输出视为可信指令；
- 不把“进程退出成功”当业务成功，Run 必须验证结果和证据。
