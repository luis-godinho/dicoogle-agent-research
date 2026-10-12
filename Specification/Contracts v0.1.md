---
status: draft
tags: [dicoogle-agent]
---

[[Home]] · [[Rules and Tests]] · [[JSON Schemas]] · [[Reference Use Case]]

# Contracts v0.1

**Status:** proposed model and illustrative JSON; not implemented. All fixtures are synthetic.

## 1. DICOM reference
Required fields: `schema_version`, opaque `reference_id`, `level` (`study|series|instance`), `study_instance_uid`, and server-resolvable `resolver_key`. A `series`/`instance` reference also requires `series_instance_uid`; an `instance` reference requires `sop_instance_uid`. Reject fields that contradict the declared level. Resolver keys resolve only through a configured adapter, never arbitrary URLs, paths, commands, or code.

The LLM receives an allowlisted alias/context, not trusted authority to resolve arbitrary UIDs. UIDs and hashes are not presumed anonymous.

Example: [`examples/dicom-reference.json`](../examples/dicom-reference.json).

## 2. Tool contract
Required: `schema_version`, `id`, `version`, named input/output ports, parameter constraints, preconditions, effects, approval policy, trusted `executor_key`, limits and failure codes.

- Port types are nominal strings. v0.1 compatibility requires exact equality; no implicit coercion.
- Parameters have required names, constraints and `additional_properties=false`.
- Each precondition has a registered deterministic handler and phase: `planning`, `before_step`, or both. Never evaluate contract text as code.
- Effects are declarative labels; they do not grant permission.
- `executor_key` maps to a server-approved adapter, never an LLM-supplied command.
- Initial limits: positive timeout; one attempt; no automatic retry.

The JSON snippets are domain descriptors, not complete published JSON Schemas. See [`examples/tool-catalog.json`](../examples/tool-catalog.json).

## 3. Plan
Required: `schema_version`, `plan_id`, positive `revision`, goal, catalog ID/version/digest, context inputs, nodes and edges.

Each node has a unique ID, exact tool ID/version, parameters and bindings for every required input port. Context bindings resolve declared references. Output bindings name an existing producer node/output and require an exact type match. Each output binding requires a direct producer-to-consumer dependency edge in v0.1. The graph must be acyclic. Additional control edges are allowed.

Plans contain no script, command, endpoint, arbitrary executor or permission declaration. Context input type labels are untrusted claims and must be checked against server-resolved data.

Example: [`examples/valid-plan.json`](../examples/valid-plan.json).

## 4. Run record
Record request/run IDs, actor, immutable plan snapshot/revision/digest, catalog digest, context revision, policy/ruleset versions, planner mode/versions, adapter versions, status, ordered events, per-node attempts, input/output references, errors and artifact references. Record rejection even if execution never began. Do not claim tamper resistance or bitwise reproducibility merely because a ledger exists.

Example: [`examples/run-record.json`](../examples/run-record.json) is a hypothetical fixture, not an executed run.

## Auxiliary values
`SeriesMetadataSnapshot` records the exact observed manifest, time/scope and per-instance properties. A property has `known` plus a value or `unknown` plus a reason. Do not infer archive completeness from `InstanceNumber`. A retrieval failure for a whole manifest instance fails this initial operation; an unavailable individual property can yield `unknown` and an `indeterminate` assessment.

`CompatibilityAssessment` records reference/snapshot/profile versions, per-rule/per-instance outcomes and limitations. `ReportArtifactRef` uses a server-managed storage key, not a destination URL supplied by a plan.

## Plan digest
The fixture convention is SHA-256 over UTF-8 JSON with sorted keys, compact separators, `ensure_ascii=false`, and no NaN/Infinity. This is a local proposal, not RFC 8785 canonical JSON. Restrict fixture values and select a cross-language canonicalization before Java/production use. Approval should bind the exact digest, user, references, effects, destination, expiry and current policy; approval must be stored in a trusted server-side source, not asserted by the plan.

## Formal model
Let `t=(id, version, I, O, Θ, Pre, Eff)` describe a tool, and `P=(V,E,B)` a plan. Let `K` be the catalog, `Π` the policy, `C` the context and `u` the user.

`StructuralValid(P,K) = DocumentShape ∧ CatalogMatch ∧ Acyclic ∧ CompleteBindings ∧ TypeCompatible ∧ ParametersValid`.

`Admissible(P,C,u,K,Π) = StructuralValid ∧ CurrentPlanningConditionsSatisfied ∧ FutureConditionsRegistered ∧ EffectsAllowed`.

`StepExecutable(v,Cnow,u,Π) = InputsAvailable ∧ RuntimeConditionsSatisfied ∧ AccessAllowed ∧ RequiredApprovalsSatisfied`.

These expressions define obligations; each predicate must be implemented and tested. `unknown` never silently counts as true for authorization or a current precondition. Future conditions may be deferred only if the contract identifies a before-step check and a trusted source for it. A valid graph does not guarantee that an upstream tool will produce its declared output.
