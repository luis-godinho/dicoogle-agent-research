---
status: evolving
tags: [dicoogle-agent, experiment, decisions]
---

[[Home]] · [[Experimental Pipeline]] · [[Annotation Ablation Protocol]]

# Experimental Pipeline Decisions

Decision snapshot: 2026-11-18. Accepted engineering decisions are distinguished from research proposals awaiting approval. This is a record of reasoning, not experimental results. Implementation checkpoints below are based on development verification reports, not an independent audit of the implementation repository.

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

## D13 — Accept the input builder against observable artifacts

**Decision:** require end-to-end synthetic acceptance of rendering, SEG alignment, montage composition, text allowlisting and input records before integrating execution.

**Evidence reported:** Stage 3 acceptance passed with independent pixel comparisons, populated/blank tile arithmetic, sentinel detection, deterministic rebuilds and cache invalidation checks. The acceptance checkpoint was 468 passed, 3 skipped and 2 pre-existing scanner warnings.

**Consequence:** preserve original path order, ordered slice index and DICOM InstanceNumber as distinct concepts. Union applicable SEG masks consistently for both mask-informed selection and overlay display. Reject unsupported geometry rather than guessing alignment. Synthetic success does not approve the study's slice policy or establish real-LIDC compatibility.

## D14 — Cache validity includes code and current dependencies

**Decision:** input reuse requires matching identity, intact generated artifacts and verifiable current manifest, template, response schema, implementation and source dependencies.

**Why:** a committed Git revision alone cannot identify different dirty worktrees, and a generated image hash does not establish that its source is current.

**Consequence:** implementation identity includes a deterministic package-relative source fingerprint. Source size and nanosecond mtime are checked against the current filesystem. Missing required sources produce explicit failures, not stale hits. Edits preserving both size and mtime remain an acknowledged limitation.

## D15 — Explicit connector capabilities, no silent weakening

**Decision:** use one bounded OpenAI-compatible adapter with synthetic transport fixtures. Strict JSON Schema output is the default; unsupported or unverified strict compatibility fails before transport. Non-strict modes require explicit configuration. Local response validation remains mandatory.

**Consequence:** preserve image bytes and the exact prepared prompt. Deliver the requested case identifier in a separate model-visible message and retain the complete outbound body for later persistence. Do not equate internal metadata with content delivered to a model. Configuration controls timeouts, token parameters and per-image raw-byte limits. Provider-specific verification does not imply universal compatible-server support. No live inference was used for adapter verification.

**Mock identity:** SHA-256 over UTF-8 case_id|condition_id|model_id|seed, with decimal seed. The schema enum supplies predictions; the declared positive class supplies the historical always_malignant mode. Oracle mode requires explicit test-only permission.

## D16 — Version run semantics, not individual ledger lines

**Decision:** run.json.schema_version governs interpretation of attempts.jsonl. New runs target version "2"; the attempt ledger has no per-line version.

**Why:** terminality and response acceptance changed, not merely the shape of optional metadata.

**Consequence:** version "1" retains original terminality; "1.1" retains the approved truncation rules without mandatory identity checking; "2" inherits truncation handling and requires case identity validation. Unknown versions are rejected. Existing runs are never silently upgraded, rewritten or retroactively evaluated under newer semantics. Lifecycle writing and full persistence remain T22 work.

## D17 — Separate content validity, identity and truncation

**Decision:** transport errors take precedence. For structurally valid content in version "2", the executor must check the returned case_id against the requested identifier. A mismatch is terminal case_id_mismatch, never reassigned or automatically retried.

**Consequence:** mismatch records exclude parsed content from valid predictions while preserving original response and accounting metadata. Length truncation is assessed separately: invalid JSON, schema-invalid content and identity-matched valid content may each qualify for the single truncation retry. A mismatch never qualifies. A schema-valid final length-truncated response remains ok rather than being falsely labelled a parsing error.

## D18 — Bound retries explicitly and preserve every call

**Decision:** the runner, not the adapter, owns one truncation-retry phase. Double the previous persisted request's output bound exactly, only when a configured output cap and the budget guard permit it. Transport retry allowances are separate per phase.

**Consequence:** preserve every persisted call separately. A content response completes the retry phase; exhausted transport retries also complete it. Resume reconstructs the phase and cannot double again. Pre-call budget controls do not consume transport retries or start a retry phase. Under uninterrupted, fully persisted execution, the call bound is max_retries + 1 without truncation retry, or 2 × (max_retries + 1) with it. Crash windows prevent an absolute remote-call bound.

## D19 — Project the actual request and expose execution uncertainty

**Decision:** keep plan-level assumed-token estimates separate from the per-call budget projection. The latter uses assumed input tokens and the effective request.max_output_tokens, including a doubled retry bound, at configured prices. Reported cost and estimated cost remain distinct. This projection is not a guaranteed billing upper bound.

**Decision:** an unexpected connector exception does not prove that no provider call occurred. Treat the outcome as terminal internal_error without automatic retry, preserve the truncation-phase marker and expose unknown execution/cost rather than fabricating usage or model output.

**Pending verification:** confirm the exact persisted location, version scope and validation of provider_call_attempted, execution_uncertain and cost_unknown. Confirm how reconstructed spend and the budget guard handle unknown cost; unknown must not silently become known zero. These details remain an integration gate before T22, not a closed accounting guarantee.

## Engineering checkpoint — 2026-11-18

The latest development report records 660 passed, 3 opt-in local-data skips and 2 pre-existing scanner fork warnings after T21 corrections. The runner core is library-only. T22 lifecycle, actual request/response persistence and CLI integration have not started in the reported work. This note does not independently certify the implementation or claim real-provider inference, real-data validation or completed end-to-end evaluation.

## Decisions still open

- Exact triage target and unit of analysis; malignancy classification is an exploratory candidate, not the only interpretation of triage.
- Final datasets and minimum acceptable reference-standard quality.
- Patient/lesion linkage and treatment of multiple series or nodules.
- Slice count, windowing, presentation and localisation policy.
- Whether any non-leaking text condition is justified for a selected dataset.
- Models, providers, repetition count, budget and data-use permission.
- Primary endpoint and inferential policy, including how seeds enter multiplicity control.
- Later planning task, reference plans and integration with Dicoogle validation.
- Approval of the draft triage wording; technical rendering does not constitute research approval.
- Persisted uncertainty-field representation and unknown-cost budget handling before T22 integration.

## Completion criterion

The engineering goal is a reproducible chain from local inputs to inspectable results. The research goal is a defensible comparison with explicit limitations, including negative or inconclusive findings. Neither requires a model to outperform another or a statistically significant improvement.
