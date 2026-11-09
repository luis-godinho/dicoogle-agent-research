---
status: index
tags: [dicoogle-agent, index]
---

# Dicoogle Agent — Research Vault

Draft research documentation for a proposed verifiable DICOM-native pipeline agent. Open this repository as an Obsidian vault; start here. Diagrams and technical identifiers are in English.

## Start here
1. [[Scope and Research Question]]
2. [[Reference Use Case]]
3. [[Architecture v0.1]]
4. [[Contracts v0.1]]
5. [[Rules and Tests]]
6. [[Decisions and Next Steps]]

## Architecture
[[System Context]] · [[Components]] · [[Sequence]] · [[Architecture Decisions]] · [[Dicoogle Integration]]

## Research
[[Reading List]] · [[Related Work Matrix]] · [[Dataset and Ground Truth]] · [[Evaluation Protocol]] · [[Reproducibility]] · [[Annotation Ablation Protocol]]

## Local LLM experiment
[[Experimental Pipeline]] · [[Experimental Pipeline Decisions]] · [[Annotation Ablation Protocol]]

The experimental harness starts from datasets already present locally; acquisition is external. The decision record explains scope, rationale, rejected alternatives and unresolved research choices. This work is separate from the Dicoogle runtime.

## Specification and security
[[JSON Schemas]] · [[Approval and Governance]] · [[Threat Model]] · [[Privacy and Retention]]

## Implementation and writing
[[Implementation Plan]] · [[Roadmap]] · [[Research Log]] · [[Paper Outline]] · [[Limitations]]

## Templates
[[ADR Template]] · [[Experiment Template]] · [[Meeting Template]]

## Status
The Dicoogle v0.1 architecture and specification materials remain proposals unless an individual page provides implementation evidence. Their synthetic examples and expected test behaviours are not clinical results.

For the separate experimental harness, application scaffolding and external-access safeguards were reported completed on 2026-11-09 (29 tests passing; not independently verified here). Synthetic imaging fixtures, the full local-processing chain and real-model experiments are not yet reported complete. See [[Experimental Pipeline]] for the evidence boundary. Pages marked `pending` remain work to do. No clinical or universal-security claims are made.
