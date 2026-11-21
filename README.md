# antigravity skill for Claude Code

A Claude Code skill and sub-agent that route a session's work to Google
Antigravity (`agy` CLI) models. Typing `/antigravity` turns the mode on: the
main loop plans, writes briefs in English and checks results; `antigravity`
sub-agents run the work through `agy`.

## Model routing (always `-high`)

| Work | Model |
|---|---|
| Default: complex work, structured logic, code, review | `gemini-4-neon-high` |
| Hard reasoning, cost-balanced work, comparison against Neon | `gemini-4-argon-high` |
| Easy, fast, high-volume work (needs a detailed brief) | `gemini-3.8-flash-high` |
| Blender model production | `claude-opus-5-5-high` |

## Files

- `skills/antigravity/SKILL.md`: the mode, model routing, brief rules, Blender permission, known traps.
- `agents/antigravity.md`: thin Haiku wrapper that forwards a brief to `agy -p` and relays the answer.

## Install

```bash
mkdir -p ~/.claude/skills/antigravity ~/.claude/agents
cp skills/antigravity/SKILL.md ~/.claude/skills/antigravity/
cp agents/antigravity.md ~/.claude/agents/
```

Requires the Antigravity CLI at `~/.gemini/bin/agy`, signed in. Blender work
also needs the `blender` MCP server registered in agy and Blender running with
the MCP add-on started.

## Use

```
/antigravity <task>
```

Tested with agy 1.5.14.
