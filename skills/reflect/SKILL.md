---
name: reflect
description: Spawn three parallel review subagents over the active transcript, surface learnings, and route each to a concrete edit on an existing skill. Use when the user says reflect.
disable-model-invocation: true
---

# Reflect

Mine the current conversation for durable learnings, then route them into skill edits.

## When to invoke

Invoke when the user says "reflect" or "/reflect". Skip when the conversation is trivial, off-topic, or already covered by an existing skill the parent followed correctly. One-offs are not learnings.

## Process

### 1. Locate the active transcript

The parent finds its own transcript file before fanning out. Your provider keeps it: Claude Code under `~/.claude/projects/<cwd-slug>/` (the working directory with `/` and `.` replaced by `-`), Codex under `~/.codex/sessions/`. Use your own working directory's path. Do not glob across `~/.claude/projects/*/`. That crosses workspace boundaries and reads private chats from unrelated projects.

```bash
ls -t ~/.claude/projects/<cwd-slug>/*.jsonl ~/.claude/projects/<cwd-slug>/*/subagents/*.jsonl ~/.codex/sessions/*/*/*/*.jsonl 2>/dev/null | head -10
```

Three transcript layouts: Claude Code session (`<id>.jsonl`), Claude Code subagent (`<parent>/subagents/<child>.jsonl`), and Codex (`YYYY/MM/DD/rollout-*.jsonl`, whose first line records the `cwd`).

For each candidate, check that it belongs to your working directory and contains the conversation's opening user prompt. Take the matching path. If no path resolves, write a tight digest of the session and pass that instead.

### 2. Spawn three reviewers in parallel

One message, three `create_agent` calls, with the profile set as below, in the profile's mode (not a read-only or plan mode). Reviewers need MCP access for context lookups (tickets, chat threads, observability traces referenced in the transcript). A read-only mode strips MCPs.

Each reviewer and the synthesizer name an override profile, a tier profile, and a default. Set the `create_agent` launch (`provider/model`, `thinkingOptionId`, `modeId`, features) from the pstack Agent profiles (read them with `list_profiles`): the role's override profile if it exists, else its tier profile, else the default. Copy the profile's `featureValues` into `features`. If `create_agent` rejects a model, use the default and say so. If it rejects the default, use the closest valid model of the same provider from `list_models`.

| Lens | Override, tier | Default | Prompt template |
|---|---|---|---|
| Judgment | `pstack-reflect-reviewers`, `pstack-judgment` | `claude/claude-opus-5-5` max | `references/judgment-reviewer.md` |
| Tooling | `pstack-reflect-tooling`, `pstack-frontier` | `codex/gpt-6-astra` max | `references/tooling-reviewer.md` |
| Divergent | `pstack-reflect-reviewers`, `pstack-judgment` | `claude/claude-opus-5-5` max | `references/divergent-reviewer.md` |

Pass each template verbatim, substituting the transcript path or digest where marked. Reviewers return findings in their final message (read it with `get_agent_activity`).

### 3. Synthesize

One `create_agent` call, with the `pstack-reflect-reviewers` profile, else `pstack-judgment` (default `claude/claude-opus-5-5` max), in the profile's mode (not a read-only or plan mode). The synthesizer's quality check includes spot-verifying citations, which can require MCP access. A read-only mode strips MCPs. Use `references/synthesizer.md` verbatim, with each reviewer's full output inlined where marked. The synthesizer returns a structured Accepted / Rejected / Backlog list.

### 4. Structural enforcement check

Sanity-check the synthesizer's Accepted list. For any item that would be enforced more reliably by a lint rule, script, metadata flag, or runtime check, move it from Accepted to Backlog. See the **encode-lessons-in-structure** principle skill.

### 5. Apply

Before applying any Accepted edit, present the synthesizer's full Accepted/Rejected/Backlog output to the user and wait for explicit approval. The user picks which subset to apply and may redirect routings. Skill changes affect every future agent in the org. Do not auto-apply.

Backlog items file to whatever devex / backlog tracker your team uses automatically. Only the Accepted list waits for approval.

For each approved Accepted item, follow the Routing field exactly:

- Trivial existing-skill edit (a one-line bullet, a tightened sentence, a stale fact corrected): parent does directly.
- Substantive existing-skill edit (a new section, a new pattern table, more than ~10 lines): hand to the `create-skill` skill if installed and run its draft / test / iterate loop.
- `tune description: <skill path>` (the skill exists but didn't trigger when it should have): hand to `create-skill` and run its description-optimization loop.
- `new skill via create-skill: <kebab-name>`: hand creation to `create-skill`. Do not invent the shape ad hoc.

If your environment ships a SKILL.md validator, run it on every touched skill before declaring done. Skip this step if it doesn't.

### 6. Summarize for the user

Short list, no preamble:

- Edits applied: `<skill path>`. What changed, one line each.
- New skills created: `<skill path>`. One line each (rare).
- Backlog filed to the devex tracker: `<issue title>` (`<tags>`). One line each.
- Dropped: one line per rejected finding + reason from the synthesizer.
