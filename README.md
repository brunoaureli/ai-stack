# ai-stack

Reusable guidance for building full-stack AI agent applications with:

- [Pydantic AI](https://ai.pydantic.dev/) for typed Python agents
- [AG-UI](https://docs.ag-ui.com/) for agent-to-user event streaming
- [PageIndex](https://github.com/VectifyAI/PageIndex) for vectorless, reasoning-based retrieval over long, structured documents
- [MCP](https://modelcontextprotocol.io/) for tools and context
- [A2A](https://a2a-protocol.org/) for cross-agent interoperability
- [CopilotKit](https://docs.copilotkit.ai/) or custom AG-UI clients for frontend experiences

## Install the skill

The canonical skill is at `.agents/skills/ai-stack/SKILL.md`.

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

Replace `/path/to/ai-stack` with the local repository path. Symlinks keep every agent on one canonical version.

## Contents

- `.agents/skills/ai-stack/SKILL.md`: cross-agent skill instructions
- `agui-docs.md`: local AG-UI documentation snapshot used as a searchable reference

## Protocol boundaries

| Protocol | Responsibility |
| --- | --- |
| AG-UI | Agent ↔ user-facing application |
| MCP | Agent ↔ tools and context |
| A2A | Agent ↔ agent |

These protocols complement rather than replace one another. PageIndex is a retrieval component rather than an interaction protocol. Expose it as a typed Pydantic AI tool for application-local use, or through its MCP integration when the retrieval capability should be shared.

## Retrieval guidance

Use PageIndex when document hierarchy and traceable section or page references matter. Benchmark it against vector, keyword, or hybrid retrieval on representative documents and questions before choosing it for production. Include answer correctness, retrieval recall, citation quality, indexing cost, latency, and authorization boundaries in the evaluation.

## License

MIT
