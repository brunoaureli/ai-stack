---
name: ai-stack
description: Design, build, review, or debug full-stack AI agent applications using Pydantic AI for typed Python agents, AG-UI for agent-to-frontend event streaming, MCP for tools and context, A2A for agent interoperability, and CopilotKit or custom clients for user interfaces. Use when choosing an agent architecture, scaffolding an AI app, exposing a Pydantic AI agent to a frontend, implementing streaming events, tools, shared state, approvals, or generative UI.
---

# AI Stack

Build interoperable agent applications with clear protocol boundaries and typed contracts.

## Default architecture

Use this stack unless the project already establishes alternatives:

- **Agent runtime:** Pydantic AI on Python.
- **Agent-to-user protocol:** AG-UI.
- **Frontend client:** CopilotKit for a production-ready UI, or `@ag-ui/client` for a custom client.
- **Tools and context:** native Pydantic AI tools; use MCP when tools must be independently discoverable or shared.
- **Agent-to-agent communication:** A2A only when agents cross process, team, or ownership boundaries.
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
- Bound attachment size, run duration, tool iterations, concurrency, and output size.
- Log IDs, timings, event types, tool outcomes, and errors without logging secrets or sensitive reasoning.
- Make side-effecting tools idempotent where possible and use confirmation gates where not.

## Testing

Cover at least:

- Typed agent behavior and tool validation.
- Exact event ordering for text and tool streams.
- State snapshot/delta application.
- Interrupt, approval, rejection, edit, and resume paths.
- Cancellation, timeout, provider failure, malformed tool output, and client disconnect.
- An end-to-end run from frontend request to terminal AG-UI event.

Use the AG-UI Dojo as an interoperability checklist when implementing or debugging an adapter.

## Documentation references

Prefer current official documentation:

- AG-UI: https://docs.ag-ui.com/
- AG-UI specification and SDKs: https://github.com/ag-ui-protocol/ag-ui
- Pydantic AI: https://ai.pydantic.dev/
- Pydantic AI + CopilotKit/AG-UI: https://docs.copilotkit.ai/pydantic-ai/
- AG-UI Pydantic AI demo: https://dojo.ag-ui.com/pydantic-ai/feature/shared_state
- MCP: https://modelcontextprotocol.io/
- A2A: https://a2a-protocol.org/

The repository root also contains `agui-docs.md`, a local AG-UI documentation snapshot. Search targeted sections rather than loading the whole file, and verify version-sensitive details against current official docs.

## Expected response shape

When advising or implementing:

1. State the selected architecture and why it fits.
2. Separate protocol responsibilities explicitly.
3. Show the smallest end-to-end vertical slice first.
4. Call out security, state ownership, and failure behavior.
5. Report exact files changed and validation run.
