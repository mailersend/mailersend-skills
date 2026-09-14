Agent Skills for use with [MailerSend](https://www.mailersend.com/).

These skills follow the [Agent Skills specification](https://agentskills.io/specification) so they can be used by any skills-compatible agent, including Claude Code and Codex CLI.

## Installation

### Marketplace

```
/plugin marketplace add mailersend/mailersend-skills
/plugin install mailersend@mailersend-skills
```

### Manually

#### Claude Code

Add the contents of this repo to a `/.claude` folder in the root of your project. See more in the [official Claude Skills documentation](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).

#### Codex CLI

Copy the `skills/` directory into your Codex skills path (typically `~/.codex/skills`). See the [Agent Skills specification](https://agentskills.io/specification) for the standard skill format.

## Skills

| Skill | Description |
|-------|-------------|
| [mailersend-cli](skills/mailersend-cli) | Send emails and SMS, manage domains, view analytics, handle email verification, and more using the [MailerSend CLI](https://github.com/mailersend/mailersend-cli) |
| [mailersend-mcp](skills/mailersend-mcp) | Monitor delivery health, trace individual emails and recipients, report open and delivery rates, and send safely using the [MailerSend MCP server](https://github.com/mailersend/mailersend-mcp) |
