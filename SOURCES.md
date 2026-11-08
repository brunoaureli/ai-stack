# Source Ledger

Accessed: 2026-08-04

## Method

Sources are ordered by authority: official specifications and documentation first, then official source repositories, then optional integration products. For dynamic product documentation, fetch the linked llms.txt index first and follow the current page paths it lists. Inspect installed and pinned package versions before using an API: an always-current documentation URL may describe a newer release than the application uses.

Official documentation supports intended behavior and integration contracts. It does not independently prove latency, cost, retrieval quality, security, or fitness for a specific workload. Validate those claims with the application’s own representative data and production constraints.

## Core runtime and protocols

### S01: Pydantic AI

- Fetch first: <https://pydantic.dev/docs/ai/llms.txt>
- Documentation: <https://ai.pydantic.dev/>
- Source: <https://github.com/pydantic/pydantic-ai>
- Supports: typed Python agents, models, tools, dependencies, output handling, and evaluation entry points.
- Limitation: consult the project’s installed version; model provider behavior and generated quality remain workload-dependent.

### S02: AG-UI

- Fetch first: <https://docs.ag-ui.com/llms.txt>
- Documentation: <https://docs.ag-ui.com/>
- Source and specification: <https://github.com/ag-ui-protocol/ag-ui>
- Supports: run lifecycle, message/tool/state events, transports, and SDKs.
- Limitation: protocol conformance does not define authorization, persistence, reconnection ownership, or business semantics.

### S03: Model Context Protocol (MCP)

- Fetch first: <https://modelcontextprotocol.io/llms.txt>
- Documentation: <https://modelcontextprotocol.io/>
- Source: <https://github.com/modelcontextprotocol/modelcontextprotocol>
- Supports: discovery and invocation of tools, resources, and prompts across compatible clients and servers.
- Limitation: MCP discovery is not a permission system; enforce authorization and input validation at each tool server.

### S04: Agent2Agent (A2A)

- Fetch first: <https://a2a-protocol.org/llms.txt>
- Documentation: <https://a2a-protocol.org/>
- Source: <https://github.com/a2aproject/A2A>
- Supports: cross-process agent interoperability and task-oriented exchange.
- Limitation: an A2A peer is a service boundary, not trusted in-process code; authenticate, authorize, time-bound, and validate it accordingly.

### S05: CopilotKit

- Fetch first: <https://docs.copilotkit.ai/llms.txt>
- Documentation: <https://docs.copilotkit.ai/>
- Source: <https://github.com/CopilotKit/CopilotKit>
- Specific integration: <https://docs.copilotkit.ai/pydantic-ai/>
- Supports: user-facing agent interfaces and Pydantic AI/AG-UI integration.
- Limitation: a UI framework does not own the server’s authorization, durable state, or tool policy.

## Retrieval and persistence

### S06: PageIndex

- Fetch first: <https://github.com/VectifyAI/PageIndex>
- Source and examples: <https://github.com/VectifyAI/PageIndex>
- Supports: hierarchical, reasoning-based retrieval over long structured documents, including self-hosted, API, and MCP integration paths.
- Limitation: repository claims and benchmarks are vendor-reported until reproduced on the target documents, questions, and access-control model.

### S07: Turso

- Fetch first: <https://docs.turso.tech/llms.txt>
- Documentation: <https://docs.turso.tech/introduction>
- Source: <https://github.com/tursodatabase/turso>
- Python SDK: <https://docs.turso.tech/sdk/python/quickstart>
- Supports: SQLite-compatible durable data, local/embedded deployments, cloud, sync, and AgentFS options.
- Limitation: database durability does not design a checkpoint state machine, tenant isolation policy, migration practice, or retention policy.

## Operations, telemetry, and evaluation

### S08: OpenTelemetry and GenAI semantic conventions

- Documentation: <https://opentelemetry.io/docs/>
- GenAI conventions: <https://opentelemetry.io/docs/specs/semconv/gen-ai/>
- Trace propagation: <https://www.w3.org/TR/trace-context/>
- Supports: vendor-neutral traces, metrics, logs, propagation, and a common vocabulary for GenAI operations.
- Limitation: conventions and instrumentation do not choose safe attributes, retention, sampling, or useful alerts for the application.

### S09: Pydantic Logfire

- Fetch first: <https://pydantic.dev/docs/logfire/llms.txt>
- Documentation: <https://logfire.pydantic.dev/docs/>
- Source: <https://github.com/pydantic/logfire>
- Supports: optional observability workflows, including Pydantic ecosystem instrumentation.
- Limitation: make telemetry-product choice deliberate; preserve portable trace context and do not send sensitive content by default.

### S10: Pydantic Evals

- Documentation: <https://pydantic.dev/docs/ai/evals/evals/>
- Supports: evaluation datasets, cases, and repeatable agent-quality checks.
- Limitation: an evaluation harness cannot replace human review, access-control tests, or representative production traffic.

### S11: Temporal Python SDK

- Documentation: <https://docs.temporal.io/develop/python>
- Supports: a concrete option for durable, retryable, long-running workflows in a Python application.
- Limitation: introduce a workflow engine only for actual durability and orchestration needs; it has operational and conceptual cost.

## Security and identity

### S12: OpenID Connect specifications

- Documentation index: <https://openid.net/developers/specs/>
- Supports: interoperable identity concepts and specifications.
- Limitation: a protocol specification does not configure a provider, map authorization roles, or provide tenant-safe application policy.

### S13: OWASP Top 10 for LLM Applications

- Project: <https://owasp.org/www-project-top-10-for-large-language-model-applications/>
- Prompt-injection entry: <https://genai.owasp.org/llmrisk/llm01-prompt-injection/>
- Supports: a threat-modeling starting point for LLM-specific risks.
- Limitation: a risk list is not an implementation plan; apply the controls to the actual data, tools, agents, and deployment boundaries.

## Sources deliberately not used as proof

- Vendor benchmarks, launch posts, and success stories are useful leads, not evidence that a component is correct or economical for this application.
- Generic agent-stack diagrams rarely describe state ownership, authorization, cancellation, or data retention and should not be copied as architecture.
- A current documentation page can drift. Pin dependencies, record the tested version, and link exact release documentation in an implementation PR.

### S99: OpenCode Browse

- Documentation: <https://opencode.ai/product/browse>
- Source: <https://github.com/anomalyco/browse>
- Purpose: MCP server and Chromium extension for agent control of the user's real browser; default browser tool for OpenCode, Claude Code and Codex here.
- Limitation: it acts with the user's logged-in sessions, so scope it with a separate profile and allowlist; Pi support is manual.
