# AI Stack: A Production Guide for Typed Agent Applications

Last researched: 2026-08-04

This repository is reusable, evidence-filtered guidance for building full-stack AI applications. It connects a typed Python agent runtime to a user interface, tools, long documents, other agents, durable state, and operations without treating those concerns as one protocol or product.

Claims and product choices are tied to the [source ledger](SOURCES.md). What has and has not been checked in this repository is recorded in [VALIDATION.md](VALIDATION.md). The installable instructions live in [the AI Stack skill](.agents/skills/ai-stack/SKILL.md).

## Executive summary

Choose this stack when an AI feature needs more than a one-off completion: it streams to a real product UI, invokes controlled tools, retrieves evidence, persists a recoverable record, and must be understandable in production.

The default is deliberately small:

```text
User interface -> AG-UI -> Pydantic AI agent -> approved tools
                       |             |
                       |             +-> Turso: durable application state
                       +-> OpenTelemetry: traces, metrics, and logs
```

Add PageIndex for long, structured documents; MCP when a tool or context source must be independently discoverable; A2A when an agent crosses a process or organizational boundary; and a durable workflow runner only when work must outlive a request. Do not introduce every component merely because it is in the guide.

This stack is not the best default for a simple form or a prototype whose main unknown is product demand. Start with a conventional endpoint and persistence first. Add the agent runtime when an agent can make a bounded, observable contribution to the user outcome.

## Philosophy and purpose

### Build a system of contracts, not a collection of AI SDKs

Each layer owns one concern and exposes a narrow contract. A typed agent must not become the authorization system, a chat protocol must not become a tool protocol, and a database must not silently become the retrieval strategy. This keeps replacement and failure local: change a model provider without changing the UI event format; add a shared tool without exposing database credentials to the model.

### Make the smallest vertical slice trustworthy first

Before adding agents that delegate to agents, build one authenticated run that streams text, performs one read-only tool call, persists enough state to be audited, and reaches a terminal event. The first goal is a boring traceable path, not maximum autonomy.

### Treat agent output as untrusted input with a useful interface

Models propose content and tool arguments; application code validates them, enforces user and tenant permissions, budgets execution, and controls side effects. A polished chat UI does not change this boundary.

### Measure the workload, not the marketing category

PageIndex, vector search, model routing, and durable workflows are workload-dependent choices. Compare them on representative tasks, evidence quality, latency, cost, and operational burden. Product documentation explains intended behavior; it is not independent proof of fitness for a particular application.

## The stack map

| Concern | Default | Purpose | Add or change it when |
| --- | --- | --- | --- |
| Agent runtime | [Pydantic AI](https://ai.pydantic.dev/) | Typed dependencies, outputs, and tool arguments in Python. | A project already has another established runtime or needs a missing capability. |
| Agent-to-user protocol | [AG-UI](https://docs.ag-ui.com/) over HTTP/SSE | Stream runs, messages, tools, state, and interrupts to a UI. | Use WebSockets only for a demonstrated bidirectional transport requirement. |
| UI client | [CopilotKit](https://docs.copilotkit.ai/) or @ag-ui/client | Consume AG-UI and render a product experience. | Choose the lower-level client when custom interaction or state handling demands it. |
| Local tools | Typed Pydantic AI tools | Keep application-local capability behind server-side validation. | Promote to MCP only when independent discovery or cross-client reuse is needed. |
| Shared tools/context | [MCP](https://modelcontextprotocol.io/) | Make a capability discoverable across compatible agents. | Do not use it merely to call code in the same process. |
| Document retrieval | [PageIndex](https://github.com/VectifyAI/PageIndex) | Retrieve hierarchical, page-level evidence from long structured documents. | Retain or choose vector/hybrid retrieval if evaluation says it performs better. |
| Cross-agent work | [A2A](https://a2a-protocol.org/) | Communicate with an independently deployed or owned agent. | Keep an in-process subtask in the local runtime. |
| Durable state | [Turso](https://docs.turso.tech/introduction) | Store conversations, runs, checkpoints, approvals, and audit records. | Use an existing compliant database when it is already the system of record. |
| Telemetry | [OpenTelemetry](https://opentelemetry.io/docs/) | Correlate traces, metrics, and logs across every boundary. | Add a backend such as Logfire when it improves collection, search, and evaluation workflow. |
| Evaluation | Pydantic Evals plus task-specific fixtures | Detect regressions in answers, tools, retrieval, and policy. | Introduce a specialized platform when dataset governance or workflow scale demands it. |
| Durable work | A queue or workflow runner | Retry, schedule, and resume work that outlives an HTTP request. | Use [Temporal](https://docs.temporal.io/) or the organization’s equivalent for long-running, stateful jobs. |
| Identity, policy, and files | Existing OIDC provider, server-side policy, object storage | Supply trusted identity and handle untrusted attachments outside the model. | Choose products that meet compliance, residency, and retention requirements. |

AG-UI, MCP, and A2A are complementary rather than interchangeable:

| Protocol | Connects | It is not |
| --- | --- | --- |
| AG-UI | agent runtime ↔ user-facing application | a way to discover third-party tools or agents |
| MCP | agent ↔ tools and context | a browser streaming protocol or a user authorization boundary |
| A2A | agent ↔ independently owned agent | a shortcut around tool permission checks |

## A concrete mental model: a claim-review assistant

Imagine a product that lets an analyst ask, “Which clauses in these filings make this renewal risky?”

1. The browser sends the message and attachments to the application. The server authenticates the analyst, applies document access rules, creates a thread and run ID, and stores request metadata.
2. The Pydantic AI agent receives typed dependencies: actor, tenant, authorization scope, budget, and a repository interface. It never receives a database token or a browser claim that it is authorized.
3. A PageIndex-backed tool finds sections and page ranges. It returns evidence identifiers and excerpts, not an unbounded document dump.
4. The agent forms a structured answer with citations. AG-UI streams safe lifecycle, message, tool, and state events to the browser.
5. OpenTelemetry records one trace linking the HTTP request, agent run, model call, retrieval, tool calls, and database writes. Turso retains the durable run record and any interrupt checkpoint.
6. If the agent proposes sending the result externally, the system interrupts. On approval it re-checks authorization, creates an idempotent side-effect record, and performs the action.

The same identifiers travel through every layer:

```text
authenticated request
  -> thread_id + run_id + trace_id
  -> AG-UI events and persisted run record
  -> tool calls, retrieval citations, interrupts, and audit entries
```

A trace ID is observability context, not an authorization credential. The server derives actor and tenant identity from its trusted session or token on every request and tool call.

## Integration rules that prevent expensive confusion

| Boundary | Rule | Why it matters |
| --- | --- | --- |
| AG-UI ↔ Turso | Keep UI transient state separate from the durable event/checkpoint record. | A reconnect must not create a second source of truth. |
| Pydantic AI ↔ MCP | Use native typed tools for local code; expose MCP only for shared discovery. | An unnecessary protocol boundary adds latency, authorization surface, and failure modes. |
| PageIndex ↔ Turso | Use PageIndex for structured evidence and Turso for application state. | Turso vector search is a separate RAG option, not an implicit PageIndex replacement. |
| A2A ↔ tools | Treat remote agents as untrusted services with explicit input/output contracts. | A remote agent must not inherit the caller’s broad tool authority. |
| telemetry ↔ all layers | Propagate trace context and record stable IDs at every hop. | A trace that stops at the model call cannot explain a failed user run. |
| Python ↔ TypeScript | Define an owned schema boundary and test it. | Manually synchronized types drift without a contract test or generation step. |

## The production envelope the original core needed

### Telemetry and operations

Instrument the HTTP boundary, AG-UI run lifecycle, model requests, tool calls, retrieval, database operations, interrupts, and background jobs with OpenTelemetry. Emit structured logs and metrics beside traces; a trace is not a substitute for aggregate error rate, latency, token/cost, queue depth, or retrieval-quality metrics.

Use the OpenTelemetry GenAI semantic conventions where they fit, but keep custom attributes low-cardinality, privacy-reviewed, and consistent. Do not store prompts, attachments, raw model responses, secrets, or private reasoning by default. Pydantic Logfire is a practical optional backend for Pydantic AI, but instrument through standard-compatible boundaries where possible.

An alert should describe a user-impacting condition: elevated terminal errors, tool failure rate, exhausted budget, stalled runs, retrieval citation failures, or approval backlog—not simply a slow span.

### Evaluation and release safety

Production traces explain a failure after it happened; evaluations prevent known failures from returning. Maintain a versioned representative dataset with expected citations, tool choices, policy outcomes, and refusal behavior. Run it against every change to prompt, model, tool, retrieval configuration, or schema.

Score factual support, task completion, citation correctness, tool-argument validity, policy compliance, latency, and cost. Keep model-judged scores separate from deterministic assertions and human review. A favorable aggregate score is not permission to ship a change that regresses a high-risk case.

### Identity, authorization, and policy

Authenticate at the application boundary using the organization’s OpenID Connect-compatible identity system. Pass only an internal actor/tenant context to the agent. Re-authorize every retrieval, read, tool call, checkpoint resume, and side effect on the server. Make document ACL filtering happen before the model sees text.

Classify tools by risk. Read-only tools can be automatically invoked within quota; consequential tools need a constrained schema, idempotency key, authorization check, and often a human interrupt. Policy belongs in regular application code with test coverage, not in a prompt.

### Durable work, cancellation, and recovery

An HTTP/SSE connection is a delivery channel, not a durable job executor. Use a bounded in-process task only for short, cancellable work. Send indexing, large ingestion, scheduled refreshes, and long-running actions to a queue or durable workflow runner with a persisted state machine.

Persist the input version, selected model/tool configuration, user decision, and side-effect idempotency key needed to resume. Make cancellation explicit: stop model and tool work when possible, record the terminal state once, and make a later retry a new attempt linked to the original run.

### Files and document ingestion

Store binaries in object storage, not conversation rows. Enforce size/type limits, malware scanning where appropriate, content extraction limits, document ownership, retention, and deletion propagation into PageIndex and derived artifacts. A file name and MIME type are claims from an untrusted client, not facts.

## A practical build sequence

1. Define the user outcome, non-goals, actor/tenant model, allowed tools, and success metrics.
2. Build one typed Pydantic AI run with deterministic unit tests and a structured output model.
3. Add one authenticated AG-UI/SSE endpoint and verify event ordering, cancellation, and reconnect behavior.
4. Persist threads, runs, messages, tool calls, checkpoints, approvals, and audit records in Turso or the established system of record.
5. Add OpenTelemetry context propagation, safe structured logging, metrics, and a dashboard before adding autonomous side effects.
6. Add one read-only tool; then add retrieval, MCP, A2A, files, and background work only when their concrete requirements appear.
7. Add evaluation fixtures and release gates before changing models or prompts routinely.
8. Add interrupt/resume flows and production controls: quotas, timeouts, concurrency limits, fallback policy, retention, and incident playbooks.

## Guide for agents working in this stack

Start with the smallest relevant source, not a random package import:

1. Read the matching current source in [SOURCES.md](SOURCES.md). Prefer its llms.txt index; it is the most useful up-to-date entry point for an agent.
2. Inspect the project’s installed version and existing architecture before choosing imports or changing a protocol boundary.
3. State the run’s identity, state ownership, tool authorization, terminal behavior, and telemetry correlation plan before implementation.
4. Change one boundary at a time and add a focused validation case.
5. Record what was actually tested in [VALIDATION.md](VALIDATION.md); do not imply a documentation or source-reachability check proves runtime behavior.

Use the concise installable [skill](.agents/skills/ai-stack/SKILL.md) for implementation guardrails. Use this guide for design review, onboarding, and trade-off discussions.

## Browser control for agents (OpenCode Browse)

OpenCode Browse (`@opencode/browse`, announced 2026-11-07) is the default browser tool in this stack. It is an MCP server plus a Chromium extension that lets an agent drive the user's real browser, with their tabs, logins and extensions, in background tabs. It replaces `agent-browser` for these agents.

```sh
npm install -g @opencode/browse
browse install --agent opencode --agent claude --agent codex
browse pair   # only if the extension did not pair automatically
```

- Install the Chrome extension from the Chrome Web Store when prompted; Brave, Edge, Arc and other Chromium browsers also work.
- `browse install` writes the MCP entry to the OpenCode config, `~/.claude.json` and `~/.codex/config.toml`.
- Pi has no installer flag; add `{"mcpServers":{"browse":{"command":"browse","args":["mcp"]}}}` through an MCP adapter if one is installed.
- Each agent's global instructions say to prefer `browse` and fall back to `agent-browser` only when it fails.
- Treat it as a user authorization boundary: use a separate browser profile, set a site allowlist, and hand the tab back to the user for logins, 2FA and payments.

## Install the skill

The canonical skill is at .agents/skills/ai-stack/SKILL.md.

### Zed / shared Agents skill directory

```sh
mkdir -p ~/.agents/skills
ln -sfn /path/to/ai-stack/.agents/skills/ai-stack ~/.agents/skills/ai-stack
```

### Codex

```sh
mkdir -p ~/.codex/skills
ln -sfn /path/to/ai-stack/.agents/skills/ai-stack ~/.codex/skills/ai-stack
```

### OpenCode

```sh
mkdir -p ~/.config/opencode/skills
ln -sfn /path/to/ai-stack/.agents/skills/ai-stack ~/.config/opencode/skills/ai-stack
```

### Claude Code

```sh
mkdir -p ~/.claude/skills
ln -sfn /path/to/ai-stack/.agents/skills/ai-stack ~/.claude/skills/ai-stack
```

Replace /path/to/ai-stack with the local repository path. Symlinks keep every agent on one canonical version.

## Contents

- .agents/skills/ai-stack/SKILL.md: concise cross-agent implementation rules.
- README.md: this human-facing architecture and onboarding guide.
- SOURCES.md: current official documentation, source repositories, purpose, and limitations.
- VALIDATION.md: scope of checks, recommended acceptance tests, and known limits.
- agui-docs.md: local AG-UI documentation snapshot; search targeted sections rather than loading the entire file.

## License

MIT
