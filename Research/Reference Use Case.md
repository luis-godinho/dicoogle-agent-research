---
status: draft
tags: [dicoogle-agent]
---

[[Home]] · [[Scope and Research Question]] · [[Contracts v0.1]] · [[Rules and Tests]]

# Reference Use Case — UC-01

## Assess metadata compatibility of a DICOM series

**Researcher request:** “For this study, assess the selected CT series against the approved preview metadata profile and provide a report. Do not run the preview or export images.”

The profile is illustrative; it is not a claim that a production preview tool or its requirements have already been selected.

## Input and target
The request contains an objective, a permitted study identifier, and authenticated user identity. The context service resolves a concrete series and records the observed instance manifest for the run. Multiple candidates require an explicit choice or criterion. The implementation must not claim that its returned manifest is the complete archive series unless that is independently established.

## Context versus pipeline
**Context resolution:** resolve study/series, check identity/access, obtain permitted metadata and a stable instance manifest.

**Pipeline:** `inspect_series_metadata` → `assess_metadata_compatibility` → `generate_compatibility_report`.

Compatibility is determined by the assessment step, not assumed during target selection.

## Example profile
`preview-metadata-profile@0.1` requires each instance in the recorded manifest to have `Modality = CT`, an accepted CT Image Storage SOP Class, and positive integer `Rows` and `Columns`. The symbolic SOP Class label must be mapped to the correct UID in a reviewed adapter before real-data use. These are experimental profile rules, not universal preview requirements.

## Expected outcomes
- `compatible`: non-empty manifest; all required observations known and every rule satisfied, within `metadata_only` scope.
- `incompatible`: at least one known rule violation.
- `indeterminate`: no known violation, but at least one required value is unknown or manifest is empty.
- `failed`: operational inability to retrieve a required manifest instance, invalid adapter output, or other execution error.

An `incompatible` assessment is a successful completion of the assessment task, not an executor failure.

## Main flow
1. Resolve and authorize the study and selected series.
2. Clarify if the target is ambiguous.
3. Build a manual candidate plan from the approved catalog.
4. Validate structure, exact versions, bindings, parameters, current preconditions, effects, and policy.
5. Before execution, re-check volatile conditions and freeze the instance manifest.
6. Inspect metadata for that manifest; represent unavailable individual properties as `unknown` with reasons.
7. Evaluate every profile rule per instance.
8. Produce an internal report and run record; return an authorized reference.

## Failure and limitations
If access is denied, stop and report it. If a whole manifest instance cannot be retrieved, mark the operation failed; do not claim a global compatibility result. Do not infer series completeness from `InstanceNumber`. No pixel decoding, preview generation, clinical interpretation, or clinical suitability is assessed.

## Effects and approval
The request reads archive metadata and writes an internal report/ledger. “Read-only with respect to the archive” does not mean “no side effects.” A proposed test policy permits these internal writes for authorized users without an additional confirmation, while external export is prohibited; confirm this policy institutionally.

## Synthetic example
See `examples/` and `fixtures/`. All identifiers and observations in them are fabricated test data.
