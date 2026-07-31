---
title: LoopEvo 知识库治理实施计划
scope: repository
status: in_progress
---

# LoopEvo 知识库治理实施计划

> **供 Agent 执行：** 在当前任务中逐项执行，并使用复选框（`- [ ]`）记录进度。

**目标：** 建立以 `CLAUDE.md` 为唯一入口、中文知识库为事实源、计划与稳定事实分离的 LoopEvo 仓库文档体系。

**架构：** 根目录负责项目身份、硬边界和文档路由，`docs/CLAUDE.md` 负责知识库分类与生命周期。稳定产品事实进入 `docs/design`，跨任务工程规则进入 `docs/reference`，执行期文档留在 `docs/plans`，完成后回写并删除。

**技术：** Git、Markdown、相对符号链接、Mermaid

## 全局约束

- `README.md` 保持英文，`README.zh-CN.md` 保持完整中文镜像。
- `CLAUDE.md` 与 `docs/` 下知识库正文统一使用中文。
- `AGENTS.md` 必须是指向同作用域 `CLAUDE.md` 的相对符号链接。
- 不创建 `archive`、`.gitkeep`、占位 README 或尚无事实支撑的工程规范。
- 已完成计划在稳定事实回写后删除，历史由 Git 保存。
- 不修改 README 产品文案、许可证或 GitHub 仓库元数据。
- 不把规划能力描述为已经实现的功能。

---

### 任务 1：建立仓库级和 docs 级唯一入口

**文件：**

- 新建：`CLAUDE.md`
- 新建：`AGENTS.md`，类型为相对符号链接，目标为 `CLAUDE.md`
- 新建：`docs/CLAUDE.md`
- 新建：`docs/AGENTS.md`，类型为相对符号链接，目标为 `CLAUDE.md`

- [ ] 编写根入口，包含项目身份、Pre-alpha 状态、硬边界、文档路由与协作规则。
- [ ] 编写 docs 入口，包含目录职责、权威顺序、文档状态、语言格式、生命周期和查找方式。
- [ ] 创建两个相对符号链接，并使用 `test -L` 与 `readlink` 验证目标。

### 任务 2：建立通用参考规范

**文件：**

- 新建：`docs/reference/repository-context.md`
- 新建：`docs/reference/collaboration-and-delivery.md`
- 新建：`docs/reference/testing-and-validation.md`
- 新建：`docs/reference/security-and-data-governance.md`

- [ ] 记录当前真实仓库资产、尚未确定的技术决策和核心阅读入口。
- [ ] 定义调研、最小变更、用户修改保护、外部反馈验证、提交授权和事实回写流程。
- [ ] 定义 L0、L1、L2 风险验证基线，并只提供当前仓库真实可执行的文档检查命令。
- [ ] 定义第三方数据、密钥、浏览器操作、Coding Agent、工作流进化和个人数据边界。

### 任务 3：迁移稳定产品与架构事实

**文件：**

- 新建：`docs/design/core/product-and-architecture.md`
- 删除：`docs/superpowers/specs/2026-08-01-repository-foundation-design.md`
- 删除：`docs/superpowers/plans/2026-08-01-repository-foundation.md`

- [ ] 将现有英文设计翻译并收敛为中文稳定事实。
- [ ] 保留产品定义、边界、原则、生命周期、系统分层、工作流契约、安全进化和主题情报用例。
- [ ] 明确区分已采纳设计与当前尚未实现的系统能力。
- [ ] 删除已经完成的初始化计划和旧目录结构。

### 任务 4：验证知识库结构与内容

**文件：**

- 检查：`CLAUDE.md`
- 检查：`AGENTS.md`
- 检查：`docs/CLAUDE.md`
- 检查：`docs/AGENTS.md`
- 检查：`docs/design/core/product-and-architecture.md`
- 检查：`docs/reference/*.md`

- [ ] 运行 `git diff --check`，预期无输出。
- [ ] 运行 `test -L AGENTS.md && test "$(readlink AGENTS.md)" = "CLAUDE.md"`，预期退出码为 0。
- [ ] 运行 `test -L docs/AGENTS.md && test "$(readlink docs/AGENTS.md)" = "CLAUDE.md"`，预期退出码为 0。
- [ ] 运行 `rg --files docs | sort`，确认目录只包含设计允许的文件。
- [ ] 运行 `rg -n 'docs/superpowers|docs/index\.md|archive/' CLAUDE.md docs --glob '*.md'`，除迁移计划自身外预期无旧路径引用。
- [ ] 检查 Front Matter、相对链接、中文正文、规划/现状措辞和敏感信息。

### 任务 5：结束执行期文档并发布

**文件：**

- 删除：`docs/plans/2026-08-01-knowledge-base-governance-design.md`
- 删除：`docs/plans/2026-08-01-knowledge-base-governance.md`

- [ ] 确认设计与实施计划中的稳定事实已经进入入口、设计或参考文档。
- [ ] 删除两个完成的执行期文档，并确认 `docs/plans` 不通过占位文件强制保留。
- [ ] 检查完整差异，只暂存本任务文件。
- [ ] 提交知识库重构并推送到用户指定的 GitHub 仓库 `main`。
- [ ] 从 GitHub 回读最新提交、两个软链类型和核心文档内容，确认远端与本地一致。
