# Dicoogle external-plugin reconnaissance — Phase A+B checkpoint

Date: 2026-10-25  
Repository: `/home/jotalma13/Documents/Bolsa/dicoogle`  
Scope: how a DICOM-native pipeline agent could be delivered as an external Dicoogle plugin  
Mode: read-only source/build inspection. No repository edits, builds, tests, services, PACS queries, patient data, logs, settings files, user files, or secrets were opened.

## Evidence labels used

- **VERIFIED IN CODE** — directly observed in repository source or build files.
- **VERIFIED IN DEPENDENCY BYTECODE/SOURCE** — observed in a locally available dependency artifact.
- **DOCUMENTED ONLY** — stated in repository documentation, not independently exercised.
- **INFERENCE** — follows from code but requires a runtime or deployment assumption.
- **REQUIRES RUNTIME VERIFICATION** — cannot be settled by static inspection.
- **NOT FOUND IN SEARCHED SCOPE** — absence after the stated search, not proof of impossibility.

## 1. Checkout and build facts

- **VERIFIED IN CODE / GIT:** checkout is on branch `dev`, HEAD `1ed8c3ad1a6c22ac941265b178cb66e993ab5b87`, `git describe` = `3.3.4-103-g1ed8c3ad`, project version `3.6.1-SNAPSHOT`. This is the original Dicoogle v3 codebase.
- **VERIFIED IN GIT:** working tree is dirty: 11 tracked modified files, 0 staged, 40 untracked files. The untracked set includes `Plugins/`, `confs/`, `index/`, logs, `users.xml`, `roles.xml`, `tags.xml`, and two pre-existing reconnaissance Markdown files. Those sensitive/runtime artifacts were not inspected.
- **VERIFIED IN CODE / DIFF:** the 11 tracked code diffs inspected are formatting-only (wrapping/whitespace) plus one `dicoogle-next` submodule pointer change. No behavioral change to the evidence below was observed in those diffs.
- **VERIFIED IN CODE:** root `pom.xml:6` version; `pom.xml:151-152` modules are only `dicoogle` and `sdk`; `pom.xml:11-13` set Jetty `9.0.3.v20130506`, Restlet `2.1.2`, dcm4che `2.0.29`; `pom.xml:23-24` Java source/target `1.8`.
- **VERIFIED IN CODE:** `dicoogle/pom.xml:446` depends on `pt.ua.ieeta:dicoogle-sdk:${project.version}`. The backend consumes the SDK, not the reverse.
- **VERIFIED IN CODE:** `dicoogle/pom.xml:117-153` and `sdk/pom.xml:58-80` use Maven Shade. Core is a fat runnable JAR. Local `Plugins/lucene-3.5.1.jar` and `Plugins/filestorage-3.5.1.jar` are also large fat JARs (about 8,204 and 5,225 entries), report `Build-Jdk-Spec: 21`, and embed SDK, Jetty, and dcm4che2 classes.
- **VERIFIED IN CODE:** local environment uses JDK 21; CI uses JDK 8/11. Compiling/running a plugin for this deployment should target Java 8 bytecode unless the deployment owner explicitly changes that constraint.
- **VERIFIED IN DEPENDENCY BYTECODE:** JSPF/ClassWorlds registers each plugin JAR as a realm whose parent is the core application classloader. Plugin class loading is parent-first. The plugin realm contains only that JAR. Consequently, core/SDK/Jetty/dcm4che classes already present in the parent win over copies inside a fat plugin JAR. Conflicting bundled versions are therefore risky; bundling the SDK is redundant. Runtime confirmation is still required for the deployed packaging.
- **VERIFIED IN CODE:** no JSON Schema or Bean Validation dependency was found in the Maven files. Jackson is present for core settings/JSON; json-lib is also used. JUnit 4 is the test framework in the root modules.

## 2. Plugin discovery, settings, and lifecycle

- **VERIFIED IN CODE:** discovery is JSPF/`PluginSet` based. `dicoogle/.../plugins/PluginFactory.java:41-52` calls `PluginManagerFactory.createPluginManager()`, `pm.addPluginsFrom(pluginDirectory.toURI())`, then `PluginManagerUtil.getPlugins(PluginSet.class)`. README documents one plugin set per JAR in `Plugins/`.
- **VERIFIED IN CODE:** `sdk/.../PluginSet.java:43-133` defines optional capability collections: index, query, storage, Restlet, Jetty, and ML. A single plugin set can expose several capabilities. Getters are expected to return immutable, thread-safe collections.
- **VERIFIED IN CODE:** settings are created before platform injection: `PluginController.configurePlugins()` creates `Plugins/settings/<set-name>.xml` via `ConfigurationHolder` and calls `applySettings` (`PluginController.java:153-175`, `244-257`). `PluginBase` or `PlatformCommunicatorInterface` receives the proxy later (`PluginController.java:226-242`, `PluginPreparer.java:44-80`).
- **VERIFIED IN CODE:** `PluginController` construction order is load plugin sets -> configure settings -> add built-in `DefaultFileStoragePlugin` -> create proxy/preparer -> inject platform -> init Restlet -> init Jetty (`PluginController.java:116-135`).
- **VERIFIED IN CODE:** `enable()` is not called during startup. It is invoked only by the admin `PluginsServlet` POST (`PluginsServlet.java:144-196`). Startup and runtime selection use each plugin's `isEnabled()` (`PluginController.getQueryPlugins`, `getStoragePlugins`, etc.). A plugin that starts disabled will be filtered out.
- **VERIFIED IN CODE:** `stopPlugin` and `startPlugin` are no-ops (`PluginController.java:330-346`). `disable()` does not unmount HTTP handlers or stop query/storage routing.
- **VERIFIED IN CODE:** plugin `shutdown()` is called only from `PluginController.shutdown()`, and `Main` invokes that shutdown hook only when `archive.call-shutdown` is true (`Main.java:193-201`). The default is false (`ArchiveImpl.createDefault`, lines 49-50). Do not rely on shutdown callbacks for durable state or cleanup by default.
- **VERIFIED IN CODE:** plugin settings use Apache Commons Configuration `XMLConfiguration` (`sdk/.../settings/ConfigurationHolder.java:30-52`). This is a plugin-set-scoped XML file, not a general artifact store.

## 3. HTTP surfaces and their lifecycle

### Core routes and authentication

- **VERIFIED IN CODE:** all core routes are registered centrally in `DicoogleWeb` (`DicoogleWeb.java:134-256`). `createServletHandler(servlet, path, Needs)` (`:362-376`) adds CORS and optionally `AuthenticatedFilter`.
- **VERIFIED IN CODE:** `/ext/*` is mounted with `null` needs, i.e. no core auth (`DicoogleWeb.java:180`). `/legacy/*` is authenticated (`:183-184`). `/experimental/*` is also public (`:233`).
- **VERIFIED IN CODE:** plugin Restlet resources are attached under `/ext/<resource.toString()>` by `PluginRestletApplication.java:59-62`; path naming is therefore tied to `toString()` and there is no central namespace/collision check.
- **VERIFIED IN CODE:** plugin Jetty handlers are attached through `DicoogleWeb.addContextHandlers(HandlerList)` (`DicoogleWeb.java:450-455`), which only adds the handler and CORS. It does not add authentication, does not check path collisions, and does not start a separate security boundary.
- **VERIFIED IN CODE:** `AuthenticatedFilter` accepts a token from `Authorization` (optional `Dicoogle ` or `Bearer ` prefix) or the `DICOOGLE_SESSION` cookie, resolves it through in-memory `Authentication`, injects the user as request attribute `dicoogleUser`, and enforces `needsAdmin`/`needsRole` (`AuthenticatedFilter.java:110-193`). `allow-unauthorized=true` bypasses the core filter entirely (`DicoogleWeb.java:140`, `370-372`).
- **VERIFIED IN CODE:** the SDK exposes `DicooglePlatformInterface.getUser(org.eclipse.jetty.server.Request)` and `getUser(String token)` (`DicooglePlatformInterface.java`; implementation `DicooglePlatformProxy.java:200-218`). A Jetty plugin can therefore resolve the authenticated user without importing core auth classes, but it must enforce its own endpoint policy. `getUser` does not apply `allow-unauthorized` logic by itself.
- **VERIFIED IN CODE:** no per-patient/study/series/instance authorization was found. Core search, dump, WADO, image, and file paths require authentication at the endpoint level only (or are public in the `/ext` case), and do not check whether the user may access the selected DICOM object. The only object-like authorization found was for user-specific export presets (`PresetsServlet.java:72-76,99-103`) and web-UI plugin roles (`WebUIServlet.java:101-111`).

### Jetty-plugin lifecycle risk

- **VERIFIED IN CODE:** `Main` initializes `PluginController` before `ControlServices` (`Main.java:169-176`). `PluginController.initJettyInterface()` calls `ControlServices.getInstance().getWebServicePlatform()` while mounting plugin handlers (`PluginController.java:283-297`).
- **VERIFIED IN CODE:** `ControlServices` auto-starts the web server during construction if `web-server.autostart` is true (`ControlServices.java:50-85`). `getWebServicePlatform()` returns the `DicoogleWeb` field, which is null when the web server was not started (`ControlServices.java:220-222`).
- **INFERENCE / REQUIRES RUNTIME VERIFICATION:** if `web-server.autostart=false` and a plugin exposes Jetty handlers, `jettyServer` can be null and `addContextHandlers` can throw. If autostart is true, plugin handlers are added after the server has already started; Jetty dynamic-add behavior and startup ordering need a runtime test.
- **INFERENCE / REQUIRES RUNTIME VERIFICATION:** restarting the web server creates a new `DicoogleWeb`, but `initJettyInterface` is not rerun. Dynamic Jetty plugin handlers are therefore likely lost on web-server restart. Restlet plugin resources are held in a static list and are reattached when a new `DicoogleWeb` is created, so they are more resilient to restart, at the cost of public `/ext/*` paths and fragile naming.
- **VERIFIED IN CODE:** `stopPluginWebServices()` removes only the core `/ext` plugin handler, not arbitrary handlers added by `addContextHandlers` (`DicoogleWeb.java:457-461`). There is no unregister path for a Jetty plugin's handlers.

## 4. Query semantics

- **VERIFIED IN CODE:** `QueryInterface.query(String, Object...)` returns a possibly lazy `Iterable<SearchResult>`; the query grammar is explicitly provider-defined, with Lucene being the only grammar documented as compatible with the standard UI (`sdk/.../QueryInterface.java:33-52`).
- **VERIFIED IN CODE:** `SearchResult` contains a storage `URI`, a score, and an untyped `HashMap<String,Object> extraData` (`sdk/.../SearchResult.java:33-97`). Fields are not guaranteed; there is no schema, type, unit, or completeness contract.
- **VERIFIED IN CODE:** `PluginController.queryAll` selects all enabled query providers and issues one task per provider (`PluginController.java:578-640`). `JointQueryTask.get()` concatenates provider results without deduplication (`JointQueryTask.java:67-76`). Provider failures are not aggregated into a structured error response.
- **VERIFIED IN CODE:** `getTaskForQuery` and `getTaskForQueryDim` catch `RuntimeException` and return `Collections.emptyList()` after logging a warning (`PluginController.java:643-699`). `QueryException` and `QueryParseException` extend `RuntimeException` (`sdk/.../utils/QueryException.java:24`, `QueryParseException.java:30`). A provider-side query failure therefore looks like an empty result to callers.
- **VERIFIED IN CODE:** `SearchServlet` applies `offset`/`psize` by iterating the full result iterable and skipping outside the window (`SearchServlet.java:253-274`). There is no pushdown/pagination contract in the SDK. `psize` defaults to `Integer.MAX_VALUE` (`:100`). The reported `numResults` is the total traversed count, not the returned page count.
- **VERIFIED IN CODE:** `DIMGeneric` groups `SearchResult.extraData` by `PatientID` (fallback `PatientName`), `StudyInstanceUID`, and `SeriesInstanceUID`; missing non-null-allowed identifiers become empty strings, and a missing SOP UID becomes `"no uid"` (`sdk/.../DIMGeneric.java:63-77, 147-198`). `Series.addImage` appends instances without de-duplication (`Series.java:91-95`); duplicate provider results can therefore duplicate instance entries.
- **VERIFIED IN CODE:** `DicooglePlatformProxy.queryAll(holder, query, params)` passes `DimLevel.INSTANCE` (`DicooglePlatformProxy.java:132-140`), while the core non-Dim `queryAll` overload can dispatch to every query provider. This creates a behavioral difference between core and proxy call paths for non-DIM providers. The agent should call explicit providers or verify the intended overload.
- **VERIFIED IN CODE:** no query-provider capability metadata, field dictionary, or query-grammar discovery API was found. `/providers?type=query|index|storage` returns names only (`ProvidersServlet.java:43-85`).

## 5. Storage and DICOM metadata

- **VERIFIED IN CODE:** `StorageInterface` exposes `getScheme()`, default `handles(URI)` by scheme equality, `at(URI,...): Iterable<StorageInputStream>`, default `get(URI,...)` returning the first matching item, `store(...)`, `remove(...)`, and optional `list(...)` (`StorageInterface.java:37-168`).
- **VERIFIED IN CODE:** `PluginController.resolveURI` iterates enabled storage plugins and returns the first handler's `at(...)`; on no match it logs the full URI and returns an empty list (`PluginController.java:422-434`). `getStorageForSchema` returns null on no match (`:448-463`). There is no structured retrieval error.
- **VERIFIED IN CODE:** the built-in `DefaultFileStoragePlugin` handles `file:` URIs, recursively enumerates directories, returns file-backed streams, and logs full filesystem paths at debug/error (`DefaultFileStoragePlugin.java:73-104, 161-218`). Its `store` methods return null (`:107-114`), so it is not a general artifact sink. Its `remove` deletes files (`:117-127`).
- **VERIFIED IN CODE:** `StorageInputStream` only guarantees `getURI()`, `getInputStream()`, and `getSize()` (`sdk/.../StorageInputStream.java`). The plugin owns stream closing.
- **VERIFIED IN CODE:** the SDK uses dcm4che2 `2.0.29` (`sdk/pom.xml:186-195`), and core also has dcm4che3-style imageio artifacts (`dicoogle/pom.xml:522-535`). The primary DICOM object model for a plugin is `org.dcm4che2.data.DicomObject`.
- **VERIFIED IN DEPENDENCY SOURCE:** dcm4che2 `DicomInputStream` has no include/exclude-pixel-data flag; its default parse reads attribute values, including PixelData, unless stopped (`DicomInputStream.java:356-400, 495-532, 561-590`). `StopTagInputHandler(Tag.PixelData)` stops at level 0 when the tag is at or after PixelData (`StopTagInputHandler.java:57-61`), which supports header-only reads for normal DICOM files. Header-only behavior should be verified with representative files at runtime.
- **VERIFIED IN CODE:** the index is not a canonical DICOM metadata source. Search results come from provider-dependent `extraData`; original headers must be read from a storage stream and parsed. `DIMGeneric`/`SearchResult` do not validate VR, length, transfer syntax, or SOP-class compatibility.
- **VERIFIED IN CODE:** server settings expose SOP classes, additional SOP classes, transfer syntaxes, and service settings through the read-only `ServerSettingsReader` (`ServerSettingsReader.java:132-200`). This is a read-only view suitable for preconditions, not a DICOM validator.

## 6. Concurrency, tasks, and failure behavior

- **VERIFIED IN CODE:** query tasks use a fixed pool with default 4 threads (`PluginController.java:94-95`), general tasks 4 (`:91-92`), ML tasks 1 (`:97-98`). `TaskManager` wraps `Executors.newFixedThreadPool` (`TaskManager.java:35-50`), which uses an unbounded queue. There is no built-in timeout, backpressure, or queue bound.
- **VERIFIED IN CODE:** `Task` is a `FutureTask` with UUID, name, creation time, completion hooks, and progress optional via `ProgressCallable` (`sdk/.../task/Task.java:35-124`). `JointQueryTask` tracks completed subtasks and concatenates results (`JointQueryTask.java:35-103`). Its `getProgress()` uses integer division (`:85`), so intermediate progress is usually 0.
- **VERIFIED IN CODE:** no registry for query tasks was found. `RunningIndexTasks` tracks index tasks only and exposes `/index/task` (`RunningTasksServlet.java`). There is no equivalent query-run status, cancellation API, correlation ID, or audit record.
- **VERIFIED IN CODE:** core query-provider RuntimeException is swallowed per provider (see section 4), while `JointQueryTask.get()` propagates the first `ExecutionException` from a non-swallowed failure. The external plugin should return explicit per-item/per-provider errors and not infer completeness from an empty result.

## 7. Configuration, logging, provenance, and artifacts

- **VERIFIED IN CODE:** server settings are read from `confs/server.xml` via `ServerSettingsManager` (`ServerSettingsManager.java:49-90`). The read-only SDK view exposes `archive.main-directory`, `archive.dim-provider`, `default-storage`, web `allow-unauthorized`, DICOM SOP classes, and transfer syntaxes (`ServerSettingsReader.java:62-200`).
- **VERIFIED IN CODE:** plugin settings are one XML file per plugin set under `Plugins/settings/` (`PluginController.java:153-167`; `ConfigurationHolder.java:30-52`). There is no built-in artifact directory API. A plugin can read `getArchiveSettings().getMainDirectory()` or use its own configured path, but must enforce its own sandboxing, retention, permissions, and cleanup.
- **VERIFIED IN CODE:** logging is SLF4J to Log4j2 with console and a rolling `dicoogle.log` at info level (`dicoogle/src/main/resources/log4j2.xml:1-29`). No MDC, correlation ID, audit log, immutable run record, or PHI redaction was found.
- **VERIFIED IN CODE:** current core code logs sensitive material in several places: full query strings at info (`SearchDicomResult.java:94`), SOP UID and providers to stderr (`Information.java:72`), file URI at error (`FileDownloadUtils.java:74`), query string at debug (`ExportServlet.java:89`), and file paths at debug/error (`DefaultFileStoragePlugin.java:84-97,180`; `PluginController.java:427,432`). An external plugin should be stricter than its surroundings and must not log patient identifiers, full queries, full URIs, or pixel data.
- **VERIFIED IN CODE / NOT FOUND:** a legacy `QueryHistorySupport` serializes query history to `QueryHistory.ser` (`QueryHistorySupport.java:37-108`), but no current caller was found in the searched source. It is not an audit or provenance mechanism. No retention policy or deletion API for run artifacts was found.

## 8. Implications for an external-plugin implementation

These are engineering implications, not yet a proposed final architecture.

- **Feasible HTTP extension points:** `JettyPluginInterface.getJettyHandlers()` and `PluginSet.getRestPlugins()`. A Jetty handler is the natural fit for a custom authenticated API because core `DicooglePlatformInterface.getUser(Request)` is available. A Restlet resource avoids the web-server-start ordering risk but is mounted publicly under `/ext/*`, uses `toString()` naming, and needs its own token parsing/auth.
- **Unique namespace is mandatory:** choose a dedicated prefix (for example `/agentic/*`) and avoid `/search`, `/searchDIM`, `/dump`, `/wado`, `/ext`, `/legacy`, `/ml`, `/webui`, `/experimental`, `/management`, `/plugins`, `/presets`, `/export`, `/indexer`, `/settings`, `/login`, `/logout`, `/user`, `/providers`, `/index`, `/dic2png`, `/roi`, `/dictags`, `/welcome`, `/admin`-style paths. There is no collision detection.
- **Authentication must be implemented:** plugin Jetty handlers are not behind `AuthenticatedFilter`; `/ext/*` is public. A plugin should resolve the user via the SDK, enforce authentication/roles, and explicitly decide how to honor `allow-unauthorized`. Do not expose DICOM metadata or artifacts anonymously by default.
- **Lifecycle must be self-contained:** do not assume `enable()` is called at startup, do not assume `disable()` stops traffic, do not assume `startPlugin/stopPlugin` work, and do not rely on `shutdown()` unless `call-shutdown` is enabled. Long-running work needs its own lifecycle, cancellation, and idempotent close.
- **Query results are provider-dependent and incomplete-looking on failure:** validate/limit provider choice, explicitly track per-provider errors, deduplicate by SOP Instance UID, and do not assume `numResults` is a complete count or that `extraData` contains every field.
- **Fetching series/instances needs explicit queries or storage scans:** the SDK has no first-class Study/Series aggregate contract for external consumers. `DIMGeneric` is an in-memory grouping helper, not a durable catalogue. A plugin must define its own selector and consistency rules.
- **Header-only metadata needs deliberate dcm4che2 handling:** use a bounded header read (e.g. stop at PixelData) and do not read pixels for catalogue/validation unless explicitly required.
- **Packaging should avoid shading core/SDK/Jetty/dcm4che classes:** JSPF parent-first loading means core versions win. Bundle only dependencies absent from the core parent, and verify at runtime.
- **Web-server lifecycle is a design constraint:** a Jetty-based plugin may not survive a web-server restart and may fail to load when web autostart is disabled. This must be tested before committing to the design, or the plugin must use the Restlet surface and accept its auth/path constraints.

## 9. Open runtime checks

1. Add a Jetty plugin handler when `web-server.autostart=true`: does the late `addContextHandlers` mount and start reliably?
2. Repeat with `web-server.autostart=false`: does plugin loading NPE, and can the handler be mounted later?
3. Stop and restart the web server: are dynamic Jetty plugin handlers restored, lost, or duplicated?
4. Do `PluginSet`/JSPF realm parent-first semantics behave as the bytecode indicates with the deployed fat JARs?
5. What exact `SearchResult.extraData` fields do the deployed Lucene/provider plugins return, and are SOP/Study/Series UIDs always present and stable?
6. Does the deployed index contain rows/columns/frame counts/transfer syntax, or must the plugin open every header?
7. Does the deployment set `allow-unauthorized`, and what should the plugin do then?
8. Can a plugin safely store derived artifacts under `archive.main-directory`, or is a separate configured path required by operations?
9. Which query-provider grammar(s) are deployed, and is the core Lucene grammar the only one that must be supported?
10. Is `call-shutdown` enabled, or must the plugin flush state during each request/background checkpoint?

## 10. Status

- Phase A: complete.
- Phase B: source-level investigation complete for build/lifecycle, HTTP/auth, query, storage/DICOM, concurrency, logging, configuration, and classloading.
- Phase B remains runtime-dependent for the ten items in section 9.
- Phase C and the final 14-section report are not started.
- No repository file was modified by this session. This checkpoint is written outside the repository at `/tmp/opencode/dicoogle-phase-a-b-reconnaissance.md`.
- Two pre-existing untracked reconnaissance Markdown files were found in the repository root. They were treated as prior context, not as primary evidence, and were not modified.
