---
name: setup-pstack
description: Configure pstack's Explorer and Worker models and migrate its three Agent profiles. Judge always uses GPT-6-Astra xhigh. Use for /setup-pstack or changing pstack model choices.
disable-model-invocation: true
---

# Setup pstack

Maintain exactly three pstack Agent profiles in `daemon.agentProfiles` in `$PASEO_HOME/config.json` (default `~/.paseo/config.json`). Skills read them through `list_profiles`.

| Role | Profile ID | Model and effort | Used for |
|---|---|---|---|
| Explorer | `pstack-explorer` | `codex/gpt-6.1-sol` xhigh by default | Read-only code, history, and source investigation |
| Worker | `pstack-worker` | `codex/gpt-6.1-sol` xhigh by default | Delegated implementation and competing design candidates |
| Judge | `pstack-judge` | Always `codex/gpt-6-astra` xhigh | Explanation, synthesis, adversarial review, and candidate evaluation |

One Judge runs per review. Arena launches three Worker candidates by default; candidate count is a task choice, independent of the number of profiles. Implementation otherwise stays in the main agent.

## Detect and load

Call `list_providers`, `list_models` for the required providers, and `list_profiles`. Confirm each selected model and reasoning effort is available. Judge requires Astra xhigh; if unavailable, report that setup cannot complete rather than substituting another model or effort.

Preserve existing Explorer and Worker settings on reruns unless the user asks to change them. For migration, take Explorer from `pstack-how-explorer` if present, otherwise `pstack-why-investigators`; take Worker from `pstack-swarm-workers`. Missing roles use the defaults above. Ignore every other old `pstack-` profile. Judge always receives its fixed model and effort, including when its existing profile differs.

## Choose settings

Apply model or budget changes only to Explorer and Worker. When choices are unspecified, use the existing settings or defaults and show the resulting three profiles. Ask only for missing choices or unavailable Explorer/Worker models. Judge remains Astra xhigh under every budget.

Set `modeId` to `full-access` for Codex or `bypassPermissions` for Claude unless the user explicitly requested another mode. Keep Explorer and Judge tasks read-only through their prompts. Do not enable fast mode unless requested; validate any requested feature with `inspect_provider` before writing it.

## Write and reload

Preserve every profile whose ID does not start with `pstack-`. Replace the entire pstack set with the three profiles above; never recreate role-specific or numbered panel profiles. Preserve other configuration and secrets. Name each profile `pstack · <role> · <model label> <effort>` and omit `notes`.

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

Run `paseo reload` without restarting the daemon. Verify the resulting profile list contains exactly `pstack-explorer`, `pstack-worker`, and `pstack-judge` for pstack. Report the models and that they apply to subsequent launches.
