# LoopEvo Repository Foundation Implementation Plan

> **For agentic workers:** Execute this plan task-by-task in the current session or hand it off to another task. Track progress with checkbox (`- [ ]`) syntax.

**Goal:** Establish a production-quality public repository foundation that explains LoopEvo accurately in English and Simplified Chinese and makes the project discoverable on GitHub.

**Architecture:** Documentation is organized around one canonical product contract: natural-language intent is compiled into a durable workflow that executes, evaluates, and evolves under governance. The README presents this contract at project level, while the design record preserves boundaries and deferred decisions for future implementation work.

**Tech Stack:** Git, GitHub repository metadata, Markdown, Mermaid, Apache License 2.0

## Global Constraints

- Keep all capability claims explicitly at vision or planned status until implementation exists.
- Keep the English and Simplified Chinese READMEs semantically equivalent.
- Do not select a runtime technology before implementation research.
- Prefer concise architecture contracts over speculative component detail.
- Push directly to `main` only because the owner explicitly approved initialization of this empty repository.

---

### Task 1: Record the Approved Product Design

**Files:**
- Create: `docs/superpowers/specs/2026-08-01-repository-foundation-design.md`
- Create: `docs/superpowers/plans/2026-08-01-repository-foundation.md`

- [x] Capture the product definition, boundaries, principles, architecture, safe-evolution model, flagship use case, and deferred decisions.
- [x] Define acceptance criteria that prohibit presenting planned capabilities as implemented.
- [x] Review both documents for internal consistency and clean Markdown formatting.
- [x] Commit the design and plan with message `docs: define repository foundation`.

### Task 2: Create the Canonical English Project README

**Files:**
- Create: `README.md`

- [x] Add the LoopEvo name, tagline, one-sentence definition, language link, and pre-alpha notice.
- [x] Explain the problem, product boundary, core loop, and product principles.
- [x] Add valid Mermaid diagrams for the lifecycle and conceptual architecture.
- [x] Document the capability model, durable workflow artifact, governance, and safe evolution.
- [x] Present topic intelligence as the flagship vertical slice with platform-access caveats.
- [x] Add differentiation, roadmap, contribution guidance, and license reference.

### Task 3: Create the Simplified Chinese Mirror

**Files:**
- Create: `README.zh-CN.md`

- [x] Mirror every substantive section and diagram from `README.md`.
- [x] Preserve technical meaning instead of translating mechanically.
- [x] Link back to the canonical English README.

### Task 4: Add the Project License

**Files:**
- Create: `LICENSE`

- [x] Add the complete Apache License 2.0 text.
- [x] Verify GitHub can identify the license after push.

### Task 5: Validate and Commit Repository Content

**Files:**
- Validate: `README.md`
- Validate: `README.zh-CN.md`
- Validate: `LICENSE`
- Validate: `docs/superpowers/specs/2026-08-01-repository-foundation-design.md`
- Validate: `docs/superpowers/plans/2026-08-01-repository-foundation.md`

- [x] Run `git diff --check` and resolve whitespace errors.
- [x] Search for placeholders or claims that imply unimplemented functionality.
- [x] Inspect the full staged diff and confirm no unrelated files are included.
- [x] Commit with message `docs: establish LoopEvo project foundation`.

### Task 6: Publish and Configure GitHub

**Files:**
- Push the two documentation commits to `origin/main`.

- [x] Set the repository description to the approved one-sentence definition.
- [x] Add focused GitHub topics for agentic workflows, orchestration, evolution, capabilities, and topic intelligence.
- [x] Push `main` with upstream tracking.
- [x] Read back the remote README, default branch, description, topics, and detected license.
- [x] Confirm the local working tree is clean and record the final commit identifiers.
