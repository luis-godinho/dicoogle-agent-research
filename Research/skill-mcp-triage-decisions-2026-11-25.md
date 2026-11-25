# Subscription-client skill and MCP triage — planning checkpoint

Date: 2026-11-25. Status: requirements discussion only; implementation is NOT authorized.

Related: [Finding-report experiment decisions and historical pilot](finding-report-decisions-2026-11-21.md). This is a separate execution-interface and future-experiment planning record. It does not overwrite historical API protocols, outputs, scores or approvals.

## 1. Why this route is being considered

The professors have access to subscription-based Codex and Claude clients but cannot use direct model APIs for this work. A professor suggested using a skill and MCP server instead of the current API-dependent tool.

By “current tool,” the researcher means the Python program that selects CT slices, sends images to a model API and compares the response with references—not the whole future Dicoogle architecture.

The proposed route reuses suitable preparation, validation and evaluation components. The professor's supported client manages model interaction using their own account; the research backend does not make direct model API calls. No subscription-token extraction, credential sharing or unofficial API proxy is intended. Supported authentication, applicable terms and actual client capabilities still require verification.

## 2. Research scope and future motivation

Current research objective: assess general-purpose multimodal models' identification, description and spatial grounding of visible abnormalities in explicitly supplied images. There may be zero, one or multiple findings. This is not merely study-level abnormal/normal or disease classification.

Future motivation, NOT an implemented or evaluated part of this experiment:

Study indexed in Dicoogle → automatic triage → select a suitable specialized model → produce annotation.

The current experiment does not measure specialist selection/execution, segmentation success, downstream diagnosis or end-to-end Dicoogle performance. Useful downstream information can be recorded without claiming those later steps work. Subscription-client orchestration is not automatically the eventual unattended Dicoogle deployment mechanism.

## 3. Roles: skill, MCP and existing Python components

- Skill: reusable workflow instructions explaining the task, tools, report contract and stopping conditions. It is not the model and is not an access-control mechanism.
- MCP server: controlled interface through which the client obtains approved case inputs/contracts and submits results. It need not contain every research operation or the datasets themselves.
- Existing software: reuse suitable deterministic preparation, validation, artifact handling and offline comparison components. Do not rewrite the entire project merely to add MCP.
- Client/model: inspects supplied images, reasons about findings and submits the report.
- Offline evaluator: runs on the researcher's machine with references; it is not available as a feedback tool to the interpreting worker.

Exact MCP tool names, transports, packaging, session orchestration and process boundaries remain undecided. No general shell/filesystem/network tool is required in the domain server. Built-in client tools also need consideration; restricting MCP alone does not prove isolation.

## 4. Packaging and dependency motivation

The researcher uses uv and multiple Python dependencies and does not want professors to install/manage that stack individually. Docker was proposed to package dependencies, not because a permanently running Docker service is intrinsically required by MCP.

Container packaging remains desirable/provisional, not a final implementation decision. Professors would still need any selected runtime and client setup. Because they will receive prepared images, their backend may need far fewer dependencies than the full DICOM-processing/evaluation program. Decide the minimal responsibilities before copying all dependencies into an image.

Software and research data should be managed separately where practical. Putting references inside a container is not an adequate substitute for withholding them: access through host paths, Docker commands or other client tools may bypass the intended interface. No claim of a security boundary has been verified.

## 5. Agreed split: preparation, inference and evaluation

### Researcher's machine — preparation

- Retain original datasets and reference materials locally.
- Prepare the exact slices/images for each study of each selected dataset.
- Export an inference-only directory for the professor.
- The professor-side workflow does not select new slices or need original CT/SR/SEG for this first version.

### Professor's machine — inference

- Receive the prepared directory.
- Invoke the skill once to start the approved batch.
- The client interprets the supplied images through the agreed controlled workflow.
- Save a traceable results package, including failures and correction history.
- No comparison against reference annotations occurs there.

### Researcher's machine — measurement

- Import the returned results.
- Verify correspondence to the prepared inputs.
- Validate and compare against local references.
- Compute localization/region, finding and supported attribute measures.
- Generate research reports.

Local MCP does not mean local inference. Supplying images to a cloud-backed client is potential remote transmission; dataset permissions and account conditions require review before research-data runs.

## 6. Input-package boundary

Agreed contents in principle:
- Approved slice/image files.
- Neutral case and panel identifiers.
- Manifest with image order, dimensions and hashes.
- Only metadata deliberately approved for the task.

Exclude SR/SEG files, reference masks, reference overlays, expected findings, previous model answers, matching scores and diagnostic reports. Check filenames, paths and manifest fields as well as image pixels. Sharing a directory should not accidentally share the full research repository or surrounding references.

Detailed export verification, de-identification checks and directory layout remain to be specified. The same information-exposure policy must apply across clients.

## 7. Client inventory and feasibility status

| Client | Known status |
|---|---|
| Codex desktop application on macOS | Confirmed by researcher; earlier “iOS” was corrected to macOS |
| Claude Code on Linux, using professor's Claude subscription | Believed likely; exact application/OS still needs confirmation |
| Exact versions, account allowances and model availability | Not established |

Do not substitute Codex CLI documentation as proof of desktop support. Do not assume browser chat, desktop chat and coding clients have identical capabilities.

Critical capabilities still to verify for each actual client:
- supported subscription authentication;
- skills and local MCP support;
- all images delivered together with stable labels;
- fresh per-study worker/session creation from one batch invocation;
- image scaling/compression and coordinate interpretation;
- access restrictions, permission prompts and observable logs;
- interruption/resume and model-selection behavior.

No end-to-end feasibility demonstration has occurred in this planning discussion.

## 8. Agreed professor experience and batch lifecycle

Target: one skill invocation starts the whole prepared batch. Repeated manual invocation for every case is undesirable.

Execution is SEQUENTIAL, not parallel:
1. Check/identify the batch and record configuration.
2. Start a fresh interpretation worker/context for one study.
3. Supply its complete image set and common instructions.
4. Receive and technically validate the report; apply bounded correction policy.
5. Save the terminal outcome.
6. Start a new worker for the next study.
7. Return a completion summary and results package.

Fresh per-study context is a preferred requirement subject to demonstrated client support. A coordinator may retain progress, but interpretation workers must not receive earlier case answers or scores. One fresh context for the entire batch is insufficient. Built-in/project instructions and file access must be considered; “fresh” does not mean an empty system context.

If a client cannot meet this automatically, report the blocker rather than silently use one accumulated conversation. One invocation does not imply one underlying model call.

Interruption policy agreed:
- Save completed studies incrementally.
- Pause on connection/subscription interruption rather than lose progress.
- Allow one explicit resume action when needed.
- Never silently rerun completed studies.
- Record any extra attempt on an interrupted study separately.
- Exact interrupted-submission reconciliation remains an implementation-design question.

A terminal invalid report after the correction limit does not stop ordinary processing of subsequent cases. Protocol/integrity failures need an explicitly designed stop policy. Completely unattended execution despite permission prompts or subscription limits is a target, not a guarantee.

## 9. Image delivery and permitted interpretation actions

Agreed: all prepared images for ONE study are supplied together, separately labelled and in fixed order, as in the earlier nine-panel LIDC requests. Other datasets need not have exactly nine images.

Do not intentionally reveal slices sequentially or let the model choose which supplied panels to receive. If a client cannot support the complete input set, treat that as a compatibility issue; do not silently replace it with a montage or different presentation.

Initial protocol excludes additional slices, original-dataset browsing, web searches for case information, external interpretation models and agent-created zooms/crops. Re-accessing the same approved image is conceptually allowed; exact delivery mechanics remain to verify.

Server-side image preparation is distinct from an agent changing its inputs. Known input hashes do not by themselves prove unchanged provider-side image representation.

## 10. Reports and bounded corrections

Initial agreement was to retain the existing finding-report contract: distinct findings, short descriptive labels, panel references, local centre coordinates where possible, supported attributes and permitted uncertainty/limitations. The later model-proposed-region decision in section 14 supersedes the centre-point-only restriction for the FUTURE experiment. Exact revised schema is not decided.

Agreed correction policy: initial submission plus at most TWO correction attempts, i.e. at most THREE report submissions per study. This counts submissions, not underlying model requests.

- Preserve every received payload before normalization and all technical validation messages.
- Return specific technical errors, not reference-based advice.
- Stop at the first technically valid submission.
- After the limit, record terminal invalidity and continue to the next case.
- Never overwrite the first answer with a corrected answer.
- Distinguish representation-only changes from changes to findings, panel assignments, coordinates or attributes. If comparison is impossible because the original is unparseable, record uncertainty rather than assert format-only repair.

Examples of technical invalidity: malformed structure, missing required fields, wrong types, nonexistent panel IDs, out-of-bounds coordinates, duplicated IDs or contract-defined internal inconsistencies. These are proposed validation categories; do not silently strengthen the historical contract. Never automatically clamp/move coordinates or invent missing clinical content.

Execution failures (missing/unreadable inputs, disconnect, no submission) are separate from invalid submitted reports. A valid empty findings list or permitted abstention is not a failure and must not trigger retries to force a positive result.

No references are distributed, so professor-side validation cannot establish clinical correctness or annotation agreement. Those are measured later. Structural validity is not medical correctness.

Report first-submission validity separately from final validity after corrections, and retain content-changing revision counts. This correction-enabled protocol is not equivalent to earlier no-repair API experiments.

## 11. Model selection and cross-client comparison

The researcher wants a shortlist to propose; professors make the final choice. Before a batch, agree on a model and record the identifier/settings actually exposed by the client. No deliberate model switching or interpretation by a different model mid-batch. Pause if the chosen model is unavailable rather than silently substitute.

A specific model/version shortlist is NOT finalized. Earlier conversational suggestions were provisional and must be reverified against official client documentation and actual account menus; do not use API catalog availability as proof of desktop/subscription availability or claim medical suitability from coding/general-vision descriptions. Record observable reasoning/effort settings. Unknown routing/backend details remain unknown.

Both professors must receive identical studies, exact image files/hashes, panel order/labels, task/report instructions and validation/correction policy. Save independent result packages.

Goal: compare models' finding and spatial performance. Qualification: observed differences may also reflect client system instructions, image processing, worker configuration and other settings. Describe complete configurations and do not attribute every difference solely to the model without evidence.

## 12. Development versus actual experiment

LIDC is for initial skill/MCP development and technical checks. Preserve the existing annotation-assisted selection limitation and historical evidence. It does not become an untouched evaluation set by moving to a subscription client.

Suggested staging, not yet an executed plan:
- synthetic image/coordinate compatibility checks;
- existing LIDC development inputs for end-to-end workflow tests;
- separately selected main-experiment datasets once requirements and permissions are settled.

The main experiment should span BOTH different imaging modalities AND different anatomical regions. No particular modality/body-region combination or dataset has been chosen. A small deliberate collection is intended; exact number remains open.

Dataset screening must consider:
- research-use permissions;
- redistribution/sharing of prepared inputs with professors;
- external processing through the actual subscription clients/accounts;
- de-identification and applicable institutional restrictions;
- expert-reference scope and quality;
- feasible preparation and spatial evaluation;
- annotation completeness.

Public download availability or a license label alone is not sufficient. Unannotated does not automatically mean absent. Do not force one directly comparable accuracy number across incompatible datasets/reference standards. Dataset search/selection has not yet been authorized in this Q&A.

## 13. Results-package and provenance requirements

The professor returns findings plus enough evidence to identify the run, not only a pasted final answer. Proposed minimum:
- neutral batch/case/run identifiers;
- input image hashes, order and manifest identity;
- exact submitted payloads and correction history;
- schema/skill/task-instruction versions;
- observable client/model/settings metadata;
- completion, invalidity, interruption and failure statuses;
- observable timestamps and tool/submission events.

Exact formats remain undecided. Missing token counts, costs, backend versions or internal retries must remain unavailable rather than invented. Do not copy old API `call_count=1`, temperature, seed or strict-output claims into this route without evidence.

## 14. Latest decision: model-proposed search regions

The researcher wants each finding to include a point AND a model-proposed region to look at, potentially useful for later specialist examination. The model should estimate the search extent, not merely output a point around which software always constructs a fixed window.

This supersedes the earlier centre-point-only/fixed-window-only plan for the FUTURE experiment. It does not rescore historical LIDC results or replace completed fixed-window exploratory analyses. A new versioned report contract will be required after the remaining choices.

The region means a bounded area suggested for further examination, NOT an exact segmentation or verified lesion boundary. There may be multiple findings and, where appropriate, different regions on different supplied images of the same finding. Detailed cross-panel reporting rules remain open.

Additional supported descriptions/uncertainty may help downstream use, but specialist selection or execution remains outside scope. Dataset-specific attributes should not become universal requirements; Calcification is not appropriate as a mandatory field for every modality/anatomy. Exact shared versus adapter-specific fields need agreement.

## 15. Region measurement: proposed framework, not frozen metric design

Researcher explicitly wants to measure whether predicted regions match the actual reference and, if not exactly, how much they match. Avoid only binary point containment.

Proposed complementary measurements discussed:
- reference coverage: intersection area divided by reference area;
- search-region burden: predicted area, fraction of image searched, number of candidates and overlap/union accounting as appropriate;
- Intersection over Union: intersection divided by union.

These are proposals to formalize after dataset/reference and region-representation decisions. Coverage plus size is important: whole-image predictions can cover references without useful localization. Tiny intersections are not equivalent to near-complete coverage. IoU alone can penalize intentionally contextual search regions.

Label mask-based and bbox-based evaluation separately. Reference-box coverage does not establish actual lesion-mask coverage. Submitted 2D coverage does not establish complete 3D lesion coverage. Preserve reference annotation identity; multiple readers are not automatically a consensus mask.

Still required: multi-finding assignment rules, handling duplicate/extra/missed predictions, reference ambiguity, thresholds and per-dataset aggregation. One large region touching several abnormalities must not automatically count as several correct finding identifications. Description correctness and localization quality are distinct.

## 16. Exact question to resume after this documentation interruption

UNANSWERED — do not mark as decided:

Should the model express its search region as:
1. a centre point plus rectangular bounding box; or
2. a centre point plus radius?

Assistant proposed a rectangle because horizontal and vertical extents can differ without requiring a contour. The researcher has NOT selected either option. Resume with this question after this entry is saved.

Coordinate units, boundary conventions, point/region consistency requirements, image scaling treatment, dimensionality and schema details also remain open. Do not implement a rectangle or radius contract before agreement.

## 17. Remaining feasibility and policy questions

- Verify actual clients/versions/accounts and Claude Linux details.
- Demonstrate one invocation plus fresh per-study context on each target client.
- Verify simultaneous multi-image delivery, label association and scaling behavior.
- Select region representation and revised report contract.
- Decide shared attributes versus dataset adapters.
- Finalize validation and precise correction/revision handling.
- Define resume state and interrupted-attempt reconciliation.
- Determine minimal backend, packaging, transport and installation steps.
- Define permissions and protection against access to other case outputs.
- Confirm model shortlist and supported settings.
- Select datasets and establish sharing/external-processing permissions.
- Define main-experiment matching, metrics and evaluation split before execution.

## 18. Execution boundary

The researcher explicitly requested a careful Q&A before sending anything to implementation agents. Continue resolving requirements together.

This checkpoint authorizes documentation only. It does NOT authorize coding, installations, client logins, server startup, configuration changes, dataset acquisition, research-image transmission, model inference, specialist execution or Dicoogle integration. A GitHub documentation commit is separate from changes to the local implementation repository.

No synthetic or clinical skill/MCP test has been reported as executed during this planning discussion. Preserve that distinction between intentions, documented capabilities and verified behavior.
