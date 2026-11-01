# Design notes for future decisions

These notes collect technical rationale, agent proposals, and rationale for confirmed owner decisions. Consult the [project brief](project-brief.md) for decision status; proposals are not approved architecture. Evaluate the relevant assumptions during design; implementation remains deferred.

## Initial conversion and transfer model

Candidate: one application deployment handles upload/download, job submission/status, conversion, and result access, with responsibilities kept distinct in code. Background conversion may invoke a conversion-engine subprocess. A job-metadata database and a message broker solve different problems; asynchronous work alone does not require a broker.

Asynchronous HTTP streaming can handle concurrent uploads and downloads while conversion runs. Dedicated transfer background services are not required merely for concurrency. The upload request stays open until receipt finishes; its response need not wait for conversion. Reading a live request body must remain within the request lifetime. A background service is a lifecycle abstraction, not necessarily a dedicated thread.

For a local-file candidate, receive into temporary storage, finalize the complete input, then make conversion eligible. Stream completed results to clients on request. Completion notification and result transfer are separate concerns. Partial uploads, publication, job recovery, cleanup, and scheduling of sequential conversions still need explicit design.

Streaming avoids holding the entire file in application memory, but active transfers still consume connections, buffers, bandwidth, disk I/O, and CPU. Conversion competes for the same host resources.

HTTP `multipart/form-data` is not S3 multipart upload. The latter coordinates separately uploaded parts, completion, and retries/resumption. The owner accepts restarting interrupted uploads from the beginning in v1; resumability is not required. If a future requirement calls for resumability, evaluate established implementations rather than inventing a protocol.

## Timing of user commitment

The owner accepted allowing the user to commit conversion options before or during upload. Once the upload is complete and the required options are committed, conversion proceeds without another user action. For a minimal future client, selecting the file and options followed by one explicit start action can initiate upload and then conversion automatically. This does not select an API request structure, automatic upload on file selection, or download initiation behavior.

The intended user benefit is avoiding a mandatory return after a potentially long upload solely to authorize an already chosen conversion. The interaction should make the current stage and any remaining need for user action clear. The owner separately decided that committed options cannot be changed in v1; this capability may be reconsidered later if it provides useful system-design learning.

## Client connection and conversion lifetime

The owner decided that accepted conversion work continues after the client disconnects, once upload is complete. The first minimal client may still require the page to remain open and need not restore earlier work after reopening. This separates server processing lifetime from client connection lifetime. Result retention is specified separately below; the status-delivery mechanism remains open.

The owner also requires automatic retry after startup for a conversion interrupted by a server restart. Recovery therefore needs access to the uploaded input, committed options, and enough information to identify the interrupted work across that restart. The storage and recovery mechanisms and restart-related retry limits remain open.

For other conversion failures, the owner requires stopping the affected conversion without automatic retry, storing the failed outcome and relevant circumstances, and logging full diagnostic context for later diagnosis and analysis. A subset of failure information may be made available through the API. The API-visible details, recorded failure fields, logging technology, and job-history retention mechanisms remain open. This distinguishes failures that have been recorded from conversions interrupted by server restart and eligible for recovery.

The owner requires a manual cancellation operation in the v1 API for a [job](../GLOSSARY.md), which covers upload and conversion. The current minimal client will not use it. The owner's rationale is to expose control over work that the server can otherwise continue or retry independently of the client. Cancellation acknowledgment, stopping semantics, and its interaction with restart recovery remain to be specified; deliberate cancellation and interruption by restart need to be distinguished.

The owner requires accepting additional jobs while an earlier conversion is unfinished, with only one conversion executing at a time in v1. Uploads and downloads can proceed independently of sequential conversion execution. The owner wants to observe work parallelisation in practice before choosing to introduce simultaneous conversions. Backlog limits, scheduling order, transfer concurrency limits, and overload behavior remain open; no parallel-conversion experiment plan is selected.

The owner considers stage reporting sufficient for v1; the API need not provide numeric conversion progress. A small delay in client awareness of a stage change is acceptable. This is separate from updating the server's recorded job state, which should happen as soon as feasible after the preceding stage finishes, within the chosen technology and architecture's actual constraints. The owner explicitly does not want that internal update goal to drive technology selection. The exact stage/status model and update-delivery mechanism remain open; no quantitative internal-update latency target is selected.

## API contract authoring

The owner chose to generate the API specification from the server's public API declarations, making those declarations the authoritative contract definition. The supporting rationale is the small initial system, coordinated server/client updates before publication, and avoiding a separately hand-maintained specification alongside the implementation. OpenAPI and generation of the client's API integration code from that document remain tooling candidates; no generator is selected. ASP.NET Core Minimal APIs are the owner's tentative implementation preference, to revisit when needed.

The alternative is a separately authored OpenAPI document as the authoritative contract, with the server implemented against it, optionally using generated server interfaces or stubs. This makes the contract independently reviewable before implementation and can support separately coordinated teams, but requires maintaining the specification and ensuring the implementation follows it. Neither approach automatically verifies all runtime behavior or selects the client-generation tool.

ASP.NET Core supports [OpenAPI generation at runtime and build time](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/aspnetcore-openapi?view=aspnetcore-10.0). Generated documents still depend on accurate [endpoint metadata](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/include-metadata?view=aspnetcore-10.0), particularly for errors and streamed or binary bodies. API declarations should express the intended external contract; generation does not replace designing the workflow or reviewing that contract.

Other HTTP API description languages include [RAML](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md) and [API Blueprint](https://apiblueprint.org/documentation/specification.html). [TypeSpec](https://typespec.io/docs/emitters/openapi3/openapi/) offers a separate API-authoring language that can emit OpenAPI, rather than requiring OpenAPI to be abandoned as the interchange format. The agent's recommendation remains OpenAPI for the current ASP.NET Core HTTP API direction; the specification standard remains open, while authoring from server API declarations is confirmed. Event/message contracts can call for [AsyncAPI](https://www.asyncapi.com/docs); background conversion alone does not establish that requirement.

Candidate reasons to reconsider the authoritative contract definition are independently coordinated client/server teams that need agreement before implementation, external consumers or approval processes requiring a separately governed contract, multiple implementations of the same API, or recurring limitations that make expressing the intended contract in server declarations costly. Publishing v1, requiring compatibility, or distributing the application would not alone require switching to specification-first authoring.

A later switch could begin with the generated OpenAPI document as a reviewed baseline, then make a separately maintained contract authoritative and adapt the existing server to it. Server interfaces or transport types could be generated if suitable tooling supports the required API features. This is a tooling and integration migration with real effort, but it need not change the external API or rewrite conversion behavior. Compatibility policy is separate from contract authoring; contract checks remain necessary in either approach.

## Storage and independent workers

Persistent local files can support a deliberately limited single-node deployment. Host-local paths are not an artifact-access contract for workers on other hosts. Storage migration has costs and may change upload routing and recovery behavior.

Candidate for a later distributed phase: keep media in durable, independently accessible storage and job state/references in a database. The API can issue scoped, time-limited upload/download authorization; clients then transfer directly to storage. This is an option, not a selected upload path or storage product.

If a queue is introduced, messages can carry job identifiers, media references, and conversion parameters rather than video bytes. Workers need durable inputs accessible for retries on another host. Private local scratch is disposable staging; publish output durably before reporting success. Duplicate attempts and retries must not expose partial or conflicting results. Work ownership and hand-off semantics remain open.

Object APIs use independently addressable objects; path-like keys do not imply filesystem semantics. Remote streaming versus local staging depends on format seekability and workload. Per-host caches or pre-staging add coordination and locality concerns and are optional optimizations.

## Converted-result retention

The owner confirmed configurable converted-result retention with a default of seven days. This is an initial setting for the local learning environment that can be adjusted after observing disk use.

The confirmed policy counts retention from when the successful result first becomes available, permits repeated downloads without extending its lifetime, preserves its expiration deadline across server restarts, and exposes the deadline through the API. Retention bounds age rather than total storage usage; capacity limits remain a separate decision. Failed-source cleanup mechanisms, source retention after cancellation, partial-upload cleanup, job-metadata retention, active downloads at expiry, cleanup timing, and the effect of configuration changes remain open.

The owner also confirmed the distinction between expiration and physical cleanup: new downloads are unavailable at expiry, while file cleanup may follow later. The cleanup mechanism and acceptable delay remain open. [S3 lifecycle documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/troubleshoot-lifecycle.html) illustrates why asynchronous cleanup can lag expiration; this is supporting context, not a storage-technology selection.

For a download already in progress at expiration, the owner prefers allowing it to finish, but considers this a soft requirement dependent on implementation and code complexity. Defer the decision until the expected implementation is clearer; a prototype is an option for resolving it, not an approved experiment plan.

The owner decided that the source video becomes eligible for cleanup once the converted result is available for downloading. Source cleanup eligibility is therefore independent of the result's seven-day default retention period. Source videos from failed conversions must instead remain available for diagnosis and reproduction under the policy below. Actual source deletion timing and retention after cancellation remain open.

## Failed-source diagnostic retention

The owner confirmed a separately configurable retention policy for failed source videos, defaulting to 60 days from recorded conversion failure, with deadlines preserved across restarts. The explicit deletion procedure and the decision whether to add or defer a deletion API are deferred until storage selection.

Agent guidance for the later deletion design: removal should preserve the failure history and leave source availability accurately represented. The procedure depends on the selected storage and metadata design. A deletion API does not inherently require user accounts in the already accepted local, shared-access environment; its scope and access policy are separate decisions.

Logs have a confirmed independently configurable age for deletion eligibility. The owner's proposed default is approximately two months; the agent recommends 60 days from log entry timestamps, pending confirmation. Actual cleanup granularity depends on the chosen logging technology. [OWASP logging guidance](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html) calls for an explicit retention period for logs; it does not prescribe this project's default duration.

The phrase "diagnostic record" referred to the failed outcome and circumstances already required to be stored with a job, as distinct from chronological diagnostic logs. The owner confirmed that failure information is part of job information and shares the job's retention policy. It does not establish a separate component, storage representation, or retention policy. Example fields can be explored later without treating them as an agreed schema.

The owner requires long-term retention of complete job history in v1 to preserve the option of future analytics and statistics. The purpose includes observing product evolution and popularity, input/output format usage, and video sizes, potentially informing scaling, worker optimization, extensions, and popular conversion paths. These are examples, not an exhaustive set of questions or an agreed metadata schema. The owner does not want retention to depend on enumerating all future analytical uses now.

Analytics and aggregation implementation are deferred beyond v1. Aggregation is not universally preferable: it can reduce storage and simplify reporting, while retaining detailed history preserves the ability to ask questions involving recorded details not included in earlier summaries. No aggregate-only retention architecture is selected. Fields, sizing, and retention implementation remain open. Job-history retention includes failure information and is independent of video-file and log retention. [Microsoft's data-lifecycle guidance](https://learn.microsoft.com/en-us/azure/well-architected/cost-optimization/optimize-data-costs) ties retention to business requirements; [Google Analytics retention documentation](https://support.google.com/analytics/answer/7667196?hl=en-au) illustrates the distinction between event-level retention and aggregate reports.

The owner accepted configurable job-history retention with no automatic expiration by default for the local v1, plus an optional finite retention period for finished jobs. The policy covers failure information together with the rest of job information; active jobs are excluded from age-based history deletion. An arbitrary ten-year cutoff currently has no established business or operational basis. No automatic expiration preserves detailed history while analytics are deferred; history size and job volume can inform a later finite policy. The configuration representation, finite-retention start time, cleanup mechanism, and treatment of existing history after configuration changes remain open. Independent video-file and log retention policies are unchanged.

## Stage-duration and transfer limits

The owner raised safeguards against active jobs remaining open too long and asked about measuring and limiting conversion, upload, and download duration. No timeout policy or default is confirmed yet.

Agent proposals: use a configurable maximum elapsed execution time per conversion attempt, separate from queue waiting and HTTP request lifetime. On timeout, actually stop conversion execution and record the failure and diagnostics without automatic retry. For transfers, distinguish maximum total duration from lack of progress or an unacceptably low transfer rate. Prefer progress/rate safeguards for uploads and downloads, with total-duration caps considered against supported file sizes and expected throughput. A timed-out upload ends that transfer attempt; a timed-out download should not change a successfully converted job into a failed conversion. Abandoned pre-conversion work and excessive queue residence need separate policy decisions.

Server-side transfer duration can be measured, but completing a server response does not prove the client saved the file. [ASP.NET Core request-timeout middleware](https://learn.microsoft.com/en-us/aspnet/core/performance/timeouts) and [Kestrel transfer-rate controls](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/options?view=aspnetcore-10.0) provide relevant mechanisms; actual enforcement depends on the selected hosting and protocols. Timeout cancellation signals must be handled; terminating the wait for a subprocess does not itself terminate its work. Implementation choices and numeric defaults remain open.

## Host placement and API reachability

The owner decided to run v1 on their own computer with access limited to that machine. Access from other machines is deferred until a requirement calls for it. Allowing clients on other machines later does not inherently require distributing the server or changing its conversion API. It would introduce decisions about reachable addresses, listener and firewall configuration, caller access restrictions, and transport protection. Browser cross-origin configuration depends on the origins serving the client and API, rather than whether the browser is on another machine.

The owner's current deployment preference is direct execution if the selected stack consists of the server and client without dependencies that call for containers. Final deployment tooling therefore follows stack selection. Docker remains an option if a later dependency, infrastructure technology, or distributed deployment calls for it; it is not a v1 requirement.

## Measurement and topology limits

For a stored-file flow, measure each bulk-transfer leg: client upload, worker input read, worker output publication, and client download. Also measure queue wait and conversion. Transfer can dominate total turnaround; distinguish result-ready time from time until download completes. Some stages may overlap if formats and processing allow it.

Benchmark job concurrency together with encoder threading: one conversion may already use multiple cores. CPU quotas can cause periodic throttling and affect short-request tail latency. Interpret latency alongside resource usage, throttling, disk/network contention, queue age, completion time, and errors.

Several containers on one host share CPU, memory, storage bandwidth, and a failure domain. Additional replicas model more capacity only within real host headroom. A single-host lab can exercise real protocols but cannot demonstrate independent host failures, disks, or multi-host bandwidth. Record topology and resource budgets alongside results. The owner selected their own computer as the v1 host; deployment tooling and resource budgets remain open. Rented or managed infrastructure, if considered later, would change operator effort, cost, transfer latency, and network dependencies.

## Evolution triggers to evaluate

| Requirement or observed limit | Candidate response for an owner decision |
| --- | --- |
| Large uploads must resume or bypass application compute | Established resumable upload support or direct object-storage uploads; the application may remain a monolith. |
| Processing must execute or recover on other hosts | Independently accessible durable inputs/outputs and explicit artifact references. |
| Conversion resource use, restarts, or deployment disrupts the API | Independently deployed processing with explicit ownership and reliable hand-off. |
| Queue age or completion time misses agreed targets | Compare concurrency, encoder settings, and per-instance capacity before adding worker replicas. |
| Job-state contention, durability, or access requirements exceed the initial database's limits | Reassess persistence and migration; a replacement database is not preselected. |
| Several client-facing APIs need unified routing or shared edge policies | Evaluate a dedicated API gateway. A controller is an application API, not automatically a gateway. |
| Reporting/status queries have distinct load, ownership, or lifecycle needs | Evaluate separating those capabilities; they may remain in the application API. |

Requirements can justify infrastructure before a throughput bottleneck appears. Worker extraction alone does not mandate a gateway or reporting service. This table is a set of discussion triggers, not a service roadmap.
