# Finding-report experiment: decisions 2026-11-21

Status: agreed direction; implementation, smoke-test completion and live inference are not established by this record.

Related: [Earlier LIDC experiment design](lidc-sr-llm-report-experiment-design.md). This decision record supersedes conflicting scope and next-step proposals in that document; historical evidence remains unchanged.

## 1. Main research objective

Evaluate whether a general-purpose multimodal LLM identifies and faithfully describes different finding types in an explicitly supplied image set without visible reference annotations. A future planner may use the report to select specialized models, but planner performance is outside this phase.

The main experiment is not benign/malignant classification, an overlay ablation, or nodule-only characterization. Finding recognition and descriptive fidelity are primary capabilities; spatial grounding is additional where references support it. Multiple findings within one modality/anatomical region are sufficient for the initial experiment.

## 2. Separate general protocol and dataset adapters

The general output supports findings, finding type, location evidence, descriptive attributes, limitations/abstention and optional supported measurements. It must not require LIDC codes, nine-panel montages, lung nodules or DICOM SR.

Dataset adapters supply reference mappings, provenance, spatial transforms and scoreability limits. Separate model-reportable fields from automatically scoreable fields and fields needing manual review. Unsupported references are not model errors. For physical measurements, the model also needs an appropriate scale representation or a defined coordinate-to-physical-measurement mechanism; evaluator access to spacing alone is insufficient.

Match predictions to supported lesion/groups, not independently to each annotation. Multiple annotations or repeated panel appearances must not multiply the expected finding count. Compare attributes against individual annotations within the matched group and retain ambiguity, omissions, duplicates and unmatched predictions.

## 3. Two parallel tracks

### A. Permissions-first dataset search

Search for a practical dataset with multiple finding types, reliable expert references, feasible small-subset retrieval and workable external-processing terms. CT is preferred for continuity, not mandatory; masks and SR are not mandatory. Assess actual licenses and additional terms, not public availability or license labels alone. Report scope/completeness and distinguish report silence from verified absence. Return at most three candidates, a preferred option and fallback; no downloads or access applications are authorized by this decision.

### B. Offline LIDC implementation and smoke test

Use the existing two LIDC patients and acquired SR/SEG references for technical testing, not main-experiment validation. Implement minimal additive reusable schema validation, matching, attribute comparison, accounting and readable output, with a separate LIDC adapter. Preserve existing classification-task semantics and unrelated worktree changes.

Exercise the full comparison path using explicitly synthetic responses and known-answer spatial fixtures, plus real-reference loading checks. Include matched findings, wrong attributes, omissions, extra predictions, duplicates, repeated-panel findings, ambiguity, abstention, unresolved mappings, malformed output and technical failures. Assert expected outcomes.

Reference-derived fixtures are never model-generated results. Old benign/malignant responses cannot substitute for the new structured outputs. Leave unresolved vocabulary comparisons unscored; do not block all software testing on them. No diagnostic accuracy, specificity or inferential performance claims from two patients.

Run offline with GEMINI_API_KEY removed from command environments. No network, inference, downloads, installations or historical-evidence overwrites. Implementation must be inspected and tested before live calls. Prepare a live-run preview with prompt/schema, image hashes, model configuration, limits and supported cost estimates; at most one image-only call per existing patient, zero retries and no fallback. Stop for explicit review before inference. Dataset/provider compatibility remains a separate prerequisite.

## 4. ReXGroundingCT: deferred alternative

Retain ReXGroundingCT/CT-RATE as a possible future option, not a dependency. Strengths include multiple lung/pleural finding types, finding text and spatial masks. Risks include gated access, confidentiality/non-redistribution terms with external-API processing unresolved, report/model-derived text, restricted annotation scope and 3D-to-2D visibility limitations.

Exhaustive entities for included report-derived findings do not establish independent exhaustive discovery of every abnormality. Unmatched predictions are not automatically false positives. Public validation case counts require patient/scan/reconstruction deduplication. Do not spend further implementation effort on this candidate while searching for a better-permitted alternative.

Official sources for future verification:
- https://huggingface.co/datasets/rajpurkarlab/ReXGroundingCT
- https://huggingface.co/datasets/ibrahimhamamci/CT-RATE
- ReXGroundingCT paper: arXiv:2507.22030; DOI 10.1056/AIdbp2501220.

## 5. Provider route and permission boundary

The local agent reported direct Gemini Developer API access through its OpenAI-compatible endpoint, API-key authentication via GEMINI_API_KEY, harness alias gemini_pilot and persisted provider model gemini-3.5-flash-lite. This is an audit report, not independent confirmation of current availability or contractual protections.

Account tier, retention, training-use, residency and contractual settings were not established locally. No secrets belong in this repository. Zero configured pricing is not provider-reported zero cost. Provider data-use provisions may depend on region/account; do not assume all free-tier processing has identical terms. Check current official terms: https://ai.google.dev/gemini-api/terms . Provider protections do not override dataset restrictions. Prior API use is not blanket permission for future transmission.

## 6. Immediate execution boundary

Proceed with the separately issued offline implementation prompt and public dataset-research prompt. Do not create further generic specifications instead of testing software. Preserve all original run/results and acquired references. Agent prompts remain in English. This repository documentation update does not execute code, download data, send communications or authorize live inference.
