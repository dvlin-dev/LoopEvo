---
title: Foundation 实施计划
scope: repository
status: in_progress
---

# Foundation 实施计划

> **供 Agent 执行：** 在当前任务中逐项完成，或把本文交给后续任务；使用复选框维护真实进度。

**目标：** 交付一个不依赖 LoopEvo 云端的桌面端最小闭环：自然语言创建 Agent，通过正式模型 Connection 增量读取 RSS，生成带来源结果，并沉淀为可恢复的本地 Loop。

**架构：** 使用 TypeScript 共享 Kernel；Electron Main 只管理窗口、Keychain 与 IPC，通过 `utilityProcess.fork` 启动 Local Host；SQLite 与本地文件系统保存事实和 Artifact；Pi 只负责 Agent Loop，Tool 必须经过 Policy 与 Capability Executor。首个 case 直接放在 `cases/info-flow`，不提前抽象通用 Connector SDK。

**技术栈：** Node.js 22.19.0、pnpm workspace、TypeScript、Vitest、Electron、React、Vite、electron-builder、SQLite、Pi。

## 全局约束

- 本地模式不调用 LoopEvo 云端；模型与 Source 请求从设备直达用户选择的 Provider。
- Renderer 不拥有 Node、文件、进程、Cookie 或 Secret 权限。
- 用户只看到 Agent、Loop、Activity、Result 和必要 Connection。
- 一次任务不强制创建 Workflow；只有重复、等待或恢复任务才创建 WorkflowRevision / Activation。
- WorkflowRevision 固定 `agentRevisionId`，Activation 只切换 `activeWorkflowRevisionId`。
- 所有依赖使用精确版本并提交 lockfile；Pi 类型不得进入 Kernel、数据库和 UI 协议。
- Grant 内低风险动作自动执行；权限、预算、数据用途、外部写入和代码发布扩大必须停止。
- 不实现云同步、本地模型、Codex、ACP、X、完整画布、团队权限、公共市场或自动代码发布。
- Git 提交只有在当前执行任务已经获得用户明确授权时进行；否则保留可审查工作区。

---

### Task 1：建立最小 Workspace 与 Kernel

**Files:**

- Create: `package.json`
- Create: `pnpm-workspace.yaml`
- Create: `.npmrc`
- Create: `.nvmrc`
- Create: `tsconfig.base.json`
- Create: `packages/kernel/package.json`
- Create: `packages/kernel/src/index.ts`
- Create: `packages/kernel/src/schema.ts`
- Create: `packages/kernel/src/ports.ts`
- Create: `packages/kernel/src/policy.ts`
- Create: `packages/kernel/test/schema.test.ts`
- Create: `packages/kernel/test/policy.test.ts`

**Interfaces:**

- Produces: `Agent`、`AgentRevision`、`Session`、`WorkflowRevision`、`Activation`、`Run`、`Artifact`、`CapabilityManifest`、`Connection`、`PolicyGrant` 的版本化 Schema。
- Produces: `AgentRuntimeAdapter`、`PolicyEngine`、`CapabilityExecutor`、`RunRepository`、`ArtifactRepository` 宿主端口。

- [ ] **Step 1：初始化四个 Workspace 的根配置**

只声明 `apps/desktop`、`packages/kernel`、`packages/runtime-pi`、`cases/info-flow`。根脚本提供 `check`、`test` 与定向 `dev`；不增加 Turborepo、Nx、发布系统和无代码目录。

- [ ] **Step 2：先写领域不变量失败测试**

覆盖：Agent 具有 `activeAgentRevisionId`；WorkflowRevision 固定 `agentRevisionId`；Activation 只绑定 WorkflowRevision 和一个 target；Run 固定版本；Artifact 具有 Provenance；Connection 只保存 Secret Reference、不含 Secret；CapabilityManifest 声明风险、执行宿主和输入 / 输出 / 错误 Schema；不符合 Schema 的 Tool 输入必须在执行前失败。

- [ ] **Step 3：实现最小 Schema 与端口**

```ts
export type ExecutionTarget = "local" | "cloud";

export type JsonValue =
  | null
  | boolean
  | number
  | string
  | JsonValue[]
  | { [key: string]: JsonValue };

export interface RuntimeStatus {
  available: boolean;
  version?: string;
  reason?: string;
  degraded?: boolean;
}

export interface RunInput {
  runId: string;
  agentRevisionId: string;
  initiatingSessionId?: string;
  modelBinding?: {
    connectionId: string;
    modelId: string;
  };
  inputArtifactIds: string[];
}

export interface CapabilityRequest {
  callId: string;
  capabilityId: string;
  action: string;
  resource: string;
  destination?: string;
  input: JsonValue;
  effectId?: string;
}

export interface CapabilityResult {
  callId: string;
  outputArtifactIds: string[];
  receiptId?: string;
  providerIdempotencyKeyAccepted?: boolean;
}

export interface EffectCapabilities {
  createsSideEffect: boolean;
  retryMode: "none" | "at_least_once";
  supportsProviderIdempotency: boolean;
  supportsReceiptLookup: boolean;
  compensatingCapabilityId?: string;
}

export interface CapabilityManifest {
  id: string;
  version: string;
  targetHosts: ExecutionTarget[];
  risk: "read" | "write" | "destructive";
  effects: EffectCapabilities;
  inputSchemaId: string;
  outputSchemaId: string;
  errorSchemaId?: string;
  resourceScopes: string[];
  destinationScopes: string[];
}

export interface RuntimeCheckpoint {
  runtime: string;
  schemaVersion: number;
  runtimeSessionId?: string;
  state: JsonValue;
}

export interface RuntimeAuthorizationRequest {
  requestId: string;
  action: string;
  resource: string;
  details?: JsonValue;
}

export interface RuntimeOutput {
  text: string;
  data?: JsonValue;
}

export type AgentEvent =
  | { type: "session_started"; checkpoint: RuntimeCheckpoint }
  | { type: "message_delta"; text: string }
  | { type: "checkpoint"; checkpoint: RuntimeCheckpoint }
  | { type: "capability_request"; request: CapabilityRequest; checkpoint: RuntimeCheckpoint }
  | { type: "authorization_request"; request: RuntimeAuthorizationRequest; checkpoint: RuntimeCheckpoint }
  | { type: "completed"; output: RuntimeOutput; checkpoint: RuntimeCheckpoint }
  | { type: "failed"; code: string; message: string; checkpoint?: RuntimeCheckpoint };

export type RuntimeResumeInput =
  | {
      type: "capability_result";
      runId: string;
      checkpoint: RuntimeCheckpoint;
      callId: string;
      result: CapabilityResult;
    }
  | {
      type: "policy_decision";
      runId: string;
      checkpoint: RuntimeCheckpoint;
      requestId: string;
      decision: PolicyDecision;
    };

export interface PolicyGrant {
  id: string;
  subject: { type: "user" | "agent"; id: string };
  actions: string[];
  resources: string[];
  destinations: string[];
  dataPolicy: { purposes: string[]; retentionDays: number };
  targetHosts: ExecutionTarget[];
  budget: { currency: string; limit: number; window: "run" | "day" | "month" };
  expiresAt: string;
  revokedAt?: string;
}

export interface PolicyContext {
  ownerUserId: string;
  agentId: string;
  runId: string;
  subject: PolicyGrant["subject"];
  targetHost: ExecutionTarget;
  dataPurpose: string;
  retentionDays: number;
  budget: {
    currency: string;
    window: PolicyGrant["budget"]["window"];
    used: number;
    estimatedIncrement: number;
  };
  now: string;
}

export type PolicyDecision =
  | { type: "allow"; decisionId: string; grantId: string }
  | { type: "ask"; reason: string; minimumScope: Partial<PolicyGrant> }
  | { type: "deny"; reason: string };

export interface AuthorizedCapabilityCall {
  authorizationRecordId: string;
  request: CapabilityRequest;
  decisionId: string;
  grantId: string;
  requestHash: string;
  runId: string;
  expiresAt: string;
  authorizationProof?: string;
}

export interface AgentRuntimeAdapter {
  probe(): Promise<RuntimeStatus>;
  start(input: RunInput): AsyncIterable<AgentEvent>;
  resume(input: RuntimeResumeInput): AsyncIterable<AgentEvent>;
  cancel(runId: string): Promise<void>;
}

export interface PolicyEngine {
  authorize(
    request: CapabilityRequest,
    context: PolicyContext,
    grants: PolicyGrant[],
  ): Promise<PolicyDecision>;
}

export interface CapabilityExecutor {
  execute(call: AuthorizedCapabilityCall): Promise<CapabilityResult>;
}
```

`RuntimeCheckpoint` 必须是可序列化 JSON、带版本且不含 Secret。交互请求可以携带 `initiatingSessionId`，计划任务不依赖产品 Session；Provider Session ID 只属于 Checkpoint。`modelBinding` 只携带 Connection / Model 标识，不携带 Credential；Pi 路径要求该字段，非模型外部 Agent 可以省略。Runtime Adapter 不写 Artifact，`completed.output` 由 Coordinator 持久化为带 Provenance 的 Artifact。未知上游事件只进入脱敏 Runtime Telemetry；若影响控制流则失败关闭，不增加伪造的 Domain Event。

- [ ] **Step 4：运行 Kernel 检查**

Run: `pnpm --filter @loopevo/kernel test && pnpm --filter @loopevo/kernel check`

Expected: Schema、Policy、授权调用防伪造测试通过，TypeScript 无错误。

- [ ] **Step 5：按授权状态交付**

如当前任务已经获得提交授权：

```bash
git add package.json pnpm-workspace.yaml .npmrc .nvmrc tsconfig.base.json packages/kernel pnpm-lock.yaml
git commit -m "feat: establish LoopEvo kernel"
```

否则只报告未提交差异。

### Task 2：实现本地事实库与 Run Ledger

**Files:**

- Create: `apps/desktop/package.json`
- Create: `apps/desktop/src/host/index.ts`
- Create: `apps/desktop/src/host/database.ts`
- Create: `apps/desktop/src/host/migrations/001-foundation.sql`
- Create: `apps/desktop/src/host/run-ledger.ts`
- Create: `apps/desktop/src/host/artifact-store.ts`
- Create: `apps/desktop/src/main/secret-store.ts`
- Create: `apps/desktop/src/shared/host-protocol.ts`
- Create: `apps/desktop/test/run-ledger.test.ts`
- Create: `apps/desktop/test/artifact-store.test.ts`
- Create: `apps/desktop/test/host-boundary.test.ts`

**Interfaces:**

- Consumes: Kernel Repository 端口。
- Produces: 运行在 Agent Host 进程内的 SQLite / Artifact 实现、Main 持有的 OS Keychain Secret Store，以及不向 Renderer 暴露的 typed Host Protocol。

- [ ] **Step 1：写进程恢复与幂等失败测试**

覆盖重复 Run、Step 完成后崩溃、租约过期接管、陈旧 owner 提交、重复 `effectId`、结果未知和错过 Schedule。测试固定 `Run.executionStatus`、`Step.status` 与 `EffectReceipt.status` 的独立枚举；`delivery_unknown` 只属于 Effect Receipt，并使 Step / Run 带错误码失败。

- [ ] **Step 2：建立最小 SQLite Schema**

只创建 Agent / Revision、Session、WorkflowRevision、Activation、Run / Step、Artifact Metadata、CapabilityManifest、Connection、PolicyGrant、Checkpoint、Lease、Authorization Record 和 Effect Receipt 表；启用 WAL、Foreign Key 与显式事务。Evaluation / ChangeSet 延后。

- [ ] **Step 3：实现 Run Ledger**

每次领取生成递增 generation；Run / Step 状态与下一 Checkpoint 在同一事务提交；错过周期按 Activation `catchUpPolicy` 补跑一次或跳过。相同 effect 先读取 Receipt；Provider 结果未知且不支持幂等 / 查询时将 Effect Receipt 标为 `delivery_unknown`，让 Step / Run 失败并停止重放。

- [ ] **Step 4：实现 Artifact 与 Secret 边界**

Artifact 使用 SHA-256 内容地址和 `pending → available`；先写 pending 元数据，再写文件并校验，最后标记 available；清理超时 pending 与孤儿文件。Secret 写入接口只返回 opaque reference；解析 Secret 只允许 Main 监管的 Host 请求，值只经进程通道进入 Host 内存，永不返回 Renderer 或写入 SQLite。测试使用内存 Fake。Host Protocol 只允许白名单请求、关联 ID、Schema 验证和超时 / 取消；Host 不接受 Renderer 消息，Main 不执行 Agent Loop 或访问业务数据库。

- [ ] **Step 5：验证并按授权状态交付**

```bash
pnpm --filter @loopevo/desktop test -- run-ledger artifact-store host-boundary
```

如已获提交授权：

```bash
git add apps/desktop/package.json apps/desktop/src/host apps/desktop/src/main/secret-store.ts apps/desktop/src/shared/host-protocol.ts apps/desktop/test pnpm-lock.yaml
git commit -m "feat: add local durable run ledger"
```

### Task 3：验证并封装 Pi Runtime

**Files:**

- Create: `packages/runtime-pi/package.json`
- Create: `packages/runtime-pi/src/pi-runtime-adapter.ts`
- Create: `packages/runtime-pi/src/event-mapper.ts`
- Create: `packages/runtime-pi/test/fixtures/agent-events.jsonl`
- Create: `packages/runtime-pi/test/pi-runtime-adapter.test.ts`
- Create: `docs/plans/pi-runtime-spike.md`

**Interfaces:**

- Consumes: Kernel `AgentRuntimeAdapter`；构造时注入 runtime-pi 自己定义的 `PiModelConnectionResolver` 端口。
- Produces: 只输出 LoopEvo `AgentEvent` 与 `RuntimeCheckpoint` 的 `PiRuntimeAdapter`。

- [ ] **Step 1：记录精确 Pi 版本与 Spike 验收项**

先从 Pi 当前 `engines.node` 选择兼容的精确 Node / Pi 版本；验收 stream、Tool Request、暂停 / 恢复、取消、follow-up、Token / Cost 和进程重启恢复。文档在验证前保持 `in_progress`。

- [ ] **Step 2：写事件与 Policy 拦截失败测试**

测试证明 Adapter 遇到 Tool Request 后只发 `capability_request`，不直接执行 Tool；每个暂停点先产生可持久化 Checkpoint；取消后不再产生请求；未知控制事件失败关闭。Pi 启动缺少 `modelBinding` 时给出稳定错误；Resolver 只收到 `modelBinding` 和 Run Context，Secret 不进入 RunInput、事件、Checkpoint 或夹具。

- [ ] **Step 3：实现 Adapter**

`PiRuntimeAdapter` 通过构造注入的 `PiModelConnectionResolver` 把 opaque `modelBinding` 解析为 Pi 可用的模型实例，runtime-pi 不依赖 desktop；Local Host 实现 Resolver，并仅在内存中取得 Main Keychain 返回的 Secret。Pi Session 转换为版本化 RuntimeCheckpoint 或可重建的规范化 Session History，不能只保留进程内引用。Capability Result 通过 `capability_result + callId` 恢复；外部 Agent 授权请求必须携带稳定 `requestId`，通过 `policy_decision + requestId` 恢复。Pi Tool 永远不持有 Secret 或直接网络能力。

- [ ] **Step 4：执行真实 Spike 与夹具回放**

Run: `pnpm --filter @loopevo/runtime-pi test`

Expected: 夹具回放一致；真实 Spike 记录版本、命令、故障注入、结果和接受 / 拒绝结论。

- [ ] **Step 5：按授权状态交付**

如已获提交授权：

```bash
git add packages/runtime-pi docs/plans/pi-runtime-spike.md pnpm-lock.yaml
git commit -m "feat: add isolated Pi runtime adapter"
```

### Task 4：实现 Local Coordinator 与正式模型 Connection

**Files:**

- Create: `apps/desktop/src/host/agent-coordinator.ts`
- Create: `apps/desktop/src/host/policy-engine.ts`
- Create: `apps/desktop/src/host/capability-executor.ts`
- Create: `apps/desktop/src/host/model-connections/openai.ts`
- Create: `apps/desktop/src/host/model-connections/pi-model-connection-resolver.ts`
- Create: `apps/desktop/test/agent-coordinator.test.ts`
- Create: `apps/desktop/test/model-connection.test.ts`
- Create: `apps/desktop/test/live/openai-model.smoke.ts`

**Interfaces:**

- Consumes: Kernel Ports、Local Repository、Main Keychain 的窄 Host Protocol 与 Pi Adapter。
- Produces: `Goal →（交互时 Session）→ Pi → Policy → AuthorizedCapabilityCall → Artifact → WorkflowRevision / Activation` 单一协调链路。

- [ ] **Step 1：先写 Coordinator 失败测试**

覆盖一次性 Run、创建每日 Loop、Grant allow / ask / deny、撤销 Grant、预算耗尽、进程恢复、模型取消、模型返回 Tool Request，以及 Tool 输入 / Provider 输出不符合 Manifest Schema 时执行前后分别失败。

- [ ] **Step 2：实现 Policy 与 Capability Executor**

Coordinator 从交互 Session 或计划 Run 的持久事实构造 PolicyContext；Policy Engine 读取有效 Grant 并生成 Decision，Host 持久化绑定 Decision ID、Request Hash、Run 与 Expiry 的单次 Authorization Record，再构造 AuthorizedCapabilityCall。同一受信 Host Utility Process 内，Executor 必须回读并原子消费该记录；只有调用跨进程或服务信任边界时才要求验证 MAC `authorizationProof`。Executor 还要按 Manifest 引用的 Schema 校验输入 / 输出 / 错误，并验证宿主、Connection、预算与沙箱。模型不能传入 owner、context 或 grant ID 绕过选择。

- [ ] **Step 3：实现一个 API Key 模型 Connection**

首个正式路径使用用户自己的 OpenAI API Key；Secret 写入 OS Keychain，数据库只存引用。Connection 保存模型选择、预算、最近健康与撤销状态；Resolver 通过 `modelBinding.connectionId` 查 Connection，再向 Main 请求对应 Secret Reference，并构造 Pi Model，Secret 只停留在 Host 内存。首次发送敏感上下文前返回接收方、字段范围和预计费用供 UI 展示。

- [ ] **Step 4：分离自动测试与 Live Smoke**

普通测试使用本地 Fake Server；`openai-model.smoke.ts` 仅在用户显式提供测试凭据时运行，发送最小无敏感内容请求，只记录模型、耗时、Token 与结果状态，不记录 Key 和正文。

- [ ] **Step 5：验证并按授权状态交付**

```bash
pnpm --filter @loopevo/desktop test -- agent-coordinator model-connection
```

如已获提交授权：

```bash
git add apps/desktop/src/host/agent-coordinator.ts apps/desktop/src/host/policy-engine.ts apps/desktop/src/host/capability-executor.ts apps/desktop/src/host/model-connections apps/desktop/test/agent-coordinator.test.ts apps/desktop/test/model-connection.test.ts apps/desktop/test/live
git commit -m "feat: coordinate local Agent execution"
```

### Task 5：打通 RSS 信息流 case

**Files:**

- Create: `cases/info-flow/package.json`
- Create: `cases/info-flow/src/rss-connector.ts`
- Create: `cases/info-flow/src/rss-capability.ts`
- Create: `cases/info-flow/src/info-flow-loop.ts`
- Create: `cases/info-flow/src/result-artifact.ts`
- Create: `cases/info-flow/test/rss-capability.test.ts`
- Create: `cases/info-flow/test/info-flow-loop.test.ts`

**Interfaces:**

- Consumes: Kernel Capability、Run、Artifact、Local Coordinator 与 runtime-pi。
- Produces: RSS Connector 注册的 `rss.collect` Capability、case 内 Source 配置、增量 Checkpoint、带 Provenance 的 Digest Artifact。

- [ ] **Step 1：写 RSS 增量与恢复失败测试**

覆盖 ETag、Last-Modified、Item ID、Feed 重排、同 ID 更新、空结果、超时、429、无效 XML、重复 Run 和 Checkpoint 崩溃恢复。

- [ ] **Step 2：实现 RSS Connector 与 Capability**

Connector 声明 Manifest 与健康检查，Capability 只访问允许的 Feed URL；先保存原始响应与 Item Artifact，再原子推进 Checkpoint；摘要失败不得重复请求已经保存的 Feed。

- [ ] **Step 3：实现固定的首个 Loop**

流程仅包含 `collect RSS → normalize → relevance → summarize with provenance → save Result`。Source Plan 保存在 info-flow 配置，不修改 Kernel Schema。

- [ ] **Step 4：运行 case 测试**

Run: `pnpm --filter @loopevo/info-flow test`

Expected: 增量、恢复、来源和预算测试通过。

- [ ] **Step 5：按授权状态交付**

如已获提交授权：

```bash
git add cases/info-flow pnpm-lock.yaml
git commit -m "feat: add local RSS information loop"
```

### Task 6：交付最小 Electron 用户体验

**Files:**

- Modify: `apps/desktop/package.json`
- Modify: `pnpm-lock.yaml`
- Create: `apps/desktop/src/main/index.ts`
- Create: `apps/desktop/src/main/ipc.ts`
- Create: `apps/desktop/src/main/host-client.ts`
- Create: `apps/desktop/src/preload/index.ts`
- Create: `apps/desktop/src/renderer/App.tsx`
- Create: `apps/desktop/src/renderer/views/agent-workspace.tsx`
- Create: `apps/desktop/src/renderer/components/loop-card.tsx`
- Create: `apps/desktop/src/renderer/components/activity-card.tsx`
- Create: `apps/desktop/src/renderer/components/result-card.tsx`
- Create: `apps/desktop/src/renderer/views/privacy-settings.tsx`
- Create: `apps/desktop/e2e/local-info-flow.spec.ts`
- Create: `apps/desktop/e2e/renderer-security.spec.ts`
- Create: `apps/desktop/e2e/packaged-app.spec.ts`
- Create: `apps/desktop/electron-builder.yml`

**Interfaces:**

- Consumes: Local Coordinator 的 typed preload API。
- Produces: Agent 对话、Loop、Activity、Result 和 Connection 最小界面，以及可做干净首次启动验证的本地打包产物。

- [ ] **Step 1：先写 E2E 失败路径**

用户创建目标、连接 Fake Model、添加 RSS、授权一次、运行、查看来源、设为每日 Loop、重启应用并看到恢复结果。另测恶意 Renderer、IPC sender、路径逃逸、任意命令、未授权导航和新窗口。

- [ ] **Step 2：建立安全 Electron 边界**

Renderer 只加载打包资源，开启 Context Isolation / Sandbox，关闭 Node Integration，保持 `webSecurity`，配置 CSP，禁用 `webview`、未授权导航与新窗口。Main 校验 IPC sender / frame、Origin 与输入 Schema；preload 不暴露通用 exec、任意路径和原始 Electron API。`host-client.ts` 使用 `utilityProcess.fork` 启动并监管 `src/host/index.ts`，只通过 Host Protocol 通信；Host 崩溃可重启，Main 不加载 Coordinator、数据库或 Pi。首版不实现第二套 Child Process 启动路径。

- [ ] **Step 3：实现单一 Agent Workspace**

默认页面只有对话时间线和 Composer；Loop、Activity、Result 以内联卡片与 Inspector 展示，不增加 Workflow、Run、Evaluation 和 Approval 顶级导航。

- [ ] **Step 4：实现准确的隐私说明**

界面原文必须表达：LoopEvo 的 Agent、Run、Memory 和 Artifact 保存在本机，不上传 LoopEvo Cloud；完成任务时，用户选择的 Prompt、文件片段和 Tool 结果可能直达所选 Provider；设备或本地 Host 停止后 Loop 暂停。

- [ ] **Step 5：打包并验证首次启动**

使用 electron-builder 生成当前目标平台的本地安装产物；从全新临时 user-data 目录启动打包应用，完成首次运行、创建 Agent、Fake Model、RSS Loop、退出与恢复。该检查只证明打包闭环；代码签名、公证、自动更新和跨平台发布在正式分发前单独验收，不用未签名产物冒充正式 Release。

- [ ] **Step 6：验证并按授权状态交付**

```bash
pnpm --filter @loopevo/desktop test
pnpm --filter @loopevo/desktop package
pnpm --filter @loopevo/desktop e2e -- local-info-flow renderer-security packaged-app
```

如已获提交授权：

```bash
git add apps/desktop/package.json apps/desktop/src/main/index.ts apps/desktop/src/main/ipc.ts apps/desktop/src/main/host-client.ts apps/desktop/src/preload apps/desktop/src/renderer apps/desktop/e2e apps/desktop/electron-builder.yml pnpm-lock.yaml
git commit -m "feat: deliver local Agent workspace"
```

### Task 7：完成 Foundation 验证与事实回写

**Files:**

- Modify: `README.md`
- Modify: `README.zh-CN.md`
- Modify: `docs/design/core/system-architecture.md`
- Modify: `docs/reference/repository-context.md`
- Modify: `docs/reference/testing-and-validation.md`
- Modify: `docs/plans/roadmap.md`

- [ ] **Step 1：运行定向完整检查**

```bash
pnpm check
pnpm test
pnpm --filter @loopevo/desktop package
pnpm --filter @loopevo/desktop e2e -- local-info-flow renderer-security packaged-app
```

Expected: Kernel、Local Host、Pi、Coordinator、RSS 和 Electron E2E 通过。

- [ ] **Step 2：执行故障恢复验收**

分别在模型返回后、Capability 执行前、Artifact available 前和 Checkpoint 提交后停止 Local Host；恢复后不得重复 Artifact，副作用达到 Manifest 声明的保证级别。

- [ ] **Step 3：执行 opt-in Live Smoke**

使用用户提供的测试 API Key 执行最小模型请求与真实 RSS Feed；不记录正文和 Secret。若没有凭据，交付必须明确 Live Smoke 未覆盖，不能把 Fake E2E 写成真实 Provider 验证。

- [ ] **Step 4：回写真实实现状态**

README 只描述已实现入口；Repository Context 记录真实目录、版本与命令；Testing 文档记录已实际运行的检查。Pi Spike 完成后把稳定结论回写架构并按知识库协议删除完成的过程文档。

- [ ] **Step 5：按授权状态提交最终回写**

如已获提交授权：

```bash
git add README.md README.zh-CN.md docs/design/core/system-architecture.md docs/reference/repository-context.md docs/reference/testing-and-validation.md docs/plans/roadmap.md
git commit -m "docs: record Foundation implementation facts"
```

## 完成条件

- 用户可以在不使用 LoopEvo 云端的情况下，通过正式模型 Connection 完成真实 RSS Loop；
- 本地业务事实、Provider 数据路径和设备 / Host 在线限制表达准确；
- 进程恢复、Artifact 幂等和未知外部副作用有自动化证据；
- 打包产物可在干净 user-data 目录首次启动并完成 Fake E2E；
- Pi 形成一个明确、可重复的集成结论；
- UI 没有暴露多余内部概念；
- 文档、代码、测试命令与真实能力一致。
