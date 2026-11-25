# Skill/MCP triage — report-contract decisions, continuation

Date: 2026-11-25. Checkpoint: through the researcher's 15:18 WET agreement. Status: planning only; implementation is not authorized.

Read together with [the original skill/MCP planning checkpoint](skill-mcp-triage-decisions-2026-11-25.md), which remains the record of the overall workflow. This continuation records all subsequent decisions and supersedes conflicting open-question/status statements in that checkpoint. It does not alter historical protocols, results or schemas.

## 1. Overall decisions carried forward

- The researcher prepares the approved images locally and sends professors an inference-only directory. References, including SR/SEG, remain on the researcher's machine.
- The professors use supported subscription clients; the proposed backend does not make direct model API calls or extract subscription credentials.
- Confirmed target: Codex desktop on macOS. Claude Code on Linux remains likely but requires confirmation. Exact versions/account capabilities remain unknown.
- One skill invocation should start the batch, sequentially, with a fresh interpretation context for each study. Client feasibility is not yet demonstrated.
- All approved images for a study are supplied together, separately labelled and ordered. No extra slices, case-related web searching, external interpretation models or agent-created zooms/crops in the initial protocol.
- Save progress, preserve completed cases and permit explicit resume after interruption without silent reruns.
- Allow an initial report plus at most two technical correction attempts. Preserve every submission and distinguish format-only from content-changing revisions. Never give reference-based feedback.
- Professors return traceable results; the researcher performs evaluation locally.
- Both configurations receive identical images, ordering, instructions, metadata and correction policy. Final model choices belong to professors; a verified shortlist remains pending.
- LIDC is development material. The main experiment is intended to span different modalities and body regions, with datasets still to be selected and permissions established.
- Future Dicoogle indexing-triggered specialist routing/annotation motivates the work but is not implemented or evaluated in this experiment.

The original checkpoint contains the fuller rationale, packaging discussion, failure policies and feasibility questions. None of these carried-forward requirements has been demonstrated by an executed skill/MCP implementation.

## 2. Resolved: rectangular search region plus a location point

The researcher selected a rectangle, not a point-plus-radius circle. The model proposes BOTH:
- a point indicating its best estimate of the abnormality's location;
- an axis-aligned rectangular bounding box indicating the suggested search area.

The point must be inside its rectangle but need not be the rectangle's geometric centre. Both are model outputs; do not automatically derive the point from the box centre. Point-in-box consistency is technical validation, not medical verification.

The rectangle is a proposed search area, not an exact segmentation or verified lesion boundary. It may include contextual tissue. Avoid calling full search-region containment complete lesion segmentation.

This resolves the unanswered rectangle-versus-radius question in section 16 of the original checkpoint. It supersedes the earlier centre-point-only decision for the future experiment only. Existing LIDC API results and fixed-window ROI analyses remain unchanged. A new versioned contract is still required before implementation.

## 3. Resolved: multiple findings and multiple supplied images

- One entry per distinct suspected abnormality.
- Multiple findings are allowed; zero findings is also valid.
- The same finding may reference multiple supplied images.
- Provide a separate point and rectangle on each image where the model identifies it.
- Do not copy coordinates automatically across panels or infer presence on intervening panels.
- Preserve the finding-level grouping rather than counting each panel appearance as an independent finding.

Exact cross-panel identity/ambiguity handling remains to be specified. The protocol must not imply verified anatomical tracking merely because the model groups panels together.

## 4. Resolved: unlocalizable findings are permitted

A finding may be reported without invented geometry if the model cannot localize it reliably. It requires an explicit unlocalizable status and explanation.

Retain such findings in the results, but do not count them as successful spatial localization or usable proposed search regions. A description, uncertainty about finding identity and inability to localize are separate dimensions.

Mixed cases, such as geometry available on only some referenced images, still need precise schema rules; do not silently settle these during implementation.

## 5. Common descriptive core and standardization

The researcher accepted a common core:
- finding category/short label;
- anatomical location, as specific as supported;
- visible characteristics;
- uncertainty/limitations where relevant;
- localization or explicit inability to localize.

The researcher emphasized measurability and machine-readable meaning for a possible future planner. Therefore this is not a primarily free-text narrative contract.

Agreed direction:
- shared, versioned vocabularies for finding categories and anatomy;
- explicit `other` with supporting description and `unknown` where appropriate;
- defined fields and allowed values for assessable attributes;
- short supporting text for detail, not a replacement for standardized fields.

No actual vocabulary has been selected. RadLex/common data elements were discussed as references, not mandated terminology or finalized mappings. Define the vocabulary after scope/dataset selection; do not reveal case-specific reference labels or expected finding counts. Both models receive the same definitions.

Do not force unsupported diagnoses, severity grades or physical measurements. Dataset-specific attributes remain separate where appropriate: Calcification must not become a universal required attribute across modalities. Supporting text is not automatically scoreable.

Distinguish structural validity, reference assessability and agreement/disagreement. Missing or unmapped references remain unscored, not automatically correct or incorrect. Unknown, not applicable and not assessable have different meanings; exact field-level encodings remain to be designed.

## 6. Resolved: preserve categorical certainty

The researcher agreed to explicit finding-assessment categories rather than numerical confidence percentages:
- `present`: the model asserts the finding is visible;
- `suspected`: it reports a possible finding with uncertainty;
- `not_assessable`: it cannot determine presence from the supplied inputs.

These are reported model assessments, not verified certainty or calibrated probabilities. `present` does not mean 100% probability. `not_assessable` is abstention, not simply the lowest confidence level.

Certainty is independent of localization status. A suspected finding may have a proposed region; a confidently identified finding may be unlocalizable. Exact placement of not-assessable entries in the schema remains to be resolved so that abstention does not imply an invented finding.

Agreed evaluation views:
1. Present-only predictions.
2. Present-plus-suspected predictions.
3. Not-assessable outcomes separately.

Also retain results stratified by reported certainty. Do not silently upgrade suspected to present. Do not remove omitted reference findings from denominators because the model abstained.

Unless a compatible reference certainty field exists, do not claim direct agreement with reference certainty. Instead examine reference agreement/coverage within reported categories and the burden of including suspected candidates. No calibration claim is supported merely by categorical reporting.

## 7. Reference evaluation, including SR

The researcher performs comparison locally using a reference representation extracted by dataset adapters. Do not compare model prose wholesale with an SR and treat text similarity as finding accuracy.

For LIDC, preserve verified SR→SEG→CT linkage, reference groups and individual annotations. Other datasets may provide masks, boxes or label tables rather than DICOM SR. The general report must not require SR or LIDC codes.

Separate:
- spatial correspondence and assignment;
- category/anatomy correctness;
- region quality;
- supported attribute agreement;
- reported certainty and abstention.

A correctly located region with an incorrect finding category should remain visible as both a localization result and category error. Multiple predictions and references require explicit assignment rules so duplicates do not multiply successes. Multiple reader annotations do not become multiple detected nodules or an implicit consensus mask.

Attributes are compared only where correspondence and reference support make that comparison eligible. Unsupported values remain unscored. An unmatched prediction is not automatically a confirmed clinical false positive when reference completeness is limited.

Proposed spatial measures remain reference coverage, region/search burden and IoU, separately labelled for masks versus boxes. Final matching, thresholds and aggregation are not yet agreed. Tiny overlap is not equivalent to near-complete coverage; large whole-image regions are not useful localization merely because they cover a reference.

## 8. Resolved: study/input-set assessment status

Require a study-level assessment status so an empty findings list is not ambiguous:
- assessable;
- partially assessable, with required explanation;
- not assessable, with required explanation.

These describe the supplied image set, not a full study that was not supplied. “No findings identified” does not establish clinical normality. An assessable report may contain an empty findings list.

Keep interpretation limitations separate from technical execution failures such as an image failing to load. Do not silently convert a missing-input failure into a completed negative report. Exact status consistency rules remain to be specified.

## 9. Resolved: minimal acquisition/geometry metadata

The researcher wants context resembling what could accompany future Dicoogle indexing. Agreed boundary: a minimal, versioned allowlist of acquisition and geometry metadata, NOT a complete DICOM header.

Approved information categories, when valid and applicable:
- modality;
- broad anatomical region, normalized from approved source metadata;
- exported-image orientation/view;
- laterality of the examination/image, not a reference-derived abnormality side;
- actual exported dimensions;
- physical pixel spacing when valid for the exported image, with explicit units;
- panel order and relative slice position for volumetric inputs.

Values must describe the supplied exports and account for preparation transforms. Keep provenance/source mappings on the researcher's machine. Missing metadata stays missing or unknown; never fill it using reference annotations. Distinguish patient position from actual image orientation.

Exclude initially:
- patient identifiers, names, birth dates and exact examination dates;
- age, sex, symptoms and clinical history unless separately approved later;
- reports, diagnoses, expected finding types/counts and reference measurements;
- SR/SEG content;
- unreviewed private tags and free-text fields.

Do not automatically pass study/series/protocol descriptions. Reviewed acquisition information may later be normalized into explicit fields. Modality-specific fields, such as MR sequence or contrast information, require a defined purpose and further agreement.

The model can receive a readable normalized summary rather than unexplained raw tags. Both configurations receive identical metadata. Exact tags/fields and de-identification/export checks are not finalized.

Do not assume the current Dicoogle installation indexes/exposes all desired fields; that remains a compatibility check. Approval of metadata categories does not authorize image or metadata transmission.

## 10. Resolved: image-local pixel coordinate convention

The researcher's 15:18 WET agreement accepts:
- coordinates refer to each individual supplied/exported image, not the montage or original DICOM array;
- top-left origin;
- x increases rightward, y downward;
- point represented as `(x, y)`;
- rectangle represented as `(x_min, y_min, x_max, y_max)`;
- units are pixels in the stated exported image dimensions, not normalized fractions or millimetres;
- preparation software retains any mapping to original source geometry.

This settles the coordinate unit/direction/representation choice but NOT detailed numerical conventions. Pixel-centre versus edge interpretation, inclusive/exclusive box boundaries, allowed numeric precision, bounds, positive-area checks and transform rules must still be specified consistently before implementation. Do not invent these as already agreed requirements.

Physical spacing metadata is separate from the report's pixel coordinates. Client resizing/compression and image delivery still require verification; identical input hashes alone do not prove identical internal model image representation.

## 11. Outstanding decisions and execution boundary

Resolved since the original checkpoint: rectangle versus radius; point-inside-box requirement; per-image geometry for grouped findings; unlocalizable reporting; controlled-vocabulary direction; categorical certainty and evaluation views; study assessment status; metadata allowlist direction; image-local pixel coordinates.

Still open:
- exact revised schema, vocabularies and attribute sets;
- detailed coordinate/boundary and mixed-localization rules;
- reference assignment, metric thresholds and aggregation;
- exact export/result formats and validation messages;
- actual clients, fresh-context and multi-image feasibility;
- Docker/minimal dependencies, transport and session lifecycle;
- model shortlist verified against actual clients/accounts;
- datasets, sharing permissions and external-processing approval;
- evaluation sample selection and size.

Continue the researcher-led Q&A before involving implementation agents. This record authorizes documentation only: no code, installations, configuration changes, dataset downloads, server startup, client login, image transmission, inference or downstream specialist/Dicoogle implementation. Preserve all historical evidence.
