---
name: setup-pstack
description: Configure which models pstack uses per role and at what reasoning budget. Detects your available Paseo models and writes pstack Agent profiles that override the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
disable-model-invocation: true
---

# Setup pstack

Write pstack's model choices as Paseo **Agent profiles** (`daemon.agentProfiles` in `$PASEO_HOME/config.json`, default `~/.paseo/config.json`). Every role has its own profile, found by a fixed `id`, and every review panel has one numbered profile per seat. Skills read them at run time with the Paseo `list_profiles` tool and fall back to the defaults below only when a profile is missing. They also show up in the app under Agent profiles, so the user can edit them there.

## Steps

### 1. Detect available models

Call the Paseo `list_providers` tool, then `list_models` for `claude` and `codex`. pstack supports these two providers natively. That is the dependable source. Record each model's `provider/model` id and its `thinkingOptions`. pstack defaults leave Codex fast mode off. Only when the user asks for it on a profile, call `inspect_provider` on `codex` with `settings.model` set to that model to confirm its `fast_mode` feature exists. A bare `codex` call returns no features. If the model has no `fast_mode`, write it without the flag and say so. If a provider is unavailable, drop it from the options and say so. If you cannot detect any, ask the user to paste the model ids they have access to. Never write a model you have not confirmed is available.

### 2. Load current state

These are the role profiles and their defaults.

| role | profile id | default |
|---|---|---|
| feature, refactoring | `pstack-feature-refactoring` | `codex/gpt-6-sol` xhigh |
| bug-fix | `pstack-bug-fix` | `codex/gpt-6-sol` xhigh |
| judgment and prose | `pstack-judgment-and-prose` | `claude/claude-opus-5-5` max |
| hardest tasks | `pstack-hardest-tasks` | `claude/claude-opus-5-5` max |
| how explorer | `pstack-how-explorer` | `codex/gpt-6-sol` xhigh |
| how explainer | `pstack-how-explainer` | `claude/claude-opus-5-5` max |
| why investigators | `pstack-why-investigators` | `codex/gpt-6-sol` xhigh |
| why synthesizer | `pstack-why-synthesizer` | `claude/claude-opus-5-5` max |
| swarm workers | `pstack-swarm-workers` | `codex/gpt-6-sol` xhigh |

These are the panels. Each seat is its own profile, `<prefix>-<n>`, and the number of seats sets the panel size. The default is three seats: `-1` `claude/claude-opus-5-5` max, `-2` `codex/gpt-6-astra` max, `-3` `codex/gpt-6-sol` xhigh.

| panel | seat prefix |
|---|---|
| arena runners | `pstack-arena-runners` |
| arena cross-judge pool | `pstack-arena-cross-judge` |
| architect runners | `pstack-architect-runners` |
| interrogate reviewers | `pstack-interrogate-reviewers` |

Call `list_profiles`. Every role profile and panel seat present is a current choice. Every role or panel missing one starts from its default. `pstack-code`, `pstack-judgment`, and `pstack-frontier` are retired tier profiles: when present, use `pstack-code` as the current choice for every missing role whose default is `codex/gpt-6-sol`, `pstack-judgment` for every missing role whose default is `claude/claude-opus-5-5`, `pstack-frontier` for every missing role whose default is `codex/gpt-6-astra`, and the three of them in that order as seats `-1` `pstack-judgment`, `-2` `pstack-frontier`, `-3` `pstack-code` for every panel with no seats, then drop them. Drop any other `pstack-` id as a retired role. The current budget is the effort all role profiles share, if they share one.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Prefer your question tool over free text. Offer these four options with these exact labels, and name the current budget when step 2 found one.

- `unlimited — keep max`
- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) Apply it.** Build the working set from step 2: the current profiles on a re-run, the defaults otherwise. `unlimited` leaves every effort as in that set. `large`, `medium`, and `small` set the `thinkingOptionId` of every profile, panel seats included, to `xhigh`, `high`, or `medium`. The ladder is `ultracode`/`ultra` > `max` > `xhigh` > `high` > `medium` > `low`. A budget lowers any option above its target, `ultracode` and `ultra` included. `off` never changes. If the target is not among the model's detected `thinkingOptions`, use the model's highest detected option at or below the target, else mark the profile as needing a choice. Codex fast mode (`fast_mode: true`) does not change with the budget. So `small` turns `claude/claude-opus-5-5 max` into `claude/claude-opus-5-5 medium`, and `codex/gpt-6-sol xhigh` into `codex/gpt-6-sol medium`.

**(c) Show the roles and confirm.** Show every role with its model, then every panel with its seats, marking any model not in the detected set as needing a choice. Also list each profile step 2 dropped. Ask whether to accept as-is, move every role on one model to another, or change specific roles or panel seats, offering the detected models. Prefer your question tool over free text. Changing a panel's seat list sets its count. `arena cross-judge pool` is also a list, but Arena selects one seat from it whose provider differs from the parent's when possible. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

Also confirm the permission mode each profile launches in. Defaults: `auto` for `claude` and `auto-review` for `codex`. Never pick a read-only or plan mode, since it strips tools subagents need. For the same reason, never write a `plan_mode` feature value other than `false`. Offer each provider's other modes from `list_providers`.

### 4. Validate

Every model written must be in the detected set, with a detected `thinkingOptionId`. If a chosen model is not available, stop and ask again.

### 5. Write the profiles

Write one profile per role, then one per panel seat, using the ids from step 2. Always write every role and every seat, even when it matches its default, so the whole configuration is visible in the app. Name each `pstack · <role> · <model label> <thinkingOptionId>`, plus ` fast` with fast mode, and panel seats `pstack · <panel> <n> · <model label> <thinkingOptionId>`. Leave `notes` out.

Read `$PASEO_HOME/config.json`. `daemon.agentProfiles` is a whole list, and a missing key means `[]`: keep every profile whose `id` does not start with `pstack-`, drop every old `pstack-` profile, and append the new ones. Overwrite the whole pstack set so re-runs stay idempotent. Write the file back, then run `paseo reload` so the daemon applies it without a restart. Shape of one role and one seat:

```json
[
  {
    "id": "pstack-bug-fix",
    "name": "pstack · bug-fix · GPT-6-Sol xhigh",
    "provider": "codex",
    "model": "gpt-6-sol",
    "thinkingOptionId": "xhigh",
    "modeId": "auto-review"
  },
  {
    "id": "pstack-arena-runners-1",
    "name": "pstack · arena runners 1 · Opus 5.5 max",
    "provider": "claude",
    "model": "claude-opus-5-5",
    "thinkingOptionId": "max",
    "modeId": "auto"
  }
]
```

### 6. Confirm

Tell the user the profiles were written and that they apply to the next agent pstack launches. Re-running this skill updates them.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever pstack is installed: project or user skills). On no, move on without pushing.
