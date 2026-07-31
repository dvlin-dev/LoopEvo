# LoopEvo

> 本文件是仓库级协作入口，只记录稳定身份、硬边界、协作约束和文档路由。

## 项目定位

LoopEvo 是一个开源平台，目标是将自然语言意图转化为可治理、可复用、自进化的 AI 工作流。

项目关注的不是一次 Agent 会话能否完成任务，而是如何把有效过程沉淀为可调度、可观测、可评估、可版本化并可安全改进的持久工作流。主题情报与信息获取是首个规划中的端到端场景，但平台边界不限于舆情监控。

## 当前阶段

项目当前处于 **Pre-alpha / 设计阶段**：

- 仓库已经形成产品定义、架构方向和知识库治理基线；
- 尚无可运行的应用、工作流运行时或生产连接器；
- README 和设计文档中的系统能力属于已采纳方向或规划能力，除非代码与验证证据明确证明，否则不得描述为已经实现。

当前真实仓库状态见 `docs/reference/repository-context.md`。

## 核心同步协议

1. 根目录 `CLAUDE.md` 是仓库级唯一协作入口。
2. 根目录 `AGENTS.md` 必须是指向 `CLAUDE.md` 的相对符号链接，用于兼容 agents.md 规范；禁止双写。
3. `docs/CLAUDE.md` 是知识库唯一导航与治理入口，`docs/AGENTS.md` 同样通过相对符号链接兼容。
4. 新需求的设计说明、实施计划和执行期评审放在 `docs/plans/*`。
5. 已采纳的核心产品与架构事实进入 `docs/design/core/*`；已落地功能的稳定事实进入 `docs/design/features/*`。
6. 跨任务复用的工程规则、协作约束和验证流程进入 `docs/reference/*`。
7. HTML 与静态原型进入 `docs/prototype/*`，只表达方向，不作为产品或实现事实源。
8. 已完成计划必须先回写稳定事实再删除；历史过程依赖 Git、Issue 和 PR，不建立 `archive/`。

## 硬边界

- 不得把目标架构、路线图或原型描述为当前已实现能力。
- 第三方平台数据必须通过官方 API、明确授权、有效合同或符合条款的方式访问。
- 密钥、令牌、Cookie、个人信息和受许可数据不得写入代码、日志、测试样例或文档。
- 浏览器操作、外部消息、账号变更和其他不可逆动作必须遵循最小权限，并取得与风险相称的明确授权。
- Coding Agent 生成的模块必须接受与人工代码相同的评审、测试、沙箱、发布和回滚控制。
- “自进化”只能产生可审计、可版本化、可验证、可回滚的变更，不能静默改写生产工作流。
- 技术栈尚未确定时，不根据偏好虚构框架、命令、接口或部署事实。

完整安全规则见 `docs/reference/security-and-data-governance.md`。

## 文档路由

- 知识库治理与分类：`docs/CLAUDE.md`
- 当前仓库上下文：`docs/reference/repository-context.md`
- 协作与交付：`docs/reference/collaboration-and-delivery.md`
- 测试与验证：`docs/reference/testing-and-validation.md`
- 安全与数据治理：`docs/reference/security-and-data-governance.md`
- 产品与核心架构：`docs/design/core/product-and-architecture.md`
- 执行中设计与计划：`docs/plans/*`
- 已落地功能事实：`docs/design/features/*`
- HTML 与静态原型：`docs/prototype/*`

## 协作规则

- 对话语言跟随用户语言；知识库正文使用中文，技术名称和代码符号保留官方写法。
- 开始任务前先读本文件、`docs/CLAUDE.md` 及任务相关事实源，再检查当前代码、Git 状态和真实环境。
- 先区分已验证事实、已采纳设计、执行中计划和未验证判断；不以历史计划代替当前实现证据。
- 优先复用已有契约和模块；聚焦当前任务，不进行无关重构、批量格式化或元数据扰动。
- 保护用户已有修改；工作区存在无关变更时，只暂存当前任务文件。
- 外部反馈、Issue 和评审意见必须先通过代码、测试或复现验证，再决定是否修改。
- 按风险执行最小充分验证，不为纯文档改动运行无关的全量构建。
- AI Agent 未经用户明确授权不得执行 `git commit`、`git push`、`git tag`、发布或外部不可逆操作。
- 交付时说明实际改动、实际验证、未覆盖范围和仍然存在的风险，不用“应该可用”代替证据。

## 查找方式

优先使用快速、可重复的仓库命令：

```bash
rg --files
rg -n "关键词" .
git status -sb
git log --oneline --decorate -10
```

具体测试和文档迁移检查见 `docs/reference/testing-and-validation.md`。
