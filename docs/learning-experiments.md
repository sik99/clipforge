# Benchmark and architecture learning ideas

These ideas can help identify the system's current limits, evaluate a completed version or stage, and decide what to develop next. They are a menu for future discussion, not an accepted benchmark plan, release checklist, implementation backlog, or architecture roadmap. At the relevant planning or review point, the owner chooses what to try, defer, or skip. Tools, targets, recording formats, and experiment scope remain open.

## Building architectural intuition

One possible exercise at the end of a stage or while planning the next version is to predict the next bottleneck before asking AI for recommendations. If the owner chooses this exercise, give them space to state their own hypothesis first; offer analysis and alternatives afterward.

A prediction could describe the expected limiting component, the workload and resource conditions that would expose it, the reason for the prediction, and observations that would support or contradict it. Preserve the original prediction so later findings do not replace it in hindsight.

A possible learning cycle is: owner's hypothesis, baseline experiment, observed limitation, alternatives and owner decision, implementation, repeated measurement, and retrospective. The retrospective could compare the prediction with the actual bottleneck, explain the evidence and any uncertainty, and identify the next question. These steps can be selected independently; a version does not need a performance change or a full experiment cycle to be useful.

Finding that more instances or a proposed architectural change are unnecessary or ineffective is useful evidence. The learning value comes from explaining the result and its trade-offs.

## Designing comparable experiments

Possible practices to select for an experiment:

- Define the question, workload, resource budget, correctness checks, and success criteria before running it. Compare sustainable throughput at agreed latency and error targets rather than treating a higher throughput number alone as success.
- Establish a baseline, separate warm-up from measurement, and repeat runs. Compare a change under comparable workload and environment conditions, recording intentional differences.
- Check that the load generator has headroom. Separate application saturation from limits of the generator, host, routing, storage, or network.
- Check correctness alongside jobs completed per minute, API latency and completion percentiles, errors, resource use, CPU throttling, garbage collection, database waits, queue length, and oldest-job age. A fast acknowledgment is different from a successfully completed conversion.
- Vary encoder threading, concurrent jobs within one process, and independently deployed instances as separate variables. Adding replicas with additional resource budgets can model added nodes within host capacity; a fixed-total-budget comparison answers a different efficiency question.
- Define timing boundaries, including client versus server measurements, connection reuse, upload completion, job acceptance, result readiness, and completed download. The existing [design notes](design-notes.md#measurement-and-topology-limits) explain transfer stages and topology limits.

## Video workloads

Candidate workload design:

- Start with repeatable short clips and one output profile. Group inputs by duration, resolution, and encoding characteristics, and report completion percentiles per group. Processing time divided by media duration can be an additional indicator, with input characteristics still recorded.
- Use a pre-uploaded dataset to isolate processing capacity and a separate upload-to-downloaded-result scenario to include transfer costs. Each answers a different question.
- Compare isolated conversions, concurrent submissions, status queries during conversion, and competing uploads/downloads. Vary one factor at a time when trying to explain an observed limit.
- Compare steady arrivals with bursts and sustained overload. Observe arrival and completion rates, backlog, queue age, API responsiveness, and recovery after load falls.

Benchmark execution could use scenario scripts with an optional suite runner, or a reusable load generator. Choose when needed; see the [repository layout](repository-layout.md) for placement.

## Failures and overload

Possible experiments as the relevant components become available:

| Question | Candidate experiment and observations |
| --- | --- |
| What happens when an instance fails during work? | Stop or restart an instance under load; inspect failed requests, work ownership, recovery time, retries, and duplicate or missing results. Check that successful jobs have complete, accessible output. |
| What happens when a dependency slows down or fails? | Slow or interrupt storage or database access; inspect waits, timeouts, retry amplification, API behavior, backlog, and recovery. |
| What happens when arrivals exceed processing capacity? | Sustain overload and observe backlog growth and resource use. Use the results to discuss backlog bounds, admission limits, backpressure, or rejection behavior. A queue can absorb bursts but does not add processing capacity. |
| What happens when processing capacity is removed? | If multiple workers exist, compare stopping workers with allowing in-flight work to finish during scale-in; inspect ownership, completion, and recovery. |

Required guarantees and mechanisms are owner decisions at the relevant stage. These scenarios do not select a broker, retry policy, or distributed deployment.

## Debugging a fast API with a late result

A candidate diagnostic exercise is a conversion accepted quickly while the result becomes available much later than expected. Follow the same job through acceptance, waiting, conversion, storage access, and output publication; correlate logs, metrics, and traces across the processes that actually exist.

Possible explanations to investigate include saturated processing capacity, slow input/output transfers, database locks or connection waits, repeated attempts, and missing telemetry context. Compare explanations against evidence before changing the architecture. This exercise can build from instrumentation in one process to cross-process diagnosis when separate services exist; no telemetry stack is selected.

## Possible experiment records

If the owner chooses to persist predictions and results, an experiment record could live under `experiments/<experiment>/`, alongside its reproducible setup. The exact structure and data format can be chosen with the first experiment. Useful record contents could include:

- System version or commit, experiment question, the owner's original hypothesis, and any later AI suggestions identified separately.
- Date and machine identity or descriptive label; CPU, memory, storage, operating system, relevant runtime and conversion-engine versions, and build configuration.
- Deployment topology, instance counts, per-instance limits, total resource budget, and whether the generator shares the application host or runs elsewhere.
- Input dataset and output profile, offered load, concurrency, measurement boundaries, warm-up and measurement durations, repetitions, and correctness criteria.
- Results, variability, supporting telemetry or retained artifact references, observed bottleneck, comparison with the prediction, and the owner's decision or remaining questions.

Raw telemetry and large media/results may need a separate retention choice. A concise versioned record can preserve conclusions and reproducibility details even when large outputs remain local. Storage, artifact retention, and a mandatory record template are not selected.

The human-facing [WHATSNEW.md](../WHATSNEW.md) provides a brief architecture summary for each released version. Detailed experiment evidence can be linked from a version entry when available.
