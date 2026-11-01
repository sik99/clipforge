# ClipForge project brief

## Confirmed goals and constraints

ClipForge is a video-conversion server with a minimalist web client. Its primary purpose is practical experience with distributed system architecture, horizontal scaling, deployment, infrastructure, operations, observability, and debugging, supporting progression toward a Software Architect role. Application coding is secondary and can be delegated to agents.

- The owner intends to develop the server with ASP.NET Core; the runtime version remains open.
- The first server version will be a modular monolith. Its internal module design will follow decisions about technology, the external API, and the end-to-end workflow.
- V1 will run on the owner's computer and be accessible only from that machine. Access from other machines is deferred until a requirement calls for it.
- Design the server API with the future minimal client interaction in mind, including how a user request becomes API calls. Client application design and implementation are outside the current design scope.
- Keep the domain and initial product small. Evolve through user requirements, operational needs, hypotheses, measurements, owner decisions, and verification. A complex distributed deployment is not an initial requirement.
- Consequential system design and infrastructure decisions belong to the owner. See [collaboration rules](agents/collaboration.md) for the decision workflow.
- Use synthetic load; real users are not needed for the learning goals.
- Supported behavior should be production-relevant within its stated limits. A single-node version is acceptable to consider; local resources must not be presented as independent machines.
- Adding replicas with additional per-instance resource budgets is an intentional way to model added nodes, within actual host capacity. Container count alone does not establish added physical capacity or availability.
- Write repository documentation in English. Create directories only alongside meaningful content.

## Active phase

Documentation and design preparation only, with root `.editorconfig` and `.gitattributes` authorized for formatting and Git line-ending consistency. Collect further owner context into the appropriate documents during this session. Application code, tests, further tooling and configuration, dependencies, and deployment assets are deferred until authorized. Revisit the remaining agent setup suggestions when the application project is created. The owner has requested a Socratic design interview: begin with technology, API, and end-to-end workflow; address modular-monolith internals afterward. Use questions to expose unresolved decisions and withhold general architecture recommendations during this initial exploration.

The modular-monolith application shape and single-machine deployment with local-only access are confirmed. Deployment tooling, persistence design, and any future service roster remain open. These notes are preparation, not an implementation specification or backlog.

## First-version user flow and scope

- The user selects a local video file through the client, starts the process, and ultimately downloads the converted video to their computer.
- The desired output video format is the only conversion choice currently in scope. A single supported output format is acceptable for the first version, depending on implementation effort with FFmpeg; the actual output format and conversion-engine selection remain unresolved.
- Other video editing or conversion options are deferred until a new user requirement justifies them.
- The workflow must support adding output formats and conversion options. Conversion requires the user to commit the required options, including when only one output format is initially supported. Upload completion alone does not authorize conversion.
- The user may commit the required conversion options before or during upload. Once the upload is complete and the required options are committed, conversion proceeds without another user action. A future minimal client can support selecting the file and options followed by one explicit start action that initiates upload and then conversion.
- Committed conversion options cannot be changed in v1. Reconsider this capability later if it provides useful system-design learning.
- The v1 API will expose manual job cancellation, covering the job's upload and conversion work. The current minimal client will not use this capability. Cancellation semantics and its interaction with restart recovery remain to be specified.
- Whether upload may begin before option commitment and how download is initiated remain open. The agreed behavior does not select an API request structure or a client layout.
- The first minimal client may require the user to keep the page open. Restoring a previous conversion after refreshing or reopening the page is not a required first-client capability.
- After upload completion, accepted conversion work continues even if the client disconnects.
- If a server restart interrupts a conversion, the server automatically attempts that conversion again after startup. Recovery mechanisms and restart-related retry limits remain open.
- For conversion failures unrelated to server restart, stop the affected conversion without automatic retry. Store its failed outcome and the relevant circumstances explaining what happened, and log the full diagnostic context for later diagnosis and analysis. A subset of the failure information may be exposed through the API; its exact contents and the diagnostic capture design remain open.
- V1 must accept additional jobs while an earlier conversion is unfinished, but execute only one conversion at a time. Uploads and downloads can proceed independently of sequential conversion execution. Backlog limits, scheduling order, transfer concurrency limits, and overload behavior remain open.
- The API must report a job's current stage in v1; numeric conversion progress is not required. A small delay before the client learns of a stage change is acceptable. The exact stage/status model and update-delivery mechanism remain open.
- Update the server's recorded job state as soon as feasible after the preceding stage finishes, within the constraints of the chosen technology and architecture. This internal update goal does not drive technology selection; no quantitative internal-update latency target is selected.
- Interrupted uploads may restart from the beginning in v1; resumable uploads are not required. Cleanup of partial uploads and the relationship between a new upload attempt and previously committed options remain open.
- Shared access to conversions and results among trusted callers is acceptable in v1; per-user isolation is not required. Access is limited to the owner's computer; any application-level authentication or other access restrictions remain open.

## Converted-result retention

- Retention is configurable, with a default of seven days starting when the successful result becomes available.
- Repeated downloads are allowed and do not extend retention.
- The expiration deadline survives server restarts and is exposed through the API.
- Expiration ends availability for new downloads; physical file cleanup is separate and may follow later.
- Prefer allowing a download already in progress at expiration to finish. This is a tentative preference, conditional on implementation and code complexity; decide later when the expected implementation is clearer, potentially with a prototype.
- Cleanup timing and mechanism and the effect of configuration changes remain open. Partial uploads, cancelled jobs' source videos, and job metadata need separate retention decisions.

## Source-video cleanup

- Once the converted result is available for downloading, the uploaded source video becomes eligible for cleanup. The actual deletion timing and mechanism remain open.
- Source videos from failed conversions remain available for diagnosis and reproduction under a separately configurable retention policy. The default is 60 days from recorded conversion failure, with the deadline preserved across server restarts.
- Revisit the explicit deletion procedure and whether to add or defer a deletion API after storage selection. The removal mechanism is not yet selected.
- Source-video retention after cancellation remains open.

## Log retention and stored failure information

- Logs have their own configurable age at which they become eligible for deletion. The owner proposes a default of approximately two months; the agent recommends 60 days from each log entry's timestamp. The default and timing semantics await confirmation; cleanup granularity and mechanism depend on logging technology selection.
- Failure information forms part of job information and shares the job's retention policy below. Its representation remains open; no separate diagnostic-record component or retention policy is selected.
- V1 must retain complete job history long term, including failure information, to preserve data for future analytics and statistics. Motivating examples include product evolution and popularity, input/output format usage, and video sizes, potentially informing scaling, worker optimization, extensions, and optimization of popular conversions. These examples are not a finalized metadata schema or an exhaustive list of future questions.
- Analytics and aggregation implementation are deferred beyond v1. Retained fields remain open; defining every future analytical use is not a prerequisite for retaining the history. V1's existing synthetic-load learning scope remains unchanged.
- Job-history retention is operator-configurable, defaulting to no automatic expiration, with an optional finite retention period for finished jobs. Apply the policy to complete job information, including failures. Active jobs are not eligible for age-based history deletion. Revisit the default using observed history growth and future analytical needs; finite-retention timing and cleanup mechanisms remain open.
- The owner raised limits for active-job duration, conversion execution, and uploads/downloads. Policies, defaults, and consequences of timeouts are under discussion and are not yet selected.

## API compatibility during v1 development

- Before v1 is published, the server and client may be updated together, including changes that break older development clients. There is no requirement to support older clients against newer servers during this phase.
- Revisit API versioning and compatibility when v1 is finalized and development of the next version begins. No post-v1 compatibility policy is selected.
- The server's public API declarations are the authoritative contract definition; generate the API specification from those declarations. The specification format, generation tools and workflow, client generation, and checks that the current server and client agree remain open.

## Owner preferences and candidates

These express direction for discussion, not final architecture decisions.

| Area | Current direction | Still unresolved |
| --- | --- | --- |
| Server API implementation | Tentatively prefer ASP.NET Core Minimal APIs; revisit when choosing implementation details. | Final endpoint implementation approach and runtime version. |
| Conversion execution | Execute one conversion at a time, with uploads and downloads proceeding independently. In-process background conversion is a candidate within the confirmed modular monolith. | Boundaries, execution and scheduling mechanisms, and mechanisms for the required automatic recovery after server restart. |
| Client | Keep the required web client minimal. | Framework, hosting, interactions, and status updates. |
| Deployment tooling | Prefer running the server and client directly on the owner's computer if the selected stack permits it. Consider Docker later if a dependency, infrastructure technology, or distributed deployment calls for it. | Final deployment approach depends on the selected stack; operating environment, resource budgets, persistence, and operating procedures remain open. |
| Media storage | Consider a persistent local filesystem for a first single-node version. | Durability, cleanup, capacity limits, and access from future worker hosts. No object-store product is selected. |
| Job metadata | Consider SQLite initially, then reassess against observed limits and requirements; PostgreSQL is a possible successor. | Concurrency, recovery, work ownership, and migration consequences. Neither database is selected. |
| Communication | REST is an earlier preference; the owner is reconsidering client-server API styles and communication mechanisms. See the comparison in the design notes. | Client-server API style, file-transfer and status-delivery mechanisms, and contracts and communication between any future services. |
| API/client tooling | Consider OpenAPI and generated client code within the confirmed approach of generating the specification from server API declarations. Coordinated server/client changes before v1 publication are permitted as described above. | Specification format, generation tools and workflow, and agreement checks. Post-v1 versioning and compatibility are deferred to the next-version design. |
| Service specification tests | Reqnroll is a candidate; library recommendations are welcome. | Test boundaries, library choice, and scenarios. |
| Performance experiments | Expect repeatable benchmark/load scenarios. | Execution approach, workloads, tools, targets, timing, and which learning exercises to adopt. |

## Agent proposals

- Use FFmpeg as the conversion engine, subject to owner evaluation and dependency selection.
- Evaluate asynchronous HTTP streaming for uploads/downloads and bounded background processing for conversion.
- Evaluate object storage and independently deployed workers when requirements justify them. A broker, gateway, reporting service, and telemetry stack have not been selected.

The [design notes](design-notes.md) preserve the supporting rationale and candidate evolution triggers. They do not approve these proposals.

## Benchmark and learning ideas

The [benchmark and architecture learning ideas](learning-experiments.md) collect options for comparable measurements, video workloads, failure experiments, debugging, and the owner's own bottleneck predictions and retrospective evaluation. During version or stage planning and review, the owner chooses which ideas to use, defer, or skip. Their inclusion does not approve an experiment plan or implementation. Persisting detailed benchmark results, machine descriptions, and predictions remains an option to decide when useful.

[WHATSNEW.md](../WHATSNEW.md) is the human-facing place for very brief architecture summaries of released versions; it does not establish a versioning scheme or release schedule.

## Open questions for the requested grilling session

- **Product scope:** supported inputs, exact output format and conversion behavior, input-size limits, and server access/authentication needs. The basic local-file-to-downloaded-result flow and acceptance of shared access without per-user isolation are confirmed above.
- **Lifecycle:** when a job becomes eligible, job states, persistence, work ownership, queue limits and scheduling order, restart-related retry limits, duplicate attempts, cancellation semantics, failure information exposed through the API, diagnostic capture, and server restart recovery mechanisms. Continued conversion after client disconnection, automatic retry of conversions interrupted by server restart, no automatic retry for other conversion failures, a manual cancellation API unused by the minimal client, acceptance of overlapping jobs, and one conversion at a time with independent uploads and downloads are confirmed above.
- **Files:** input/output durability mechanisms, failed-source cleanup and explicit deletion mechanisms, source-video retention after cancellation, finite job-history expiration timing and cleanup, cleanup of complete and partial files, active downloads at result expiry, retention-configuration changes, and disk exhaustion behavior. Restarting interrupted uploads from the beginning, the converted-result retention policy, source cleanup eligibility after result availability, the 60-day default failed-source retention policy, and configurable job-history retention with no automatic expiration by default are confirmed above.
- **Interfaces and tests:** client technology and hosting, API contracts, status delivery, and service specification tests.
- **Operations:** operating environment and deployment tooling within the confirmed single-machine scope, resource budgets, load scenarios, latency/completion/error targets, and observability sufficient to explain results.
- **Evolution:** evidence or requirements that justify another database, object storage, a broker, independently deployed workers, additional replicas, or another service.

The product name is resolved as **ClipForge**. The remaining questions are discussion topics, not ready implementation tickets.
