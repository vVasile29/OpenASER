# OpenASER

Open Agentic Software Engineering Runtime.

OpenASER is the integration glue around coding agents and engineering tools,
not a replacement for them. Narrow capability contracts connect replaceable
implementations for execution, work management, verification, state, knowledge,
models, isolation, and recovery. Existing tools do the heavy lifting; OpenASER
owns the engineering decisions and the connections between those tools.
Simple work should stay simple; larger work should use only the capabilities it needs.

**Status:** planning only. No runtime, adapters, or CLI commands are implemented.
This README is the concise implementation plan. [OpenASER.md](OpenASER.md) remains
unchanged as background documentation; its broader proposals do not authorize work.

## Working Rules

- Implement only requirements explicitly agreed with the user. Do not infer features from the broader design.
- Document only agreed decisions. Anything not documented as agreed is outside the approved scope by default.
- Ask before choosing the next milestone, resolving an unclear requirement, or expanding scope.
- Before coding, record the milestone's behavior, required capabilities, acceptance checks, and exclusions here.
- Implement one approved milestone at a time. Verify it, report the result, and ask how to continue.
- Keep changes small and understandable. No speculative helpers, configuration, abstractions, or dependencies.
- Each implemented capability gets a narrow contract and a concrete implementation. Do not expose every feature of the underlying tool.
- Reuse capabilities provided by existing tools; do not rebuild their mechanisms inside OpenASER.
- The planned implementations below are commitments to provide support, not requirements to build or enable everything in the first milestone.

## Architecture

OpenASER uses hexagonal architecture (ports and adapters). Its capability
contracts are the ports; tool-specific integrations are the adapters.

```text
Domain semantics -> Capability contracts -> Concrete implementations
```

OpenASER owns engineering goals, agreed constraints, workflow decisions, and
acceptance.
The tools supply execution, scheduling, storage, and other mechanisms without
defining the domain model. Flexibility means replaceable components, not a
framework that anticipates every possible feature.

### Engineering and Execution

The orchestrator owns dispatch, dependency coordination, execution/session
lifecycles, and execution-attempt bookkeeping. WorkStore persists the work
records. OpenASER uses the information required to request work and evaluate
results through narrow contracts, rather than reproducing the orchestrator's
task/session model, retry history, or execution registry.
Keep state with its established owners rather than introducing a separate
RuntimeStore or general runtime database. SQLite FTS5 remains a derived
knowledge-search implementation, not a duplicate orchestration store.

For split work, the selected engineering process determines the intended work
and acceptance requirements; the orchestrator coordinates its execution.
Reuse an orchestrator's existing WorkStore integration instead of creating a
competing task tracker or scheduler. Contracts expose required functionality,
not copies of a concrete tool's object model.

### Output Ownership

| Output | Existing Owner or Mechanism |
| --- | --- |
| Code changes | Git and the execution workspace |
| Execution results | Harness and orchestrator |
| Verification results | Verifier, consumed by OpenASER |
| Reusable findings and decisions | Project knowledge storage |

Access required outputs through the existing integrations and keep them
available for review and verification. This does not introduce a separate
ArtifactStore or an aggregate storage wrapper around these responsibilities.

### Contracts and Adapters

Toolchain integration uses narrow functionality contracts and tool-specific
adapters. Contracts define the required behavior, guarantees, and failure
reporting. Adapters translate those requirements into the tool's supported
commands or APIs, configuration, paths, and output formats.

```text
OpenASER engineering logic -> Capability contract -> Adapter -> Tool
```

This localizes compatibility work instead of spreading tool-specific handling
through the runtime. Use existing integration interfaces and expose only the
functionality OpenASER needs; do not mirror each tool's full feature set.

- Selecting another supported implementation does not change OpenASER's engineering logic.
- Supporting a new implementation adds an adapter that satisfies the same contract.
- Tool-specific details stay inside adapters, not vendor-specific branches in shared engineering logic.
- An adapter cannot supply a capability or guarantee the underlying tool lacks. Replaceability requires satisfying the contract, not merely fitting an interface.

### Workflow Frame

Workflows are a major part of the glue: they will describe how capabilities
cooperate to perform engineering work. Their structure has explicit steps,
permitted decision outcomes, and predefined transitions. Each decision node
specifies its mechanism: deterministic check, bounded agentic judgment, or human
input. Reaching that node invokes the specified mechanism; validating its answer
determines which predefined branch follows.

```text
Defined decision node -> Specified mechanism -> Validated answer -> Defined branch
```

Laya supplies judgments at specified nodes, not observable check results or human
authorization. It does not decide when it is called or invent workflow branches.
Invalid responses cannot become new transitions. OpenASER owns these engineering
semantics; the orchestrator supplies execution machinery rather than OpenASER
building a competing workflow engine. Concrete workflow design is deferred.

## Planned Implementations

The runtime will be written in Rust. The following implementations are planned;
none are implemented yet. Laya, the open-source variant specified by the user,
replaces Jev for agentic assessments. This table records the committed roles and
implementations, not detailed integration contracts.

| Abstract Role | Responsibility | Planned Implementation |
| --- | --- | --- |
| Orchestrator | Coordinate work, dependencies, and harness sessions; own execution bookkeeping. | Gas City |
| WorkStore | Persist tasks, status, and dependencies. | Beads |
| DecisionEngine | Return bounded assessments at specified decision points. | Laya; deterministic rules where appropriate |
| Harness | Perform the coding-agent loop within granted permissions. | OpenCode |
| Sandbox | Enforce execution permissions and resource boundaries. | OpenShell |
| SourceControl | Track repository history and changes. | Git |
| WorkspaceIsolation | Separate worker workspaces. | Git worktrees |
| Verifier | Check output against agreed acceptance criteria. | Project-native tests, builds, lint, and acceptance scripts |
| KnowledgeStore | Store durable project knowledge. | Markdown + YAML frontmatter |
| KnowledgeIndex | Search project knowledge. | SQLite FTS5 |

Support does not mean mandatory use: for example, Graphify and local inference
must not be required for every task.

## Runtime Boundaries

### Installation: Agreed Split

OpenASER ships as its own executable containing the Rust runtime and implemented
adapters. External tools such as OpenCode and Git are not bundled; OpenASER uses
compatible, independently installed tools. An adapter implements the connection
to a tool, not the tool itself.

Compiled library dependencies are distinct from external programs and services.
Each integration must identify what it needs and how compatibility is checked;
OpenASER must report missing or incompatible prerequisites rather than silently
proceed. Installing OpenASER does not automatically install or update external
tools.

### Repository Initialization: Agreed Purpose

The terminal is the entry point. `openaser init` prepares a repository for later
OpenASER use, analogous in purpose to `git init`. It does not install external
tools or launch a harness.
Keep initialization as the entry point for repository-specific preparation and
configuration as concrete needs emerge. Do not invent settings or structures
to give the command something to initialize.

### Configuration: Agreed Format and Locations

OpenASER settings use TOML: a readable format for explicit settings with mature
Rust support. No second configuration format is planned.

- User defaults: `$XDG_CONFIG_HOME/openaser/config.toml`, defaulting to `~/.config/openaser/config.toml` when `XDG_CONFIG_HOME` is unset or empty.
- Repository settings: `.openaser/config.toml` at the repository root.

The user location follows the XDG convention; the repository location is an
OpenASER convention. Harness configuration remains owned by the harness.

### Interactive Entry Point

Running `openaser` prepares the execution environment and launches the selected
harness's native interactive TUI. Users enter work and continue the conversation
there, rather than supplying long task descriptions as command-line arguments.
OpenASER does not implement a replacement chat TUI.

```text
User runs openaser -> Prepared environment -> Selected harness's native TUI
```

### Harness: Agreed Boundary

The harness owns the coding-agent loop: model interaction, repository
investigation, edits, and command execution within granted permissions.
OpenASER supplies the work and constraints, observes the attempt, and owns
acceptance. The harness cannot declare its own output accepted.

OpenCode is the planned harness implementation. An OpenASER adapter connects
to it; OpenASER does not implement a second coding-agent loop.

### Models and Inference

Coding inference belongs to the harness: reuse its native provider configuration,
model requests, and supported authentication flows. Support local models through
vLLM or Ollama, remote API-key providers, and supported subscription login such
as OpenCode's ChatGPT integration. Availability depends on the selected harness
and provider; subscription login is not interchangeable with an API key.
OpenASER does not add a separate ModelProvider, inference client, or provider
registry for the same coding loop. Endpoint access and required authentication
must respect the execution boundary. Laya's inference belongs to its own
decision-engine integration, not assumed reuse of coding subscription credentials.

### Verifier: Agreed Boundary

The verifier establishes and reports the results of supplied checks through
actual execution, not the harness's assertion that they passed. OpenASER
evaluates those results against the agreed acceptance requirements.
Reuse project-native tests, builds, linters, and acceptance scripts through the
agreed execution environment rather than building a separate test framework.
Check results establish what was verified, not blanket proof of correctness.

### Project Knowledge

KnowledgeStore exposes reusable project decisions, constraints, conventions,
and supported findings, separate from orchestration bookkeeping. Store knowledge
in one separate, centralized vault scoped by repository and branch, rather than
distributing notes across source repositories. Use Markdown with YAML frontmatter
and supporting references; writing a claim does not make it authoritative.
Humans and agents can use the files directly within their allowed scope.
Obsidian is an optional view/editor, not a runtime dependency.

Specific scope tags follow a consistent repository/branch hierarchy; backlinks
express relationships between notes without granting access to another scope.
Use these scopes rather than copying whole note collections for each branch.
Knowledge access is restricted to the relevant repository and branch. Branch-local
information must not flow into another branch's retrieved context. Tags and links
describe scope; the knowledge-access boundary enforces it. A centralized lookup
location does not give each agent access to the entire vault.

### Knowledge Index

KnowledgeIndex locates relevant notes within the permitted repository/branch
scope. SQLite FTS5 is the planned implementation. Its search data is derived
from the Markdown vault and rebuildable; the vault remains the source of note
content. The index does not create knowledge, judge its truth, or automatically
include every match in an agent's context.

### Context Gathering

The harness gathers and uses working context through its existing tools.
OpenASER supplies the authorized assignment, constraints, relevant instructions,
and scoped knowledge access rather than building a separate context engine.
Sources include the execution workspace, Git, the knowledge vault, relevant
orchestrator state, and verification results. Start with the assignment context
and discover additional information as needed, not entire repositories or vaults.
Reuse native filesystem, ripgrep, and Git discovery; Graphify is an optional
richer discovery integration, not a separate OpenASER CodeIntelligence subsystem.

Knowledge access supports scoped search, reading selected notes, and following
links within the same permitted scope. Scope comes from the execution context,
not an agent's guess. Retrieved information does not grant authority or override
explicit authorization requirements. Laya is invoked at defined decision nodes,
not for every retrieval. Vault notes do not replace current code and evidence.

### Sandbox: Agreed Boundary

The sandbox is an enforced execution environment for the harness and its child
processes, including access to working files. OpenASER specifies permitted
access; the sandbox enforces it. OpenShell is the planned implementation.
A separate working directory alone is not a security boundary.

### Credential Handling

Reuse the harness's authentication, existing credential storage, and sandbox
controls through integration adapters rather than adding a separate SecretProvider.
Make required authentication available deliberately; do not automatically inherit
unrelated host credentials or the user's complete environment. Ordinary coding
access does not grant remote publication authority. Keep secret values out of
project configuration, knowledge notes, and prompts.

### Source Control and Workspaces

SourceControl provides repository history and change operations; WorkspaceIsolation
provides execution working files and separation where required. Git and Git
worktrees are the planned mechanisms, not independent OpenASER controllers.
Delegate workspace preparation and lifecycle to the orchestrator's facilities
rather than building a competing workspace manager. OpenASER's engineering
process specifies permitted operations and required separation; adapters apply
those requirements through the orchestrator. Workspace separation does not
replace sandbox enforcement or establish that changes are safe to integrate.

### Remote Publication

Publishing code to a remote repository requires explicit user authorization.
Neither the harness nor orchestration workflows may automatically push code or
publish equivalent changes through a hosting API. Completing work or accepting
its result does not grant permission to publish it. Enforce this boundary for
all execution paths rather than relying on agent instructions.

### Automated Assessment and Human Authority

Laya supplies bounded assessments; OpenASER applies agreed rules within the
user's existing authorization. Assessments do not grant permissions or change
the user's intentions. Changes to goals, expansion of scope, and additional
authority require a human decision unless already explicitly authorized.
Supplied check results are determined by the checks, not model judgment.
Agents investigate factual questions within their assignment; human requests
are for input or authority they cannot establish from existing requirements.

### Human Interaction: Action Inbox

A replaceable human-interaction capability collects outstanding requests from
agents and subagents at any level into an action inbox, rather than a stream of
notification overlays. It shows the source, question, and available answers,
routes a response to the originating request, and offers session navigation
where supported. Reading a request does not resolve it or grant permission.
Sending an ordinary message is not a substitute for answering a waiting request.
The inbox complements the harness's native TUI; it does not replace it.
Questions, choices, and approval requests use this capability rather than a
separate HumanApprovalProvider. The inbox presents requests and routes responses;
OpenASER's rules determine their authorized scope, and execution controls enforce
the resulting permissions.

```text
Agent request -> Integration adapter -> Action inbox -> Human response
                      ^                                      |
                      +-------- Original request <-----------+
```

### Orchestrator: Agreed Boundary

The orchestrator owns harness-session coordination and execution bookkeeping;
Gas City is the planned implementation. OpenASER retains engineering decisions
and acceptance rather than building a competing session launcher and supervisor.

Session coordination must not require the user to operate a fleet of separate
harness terminals waiting for input.

The orchestrator-managed parent worker owns the assigned work's claiming and
completion protocol. A harness may delegate internally to subagents; those
subagents do not independently own the outer assignment or its lifecycle.

OpenASER supplies work and a small, explicit set of required guarantees. The
adapter establishes those guarantees through supported configuration and
interfaces before execution; the orchestrator then manages execution in its
native way. OpenASER does not prescribe identical directory layouts, worker
roles, or scheduling algorithms across implementations.

An orchestrator is compatible only if it can satisfy the agreed contract.
Workflows and enforcement must align with that contract. If compatibility
requires extensive patches, fragile interception, or a competing controller,
reconsider the integration rather than weaken guarantees or add machinery.

### Managed Harness Control

Use ACP (Agent Client Protocol) for the initial Gas City-to-OpenCode managed
control integration. ACP provides structured session communication instead of
relying on terminal keystrokes and output scraping. It is a programmatic control
interface, not the native TUI launch path. OpenASER's capability ports remain
independent of this protocol; adapters implement the connection.

```text
OpenASER: engineering decisions and acceptance
    |
Gas City: work and harness-session coordination
    |
ACP connection through integration adapters
    |
OpenShell sandbox: OpenCode, child processes, working files
```

Adapters must preserve the sandbox boundary when launching and controlling the
harness, including workspace paths and process lifecycle. Verify the complete
connection against the selected tool versions and required guarantees before
using it; protocol support alone is not proof of end-to-end compatibility.
Harness completion is reported output, not OpenASER acceptance.

## Implementation Plan

### 1. Clarify Concepts and Solutions

Review one concept at a time before selecting an implementation milestone:
why it is needed, what OpenASER and the tool each own, what crosses the boundary,
how failure is handled, whether the selected solution satisfies the requirement,
and what is excluded. Discuss questions and proposals with the user; record only
agreed conclusions here. Each concept must be explainable with a simple sketch.

### 2. Agree on the First Milestone

Agree on the behavior, required capabilities, acceptance checks, and exclusions.
Record the smallest agreed scope here. Do not scaffold the entire architecture first.
A milestone may prove one narrow integration without defining an end-to-end workflow.

### 3. Implement Only the Approved Scope

Confirm the relevant tools' actual interfaces. Add only the domain types,
capability contracts, adapters, and configuration required by that milestone.
Do not add retries, routing, parallelism, recovery, or CLI commands unless approved.

### 4. Verify and Choose Together

Run the milestone's acceptance checks, including failure cases. Ensure a false
completion claim cannot bypass verification and provider details do not leak into
domain semantics. Report what works and what remains; ask the user what to do next.
Update this README before starting the next approved milestone.

**Current milestone:** none approved. Current work: concept clarification.
