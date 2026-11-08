# Validation Report

Run: 2026-08-04

## Result

This repository now has a human-facing architecture guide, a source ledger, and concise installable implementation rules. The guide adopts the Rust reference project’s separation of executive guidance, source evidence, and validation scope.

It is documentation, not a runnable application. No claim below should be read as a benchmark or an end-to-end verification of Pydantic AI, AG-UI, PageIndex, MCP, A2A, CopilotKit, Turso, or a telemetry backend in one deployment.

## Checks performed

| Check | Result |
| --- | --- |
| Skill YAML frontmatter | pass: name and description are valid |
| Markdown patch whitespace | pass: git diff --check clean |
| Core current-documentation entry points | pass: Pydantic AI, AG-UI, PageIndex, MCP, A2A, CopilotKit, and Turso returned HTTP 200 with redirects enabled |
| Operations-documentation entry points | pass: OpenTelemetry, GenAI semantic conventions, Pydantic Logfire, Pydantic Evals, Trace Context, OpenID Connect, OWASP prompt-injection guidance, and Temporal Python docs returned HTTP 200 with redirects enabled |
| Local AG-UI snapshot | present as agui-docs.md; it was not revalidated against every current AG-UI page |
| Fresh-task skill forward test | pass: independently selected AG-UI for the browser, MCP for shared search, no A2A in v1, durable approval/idempotency state, end-to-end correlation, and evaluation gates for a regulated contract-review workflow |

## Required acceptance suite for an application built with this guide

| Area | Minimum evidence |
| --- | --- |
| Typed runtime | Unit tests for dependencies, tool-argument validation, structured outputs, model failure, and bounded retry/fallback behavior. |
| AG-UI | Contract tests for RUN_STARTED, valid message/tool event ordering, snapshots/deltas, exactly one terminal event, cancellation, reconnect, and malformed client input. |
| State | Migration test; tenant-scoped read/write tests; idempotent checkpoint/resume and side-effect tests; retention/deletion test. |
| Retrieval | Representative question set scored for answer correctness, citation/page accuracy, authorization boundaries, latency, and cost against a relevant baseline. |
| MCP and A2A | Authentication, authorization, schema validation, timeout, retry, and remote-error tests at each service boundary. |
| Telemetry | One trace links request, run, model, tools, retrieval, persistence, and background work; sensitive content is absent from sampled telemetry; metrics and alerts exercise a failure path. |
| Evaluation | Versioned fixtures assert high-risk behavior, citations, tool selection, policy/refusal outcomes, latency, and cost before prompt/model/tool changes ship. |
| Security | Threat-model review covers prompt injection, tool misuse, data exfiltration, cross-tenant access, SSRF, attachments, secrets, quotas, and approval bypass. |

## Known limits

- Live URL checks are point-in-time reachability checks. They do not prove that linked content is stable, complete, compatible with an installed version, or correct for a workload.
- This repository does not pin package versions or provide a starter application, credentials, migrations, infrastructure, or a deployed telemetry collector.
- Turso persistence guidance is a recommended default, not a mandate to replace an organization’s established system of record or identity provider.
- PageIndex and Turso vector search have not been benchmarked against a shared corpus here; the guide intentionally requires a workload-specific comparison.
- No source controls an application's retention, residency, privacy, or access policy. Those are product and organizational decisions requiring explicit implementation and review.

## Reproduction

From the repository root:

```console
git diff --check
ruby -ryaml -e 'text = File.read(".agents/skills/ai-stack/SKILL.md"); front = text.match(/\A---\n(.*?)\n---\n/m) or abort "missing frontmatter"; data = YAML.safe_load(front[1]); abort "invalid fields" unless data.keys.sort == %w[description name] && data["name"] == "ai-stack"; puts "frontmatter valid"'
```

Fetch each source from [SOURCES.md](SOURCES.md) with redirects enabled before using a version-sensitive API. Then record installed versions and actual application test results in the implementation’s own validation report.
