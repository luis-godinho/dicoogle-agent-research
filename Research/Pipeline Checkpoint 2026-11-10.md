---
status: checkpoint
tags: [dicoogle-agent, experiment, progress, decisions]
---

[[Home]] · [[Experimental Pipeline]] · [[Experimental Pipeline Decisions]] · [[Local Catalogue Implementation Decisions]] · [[Annotation Ablation Protocol]]

# Pipeline Checkpoint — 2026-11-10

## Evidence boundary

This checkpoint records developer-reported implementation and executed-test results supplied on 2026-11-10. The implementation repository and tests were not independently audited for this note. This is a research-documentation checkpoint, not a commit of the local Python implementation or a scientific result.

It supersedes earlier progress snapshots, while preserving their decision history. No complete real-model experiment or real LIDC validation is reported.

## Milestone reached

The local synthetic-data chain now works from catalogue construction through cohort publication:

Existing local files → header catalogue and annotation references → validated dataset definition → clinical-table loading → label resolution → cohort, model-safe case view and audit.

Acquisition remains external. The application never fetches missing dataset files. Rendering, model-ready input construction and inference remain future work.

## Reported progress

| Component | Reported result | Scope |
|---|---|---|
| Catalogue command and reporting (T06) | 20 focused / 90 total tests passed; fixture scan executed | Local catalogue, reference accounting, duplicate-group content comparison and controlled publication |
| Manifest validation and alignment (T07) | 59 focused / 149 total passed | Safe YAML, schema/invariants, explicit paths and physical/logical column conventions |
| Table loading (T08) | 20 focused / 169 total passed | String-preserving CSV/TSV loading, provenance and non-destructive duplicate diagnostics |
| Label resolution (T09) | 15 focused / 184 total passed | Patient-level mapping with semantic-conflict and analysis-ambiguity reporting |
| Cohort construction (T10) | 28 focused / 212 total passed | Deterministic eligibility, separate case view, detailed audit and metadata |
| LIDC integration preparation (T11) | 7 passed / 3 skipped; full suite 219 passed, 3 skipped, 2 warnings | Draft manifest and opt-in integration checks; not real-data verified |

These totals describe successive suite snapshots and must not be added together.

## Observed synthetic outcomes

The executed catalogue smoke scan indexed 35 DICOM files: 30 CT, three SEG and two SR, across five patients, five studies and ten series. Of 38 files encountered, two were sidecars and one was a deliberate parse error. Five annotation objects were accounted for: four resolved and one with a known missing target.

The primary cohort included p0001 and p0002, one benign and one malignant. It excluded three units with distinct reasons: unknown diagnosis, absent clinical row and absent annotation. Included/excluded counts reconciled with detailed audit records. These outcomes test software logic, not clinical performance.

## Decisions now operational

### Local dataset and table identity

A dataset directory resolves under `<data-root>/data/<data.dir>/`. Table mappings use logical names as keys and physical headers as values. `key_column` is a physical header and is read independently even when not mapped to a logical field. Leading zeros and literal strings such as NA remain intact.

### Review is not validation

A draft may validate structurally without being approved. Approved configurations require reviewer/date metadata. A deliberate draft override can support integration tests, but must be recorded and never changes approval status.

### Preserve evidence before deciding

The table loader retains every source row and source record number. The label resolver examines all rows for a unit. Different semantics remain conflicting even when they map to the same binary target. Known and unusable evidence is not silently reduced to a convenient row.

### Ambiguous analysis fields

A resolved label may enter an unfiltered primary cohort despite analysis-only disagreement. Ambiguous scalar analysis fields are null in the cohort, with all values/provenance retained in the audit. A filter that depends on such a field excludes the unit as ambiguous rather than choosing the favourable value.

### Annotation eligibility

Support counts distinct selected annotation objects, not reference-row multiplicity. Only annotations linked to the selected series satisfy its role. Missing targets and conflicting annotation content do not satisfy requirements. An explicit ignore policy can include labelled cases without SEG; it does not create an overlay or make absence a negative diagnosis.

### Separate reference data from model inputs

`cohort.parquet` holds evaluation labels and analysis fields. `cases.parquet` contains only case/unit identifiers, selected-series and file references, and dataset/cohort identifiers. This internal case view is not claimed to be anonymised or suitable for wholesale remote transfer.

## Explicit support limits

- Non-null label aggregation is rejected: sub-unit structure and combination semantics are not defined.
- `treat_as_negative` is rejected at cohort construction; a written justification alone does not define safe label-override semantics.
- Analysis/filter columns currently come from the label-source table; cross-table joins are not guessed.
- Whole-scan content hashing is unsupported. CLI `--hash` is unrecognised and exits non-zero; duplicate groups are content-hashed. API rejection was inspected, not separately executed in the reported T06 validation.
- Two multiprocessing fork warnings persist in the parallel scanner test. Use workers=1 for optional real-data checks until the context is addressed.
- Enumeration starts from image-modality rows; units with no such rows are not a complete population represented in the cohort audit.
- The primary cohort identifier remains `annotated_usable`; configuration and policy, not that name, establish actual annotation eligibility.
- Label-free planning remains a later contract issue; current manifest validation is triage-oriented.

## LIDC-IDRI remains provisional

No configured local LIDC image copy, clinical table or reusable catalogue was available. The draft manifest therefore has not been checked against real headers, key uniqueness, row counts or annotation-source provenance.

The SEG selection currently accepts all locally present SEG sources as a draft fallback, not a scientifically approved mixture. Header fields do not yet provide a verified source discriminator. Dataset terms, citation, final eligibility and review approval remain unresolved.

A synthetic fixture renamed for exercising the opt-in integration path passed ten tests. It is not LIDC verification. Its biopsy/surgery subset used in that smoke run requires clarification of the temporary method-code variant, because the canonical fixture contains analysis sentinels rather than clinical method codes.

The exploratory estimate of approximately 118 usable annotated rows remains provisional and is not a real-data success assertion.

## Next execution order

1. T20: experiment configuration, effective input specifications and response-schema compilation. Deliberately scheduled early; not reported implemented at this checkpoint.
2. T12 and T13: CT rendering, slice selection, montage and SEG overlay.
3. T15, T14 and T16: prompt construction, input records/cache and input-stage acceptance.
4. T17/T18: mock interface and provider connector; additional adapters only if required.
5. T21–T23: execution, persistence/resume and mock-run acceptance.
6. T24–T27: evaluation and reporting.
7. T28: separately approved small real-model pilot on a suitable local dataset.

Real-data integration can be revisited independently when a dataset is available. It must not block synthetic development or force selection of LIDC as the final dataset.

## Research decisions still pending

Exact triage task, final datasets/reference standards, patient-versus-lesion interpretation, slice/localisation policy, permissible text conditions, models and budgets, repeated-run strategy and confirmatory statistical family remain subject to review before real experiments. Software test success does not approve these choices or establish clinical validity.
