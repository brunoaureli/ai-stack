---
name: ai-stack
description: Design, build, review, or debug full-stack AI agent applications using Pydantic AI for typed Python agents, AG-UI for agent-to-frontend event streaming, PageIndex for reasoning-based document retrieval, MCP for tools and context, A2A for agent interoperability, CopilotKit or custom clients for user interfaces, Turso for durable application and agent state, and OpenTelemetry-compatible telemetry and evaluation. Use when choosing an agent architecture, scaffolding an AI app, exposing a Pydantic AI agent to a frontend, implementing RAG, streaming events, tools, persistence, observability, evaluations, shared state, approvals, or generative UI.
---

# AI Stack

Build interoperable agent applications with clear protocol boundaries and typed contracts.

## Default architecture

Use this stack unless the project already establishes alternatives:

- **Agent runtime:** Pydantic AI on Python.
- **Agent-to-user protocol:** AG-UI.
- **Frontend client:** CopilotKit for a production-ready UI, or `@ag-ui/client` for a custom client.
- **Tools and context:** native Pydantic AI tools; use MCP when tools must be independently discoverable or shared.
- **Document retrieval:** PageIndex when long, structured documents benefit from hierarchical, reasoning-based retrieval. Keep conventional vector or hybrid retrieval when measurements show it fits better.
- **Agent-to-agent communication:** A2A only when agents cross process, team, or ownership boundaries.
- **Persistence:** Turso for SQLite-compatible application data, conversation history, checkpoints, approvals, and audit records. Select Turso Cloud, embedded Turso Database, or AgentFS according to deployment and state-access needs.
- **Telemetry and evaluation:** OpenTelemetry-compatible traces, metrics, and structured logs with safe Pydantic AI instrumentation; maintain a versioned evaluation set for answers, citations, tools, and policy outcomes.
- **Transport:** HTTP with Server-Sent Events for the first implementation. Use WebSockets only when SSE plus normal HTTP requests cannot satisfy bidirectional requirements.
- **Schemas:** Pydantic models in Python and generated or explicitly synchronized TypeScript types at the frontend boundary.

Do not conflate the protocols:

- AG-UI connects agents to user-facing applications.
- MCP connects agents to tools and context.
- A2A connects agents to other agents.
- Generative UI specifications describe renderable UI; they can travel over AG-UI but do not replace it.

## Before implementation

1. Inspect dependency files, existing framework conventions, application boundaries, and tests.
2. Identify which capabilities are actually required: streaming text, tools, shared state, multimodal input, interrupts, reasoning, or generative UI.
3. Prefer an existing AG-UI integration for the selected framework over a custom protocol adapter.
4. Confirm current package APIs from installed versions or official documentation; do not invent imports or pin stale versions from memory.
5. Define ownership of conversation history and shared state. Avoid keeping competing sources of truth in the browser and agent runtime.
6. If retrieval is required, evaluate representative documents and questions before selecting PageIndex, vector search, keyword search, or a hybrid.
7. If persistence is required, decide whether the application needs managed remote data, embedded or offline data, synchronization, or filesystem-shaped agent state before selecting a Turso product and client.
8. Define trace propagation, stable correlation IDs, safe telemetry attributes, alert conditions, and evaluation gates before adding consequential autonomy.

## Document retrieval with PageIndex

Use [PageIndex](https://github.com/VectifyAI/PageIndex) for document-heavy workloads where structure, section context, and traceable page-level retrieval matter, such as financial reports, legal documents, regulatory filings, technical manuals, and academic material.

PageIndex builds a hierarchical tree index and performs reasoning-based tree search without requiring document chunks or a vector database. It can run from the open-source repository or through its API or MCP service.

Integration options:

- Wrap self-hosted PageIndex retrieval in a typed Pydantic AI tool when it belongs to one application.
- Use the PageIndex MCP server when multiple agents or clients should discover and reuse the retrieval capability.
- Use the hosted API when its OCR and managed indexing fit the project's data, privacy, latency, and cost constraints.
- Return source document IDs, section paths, and page ranges from retrieval tools so answers can cite evidence.

Do not assume vectorless retrieval is better for every corpus. Compare answer correctness, retrieval recall, citation quality, indexing cost, query latency, and operational complexity against a relevant baseline. Treat repository benchmark claims as vendor-reported until reproduced on the project's data.

## Persistence with Turso

Use [Turso](https://docs.turso.tech/introduction) as the default persistence option when an AI application benefits from a SQLite-compatible relational model:

- Use Turso Cloud or an official libSQL-compatible client for managed application data, remote conversation history, run metadata, checkpoints, approvals, and audit records.
- Use embedded Turso Database for on-device, browser, local-first, or offline data. Add Turso Sync only when local reads and writes must synchronize with the cloud.
- Evaluate AgentFS when an agent specifically needs filesystem-shaped state and built-in auditability; do not treat it as an automatic replacement for relational application tables.
- Store stable thread, run, message, tool-call, interrupt, and user IDs so persisted records can be correlated with AG-UI events.
- Make checkpoint and side-effect records transactional and resumption idempotent. Apply per-user or per-tenant authorization before every read and write.
- Set retention rules for message content, tool payloads, attachments, and audit data. Never persist credentials or private chain-of-thought.

Turso persistence and PageIndex retrieval solve different problems. Store application and agent state in Turso; use PageIndex for hierarchical retrieval over long structured documents. If Turso vector search is considered for RAG, benchmark it as a separate retrieval design rather than silently replacing PageIndex.

## Implementation sequence

1. Define typed Pydantic input, output, dependency, tool-argument, and state models.
2. Implement and test the Pydantic AI agent independently of the UI transport.
3. Add the supported Pydantic AI ↔ AG-UI adapter and expose one streaming endpoint.
4. Map a run to AG-UI lifecycle events:
   - `RUN_STARTED`
   - message or tool-call events in valid start/content/end order
   - state snapshot or JSON Patch delta events when needed
   - exactly one terminal `RUN_FINISHED` or `RUN_ERROR`
5. Connect the frontend using CopilotKit or an AG-UI client and render events incrementally.
6. Add tools, approvals, shared state, multimodal input, and generative UI one capability at a time.
7. Add authentication, authorization, cancellation, timeouts, tracing, and persistence before production rollout.
8. Send ingestion, indexing, and work that must outlive an HTTP request to a bounded queue or durable workflow; an SSE connection is not a job executor.

## Event-stream rules

- Preserve `thread_id`, `run_id`, `message_id`, and `tool_call_id` consistently.
- Never emit content before its corresponding start event or after its end event.
- Stream tool arguments as valid incremental JSON and associate results with the correct call ID.
- Send complete state with snapshots and RFC 6902 JSON Patch operations with deltas.
- Treat disconnects and cancellation as normal lifecycle paths; release model and tool resources promptly.
- Keep provider-native payloads behind adapters. Expose vendor-neutral AG-UI messages to clients.
- Do not expose private chain-of-thought. Only emit safe summaries or provider-supported encrypted reasoning continuity data.

## Human-in-the-loop

Use interrupts for consequential or underspecified actions such as payments, deletion, sending messages, permission changes, or missing business input.

- Include a stable interrupt ID, clear reason, user-facing message, and constrained response schema.
- Persist enough checkpoint state to resume safely.
- Make resumption idempotent and auditable.
- Revalidate edited tool arguments and authorization after approval.

## Security and reliability

- Keep model and service credentials server-side.
- Validate all user messages, attachments, tool arguments, state patches, interrupt payloads, and generated UI schemas.
- Apply tool allowlists and per-user authorization on the server; frontend filtering is not a security boundary.
- Bound attachment size, document size, index depth, run duration, tool iterations, concurrency, and output size.
- Apply document-level authorization before retrieval and before returning source text; never rely on the model to enforce access controls.
- Log IDs, timings, event types, tool outcomes, and errors without logging secrets or sensitive reasoning.
- Propagate `trace_id` alongside stable thread, run, message, tool-call, interrupt, actor, and tenant IDs. Treat trace context as observability metadata, never as authorization.
- Enforce input and output budgets, per-user/tenant quotas, concurrency limits, timeouts, and cancellation for models, tools, retrieval, files, and background work.
- Make side-effecting tools idempotent where possible and use confirmation gates where not.

## Testing

Cover at least:

- Typed agent behavior and tool validation.
- Retrieval quality on a representative question set, including citations and access-control boundaries.
- Telemetry correlation across the HTTP request, AG-UI run, model, tools, retrieval, persistence, and background work, with sensitive content redacted or excluded.
- Versioned evaluations for expected answers, citations, tool choice, policy/refusal outcomes, latency, and cost.
- Exact event ordering for text and tool streams.
- State snapshot/delta application.
- Interrupt, approval, rejection, edit, and resume paths.
- Cancellation, timeout, provider failure, malformed tool output, and client disconnect.
- An end-to-end run from frontend request to terminal AG-UI event.

Use the AG-UI Dojo as an interoperability checklist when implementing or debugging an adapter.

## Documentation references

Prefer current official documentation and inspect the upstream repository when implementation details are not covered by the docs:

| Component | Documentation | Source repository |
| --- | --- | --- |
| Pydantic AI | https://ai.pydantic.dev/ | https://github.com/pydantic/pydantic-ai |
| AG-UI | https://docs.ag-ui.com/ | https://github.com/ag-ui-protocol/ag-ui |
| PageIndex | Repository README and examples | https://github.com/VectifyAI/PageIndex |
| MCP | https://modelcontextprotocol.io/ | https://github.com/modelcontextprotocol/modelcontextprotocol |
| A2A | https://a2a-protocol.org/ | https://github.com/a2aproject/A2A |
| CopilotKit | https://docs.copilotkit.ai/ | https://github.com/CopilotKit/CopilotKit |
| Turso | https://docs.turso.tech/introduction | https://github.com/tursodatabase/turso |
| OpenTelemetry | https://opentelemetry.io/docs/ | https://github.com/open-telemetry/opentelemetry-specification |

Integration-specific references:

- Pydantic AI + CopilotKit/AG-UI: https://docs.copilotkit.ai/pydantic-ai/
- AG-UI Pydantic AI demo: https://dojo.ag-ui.com/pydantic-ai/feature/shared_state
- Turso documentation index: https://docs.turso.tech/llms.txt
- Turso Python SDK quickstart: https://docs.turso.tech/sdk/python/quickstart
- AgentFS Python SDK: https://docs.turso.tech/agentfs/sdk/python
- Turso Cloud MCP integration: https://docs.turso.tech/integrations/mcp
- OpenTelemetry GenAI semantic conventions: https://opentelemetry.io/docs/specs/semconv/gen-ai/
- Pydantic Evals: https://pydantic.dev/docs/ai/evals/evals/
- Pydantic Logfire documentation index: https://pydantic.dev/docs/logfire/llms.txt

The repository root also contains `agui-docs.md`, a local AG-UI documentation snapshot. Search targeted sections rather than loading the whole file, and verify version-sensitive details against current official docs.

For a human-facing architecture guide, a fetch-first documentation ledger, and the exact scope of repository checks, read the repository-root `README.md`, `SOURCES.md`, and `VALIDATION.md`.

## Expected response shape

When advising or implementing:

1. State the selected architecture and why it fits.
2. Separate protocol responsibilities explicitly.
3. Show the smallest end-to-end vertical slice first.
4. Call out security, state ownership, and failure behavior.
5. Report exact files changed and validation run.
