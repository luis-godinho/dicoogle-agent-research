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

---

## 7. Status update — 2026-11-22

Sections 1–6 are preserved as the historical 2026-11-21 decision record. This dated update supersedes their execution status and immediate-next-step instructions where applicable, without overwriting historical evidence. Results below are transcribed from local-agent reports supplied in the research conversation; this documentation update does not independently inspect local artifacts.

Scope label: personal technical pilot; strict-output protocol development; annotation-assisted slice selection; not independent detection or clinical validation. Future specialist routing motivates ROI analysis but specialist/planner performance has not been tested.

### 7.1 Strict protocol and development cases

- Implemented an additive strict provider protocol plus approved null-placeholder conversion followed by unchanged canonical validation. Focused offline tests reported 76 passing. Historical configurations remain separate.
- Provider schema: `structured_findings_provider_strict_centerpoint_v1`. SHA-256: `276252cee8c4dcd4787be71e9c0ac275f6906c8fffa7455105955bcf40e1fd56`.
- Conversion: `strict_centerpoint_null_placeholder_to_canonical_v1`. SHA-256: `ddc1b70d4cf386711ac8312c9945b2761ecea075e847799e659c2c3367624414`.
- Prompt SHA-256: `8ef1d825b9faf861ab83df5a9c77f0ecd0309d4717bdeca4f4f6b04ee649e2d7`.
- Known semantic gap: canonical validation does not require non-empty explanations for abstention or unlocalizable findings. No new rule was added.
- LIDC-IDRI-0072 and LIDC-IDRI-0149 are development patients, excluded from the subsequent batch. Preserve all their attempts separately; do not pool different configurations into an accuracy headline. In particular, 0072 attempt4 returned one unmatched prediction; do not conflate that attempt with any earlier positive development run.
- The strict 0149 request used `openai/gpt-4.1-mini`, pinned OpenAI provider, and passed provider/canonical validation. Two predictions matched none of seven in-scope reference groups; no Calcification comparison was eligible.
- An initial 0149 audit was invalid because its script treated a 512×512 panel as the 1536×1536 montage. It remains preserved as invalid. The corrected audit found 9/9 exact panel/source matches and no geometry/linkage inconsistency. This was an audit-script defect, not an established evaluator/input defect.

### 7.2 Ten-patient batch preparation

Selection: ascending SHA-256 of `lidc_finding_eval_v1:` concatenated with patient ID, tie-break patient ID. From 140 metadata-level eligible candidates after excluding development patients, take the first ten passing pre-inference checks; use predeclared replacements in order. Diagnosis was not a selection criterion.

Final ordered patients: LIDC-IDRI-0242, 0068, 0194, 0875, 0568, 0185, 0624, 1004, 1011, 0493.

Excluded for fewer than nine mask-positive slices: 0090 (3), 0207 (8), 0257 (6), 0248 (2). Candidate 0171 was not reached. No padding or manual slice substitution was used.

- CT provenance: original LIDC-IDRI series. SEG/SR provenance: `dicom_lidc_idri_nodules`; QIN-LungCT-Seg is not included.
- Input method: `image_only_mask_centred`, nine evenly spaced mask-positive slices ordered by z, window centre −600/width 1500, nine separately labelled 512×512 panels, no reference overlay or header text. Slice selection is annotation-assisted.
- All 90 panels passed montage/source pixel-identity checks. All 1,428 SR attribute rows had verified SR→SEG links; 2,731 retained files had unique SOP Instance UIDs.
- References: 40 groups total, 37 in scope, 3 out of scope; 119 annotations, 115 in in-scope groups; 275 annotation-panel regions.
- Out of scope: 0068/Nodule 4, 0624/Nodule 7, 1011/Nodule 4. Do not count these as omissions.
- No negative-case reference exists; specificity and negative agreement are not estimable.
- Acquisition deviation: total transferred data was 1.694 GB, exceeding the 1.5 GB gate including replacements; retained data was 1.363 GB. The deviation was acknowledged and must not be hidden by reporting retained size alone.
- 0493 spacing warning: 1.24998–1.25 mm; valid panels were produced without manual adjustment.

Local-agent provenance review reported CC BY 3.0. Retain required attribution and verify terms for any new use:
- Armato et al. (2015), Data From LIDC-IDRI, DOI 10.7937/K9/TCIA.2015.LO9QL9SX.
- Fedorov et al. (2018), Standardized representation of the TCIA LIDC-IDRI annotations using DICOM, DOI 10.7937/TCIA.2018.H7UMFURQ.
- IDC acknowledgement: Fedorov et al., RadioGraphics 2023, DOI 10.1148/rg.230180.

### 7.3 Completed batch: execution and historical results

One request per patient, ten total; zero retries, fallback, substitutions or replacements. Model `openai/gpt-4.1-mini`, pinned OpenAI provider, temperature 0.0, max_tokens 2048, timeout 60 seconds. All ten returned HTTP 200, finish reason `stop`, valid provider/canonical outputs and completed comparisons. Internal persisted `seed=0` was omitted from the outbound body; do not describe seed as transmitted.

| Measure | Result |
|---|---:|
| Attempted / valid patients | 10 / 10 |
| Patients with a matched group | 1 / 10 |
| Predictions | 10 |
| Matched predictions / groups | 1 / 1 |
| In-scope groups unmatched (historical omissions) | 36 / 37 |
| Unmatched predictions | 9 |
| Duplicates / ambiguity / spatially unverified | 0 |
| Calcification comparisons / agreements | 4 / 4 |
| Reported total cost | $0.0212984 |
| Input / output tokens | 44,958 / 2,072 |
| Total reported request latency | 44,461.2 ms |

Only 1004/Nodule 3 matched, with four annotation-level Calcification agreements, all Absent. These are four annotations of one reference group, not four independent nodule or patient successes. The three Central calcification references (RID5827) in 0185 remain unmapped and were not eligible because there was no spatial match. Mapping remains Absent→absent and Solid appearance→solid_appearance.

Historical spatial matching means bounding-box containment at `tolerance_px=0.0`, NOT SEG-mask containment. Preserve results exactly. Unmatched predictions are not confirmed clinical false positives; omitted groups are not confirmed clinical misses.

### 7.4 Completed offline diagnostic review

No input, reference, conversion or geometry defect was found across the ten patients. Every patient returned exactly one finding in raw and canonical output; conversion removed none. Each finding reused one coordinate across its reported panels. All labels were Pulmonary nodule; all Calcification values were Absent; no explicit uncertainty was returned.

There were 41 reported panel points: two fell inside reference bboxes, both in 1004; none fell inside any SEG mask. Nearest-mask distances across points: minimum 1.0 px, median 70.661 px, maximum 158.231 px. The historical positive match remains bbox-admissible despite no point-in-mask containment. Separate annotation contours are not consensus masks. Clinical correctness/visibility remains unresolved.

Prompt observation: “Report one finding once, with multiple panel references where appropriate” is ambiguous. It may encourage one finding, but causation is unproven. Provider findings-array schema has no maximum; canonical maximum is 50. Neither requires exactly one finding.

### 7.5 Completed exploratory ROI analysis

Purpose: test bounded candidate-region handoff, distinct from precise localization and from actual specialist success. Existing points/panels were unchanged. Square crops used side lengths 10, 20, 40 and 80 mm, converted using actual CT spacing; no crops clipped. These are search-window sizes, not estimated lesion diameters or validated specialist input requirements. No crops were added to unreported panels.

An initial implementation computed full containment but omitted required intersection counts/fractions. Its legacy `groups_zero_coverage` meant no fully contained region, not zero intersection. A versioned completion analysis calculated the missing fractions and reconciled full-containment counts. Use the completed outputs for future comparisons.

Group categories are mutually exclusive: A=no intersection; B=some intersection but no full region; C=some full regions but not all; D=all submitted annotation-panel regions full. Denominator remains all 37 in-scope groups.

| Crop side | A | B | C | D | Any intersection |
|---|---:|---:|---:|---:|---:|
| 10 mm | 36 | 1 | 0 | 0 | 1/37 |
| 20 mm | 36 | 1 | 0 | 0 | 1/37 |
| 40 mm | 35 | 1 | 0 | 1 | 2/37 |
| 80 mm | 30 | 6 | 0 | 1 | 7/37 |

| Crop side | Zero / partial / full annotation-panel regions |
|---|---|
| 10 mm | 259 / 16 / 0 |
| 20 mm | 259 / 16 / 0 |
| 40 mm | 258 / 1 / 16 |
| 80 mm | 208 / 51 / 16 |

Only 1004/Nodule 3 had full containment, with 16/16 submitted regions at 40 and 80 mm. Its 10 mm partial fractions had median 0.092; 20 mm median 0.565. At 40 mm the additional partial region was 0194/Nodule 1 with fraction 0.0047. At 80 mm partial fractions reached 0.992 for one 1011/Nodule 1 region. Tiny intersection is not equivalent to near-complete coverage. Submitted 2D region coverage is not complete 3D-nodule coverage.

Conclusion: larger crops reached additional groups partially but did not add fully contained groups. Keep ROI endpoints; do not claim either universal ROI failure or demonstrated specialist usability. No winning crop size was selected and no new Calcification comparisons were created.

### 7.6 Current decisions and next action

1. Preserve original outputs, audits, scores and protocol identities. “Frozen” means unchanged for that historical experiment, not prohibited from improvement in a new version.
2. Keep precise localization and ROI handoff measures separate. More candidates may improve coverage but candidate count, duplicates and crop area are part of the burden; simply producing more findings is not success.
3. Next: prepare an OFFLINE, separately versioned, single-change prompt-comparison plan. No new inference has been reported for this comparison, and this note does not authorize it.
4. Proposed replacement for the ambiguous sentence:

> Report all distinct visible findings supported by the images. The findings array may contain zero, one, or multiple entries. Create a separate entry for each distinct finding. When the same finding appears in multiple panels, include it once with its corresponding panel references. Do not invent additional findings to reach a particular count.

5. Return exact full prompt diff/hash and schema compatibility. Keep model, provider, images, schema, conversion, matching and fixed ROI sizes unchanged. Do not add model-predicted radii yet. Keep output-token limit unchanged and report truncation rather than silently raising it or retrying.
6. Proposed reuse of these ten patients is a paired DEVELOPMENT comparison on an inspected batch, not an untouched evaluation. Additional unseen patients would be needed for a later evaluation not used to choose the prompt.
7. Compare validity, truncation, findings per patient, matched groups, unmatched/duplicate predictions, completed fractional ROI coverage, full containment, crop count/area, tokens, latency and cost. Include zero-coverage regions; report all crop sizes, not only the best.
8. Prepare request count, budget and isolated destinations; stop for explicit approval before provider calls or remote image transmission. No automatic second-model run, downloads or production-evaluator changes.

### 7.7 Local evidence index

Local artifacts are not uploaded by this documentation update. Relative identifiers for finding the preserved reports in the local pilot workspace:

- `audit_0149_strict_centerpoint_corrected_2026-11-22/`: corrected development-case audit.
- `eval_batch_2026-11-22/batch_manifest.json` and `README.md`: final batch preparation.
- Batch `artifacts/live_strict_centerpoint/<case_id>/`: per-case raw responses and validation/comparison records.
- Batch `artifacts/live_batch_report_2026-11-22/`: attempt ledger and aggregate historical results.
- Batch `artifacts/diagnostic_review_2026-11-22/`: geometry, cardinality, point containment and overlays.
- Batch `artifacts/roi_coverage_review_2026-11-22/`: original full-containment analysis and wording correction.
- Batch `artifacts/roi_coverage_completion_2026-11-22/`: completed intersections/fractions, reconciliation and coverage plot; authoritative source for fractional ROI summaries.
