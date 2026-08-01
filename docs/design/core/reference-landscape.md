---
title: 参考项目与差异化
scope: repository
status: active
---

# 参考项目与差异化

## 文档定位

本文记录 LoopEvo 产品与架构决策的外部依据。事实基于截至 2026-08-01 的官方文档和官方 GitHub；“借鉴、避免、决策”属于 LoopEvo 的判断。外部能力、版本、许可和条款在采用前必须重新核验。

## 总体结论

LoopEvo 不应复刻一个更大的聊天 Agent、节点自动化平台或垂直舆情工具。应组合现有产品已经证明有效的部分：

```text
Gumloop 的自然语言创建与渐进配置
+ Hermes / OpenClaw 的本地常驻、Memory 与调度
+ Codex / Claude Code 的工具、沙箱与可验证执行
+ Cloudflare Workflows 的云端持久运行
+ 商业情报产品的来源、结果与运营体验
= 本地私有或云端运行、可沉淀并可验证进化的通用 Agent
```

差异不在“Agent 会搭流程”或“流程会反思”，而在：

- 普通用户只描述目标，不先编排；
- 一次任务可以直接执行，只有必要时才沉淀 Loop；
- AgentRevision 与 WorkflowRevision 可移植，WorkflowRevision 固定所用 AgentRevision，Activation 明确绑定 local 或 cloud；
- 本地私有模式不依赖 LoopEvo 云端；
- Capability、Connection、PolicyGrant 与 Skill 分离；
- 自动进化能在授权范围内运行、验证和回滚，而不是每次审批或无边界自改。

## 许可与产品形态

| 项目 | 许可 / 形态 | 对 LoopEvo 的意义 |
| --- | --- | --- |
| [OpenClaw](https://github.com/openclaw/openclaw/blob/main/LICENSE) | MIT，自托管开源 | 借鉴本地 Gateway、Channel、Session 和 Schedule |
| [Hermes Agent](https://github.com/NousResearch/hermes-agent/blob/main/LICENSE) | MIT，自托管开源 | 借鉴本地 Daemon、Memory、Skills、Cron 和 Observer |
| [OpenAI Codex](https://github.com/openai/codex/blob/main/LICENSE) | Apache-2.0，开源 Coding Agent | 官方本地外部 Agent 与 App Server 集成目标 |
| [Claude Code](https://github.com/anthropics/claude-code/blob/main/LICENSE.md) | Anthropic 商业条款 | API / Companion 候选；订阅后台集成有明确限制 |
| [Goose](https://github.com/block/goose/blob/main/LICENSE) | Apache-2.0，本地桌面 / CLI | 借鉴外部 Agent、ACP 与多 Provider 桌面体验 |
| [OpenHands](https://github.com/OpenHands/OpenHands/blob/main/LICENSE) | MIT，开源 Agent 平台 | 借鉴本地 / 远程 Backend 与 Agent Adapter |
| [Pi](https://github.com/earendil-works/pi/blob/main/LICENSE) | MIT，Agent 工具包 | LoopEvo 原生 Agent Loop 候选 |
| [Gumloop](https://www.gumloop.com/) | 托管商业产品 | 最接近的自然语言 Agent / 自动化交互参照 |
| [n8n](https://github.com/n8n-io/n8n/blob/master/LICENSE.md) | Sustainable Use License | 借鉴 Connector 与执行体验，不作为开源核心 |
| [Dify](https://github.com/langgenius/dify/blob/main/LICENSE) | 带附加条件的 Dify Open Source License | 借鉴 Provider、Workflow 与 Plugin，不作为核心依赖 |
| [Cloudflare](https://developers.cloudflare.com/) | 托管平台服务 | 云端首选宿主，不进入共享领域类型 |

许可证列表只描述当前调研结论，不替代采用时的逐组件审查。

## 通用 Agent 参照

### OpenClaw

[OpenClaw](https://github.com/openclaw/openclaw) 以常驻本地 Gateway 统一接入渠道、Session、Skills、Plugins、Subagents 和 Cron。

借鉴：

- 本地 Host 与 UI / Channel 分离；
- 长期 Session、状态变化触发和自适应下次检查；
- Skill 信任边界与无人值守任务的固定上下文。

不照搬：

- 不把几十个消息渠道、个人助理和语音作为首版；
- 不让一个 Gateway 同时成为权限、事实库、调度与业务模型；
- Session 不能替代 Revision、Activation、Run 和 Artifact。

### Hermes Agent

[Hermes Agent](https://github.com/NousResearch/hermes-agent) 覆盖多模型、Session、Memory、Skills、MCP、Cron、Delegation、Sandbox 和 Observer Hooks。

借鉴：

- 本地 Daemon 与长期任务；
- 需要判断的 Agent 工作与机械脚本分离；
- Memory 分层、上下文压缩和只观测不改行为的 Hooks；
- 学习变更展示 Diff，而不是静默改变人格。

不照搬：

- 不建设掌握所有 Tool、Session、Schedule、Memory 和存储的中央巨型 Agent；
- Agent 人格不是 LoopEvo 的唯一资产；
- 后台反思不能绕过 Policy、Evaluation 与回滚。

### Codex

[Codex](https://github.com/openai/codex) 已提供 CLI、Desktop、Cloud、AGENTS.md、Skills、MCP、Automations、审批与沙箱。[App Server](https://learn.chatgpt.com/docs/app-server) 明确面向在自有产品中进行深度集成，并覆盖认证、会话、审批与流式事件。

借鉴：

- Plan → Execute → Verify 与 typed event protocol；
- 规则、Skill、Memory、任务上下文分层；
- 沙箱与 Policy / Approval 正交；
- 本地进程、隔离工作区和可验证结果。

采用方式：

- 桌面端启动官方 `codex app-server`，首选稳定 stdio JSONL；
- 认证由 App Server 管理，LoopEvo 不复制其 Token；
- 探测用户安装的 Codex 并限制受支持版本范围；只有随 LoopEvo 分发时才锁定精确版本与 Schema；禁用 experimental API；
- `codex exec --json` 作为同一 Adapter 的降级路径；
- Codex 是外部 Agent Runtime / Coding Capability，不是 Pi 的模型 Provider。

### Claude Code

[Claude Code](https://code.claude.com/docs/en/overview) 提供 Agent Loop、Skills、MCP、Subagents、Permissions、Sandbox 和 [headless mode](https://code.claude.com/docs/en/headless)。这些能力证明 CLI 子进程技术可行。

但 [Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) 对第三方产品有明确认证边界：未经 Anthropic 事先批准，不允许第三方提供 Claude.ai 登录或代表用户路由 Free、Pro、Max 订阅额度。

因此：

- 正式后台 Adapter 使用用户 API Key、Bedrock、Vertex AI 或其他 Anthropic 支持方式；
- 无 API Key 场景通过 MCP、Skill、Plugin 或安装入口，让用户在原生 Claude Code 中主动执行和控制；Companion 不通过 `claude -p`、后台代理或定时任务消费订阅，也不能驱动无人值守 Loop；
- 只有取得 Anthropic 书面许可后，才增加 Claude Subscription Adapter；
- 开源项目能够技术性调用订阅，不构成 LoopEvo 的分发许可依据。

## 外部 Agent 协议参照

### ACP

[Agent Client Protocol](https://agentclientprotocol.com/overview/introduction) 使用本地子进程与 JSON-RPC over stdio 连接编辑器 / Client 和 Agent，适合隔离完整 Agent Runtime。[codex-acp](https://github.com/agentclientprotocol/codex-acp) 展示了 Codex App Server 到 ACP 的适配方式。

LoopEvo 借鉴其进程与事件边界，但首版不立即实现通用 ACP：

- Codex 先使用官方 App Server Adapter；
- 第二个真实外部 Agent 需要同一协议时再提取 ACP Adapter；
- ACP 只解决通信，不替代 LoopEvo Policy、Run、Artifact 和 Capability 边界。

### Goose 与 OpenHands

- [Goose ACP Providers](https://goose-docs.ai/docs/guides/acp-providers) 证明桌面 / CLI 可以把外部 Agent 作为子进程 Provider；借鉴配置、事件映射和进程生命周期，不把其订阅做法当作供应商许可。
- [OpenHands](https://github.com/OpenHands/OpenHands) 展示本地、远程和云端 Agent Backend 的切换；借鉴 Adapter 与 Sandbox，不复制复杂多 Backend 产品层。

## Agent 与自动化产品

| 产品 | 已验证能力 | 借鉴 | LoopEvo 保持的边界 |
| --- | --- | --- | --- |
| [Gumloop Agents](https://docs.gumloop.com/core-concepts/agents) | 对话更新 Instructions、Skills、Triggers 与 Tool 权限 | 自然语言创建、渐进配置、按需 Tool | 开放 Kernel、本地私有、portable Revision / Activation |
| [Gumloop Evaluations](https://docs.gumloop.com/core-concepts/evaluations) / [Reflections](https://docs.gumloop.com/core-concepts/reflections) | 测试变化、分析历史、提出并可自动应用部分建议 | Evidence-based Change、低风险自动应用 | PolicyGrant、宿主独立、版本固定和自动回滚 |
| [Dust](https://docs.dust.tt/docs/user-documentation/agents/create-your-first-agent) | Agent Builder、数据 / Tool 建议、实时预览 | 最小知识范围、试运行与团队分发 | 不把团队 SaaS 作为首版产品重心 |
| [n8n AI Workflow Builder](https://docs.n8n.io/build/ways-of-building-workflows/ai-workflow-builder/) | 自然语言创建、编辑、测试与排错 Workflow | Connector 生命周期、凭据、执行历史 | 不从画布开始，不让用户理解节点才能使用 |
| [Dify Agent](https://docs.dify.ai/en/self-host/use-dify/build/new-agent/build) | Agent、Workflow、Trigger、Human Input 与版本 | Provider、Plugin 和运行日志 | 不以 Dify 为底座，不暴露过多应用构建概念 |

Gumloop 是交互和自动改进上最接近的商业参照。LoopEvo 不声称“会反思”独有，而是把本地私有、可移植定义、持久运行与授权边界做成统一产品。

## Agent 与执行基础设施

| 项目 | 可复用或借鉴 | 决策 |
| --- | --- | --- |
| [Pi](https://github.com/earendil-works/pi) | 模型适配、Agent Loop、Tool、事件和上下文 | 唯一原生 Agent 候选；先做 Local 与 Workers Spike |
| [Cloudflare Workflows](https://developers.cloudflare.com/workflows/) | Durable Step、重试、等待、事件与恢复 | 唯一云端持久 Run 引擎 |
| [Cloudflare Agents SDK](https://developers.cloudflare.com/agents/) | Durable Object identity、实时会话与状态 | 首版不采用；避免与 Pi / Workflows 出现三个主脑 |
| [Cloudflare Hyperdrive](https://developers.cloudflare.com/hyperdrive/) | 连接池与边缘数据库访问 | 连接 PostgreSQL 云端事实源 |
| [Cloudflare R2](https://developers.cloudflare.com/r2/) | 大对象与原始 Artifact | 信息流 Alpha 按真实 Artifact 使用 |
| [Temporal](https://docs.temporal.io/temporal) | 自托管持久执行、Timer、Signal 和 Event History | 重要架构参照；Cloudflare 优先路线不首发两套引擎 |
| [LangGraph](https://github.com/langchain-ai/langgraph) | Checkpoint、Interrupt、图与 Agent 混合 | 学习契约，不增加第二套 Agent Runtime |
| [OpenTelemetry](https://opentelemetry.io/docs/what-is-opentelemetry/) | 开放 Trace / Metric / Log | 后续映射 Domain Event，不能成为事实源 |

首版云端不采用 Agents SDK，也不让 Durable Objects 承担业务状态，因为它们会与 Pi 的 Session / Memory 和 Workflows 的恢复 / Schedule 重叠。若 Pi 必须运行于 Cloudflare Container，可使用平台要求的 Durable Object Binding，但它只管理 Container 基础设施、不保存 LoopEvo 业务状态；其他 Durable Objects 等真实 Presence 或单写者协调需求出现后再引入，并只保存可重建连接态。

## 信息流 case 的直接参照

| 类别 / 项目 | 已验证能力 | LoopEvo 的判断 |
| --- | --- | --- |
| [Brandwatch](https://www.brandwatch.com/products/consumer-research/) / [Meltwater](https://www.meltwater.com/en/capabilities/social-listening) | 跨渠道监听、实时信号、AI 分析和报告 | 成熟结果 UX；价格与封闭数据链推动开源可替代需求 |
| [Feedly AI](https://feedly.com/ai) | Topic / Model / Source 组合与 AI 去噪 | 研究体验参照，不是通用 Agent Runtime |
| [Bright Data Social](https://brightdata.com/products/web-scraper/social-media-scrape) | 付费社交数据采集与结构化交付 | X 等 Source Provider 候选；逐平台核验许可、增量、删除和成本 |
| [IFTTT Applets](https://ifttt.com/docs/applets) | Trigger、Filter 与 Action | 借鉴事件 / 增量触发和统一 Action；不复刻用户手工编排 |
| [RSSHub](https://github.com/DIYgod/RSSHub) | 大量站点转 RSS，可自托管 | 低成本 Source Adapter；逐 Route 核验稳定与条款 |
| [Huginn](https://github.com/huginn/huginn) | 自托管 Event Agent 与 Action | 借鉴 Event、状态和通知；其编排仍主要由用户完成 |
| [changedetection.io](https://github.com/dgtlmoon/changedetection.io) | 页面 Diff、条件、Schedule、浏览器和通知 | 借鉴变化检测，不等于社区讨论数据 |
| [Firecrawl](https://github.com/firecrawl/firecrawl) | Search、Scrape、Crawl 与结构化输出 | 可选 Web Capability，不替代社交数据授权 |

LoopEvo 不自建社交数据网络。Kernel 只统一 Capability、Connection、Artifact、Checkpoint、成本和 Policy；具体 Source Plan 留在 info-flow case。

## Build / Reuse / Buy

### 必须自研

- Agent / Revision / Activation / Run / Artifact Kernel；
- 本地 Run Ledger 与双宿主一致性契约；
- PolicyGrant、Capability Executor 和低打扰授权体验；
- Evaluation、ChangeSet、自动启用与回滚；
- 对话、Loop、Activity、Result 和 Connection 用户体验。

### 候选复用

- Pi、Electron、SQLite、React 与 MCP SDK；
- Cloudflare Workers、Workflows、Hyperdrive 与 R2；
- Codex App Server、受 Anthropic Commercial Terms 约束且逐版本审查的 Claude Agent SDK；
- RSSHub、Firecrawl 等按 Capability / Connector 边界接入；
- OpenTelemetry 作为后续观测协议。

### 接入或购买

- 模型、搜索、云浏览器与通知；
- X、Reddit 等官方 API 或合同明确的数据 Provider；
- 企业身份、Secret 与观测托管服务。

## 最终差异化

```text
OpenClaw / Hermes：让本地 Agent 持续存在、拥有 Memory 与 Skills
Codex / Claude Code：让 Agent 完成可验证的软件与计算机任务
n8n / Dify：让用户或 AI 构建并运行自动化 / AI 应用
Gumloop：用对话配置、评估和反思 Agent 自动化
LoopEvo：让普通用户只说明目标，Agent 在本地私有或云端形成 Loop，并在授权边界内持续改进
```

LoopEvo 必须坚持：

1. **Agent first：** Agent 是长期用户资产，Workflow 只在需要时生成；
2. **Simple outside：** 用户看到 Agent、Loop 和结果，不被内部治理对象淹没；
3. **Complete inside：** Revision、Run、Artifact、Capability、Memory 和 Policy 边界不能省略；
4. **Local private by design：** 本地模式不依赖 LoopEvo 云端，同时准确披露模型 Provider 数据流；
5. **Cloud when useful：** Cloudflare 提供全天候宿主，不污染共享领域模型；
6. **Evolution inside grants：** 低风险变更自动验证和回滚，边界扩张才请求用户；
7. **Vertical proof before abstraction：** 信息流跑通后用第二个 case 验证通用性。

## 明确不做

- 不把 Connector、模型和消息渠道数量当核心竞争力；
- 不从空白画布、节点库和复杂项目配置开始；
- 不把每次聊天强制编译为 Workflow；
- 不把技术可行的订阅复用误当 Provider 官方授权；
- 不在首版并存 Pi、Cloudflare Agents SDK、Temporal 等多个主脑；
- 不为本地模型、团队市场、多 Agent Supervisor 和全平台社交覆盖提前设计；
- 不让网页、Webhook、Skill、MCP 或模型输出改变 Policy；
- 不把进程退出成功当业务成功，Result 必须有来源或验证依据。
