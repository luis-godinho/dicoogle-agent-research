---
status: evolving
tags: [dicoogle-agent, experiment, catalogue, decisions]
---

[[Home]] · [[Experimental Pipeline]] · [[Experimental Pipeline Decisions]] · [[Annotation Ablation Protocol]]

# Local Catalogue Implementation Decisions

Recorded: 2026-11-09. This note explains implementation choices for the local catalogue and annotation-reference resolver. It supplements the broader experiment decisions without redefining the research protocol.

## Evidence and progress

The evidence below consists of developer reports supplied during implementation discussions, not an independent source-code audit or a new test run performed for this note.

| Component | Reported progress | Evidence boundary |
|---|---|---|
| Synthetic dataset generator (T03) | Completed; 11 focused tests, 40 total tests passing | Structural synthetic CT/SEG/SR objects; not fully conformant clinical IODs |
| Header-only scanner (T04) | Completed; 15 focused tests, 55 total tests passing | Real-dataset validation not reported |
| Annotation-reference resolver (T05) | Completed; 15 focused tests, 70 total tests passing | Synthetic reference-path coverage, not universal DICOM support |
| Catalogue CLI and reporting (T06) | Next step | No completion reported at this snapshot |

T03/T04/T05 work was reported uncommitted at the latest checkpoint. Do not infer its Git status from the research-vault commit containing this note.

This progress supersedes the earlier progress snapshot in [[Experimental Pipeline]], where the synthetic fixture and catalogue were still listed as future work. No completed real-model experiment is reported.

## 1. Do not follow directory symlinks

**Status:** implementation instruction for T04; exact behaviour still to confirm against code/tests.

**Decision:** do not follow directory symlinks in the initial scanner.

**Rationale:** avoid traversal cycles and unintended access outside the selected dataset tree.

**Alternative not adopted:** following every directory symlink automatically.

**Consequence:** data reachable only through such links is not indexed through them. This is an explicit change from the earlier traversal proposal and must remain visible in the scanner's behaviour and documentation.

## 2. Treat image-series identification as provisional

**Status:** implemented according to the T05 report.

**Decision:** identify candidate image targets from catalogue metadata, excluding SEG, RTSTRUCT, SR and other known non-image modalities through a blocklist.

**Rationale:** reference resolution needed an operational definition of an image target; the original contract did not provide one.

**Limitation:** exclusion from a non-image list is not proof that an arbitrary object is a supported image object. The rule needs verification before extending to new object types or modalities.

**Consequence:** the fixture CT references resolve, but this does not establish universal image-target classification.

## 3. Support tested reference paths, not speculative fallbacks

**Status:** implemented according to the T05 report.

**Decision:** support the following extraction sites:
- SEG: `ReferencedSeriesSequence[i].SeriesInstanceUID`.
- RTSTRUCT: `ReferencedFrameOfReferenceSequence[i].RTReferencedStudySequence[j].RTReferencedSeriesSequence[k].SeriesInstanceUID`.
- SR: `CurrentRequestedProcedureEvidenceSequence[i].ReferencedSeriesSequence[j].SeriesInstanceUID`.

**Rationale:** implement paths exercised locally by tests rather than assume that a sequence mentioned during design is a valid general fallback.

**Alternative not adopted:** speculative use of `ReferencedPerformedProcedureStepSequence`.

**Consequence:** other evidence/reference paths remain unsupported until verified and tested. Additional annotation modalities are not emitted as supported merely because the schema leaves room for them.

## 4. Preserve a reference when its target is absent

**Status:** implemented according to the T05 report.

**Decision:** retain the extracted series UID and record `target_missing` when the corresponding image series is absent locally. Use `no_reference_extracted` and a null target UID only when no usable reference was extracted.

**Rationale:** incomplete local data and absence of a usable reference are different problems. Erasing a known UID would make diagnosis and subsequent resolution harder.

**Consequence:** missing targets are recorded, never downloaded. For a no-reference row, `reference_path` uses the documented empty-string sentinel.

## 5. Keep unreadable and ambiguous objects visible

**Status:** implemented according to the T05 report; aggregation into the catalogue report is pending.

**Decision:** unreadable annotation files produce structured resolver problems, not fabricated no-reference rows. Conflicting reference signatures for duplicate annotation SOP UIDs produce ambiguity diagnostics rather than trusted reference rows.

**Rationale:** failure to inspect an object is not evidence that it lacks a reference, and ambiguous objects must not silently look resolved.

**Consequence:** the final report cannot count annotations solely from `refs.parquet`. It must also account for resolver problems and ambiguities. An annotation may reference multiple series, so object counts and reference-row counts must stay distinct.

## 6. Equal extracted references do not prove equal contents

**Status:** recognised limitation of T05; content-based duplicate accounting is pending.

**Decision:** extraction signatures may deduplicate metadata links but must not establish that two files are identical.

**Rationale:** two annotations can reference the same image series while containing different masks or other content.

**Consequence:** the later catalogue reporting step must distinguish identical from conflicting duplicate files using content comparison under the duplicate policy. Matching reference signatures must not hide conflicting content. T05 does not implement content hashing/classification.

## 7. Real-data validation remains a separate step

**Status:** not performed in the reported scanner/resolver work.

**Decision:** without an explicitly supplied dataset path, do not search the machine or acquire data automatically. Use synthetic fixtures for initial acceptance.

**Rationale:** respect the local-input boundary and make tests independent of large downloads or credentials.

**Consequence:** real-data reference coverage, annotation-source discrimination, performance and additional transfer-syntax behaviour remain unverified. No LIDC-specific selection rule was inferred without real-data checks.

## Reported fixture outcome

The generator produces five patients, 30 CT instances, three SEG instances and two SR instances, plus a clinical table and deliberate non-imaging/error files. The resolver reported five reference rows: four resolved annotation objects and one missing target; no resolver problems or ambiguities on the unchanged fixture.

Tests additionally exercised missing references, multiple targets, a synthetic RTSTRUCT chain, unreadable annotation files, duplicate-UID ambiguity and header-only reads. Reference resolution establishes metadata linkage, not spatial mask alignment, lesion-level label correctness or clinical validity.

## Follow-up before extending the catalogue

- Aggregate scanner and resolver diagnostics without losing annotations that have no reference rows.
- Complete content-based duplicate accounting and test conflicting files with equal reference signatures.
- Confirm the directory-symlink policy against the implementation.
- Capture and investigate the two fork-related warnings reported in the full T05 test run; their exact messages and origin are not yet established here.
- Verify reference paths and image-target classification on a user-selected real local dataset.
- Keep these implementation limitations separate from approval of the scientific protocol.
