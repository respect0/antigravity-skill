---
name: antigravity
description: "Antigravity mode: routes this session's work to Google Antigravity (agy CLI) sub-agents. The main loop plans, writes briefs in English and checks results; `antigravity` sub-agents do the work. Model routing (always high): default gemini-4-neon-high, hard reasoning/comparison gemini-4-argon-high, easy and fast work gemini-3.8-flash-high, Blender model production claude-opus-5-5-high. Load when the user types /antigravity or says 'Antigravity ile yap', 'Gemini'ye yaptır', 'agy alt ajanları kullan', 'use Antigravity', 'have Gemini do it'."
argument-hint: "[task]"
---

# Antigravity mode

Once this command is invoked, **Antigravity sub-agents do the work** for the
rest of the session. You (the main loop) split the work into pieces, pick a
model for each, write the brief, check the result and report to the user. The
mode stays on until the user says to stop using Antigravity or to do the work
yourself.

If an argument was given, start that task now under these rules. With no
argument, say in one line that the mode is on and apply it from the next
request.

What you keep for yourself: the plan, the briefs, checking results, small
things a few tool calls settle (reading one file, a one-line fix), and steps
that are hard to undo or face outward (commit, push, delete, publish).

## Language

**Every brief sent to a model is written in English**, whatever language the
user writes in. Translate the user's request into the brief; keep file paths,
identifiers and any text that must appear verbatim (UI strings, file names,
quoted content) exactly as they are. If the deliverable itself must be in
another language (Turkish UI copy, a Turkish document), say so in the brief:
"Write the output in Turkish." Your own replies to the user stay in the user's
language.

## Model routing

Always the `-high` variant. Another effort level (`-low`, `-medium`, `-xhigh`)
only when the user asks for it.

| Work | Model id |
|---|---|
| **Default.** Complex work, structured business logic, writing code, review, analysis | `gemini-4-neon-high` |
| Hard reasoning, cost-balanced work, comparing Neon's answer against a second model | `gemini-4-argon-high` |
| Easy work: summaries, translation, quick lookups, format conversion, high-volume repetitive tasks | `gemini-3.8-flash-high` |
| Producing, editing or exporting models in Blender | `claude-opus-5-5-high` |

- When unsure, Neon. Send work to Flash only when you are sure it is easy.
- Flash-Lite does not exist in agy (it is not in `agy models`); work of that
  class runs on `gemini-3.8-flash-high`.
- `gemini-4-hydrogen-*` exists in agy but is not part of this mode; use it only
  when the user names it.
- Current list: `~/.gemini/bin/agy models`. If a model named here is gone from
  the list, tell the user; do not switch models silently.

## Calling the sub-agent

Agent tool, `subagent_type: "antigravity"`, `model: "haiku"`. The wrapper only
relays; the model inside agy does the work. Start the prompt with this header,
then the brief:

```
MODEL: gemini-4-neon-high
DIR: /absolute/project/path
ACCESS: read-only | edit | blender
---
<brief, in English>
```

- `read-only` → `--mode plan` (research, analysis, review).
- `edit` → `--mode accept-edits` (writes files). Prefer a folder whose changes
  are committed, so the diff can be checked.
- `blender` → opens the Blender MCP permission (see below).

Access must match the brief: if the brief asks for files to be written, give
`edit`; under `read-only` a write request comes back silently undone.

Launch independent pieces in parallel, as several Agent calls in one message.
Two agents with `edit` access never run in the same folder at once (read-only
agents are free); run edits one after another or in separate folders. Put a
long brief in a file in the scratchpad and give its absolute path in the
prompt.

## Writing the brief (per model)

The brief is all the model sees: it does not know this conversation. Every
brief carries the goal, absolute paths, what done looks like and the output
format wanted.

**Flash 3.8: detailed and explicit.** Flash does not go beyond what it is
given: it skips a step you did not write, does not open a file you did not
name, and does not fill gaps on its own. So:

- Write the steps numbered and in order; state each step's input and output.
- Name every file and folder to look at, one by one, by absolute path.
- Give the exact output format (headings, fields, language, length); add one
  example input/output pair when it helps.
- Write the edge cases: "if the file is missing, write X", "if unsure, say Y".
- Ask for side work explicitly (run the tests, update the related places too);
  it will not do what you did not ask for.

**Gemini 4 (Neon, Argon): goal-directed.** These models fill gaps themselves
and need no step-by-step recipe. Give the goal and why it matters, the
context, the constraints and what done looks like; leave the route to the
model. State separately what it must not touch and which decisions the user
has already made. For a second opinion, give the problem, the evidence and the
options; leave out your own conclusion.

**Opus 5.5 (Blender): goal plus quality bar.** What the model is, references
(absolute paths), scale (for Roblox, against the 5-stud character), polygon
and part budget, export format and target folder. No step-by-step modelling
recipe.

## Blender work

agy has a `blender` MCP server registered, but in print mode MCP tools are
denied unless a rule allows them (`denied_actions: mcp`). The `ACCESS: blender`
line makes the wrapper add this setting for that call only:

```
--settings '{"permissions":{"allow":["mcp(blender/*)"]}}'
```

The format is `mcp(<server>/<tool>)`; `mcp(blender)` does not work. Do not
write it to the permanent settings file and do not grant `mcp(*)`.

- Blender must be open with "Start MCP Server" pressed in the add-on
  (localhost:9876). If it cannot connect, tell the user; when the add-on is
  outdated the error text suggests `uvx mcp-for-blender install-addon`.
- One agent per Blender at a time. Run Blender work in sequence.
- Model production is slow: give one stage per call (blockout, detail,
  materials, export) and check each stage with a render or screenshot.

## Checking the result

A sub-agent's report is a claim. Before telling the user it is done:

- For edits, look at the diff or the changed files.
- For research, verify from the source the few claims the decision rests on.
- For Blender, look at a render or screenshot.
- If the result is incomplete, continue the same conversation (the agent
  returns a `conversation_id`; add a `CONTINUE: <id>` line to the header) or
  fix the brief and send it again. When Flash left something out, first check
  whether the brief asked for that step explicitly.
- If two attempts fail, move up a model (Flash → Neon → Argon) and tell the
  user.

## Known traps

- `agy` is not on PATH: `~/.gemini/bin/agy`.
- Never use `--dangerously-skip-permissions`.
- `--mode auto` is not available on this account; it warns and falls back to
  the default mode.
- When several agy processes start in the same second, one can fail with
  `backend returned an empty conversation id` (in testing it was always the
  Opus call). It is transient; the wrapper retries once. Do not start an
  Opus/Blender call at the same moment as the others.
- If every call times out with 0 tokens, an MCP server is stuck connecting:
  search the newest log in `~/.gemini/antigravity-cli/log/` for
  "still connecting". `agy mcp remove <name>` fixes it, `disable` does not
  (ask the user before removing).
- A simple call takes about 10 s; give heavy work up to 10 minutes.
