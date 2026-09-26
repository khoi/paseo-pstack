---
name: setup-pstack
description: Configure which models pstack uses per role and at what reasoning budget. Detects your available Paseo models and writes pstack Agent profiles that override the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
disable-model-invocation: true
---

# Setup pstack

Write pstack's per-role model choices as Paseo **Agent profiles** (`daemon.agentProfiles` in `$PASEO_HOME/config.json`, default `~/.paseo/config.json`). Every pstack profile has an `id` starting with `pstack-`, and its `notes` name the pstack roles it serves. Skills read them at run time with the Paseo `list_profiles` tool. They also show up in the app under Agent profiles, so the user can edit them there.

## Steps

### 1. Detect available models

Call the Paseo `list_providers` tool, then `list_models` for `claude` and `codex`. pstack supports these two providers natively. That is the dependable source. Record each model's `provider/model` id and its `thinkingOptions`, and call `inspect_provider` on `codex` to confirm the `fast_mode` feature exists. If a provider is unavailable, drop it from the options and say so. If you cannot detect any, ask the user to paste the model ids they have access to. Never write a model you have not confirmed is available. The alias `inherit-parent` is always valid even though it is not a detected model.

### 2. Load current state

The default role-to-model mapping is the profile set shown in step 5 below. Call `list_profiles`. If any profile's `id` starts with `pstack-`, treat those profiles as the current choices: each role it names is assigned to it, and its `pstack budget:` line is the current budget. A role named by no pstack profile, when pstack profiles exist, is `inherit-parent`. Otherwise start from the defaults. A role name that is not in step 5, such as `how critics`, is from a retired role. Drop it.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Prefer your question tool over free text. Offer these four options with these exact labels, and name the current budget when the profiles record one.

- `unlimited — keep max`
- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) Apply it.** Build the working table from the skill defaults, and on a re-run keep any role you changed by model, list, or alias (`inherit-parent`). `unlimited` leaves every effort as in that table. `large`, `medium`, and `small` set the `thinkingOptionId` of every real model, panel entries included, to `xhigh`, `high`, or `medium`. The ladder is `max` > `xhigh` > `high` > `medium` > `low`. If the target is not among the model's detected `thinkingOptions`, use the model's highest detected option at or below the target, else mark the role as needing a choice. Codex fast mode (`fast_mode: true`) does not change with the budget. `inherit-parent` does not change. So `small` turns `claude/claude-opus-5-5 max` into `claude/claude-opus-5-5 medium`, and `codex/gpt-6-sol xhigh fast` into `codex/gpt-6-sol medium fast`.

**(c) Show the roles and confirm.** Show every role with its model, marking any real model not in the detected set as needing a choice. Also list each role step 2 dropped. Ask whether to accept as-is or change specific roles, offering the detected models plus `inherit-parent` (this role runs on the parent agent's own provider and model) as the options. Prefer your question tool over free text. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, so the list length sets the count. Panel entries must be real models. `inherit-parent` applies only to a whole role. `arena cross-judge pool` is also a list, but Arena selects one value from it whose provider differs from the parent's when possible. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

Also confirm the permission mode each profile launches in. Defaults: `auto` for `claude` and `auto-review` for `codex`. Never pick a read-only or plan mode, since it strips tools subagents need. Offer each provider's other modes from `list_providers`.

### 4. Validate

Every real model written must be in the detected set, with a detected `thinkingOptionId`. `inherit-parent` always passes. If a chosen real model is not available, stop and ask again.

### 5. Write the profiles

Group the roles by launch bundle (provider, model, thinking option, mode, features). Write one profile per distinct bundle. Its `notes` has a `pstack roles:` line naming every role it serves, separated by `;`, and a `pstack budget:` line with the chosen label and its target effort. A panel list that repeats a bundle writes the role with a count suffix, for example `arena runners x2`. `inherit-parent` roles go in no profile.

Read `$PASEO_HOME/config.json`. `daemon.agentProfiles` is a whole list: keep every profile whose `id` does not start with `pstack-`, drop every old `pstack-` profile, and append the new ones. Overwrite the whole pstack set so re-runs stay idempotent. Write the file back, then run `paseo reload` so the daemon applies it without a restart. Default shape:

```json
[
  {
    "id": "pstack-opus-max",
    "name": "pstack · Opus 5.5 max",
    "provider": "claude",
    "model": "claude-opus-5-5",
    "thinkingOptionId": "max",
    "modeId": "auto",
    "notes": "pstack model configuration. Delete a role to make it inherit the parent agent's model.\npstack budget: unlimited (max)\npstack roles: judgment and prose; hardest tasks; how explainer; why synthesizer; reflect judgment, divergent, synthesizer; arena runners; arena cross-judge pool; architect runners; interrogate reviewers"
  },
  {
    "id": "pstack-astra-max",
    "name": "pstack · GPT-6-Astra max",
    "provider": "codex",
    "model": "gpt-6-astra",
    "thinkingOptionId": "max",
    "modeId": "auto-review",
    "notes": "pstack model configuration. Delete a role to make it inherit the parent agent's model.\npstack budget: unlimited (max)\npstack roles: reflect tooling; arena runners; arena cross-judge pool; architect runners; interrogate reviewers"
  },
  {
    "id": "pstack-sol-xhigh-fast",
    "name": "pstack · GPT-6-Sol xhigh fast",
    "provider": "codex",
    "model": "gpt-6-sol",
    "thinkingOptionId": "xhigh",
    "modeId": "auto-review",
    "featureValues": { "fast_mode": true },
    "notes": "pstack model configuration. Delete a role to make it inherit the parent agent's model.\npstack budget: unlimited (max)\npstack roles: feature, refactoring; bug-fix; perf-issue; hillclimb; how explorer; why investigators; swarm workers; arena runners; arena cross-judge pool; architect runners; interrogate reviewers"
  }
]
```

### 6. Confirm

Tell the user the profiles were written and that they apply to the next agent pstack launches. Re-running this skill updates them.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever pstack is installed: project or user skills). On no, move on without pushing.
