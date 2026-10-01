# OpenASER

Open Agentic Software Engineering Runtime.

OpenASER is a flexible runtime around coding agents, not a replacement for them.
It owns engineering semantics and uses replaceable implementations for execution,
work management, verification, state, knowledge, models, isolation, and recovery.
Simple work should stay simple; larger work should use only the capabilities it needs.

**Status:** planning only. No runtime, adapters, or CLI commands are implemented.
This README is the concise implementation plan. [OpenASER.md](OpenASER.md) remains
unchanged as background documentation; its broader proposals do not authorize work.

## Working Rules

- Implement only requirements explicitly agreed with the user. Do not infer features from the broader design.
- Ask before choosing the next milestone, resolving an unclear requirement, or expanding scope.
- Before coding, record the milestone's behavior, required capabilities, acceptance checks, and exclusions here.
- Implement one approved milestone at a time. Verify it, report the result, and ask how to continue.
- Keep changes small and understandable. No speculative helpers, configuration, abstractions, or dependencies.
- Each implemented capability gets a narrow contract and a concrete implementation. Do not expose every feature of the underlying tool.
- The planned implementations below are commitments to provide support, not requirements to build or enable everything in the first milestone.

## Architecture

```text
Domain semantics -> Capability contracts -> Concrete implementations

Work -> Decide -> Prepare -> Execute -> Verify -> Record
```

OpenASER owns task/run lifecycles, workflow transitions, acceptance, and policy.
Infrastructure supplies mechanisms without defining the domain model.
Classification informs policy; it does not control execution by itself.
Evaluation and policy changes use measured results, not assumptions.

| Concept | Meaning |
| --- | --- |
| Task | Requested work and its acceptance criteria. |
| Run | One attempt to perform a task, with an explicit outcome. |
| Workflow | Preparation, execution, verification, and allowed transitions. |
| Policy | Choices about workflow, context, models, resources, retries, and approval. |
| Artifact | Output attributable to a run, such as a patch or report. |
| CompletionClaim | A worker's assertion that its output is ready for verification. |
| Evidence | Check results supporting acceptance or rejection, not merely logs. |
| Knowledge | Reusable project information, separate from task state and evidence. |

Contracts must allow implementations to change without changing these meanings.
Define their exact methods and data only when an approved milestone needs them.

## Planned Implementations

The runtime will be written in Rust. The following implementations are planned;
none are implemented yet. Laya, the open-source variant specified by the user,
replaces Jev for classification. Confirm its upstream project and integration
interface before implementing its adapter.

| Abstract Role | Responsibility | Planned Implementation |
| --- | --- | --- |
| Orchestrator | Schedule and coordinate work. | Gas City |
| WorkStore | Persist tasks, status, and dependencies. | Beads |
| DecisionEngine | Return bounded, typed classifications. | Laya; deterministic rules where appropriate |
| Executor | Start, observe, and stop coding-agent attempts. | OpenCode |
| ModelProvider | Provide local and remote inference access. | vLLM with a capable Qwen-class model; remote model APIs |
| Sandbox | Enforce execution permissions and resource boundaries. | OpenShell |
| SourceControl | Track repository history and changes. | Git |
| WorkspaceIsolation | Separate worker workspaces. | Git worktrees |
| Verifier | Check output against agreed acceptance criteria. | Project-native tests, builds, lint, and acceptance scripts |
| ArtifactStore | Preserve run outputs and attribution. | Filesystem + Git |
| RuntimeStore | Persist runs, evidence, and operational metadata. | SQLite |
| KnowledgeStore | Store durable project knowledge. | Markdown + YAML frontmatter |
| KnowledgeIndex | Search project knowledge. | SQLite FTS5 |
| CodeIntelligence | Discover code and repository relationships. | Filesystem + ripgrep + Git; Graphify for richer discovery |
| SecretProvider | Supply credentials without storing them as project state. | Environment variables, OS keychains, secret managers, or CI secret stores; specific backends to agree |
| HumanApprovalProvider | Obtain explicit authorization where required. | CLI approval |

Support does not mean mandatory use: for example, Graphify and local inference
must not be required for every task. Exact model versions, remote providers,
secret backends, and integration interfaces remain decisions to make together.
LSP, AST analysis, embeddings, alternate harnesses, and other previously mentioned
extension possibilities are not additional implementation commitments.

## Runtime Boundaries

- Agent output is a proposal. A completion claim alone must never mark a task accepted.
- Verification is runtime-controlled and uses agreed acceptance checks. Failed or missing checks must not silently become success.
- Record enough task/run state, artifacts, and evidence to explain the outcome. Durability lasts as long as the agreed use case requires.
- Recovery and fresh-agent continuation must use explicit state, not depend on reconstructing a previous chat.
- Work state, knowledge, knowledge indexes, and evidence remain separate. Worker observations are knowledge candidates, not automatically permanent knowledge.
- Context comes from the task, project instructions, relevant code, and knowledge. Discovery can add context; the whole project need not fit in one conversation.
- Models, inference servers, coding harnesses, and the runtime are separate. Context capacity and resource policy reflect the selected model and environment.
- Git worktrees separate changes; they are not a security sandbox. Prompts are not permission enforcement.
- Retries, decomposition, parallel work, model escalation, knowledge consolidation, and evaluation are explicit capabilities, not automatic defaults.

These boundaries guide implementation; they do not authorize extra subsystems in a milestone.

## Implementation Plan

### 1. Agree on the First Milestone

Choose one real engineering use case and answer:

- What does the user submit, and what result should the runtime produce?
- Which acceptance checks determine success, and who defines them?
- Which planned capabilities are necessary for that result?
- What filesystem, command, network, and credential access is allowed?
- Must this milestone survive interruption and support fresh-agent continuation?

Then record the smallest agreed scope here. Do not scaffold the entire architecture first.

### 2. Implement Only the Approved Scope

Confirm the relevant tools' actual interfaces. Add only the domain types,
capability contracts, adapters, and configuration required by that milestone.
Do not add retries, routing, parallelism, recovery, or CLI commands unless approved.

### 3. Verify and Choose Together

Run the milestone's acceptance checks, including failure cases. Ensure a false
completion claim cannot bypass verification and provider details do not leak into
domain semantics. Report what works and what remains; ask the user what to do next.
Update this README before starting the next approved milestone.

**Current milestone:** none approved. Next action: agree on the first use case.

## Out of Scope

No custom coding harness, inference server, database, general workflow DSL,
plugin marketplace, or unrestricted self-modification. No automatic installation,
large configuration/profile system, or additional integrations without agreement.
Planned provider support is not permission to implement every upstream feature.
