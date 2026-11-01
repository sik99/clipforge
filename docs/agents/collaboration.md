# Collaboration with AI agents

## Decision ownership

The owner is practicing system architecture. Preserve that learning opportunity by leaving consequential system design and infrastructure decisions to them.

Before a decision that changes service boundaries, communication patterns, data ownership or storage, consistency, deployment topology, infrastructure, or observability architecture:

1. State the requirement or observed limitation driving the decision.
2. Explain viable options and their trade-offs at the level needed for the owner's choice. Recommend an option when useful, and identify assumptions.
3. Wait for the owner's decision before implementing the affected design. Record the decision in its authoritative document.

Use existing owner decisions without requesting repeated approval. Agents can choose routine implementation details within those boundaries, including local code organization and function implementations. Escalate an implementation choice when it materially changes system behavior, the operational model, or architectural constraints. Concrete library recommendations are welcome; keep tentative preferences distinct from settled decisions.

## Documentation intake

Consult the [project brief](../project-brief.md) for the active phase and scope. During documentation preparation, update documentation only; create code, tests, dependency declarations, deployment files, and configuration after implementation is authorized.

As the owner supplies context, update the relevant source of truth in the same session:

- Goals, scope, constraints, technology direction, and open design questions: [project brief](../project-brief.md).
- Technical rationale and candidate evolution triggers: [design notes](../design-notes.md).
- Optional benchmark and architectural intuition exercises: [learning ideas](../learning-experiments.md).
- Brief human-facing architecture summaries for released versions: [WHATSNEW.md](../../WHATSNEW.md).
- Collaboration rules: this file; keep universally necessary instructions in [AGENTS.md](../../AGENTS.md).
- Directory responsibilities: [repository layout](../repository-layout.md).
- Resolved domain terms: root `GLOSSARY.md`, created lazily following [domain conventions](domain.md).
- Feature specifications and actionable work: local files following [issue tracker conventions](issue-tracker.md), with [triage labels](triage-labels.md) when applicable.
- Significant architectural decisions: `docs/adr/`, following the domain-modeling skill when a real trade-off produces an owner decision that is costly to reverse and needs its rationale preserved.

Capture durable meaning rather than a transcript. Explicitly distinguish confirmed owner decisions, tentative owner preferences, agent proposals, and open questions. When information changes, revise its authoritative location and remove obsolete statements. Link to that location where another document needs the context. Exploratory notes are not ADRs or ready implementation tickets.

## Progressive disclosure and workflow

Keep README short and intended for humans. Keep AGENTS.md a small entry point with essential guardrails and pointers that state when to read each document. Load documents relevant to the task; keep full specs, issue histories, and architecture discussions in their task-specific documents.

Use the repository's existing Matt Pocock skill conventions for local specs, per-issue files, work maps, triage, domain vocabulary, and decision records. Invoke grilling when the owner requests it. Once a feature is agreed, capture its behavior in a spec before deriving implementation issues; mark work ready for an agent only when it is sufficiently specified within owner-approved boundaries.

Create documents and directories when they contain useful information. Record concise validation evidence with completed work. Add an operational guide or experiment record when there is an actual procedure or result to preserve.

When planning or reviewing a system version or stage, consult the [benchmark and architecture learning ideas](../learning-experiments.md) and bring up the relevant options for the owner's selection. Apply the exercises they choose and keep deferred ideas as candidates. If they choose to predict a bottleneck without AI assistance, let them state and record their hypothesis before offering recommendations. When a version is released, add a very brief description of its delivered architecture to [WHATSNEW.md](../../WHATSNEW.md).
