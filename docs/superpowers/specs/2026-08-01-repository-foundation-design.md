# LoopEvo Repository Foundation Design

**Date:** 2026-08-01
**Status:** Approved for repository initialization

## Context

LoopEvo starts from a practical information-intelligence use case: a user describes a topic, and the system discovers useful sources, builds a collection and analysis workflow, runs it repeatedly, evaluates the results, and improves the workflow over time.

The product is intentionally broader than social listening. Information acquisition is the first high-quality vertical slice, while the underlying platform is designed for durable AI workflows across research, learning, monitoring, analysis, and other repeatable knowledge work.

## Product Definition

LoopEvo is an open-source platform that turns natural-language intent into governed, reusable, and self-evolving AI workflows.

Its core loop is:

1. Understand the user's intent and success criteria.
2. Discover relevant sources, capabilities, and constraints.
3. Compile a durable, inspectable workflow.
4. Execute the workflow with state, scheduling, and evidence capture.
5. Evaluate quality, cost, freshness, and reliability.
6. Propose versioned improvements under explicit governance.

The system should optimize for reliable outcomes over repeated runs, not merely for completing a single agent session.

## Product Boundaries

LoopEvo is:

- an intent-driven workflow discovery and orchestration layer;
- a durable runtime for scheduled and event-driven work;
- a registry for versioned workflows, capabilities, policies, and evidence;
- an evaluation and governed-evolution system;
- an extensible platform for Skills, MCP servers, APIs, browsers, and coding agents.

LoopEvo is not:

- a claim that arbitrary workflows can safely rewrite themselves without review;
- a replacement for every general-purpose chat agent or automation product;
- a bundled promise of unrestricted access to third-party platforms;
- a social scraper that ignores platform policies, consent, or data licensing.

## Design Principles

1. **Intent first, graph second.** Users describe outcomes; the system explains and persists the workflow it derives.
2. **Durable by default.** Useful work becomes versioned, schedulable, replayable, and observable.
3. **Evidence over confidence.** Outputs retain provenance, run evidence, and evaluation results.
4. **Evolution is governed.** The system proposes changes; tests, policies, approval, canaries, and rollback control adoption.
5. **Capabilities are modular.** Skills, MCP tools, APIs, browser operators, and generated modules share explicit contracts.
6. **Cost is a first-class constraint.** Plans and evaluations consider freshness, coverage, latency, and spend together.
7. **Humans stay in control.** High-impact actions and capability changes cross explicit approval boundaries.
8. **Vertical slices validate the platform.** Topic intelligence is the first end-to-end case, not a special-purpose architecture.

## Conceptual Architecture

The platform has five primary planes:

- **Experience plane:** conversational intent capture, workflow inspection, dashboards, alerts, and approvals.
- **Control plane:** discovery, planning, workflow compilation, policy checks, scheduling, and version management.
- **Execution plane:** isolated runs, state management, retries, queues, and connector invocation.
- **Intelligence plane:** enrichment, synthesis, evaluation, feedback analysis, and evolution proposals.
- **Data plane:** workflow definitions, source registry, artifacts, evidence, lineage, metrics, and secrets references.

Capabilities sit behind adapters and declared contracts. The initial capability layer may include Skills, MCP servers, approved APIs, browser automation, and coding agents. A coding agent may propose or implement a missing module, but generated code must pass the same review, testing, versioning, and deployment controls as human-written code.

## Durable Workflow Artifact

A workflow is a versioned artifact rather than an opaque chat transcript. Its conceptual contract includes:

- intent, scope, and measurable success criteria;
- triggers and scheduling policy;
- typed execution graph and capability dependencies;
- source-selection rationale and access constraints;
- state, checkpoint, deduplication, and retry semantics;
- budget, privacy, safety, and approval policies;
- evaluation suite and acceptance thresholds;
- provenance, version history, and rollback target.

The exact schema will be designed during the foundation phase and evolved through versioned migrations.

## Safe Evolution Model

“Self-evolving” means evidence-driven, governed improvement—not uncontrolled production mutation.

1. Runs produce outputs, metrics, traces, and user feedback.
2. Evaluators identify gaps such as poor recall, stale sources, repeated failures, or excessive cost.
3. The evolution engine drafts a change proposal with expected benefit and risk.
4. The proposal is tested against fixtures, historical runs, policies, and budget limits.
5. Depending on risk, the change is rejected, approved automatically, or sent for human approval.
6. Approved versions are released through a canary and remain rollbackable.

Workflow definitions, prompts, policies, connectors, and generated modules all follow this model.

## Flagship Vertical Slice: Topic Intelligence

The first end-to-end case accepts a natural-language monitoring goal such as tracking a product category, competitors, user complaints, feature requests, and emerging design trends.

The system should:

1. research the topic and propose entities, keywords, competitors, communities, and sources;
2. explain why each source matters and how it can be accessed lawfully;
3. persist the approved source and collection plan;
4. collect incrementally using checkpoints, deduplication, and source-specific strategies;
5. enrich and cluster items, then produce alerts, digests, and inspectable evidence;
6. measure usefulness and evolve source coverage, filters, cadence, and summaries.

Potential sources include X, Reddit, Facebook, Instagram, Bluesky, RSS, websites, and licensed data providers. Availability depends on official APIs, authorization, provider contracts, regional rules, and platform terms. Connectors must make these constraints visible rather than hiding them.

## Initial Repository Deliverables

The initial repository contains:

- a canonical English README;
- a complete Simplified Chinese README;
- an Apache-2.0 license;
- this approved design record;
- an implementation plan for the repository foundation;
- GitHub description and topics aligned with the product definition.

The repository must clearly state that LoopEvo is in pre-alpha/design stage and must not imply that the runtime or integrations already exist.

## Deferred Decisions

The following decisions require implementation-focused research and are intentionally deferred:

- runtime language and framework;
- workflow schema and execution engine;
- deployment topology and tenancy model;
- storage, queue, and observability stack;
- connector licensing and platform-specific access strategy;
- evaluation datasets and automated-approval thresholds;
- security model for secrets, sandboxes, and generated code.

## Acceptance Criteria

- A new visitor can understand the problem, product boundary, core loop, architecture, first use case, roadmap, and current status from the README.
- English and Simplified Chinese documents communicate the same product contract.
- Architecture and evolution safety are represented visually and in prose.
- GitHub metadata is concise, searchable, and consistent with the README.
- All claims distinguish vision and planned capabilities from implemented functionality.
