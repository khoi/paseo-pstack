---
name: setup-pstack
description: Configure which models pstack uses per role and at what reasoning budget. Detects your available Paseo models and writes pstack Agent profiles that override the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
disable-model-invocation: true
---

# Setup pstack

Write pstack's model choices as Paseo **Agent profiles** (`daemon.agentProfiles` in `$PASEO_HOME/config.json`, default `~/.paseo/config.json`). pstack finds them by fixed `id`. Three tier profiles cover every role, and an optional override profile moves one role to another model. Skills read them at run time with the Paseo `list_profiles` tool. They also show up in the app under Agent profiles, so the user can edit them there.

## Steps

### 1. Detect available models

Call the Paseo `list_providers` tool, then `list_models` for `claude` and `codex`. pstack supports these two providers natively. That is the dependable source. Record each model's `provider/model` id and its `thinkingOptions`. pstack defaults leave Codex fast mode off. Only when the user asks for it on a profile, call `inspect_provider` on `codex` with `settings.model` set to that model to confirm its `fast_mode` feature exists. A bare `codex` call returns no features. If the model has no `fast_mode`, write it without the flag and say so. If a provider is unavailable, drop it from the options and say so. If you cannot detect any, ask the user to paste the model ids they have access to. Never write a model you have not confirmed is available.

### 2. Load current state

These are the tier profiles. Every role uses its tier unless an override profile exists.

| tier profile | default | roles |
|---|---|---|
| `pstack-code` | `codex/gpt-6-sol` xhigh | feature, refactoring; bug-fix; perf-issue; hillclimb; how explorer; why investigators; swarm workers |
| `pstack-judgment` | `claude/claude-opus-5-5` max | judgment and prose; hardest tasks; how explainer; why synthesizer; reflect judgment, divergent, synthesizer |
| `pstack-frontier` | `codex/gpt-6-astra` max | reflect tooling |

Panels (arena runners, arena cross-judge pool, architect runners, interrogate reviewers) run one seat on each of the three tiers.

These are the override ids. A profile with one of them takes that role off its tier.

| role | override id |
|---|---|
| feature, refactoring | `pstack-feature-refactoring` |
| bug-fix | `pstack-bug-fix` |
| perf-issue | `pstack-perf-issue` |
| hillclimb | `pstack-hillclimb` |
| judgment and prose | `pstack-judgment-and-prose` |
| hardest tasks | `pstack-hardest-tasks` |
| how explorer | `pstack-how-explorer` |
| how explainer | `pstack-how-explainer` |
| why investigators | `pstack-why-investigators` |
| why synthesizer | `pstack-why-synthesizer` |
| reflect judgment, divergent, synthesizer | `pstack-reflect-reviewers` |
| reflect tooling | `pstack-reflect-tooling` |
| swarm workers | `pstack-swarm-workers` |

A panel is overridden by every profile whose `id` starts with its prefix, one seat each: `pstack-arena-runners-<n>`, `pstack-arena-cross-judge-<n>`, `pstack-architect-runners-<n>`, `pstack-interrogate-reviewers-<n>`. When any exist, they replace the three tier seats for that panel.

Call `list_profiles`. Every profile whose `id` starts with `pstack-` is a current choice. Any other `pstack-` id is from a retired role. Drop it. With no pstack profiles, start from the defaults. The current budget is the effort the three tier profiles share, if they share one.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Prefer your question tool over free text. Offer these four options with these exact labels, and name the current budget when step 2 found one.

- `unlimited — keep max`
- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) Apply it.** Build the working set from step 2: the current profiles on a re-run, the defaults otherwise. `unlimited` leaves every effort as in that set. `large`, `medium`, and `small` set the `thinkingOptionId` of every profile, overrides and panel seats included, to `xhigh`, `high`, or `medium`. The ladder is `ultracode`/`ultra` > `max` > `xhigh` > `high` > `medium` > `low`. A budget lowers any option above its target, `ultracode` and `ultra` included. `off` never changes. If the target is not among the model's detected `thinkingOptions`, use the model's highest detected option at or below the target, else mark the profile as needing a choice. Codex fast mode (`fast_mode: true`) does not change with the budget. So `small` turns `claude/claude-opus-5-5 max` into `claude/claude-opus-5-5 medium`, and `codex/gpt-6-sol xhigh` into `codex/gpt-6-sol medium`.

**(c) Show the roles and confirm.** Show every tier with its model and roles, then every override and panel seat, marking any model not in the detected set as needing a choice. Also list each profile step 2 dropped. Ask whether to accept as-is, change a tier's model, or move specific roles or panel seats to another model, offering the detected models. Prefer your question tool over free text. Moving a role writes its override profile. Moving it back to its tier deletes the override. A panel override lists every seat, so its length sets the count. `arena cross-judge pool` is also a list, but Arena selects one seat from it whose provider differs from the parent's when possible. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

Also confirm the permission mode each profile launches in. Defaults: `auto` for `claude` and `auto-review` for `codex`. Never pick a read-only or plan mode, since it strips tools subagents need. For the same reason, never write a `plan_mode` feature value other than `false`. Offer each provider's other modes from `list_providers`.

### 4. Validate

Every model written must be in the detected set, with a detected `thinkingOptionId`. If a chosen model is not available, stop and ask again.

### 5. Write the profiles

Write the three tier profiles, then one profile per override and panel seat. Use the ids from step 2. Name each `pstack · <tier or role> · <model label> <thinkingOptionId>`, plus ` fast` with fast mode. Leave `notes` out.

Read `$PASEO_HOME/config.json`. `daemon.agentProfiles` is a whole list, and a missing key means `[]`: keep every profile whose `id` does not start with `pstack-`, drop every old `pstack-` profile, and append the new ones. Overwrite the whole pstack set so re-runs stay idempotent. Write the file back, then run `paseo reload` so the daemon applies it without a restart. Default shape:

```json
[
  {
    "id": "pstack-code",
    "name": "pstack · code · GPT-6-Sol xhigh",
    "provider": "codex",
    "model": "gpt-6-sol",
    "thinkingOptionId": "xhigh",
    "modeId": "auto-review"
  },
  {
    "id": "pstack-judgment",
    "name": "pstack · judgment · Opus 5.5 max",
    "provider": "claude",
    "model": "claude-opus-5-5",
    "thinkingOptionId": "max",
    "modeId": "auto"
  },
  {
    "id": "pstack-frontier",
    "name": "pstack · frontier · GPT-6-Astra max",
    "provider": "codex",
    "model": "gpt-6-astra",
    "thinkingOptionId": "max",
    "modeId": "auto-review"
  }
]
```

### 6. Confirm

Tell the user the profiles were written and that they apply to the next agent pstack launches. Re-running this skill updates them.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever pstack is installed: project or user skills). On no, move on without pushing.
