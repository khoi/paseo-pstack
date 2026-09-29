# Set up pstack

In this page you install the skills, configure Explorer, Worker, and Arena Runner models, and run your first task. Setup is one command plus a short conversation.

## Install the skills

In a terminal, run:

```bash
npx skills add getpaseo/paseo
npx skills add khoi/paseo-pstack
```

The installer confirms the skills are installed.

## Pick your models

Run:

```text
/setup-pstack
```

[`/setup-pstack`](../../skills/setup-pstack/SKILL.md) detects available models and maintains six profiles:

| Profile | Default model | Role |
|---|---|---|
| `pstack-explorer` | GPT-6.1 Sol xhigh | Investigation |
| `pstack-worker` | GPT-6.1 Sol xhigh | Delegated implementation and swarm tasks |
| `pstack-judge` | Astra xhigh, fixed | Explanation, synthesis, and review |
| `pstack-arena-runner-1` | Astra xhigh | Arena and architect candidate 1 |
| `pstack-arena-runner-2` | GPT-6.1 Sol xhigh | Arena and architect candidate 2 |
| `pstack-arena-runner-3` | Opus 5.5 xhigh | Arena and architect candidate 3 |

Explorer, Worker, and Arena Runners retain existing choices on reruns. Judge always uses Astra xhigh. Migrating from the three-profile setup adds the three Arena Runners with their distinct defaults.

With no pstack profiles, every role keeps the skill's default. To restore the defaults, delete the `pstack-*` profiles. A rerun of `/setup-pstack` starts from your current profiles.

Each review launches one Judge. Arena and architect launch one candidate per Arena Runner by default; request a different candidate count or per-candidate models per task. Setup keeps the six IDs above and removes retired pstack profiles.

## Optional project verification

Run [`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md) when your project needs a way for agents to drive the app and prove its behavior. [Verify and ship](./06-verify-and-ship.md#create-a-project-verification-skill) covers when it earns its place.

The profiles apply to subsequent subagent launches.

## Run your first task

Pick something real but small, and describe it the way you'd describe it to a colleague:

```text
/poteto-mode add a --json flag to this command. text output stays byte-identical. verify both.
```

Watch the todo list. Its first items are the matched playbook's steps copied in, the Feature playbook for this prompt. If `/poteto-mode` skips a step, the step stays in the list with `skip: <reason>`, so you can see what it chose not to do.

From here you can type normal follow-ups. `/poteto-mode` is sticky. It stays on for the conversation until you opt out by saying so.

Next: [Route work through `/poteto-mode`](./02-poteto-mode.md).
