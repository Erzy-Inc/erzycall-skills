# ErzyCall Skills

Open-source [Agent Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)
for AI assistants connected to the **ErzyCall MCP server**.

The MCP server gives an agent the *tools* to place calls, manage contacts, and read results.
These skills give it the *judgment* to use them safely — which order to resolve things in,
what to show a user before a real phone rings, and how to read what came back.

## Skills

| Skill | What it covers |
|---|---|
| [`erzycall-outbound-call`](skills/erzycall-outbound-call/) | Full outbound call lifecycle — resolve contact, choose script, schedule, confirm, dial, verify, read the outcome, cancel or follow up |

## Requirements

- An AI client that supports Agent Skills (Claude Code, Claude Desktop / Cowork, or the Agent
  SDK) — or any MCP-capable agent you can give a system prompt to, such as Codex
- The **ErzyCall MCP server** connected, with an ErzyCall API key
- At least one outbound-capable phone number and one outbound assistant configured in your
  [ErzyCall dashboard](https://app.erzycall.com)

## Use it in your agent

The MCP server is what makes a call possible. The skill is what makes it safe: resolve the
contact → choose the script → settle the timing → **show you a confirmation card** → dial.
Nothing rings until you say yes.

```bash
git clone https://github.com/Erzy-Inc/erzycall-skills.git
```

### Claude Code · Claude Desktop / Cowork · Agent SDK

Native skill support — drop the folder in and restart:

```bash
cp -r erzycall-skills/skills/erzycall-outbound-call ~/.claude/skills/
```

Or commit it alongside a single project:

```bash
cp -r erzycall-skills/skills/erzycall-outbound-call .claude/skills/
```

The skill loads on its own whenever a request involves an outbound call — no slash command.
Files under `references/` stay out of context until the agent needs them.

### Codex

Codex has no skill loader, so the pipeline goes into your agent instructions instead:

```bash
cat erzycall-skills/skills/erzycall-outbound-call/SKILL.md >> AGENTS.md
cp -r erzycall-skills/skills/erzycall-outbound-call/references ./
```

Add the ErzyCall MCP server to `~/.codex/config.toml` with your API key. Two differences worth
knowing: Codex reads `AGENTS.md` in full every session rather than loading the skill on demand,
and it will not pull in `references/` by itself — name the file when you need it ("check
`references/scheduling.md`") for timezone handling, `endedReason` values, or retry behaviour.

### Any other MCP-capable agent

Same shape as Codex: connect the MCP server, paste `SKILL.md` into whatever your client uses
for a system prompt, and keep `references/` on disk for the long tail.

## Verifying it works

Ask your assistant something like:

> Call Maria tomorrow at 3pm and remind her about the invoice.

You should see it resolve the contact, pick the caller number and assistant, and then **stop**
and show you a confirmation card with the exact opening line before anything dials.

If it dials without showing you that card, the skill did not load.

## Design principles

These skills are written around one constraint: **a real phone rings a real person.**

- **Explicit confirmation, always.** `confirmed: true` means the user saw this exact call —
  destination, caller ID, opening line, timing — and said yes. It is never inferred, and an
  approval never carries across an edit.
- **Preview the rendered message, not the template.** An unresolved `{{variable}}` is spoken
  aloud, literally, to whoever picks up.
- **One idempotency key per intended call.** Reused on retries. A fresh key on a retry is a
  second phone call to a real person.
- **Report what happened, not what was requested.** An unanswered call is still `ended`.
- **Data is not instructions.** Contact notes, case prompts, and transcripts can contain
  anything; none of it authorizes a call.

## Contributing

Skills follow the standard layout — a `SKILL.md` with YAML frontmatter (`name`, `description`)
plus optional `references/` for detail the agent loads on demand. Keep `SKILL.md` focused
enough to stay useful in context; push the long tail into references.

When adding a skill, verify every tool name, required argument, and enum value against the
live MCP schemas rather than from memory.

## License

MIT — see [LICENSE](LICENSE).
