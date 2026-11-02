---
status: draft
tags: [dicoogle-agent, experiment, protocol]
---

[[Home]] · [[Scope and Research Question]] · [[Reference Use Case]] · [[Contracts v0.1]]

# Annotation Ablation Protocol — v0.1

**Status:** draft proposal for supervisor review. No dataset has been selected, no experiment has been run and no LLM has been queried. Nothing in this page is a result.

## 1. Purpose and research questions
Supervisor direction: start with general-purpose LLMs for a triage step, plan around its output, and prepare a pipeline that obtains annotated public datasets and removes annotations to measure the difference in results.

- **RQ-A (primary):** On annotated public radiology datasets, how does the triage performance of general-purpose LLMs change when DICOM annotation objects and/or text are removed from the input?
- **RQ-B (secondary, later):** How can an uncertain, untrusted triage output be used by the planner without bypassing deterministic validation? Out of scope for the first experiment.

Motivation, not a result of this project: a published study of GPT-4V on brain MRI cases (Radiology, 2025) reported that textual description of findings was the dominant factor in diagnostic accuracy, while image annotations did not help. Verify details before citing; record in [[Reading List]] when that page exists.

## 2. Scope
In scope: radiology; general-purpose multimodal LLMs; public de-identified datasets; one triage task at a time.
Out of scope: specialized medical models (later stage); non-public data; clinical claims; fine-tuning; planner integration; pixel-level AI pipelines.

## 3. Annotation taxonomy
| Class | Examples | Status in v0.1 |
|---|---|---|
| A. DICOM annotation objects | Segmentation (SEG), Structured Report (SR), RT Structure Set; other annotation-bearing objects found during inventory | Required: must be removed and varied |
| B. Text | Radiology reports, clinical history or indication, free text in DICOM header fields, meaningful text in file or series names | Required: must be removed and varied |
| C. Marks in pixel data | Arrows, boxes, burned-in text | Exploratory; at minimum audited as leakage |

## 4. Conditions
Paired design: every case is evaluated under every condition.

| ID | Image data | Class A | Class B |
|---|---|---|---|
| C0 | yes | no | no |
| C1 | yes | yes | no |
| C2 | yes | no | yes |
| C3 | yes | yes | yes |

C0 vs C1 isolates DICOM annotation objects; C0 vs C2 isolates text; C3 vs C1/C2 tests interaction. How class A is presented to a model (rendered overlay, mask image, serialized values) and how image data are presented (slices, montage, windowing) are open decisions; each must be fixed and documented before running.

## 5. Task and reference standard
The triage task is chosen before data are downloaded; the pilot uses one task. Candidates, to decide with the supervisor: normal vs abnormal; finding present or absent; finding category or body region; series relevance; urgency level.

Reference labels come from the dataset's own annotations or curated metadata, are fixed before any model is run, and are stored apart from model inputs. Labels may derive from the same annotations that are removed, so the label source must be documented per dataset and the task framed as inference from the remaining input. Model output uses a fixed JSON schema; parse failures are counted as invalid and never silently repaired.

## 6. Pipeline stages
1. Inventory candidate collections. Record per dataset: license and terms, access route, modality, annotation classes, text availability, de-identification statement, size.
2. Select one collection for the pilot (about 20–30 cases).
3. Acquire from the source; record versions, dates and hashes.
4. Validate DICOM metadata; define series-selection rules in advance.
5. Build variants C0–C3 from the same source case with deterministic scripted transformations; no manual edits.
6. Leakage audit (section 7).
7. Package with a per-case manifest: source identifiers, transformations applied, hashes; reference labels kept separate.
8. LLM execution, gated by section 9.
9. Analysis.

## 7. Leakage controls
Stripped variants must not reveal removed information through: DICOM header fields (series or study description, protocol name, image comments, referenced series or object UIDs), file or directory names, manifest or prompt text, UIDs or identifiers that encode meaning, burned-in text or marks. Use an allowlist of what the model sees, not a blocklist. Audit programmatically and with a manual sample, and report the method and findings. Do not assume UIDs or hashes are anonymous.

## 8. Model runs and analysis (proposed)
- At least one general-purpose multimodal LLM. Record provider, model name and version, access date, parameters and prompt-template version.
- Prompts and schema are versioned and frozen before running; no tuning on evaluation cases.
- Repeat each case and condition N times (pilot default N=3, to be justified) to measure run-to-run variability.
- Compare conditions pairwise on the same cases. Report the task metric (for example accuracy, sensitivity, specificity as appropriate), invalid-output rate, consistency across repeats, cost and latency.
- Choose the statistical method before collecting data (for example paired tests or bootstrap intervals). Report null and negative results.

## 9. Data governance and gate before the first external LLM call
Only public, de-identified datasets; confirm each dataset's terms; no non-public data. All items must be recorded as satisfied before any request to an external provider:
- [ ] Dataset terms allow sending the data to the chosen provider.
- [ ] Provider terms on retention and training use reviewed.
- [ ] De-identification and burned-in text audit done on the exact files sent.
- [ ] Leakage audit completed for C0–C3.
- [ ] Prompt, schema, model version and analysis plan frozen.
- [ ] Supervisor approval recorded.

This is a technical checklist, not a legal conclusion about GDPR or the EU AI Act; institutional review remains open.

## 10. Relation to the agent
Triage output is an untrusted proposal. If used later, it informs the planner's input while the deterministic validator in [[Contracts v0.1]] still decides admissibility. The experiment harness stays separate from the Dicoogle plugin; a tool enters the catalogue only after a result justifies it. Details may move to [[Dataset and Ground Truth]] and [[Evaluation Protocol]] when those pages are written.

## 11. Threats to validity
Training-data contamination of public datasets; case selection bias; prompt sensitivity; model version drift; label circularity; small pilot sample; limited generalization beyond the chosen modality and task.

## 12. Open decisions
1. Triage task for the pilot.
2. Collection for the pilot.
3. How class A objects are presented to the model.
4. Whether SR content converted to text counts as class A or class B.
5. Image presentation (slices, montage, windowing).
6. Model(s) and provider(s).
7. Sample size and statistical method.
8. Whether class C goes beyond a leakage audit.
