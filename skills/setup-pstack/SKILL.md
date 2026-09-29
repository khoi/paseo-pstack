---
name: setup-pstack
description: Configure pstack's Explorer, Worker, and three Arena Runner models and migrate its six Agent profiles. Judge always uses GPT-6-Astra xhigh. Use for /setup-pstack or changing pstack model choices.
disable-model-invocation: true
---

# Setup pstack

Maintain exactly six pstack Agent profiles in `daemon.agentProfiles` in `$PASEO_HOME/config.json` (default `~/.paseo/config.json`). Skills read them through `list_profiles`.

| Role | Profile ID | Model and effort | Used for |
|---|---|---|---|
| Explorer | `pstack-explorer` | `codex/gpt-6.1-sol` xhigh by default | Read-only code, history, and source investigation |
| Worker | `pstack-worker` | `codex/gpt-6.1-sol` xhigh by default | Delegated implementation and swarm tasks |
| Judge | `pstack-judge` | Always `codex/gpt-6-astra` xhigh | Explanation, synthesis, adversarial review, and candidate evaluation |
| Arena Runner 1 | `pstack-arena-runner-1` | `codex/gpt-6-astra` xhigh by default | First arena and architect design candidate |
| Arena Runner 2 | `pstack-arena-runner-2` | `codex/gpt-6.1-sol` xhigh by default | Second arena and architect design candidate |
| Arena Runner 3 | `pstack-arena-runner-3` | `claude/claude-opus-5-5` xhigh by default | Third arena and architect design candidate |

One Judge runs per review. Arena and architect launch one candidate per Arena Runner profile by default. An explicit candidate count cycles through runners 1, 2, 3; explicit per-candidate models override the rotation. Implementation otherwise stays in the main agent.

## Detect and load

Call `list_providers`, `list_models` for the required providers, and `list_profiles`. Confirm each selected model and reasoning effort is available. Judge requires Astra xhigh; if unavailable, report that setup cannot complete rather than substituting another model or effort.

Preserve existing Explorer, Worker, and Arena Runner settings on reruns unless the user asks to change them. For migration, take Explorer from `pstack-how-explorer` if present, otherwise `pstack-why-investigators`; take Worker from `pstack-swarm-workers`. Missing roles use the defaults above. In particular, migrating from the three-profile setup adds the three distinct Arena Runner defaults without copying Worker into them. Ignore retired `pstack-` profiles outside the six IDs above. Judge always receives its fixed model and effort, including when its existing profile differs.

## Choose settings

Apply model or budget changes only to Explorer, Worker, and Arena Runners. When choices are unspecified, use the existing settings or defaults and show the resulting six profiles. Ask only for missing choices or unavailable configurable models; do not silently replace an unavailable Arena Runner with Worker. Judge remains Astra xhigh under every budget.

Set `modeId` to `full-access` for Codex or `bypassPermissions` for Claude unless the user explicitly requested another mode. Keep Explorer, Arena Runner, and Judge tasks read-only through their prompts. Do not enable fast mode unless requested; validate any requested feature with `inspect_provider` before writing it.

## Write and reload

Preserve every profile whose ID does not start with `pstack-`. Replace the entire pstack set with the six profiles above, including the three numbered Arena Runners; do not recreate retired role-specific profiles or judge panels. Preserve other configuration and secrets. Name each profile `pstack · <role> · <model label> <effort>` and omit `notes`.

The Judge profile is:

```json
{
  "id": "pstack-judge",
  "name": "pstack · Judge · GPT-6-Astra xhigh",
  "provider": "codex",
  "model": "gpt-6-astra",
  "thinkingOptionId": "xhigh",
  "modeId": "full-access"
}
```

When configuration has a managed source, update its profile list too without copying live secrets into it. Save only the affected source paths using the host's dotfile workflow.

Run `paseo reload` without restarting the daemon. Verify the resulting pstack profile list contains exactly the six IDs in the table, with each model and effort matching the selected settings. Report the models and that they apply to subsequent launches.
