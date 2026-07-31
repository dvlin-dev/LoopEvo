---
title: 产品与工程路线图
scope: repository
status: in_progress
---

# 产品与工程路线图

## 路线图原则

本文记录当前执行顺序，不代表已实现能力或日期承诺。每个阶段必须交付可验证的用户闭环；上一阶段退出条件未满足时，不以增加功能数量掩盖核心问题。

优先级原则：

1. 先做一个真正长期运行的主题情报工作流，再抽象通用平台；
2. 先保证来源合法、证据可追溯、运行可恢复，再扩大 Connector 数量；
3. 先让 Agent 提出改进并由人审批，再讨论自动晋升；
4. 先使用单 Agent + 隔离 Worker，再依据测量结果增加并行编排；
5. 以退出证据推进阶段，不用虚假日期和功能清单代表进度。

## 当前状态

当前为 **Pre-alpha / 设计阶段**。产品、架构、UI、治理和路线图已经形成文档；仓库尚无可运行应用、数据库、Worker、Connector、CI 或部署。

## Phase 0：Foundation Walking Skeleton

### 用户结果

贡献者可以在本地启动一条真实但受限的链路：用户输入主题，系统从一个 RSS Source 采集内容、生成摘要并保存 Evidence；页面能看到 Run，Worker 重启后仍能恢复。

### Phase 0A：高风险 Spike

Spike 先于产品脚手架，产物是可运行证据和技术决策，不追求可复用 UI：

1. **Pi：** 精确锁版本，通过 Adapter 验证 stream、tool request 暂停 / 恢复、abort、follow-up、cost 和事件映射；LoopEvo API / Schema 不得暴露 Pi 类型。若权限代理无法在 Tool 执行前稳定拦截，或升级夹具必须大面积修改领域代码，则拒绝当前版本。
2. **Temporal：** 用同一链路验证 Durable Timer、Signal、Activity Retry、Worker Stop / Restart、Cancel、Continue-As-New 和跨两个 Worker Build 的 Replay / Versioning。开发环境必须有一条文档化启动路径，自托管不得依赖 Cloud 专有能力。
3. **跨系统一致性：** 验证数据库提交后启动 Workflow、重复请求、Worker 崩溃和恢复对账；确定稳定 Workflow ID 与幂等规则；验证同一 Source 的 single-flight Lease、超时接管、fencing token 和 checkpoint CAS，确保陈旧 Run 无法推进检查点或清理新状态。
4. **Evidence：** 验证原始 Artifact、内容哈希、派生摘要、引用和级联删除；由真实大小决定 Phase 0 是否需要对象存储。
5. **X Provider：** 在写 Connector 前核验许可、搜索字段、历史窗口、增量方式、删除、配额、费用和可替换性。

Temporal Spike 只有全部通过才升级为实现技术。若被拒绝，只使用同一 `WorkflowRuntimePort` 和同一负载验证 Trigger.dev；完成对比并更新架构后选择一个运行时，禁止双引擎并存。

### Phase 0B：单一 Walking Skeleton

- 最小 TypeScript workspace：Web、Worker 与 Domain / Runtime Adapter；
- 无正式账号系统的本地单 Workspace；
- 一个 RSS Connector 和一个顺序受限的 WorkflowSpec 子集；
- Goal → 固定模板 WorkflowVersion → Run → RSS → Pi Summary → Evidence；
- PostgreSQL 最小 Schema、迁移、稳定 Run ID 与必要的一致性机制；
- Conversation、Run、Evidence 三个最小视图；
- Secret Reference 与默认拒绝的 Tool Policy，不建设完整权限中心；
- 关联 ID、结构化日志和该链路的 Type / Unit / E2E 检查。

对象存储、完整 Outbox、通用图解释器、正式身份、完整 OTel Collector、Projects 信息架构和扩展 SDK 只有被 Spike 或这条链路证明需要时才进入 Phase 0，否则后移。

### 退出条件

- Pi、Workflow Runtime 和 Evidence Spike 均有可重现报告、接受 / 拒绝结论和更新后的事实源；
- 从主题输入到 WorkflowVersion、Run、RSS Evidence 和摘要的本地 E2E 可重复通过；
- 中途停止并重启 Worker 后，Run 从检查点恢复且不重复副作用；
- 同一幂等键重复提交不创建第二个逻辑 Run；
- 同一 Source 的重叠触发被合并；Lease 超时接管后，旧 Run 的写入与 checkpoint 提交被拒绝；
- UI、数据库和日志使用同一组关联 ID；
- Secret 不进入日志、Prompt 快照、Evidence 或 Git；
- README 的启动、定向测试和真实实现状态与代码一致。

## Phase 1：Topic Intelligence Alpha

### 用户结果

用户描述一个长期主题，系统完成首次调研，提出信源与预算方案；用户批准后，系统持续采集增量信息、生成有引用的简报，并通过 Web 与通用 Webhook 交付。

### MVP 来源

| 优先级 | 来源 | 目标 |
| --- | --- | --- |
| P0 | X | 必须验证；通过官方 API、用户授权或合同明确的 Provider 实现搜索与增量采集 |
| P0 | RSS / Atom | 低成本、稳定的产品博客、发布日志和媒体来源 |
| P0 | 普通公开网页 | 受 robots、条款和频率策略约束的定向页面，不做通用爬虫 |
| P1 | Reddit | 在 API / Provider 条款和成本核验后加入社区讨论 |
| P1 | 通用 Webhook / Email | 交付摘要与高优先级告警 |
| P2 | Facebook / Instagram / Bluesky | 按真实用户价值、授权可得性和成本决定，不在 Alpha 承诺 |

公司版的如流通知通过私有 Adapter 使用同一 Delivery Contract，不进入开源核心的必选依赖。

### 范围

- Goal Summary、Research Plan、Source Proposal 与授权确认；
- SourceStrategyVersion：选择理由、覆盖、许可、费用、刷新方式和缺口；
- RSS、Web 和一个 X Provider Connector；
- backfill 与 incremental 分离；页内 `inProgressScanCheckpoint`、跨轮 `durableCheckpoint`、late arrival、去重和限流；每个 Source 使用 single-flight Lease、fencing token 与 checkpoint CAS 防止重叠 Run 回退进度；
- 内容规范化、相关性分类、聚类、趋势与带引用总结；
- Run Progress、Evidence、Coverage Gap 与 Source Health；
- 即时告警、每日 / 每周 Digest、Webhook Delivery；
- 用户反馈：有用、噪声、错误、漏报；
- Provider 与模型预算、停止条件和成本归因；
- 正式 Project / Provider 身份、必要的对象存储、Outbox Dispatcher 和 OpenTelemetry 投影；
- Projects、Create、Workflow、Sources、Notifications 与 Usage 页面。

### Provider 选择门槛

X 或其他社交 Provider 必须同时满足：

- 来源和再处理许可链路可说明；
- 支持所需搜索字段、历史窗口和可靠增量 checkpoint（Provider cursor、since-id 或复合 watermark）；
- 配额、限流、计费、内容更新 / 删除、撤订和地区限制可获取；
- 授权失效、覆盖下降和上游故障能够被健康检查发现；
- 可通过 Connector 契约替换，领域模型不保存 Provider 专有类型。

### 退出条件

- 首要用户无需编辑流程图即可创建主题、审查来源并启动 Workflow；
- Alpha 实施计划在试用前固定验证时窗、每类来源最小样本、逻辑重复定义、故障注入场景和结果报告格式；
- RSS、Web、X 在固定验证方案内连续运行，不产生定义内的逻辑重复；授权失效和限流状态可见；
- 每条核心结论可以回到原始 Evidence，无法覆盖的来源被明确标注；
- 同一逻辑交付键只产生一次用户可见交付；Provider 调用采用预算预留、请求回执和有限重试，并在报告中说明无法消除的重复计费窗口；
- 用户可以暂停后恢复；取消为终态，只能从已提交 checkpoint 创建新 Run；用户可以修改预算并撤销当前 Release；
- 通过 401 / 429 / timeout / late arrival / Worker restart / delivery retry、重叠 Run、Lease 超时接管和陈旧提交拒绝场景；
- 用真实试用报告建立有用信号率、噪声率、重复率、延迟和每个有用信号成本基线。

## Phase 2：Reusable Workflow Core

### 用户结果

主题情报中被验证的能力可以组合成其他长期目标；开发者能通过稳定契约扩展 Connector、Capability 和 Delivery，而不用修改核心运行时。

### 范围

- WorkflowSpec v1 正式兼容规则、JSON Schema 和迁移；
- Capability / Connector SDK、契约测试套件和示例；
- manual、schedule、event、webhook、adaptive wake-up；
- 版本 Diff、Dry Run、Replay、Release、Rollback；
- 逐能力 allow / ask / deny、最小网络与 Secret 范围；
- Source、Capability、Prompt 和 Model 的独立版本固定；
- 可导出 / 导入的 Workflow Template；
- 自托管安装、备份、升级、Temporal Worker Versioning、旧 Worker 保留、历史 Replay、`doctor` 与故障恢复说明；
- n8n / MCP / Webhook 等边界桥接，而非核心依赖。

### 退出条件

- 至少一个非主题情报场景只通过新增配置或 Capability 完成；
- 第三方开发者能按照 SDK 实现 Connector，并通过分页、checkpoint、乱序更新、删除、撤订、限流、幂等和错误契约测试；
- 旧 WorkflowVersion 在升级后由兼容 Worker 继续运行或 Continue-As-New；历史 Replay 在发布前阻止非确定性变更；
- Dry Run / Replay 不触发未授权外部副作用；
- 自托管者能完成安装、升级、备份恢复和版本回滚演练。

## Phase 3：Governed Evolution

### 用户结果

系统能从反馈、失败、覆盖下降和成本变化中提出最小工作流改进，并用可重复证据说明候选版本是否优于当前版本。

### 范围

- Evaluation Dataset、Evaluator、Baseline 与 Experiment；
- 反馈、失败和 Coverage Gap 聚类；
- `EvolutionProposal` 与结构化 Workflow Diff；
- 静态校验、历史回放、离线质量 / 成本 / 安全对比；
- 人工 Review、Canary、晋升、停止和 Rollback；
- Prompt、Source、Filter、Model、Capability 与 Workflow 结构的独立变更；
- 退化监控和 Proposal 审计。

### 自动化边界

- 默认只自动生成 Proposal，不自动晋升；
- 只有可逆、低风险、固定评估门槛且积累足够真实证据的变更，才可逐类开启有限自动批准；
- Source 授权、外部写入、数据删除、预算上调、生产代码和权限扩大始终需要人工审批。

### 退出条件

- 候选版本在固定数据集上可重复对比质量、成本和延迟；
- Review 页面能解释改了什么、为什么、证据、风险和回滚点；
- Canary 退化可停止当前 Run 并把后续 Release 路由回稳定版本；已发生的通知或外部写入只能通过显式补偿能力处理；
- 所有晋升能追溯到 Proposal、评估、审批和 Release；
- 至少一种真实用户反馈能稳定提高下一版本指标，而非只改变模型文案。

## Phase 4：Sandboxed Coding Extension

### 用户结果

当缺少 Connector、Skill 或评估器时，系统可以让 Coding Agent 在隔离环境生成候选模块，并以普通开源贡献的质量门槛交付 Review。

### 范围

- 可替换的 Codex / Claude Code / OpenHands / Pi Coding Adapter；
- 临时分支或一次性 Sandbox、文件 / 进程 / 网络 / Secret 策略；
- 自动生成契约测试、Eval、依赖、许可证、权限和供应链报告；
- PR / Diff Review、签名、发布与 Capability Registry；
- 失败模块隔离、撤销和版本兼容。

### 退出条件

- Coding Agent 无法访问未声明 Secret、宿主目录和网络目标；
- 候选模块不会被运行中的 Workflow 动态加载；
- 生成 Connector 必须通过与人工 Connector 相同的契约、安全和许可证检查；
- 发布与回滚具备明确操作者、版本、校验值和审计记录。

## Phase 5：Open Ecosystem & Teams

### 用户结果

团队可以安全共享项目和能力；开源社区可以分发有来源、可验证、可撤销的模板与扩展。

### 候选范围

- 团队角色、项目共享、审批策略和企业 Identity；
- Connector / Skill / Workflow Template 注册表；
- 包签名、来源证明、兼容矩阵、恶意扩展举报和撤销；
- Managed Cloud 与 Self-host 的可移植导出；
- 用量、配额、审计、数据区域和保留策略；
- 经验证的生态贡献流程。

只有 Phase 2 的 SDK 和 Phase 3 的治理在真实使用中稳定后，才建设公共市场。

## 横向质量轨道

以下工作不是最后补齐，而是每阶段的进入条件：

| 轨道 | 持续要求 |
| --- | --- |
| 安全 | Threat Model、最小权限、Secret、依赖与容器扫描、Prompt Injection 防线 |
| 数据治理 | 来源许可、保留、删除、导出、地域、个人信息和受限内容 |
| 可靠性 | 幂等、重试、恢复、对账、备份、升级和回滚演练 |
| 可观测 | 统一关联 ID、Trace、成本、Provider 健康、审计和告警 |
| 评估 | 固定数据集、回归、用户反馈、质量与成本基线 |
| 开发者体验 | 一条本地启动路径、可重复测试、清楚错误和版本迁移 |
| 无障碍 | WCAG 2.2 AA、键盘、读屏、移动端和 reduced motion |

## 指标框架

在 Alpha 首批真实数据形成前只定义指标，不虚构数字目标：

- **激活：** 首个 Source Proposal、首个 WorkflowVersion、首个有用结果耗时；
- **质量：** 有用信号率、噪声率、漏报反馈、证据覆盖和摘要纠错率；
- **新鲜度：** 来源事件到采集、分析和交付的延迟；
- **可靠性：** Run / Step 成功与恢复、重复、积压、数据缺口和交付失败；
- **成本：** 每个 Run、Source、模型调用和每个有用信号成本；
- **治理：** 未授权动作、审批等待、退化、回滚和删除完成度；
- **进化：** Proposal 接受率、候选相对基线提升、Canary 退化与撤销率。

Phase 1 结束后，用真实基线为 Phase 2 设置数值 SLO；SLO 按部署形态和 Provider 能力区分。

## 明确延后

- 完整拖拽式 Workflow 编辑器；
- 通用个人助理、桌面控制和语音；
- 多层 Supervisor、Agent Teams 和 Agent 社交网络；
- Facebook / Instagram 等全部平台覆盖承诺；
- 公共扩展市场和商业计费系统；
- Kubernetes、Kafka、Redis、Elasticsearch 和独立向量数据库；
- 无人工门禁的 Workflow、Skill、Connector 和生产代码自修改。

延后项只有在真实用户结果、故障或测量数据证明其必要性时进入新设计，不因竞品已有而自动加入。

## 近期执行顺序

1. 为 Phase 0 编写可执行的 Foundation 设计与实施计划；
2. 完成 Pi Adapter、Temporal 重放和 X Provider 可得性三个高风险 Spike；
3. 先打通 RSS → Evidence → Digest 的低成本垂直链路；
4. 接入一个经授权的 X Provider，并验证增量、删除、成本与覆盖；
5. 用真实试用反馈决定 Reddit、搜索和下一类工作流，而不是提前扩张平台数量。
