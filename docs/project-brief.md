# ClipForge project brief

## Confirmed goals and constraints

ClipForge is a video-conversion server with a minimalist web client. Its primary purpose is practical experience with distributed system architecture, horizontal scaling, deployment, infrastructure, operations, observability, and debugging, supporting progression toward a Software Architect role. Application coding is secondary and can be delegated to agents.

- The server uses ASP.NET Core Minimal APIs. Use .NET 10 capabilities as the provisional design reference and choose the actual runtime at implementation setup, considering .NET 11 if its release status and toolchain compatibility make it suitable. Revisit earlier if a design or tooling choice requires version-specific functionality. This does not require supporting both runtimes.
- FFmpeg is the selected conversion engine. Invoke an independently installed executable for local v1; a redistributable GPL-enabled CLI build is permitted. The concrete build/version and installation method remain deferred to environment setup. Encoder directions and all third-party software obligations are maintained in the [dependency licensing guide](dependency-licensing.md).
- Publish original ClipForge code under MIT in a public GitHub repository. Actual publication and future Docker/image distribution procedures remain deferred; public source and build instructions do not by themselves require publishing a prebuilt image. See the [licensing guide](dependency-licensing.md#public-source-repositories-build-recipes-and-private-images).
- The first server version will be a modular monolith. Its internal module design will follow decisions about technology, the external API, and the end-to-end workflow.
- V1 will run on the owner's computer in the Czech Republic and be accessible only from that machine. Access from other machines is deferred until a requirement calls for it. Travel and changes of execution country require considering the territorial patent guidance in the [licensing guide](dependency-licensing.md).
- Store source and converted video files on a persistent local filesystem in v1. No separate media-storage service is selected for this version.
- SQLite is the selected v1 store for job information, including failure information and retained job history. The .NET provider, data-access approach, schema, and concrete versions remain open; apply the [dependency licensing checkpoints](dependency-licensing.md) when selecting the integration and artifacts.
- Design the server API with the future minimal client interaction in mind, including how a user request becomes API calls. Include enough client language/runtime, framework, workflow, and generator design to resolve integration-tooling uncertainty before application implementation. Detailed client UI design and application implementation remain deferred.
- Keep the domain and initial product small. Evolve through user requirements, operational needs, hypotheses, measurements, owner decisions, and verification. A complex distributed deployment is not an initial requirement.
- Consequential system design and infrastructure decisions belong to the owner. See [collaboration rules](agents/collaboration.md) for the decision workflow.
- Use synthetic load; real users are not needed for the learning goals.
- Supported behavior should be production-relevant within its stated limits. A single-node version is acceptable to consider; local resources must not be presented as independent machines.
- Adding replicas with additional per-instance resource budgets is an intentional way to model added nodes, within actual host capacity. Container count alone does not establish added physical capacity or availability.
- Write repository documentation in English. Create directories only alongside meaningful content.

## Active phase

Documentation and design preparation only, with root `.editorconfig` and `.gitattributes` authorized for formatting and Git line-ending consistency. Collect further owner context into the appropriate documents during this session. Application code, tests, further tooling and configuration, dependencies, and deployment assets are deferred until authorized. Revisit the remaining agent setup suggestions when the application project is created. The owner has requested a Socratic design interview: begin with technology, API, and end-to-end workflow; address modular-monolith internals afterward. Use questions to expose unresolved decisions and withhold general architecture recommendations during this initial exploration.

The modular-monolith application shape, single-machine deployment with local-only access, local-filesystem media storage, and SQLite job-information storage are confirmed. Deployment tooling, detailed persistence and recovery design, and any future service roster remain open. These notes are preparation, not an implementation specification or backlog.

Resolve API tooling uncertainty before finishing design and starting application implementation. Decide the frontend framework and client API generator together, after progressing other server and API decisions that may narrow the candidates. Retain both framework-neutral and framework-aware generation as options, and evaluate server specification generation alongside that selection where relevant. The owner permits any suitable order that makes interdependent questions easier to answer. This expands the design investigation, not implementation authorization; decide separately on any executable prototype needed for validation.

## First-version user flow and scope

- The user selects a local video file through the client, starts the process, and ultimately downloads the converted video to their computer.
- V1 supports two output choices with fixed codec combinations: MP4 with H.264/AVC video and AAC-LC audio, and WebM with VP9 video and Opus audio. The user selects the output format, not individual codecs. Encoder directions are recorded in the [licensing guide](dependency-licensing.md); concrete versions/builds, detailed codec profiles, and quality/settings remain unresolved.
- Conversion paths must provide nontrivial processing work, using test inputs that are easy to generate or obtain. The exact supported-input policy and reference test cases remain open.
- Other video editing or conversion options are deferred until a new user requirement justifies them.
- A future version may add AV1 and codec selection accompanied by practical client guidance comparing each codec's main attractions and trade-offs. Both AV1 output and this client guidance are outside v1 scope.
- The workflow must support adding output formats and conversion options. Conversion requires the user to commit the required options. Upload completion alone does not authorize conversion.
- V1 submission uses two ordered requests: first create a job with the required conversion options in the request body, committing those options; the server generates and returns the job identifier. Then upload the source video against that existing job using the returned identifier. Upload does not create a job. This replaces the earlier before-or-during-upload commitment flexibility with options-first submission.
- The minimal client has one Start button: after file and option selection, it creates the job and then uploads the video automatically. Once upload completes, conversion proceeds without another user action, subject to the single-conversion scheduling constraint.
- Committed conversion options cannot be changed in v1. Reconsider this capability later if it provides useful system-design learning.
- The v1 API will expose manual job cancellation, covering the job's upload and conversion work. The current minimal client will not use this capability. Defer cancellation acknowledgment behavior until implementation complexity is understood. The owner's hybrid candidate is to confirm cancellation immediately for queued work that can be prevented from starting, but acknowledge only the request for running work and expose the final outcome through status, avoiding response latency while waiting for running work to stop. This is not a selected response contract; detailed stopping semantics and restart-recovery interaction remain open.
- Upload begins only after the job has been created with committed options. Defer automatic download versus a user-operated Download button to detailed client design; this interaction choice is not a prerequisite for technology selection. Endpoint paths, payload schemas, and upload encoding remain open; the selected request sequence does not determine the detailed client layout.
- The first minimal client may require the user to keep the page open. Restoring a previous conversion after refreshing or reopening the page is not a required first-client capability.
- After upload completion, accepted conversion work continues even if the client disconnects.
- If a server restart interrupts a conversion, the server automatically attempts that conversion again after startup. Recovery mechanisms and restart-related retry limits remain open.
- For conversion failures unrelated to server restart, stop the affected conversion without automatic retry. Store its failed outcome and the relevant circumstances explaining what happened, and log the full diagnostic context for later diagnosis and analysis. A subset of the failure information may be exposed through the API; its exact contents and the diagnostic capture design remain open.
- V1 must accept additional jobs while an earlier conversion is unfinished, but execute only one conversion at a time. Uploads and downloads can proceed independently of sequential conversion execution. Backlog limits, scheduling order, transfer concurrency limits, and overload behavior remain open.
- The API must report a job's current stage in v1; numeric conversion progress is not required. The client learns stage changes by polling job status over HTTP. A small delay before the client learns of a stage change is acceptable. The exact stage/status model, polling interval, and polling-error behavior remain open.
- Update the server's recorded job state as soon as feasible after the preceding stage finishes, within the constraints of the chosen technology and architecture. This internal update goal does not drive technology selection; no quantitative internal-update latency target is selected.
- Interrupted uploads may restart from the beginning against the same job, preserving its identifier and committed options. Resumable uploads are not required. A transfer interruption does not by itself end the job; retries are allowed only while no complete source has been accepted and the job remains eligible for upload.
- Allow only one active upload per job; an earlier attempt must stop before a retry begins. Partial data and cleanup from different attempts must not interfere. Reject replacing an accepted source, and do not reopen a cancelled or abandoned job through a retry. After an uncertain upload response, the client checks job status before retransmitting. Coordination mechanisms, partial-file cleanup timing, attempt diagnostics, and source-identity checks remain open; the abandonment policy is specified below.
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

## Stage-duration and transfer limits

- Conversion has a configurable execution timeout per attempt, separate from queue waiting and HTTP request lifetime. When exceeded, stop the conversion and record failure and diagnostics without automatic retry.
- Choose the conversion-timeout default later, considering the supported workload. Unlimited execution is an explicit candidate for the default, not a selected default.
- Defer the decision on v1 upload/download timeout safeguards until the technology stack is selected and implementation complexity can be assessed. Neither inclusion nor exclusion of these safeguards is selected yet.
- Jobs waiting for an initial upload or retry have a configurable waiting-inactivity timeout defaulting to 24 hours. Start the period at job creation and start a fresh period after an interrupted upload ends. Apply it only while waiting, excluding active transfer, queue waiting, and conversion. Status polling does not extend it, and the recorded waiting deadline survives restarts.
- At the waiting deadline, end the job in a terminal Abandoned state. A later submission requires a new job; retain history under the existing job-retention policy. Partial-file cleanup is a separate decision. This waiting timeout does not replace safeguards for active transfers or impose a maximum total job lifetime.
- Queue-wait limits and their defaults remain open.

## API contract and compatibility

- The v1 client-server API uses RESTful HTTP, with the two-request job-creation/upload sequence and polling for job status specified above. Detailed endpoint contracts, file-transfer encoding, and polling policy remain open.
- Job creation uses a client-generated idempotency key: repeating the same creation request with the same key returns the same job; a new key represents an intentional new submission. This key is distinct from the server-generated job identifier. Key transport, scope, retention, persistence mechanisms, concurrent-retry handling, and conflicting-payload behavior remain to be specified.
- Before v1 is published, the server and client may be updated together, including changes that break older development clients. There is no requirement to support older clients against newer servers during this phase.
- Revisit API versioning and compatibility when v1 is finalized and development of the next version begins. No post-v1 compatibility policy is selected.
- The server's public API declarations are the authoritative contract definition; OpenAPI generated from those declarations is the selected specification approach. The OpenAPI version, server-side generator, generation timing and workflow, and agreement checks remain open.
- Generated client API integration is the intended, revisitable direction. TypeScript is selected for the client. Defer the frontend framework and client generator selection as a joint decision until other server/API choices have progressed; assess both framework-neutral and framework-aware integration. Reconsider generation if tool support for the required workflows makes it unsuitable; compatibility with schemas, file transfers, errors, and cancellation must be assessed before adopting a concrete toolchain. The UI framework and generators remain unselected.
- Prefer evaluating the server-specification generator and client-code generator together when their criteria are known. A shared toolchain is a candidate for reducing maintenance, not a selected tool or a requirement to use one product.
- OpenAPI-generation timing remains open. Client language/workflow/generator investigation is now part of the design phase to resolve tooling uncertainty. Endpoint calling and HTTP integration tests do not inherently require OpenAPI, although specification-driven tools and checks do.

## Owner preferences and candidates

These express direction for discussion, not final architecture decisions.

| Area | Current direction | Still unresolved |
| --- | --- | --- |
| Server API implementation | ASP.NET Core Minimal APIs are confirmed; use .NET 10 capabilities as the provisional design reference. | Choose the runtime at implementation setup, considering .NET 11 release/toolchain compatibility; revisit earlier for version-specific requirements. Detailed endpoint organization and contracts remain open. |
| Conversion execution | Execute one conversion at a time, with uploads and downloads proceeding independently. In-process background conversion is a candidate within the confirmed modular monolith. | Boundaries, execution and scheduling mechanisms, and mechanisms for the required automatic recovery after server restart. |
| Client | Use TypeScript with a UI framework and HTTP polling for job status. Keep the web client minimal; frontend expansion should follow a useful requirement or learning opportunity while distributed backend systems remain the main focus. | Defer the React-versus-Vue choice and client API generator selection together until other server/API decisions have progressed; the brief evaluation is in the [design notes](design-notes.md#minimal-client-technology-candidates). Build tooling, hosting, remaining interactions, and polling policy remain open. |
| Deployment tooling | Prefer running the server and client directly on the owner's computer if the selected stack permits it. Consider Docker later if a dependency, infrastructure technology, or distributed deployment calls for it. A short future trial on an internet-accessible hosted server is an optional learning candidate; the server may be outside the Czech Republic. Public hosting is not important for v1, and the owner is willing to omit it and keep this project local/private. | Hosting provider, actual execution country, service scope, and timing remain unselected. Defer deeper hosted-service analysis until the owner wants the trial; apply the [licensing and territorial patent review](dependency-licensing.md) before deployment. Operating environment, resource budgets, persistence, and operating procedures remain open. |
| Communication mechanisms | RESTful HTTP with polling for job status is confirmed for the v1 client-server API as described above. | File-transfer encoding, polling policy, and contracts and communication between any future services. |
| API/client tooling | OpenAPI generated from server API declarations is confirmed. Generated client integration is planned but revisitable after TypeScript/browser generator validation during this design phase. Coordinated server/client changes before v1 publication are permitted as described above. | Resolve compatible OpenAPI version, generators, and workflow before application implementation. Select the client generator and frontend framework together later; neutral and framework-aware targets remain candidates. Document-generation timing and agreement checks remain open. Post-v1 versioning and compatibility are deferred to the next-version design. |
| Service specification tests | Reqnroll is a candidate; library recommendations are welcome. | Test boundaries, library choice, and scenarios. |
| Performance experiments | Expect repeatable benchmark/load scenarios. | Execution approach, workloads, tools, targets, timing, and which learning exercises to adopt. |

## Agent proposals

- Evaluate asynchronous HTTP streaming for uploads/downloads and bounded background processing for conversion.
- Evaluate object storage and independently deployed workers when requirements justify them. A broker, gateway, reporting service, and telemetry stack have not been selected.

The [design notes](design-notes.md) preserve the supporting rationale and candidate evolution triggers. They do not approve these proposals.

## Benchmark and learning ideas

The [benchmark and architecture learning ideas](learning-experiments.md) collect options for comparable measurements, video workloads, failure experiments, debugging, and the owner's own bottleneck predictions and retrospective evaluation. During version or stage planning and review, the owner chooses which ideas to use, defer, or skip. Their inclusion does not approve an experiment plan or implementation. Persisting detailed benchmark results, machine descriptions, and predictions remains an option to decide when useful.

[WHATSNEW.md](../WHATSNEW.md) is the human-facing place for very brief architecture summaries of released versions; it does not establish a versioning scheme or release schedule.

## Open questions for the requested grilling session

- **Product scope:** supported inputs, encoder implementations and detailed output settings, conversion behavior, input-size limits, and server access/authentication needs. The two output container/codec combinations, basic local-file-to-downloaded-result flow, and acceptance of shared access without per-user isolation are confirmed above.
- **Lifecycle:** when a job becomes eligible, job states, persistence, work ownership, queue limits and scheduling order, restart-related retry limits, duplicate attempts, cancellation semantics, failure information exposed through the API, diagnostic capture, and server restart recovery mechanisms. Continued conversion after client disconnection, automatic retry of conversions interrupted by server restart, no automatic retry for other conversion failures, a manual cancellation API unused by the minimal client, acceptance of overlapping jobs, and one conversion at a time with independent uploads and downloads are confirmed above.
- **Files:** input/output durability mechanisms, failed-source cleanup and explicit deletion mechanisms, source-video retention after cancellation, finite job-history expiration timing and cleanup, cleanup of complete and partial files, active downloads at result expiry, retention-configuration changes, and disk exhaustion behavior. Restarting interrupted uploads from the beginning, the converted-result retention policy, source cleanup eligibility after result availability, the 60-day default failed-source retention policy, and configurable job-history retention with no automatic expiration by default are confirmed above.
- **Interfaces and tests:** client technology and hosting, API contracts, status delivery, and service specification tests.
- **Operations:** operating environment and deployment tooling within the confirmed single-machine scope, resource budgets, load scenarios, latency/completion/error targets, and observability sufficient to explain results.
- **Evolution:** evidence or requirements that justify another database, object storage, a broker, independently deployed workers, additional replicas, or another service.

The product name is resolved as **ClipForge**. The remaining questions are discussion topics, not ready implementation tickets.
