# Feasibility report: a DICOM-native pipeline agent as an external Dicoogle plugin

Date: 2026-10-25  
Repository: `/home/jotalma13/Documents/Bolsa/dicoogle`  
Mode: read-only source/build inspection; no code, build, test, service, PACS query, patient data, log, settings, user file, or secret was opened or modified.

## 1. Executive summary

**Verdict: conditionally feasible, but not production-ready as an unmodified external plugin.**

A standalone Dicoogle plugin can expose a custom HTTP API, authenticate callers through the platform, read query results, resolve storage streams, parse DICOM headers, maintain its own catalogue and artifact store, and orchestrate pipeline runs. The SDK supports this shape through `PluginSet`, `JettyPluginInterface`, `DicooglePlatformInterface`, `QueryInterface`, and `StorageInterface`.

The main limitations are not in DICOM access; they are in platform guarantees:

- A Jetty plugin is responsible for its own authentication. Core auth is not automatically applied to plugin handlers, and `/ext/*` Restlet resources are public.
- The core has no object-level authorization, so a plugin cannot inherit per-patient/per-study access control.
- The plugin lifecycle does not reliably support mounted-service ownership: `enable()` is not called at startup, `disable()` does not unmount, start/stop are stubs, and shutdown is disabled by default.
- A Jetty plugin may not survive a web-server restart, and mounting may fail when web autostart is disabled. This needs runtime verification.
- There is no SDK-level catalogue, orchestration, artifact store, provenance, or audit facility. The plugin must implement these itself.
- Core query results are provider-dependent, duplicated across providers, and provider query failures are converted to empty results. The plugin must not treat an empty result as proof of absence.
- The default file storage is read-only (`store(...)` returns null), so a plugin needs its own artifact path and lifecycle.
- Core logs can contain full queries and URIs. A verifiable agent must be stricter than the surrounding application.

Recommended use of an external plugin: a controlled pilot with authenticated Jetty endpoints, a plugin-owned bounded executor, an append-only run/event store under a dedicated path, content-addressed artifacts, and explicit runtime acceptance tests for mounting, restart, and classloading. If the requirement is a generally deployable, centrally governed agent with reliable lifecycle and per-object authorization, a small core/SDK integration or a separate sidecar service is safer.

## 2. Scope, assumptions, and evidence standard

### Scope

This report assesses whether a proposed DICOM-native pipeline agent can be delivered as an external Dicoogle plugin. It covers:

- plugin discovery and lifecycle;
- HTTP routing and authentication;
- catalogue/discovery of existing Dicoogle capabilities;
- DICOM selection, header retrieval, and metadata;
- query semantics and result integrity;
- pipeline execution and concurrency;
- artifact and run persistence;
- validation, provenance, and audit;
- security and privacy;
- runtime checks needed before a commitment.

It does not implement the plugin, choose a clinical workflow, define a medical validation protocol, or inspect live deployment data.

### Assumptions

- The agent must operate against the existing Dicoogle v3.6.1-SNAPSHOT server with the deployed query/storage providers.
- The agent may expose a new HTTP namespace and may use an external process or model only through its own configured integration.
- The agent’s outputs must be traceable to exact input DICOM references and tool versions.
- Patient data must not be logged or copied into unmanaged locations.
- No core change is assumed for the external-plugin option; if core changes are allowed, some limitations disappear.

### Evidence labels

- **VERIFIED IN CODE** — directly observed in repository source/build files.
- **VERIFIED IN DEPENDENCY BYTECODE/SOURCE** — observed in an available dependency artifact.
- **INFERENCE** — follows from code, but depends on deployment/runtime behavior.
- **REQUIRES RUNTIME VERIFICATION** — cannot be settled statically.
- **NOT FOUND IN SEARCHED SCOPE** — absence after the stated search, not proof of impossibility.

A full evidence index and Phase A/B detail is in `/tmp/opencode/dicoogle-phase-a-b-reconnaissance.md`.

## 3. Repository and build baseline

- **VERIFIED IN CODE / GIT:** branch `dev`, HEAD `1ed8c3ad1a6c22ac941265b178cb66e993ab5b87`, `git describe` = `3.3.4-103-g1ed8c3ad`, version `3.6.1-SNAPSHOT`. This is the original Dicoogle v3 codebase.
- **VERIFIED IN CODE:** root `pom.xml` has two Maven modules only: `dicoogle` and `sdk` (`pom.xml:150-153`). The runnable server is the shaded `dicoogle` JAR. `webcore` is not a Maven module.
- **VERIFIED IN CODE:** Java source/target is `1.8` (`pom.xml:23-24`). CI uses JDK 8/11; the local environment has JDK 21. A plugin should be built for Java 8 compatibility unless the deployment explicitly changes that contract.
- **VERIFIED IN CODE:** `dicoogle` depends on `pt.ua.ieeta:dicoogle-sdk:${project.version}` (`dicoogle/pom.xml:446`). The backend consumes the SDK; the SDK is the plugin-facing API.
- **VERIFIED IN CODE:** Jetty is `9.0.3.v20130506`, Restlet `2.1.2`, dcm4che2 `2.0.29` (`pom.xml:11-13`; `sdk/pom.xml:186-245`). Jackson is present; no JSON Schema or Bean Validation dependency was found.
- **VERIFIED IN CODE:** the core and SDK use Maven Shade. Local `Plugins/lucene-3.5.1.jar` and `Plugins/filestorage-3.5.1.jar` are fat JARs embedding SDK/Jetty/dcm4che2 classes.
- **VERIFIED IN DEPENDENCY BYTECODE:** JSPF/ClassWorlds uses parent-first plugin realms whose parent is the core classloader. Core/SDK/Jetty/dcm4che classes win over bundled copies. Bundling conflicting versions is risky and unnecessary; only dependencies absent from core should be packaged.
- **VERIFIED IN GIT:** the tree is dirty: 11 tracked modified files, 0 staged, 40 untracked entries including runtime data and prior reports. The tracked code diffs inspected are formatting-only plus one submodule pointer. No repository file was modified by this investigation.

## 4. Plugin architecture and available extension points

- **VERIFIED IN CODE:** discovery is JSPF-based. `PluginFactory` loads `pt.ua.dicoogle.sdk.PluginSet` implementations from JARs under `Plugins/` (`PluginFactory.java:41-52`).
- **VERIFIED IN CODE:** one `PluginSet` can expose multiple capabilities: index, query, storage, Restlet, Jetty, and ML (`PluginSet.java:43-133`). `PluginSet.getName()` is the provider identity and should be unique.
- **VERIFIED IN CODE:** a plugin receives the platform proxy by injection/`PluginBase`; the proxy exposes query, storage, index, settings, and user resolution (`DicooglePlatformInterface.java`; `DicooglePlatformProxy.java:200-218`).
- **VERIFIED IN CODE:** the SDK offers no “agentic” or “pipeline” extension point. `MLProviderInterface` is explicitly experimental/unstable (`MLProviderInterface.java:24-46`) and is not a catalogue, validator, or orchestration host. It is not injected with the platform proxy.
- **VERIFIED IN CODE:** the two viable external HTTP surfaces are:
  1. `JettyPluginInterface.getJettyHandlers()` returning a `HandlerList`; this gives direct access to `org.eclipse.jetty.server.Request` and therefore to `platform.getUser(request)`.
  2. Restlet `PluginSet.getRestPlugins()`, mounted under `/ext/<toString()>`; this has no core auth and does not expose the Jetty request directly.

**Implication:** a custom Jetty handler under a dedicated namespace is the better external API surface, provided its mounting and restart behavior pass runtime checks. Restlet is a fallback, but it is public by default and its path is coupled to `toString()`.

## 5. Lifecycle, deployment, and HTTP serving behavior

- **VERIFIED IN CODE:** startup order is plugin load -> settings -> built-in default storage -> proxy injection -> Restlet mount -> Jetty mount (`PluginController.java:116-135`). `Main` initializes `PluginController` before `ControlServices` (`Main.java:169-176`).
- **VERIFIED IN CODE:** `enable()` is not called at startup. It is invoked only by the admin `PluginsServlet` (`PluginsServlet.java:144-196`). Selection uses `isEnabled()`. A plugin that defaults to disabled will not serve through core selection paths.
- **VERIFIED IN CODE:** `stopPlugin` and `startPlugin` are no-ops; `disable()` does not unmount HTTP handlers (`PluginController.java:330-346`). There is no plugin reload.
- **VERIFIED IN CODE:** plugin `shutdown()` is called only if `archive.call-shutdown` is true (`Main.java:193-201`), which is false by default (`ArchiveImpl.java:49-50`). Do not rely on shutdown for durable writes or cleanup.
- **VERIFIED IN CODE:** Jetty plugin handlers are added via `DicoogleWeb.addContextHandlers`, which adds the handler and CORS only. It does not authenticate, check collisions, or provide an unregister path (`DicoogleWeb.java:450-455`). `stopPluginWebServices` removes only the core Restlet plugin handler, not arbitrary Jetty handlers (`:457-461`).
- **VERIFIED IN CODE:** when web autostart is false, `ControlServices.getWebServicePlatform()` can return null and `PluginController.initJettyInterface()` can NPE while adding handlers (`ControlServices.java:50-85,220-222`; `PluginController.java:283-297`). This is source-level, not runtime-tested.
- **INFERENCE / REQUIRES RUNTIME VERIFICATION:** when autostart is true, Jetty handlers are added after the server starts. Whether the handler starts reliably and whether it is lost after a web-server restart must be tested. A new `DicoogleWeb` does not rerun `initJettyInterface`; Restlet resources are static and are more likely to survive restart. “Works after restart” cannot be assumed.

**Implication:** the plugin must own its lifecycle, own its executor, persist per operation rather than at shutdown, and be tested in both autostart modes and across a web-server restart. If the deployment cannot guarantee those tests, the external-plugin route is not suitable for production execution.

## 6. Authentication and authorization

- **VERIFIED IN CODE:** core routes are registered in `DicoogleWeb` with `Needs.authenticated()`, `Needs.admin()`, or `Needs.role(...)` (`DicoogleWeb.java:134-256,362-376`). `/ext/*` is registered with `null` needs (`:180`), so Restlet plugin endpoints are public.
- **VERIFIED IN CODE:** `AuthenticatedFilter` obtains a token from an `Authorization` header (optional `Dicoogle ` or `Bearer ` prefix) or the `DICOOGLE_SESSION` cookie, resolves it through `Authentication`, and injects `dicoogleUser` (`AuthenticatedFilter.java:110-193`). `allow-unauthorized=true` bypasses core auth (`DicoogleWeb.java:140,370-372`).
- **VERIFIED IN CODE:** plugin handlers and `/ext` resources are outside that filter. A Jetty plugin can call `DicooglePlatformInterface.getUser(Request)` and enforce its own policy; a Restlet resource has to parse the token itself.
- **VERIFIED IN CODE:** no per-patient, per-study, per-series, or per-instance authorization exists in core search/WADO/dump/file paths. Only user-specific export presets and web-UI plugin roles show an object-like check (`PresetsServlet.java:72-76,99-103`; `WebUIServlet.java:101-111`).
- **VERIFIED IN CODE:** `DicoogleUser` exposes username, admin flag, and roles. There is no fine-grained permission model for DICOM objects in the SDK.

**Implication:** the plugin must implement its own authentication, role checks, and any object-level policy. In a clinical deployment, the lack of core object-level authorization means the plugin cannot claim that its results respect the same access rules as the underlying PACS unless it derives and enforces those rules independently. `allow-unauthorized` must be treated as a deployment risk, not a substitute for the plugin’s policy.

## 7. Tool catalogue and provider discovery

- **VERIFIED IN CODE:** live provider inventory is available through the platform: query provider names, query plugins, storage plugins, and query/storage requests (`DicooglePlatformInterface.java`; `ProvidersServlet.java:43-85`). This is a name-level inventory, not a capability catalogue.
- **VERIFIED IN CODE:** the SDK has no query-grammar descriptor, field dictionary, tool schema, preconditions, side-effect class, or cost model. Any catalogue must be supplied by the plugin.
- **VERIFIED IN CODE:** `SearchResult.extraData` is an untyped map, and field availability is provider-dependent (`SearchResult.java:33-97`). A catalogue cannot declare a field as universally present without deployment-specific verification.
- **VERIFIED IN CODE:** `MLProviderInterface` is not a reliable catalogue source. It is explicitly unstable and not included in proxy injection. Treat it at most as an optional, experimental adapter, never as the catalogue/orchestration substrate.

**Implication:** implement a versioned, plugin-owned catalogue that combines static tool definitions with live provider names/state. Declare optional fields and preconditions explicitly. Do not advertise a capability merely because a provider name exists; verify it at deployment.

## 8. DICOM object selection and metadata access

- **VERIFIED IN CODE:** a stable reference should be UID-based. The core supports patient/study/series/instance DIM levels and SOP Instance lookup (`DIMGeneric.java`; `WadoServlet.java`; `SocketUtil`/storage paths). File paths and storage URIs are provider-specific and should not be the durable identity of an input.
- **VERIFIED IN CODE:** `DIMGeneric` groups search results by `PatientID` (fallback `PatientName`), `StudyInstanceUID`, and `SeriesInstanceUID`; missing identifiers become empty strings and missing SOP UID becomes `"no uid"` (`DIMGeneric.java:63-77,147-198`). It is an in-memory view, not a validated catalogue.
- **VERIFIED IN CODE:** `Series.addImage` appends without de-duplication (`Series.java:91-95`). Duplicate query hits can create duplicate instance entries.
- **VERIFIED IN CODE:** `StorageInterface`/`StorageInputStream` provide URI-based stream access; `PluginController.resolveURI` returns the first enabled storage handler’s iterable, or an empty list with a full-URI error log (`PluginController.java:422-434`; `StorageInterface.java:37-168`; `StorageInputStream.java`).
- **VERIFIED IN CODE:** dcm4che2 `2.0.29` is the primary object model. Header-only reads are possible with a stop handler at PixelData (`StopTagInputHandler.java:57-61`); dcm4che2 has no include-pixel-data flag (`DicomInputStream.java:356-400,495-532,561-590`). Header-only behavior must still be tested with representative files.
- **VERIFIED IN CODE:** the index is not a canonical DICOM source. Metadata must be obtained from a storage stream/header parse or from provider-specific `extraData`. No VR/length/transfer-syntax/SOP-class validation is performed by `DIMGeneric`.

**Implication:** define a canonical `DicomRef` with Study/Series/SOP UIDs plus an optional frame range and provider/level metadata. Resolve it through storage, parse headers with bounded reads, validate identifiers, and deduplicate by SOP UID. Never rely on file paths as durable references, and never assume `DIMGeneric` has already validated the object.

## 9. Query behavior and data integrity

- **VERIFIED IN CODE:** `QueryInterface.query` is provider-defined; the standard UI uses Lucene syntax, but the SDK does not guarantee a common grammar (`QueryInterface.java:33-52`).
- **VERIFIED IN CODE:** `PluginController.queryAll` dispatches one task per enabled provider and `JointQueryTask.get()` concatenates results without de-duplication (`PluginController.java:578-640`; `JointQueryTask.java:67-76`).
- **VERIFIED IN CODE:** provider `RuntimeException` is caught in `getTaskForQuery`/`getTaskForQueryDim` and converted to an empty list (`PluginController.java:643-699`). `QueryException` and `QueryParseException` are runtime exceptions. An empty aggregate result therefore does not prove that no matching DICOM objects exist.
- **VERIFIED IN CODE:** `SearchServlet` applies `offset`/`psize` after collecting the iterable (`SearchServlet.java:253-274`). There is no SDK pagination/pushdown contract. `psize` defaults to `Integer.MAX_VALUE`; `numResults` is a total traversed count, not a page count.
- **VERIFIED IN CODE:** the proxy `queryAll` uses `DimLevel.INSTANCE` while core has an overload that can dispatch to every query provider (`DicooglePlatformProxy.java:132-140`). Non-DIM providers can behave differently between those paths.

**Implication:** the agent should use explicit provider names, record per-provider success/failure, enforce a bounded result limit, deduplicate by SOP UID, and treat empty results as “no complete answer” rather than “not present.” It should not use `numResults` as a validated total or assume offset-based paging is stable across providers.

## 10. Pipeline orchestration and task management

- **VERIFIED IN CODE:** core task pools are fixed-size with unbounded queues: query 4 threads, general 4, ML 1 by default (`PluginController.java:91-98`; `TaskManager.java:35-50`). There is no timeout, backpressure, queue bound, or query-task registry.
- **VERIFIED IN CODE:** `Task` is a `FutureTask` with UUID, name, timestamps, completion hooks, and optional progress (`Task.java:35-124`). `JointQueryTask` tracks completion and concatenates results; its integer progress division is coarse (`JointQueryTask.java:35-103`).
- **VERIFIED IN CODE:** only index tasks are tracked by `RunningIndexTasks` and `/index/task`; there is no equivalent query/pipeline status endpoint (`RunningTasksServlet.java`).
- **VERIFIED IN CODE:** provider failures can be swallowed per provider, while `JointQueryTask.get()` can propagate a non-swallowed `ExecutionException`. There is no structured partial-result model.

**Implication:** do not use the core task manager as the agent’s run engine. The plugin should own a bounded `ExecutorService`, a run state machine, timeouts, cancellation, idempotency keys, and a durable run/event log. Each provider/subtask must have an explicit outcome (success, empty, failed, timeout, invalid input) recorded independently.

## 11. Artifact store and run persistence

- **VERIFIED IN CODE:** the default file storage plugin handles `file:` URIs for reading but its `store` methods return null (`DefaultFileStoragePlugin.java:107-114`). It is not an artifact sink.
- **VERIFIED IN CODE:** the platform exposes the archive main directory through `ServerSettingsReader`, and plugin settings live under `Plugins/settings/<set-name>.xml` (`ServerSettingsReader.java:62-200`; `PluginController.java:153-167`). There is no artifact-store API, retention API, locking service, or database abstraction.
- **VERIFIED IN CODE:** the plugin settings holder is `XMLConfiguration`, suitable for configuration, not for concurrent run records (`ConfigurationHolder.java:30-52`).
- **VERIFIED IN CODE:** no query-history/audit retention policy was found. `QueryHistorySupport` serializes query history to `QueryHistory.ser` but had no current caller in the searched source; it is not an audit store.

**Implication:** create a dedicated, configurable agent data root outside `index/`, `confs/`, `users.xml`, and plugin settings. Use content-addressed artifacts (SHA-256), write to a temporary file and atomically rename, maintain a separate append-only run/event index, and enforce retention/cleanup. Do not write run state only at shutdown, because shutdown is not guaranteed. Avoid using the default storage plugin for writes. If the deployment requirements include encryption, access control, or deletion guarantees, the plugin must implement them; core provides none.

## 12. Validation, provenance, and auditability

- **VERIFIED IN CODE:** the core has no DICOM validation, anonymisation/de-identification, schema validation, or audit service. `DIMGeneric` only groups metadata; the default storage only reads. No JSON Schema/Bean Validation dependency was found.
- **VERIFIED IN CODE:** logging is SLF4J to Log4j2 with console and rolling files, no MDC/correlation ID, no audit log, and no PHI redaction (`log4j2.xml:1-29`). Rolling is startup + 2.0 MB; no max-history policy was found.
- **VERIFIED IN CODE:** core logging can include full queries at INFO (`SearchDicomResult.java:94`), SOP UID/provider data to stderr (`Information.java:72`), file URIs at ERROR (`FileDownloadUtils.java:74`), query strings at DEBUG (`ExportServlet.java:89`), and file paths (`DefaultFileStoragePlugin.java:84-97,180`; `PluginController.java:427,432`).

**Implication:** “verifiable” must be implemented by the plugin. At minimum:

- schema-validate tool inputs/outputs;
- verify DICOM UIDs/SOP class/transfer syntax and required attributes;
- reject invalid objects with explicit error codes;
- record tool ID/version, input `DicomRef`, parameter hash, provider/version, start/end time, outcome, output artifact hashes, and user/role;
- use an append-only, tamper-evident event chain (for example SHA-256 chaining) if the verification claim requires it;
- redact PHI from logs and store explicit error metadata separately;
- never claim audit coverage inherited from Dicoogle, because none was found.

## 13. Recommended external-plugin design and feasibility verdict

### Recommended shape for a controlled pilot

1. **One plugin set, one JAR.** Implement `PluginSet` and `JettyPluginInterface`. Use a dedicated namespace such as `/agentic/*`. Do not overload query/index/storage semantics.
2. **Authenticate every request.** Resolve `platform.getUser(request)`, require a non-null user, enforce a configured role, and fail closed. Do not rely on `/ext` or `AuthenticatedFilter`. Treat `allow-unauthorized` explicitly.
3. **Static catalogue + live inventory.** Ship versioned tool contracts and schemas; merge with live provider names/state at runtime. Mark provider-dependent fields and stability levels. Exclude unstable ML from the core contract.
4. **Canonical DICOM references.** Use Study/Series/SOP UIDs and explicit level/provider data; resolve through storage; parse headers with a bounded stop at PixelData; deduplicate by SOP UID.
5. **Plugin-owned execution.** Use a bounded executor and a persisted run state machine with idempotency keys, timeouts, cancellation, and per-provider outcomes. Do not use the core unbounded task pools as the run engine.
6. **Plugin-owned artifact store.** Use a dedicated configured root, content-addressed files, atomic writes, an append-only run/event index, and retention rules. Do not use `DefaultFileStoragePlugin.store` and do not write into core index/config directories.
7. **Explicit provenance.** Use a hash-chained event log if tamper evidence is required. Record user, input refs, tool versions, parameters, outputs, and failures. Redact PHI from logs.
8. **Packaging discipline.** Target Java 8; depend on the SDK and core at runtime; do not shade SDK/Jetty/dcm4che classes; verify with the deployed classloader.
9. **Lifecycle discipline.** Do not depend on `enable()` at startup, `disable()` to unmount, start/stop, or shutdown. Persist per operation and make the handler safe for a web restart or a failed mount.
10. **Runtime acceptance tests.** Test autostart on/off, Jetty late mounting, web restart, classloader behavior, auth, restart recovery, artifact atomicity, and DICOM header-only reads with synthetic data.

### Feasibility conclusion

- **Read-only catalogue/validation endpoint:** feasible today with moderate effort; the external plugin can expose an authenticated Jetty API and read provider inventory.
- **Pipeline orchestration and artifact persistence:** technically feasible, but it is a plugin-owned subsystem, not a platform capability. It requires careful concurrency, storage, retention, and provenance design.
- **Verifiable clinical execution with inherited authorization/audit:** not feasible as an unmodified external plugin. Core has no object-level authorization or audit, and lifecycle guarantees are insufficient. This needs a core/SDK enhancement, a deployment-specific policy layer, or a separate service that owns identity, policy, and audit.
- **Long-term production choice:** use the external plugin for a bounded pilot only if the runtime checks pass. If the agent becomes a governed platform capability, move the stable contract into the core/SDK or operate it as a sidecar with explicit integration, rather than relying on plugin lifecycle and public `/ext` behavior.

## 14. Investigation limitations and required runtime checks

### Limitations

- This was a static, read-only investigation. No build, test, service start, DICOM transfer, PACS query, or patient-data handling was performed.
- The working tree is dirty. Evidence reflects the current tree; the tracked code diffs inspected were formatting-only, but a submodule pointer change was not audited.
- The deployed query provider grammar, `SearchResult.extraData` contents, storage schemes, index completeness, `allow-unauthorized`, web autostart, `call-shutdown`, and classloader behavior are environment-specific.
- The existing untracked reconnaissance reports were treated as prior context, not primary evidence. No sensitive content from them is reproduced here.
- The absence findings are limited to the searched core/SDK/resources scope; third-party plugin behavior may add capabilities.

### Required runtime checks before commitment

1. Mount a Jetty plugin handler with `web-server.autostart=true`; confirm it starts and serves.
2. Repeat with autostart disabled; confirm whether the null web platform breaks plugin loading and whether a later mount is possible.
3. Restart the web server; confirm whether the Jetty handler survives, is lost, or duplicates.
4. Verify JSPF parent-first classloading with the actual deployed fat JARs and plugin dependencies.
5. Inventory deployed query providers, their grammars, and the exact `extraData` fields, including UID completeness and duplicate behavior.
6. Verify whether `SearchResult.extraData` contains rows/columns/frame counts/transfer syntax or whether every header must be opened.
7. Verify the deployment’s `allow-unauthorized` setting and the plugin’s intended behavior under it.
8. Confirm the permitted artifact root, filesystem permissions, backup/retention policy, and whether derived artifacts may contain PHI.
9. Verify whether `archive.call-shutdown` is enabled and whether any clean shutdown assumption is safe.
10. Test header-only DICOM parsing and pixel-data avoidance on representative synthetic studies, including compressed transfer syntaxes.
11. Test authentication/authorization for the plugin’s own routes, including missing, malformed, expired, and role-insufficient tokens.
12. Test failure injection: provider exception, empty result, malformed DICOM, missing storage handler, concurrent runs, cancellation, and crash recovery.

Until those checks pass, the correct conclusion is **“feasible as a controlled external-plugin pilot; not yet demonstrated as a production-grade verifiable agent.”**
