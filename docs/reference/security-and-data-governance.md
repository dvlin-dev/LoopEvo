---
title: 安全与数据治理
scope: repository
status: active
---

# 安全与数据治理

## 适用范围

本规范适用于 LoopEvo 的桌面端、云端、Agent、Loop、Capability、Connector、Skills、MCP、浏览器、Coding Agent、数据存储、评估和自动进化。

安全目标是：普通用户在已授权范围内尽量不被打断，同时任何 Agent 都不能越过宿主沙箱、真实凭据和 PolicyGrant。

## 执行模式与隐私承诺

| 模式 | LoopEvo 业务数据 | 外部请求 | 默认同步 |
| --- | --- | --- | --- |
| 本地私有 | Agent、Memory、Run、Artifact、索引和配置保存在用户设备 | 设备可直接访问模型、数据源和通知 Provider | 无 |
| LoopEvo 云端 | 保存在用户选择的云端账号与区域 | 由 LoopEvo 云端访问已连接 Provider | 不与本地自动同步 |

“本地私有”只能承诺 **数据不经过或存入 LoopEvo 云端**，不能表述为“数据永远不离开设备”。模型提示、用户选中的文件片段、Tool 结果和查询可能由设备直接发送给 OpenAI、Anthropic 或其他用户选择的 Provider。

产品必须在连接 Provider 与首次发送敏感上下文前说明：

- 接收方、目的、数据类别和大致范围；
- 是否保留、训练或受 Provider 自身政策约束；
- 如何缩小文件、目录、字段和时间范围；
- 如何停止、撤销 Connection 和删除本地派生数据。

本地模型属于未来能力，不作为当前隐私承诺的前提。

## 本地私有边界

- 默认不要求 LoopEvo 账号，不连接 LoopEvo 云端控制面；
- SQLite、Artifact Store 和本地索引不得放入默认云盘同步目录；
- Secret 使用操作系统钥匙串或由 Codex / Claude 等 Provider 客户端自行保管；
- Renderer 不直接访问 Node、Shell、文件系统、Cookie 或 Secret；
- 文件 Capability 默认只访问用户选择的文件或文件夹，符号链接和路径逃逸必须拦截；
- 本地日志默认只保留必要元数据，Prompt、文件正文和 Tool 返回不自动写入诊断日志；
- 本地私有默认不上传业务 Payload；崩溃报告必须由用户显式选择并在发送前脱敏；
- 更新检查只发送必要版本元数据，不得成为远程控制或业务数据通道；
- 导出默认不包含 Secret、Connection Token、浏览器状态和本地文件内容；
- 删除必须覆盖结构化记录、Artifact、索引、缓存和可控备份。

电脑关闭、休眠或本地 Host 停止后，定时任务可能延迟或暂停。产品不得用“本地持续运行”暗示云端级别的全天候可用性。

## 云端隔离

首版只建设基础多用户，不提前建设企业组织模型：

- 所有根记录携带 `ownerUserId`，值只能来自验证后的 Session，不能由请求 Payload 指定；
- Alpha 使用关闭查询缓存的 Hyperdrive Binding；认证、Policy、预算、Lease、Checkpoint 和读后写路径不得读取缓存；
- 应用查询约束与 PostgreSQL 行级策略双重隔离；RLS 上下文必须在每个显式事务中通过 `SET LOCAL` 设置，不能依赖连接池 Session 状态；
- Scheduler / Workflow 使用独立数据库角色，并在查询中显式限制 owner；
- R2 使用不可猜测对象键，并只经授权 Worker 返回；
- Workflow Payload 只传实体引用，不携带明文 Token；
- 每位用户拥有独立预算、并发和采集频率限制；
- 平台级 Secret 使用 Cloudflare Secret；用户凭据使用信封加密，主密钥与数据分离；
- 如引入 AI Gateway，默认关闭 Prompt / Response Payload 日志，只保留必要用量元数据；
- Durable Objects 如被引入，只保存可重建连接态，不保存第二份业务事实。

团队、组织、共享所有权、跨成员审批和复杂 RBAC 必须另行设计，不能从 `ownerUserId` 临时拼接。

## 数据访问与来源合规

- 第三方平台数据必须通过官方 API、用户明确授权、有效商业合同或符合平台条款与适用法律的方式获得。
- 公开可见不自动等于允许批量收集、长期保存、训练模型或再次分发。
- 每个 Connector 必须声明来源、授权主体、允许用途、字段范围、地区限制、保留期限和删除方式。
- 不得绕过登录、验证码、访问控制、付费限制、速率限制或技术保护措施。
- 使用第三方数据 Provider 时，必须核验数据来源、许可链、服务条款、删除能力和责任边界。
- API、价格、配额和条款会变化；Connection 建立和重要续期时必须重新验证。

## Connector 与 Connection

每个 Connector 至少公开：

- 所需账号、权限、授权方式和支持地区；
- 可访问对象、字段、历史窗口和覆盖限制；
- Event、Webhook、Stream、Cursor 或 Polling 语义；
- Checkpoint、去重、幂等、删除和断点续传语义；
- 速率限制、调用成本、延迟、配额和错误模型；
- 数据保留、导出、撤销、删除和审计能力；
- 服务条款与授权来源。

Connection 是具体用户的授权实例。Connector 不能把授权失效、覆盖下降或数据缺口静默包装成“成功”，也不能从一个 Connection 扩大到未授权账号或数据范围。

## Secret 与外部 Agent 认证

- API Key、Token、Cookie、密码和私钥不得进入源码、Git、文档、Prompt、Run Artifact、截图、测试夹具或普通日志。
- 凭据只在执行具体 Capability 时注入，且不得通过 Tool Result 返回给 Agent。
- 日志和错误信息必须移除 Authorization Header、完整请求体和可复用会话信息。
- Connection 撤销或泄漏时，系统必须能定位影响、停止 Activation、轮换凭据并删除缓存。
- Codex App Server 自行管理 ChatGPT 登录；LoopEvo 不读取、复制或上传 `~/.codex/auth.json`。
- Codex 委派运行在宿主限定的 Workspace、Sandbox 与网络范围；其命令和文件审批请求必须映射到 PolicyGrant，不能默认无条件批准。
- Anthropic 书面批准前，LoopEvo 不提供 Claude.ai 登录，也不代表用户消费 Free、Pro 或 Max 订阅额度；正式后台集成使用用户 API Key 或支持的云 Provider。
- 本地 Agent 的订阅凭据永远不能自动迁移到 LoopEvo 云端。

## PolicyGrant 与低打扰审批

授权不是每个 Tool Call 的弹窗。用户确认一次后，应形成包含 subject、action、resource、destination、数据用途 / 保留、目标宿主、预算窗口、有效期和撤销状态的 `PolicyGrant`。

在 Grant 内可以自动执行并记录：

- 公开只读访问和已连接数据源读取；
- 用户已选择目录内的必要读取；
- 已批准模型、预算、通知目标和运行周期；
- 可逆的重试、恢复、去重和低风险参数调整。

以下动作必须停止并请求确认：

- 新 Credential、私有数据或新的本地路径；
- 权限、预算、数据用途、保留期或网络目标扩大；
- 对外发送、发布、账号修改、付款和新的写入目标；
- 删除、覆盖和其他不可逆动作；
- 将生成代码或新 Capability 发布到真实执行环境；
- Provider 合规边界发生实质变化。

Host 从验证后的交互 Session 或计划 Run 持久事实构造包含 owner、Agent、Run、宿主、数据用途 / 保留和当前 / 预计预算的 PolicyContext；模型与请求 Payload 不能指定这些字段。Policy Engine 产生 `allow / ask / deny` 后，Host 持久化绑定 Decision、Request Hash、Run 和过期时间的单次 Authorization Record，再构造 `AuthorizedCapabilityCall`；Capability Executor 不能直接接受模型选择的 Grant。同一受信进程内回读并原子消费记录，跨进程或服务信任边界才要求 MAC。拒绝必须安全停止相关分支，不诱导用户扩大权限。无人值守 Run 不继承交互 Session 的临时 Grant。

## 个人信息与受许可数据

- 采集前明确目的、最小字段、合法依据、保留期和可访问主体。
- 账号标识、画像、情感分析和跨平台关联可能构成个人信息处理，不能只按普通文本治理。
- UI 应展示当前来源、保存数据、用途、保留期和删除入口。
- 删除覆盖原始数据、派生 Artifact、索引、缓存和可控备份；无法立即删除的范围必须说明。
- 导出和分析不得扩大原授权用途或泄漏第三方受限内容。
- 发送给模型前应支持字段最小化、敏感信息过滤和内容范围预览；过滤不能取代用户授权。

## 浏览器与本地计算机操作

- 官方 API、RSS 和结构化授权优先；只有不足且操作被允许时才使用浏览器。
- 浏览器和命令执行使用独立 Capability、明确的域名 / 路径 / 命令范围与资源上限。
- Electron Renderer 只加载打包资源，保持 CSP 与 `webSecurity`，禁用 `webview`、未授权导航和新窗口；IPC 校验 sender / frame、Origin 与输入 Schema。
- preload 不暴露通用 `exec`、任意路径读取和原始 Electron API；Secret 永不返回 Renderer。
- 页面、网页内容、Webhook、Skill、MCP 返回和模型输出一律视为不可信输入，不能改变 Policy。
- 发送消息、发布内容、付款、删除、账号或授权变更属于高影响动作。
- 出现验证码、权限不明、页面结构异常或目标漂移时安全停止，不能通过扩大范围盲目重试。
- 截图、录屏、Trace 和调试输出必须避免 Token、Cookie、个人数据和商业敏感信息。

## Coding Agent 与生成模块

- Coding Agent 可以生成候选 Connector、Skill 或评估器，但不能被正在运行的 Agent 热加载。
- 候选模块必须具备所有者、源码、依赖、许可、权限、输入输出契约和版本。
- 生成代码接受与人工代码相同的 Review、测试、安全扫描、Secret 检查和许可证检查。
- 执行环境隔离文件、网络、进程、Secret 和资源；默认拒绝未声明能力。
- 自动下载依赖或外部代码时，必须验证来源、完整性、许可和供应链风险。
- 生产启用需要显式授权、签名或校验值、灰度和回滚；不能由低风险自动进化 Grant 覆盖。

## 受治理的自动进化

低风险变更可以自动启用，但必须满足：

1. 由运行证据、评估或用户反馈指出可衡量问题；
2. 生成不可变的新 Revision 和最小 ChangeSet；
3. 在隔离环境执行静态检查、历史回放、质量、预算和 Policy 验证；
4. 不扩大权限、预算、数据用途、保留期和外部副作用；
5. 使用最小样本、冷却期和受控范围，避免对噪声过拟合；
6. 持续测量，退化时自动停止并回滚；
7. 保存 Diff、依据、评估、激活和回滚记录，并向用户提供简洁说明。

扩大边界的变更进入 `ask`，不是自动失败，也不是静默执行。生产代码与高风险 Capability 始终走软件发布门禁。

## 成本与资源

- Activation 声明预算、并发、最大重试和停止条件。
- 增量采集优先使用 Event、Webhook、Stream 和可靠 Cursor；Polling 根据来源价值、更新历史和配额自适应。
- 每个模型、Provider、浏览器和外部副作用可归因到 Run 与用户。
- 成本异常、无限重试、事件风暴和下游积压必须触发限流、降级或暂停。
- 缓存、去重和批处理不能破坏删除、权限更新和来源可追溯性。

## 审计与事件响应

- 高影响动作记录操作者、Revision、Grant、目标、时间和结果，不记录 Secret。
- 数据访问、Run、Artifact 和自动变更能追溯到来源、Capability 版本和处理链。
- 发生凭据泄漏、越权、错误采集或误发时，优先停止相关 Activation 并保护必要证据。
- 响应至少覆盖影响评估、凭据轮换、数据隔离 / 删除、必要通知、修复验证和规则回写。

## 新设计检查

涉及外部数据或自动执行的新设计必须说明：

- 运行目标是 local 还是 cloud；
- 数据、模型和授权链路；
- Secret、路径、网络和 Capability 边界；
- 保留、删除、导出和跨端迁移；
- 费用、配额、失败和恢复；
- PolicyGrant、必须打断的动作和撤销方式；
- 评估、灰度、停止和回滚。

缺少这些内容的设计不能进入生产实现。
