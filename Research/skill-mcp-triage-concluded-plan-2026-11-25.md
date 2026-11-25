# Skill/MCP triage — concluded high-level experiment plan

Date: 2026-11-25. Decisions through 15:33 WET.
Status: high-level planning concluded; implementation and inference remain unauthorized. This is not a frozen executable protocol or evidence of client feasibility.

## Relationship to earlier records

Read with [the workflow checkpoint](skill-mcp-triage-decisions-2026-11-25.md) and [the report-contract checkpoint](skill-mcp-triage-report-decisions-2026-11-25.md). This document consolidates the plan and records the subsequently agreed decisions. It supersedes conflicting open-question statements on the topics resolved below. Existing documents, historical schemas, pilot responses and scoring remain unchanged.

## 1. Purpose and execution boundary

Evaluate a general-purpose model's triage-stage finding identification and coarse localization on supplied medical images. Downstream specialist processing and future Dicoogle routing are background motivation, not evaluated outcomes.

The researcher prepares approved inference-only inputs locally and retains reference data locally. Professors use supported subscription clients; no direct model API backend or extraction/persistence of subscription credentials is authorized. Both configurations receive identical prepared images, ordering, instructions, metadata and correction policy. Model choices remain with professors.

The intended workflow is one skill invocation, sequential study processing and a fresh interpretation context per study. Codex desktop on macOS is the confirmed target; Claude Code on Linux remains subject to confirmation. These client capabilities have not been demonstrated. No extra slices, case-related web searches, external interpretation models or agent-created crops/zooms in the initial protocol.

## 2. Input selection: positive and separate negative sets

- Positive inputs use selected slices containing reference SEG annotations. Only prepared images and approved metadata reach the model, never SEG/SR annotations or expected findings.
- Include separate negative input sets. No dedicated mixed positive/negative panel condition initially.
- Dataset selection must establish what negative means and what the reference supports. Absence of a segmented target alone does not establish absence of every abnormality.
- Annotation-assisted selection must be disclosed. This is not independent detection across a complete study.
- Record whether selected slices or a full study were actually supplied. An empty report refers only to the supplied images, not an unseen complete study.
- Exact sampling rules, positive/negative proportions, sample size and datasets are not yet selected.

## 3. Reporting scope and structured output

Report all distinct abnormalities supported by the supplied images, including incidental findings, rather than limiting reporting to a dataset's annotated categories. Multiple findings and an empty findings list are permitted.

Retain the agreed common structured core: finding category, anatomy, visible characteristics, reported certainty, limitations and localization. Use versioned controlled vocabularies, with other plus description and unknown where appropriate. Exact vocabularies and attribute sets remain open; do not require dataset-specific attributes universally.

Reported finding assessments remain present, suspected and not_assessable, with precise schema semantics still to be specified. These are model reports, not calibrated probabilities. Study/input-set assessability is separate: assessable, partially assessable or not assessable, with explanations for limitations.

## 4. Localization, including partial localization

- Each distinct finding may reference multiple supplied images. Grouping is model-proposed, not verified anatomical tracking.
- On each localized image, the model supplies a location point and an axis-aligned rectangle. The point must lie inside the rectangle but need not be its centre. Neither is automatically derived from the other.
- Coordinates are local to each supplied/exported image: top-left origin, x rightward, y downward, pixel units; point (x, y), box (x_min, y_min, x_max, y_max). Preparation software retains source mappings.
- A finding may be partially localized: retain geometry on images where available and an explicit unlocalizable status with explanation on other referenced images. Do not invent geometry, silently drop limitations or count missing geometry as successful localization.
- A wholly unlocalizable finding is also permitted and retained without fabricated coordinates.
- Reported certainty, cross-panel identity uncertainty and localization ability are distinct. Do not force cross-panel grouping when the model cannot support it.
- Rectangles are proposed search regions, not exact segmentations.
- Pixel-centre/edge semantics, boundary inclusivity, precision, bounds, transforms and exact mixed-localization schema rules remain to be specified.

## 5. Evaluation scope and three separate results

Keep three main results separate; no composite score or single primary outcome has been selected:

1. Finding identification: identification of reference-supported abnormalities, including multiple findings.
2. Localization quality: reference-region coverage and proposed search-area size for eligible correspondences. Exact metrics, matching rules, thresholds and aggregation remain open.
3. Negative-input performance: empty reports on reference-supported negative input sets and characterization of findings reported on those inputs.

Score only categories and attributes supported by suitable references. Retain findings outside annotation scope as unscored rather than automatically calling them false positives. Reference incompleteness must remain explicit.

Keep spatial correspondence, category/anatomy correctness, region quality and attribute agreement distinct. Attribute comparison requires eligible correspondence and suitable reference support. Preserve individual annotations and reference-group identity rather than inventing consensus.

Retain present-only and present-plus-suspected views, certainty-stratified results, abstention, partial/unlocalizable findings and technical failures as separate breakdowns. Do not silently upgrade suspected findings or remove reference omissions from denominators because the model abstained. Exact denominator and partially assessable-case rules still require pre-evaluation specification.

## 6. Metadata and fair input delivery

Use a minimal versioned acquisition/geometry metadata allowlist, not complete DICOM headers. Earlier agreements cover modality, broad anatomy, exported orientation/view, examination/image laterality, exported dimensions, valid physical spacing and relative slice order/position where applicable. Missing information remains unknown; it must not be filled from annotations.

Exclude identifiers, reports, diagnoses, expected finding types/counts, reference measurements, SR/SEG content and unreviewed free text/private tags. Additional clinical or modality-specific metadata requires separate agreement. Exact fields, export transformations, de-identification and client image handling require verification.

## 7. Accepted failure and resume policy

- If a study's images cannot be loaded or its input package fails validation, record a technical failure and reason, preserve evidence and continue to the next study. Do not interpret an incomplete input set.
- Allow the initial report plus at most two technical correction attempts, preserving every submission and distinguishing format-only from content-changing revisions. No reference-based feedback. If still invalid, record a technical failure and continue.
- Explicit model inability to assess otherwise valid inputs is an assessment outcome, not a technical failure.
- Authentication, connection or shared-service failures pause the batch and preserve completed work.
- Study/image identity mismatches, reference-data exposure or result-overwrite risks stop the batch for investigation.
- Resume is explicit and skips completed studies. Failed or interrupted studies are retried only when explicitly requested; preserve earlier attempts and never silently replace results.

These are accepted requirements, not implemented or tested behavior.

## 8. Remaining work, not an indefinite planning questionnaire

High-level decision rounds are concluded. Next produce a reviewable specification through:

1. Dataset/reference suitability: labels, negative-case support, spatial references, annotation scope, permissions and external-processing restrictions.
2. Client feasibility: fresh contexts, multi-image delivery, sequential execution, technical correction and resume behavior in the actual accounts/clients.
3. Protocol draft: exact schema, vocabularies, coordinate rules, packaging, validation, assignment, metrics, thresholds, denominators and aggregation proposed together.
4. Pre-experiment review: settle consequential tradeoffs, select sample and model configurations, and freeze the protocol before evaluation.

No further general planning questionnaire is needed now. Revisit only consequential unresolved choices or constraints discovered during verification.

## 9. Authorization and historical safeguards

This record authorizes documentation only. It does not authorize implementation, installations, configuration changes, dataset downloads, server startup, client login, data transmission, inference or downstream Dicoogle/specialist work. Permissions and actual client capability remain prerequisites.

Do not alter historical raw responses, matching, schemas or scores; do not retry cases to force matches. No clinical-validation claim is established by the personal LIDC development pilot or by this plan.
