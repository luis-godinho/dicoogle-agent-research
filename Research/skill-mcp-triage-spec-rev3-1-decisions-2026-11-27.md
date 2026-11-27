# Skill/MCP triage — rev3.1 specification decisions

Date: 2026-11-27. Status: planning/specification only; implementation, inference and data transmission remain unauthorized.

Read with the three 2026-11-25 records ([workflow](skill-mcp-triage-decisions-2026-11-25.md), [report contract](skill-mcp-triage-report-decisions-2026-11-25.md), [concluded plan](skill-mcp-triage-concluded-plan-2026-11-25.md)). This record adds decisions made after the specification drafts were reviewed. Where it conflicts with those records on the topics below, this record wins. Historical LIDC pilot protocols, schemas, raw responses and scores are unchanged.

## 1. Where the specification lives

The working specification is `SPEC_rev3_1.md`, kept locally and untracked in `drafts/triage_spec_2026-11-27/` of the pilot repository (HEAD 2ecd777 at the time). It is not committed here. Earlier revisions (`SPEC.md`, `SPEC_rev1.md`, `SPEC_rev2.md`, `SPEC_rev3.md`) are preserved unchanged.

Recorded local SHA-256 hashes:

| File | SHA-256 |
|---|---|
| SPEC_rev3.md | 2e8168a45c0d055ac6967a26dc0e966393ded8f8215a43fb0413cc240d77eb2a |
| SPEC_rev3_1.md | 17982bdc60a53e37ebaf04b2a0e84bef085148299f8fc321d52dfa5553c7ec60 |
| FEASIBILITY.md (current) | 6a07d8506fab0332b38761e456db67a00afb6e06e91a53ce24a16d40f902f390 |
| FEASIBILITY_rev2.md | 621b890dd50879356c67a4a5bbfd02d72ca02b329f74f3dffa6ad7728b4df190 |

The 18,379-byte first-turn FEASIBILITY.md was replaced before the rev2 task by an edit whose author is not provable from the session. FEASIBILITY_rev2.md is the version to use.

## 2. Decisions agreed 2026-11-27 (first round)

- Coordinates: integer inclusive pixel indices, image-local. Fractional, non-finite or non-numeric values are invalid and never rounded or clamped.
- Matching: any positive-area intersection defines a spatial candidate for one-to-one assignment only. It does not define "identified". Identification is reported at several pre-declared coverage levels, always alongside search-area burden. No single threshold is called success.
- `not_assessable`: at study level, empty findings plus a required explanation. At finding level, an abstention entry kept in its own view, never counted as a negative or a detection.
- Vocabulary: broad category and broad anatomy for v0. `unknown` is reported separately and unscored. Actual vocabularies are deferred until datasets are chosen.
- Unmatched predictions are reported as unmatched, never as confirmed false positives.
- Required metadata: only modality, exported dimensions and panel order. Everything else is optional, and `unknown` is valid for optional fields. Missing required metadata is a hard validation error.
- Corrections and resume: at most three submissions per study. One explicit resume action per interruption, with input-manifest, image, study-ID and output-hash verification before skipping a completed study. Format-only versus content-changing revisions are decided by canonical semantic comparison.
- Output: multi-file layout with raw responses preserved. Structural and spatial errors are hard errors. Style issues are warnings that never trigger corrections. Parsed copies are faithful parses, never repairs.

## 3. Decisions agreed 2026-11-27 (second round)

- Assignment: maximum-cardinality one-to-one, then total coverage, then deterministic tie-break by finding_id and group_id. Report ambiguity counts and stability across levels. A candidate assignment (A0) is used for audit and localization pairing, and a separate assignment is computed at each coverage level.
- Coverage levels, frozen before any results: any positive overlap, 0.25, 0.50, 0.75, 0.90. They are held researcher-side only and are never part of model-facing input. Changing them after results are seen invalidates the comparison.
- Multiple readers: score each annotation and report per-annotation and best-match results. No consensus is implied.
- Minimum box size: one pixel is valid. Tiny boxes are flagged in burden metrics, and the full box-area distribution is reported.
- Optional metadata: relative slice position for volumetric studies only. Pixel spacing is `unknown` after resampling. Laterality is included only when the source metadata supports it.
- Timestamps and client metadata are optional and null when unavailable. The contract version appears in every structured output file; raw and parsed payloads are not modified.
- Integrity stops are not resumable. They require investigation and an explicit new restart.
- A negative outcome is only an assessable report with zero findings. A report containing only suspected or abstained findings is never counted as negative.

## 4. Dicoogle is out of scope for this experiment

Metadata is prepared by the researcher from local DICOM headers. Nothing in the experiment depends on Dicoogle indexing, routing or its query API. Dicoogle compatibility with the allowlist is future work and is not evaluated here. Rev3.1 removed the Dicoogle API exposure from the open items.

## 5. Dataset direction (not selection)

- Aim for about five datasets across different modalities and body regions. No dataset is selected.
- Prefer datasets with masks or bounding boxes, which support coverage-level localization. At most one point-only or image-level-label dataset may be included, with its results reported separately; point-only references support spatial candidacy only.
- Each dataset needs negative cases supported by its own reference. Negative must be defined per dataset.
- Licence and permission for external processing are a hard screening filter for every dataset, assessed individually.
- Whether LIDC counts among the five or remains separate development material is undecided. LIDC-IDRI is distributed on TCIA under CC BY 3.0 with attribution requirements; this record does not establish permission for external model-service processing, which still needs the researcher's and institution's confirmation.

## 6. Still open

1. Actual vocabularies (depend on datasets).
2. Datasets, including the LIDC question.
3. Sample size.
4. Model shortlist (professors).
5. Client capabilities: Codex desktop on macOS and Claude Code on Linux, none verified.

A synthetic live-test kit exists locally under `live_test_kit/` (no patient or dataset data, no MCP server, skill, client configuration or credentials). No test has been run. Live tests need the professors' machines.

## 7. Authorization boundary

This record authorizes documentation only. It does not authorize implementation, installations, configuration changes, dataset downloads, server startup, client login, transmission of images or metadata, inference or Dicoogle work. Do not alter historical raw responses, matching, schemas or scores.
