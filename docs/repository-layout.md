# Repository layout

Create directories only when adding meaningful files. The layout describes responsibilities, not a fixed service decomposition.

## Current documentation

| Location | Responsibility |
| --- | --- |
| [README.md](../README.md) | Brief human introduction and starting links. |
| [WHATSNEW.md](../WHATSNEW.md) | Very brief human-facing architecture summaries by released version. |
| [AGENTS.md](../AGENTS.md) | Essential agent guardrails and task-specific pointers. |
| [CLAUDE.md](../CLAUDE.md) | Direct Claude Code to the shared agent instructions. |
| [docs/project-brief.md](project-brief.md) | Goals, active scope, decision status, and unresolved questions. |
| [docs/design-notes.md](design-notes.md) | Technical rationale and candidate evolution triggers. |
| [docs/dependency-licensing.md](dependency-licensing.md) | Main source of truth for third-party software licensing decisions, obligations, source links, and setup/integration/distribution checkpoints. |
| [docs/learning-experiments.md](learning-experiments.md) | Optional benchmark, failure, debugging, and architectural intuition exercises to select during stage planning or review. |
| [docs/agents/](agents/) | Collaboration, local issue tracker, triage, and domain documentation conventions. |

## Add when needed

These are placement conventions for later work. They do not authorize implementation or require creating any directory now.

| Location | Create when |
| --- | --- |
| `GLOSSARY.md` | Domain terms are resolved; use the single-context conventions. |
| `docs/adr/` | An owner-approved architectural trade-off needs a durable decision record. Examples in the domain conventions are illustrative filenames, not decisions. |
| `.scratch/<feature-or-effort>/` | An agreed feature needs a spec, or an exploration needs a work map and per-issue files; follow the [issue tracker conventions](agents/issue-tracker.md). Durable project decisions belong in the documentation above. |
| `src/` | Application implementation is authorized. Keep server/client code here, naming projects after approved responsibilities. A project or folder does not imply an independently deployed service. |
| `tests/` | Authorized automated tests have actual scenarios to verify. Organize by test purpose and the approved application boundaries. |
| `experiments/` | A benchmark scenario or record exists. Keep scenario scripts here, with an optional suite runner, setup, results, machine descriptions, hypotheses, and retrospective evaluation. See the [record ideas](learning-experiments.md#possible-experiment-records). |
| `deploy/` | An approved deployment needs assets, such as Compose files. |
| `docs/operations/` | There is an actual deployment, recovery, observability, or debugging procedure to document. |
| `.local/` | Local development or experiment runtime data is needed, such as database files, uploaded/generated media, worker scratch, or large raw diagnostic and benchmark outputs. This directory is ignored by Git; versioned setup and useful experiment conclusions belong with their source or experiment record. |

Keep supporting material beside the feature or experiment it explains. Choose further subdivisions from real content; avoid speculative service directories and placeholder files.
