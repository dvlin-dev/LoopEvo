---
title: 产品与工程路线图
scope: repository
status: in_progress
---

# 产品与工程路线图

## 路线图原则

本文记录执行顺序，不代表已实现能力或日期承诺。每个阶段必须交付可验证的用户结果，上一阶段没有形成证据时，不通过增加框架和模块掩盖问题。

1. 通用 Agent 平台定位不变，自动信息流只是首个 vertical case；
2. 先打通本地私有链路，再复用同一内核增加全天候云端；
3. 用户先看到 Agent、Loop 和结果，内部复杂度按需展开；
4. 授权范围内自动执行，只有边界扩大和高风险动作打断；
5. 第二个真实 case 出现前，不冻结通用 Connector、Evaluation 或 Workflow SDK；
6. 云端优先 Cloudflare，但领域语义必须可移植；
7. 本地模型只列未来方向，不进入近期架构和排期。

## 当前状态

当前为 **Pre-alpha / 设计阶段**。仓库只有产品、架构、UI、路线图、安全与协作文档，没有应用、数据库、运行时、Connector、CI 或部署。

## Phase 0：本地私有 Foundation

### 用户结果

用户安装桌面端后，不需要 LoopEvo 云端账号，即可用自然语言创建一个 Agent。Agent 从 RSS 获取增量信息、生成带来源结果、沉淀为每日 Loop；关闭并重启应用后能够恢复状态。

### Phase 0A：关键 Spike

Spike 只验证高风险假设，不建设备用框架：

1. **Pi Local Adapter**
   - 锁定精确版本；
   - 验证流式事件、Tool 请求、暂停 / 恢复、取消、Token / Cost 和上下文恢复；
   - 证明 Tool 在执行前一定经过 LoopEvo Policy 与 Capability Executor；
   - Pi 类型不进入 Kernel Schema、SQLite 和 UI 协议。

2. **Local Run Ledger**
   - 用 SQLite 验证 Schedule、Step Checkpoint、Attempt、Lease 和稳定 `effectId`；
   - 在进程强制退出、设备错过执行周期和重复触发后恢复；
   - 对支持幂等或回执查询的测试 Provider 证明不重复；不支持时验证未知结果停止重放。

### Phase 0B：单一本地 Walking Skeleton

- 最小 TypeScript Workspace：`apps/desktop`、`packages/kernel`、`packages/runtime-pi`、`cases/info-flow`；
- Electron Renderer + Local Host，Renderer 不拥有 Node、文件或进程权限；
- SQLite WAL、内容寻址 Artifact Store 和 OS Keychain；
- `Agent → WorkflowRevision → Activation(local) → Run → Artifact` 最小链路，交互启动时关联 Session；
- 最小 Local Coordinator 串联 Goal →（交互时 Session）→ Pi → Policy → Capability → Artifact → Loop；
- 一个 API Key 模型 Connection，Secret 进入 OS Keychain，提供预算、发送范围预览和 opt-in Live Smoke；
- 一个 RSS Connector 注册 `rss.collect` Capability，Source 配置保留在 info-flow case，并支持 ETag / Last-Modified 或 Item ID 增量；
- Pi 完成相关性判断与带来源摘要；
- PolicyGrant 覆盖已选 RSS、模型、预算和运行周期；
- UI 只提供 Agent 对话、Loop 摘要、Activity 和 Result；
- 结构化事件、定向单元测试和一条真实 E2E。

暂不进入 Phase 0：云端账号、X 生产 Connector、通用 Workflow 画布、ACP、团队权限、向量数据库、本地模型和自动生成代码。

### 退出条件

- Pi 与 Local Run Ledger Spike 有可重复命令、结果与接受 / 拒绝结论；
- 从打包产物在干净用户目录首次启动后，可以创建 Agent、连接模型并运行 RSS Loop；签名、公证和正式分发在发布前单独验收；
- 强制停止本地 Host 后，Run 从 Checkpoint 恢复且不重复 Artifact；
- 设备错过周期时按清楚策略补跑或跳过；
- Secret 不进入 SQLite 正文、Prompt 快照、Artifact、日志或 Git；
- UI 明确说明 Agent、Run、Memory 和 Artifact 保存在本机且不上传 LoopEvo Cloud；选定 Prompt、文件片段和 Tool 结果可能直达 Provider；设备或本地 Host 停止后任务暂停；
- 普通用户无需接触 Workflow 图、Run ID 或逐 Tool Approval 即可完成任务；
- README 的启动方式和实现状态与真实代码一致。

## Phase 1：云端信息流 Alpha

### 用户结果

用户可以选择云端运行。即使电脑关闭，Agent 仍能持续监控 RSS、定向 Web 和一个合法 X Provider，生成带来源简报并通过 Webhook 或邮件交付。

### 云端最小拓扑

- Cloudflare Workers + Static Assets：Web、API、SSE、Webhook 和中心 Schedule Tick；
- Cloudflare Workflows：唯一持久 Run 引擎；
- PostgreSQL via 关闭查询缓存的 Hyperdrive Binding：用户、Revision、Activation、Run、Policy 和 Artifact 元数据事实源；
- R2：原始页面、附件和大 Artifact；
- 经过 Phase 1A Spike 选择的 Pi Executor；
- 每条根记录 `ownerUserId`，不建设组织与团队 RBAC。

### Phase 1A：云端可靠性与可得性 Spike

按以下顺序验证，不先接生产 X：

1. **身份与数据库：** Worker 鉴权、Session 撤销、`ownerUserId` 服务端获取、关闭缓存的 Hyperdrive、事务内 `SET LOCAL` RLS；
2. **Run 派发：** 同一事务写唯一 `(activationId, scheduledFor)`、`executionStatus = queued`、`dispatchStatus = pending` 的 Run 与 `nextRunAt`，事务外创建 Workflow，崩溃后按派发状态重试与对账；
3. **空 AgentRunWorkflow：** 使用小型 ID Payload，在 Step 边界故障注入，验证恢复、取消与当前 CPU / Payload / Result / 保留限制；
4. **R2：** 验证 Workflow 仅传 ID 的 Payload 外置，以及 pending → upload → available、哈希、私有访问、超时清理、孤儿对账与 Tombstone；
5. **Pi Executor：** 只选择 Worker 内或 Workflows 驱动的 Container / Node Executor 一个路径；
6. **云端 RSS E2E：** 串联 Schedule、Run、Workflow、Pi、R2 与 Result，证明恢复后不重复采集和 Artifact；
7. **X Provider：** 核验许可、字段、窗口、增量、删除、配额、费用和替换路径；不可行时不以模拟数据冒充支持；
8. **Delivery：** 用 Webhook 或邮件验证稳定 `effectId`、Provider Receipt、未知结果和不盲目重试。

### 信息源优先级

| 优先级 | 来源 | Alpha 目标 |
| --- | --- | --- |
| P0 | X | 通过官方 API、用户授权或合同有效的 Provider 搜索与增量采集 |
| P0 | RSS / Atom | 产品博客、发布日志、媒体与社区 Feed |
| P0 | 定向公开 Web | 遵守条款、robots 和频率策略，不建设通用爬虫网络 |
| P1 | Reddit | Provider 条款、字段和成本核验后加入 |
| P1 | Webhook / Email | 定时简报和高价值告警 |
| P2 | Bluesky | 按真实价值和接入稳定性加入 |
| Later | Facebook / Instagram | 只有授权和用户价值成立时加入，不承诺覆盖 |

公司版如流通知通过私有 Delivery Adapter 实现同一个契约，不进入开源核心必选依赖。

### 范围

- Agent 首次自动调研竞品、关键词、账号、Feed、社区和数据方案；
- Source Plan 作为 info-flow Artifact / 配置保存，不提升为通用 Kernel 对象；
- backfill 与 incremental 分离，保存 Cursor、重叠窗口、late arrival 和删除语义；
- Source 单飞租约、fencing token、Checkpoint CAS、去重和自适应检查周期；
- 规范化、相关性、聚类、趋势、带来源摘要与 Coverage Gap；
- 用户只确认数据授权、预算和交付边界，日常抓取和恢复自动执行；
- Webhook 校验签名、时间窗、重放 ID 和 Payload Schema；
- 云端预算、Provider 健康、删除、导出和基础账号隔离；
- 本地与云端只通过显式 Revision 导入 / 导出迁移，不自动同步 Run 和 Secret。

### 暂缓的 Cloudflare 能力

- Agents SDK / Durable Objects：直到出现多客户端实时会话或单写者协调；
- Queues：直到有可测的采集突发、扇出或独立投递重试；
- AI Gateway：直到需要多 Provider 路由或统一成本治理；
- Browser Run：静态 HTTP / RSS 不足且访问被允许时按 Source 启用；
- Sandbox / Containers：除 Phase 1A 证明 Pi 需要外，等云端代码执行再引入；Container 所需 DO 只作基础设施绑定；
- Vectorize：等固定检索评测证明 PostgreSQL 搜索不足。

### 退出条件

- 真实目标在连续运行窗口内稳定产出有用、有来源结果；
- X、RSS 与 Web 的授权、删除、费用、覆盖和替代方式可解释；
- PostgreSQL 唯一约束保证逻辑 Run 不重复；外部副作用达到 Capability 声明的 Provider 保证级别，未知结果不会盲目重放；
- Postgres 是唯一业务事实源，Workflows 状态和 R2 可以对账；
- 同一 Source 重叠触发被合并，陈旧 Run 不能提交新 Checkpoint；
- 每个结果可以回到来源、采集时间、处理链和当前覆盖缺口；
- 电脑关闭后云端 Activation 继续运行，本地数据没有被隐式上传；
- 至少一组真实试用证明价值超过 Provider 与模型成本。

## Phase 2：通用 Agent 验证

### 用户结果

用户可以用同样的 Agent、Loop 和 Result 体验完成第二个非信息流任务，例如学习计划、重复数据分析或运营检查。

### 范围

- 选择一个不依赖 Source / Cursor 的真实 case；
- 验证 Agent、Session、WorkflowRevision、Activation、Run、Artifact、Capability、Memory 和 PolicyGrant 是否仍成立；
- 只提取两个 case 都需要的 Capability / Connector / Evaluation 契约；
- 稳定 portable Revision Schema 与 host conformance tests；
- 增加 Dry Run、手动运行、事件 / Schedule 触发与基础结果导出；
- 验证 Codex App Server：账号生命周期、Thread / Turn、双向审批、受控 Workspace / Sandbox、版本范围与 degraded CLI 降级；该探索不阻塞 Phase 0；
- 如出现第二个真实外部 Agent，再实现 ACP Adapter。

### 退出条件

- 第二个 case 不需要修改 Kernel 核心语义即可交付；
- 本地和云端通过同一 Host Conformance Suite；
- 扩展作者无需修改 Kernel 即可添加一个受控 Capability；
- 用户体验仍只需要 Agent、Loop、Activity、Result 和 Connection；
- 没有为了通用性引入空壳页面、未使用包或第二套运行引擎。

## Phase 3：可验证的自动进化

### 用户结果

Agent 能根据反馈、失败、成本和环境变化自动改进低风险参数；用户收到简短变更说明，退化时系统自动回滚。

### 范围

- Run / Artifact Evaluation 与最小固定数据集；
- `ChangeSet = targetType + fromRevisionId + toRevisionId + diff + evidence + evaluation`；
- 只允许单一参数组变化，设置样本量和冷却期；
- 自动回放、Policy / Budget 检查、小范围 Activation 和退化监测；
- Grant 内的查询、过滤、摘要 Prompt、周期和等价 Capability 调整可自动启用；
- 权限、预算、数据用途、保留期、外部写入和代码发布始终请求确认；
- 自动停止、回滚与用户可读 Change Log。

### 退出条件

- 候选相对固定基线的质量、成本和延迟可以重复测量；
- 一项真实反馈能稳定提升结果，而不只是改变措辞；
- 自动启用不扩大 Grant，退化能在既定窗口内停止并回滚；
- 所有变更可追溯到依据、Diff、Evaluation、Activation 和结果；
- 重复 Approval 明显减少，但未授权动作保持为零。

## Phase 4：受控 Coding Extension

### 用户结果

当现有能力不足时，Agent 可以让 Coding Agent 生成候选模块，但候选必须像普通开源贡献一样经过验证和发布。

### 范围

- 将 Phase 2 验证通过的 Codex Adapter 用于正式本地 Coding Capability；
- Claude 使用 API Adapter 或 Companion，未获授权前不消费 Claude.ai 订阅；
- 临时工作区、文件 / 进程 / 网络 / Secret 沙箱；
- 契约测试、Evaluation、依赖、许可和权限报告；
- Diff Review、签名、发布、灰度和回滚；
- 候选模块不被运行中的 Agent 热加载。

### 退出条件

- Coding Agent 无法访问未声明 Secret、宿主路径和网络目标；
- 生成 Connector 通过与人工 Connector 相同的质量和合规检查；
- 发布与回滚有明确操作者、版本、校验值和审计；
- 失败候选不会影响现有 Activation。

## Phase 5：开放生态与团队

候选范围：团队角色、共享 Agent、企业 Identity、审计策略、可信扩展分发、签名与撤销、Managed Cloud / Self-host 导出、数据区域和商业用量。

只有 Phase 2 的公共契约和 Phase 3 的治理在真实使用中稳定后，才建设公共市场与复杂团队能力。

## 横向质量轨道

| 轨道 | 持续要求 |
| --- | --- |
| 简单性 | 用户概念不扩张；新模块必须有第二个实现或已测问题 |
| 安全 | Threat Model、PolicyGrant、沙箱、Secret 和 Prompt Injection 防线 |
| 隐私 | 本地 / 云端数据路径、Provider 披露、最小化、删除与导出 |
| 可靠性 | 幂等、Checkpoint、重试、恢复、对账、升级与回滚 |
| 评估 | 固定样本、回归、用户反馈、质量与成本基线 |
| 可观测 | 统一关联 ID、状态、费用、Provider 健康和审计 |
| 可访问性 | WCAG 2.2 AA、键盘、读屏、移动端和 reduced motion |
| 开发体验 | 一条本地启动路径、定向测试、清楚错误和迁移 |

## 指标框架

- **激活：** 首个有用结果和首个自动 Loop 的耗时；
- **质量：** 有用性、正确性、验证依据、完整性和纠错；信息流另记录噪声、漏报和来源覆盖；
- **自动化：** 无需打断完成率、重复审批率和人工修复次数；
- **可靠性：** Run / Step 成功与恢复、重复、积压和延迟；
- **成本：** 每个 Run、Provider 和有用结果成本；
- **治理：** 未授权动作、边界请求、退化、回滚和删除完成度；
- **进化：** ChangeSet 收益、自动启用成功率、回滚率和过拟合信号。

真实 Alpha 数据形成前只定义指标，不虚构数字目标。

## 明确延后

- 本地模型下载、推理服务和 GPU 调度；
- 完整拖拽 Workflow 画布；
- 企业组织、复杂 RBAC 和公共市场；
- 多层 Supervisor 与 Agent Teams；
- Facebook / Instagram 等全平台覆盖承诺；
- 自动云同步本地文件、Memory、Run、Secret 和浏览器状态；
- Kubernetes、Kafka、Redis、Elasticsearch 和独立向量数据库；
- 无门禁的权限扩张、生产代码发布和不可逆自修改。

## 近期执行顺序

1. 按 `foundation-implementation-plan.md` 创建最小 Workspace 与 Kernel；
2. 完成 Pi Local 与 Local Run Ledger Spike；
3. 打通本地 Coordinator、API Key 模型、RSS → Artifact → Digest → Schedule 恢复；
4. 按“身份 / Postgres → Run 派发 → 空 Workflow → R2 → Pi Executor → 云端 RSS E2E”完成云端可靠性骨架；
5. 核验并接入一个合法 X Provider，再验证 Delivery；
6. 验证 Codex 与第二个非信息流 case，再决定通用 Adapter / SDK，而不是提前扩张抽象。
