---
status: draft
tags: [dicoogle-agent]
---

[[Home]] · [[System Context]] · [[Components]] · [[Sequence]] · [[Architecture Decisions]]

# Architecture v0.1

**Status:** logical proposal. Host, APIs and deployment topology remain unconfirmed.

## Responsibilities
| Component | Responsibility | Boundary |
|---|---|---|
| Interface/API | Requests, clarification, review, results | Does not grant access rights |
| Request Coordinator | Request state and routing | Does not replace validator rules |
| Context Service | Resolve target and assemble permitted context | It is not Dicoogle |
| Dicoogle Adapter | Map abstract operations to verified interfaces | No automatic fallback routing |
| Versioned Tool Catalog | Approved tool contracts and profiles | Changes require governance/versioning |
| Planner | Manual plan first; LLM proposal later | No arbitrary tool calls or code |
| Validator | Deterministic rules and explanations | Does not establish clinical correctness |
| Approval Control | Check required approvals | LLM output is not authorization |
| Executor | Run authorized nodes and handle outcomes | Only approved catalog adapters |
| Tool Adapters | Inspect, assess, and generate report | Validate output against contract |
| Persistent Ledger | Record decisions, attempts, and outputs | Not merely diagnostic console logging |

## First implementation slice
Pure domain types, contracts and deterministic validator with fixtures. No HTTP endpoint, LLM, frontend, SDK change, plugin, or workflow engine is required for this slice. Use mock context/adapters. Integrate with a real Dicoogle deployment only after confirming backend/version, metadata APIs, and object-level authorization.

## Proposed states
`proposed → validated → awaiting_approval` (if required) `→ queued → running → completed`.
Alternative states: `invalid`, `awaiting_clarification`, `blocked`, `failed`, `cancelled`. A rejected candidate is not a failed tool execution. Corrections produce a new revision.

## Time-dependent conditions
Check catalog/plan structure at planning time. Re-check volatile access/availability before execution and before the affected step. A future output is an obligation, not an already-known value. Revalidation reduces stale-context risk but cannot prevent every race or adapter failure.

## Trust and data handling
Treat user requests, LLM proposals, metadata strings and adapter outputs as untrusted. Resolve opaque aliases on the server. Do not assume DICOM UIDs or hashes are anonymized. Define allowlists for LLM context; avoid PHI in prompts, errors and ledger. Review/redact persisted prompts. These controls need implementation and institutional confirmation.

## Decisions pending
See [[Architecture Decisions]], [[Dicoogle Integration]], [[Approval and Governance]], [[Threat Model]], and [[Privacy and Retention]].
