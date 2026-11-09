---
status: evolving
tags: [dicoogle-agent, experiment, decisions]
---

[[Home]] · [[Experimental Pipeline]] · [[Annotation Ablation Protocol]]

# Experimental Pipeline Decisions

Decision snapshot: 2026-11-09. Accepted engineering decisions are distinguished from research proposals awaiting approval. This is a record of reasoning, not experimental results.

## D1 — Keep acquisition outside the harness

**Decision:** datasets are downloaded and organised independently; the harness reads existing local files only.

**Why:** the research question concerns model inputs and outputs, not repository access or download infrastructure. Acquisition tooling would increase scope without helping answer the initial question.

**Consequence:** no production IDC/TCIA dependency or automatic fetching. Earlier IDC exploration informs dataset assessment but is not the runtime architecture. Missing data is reported explicitly.

## D2 — Separate the experiment from the Dicoogle runtime

**Decision:** build and test the experimental harness independently, with a later interface for planning and validation.

**Why:** triage evaluation should not wait for PACS integration or a production execution engine. Conversely, model accuracy must not substitute for deterministic plan validation.

**Alternative not adopted:** embedding the first experiment directly into the Dicoogle runtime.

## D3 — Reusable core, bounded adapters

**Decision:** keep dataset rules in reviewed configurations and provider behaviour behind connectors. Permit small adapters for genuinely unsupported formats or protocols.

**Why:** neither one dataset-specific script nor a universal configuration language is a suitable foundation. New datasets using supported formats should not require changes to orchestration or evaluation.

## D4 — Deterministic preparation, untrusted model output

**Decision:** code constructs cohorts, resolves labels and prepares inputs; a human reviews dataset definitions. Any future assistant may propose configuration, never silently approve it.

**Why:** eligibility, labels and transformations must be inspectable and reproducible, independent of a model's interpretation.

**Consequence:** model outputs remain proposals or predictions, not ground truth.

## D5 — Keep ground truth separate

**Decision:** store evaluation labels and analysis-only columns separately from the model-facing case view. Use allowlisted text and explicit forbidden sources.

**Why:** reports, diagnosis fields and annotation semantics may reveal the target. A good score obtained through accidental disclosure would not answer the intended question.

**Consequence:** class vocabulary can appear in the task schema, but the actual case label must not be read or interpolated into the input. Public/de-identified data still requires leakage and data-use review.

## D6 — Do not interpret missing annotation as a negative finding

**Decision:** absence of SEG or another annotation is not a negative diagnosis by default.

**Why:** in the reported LIDC exploration, seven of twelve usable-diagnosis rows in the group without SEG were malignant. Missing annotation reflects annotation availability, not necessarily absence of disease.

**Consequence:** eligibility and missing-annotation policy are explicit. Patient-level diagnosis is not automatically a label for every nodule.

## D7 — Do not commit the study to LIDC-IDRI

**Decision:** use LIDC for exploration and optional integration checks, while retaining the ability to select a better dataset.

**Why:** the clinical table has limited coverage, heterogeneous diagnosis methods and patient-versus-nodule ambiguity. The reported annotated usable subset contained 118 rows: 31 benign, 41 primary lung malignancy and 46 metastatic malignancy. These are provisional observations; unique-patient counts and local eligibility must be rechecked.

**Consequence:** those counts are references, not hard-coded success criteria. Real-data discrepancies are investigated; synthetic tests validate logic rather than force a local copy to reproduce exploration totals.

## D8 — Distinguish overlay display from localisation assistance

**Decision:** explicitly name which effect an experiment measures. Final slice policy requires research approval.

**Option A, current proposal:** use the same mask-selected slices in both arms and display the mask in only one. This isolates the display effect, but the image-only arm is already annotation-informed.

**Option B:** use annotation-independent slices in the baseline and mask-guided slices plus overlay in the other arm. This measures a combined localisation and display effect.

**Why:** changing both location and display while claiming an overlay-only effect would confound the interpretation.

## D9 — Report coverage, uncertainty and suitable baselines

**Decision:** do not rely on accuracy alone. Proposed initial binary endpoint is balanced accuracy, with AUROC and other secondary metrics; final protocol requires approval.

**Why:** for the reported 31/87 class distribution, always predicting malignant yields about 73.7% accuracy and 0.50 balanced accuracy. The 73.7% figure is not a threshold for AUROC or balanced accuracy.

**Consequence:** report class counts, successful/failed cases, intervals and paired effect estimates. Do not use a universal 16-percentage-point detectability rule. Seed handling and confirmatory test families must be declared before confirmatory runs.

## D10 — Failures remain visible

**Decision:** preserve invalid outputs and transport failures rather than silently repairing or dropping them from the report.

**Why:** success-only scores can be misleading if difficult cases fail selectively.

**Consequence:** conditional-on-success metrics accompany coverage. With N eligible cases, C correct successful responses and F failures, accuracy bounds over unresolved outcomes are C/N and (C+F)/N; conditional accuracy C/(N-F) is a different quantity. Do not invent probability scores for failed cases.

## D11 — Simple files and resumable execution

**Decision:** use local configuration and artifact files rather than a service, database server or scheduler. Preserve completed attempt records and raw model exchanges.

**Why:** a small research harness benefits more from inspectability than infrastructure.

**Consequence:** validate the ledger before remote calls. Malformed records stop automatic resume; genuine corruption requires an explicit backed-up manual recovery. A valid final record without a newline can be terminated before appending. Remote calls may repeat after a crash before persistence; no exactly-once or perfect cost-accounting guarantee is claimed.

## D12 — Synthetic first, then real-data verification

**Decision:** test each stage on a tiny generated fixture and a mock connector before using large datasets or paid inference.

**Why:** failures should be reproducible without downloads, credentials or provider availability.

**Limit:** passing fixtures establishes tested logic, not universal DICOM conformance, support for every transfer syntax or clinical validity. Real-data visual and reference checks remain necessary.

## Decisions still open

- Exact triage target and unit of analysis; malignancy classification is an exploratory candidate, not the only interpretation of triage.
- Final datasets and minimum acceptable reference-standard quality.
- Patient/lesion linkage and treatment of multiple series or nodules.
- Slice count, windowing, presentation and localisation policy.
- Whether any non-leaking text condition is justified for a selected dataset.
- Models, providers, repetition count, budget and data-use permission.
- Primary endpoint and inferential policy, including how seeds enter multiplicity control.
- Later planning task, reference plans and integration with Dicoogle validation.

## Completion criterion

The engineering goal is a reproducible chain from local inputs to inspectable results. The research goal is a defensible comparison with explicit limitations, including negative or inconclusive findings. Neither requires a model to outperform another or a statistically significant improvement.
