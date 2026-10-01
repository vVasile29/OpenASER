**OpenASER — Open Agentic Software Engineering Runtime**

OpenASER is an open-source runtime for reliable agentic software engineering.

It provides the engineering system around coding agents: work management, workflow selection, execution, verification, durable project state, project knowledge, recovery, model and compute management, isolation, evidence, and evaluation.

OpenASER is designed to scale with the work.

A small task may require one worker and one verification step.

A large task may require decomposition, multiple workers, retries, coordination, durable state, integration, and continuation across sessions.

The runtime should use only as much machinery as the engineering task requires.

---

# 1. Introduction

## 1.1 What is OpenASER?

OpenASER is an open-source runtime for agentic software engineering.

Coding agents can already:

- inspect repositories,
    
- modify files,
    
- run commands,
    
- create tests,
    
- debug failures,
    
- implement features,
    
- and reason about code.
    

OpenASER does not attempt to replace those agents.

Instead, it provides the engineering runtime around them.

Conceptually:

```
Coding Agent
    =
performs engineering work


OpenASER
    =
manages the engineering system
around that work
```

OpenASER manages capabilities such as:

- work and task state,
    
- workflow selection,
    
- execution,
    
- orchestration,
    
- agent lifecycle,
    
- verification,
    
- artifacts,
    
- evidence,
    
- project knowledge,
    
- context,
    
- model selection,
    
- compute usage,
    
- sandboxing,
    
- recovery,
    
- resumability,
    
- parallel work,
    
- human approval,
    
- and evaluation of engineering policies.
    

---

## 1.2 Why OpenASER exists

A coding agent is a powerful worker.

It is not, by itself, a complete software-engineering system.

Even small engineering tasks benefit from:

- clear task state,
    
- reproducible execution,
    
- isolation,
    
- verification,
    
- and evidence.
    

Larger tasks introduce additional requirements:

- decomposition,
    
- dependencies,
    
- multiple workers,
    
- parallel execution,
    
- recovery,
    
- persistent state,
    
- integration,
    
- accumulated project knowledge,
    
- and continuation across sessions.
    

Without a runtime, these responsibilities tend to become scattered across:

- prompts,
    
- chat history,
    
- shell scripts,
    
- manually maintained notes,
    
- ad-hoc agent coordination,
    
- and developer memory.
    

OpenASER provides a structured runtime for those responsibilities.

---

## 1.3 Core idea

A central architectural principle is:

> **Engineering state should survive for as long as the work requires.**

For a trivial task that may mean one run.

For a complex project it may mean many runs, agents, models, machines, and restarts.

An agent is a worker.

The project is the durable engineering system when durability is required.

Related principles are:

> **Agents do the work. Evidence determines acceptance.**

> **Task complexity should determine runtime complexity.**

> **A finite context window must not impose a finite engineering horizon.**

> **Policies are hypotheses and should change when evidence shows a better approach.**

---

## 1.4 What OpenASER is not

OpenASER is not primarily:

- a coding model,
    
- a coding harness,
    
- a multi-agent scheduler,
    
- a task tracker,
    
- a sandbox,
    
- a RAG framework,
    
- a vector database,
    
- a code graph,
    
- a model server,
    
- a Git wrapper,
    
- or a prompt collection.
    

OpenASER may use tools providing all of these capabilities.

Those tools are implementations.

They are not the architecture itself.

For example:

```
Capability             Possible implementation

Orchestrator        →  Gas City
WorkStore           →  Beads
Executor            →  OpenCode
Sandbox             →  OpenShell
SourceControl       →  Git
RuntimeStore        →  SQLite
KnowledgeStore      →  Markdown
CodeIntelligence    →  rg + Git / Graphify
ModelProvider       →  vLLM / remote API
DecisionEngine      →  Jev / local classifier / rules
```

OpenASER depends on capabilities rather than those product names.

---

## 1.5 Mental model

At its simplest:

```
WORK
  ↓
DECIDE
  ↓
PREPARE
  ↓
EXECUTE
  ↓
VERIFY
  ↓
RECORD
  ↓
EVALUATE
  ↺
```

Not every task requires a complicated path through this loop.

A small change may be:

```
Task
 ↓
Decision: simple
 ↓
one worker
 ↓
verify
 ↓
done
```

A complicated change may become:

```
Goal
 ↓
Decision: complex
 ↓
decompose
 ↓
multiple dependent tasks
 ↓
parallel/sequential workers
 ↓
integration
 ↓
verification
 ↓
record
 ↓
done
```

---

# 2. Design Foundations

## 2.1 Architecture from reality

OpenASER should not begin with arbitrary architectural preferences.

It should begin with properties of agentic software engineering.

The design process is:

```
Reality
  ↓
Constraints
  ↓
Requirements
  ↓
Policies
  ↓
Capabilities
  ↓
Implementations
```

This distinction prevents current implementation choices from becoming permanent doctrine.

---

## 2.2 Agents are fallible

Agent output cannot be assumed correct merely because an agent reports completion.

Therefore:

```
Agent
  ↓
CompletionClaim
  ↓
Verification
  ↓
Evidence
  ↓
Accepted / Rejected
```

An agent saying:

```
"I fixed the issue."
```

is a claim.

A successful verification contract provides evidence.

---

## 2.3 Agent execution is not reliably deterministic

Agent behavior may depend on:

- model behavior,
    
- prompts,
    
- context,
    
- sampling,
    
- tool responses,
    
- environment state,
    
- previous actions,
    
- timing,
    
- model version,
    
- and external systems.
    

OpenASER therefore surrounds probabilistic workers with explicit runtime state and engineering guardrails.

Conceptually:

```
probabilistic worker
        ↓
engineering constraints
        ↓
verification
        ↓
evidence
        ↓
durable state
```

---

## 2.4 Software engineering provides strong verification mechanisms

Software engineering has something many other agentic domains do not have: a large number of machine-checkable signals.

Examples include:

- tests,
    
- compilers,
    
- type systems,
    
- static analysis,
    
- linters,
    
- build systems,
    
- schema validators,
    
- benchmarks,
    
- dependency checks,
    
- and version control.
    

OpenASER should exploit these wherever they are useful.

---

## 2.5 Context is finite

Models have finite effective context windows.

OpenASER should work with practical contexts such as 64K without requiring frontier-scale context windows.

This does not mean OpenASER should minimize context for its own sake.

If useful information fits and improves the result, it should be used.

The requirement is:

> **Use available context efficiently without requiring the entire project state or history to fit inside it.**

A model with a larger useful context should be able to exploit it.

---

## 2.6 Projects may outlive individual agents

An agent may:

- terminate,
    
- fail,
    
- be cancelled,
    
- exhaust its context,
    
- be replaced,
    
- switch models,
    
- or run on another machine.
    

Important engineering state must survive whenever the task requires continuity.

Therefore:

```
agent lifetime
     ≠
required engineering-state lifetime
```

---

## 2.7 External components fail

OpenASER interacts with:

- models,
    
- harnesses,
    
- Git,
    
- filesystems,
    
- databases,
    
- test runners,
    
- sandboxes,
    
- network services,
    
- and external processes.
    

Failures are normal.

Recovery must therefore be part of the runtime design.

---

## 2.8 Parallel workers may interfere

Parallelism can improve throughput.

It can also cause:

- conflicting edits,
    
- duplicated work,
    
- inconsistent assumptions,
    
- integration failures,
    
- unnecessary model usage,
    
- and coordination overhead.
    

Parallelism is therefore a runtime policy, not an objective in itself.

---

## 2.9 Design principles

OpenASER follows several broad principles.

### Engineering state survives as long as required

Durability scales with task needs.

### Evidence over agent confidence

The worker does not unilaterally decide correctness.

### Task complexity determines runtime complexity

Simple work should remain simple.

### Use context efficiently

Do not require huge contexts, but do not artificially starve capable models.

### Use the simplest reliable mechanism

If deterministic code solves a problem reliably, use it.

If classification is genuinely ambiguous, use a classifier.

If strong reasoning is required, use an appropriate model.

### Complexity must pay rent

Every additional subsystem should justify:

- its operational complexity,
    
- maintenance burden,
    
- compute,
    
- latency,
    
- and failure modes.
    

### Measure rather than assume

Policies should be evaluated empirically.

### Own engineering semantics; reuse mechanisms

OpenASER should own what the engineering system means.

External tools should implement capabilities beneath those semantics.

---

# 3. Architectural Layers

OpenASER is easiest to understand as three layers.

## 3.1 Domain concepts

These describe OpenASER's engineering model.

```
Task
Run
Artifact
Evidence
Knowledge
CompletionClaim
Workflow
Policy
```

---

## 3.2 Capability contracts

These describe what the runtime needs from infrastructure.

```
Orchestrator
WorkStore
Executor
DecisionEngine
ModelProvider
Sandbox
SourceControl
WorkspaceIsolation
Verifier
ArtifactStore
KnowledgeStore
KnowledgeIndex
RuntimeStore
CodeIntelligence
SecretProvider
HumanApprovalProvider
```

---

## 3.3 Concrete implementations

These provide actual mechanisms.

```
Gas City
Beads
OpenCode
Jev
OpenShell
Git
Git worktrees
SQLite
Markdown
ripgrep
Graphify
vLLM
local models
remote model APIs
project-native test/build tools
```

The relationship is:

```
Domain
  ↓
Capability
  ↓
Implementation
```

For example:

```
Run
 ↓
Executor
 ↓
OpenCode
```

or:

```
Knowledge
 ↓
KnowledgeStore
 ↓
Markdown
```

---

# 4. Core Domain Model

## 4.1 Task

A `Task` represents engineering work.

Example:

```
id: auth-172

goal: Fix the refresh-token rotation race condition

status: active

acceptance:
  commands:
    - cargo test
    - cargo clippy

dependencies: []
```

A task may originate from:

- a user command,
    
- an issue tracker,
    
- a parent goal,
    
- decomposition,
    
- another task,
    
- CI,
    
- or automated maintenance.
    

---

## 4.2 Run

A `Run` represents one attempt to perform work.

A run records enough information to answer:

- what task was attempted?
    
- what workflow was used?
    
- which agent or harness performed it?
    
- which model was used?
    
- what context was supplied?
    
- what environment was used?
    
- what commands ran?
    
- what changed?
    
- what evidence was generated?
    
- what did the run cost?
    
- how did it end?
    

Possible lifecycle:

```
Created
  ↓
Prepared
  ↓
Executing
  ↓
Verifying
  ↓
Accepted

or

Failed
Cancelled
Interrupted
```

---

## 4.3 Artifact

An `Artifact` is a durable output produced by engineering work.

Examples:

- code changes,
    
- commits,
    
- patches,
    
- test additions,
    
- migration scripts,
    
- reports,
    
- benchmark output,
    
- generated documentation,
    
- review findings.
    

Artifacts should be attributable to the run that produced them.

---

## 4.4 Evidence

`Evidence` is information relevant to determining whether an engineering claim should be accepted.

Examples:

```
evidence:
  - type: test
    command: cargo test
    result: pass

  - type: lint
    command: cargo clippy
    result: pass
```

Evidence is not the same as logs.

```
Log
=
what happened

Evidence
=
information supporting or rejecting
an engineering claim
```

---

## 4.5 Knowledge

`Knowledge` is durable project information expected to improve future engineering work.

Examples:

- architectural decisions,
    
- constraints,
    
- conventions,
    
- verified lessons,
    
- known failure modes,
    
- important subsystem behavior,
    
- project-specific rules.
    

Example:

```
---
id: auth-refresh-rotation
type: constraint
scope:
  - src/auth/**
status: active
---

Refresh-token rotation must be atomic per session.

## Evidence

- tests/auth/refresh_race.rs
```

---

## 4.6 CompletionClaim

A `CompletionClaim` is a worker's assertion that its work is ready for verification.

It is deliberately separate from actual task completion.

```
Worker
  ↓
CompletionClaim
  ↓
Verifier
  ↓
Evidence
  ↓
Task accepted or rejected
```

---

## 4.7 Workflow

A `Workflow` describes a structured engineering process.

Examples:

```
bugfix
feature
refactor
migration
dependency-update
investigation
review
performance
security-fix
```

A workflow may specify:

- preparation,
    
- expected artifacts,
    
- allowed transitions,
    
- decomposition behavior,
    
- verification,
    
- retries,
    
- escalation,
    
- approval points,
    
- and completion rules.
    

---

## 4.8 Policy

A `Policy` controls decisions that may change depending on evidence or environment.

Examples:

- which workflow to select,
    
- which model tier to use,
    
- whether to decompose,
    
- whether to parallelize,
    
- how much context to provide,
    
- when to escalate,
    
- which verification checks to run,
    
- which knowledge to surface.
    

Policies are not assumed permanently correct.

They are candidates for evaluation and improvement.

---

# 5. Decision and Workflow Selection

## 5.1 Why a DecisionEngine exists

Not every incoming task should trigger the same runtime flow.

Consider:

```
"Rename this variable."
```

versus:

```
"Replace the authentication subsystem while preserving backward compatibility."
```

The first likely needs:

```
one worker
→ edit
→ verify
```

The second may need:

```
investigation
→ decomposition
→ multiple tasks
→ coordination
→ integration
→ extensive verification
```

OpenASER therefore needs to determine what kind of engineering process is appropriate.

---

## 5.2 DecisionEngine

`DecisionEngine` provides bounded classification or routing decisions.

Possible implementation:

```
DecisionEngine
   ├── deterministic rules
   ├── Jev
   ├── local classifier
   └── another structured classifier
```

Typical decisions include:

```
task_type
task_complexity
workflow
needs_decomposition
parallelism_candidate
model_tier
risk_level
approval_requirement
```

The output should be typed and constrained.

Example:

```
workflow: bugfix
complexity: small
needs_decomposition: false
parallelism: none
model_tier: local-strong
confidence: 0.91
```

---

## 5.3 DecisionEngine vs PolicyEngine

These are different responsibilities.

```
DecisionEngine
"What does this appear to be?"


PolicyEngine
"Given that classification and runtime state,
what should OpenASER do?"
```

Example:

```
DecisionEngine:

complexity = small
workflow = bugfix
confidence = 0.94


PolicyEngine:

small bugfix
→ one worker
→ no decomposition
→ local coding model
→ standard verification
```

The classifier does not directly control the runtime.

OpenASER owns the policy.

---

## 5.4 Deterministic flow after probabilistic classification

An important pattern is:

```
ambiguous input
     ↓
bounded classification
     ↓
typed result
     ↓
explicit workflow/state machine
```

Rather than repeatedly asking an LLM:

```
"What should happen next?"
```

the runtime can execute defined transitions.

Example:

```
simple_bugfix
    ↓
prepare
    ↓
execute
    ↓
verify
    ↓
pass → record
fail → retry/escalate
```

This keeps probabilistic judgment at the boundaries where judgment is actually useful.

---

## 5.5 Deterministic rules first where appropriate

The runtime should not invoke a classifier when the answer is already explicit.

Example:

```
if user explicitly requests verification_only:
    workflow = verification_only
```

A useful hierarchy is:

```
Can deterministic logic decide this reliably?
          │
        yes
          │
          ▼
      use rules

          no
          │
          ▼
    DecisionEngine
          │
          ▼
      typed decision
```

---

# 6. Runtime Architecture

## 6.1 Main loop

The runtime loop is:

```
WORK
  ↓
DECIDE
  ↓
PREPARE
  ↓
EXECUTE
  ↓
VERIFY
  ↓
RECORD
  ↓
EVALUATE
  ↺
```

---

## 6.2 Work

The runtime first determines what engineering work exists.

This may include:

- one task,
    
- a graph of tasks,
    
- dependencies,
    
- blocked work,
    
- follow-up work,
    
- or a larger goal requiring decomposition.
    

---

## 6.3 Decide

OpenASER determines what engineering process is appropriate.

Inputs may include:

- task description,
    
- repository metadata,
    
- known dependencies,
    
- project policy,
    
- risk,
    
- model availability,
    
- cost constraints,
    
- previous run history.
    

Outputs may include:

- workflow,
    
- model tier,
    
- decomposition policy,
    
- verification level,
    
- approval requirements,
    
- concurrency strategy.
    

---

## 6.4 Prepare

Preparation may include:

- selecting the executor,
    
- selecting a model,
    
- constructing context,
    
- creating a worktree,
    
- creating a sandbox,
    
- loading project constraints,
    
- retrieving relevant knowledge,
    
- checking dependencies,
    
- preparing credentials,
    
- and registering the run.
    

---

## 6.5 Execute

The chosen executor performs engineering work.

The worker may:

- inspect files,
    
- search the repository,
    
- inspect history,
    
- modify code,
    
- add tests,
    
- run commands,
    
- ask for additional project information,
    
- and produce artifacts.
    

---

## 6.6 Verify

When the worker claims completion, OpenASER verifies the result.

Verification may include:

- tests,
    
- builds,
    
- type checks,
    
- lint,
    
- static analysis,
    
- benchmarks,
    
- policy checks,
    
- independent model review,
    
- human approval.
    

The outcome is evidence.

---

## 6.7 Record

OpenASER records relevant state:

- run outcome,
    
- task status,
    
- artifacts,
    
- evidence,
    
- metrics,
    
- context metadata,
    
- resource usage,
    
- knowledge candidates,
    
- follow-up work.
    

---

## 6.8 Evaluate

The runtime may evaluate whether its policies worked well.

For example:

- Was decomposition useful?
    
- Did the chosen model succeed?
    
- Was the context sufficient?
    
- Did parallel execution help?
    
- Was escalation necessary?
    
- Did a knowledge item contribute?
    
- Which verification step caught the problem?
    

This information can inform future policy changes.

---

# 7. Capabilities

## 7.1 Capability philosophy

> **OpenASER depends on capabilities, not products.**

Users configure implementations for stable capability contracts.

---

## 7.2 Capability overview

|Capability|Responsibility|
|---|---|
|`Orchestrator`|Schedule and coordinate work|
|`WorkStore`|Persist tasks and dependency state|
|`DecisionEngine`|Provide bounded classification decisions|
|`Executor`|Run a coding agent|
|`ModelProvider`|Supply model inference|
|`Sandbox`|Restrict execution|
|`SourceControl`|Track repository state and changes|
|`WorkspaceIsolation`|Isolate concurrent workers|
|`Verifier`|Evaluate completion claims|
|`ArtifactStore`|Persist run outputs|
|`KnowledgeStore`|Persist durable project knowledge|
|`KnowledgeIndex`|Search project knowledge|
|`RuntimeStore`|Persist runs, evidence, metrics and metadata|
|`CodeIntelligence`|Discover repository relationships|
|`SecretProvider`|Safely provide credentials|
|`HumanApprovalProvider`|Handle explicit human approval|

---

## 7.3 Orchestrator

The `Orchestrator` coordinates execution.

Responsibilities may include:

- scheduling,
    
- dependency handling,
    
- retries,
    
- worker lifecycle,
    
- parallel execution,
    
- waiting,
    
- cancellation,
    
- and supervision.
    

Current candidate implementation:

```
Gas City
```

OpenASER should not require Gas City semantically.

---

## 7.4 WorkStore

The `WorkStore` persists engineering work state.

Responsibilities include:

- tasks,
    
- dependencies,
    
- status,
    
- ownership,
    
- ready/blocked state,
    
- continuation.
    

Current candidate:

```
Beads
```

Possible alternatives could include:

- local SQLite,
    
- Git-backed task state,
    
- issue trackers,
    
- custom enterprise stores.
    

---

## 7.5 DecisionEngine

The `DecisionEngine` supplies typed classification decisions.

Possible implementations:

```
Jev
local classifier
small local model
deterministic rule engine
```

It should return bounded outputs rather than arbitrary prose wherever possible.

---

## 7.6 Executor

The `Executor` actually runs an engineering agent.

Conceptually:

```
Executor.start(run)
Executor.status(run)
Executor.result(run)
Executor.cancel(run)
```

Current candidate:

```
OpenCode
```

Potential alternatives:

- other coding harnesses,
    
- OpenHands,
    
- Claude-based harnesses,
    
- Codex-style harnesses,
    
- enterprise internal agents.
    

OpenASER is:

> **harness-independent, not harness-free.**

---

## 7.7 ModelProvider

The `ModelProvider` represents access to model inference.

Possible implementations:

```
vLLM + local model
remote OpenAI-compatible API
provider-specific APIs
```

The runtime may care about properties such as:

- context capacity,
    
- model capabilities,
    
- latency,
    
- cost,
    
- tool support,
    
- structured output support,
    
- historical success.
    

---

## 7.8 Sandbox

The `Sandbox` constrains worker actions.

Responsibilities may include:

- filesystem boundaries,
    
- network restrictions,
    
- process permissions,
    
- resource limits,
    
- secret isolation,
    
- environment policies.
    

Current candidate:

```
OpenShell
```

---

## 7.9 SourceControl

`SourceControl` provides repository history and change tracking.

Likely default:

```
Git
```

OpenASER may use it for:

- diffs,
    
- commits,
    
- rollback,
    
- history,
    
- integration,
    
- attribution.
    

---

## 7.10 WorkspaceIsolation

This capability provides isolated work environments.

Likely initial implementation:

```
Git worktrees
```

Other mechanisms may eventually include:

- containers,
    
- snapshots,
    
- remote environments.
    

---

## 7.11 Verifier

The `Verifier` evaluates completion claims.

OpenASER owns the verification semantics.

Actual checks are typically supplied by the project.

Examples:

```
cargo test
pytest
npm test
tsc
cargo clippy
eslint
custom acceptance scripts
benchmarks
```

---

## 7.12 ArtifactStore

The `ArtifactStore` persists outputs.

Likely v0 implementation:

```
filesystem + Git
```

---

## 7.13 KnowledgeStore

The `KnowledgeStore` persists durable project knowledge.

Likely initial implementation:

```
structured Markdown + YAML frontmatter
```

The domain concept is `Knowledge`, not “Markdown.”

---

## 7.14 KnowledgeIndex

The `KnowledgeIndex` supports efficient search over knowledge.

Likely v0:

```
SQLite FTS5
```

Potential future implementations:

- embeddings,
    
- vector search,
    
- remote indexes,
    
- hybrid retrieval.
    

---

## 7.15 RuntimeStore

The `RuntimeStore` stores machine-oriented OpenASER state.

Likely initial implementation:

```
SQLite
```

Possible contents:

- runs,
    
- evidence,
    
- metrics,
    
- retrieval metadata,
    
- provider state,
    
- policy evaluation data,
    
- execution metadata.
    

---

## 7.16 CodeIntelligence

`CodeIntelligence` helps OpenASER and workers understand repository structure.

Baseline implementation:

```
filesystem
+ ripgrep
+ Git
```

Optional richer implementations:

```
LSP
AST analysis
Graphify
dependency graph
semantic index
```

---

## 7.17 SecretProvider

`SecretProvider` gives workers access to credentials without persisting secrets as normal project state.

Possible implementations:

- environment variables,
    
- operating-system keychain,
    
- secret managers,
    
- CI secret stores.
    

---

## 7.18 HumanApprovalProvider

Some operations should be able to require explicit human authorization.

Examples:

- destructive actions,
    
- production changes,
    
- expensive escalation,
    
- security-sensitive changes,
    
- ambiguous acceptance.
    

Initial implementation may simply be the OpenASER CLI.

---

# 8. Shipped and Recommended Implementations

A likely initial stack is:

|Capability|Initial implementation|
|---|---|
|OpenASER runtime|Rust|
|Orchestrator|Gas City|
|WorkStore|Beads|
|DecisionEngine|Jev or local/open classifier|
|Executor|OpenCode|
|Sandbox|OpenShell|
|SourceControl|Git|
|WorkspaceIsolation|Git worktrees|
|RuntimeStore|SQLite|
|KnowledgeStore|Markdown + YAML|
|KnowledgeIndex|SQLite FTS5|
|Basic CodeIntelligence|`rg` + Git + filesystem|
|Advanced CodeIntelligence|Graphify, optional|
|Local inference|vLLM|
|Local model|capable Qwen-class model|
|Remote inference|provider APIs|
|Verification|project-native engineering tools|
|ArtifactStore|filesystem + Git|

These are not permanent architectural requirements.

They are current implementations of capability contracts.

---

# 9. Why Rust for the OpenASER Runtime

Rust is a strong candidate for the OpenASER core because OpenASER is primarily systems software.

It manages:

- child processes,
    
- persistent state,
    
- concurrency,
    
- timeouts,
    
- cancellation,
    
- filesystems,
    
- Git,
    
- workspaces,
    
- external services,
    
- failure recovery,
    
- and state transitions.
    

Rust offers:

- strong type safety,
    
- explicit error handling,
    
- resource safety,
    
- predictable runtime behavior,
    
- good CLI tooling,
    
- strong concurrency support,
    
- and straightforward binary distribution.
    

The reason is not primarily raw performance.

The more important property is:

> **OpenASER is reliability-oriented systems software surrounding unreliable external components.**

The implementation language itself remains a project decision rather than an architectural requirement.

---

# 10. Configuration

## 10.1 Configuration philosophy

> **Capabilities define what OpenASER needs. Providers define how those capabilities are implemented.**

Configuration should select providers for capabilities.

Good:

```
executor:
  provider: opencode
```

Less desirable:

```
opencode:
  enabled: true
```

The first expresses architecture.

The second merely exposes dependencies.

---

## 10.2 Default configuration

A default profile might look like:

```
orchestrator:
  provider: gas-city

work:
  provider: beads

decision_engine:
  provider: jev

executor:
  provider: opencode

sandbox:
  provider: openshell

source_control:
  provider: git

workspace:
  provider: git-worktree

runtime_store:
  provider: sqlite

knowledge:
  store: markdown
  index: sqlite-fts

code_intelligence:
  provider: basic
```

---

## 10.3 Configuration precedence

Configuration should be layered:

```
built-in defaults
       ↓
user configuration
       ↓
project configuration
       ↓
CLI overrides
```

Precedence:

```
CLI
 >
project
 >
user
 >
built-in defaults
```

---

## 10.4 User configuration

Machine/user-specific settings may live in:

```
~/.config/openaser/config.yaml
```

Typical contents:

- preferred models,
    
- local inference endpoint,
    
- preferred executor,
    
- machine-specific sandbox setup,
    
- API provider configuration.
    

---

## 10.5 Project configuration

Project-specific configuration may live in:

```
.openaser/config.yaml
```

Typical contents:

- verification policy,
    
- project workflows,
    
- knowledge settings,
    
- repository-specific execution constraints,
    
- required capabilities,
    
- project conventions.
    

---

## 10.6 Switching providers

Replacing an implementation should be simple.

Example:

```
executor:
  provider: another-harness
```

or:

```
code_intelligence:
  provider: graphify
```

Changing providers should not require changing OpenASER's domain model.

---

## 10.7 Multiple providers

Some capabilities may use several providers simultaneously.

Example:

```
models:
  coding:
    provider: local-vllm
    model: qwen

  difficult:
    provider: remote
    model: frontier

  review:
    provider: remote-review
```

Code intelligence may also be compositional:

```
code_intelligence:
  providers:
    - basic
    - graphify
```

---

## 10.8 Profiles

OpenASER may ship reusable profiles.

Examples:

```
local
cloud
minimal
custom
```

A local profile might favor:

- local inference,
    
- local storage,
    
- local classifier,
    
- local sandboxing.
    

A cloud profile might use remote models and distributed workers.

---

## 10.9 Inspecting the active stack

A command such as:

```
openaser stack
```

could show:

```
Orchestrator       gas-city
Work store         beads
Decision engine    jev
Executor           opencode
Sandbox            openshell
Source control     git
Workspace          git-worktree
Runtime store      sqlite
Knowledge store    markdown
Knowledge index    sqlite-fts
Code intelligence  basic
Coding model       local-vllm/qwen
```

And:

```
openaser stack --why
```

could show where each value came from.

---

## 10.10 Configuration validation

A command such as:

```
openaser config check
```

should validate provider requirements before execution.

Example:

```
✓ orchestrator: gas-city
✓ work store: beads
✓ executor: opencode
✓ runtime store: sqlite
✗ sandbox: openshell
  executable not found
```

---

# 11. Task Lifecycle

## 11.1 Overview

```
Goal
 ↓
Task
 ↓
Decision
 ↓
Workflow
 ↓
Run
 ↓
Prepare
 ↓
Execute
 ↓
CompletionClaim
 ↓
Verify
 ↓
Record
 ↓
Complete / Retry / Escalate / Continue
```

---

## 11.2 Simple task

A trivial task may require:

```
Task
 ↓
classify as simple
 ↓
one Run
 ↓
one worker
 ↓
verification
 ↓
complete
```

OpenASER should not add multi-agent machinery merely because it exists.

---

## 11.3 Complex task

A complex task may require:

```
Goal
 ↓
classify as complex
 ↓
investigation
 ↓
decomposition
 ↓
Task A ──┐
Task B ──┼── dependencies / parallelism
Task C ──┘
 ↓
integration
 ↓
system-level verification
 ↓
complete
```

---

## 11.4 Retry and escalation

A failed verification does not necessarily mean the entire task fails.

Possible policy:

```
verification failed
      ↓
worker retry?
      ↓
different strategy?
      ↓
stronger model?
      ↓
human escalation?
```

The actual policy should be explicit and measurable.

---

# 12. Verification and Evidence

## 12.1 Verification boundary

A worker may generate code and tests.

It does not have unilateral authority to accept its own output.

```
worker
  ↓
CompletionClaim
  ↓
Verifier
  ↓
Evidence
  ↓
accept / reject
```

---

## 12.2 Acceptance contracts

Tasks may define acceptance requirements.

Example:

```
acceptance:
  commands:
    - cargo test
    - cargo clippy

  conditions:
    - refresh tokens remain single-use
    - concurrent rotation produces only one valid successor
```

---

## 12.3 Deterministic verification

Preferred where available:

- test suites,
    
- builds,
    
- type checks,
    
- linting,
    
- static analysis,
    
- schema checks,
    
- benchmark thresholds,
    
- project scripts.
    

---

## 12.4 Probabilistic review

Some properties are difficult to verify mechanically.

Examples:

- architecture quality,
    
- maintainability,
    
- suspicious edge cases,
    
- design consistency.
    

Model-based review can supplement deterministic verification.

It should remain distinguishable from machine-verifiable evidence.

---

## 12.5 Human verification

Humans may remain necessary for:

- ambiguous requirements,
    
- high-risk actions,
    
- architecture decisions,
    
- security-sensitive operations,
    
- production changes.
    

OpenASER should support human approval rather than pretending every judgment can be automated.

---

# 13. Project Knowledge

## 13.1 Why knowledge exists

Some information should survive individual tasks because future work benefits from it.

Examples:

- architectural constraints,
    
- project conventions,
    
- subtle behavior,
    
- known traps,
    
- decisions,
    
- verified lessons.
    

---

## 13.2 Work state is not knowledge

```
Work:
"What are we doing?"

Knowledge:
"What does the project know?"

Evidence:
"Why do we believe it?"

Artifact:
"What did the work produce?"
```

These should remain separate.

---

## 13.3 Knowledge candidates

Workers should not freely write permanent memory.

Instead:

```
Observation
   ↓
KnowledgeCandidate
   ↓
Consolidation
   ↓
Durable Knowledge
```

A candidate may include:

```
type: constraint
scope:
  - src/auth/**

summary:
  Refresh-token rotation requires
  per-session atomicity.

evidence:
  - tests/auth/refresh_race.rs
```

---

## 13.4 Consolidation

Before promotion, OpenASER may check:

- duplication,
    
- conflict,
    
- scope,
    
- evidence,
    
- freshness,
    
- relevance,
    
- existing knowledge,
    
- supersession.
    

---

## 13.5 Forgetting and supersession

More memory is not automatically better.

Knowledge may become:

- stale,
    
- redundant,
    
- contradicted,
    
- superseded,
    
- irrelevant.
    

Therefore forgetting and supersession are first-class operations.

The goal is:

> **Preserve information that improves future engineering work.**

---

# 14. Context and Discovery

## 14.1 Context philosophy

OpenASER is not a context-minimization product.

Its requirement is:

> **A project should not require its entire state or history to fit into one model context.**

Useful context should be provided when it fits and helps.

Larger-capacity models should be able to receive more.

---

## 14.2 Initial context

Initial context may contain:

- task description,
    
- acceptance criteria,
    
- project instructions,
    
- relevant files,
    
- related tests,
    
- relevant knowledge,
    
- explicit constraints.
    

---

## 14.3 On-demand discovery

Workers should also be able to discover additional information.

Possible channels:

```
filesystem
ripgrep
Git
tests
LSP
dependency analysis
Graphify
knowledge search
documentation
semantic search
```

Initial context is a starting point, not an information cage.

---

## 14.4 Context provenance

Automatically selected context should be attributable.

Example:

```
context:
  - source: knowledge/auth-rotation
    reason: scope_match
    tokens: 184

  - source: tests/auth/refresh.rs
    reason: related_test
    tokens: 412
```

This allows later evaluation.

---

## 14.5 Context capacity

OpenASER should support practical local-model contexts such as approximately 64K.

But the same runtime should also exploit:

- 128K,
    
- 200K,
    
- 1M,
    
- or future larger contexts
    

when available and useful.

---

# 15. Models and Compute

## 15.1 Models are replaceable workers

OpenASER should support:

- open models,
    
- proprietary models,
    
- local models,
    
- remote models,
    
- specialized models.
    

The runtime should not pretend they behave identically.

---

## 15.2 Harness portability vs model behavior

Portable infrastructure does not mean behavioral model agnosticism.

Models differ in:

- reasoning,
    
- tool use,
    
- coding quality,
    
- instruction adherence,
    
- structured output,
    
- context utilization,
    
- latency,
    
- and cost.
    

Policies may therefore be model-specific.

---

## 15.3 Local inference

A typical local path could be:

```
OpenCode
   ↓
OpenAI-compatible endpoint
   ↓
vLLM
   ↓
local model
   ↓
GPU
```

OpenASER should not need to know model-server internals.

---

## 15.4 Logical concurrency

Multiple workers do not require multiple model copies.

```
Worker A ─┐
Worker B ─┼──► shared inference server ─► model
Worker C ─┘
```

Workers may time-share inference while independently performing:

- file operations,
    
- tests,
    
- builds,
    
- repository search.
    

---

## 15.5 Model routing

OpenASER may route work based on:

- complexity,
    
- task type,
    
- context requirements,
    
- previous failures,
    
- cost,
    
- latency,
    
- local availability,
    
- historical success.
    

Example:

```
routine work
   ↓
local model

hard reasoning
   ↓
stronger model

local attempt fails
   ↓
escalate
```

---

# 16. Orchestration and Parallel Work

## 16.1 What orchestration means

The orchestration layer handles mechanics such as:

- task readiness,
    
- worker scheduling,
    
- dependencies,
    
- retries,
    
- waiting,
    
- cancellation,
    
- concurrency,
    
- worker lifecycle.
    

Gas City is a current candidate implementation.

---

## 16.2 Parallelism is conditional

OpenASER should parallelize only when useful.

Potential benefits:

- reduced wall-clock time,
    
- independent investigation,
    
- simultaneous implementation.
    

Potential costs:

- conflicts,
    
- duplicated work,
    
- integration overhead,
    
- increased model use.
    

Therefore the DecisionEngine and PolicyEngine may decide:

```
parallelism = none
parallelism = independent-subtasks
parallelism = investigation-only
```

rather than always spawning multiple agents.

---

## 16.3 Workspace isolation

Parallel coding workers should generally operate in isolated workspaces.

Git worktrees are a practical default.

Example:

```
main repository
   │
   ├── worktree A → worker A
   ├── worktree B → worker B
   └── worktree C → worker C
```

---

# 17. Sandboxing and Security

Agent-generated commands form a security boundary.

OpenASER should support:

- filesystem restrictions,
    
- network restrictions,
    
- process limits,
    
- secret isolation,
    
- command auditing,
    
- dependency-installation rules,
    
- human approval.
    

A model instruction such as:

```
"Do not access X."
```

is not a substitute for an actual permission boundary.

---

# 18. Failure Recovery and Resumability

## 18.1 Failure is expected

OpenASER should treat the following as normal operational conditions:

- agent crash,
    
- test failure,
    
- model outage,
    
- provider timeout,
    
- process termination,
    
- machine restart,
    
- worktree conflict,
    
- verification failure.
    

---

## 18.2 Fresh-agent continuation

A useful runtime test is:

```
Agent A
  ↓
significant work
  ↓
agent disappears
  ↓
durable state
  ↓
Agent B
  ↓
continue correctly
```

If continuation requires manually rebuilding the previous chat conversation, too much important state remained implicit.

---

# 19. Evaluation and Adaptation

## 19.1 Why measure the runtime

OpenASER makes many choices:

- workflow,
    
- model,
    
- context,
    
- decomposition,
    
- parallelism,
    
- verification,
    
- retry behavior.
    

Those choices should be evaluated.

---

## 19.2 Metrics

Possible metrics include:

- verified completion rate,
    
- regression rate,
    
- human intervention,
    
- wall-clock time,
    
- tokens,
    
- model inference time,
    
- cost,
    
- retries,
    
- retrieval calls,
    
- integration failures,
    
- context usage,
    
- resumability,
    
- worker count.
    

---

## 19.3 Policy evaluation

A policy change should ideally follow:

```
baseline
   ↓
candidate policy
   ↓
evaluation workload
   ↓
metrics
   ↓
comparison
   ↓
adopt / reject
```

---

## 19.4 Examples

If larger context performs better:

```
use larger context
```

If Graphify does not improve outcomes:

```
do not require Graphify
```

If three workers perform worse than one:

```
avoid unnecessary parallelism
```

If a local model consistently fails one task class:

```
route that class elsewhere
```

If retained knowledge hurts future tasks:

```
improve consolidation / forgetting
```

---

# 20. Installation and Distribution

OpenASER should exist primarily as an open-source repository and installable developer tool.

The repository contains:

- runtime source,
    
- provider adapters,
    
- schemas,
    
- documentation,
    
- examples,
    
- tests,
    
- release tooling.
    

Users should normally install OpenASER rather than clone the source merely to use it.

Possible distribution:

```
GitHub repository
      ↓
releases
      ↓
prebuilt OpenASER binary
      ↓
user installation
```

Potential install methods:

```
package manager
cargo install
prebuilt release
install script
```

---

# 21. Setup

A setup process might look like:

```
openaser setup
```

OpenASER could inspect the environment:

```
Git                  ✓
Gas City             ✓
Beads                 ✓
OpenCode              ✓
OpenShell             ✗ optional
local inference       ✓
```

Missing components could be installed, configured, or reported.

---

# 22. Project Initialization

A developer could run:

```
cd my-project
openaser init
```

Possible project structure:

```
my-project/
├── .openaser/
│   ├── config.yaml
│   ├── workflows/
│   └── knowledge/
├── src/
├── tests/
└── ...
```

Committed OpenASER state may include:

```
.openaser/config.yaml
.openaser/workflows/
.openaser/knowledge/
```

Machine-local operational data may live elsewhere.

---

# 23. Usage

## 23.1 Run work

```
openaser run "Fix the authentication race condition"
```

OpenASER may internally:

```
create Task
    ↓
classify
    ↓
select Workflow
    ↓
prepare Run
    ↓
select executor/model
    ↓
execute
    ↓
verify
    ↓
record evidence
    ↓
complete
```

---

## 23.2 Status

```
openaser status
```

Possible output:

```
Task: auth-172

Status: verifying

Workflow:
  bugfix

Worker:
  opencode

Model:
  local-qwen

Verification:
  ✓ unit tests
  … integration tests
```

---

## 23.3 Resume

```
openaser resume
```

This should resume durable work without requiring the original agent session.

---

## 23.4 Inspect stack

```
openaser stack
```

---

## 23.5 Inspect configuration

```
openaser config
```

---

## 23.6 Validate environment

```
openaser doctor
```

---

# 24. Extending OpenASER

Providers implement capability contracts.

Potential extension points include:

```
ExecutorProvider
OrchestratorProvider
WorkStoreProvider
DecisionEngineProvider
ModelProvider
SandboxProvider
KnowledgeStoreProvider
KnowledgeIndexProvider
CodeIntelligenceProvider
SecretProvider
```

A provider should declare:

- capability,
    
- configuration schema,
    
- supported features,
    
- version,
    
- health/availability,
    
- required external dependencies.
    

---

# 25. OpenASER-Owned Semantics vs External Mechanisms

This is one of the most important boundaries.

## OpenASER owns

```
Task / Run semantics
Workflow semantics
Decision policy
Verification boundary
CompletionClaim
Evidence model
Knowledge lifecycle
Context policy
Model/resource policy
Recovery semantics
Evaluation
Adaptation
```

## External systems implement mechanisms

```
Gas City       orchestration
Beads          work persistence
OpenCode       agent harness
Jev            bounded classification
OpenShell      sandboxing
Git            source control
SQLite         database
Graphify       code graph
vLLM           inference serving
```

The runtime should remain understandable even if every concrete mechanism changes.

---

# 26. Default Stack

A plausible v0:

```
OpenASER core
   Rust

Decision
   Jev / local classifier / rules

Orchestration
   Gas City

Work state
   Beads

Execution
   OpenCode

Isolation
   Git worktrees
   OpenShell where required

Source control
   Git

Runtime state
   SQLite

Knowledge
   Markdown + YAML

Knowledge search
   SQLite FTS5

Code discovery
   filesystem
   ripgrep
   Git

Advanced code intelligence
   optional Graphify

Local inference
   vLLM

Local models
   capable Qwen-class model

Remote models
   optional provider APIs

Verification
   project-native engineering tools
```

---

# 27. What OpenASER Should Not Build Initially

OpenASER should avoid prematurely building:

- its own coding harness,
    
- its own inference server,
    
- its own orchestration infrastructure if Gas City suffices,
    
- a custom vector database,
    
- a graph database,
    
- a huge ontology,
    
- mandatory embeddings,
    
- a large workflow DSL,
    
- a plugin marketplace,
    
- mandatory Graphify,
    
- mandatory Obsidian,
    
- unrestricted self-modification.
    

The initial system should remain small enough to evaluate.

---

# 28. v0

A practical v0 can consist of:

```
OpenASER core
+ DecisionEngine
+ Gas City
+ Beads
+ OpenCode
+ Git/worktrees
+ SQLite
+ Markdown knowledge
+ rg/Git discovery
+ project verification
+ local and/or remote model
```

The first vertical slice should prove:

1. A user can submit a real engineering task.
    
2. The runtime can classify the task.
    
3. The appropriate workflow is selected.
    
4. A coding agent can perform the work.
    
5. The worker can discover repository information.
    
6. Verification is independent from the worker's completion claim.
    
7. Evidence is stored.
    
8. Work can survive worker termination when necessary.
    
9. A fresh agent can continue if required.
    
10. The result is auditable.
    

---

# 29. Evaluation Plan

Important comparisons include:

```
coding agent directly

vs

orchestrator + coding agent

vs

OpenASER + coding agent
```

Keep constant:

- model,
    
- harness,
    
- repository,
    
- task.
    

Measure:

- verified completion,
    
- regressions,
    
- human intervention,
    
- time,
    
- compute,
    
- tokens,
    
- retry rate,
    
- continuation quality.
    

---

# 30. Architectural Boundaries

Keep the following distinctions explicit:

```
work state        ≠ knowledge

knowledge         ≠ knowledge index

retrieval         ≠ context

context           ≠ conversation history

model             ≠ inference server

model             ≠ agent

agent             ≠ harness

harness           ≠ runtime

orchestration     ≠ engineering policy

classification    ≠ policy

observation       ≠ durable knowledge

agent confidence  ≠ verification

completion claim  ≠ completion

parallelism       ≠ progress

more agents       ≠ better engineering

logs              ≠ evidence
```

---

# 31. Architectural Tests

## 31.1 Replacement test

Can a provider be replaced without changing OpenASER's domain semantics?

Example:

```
OpenCode
   ↓ replace
another Executor
```

If core task/run semantics must change, the abstraction is probably leaking.

---

## 31.2 Deletion test

Deleting an optional component should degrade a capability, not destroy the architecture.

Examples:

```
No Graphify
→ weaker structural discovery

No embeddings
→ weaker semantic retrieval

No Gas City
→ another orchestrator required

No OpenShell
→ another sandbox required

No local model
→ remote models still work
```

---

## 31.3 Resumability test

```
start task
 ↓
perform work
 ↓
kill worker
 ↓
restart runtime
 ↓
fresh worker
 ↓
continue
```

---

## 31.4 Verification test

A deliberately incorrect worker completion claim must not automatically complete the task.

---

## 31.5 Simplicity test

A trivial task should not require a complex multi-agent workflow.

---

# 32. Product Identity

OpenASER is not primarily:

```
orchestration
memory
RAG
context optimization
local-model tooling
multi-agent software
```

Those are capabilities or mechanisms.

The product is:

> **An open-source runtime for reliable agentic software engineering.**

It can scale from:

```
one task
one worker
one run
```

to:

```
large goal
many tasks
multiple workers
multiple models
multiple runs
recovery
integration
continued project state
```

without requiring the user to switch to a different architecture.

---

# 33. Core Statements

## Product statement

> **OpenASER is an open-source runtime for reliable agentic software engineering. It coordinates replaceable coding agents around explicit engineering workflows, durable state where required, independent verification, project knowledge, model and compute resources, and measurable runtime policies.**

## Engineering-state statement

> **Engineering state should survive for as long as the work requires.**

## Verification statement

> **Agent output is a proposal. Evidence determines acceptance.**

## Complexity statement

> **Task complexity should determine runtime complexity.**

## Context statement

> **Use available context efficiently without requiring the full project state or history to fit inside one model context.**

## Capability statement

> **OpenASER depends on capabilities, not products.**

## Infrastructure statement

> **Own the engineering semantics. Reuse and replace the mechanisms.**

## Adaptation statement

> **Policies are hypotheses. Measure them.**

---

# 34. Final Mental Model

```
                         OpenASER
             Agentic Software Engineering Runtime

                              │
                              ▼
                            GOAL
                              │
                              ▼
                             WORK
                              │
                              ▼
                           DECIDE
                              │
                    DecisionEngine
                              │
                              ▼
                           WORKFLOW
                              │
                              ▼
                           PREPARE
                              │
            ┌─────────────────┼─────────────────┐
            ▼                 ▼                 ▼
         Context           Workspace          Model
            │                 │                 │
            └─────────────────┼─────────────────┘
                              ▼
                           EXECUTE
                              │
                    Executor / Agent
                              │
                              ▼
                         ARTIFACTS
                              │
                              ▼
                    COMPLETION CLAIM
                              │
                              ▼
                           VERIFY
                         /        \
                      FAIL        PASS
                       │            │
                       ▼            ▼
                  retry/escalate   RECORD
                                    │
                        ┌───────────┼───────────┐
                        ▼           ▼           ▼
                    Artifacts    Evidence    Knowledge
                                              Candidate
                                                 │
                                                 ▼
                                            Consolidate
                                                 │
                                                 ▼
                                             Knowledge
                                                 │
                                                 ▼
                                              EVALUATE
                                                 │
                                                 ▼
                                              Policies
                                                 │
                                                 └──────────↺
```

Underneath this architecture sit replaceable implementations:

```
Capability             Current implementation candidate

Orchestrator        →  Gas City
WorkStore           →  Beads
DecisionEngine      →  Jev / local classifier / rules
Executor            →  OpenCode
Sandbox             →  OpenShell
SourceControl       →  Git
WorkspaceIsolation  →  Git worktrees
RuntimeStore        →  SQLite
KnowledgeStore      →  Markdown
KnowledgeIndex      →  SQLite FTS5
CodeIntelligence    →  rg + Git / Graphify
ModelProvider       →  vLLM / remote APIs
Verifier            →  project-native tools
```

The left-hand side defines OpenASER's needs.

The right-hand side is replaceable.

---

# 35. Final Definition

> **OpenASER is an open-source Agentic Software Engineering Runtime that manages the engineering system around coding agents. It classifies work into appropriate workflows, coordinates execution through replaceable infrastructure, maintains engineering state for as long as required, verifies agent output independently, records artifacts and evidence, preserves useful project knowledge, manages models, context and compute, recovers from failure, and evaluates its own runtime policies using measured engineering outcomes.**

The goal is not maximal agent autonomy.

The goal is not maximal context reduction.

The goal is not maximal parallelism.

The goal is:

> **Reliable agentic software engineering, with the runtime using only as much complexity as the work actually requires.**