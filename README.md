# ai-stack

Reusable guidance for building full-stack AI agent applications with:

- [Pydantic AI](https://ai.pydantic.dev/) for typed Python agents
- [AG-UI](https://docs.ag-ui.com/) for agent-to-user event streaming
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

These protocols complement rather than replace one another.

## License

MIT
