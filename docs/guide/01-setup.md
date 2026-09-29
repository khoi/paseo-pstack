# Set up pstack

In this page you install the skills, configure Explorer and Worker models, and run your first task. Setup is one command plus a short conversation.

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

[`/setup-pstack`](../../skills/setup-pstack/SKILL.md) detects available models and maintains three profiles: `pstack-explorer` for investigation, `pstack-worker` for implementation and design candidates, and `pstack-judge` for explanation, synthesis, and review. Explorer and Worker default to Sol xhigh and retain existing choices on reruns. Judge always uses Astra xhigh.

With no pstack profiles, every role keeps the skill's default. To restore the defaults, delete the `pstack-*` profiles. A rerun of `/setup-pstack` starts from your current profiles.

Each review launches one Judge. Arena launches three Worker candidates by default; request a different candidate count per task. Setup removes retired role-specific and numbered profiles.

## Accept the verification offer, or don't

At the end of setup, `/setup-pstack` looks for a way to prove app behavior in your project, either a `verify-*` skill or an existing harness. If it finds neither, it offers once to generate one with [`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md).

Say yes and it writes `.agents/skills/verify-<app>/`, a project-local skill that teaches agents to drive your app the way a user does. It proves the skill works once before handing it over. Say no and setup moves on. You can run `/create-verification-skill` yourself any time. [Verify and ship](./06-verify-and-ship.md#create-a-project-verification-skill) covers when it earns its place.

After setup, start a new chat. The profiles apply to new sessions.

## Run your first task

Pick something real but small, and describe it the way you'd describe it to a colleague:

```text
/poteto-mode add a --json flag to this command. text output stays byte-identical. verify both.
```

Watch the todo list. Its first items are the matched playbook's steps copied in, the Feature playbook for this prompt. If `/poteto-mode` skips a step, the step stays in the list with `skip: <reason>`, so you can see what it chose not to do.

From here you can type normal follow-ups. `/poteto-mode` is sticky. It stays on for the conversation until you opt out by saying so.

Next: [Route work through `/poteto-mode`](./02-poteto-mode.md).
