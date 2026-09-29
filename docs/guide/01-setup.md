# Set up pstack

In this page you install the skills, pick which models pstack uses, and run your first task. Setup is one command plus a short conversation.

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

[`/setup-pstack`](../../skills/setup-pstack/SKILL.md) detects the models you have access to, asks for a reasoning budget, shows you each role (explorers, judgment, the review panels), and asks what you want. Answer the questions. It writes the pstack Agent profiles (the `pstack-*` entries in `~/.paseo/config.json`), which every pstack skill reads with `list_profiles`. Every role has its own profile, such as `pstack-how-explorer` or `pstack-why-investigators`, so moving a role means editing one profile.

With no pstack profiles, every role keeps the skill's default. To restore the defaults, delete the `pstack-*` profiles. A rerun of `/setup-pstack` starts from your current profiles.

The review panels have one numbered profile per seat, such as `pstack-arena-runners-1` and `pstack-arena-runners-2`. One subagent runs per profile, so the count sets the panel size. Setup also configures `swarm workers`, the default model for every `/swarm` worker unless a race names a model for each arm.

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
