---
status: draft
tags: [dicoogle-agent, experiment, protocol]
---

[[Home]] · [[Scope and Research Question]] · [[Reference Use Case]] · [[Contracts v0.1]] · [[Experimental Pipeline]] · [[Experimental Pipeline Decisions]]

# Annotation Ablation Protocol — v0.2

Status: proposal for supervisor review, updated 2026-11-09. Dataset exploration and initial software foundations have been reported; no completed model experiment is reported. The clinical task, dataset, models and confirmatory analysis are not yet approved.

## 1. Purpose and research questions

Start with general-purpose multimodal LLMs for medical-imaging triage and investigate how available annotations affect their outputs. Later, study how an uncertain, untrusted triage output can inform planning without bypassing deterministic validation.

- RQ-A: how does task performance change when selected annotation information is shown or withheld, on the same cases?
- RQ-B, later: how can triage output inform a technically admissible tool plan?

These are distinct questions. Classification performance does not establish plan validity or clinical usefulness.

## 2. Scope and acquisition boundary

Initial scope: one triage task at a time, general-purpose multimodal models and public de-identified radiology data subject to permission checks. No fine-tuning or clinical deployment claims. Non-public data and live Dicoogle integration require a separate reviewed scope.

Data acquisition is external. The user independently downloads and organises images, annotations and tables. The experimental harness begins from that local folder, never downloads missing dependencies, and does not require IDC/TCIA at runtime. See [[Experimental Pipeline]].

## 3. Annotation taxonomy

| Class | Examples | Treatment |
|---|---|---|
| A. Annotation objects | SEG, SR, RTSTRUCT | Inventory locally; interpret each object's content and permitted presentation |
| B. Text | Reports, history, header text, meaningful file names | Include only if justified and not an unintended disclosure of the reference target |
| C. Pixel marks | Arrows, boxes, burned-in text | Audit; an additional experimental condition requires explicit design |

An object is not automatically safe because it is called an annotation. SR or report content may disclose diagnosis or reader malignancy ratings. Its use may change the task from image inference to interpretation of supplied findings.

## 4. Conditions and localisation

Initial engineering target: image without displayed mask versus image with segmentation overlay. Use identical cases.

The localisation policy remains a decision for approval:
- Same mask-selected slices in both arms: estimates the benefit of displaying the mask, conditional on annotation-informed localisation. The image-only arm is not annotation-independent.
- Annotation-independent selection versus mask-guided selection plus overlay: estimates the combined localisation and display effect.

The earlier four-condition image/object/text factorial design remains a possible extension, not a mandatory initial experiment. Text conditions are added only after defining their information content and leakage implications. Image rendering, windowing, slice count, mask interpretation and prompt must be fixed before the real run.

## 5. Task and reference standard

The exact triage target remains open: malignancy classification is one explored candidate, not an approved universal definition of triage. Select a dataset whose labels support the chosen question.

Reference labels are derived deterministically from declared sources and kept separate from model inputs. Record diagnosis method, label granularity, coverage, conflicts and limitations. Absence of annotation is not a negative finding. A patient diagnosis must not be silently assigned to every nodule.

LIDC-IDRI is an optional integration example, not the committed final dataset. The reported usable annotated subset is provisional and requires local uniqueness and eligibility checks; see [[Experimental Pipeline Decisions]].

## 6. Local experimental stages

1. Index existing local DICOM files and annotation references.
2. Review the dataset definition; construct the eligible cohort and exclusion audit.
3. Prepare deterministic condition-specific inputs, excluding ground truth.
4. Audit leakage and visually inspect representative inputs.
5. Execute the configured models through isolated connectors; preserve raw exchanges and failures.
6. Evaluate on matched cases, report coverage, uncertainty, limitations and reproducibility information.

Synthetic fixtures and mock inference precede real-data integration and a small approved real-model pilot. Pilot size is a feasibility choice, not evidence of statistical power.

## 7. Leakage controls

Use allowlists for model-visible text. Check headers, file names, prompts, manifests, annotation semantics, identifiers and burned-in content. Do not assume UIDs or hashes anonymise information.

Class names in a response schema are legitimate task instructions; exposing the true label of the individual case is not. Automated tests use distinctive forbidden-content sentinels and structural separation from label tables, complemented by manual inspection of real inputs.

## 8. Model runs and analysis proposal

Record model/provider version, parameters, repetitions/seeds, exact input and prompt identities, output schema, failures, latency and available usage/cost data. Do not tune prompts on evaluation outcomes.

For an initial binary task, balanced accuracy is the proposed primary metric, with secondary accuracy, positive-class sensitivity/specificity and AUROC where meaningful. The positive class is explicit. Report matched-condition effect estimates with intervals and class/coverage counts. Success-only metrics are labelled conditional on successful inference.

Exact McNemar on paired correctness is a candidate hypothesis test, not a test of balanced accuracy or AUROC. Any confirmatory family and its seed policy require prior approval; otherwise results remain exploratory. No universal minimum-effect threshold is imposed. Multiple datasets are initially reported separately.

Binary predictions may be analysed by true diagnostic subtype, but must not be presented as three-class predictions. Failed cases receive no invented probability scores. Null, negative and inconclusive findings are retained.

## 9. Gate before external LLM calls

- [ ] Dataset and task selected, with eligible cases and reference-standard limitations reviewed.
- [ ] Dataset terms permit the intended transfer and use.
- [ ] Provider retention/training terms reviewed.
- [ ] De-identification and burned-in content audit completed for the actual inputs.
- [ ] Leakage and input-localisation policies reviewed.
- [ ] Prompt, output schema, model configuration and analysis decisions frozen.
- [ ] Budget and expected call count approved.
- [ ] Supervisor approval recorded.

Public availability alone does not authorise arbitrary external transfer. This checklist is not a legal conclusion; institutional requirements remain applicable.

## 10. Relation to the agent and validity limits

Triage output is untrusted. Later planning must still pass deterministic contracts and validation in [[Contracts v0.1]]. The experiment harness does not establish a production integration.

Threats include training-data contamination, dataset/annotation selection bias, label circularity, patient-versus-lesion mismatch, selective failures, rendering choices, prompt sensitivity, provider drift, small samples and limited generalisation. Current engineering progress is recorded in [[Experimental Pipeline]], separately from experimental evidence.
