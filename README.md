# paseo-pstack

## Routing

Engineering workflows for Paseo, organized into five playbooks, reusable skills, and 23 principles.

`poteto-mode` selects a playbook and invokes skills through its steps and shared rules. Skills can invoke other skills and reference principles. You can also invoke skills directly.

- `[B]` Playbook.
- `[S]` Skill.
- `[P]` Principle, with the `principle-` prefix omitted.
- `[A]` Separate subagent launch, followed by its default model and reasoning effort.

Conditions in parentheses determine when a route applies. Reading a skill or principle does not itself launch an agent. Explorer and Worker counts follow task scope. Each review uses one Astra xhigh Judge.

```text
YOUR REQUEST
|
+-- Invoke any [S] directly
|   +-- Follow that skill's dependencies below
|
+-- [S] poteto-mode
    |
    +-- PLAYBOOK ROUTING: choose by task
    |   |
    |   +-- [B] investigation
    |   |   +-- [S] how
    |   |   +-- [S] why
    |   |   +-- [S] unslop
    |   |   +-- Code change needed? -> bug-fix / feature
    |   |
    |   +-- [B] bug-fix
    |   |   +-- Reproduce -> investigate -> fix -> verify
    |   |   +-- [S] how / why
    |   |   +-- [S] architect (cross-function changes)
    |   |   +-- Main agent implements
    |   |   +-- [S] tdd (cheap local test path)
    |   |   +-- [P] sequence-verifiable-units
    |   |   +-- [B] opening-a-pr
    |   |
    |   +-- [B] feature
    |   |   +-- [S] how
    |   |   +-- [S] architect
    |   |   +-- Main agent implements
    |   |   +-- [S] interrogate (contested design)
    |   |   +-- [P] model-the-domain
    |   |   +-- [P] separate-before-serializing-shared-state
    |   |   +-- [P] sequence-verifiable-units
    |   |   +-- [B] opening-a-pr
    |   |
    |   +-- [B] refactoring
    |   |   +-- [S] how
    |   |   +-- [S] architect (cross-function changes)
    |   |   +-- Main agent implements
    |   |   +-- [S] figure-it-out (large changes)
    |   |   +-- [P] model-the-domain
    |   |   +-- [P] foundational-thinking
    |   |   +-- [P] redesign-from-first-principles
    |   |   +-- [P] subtract-before-you-add
    |   |   +-- [P] laziness-protocol
    |   |   +-- [P] migrate-callers-then-delete-legacy-apis
    |   |   +-- [P] prove-it-works
    |   |   +-- [P] minimize-reader-load
    |   |   +-- [P] sequence-verifiable-units
    |   |   +-- [B] opening-a-pr
    |   |
    |   +-- [B] opening-a-pr
    |       +-- [S] deslop (external, if installed)
    |       +-- [S] technical-writing
    |       +-- [S] unslop
    |       +-- [S] interrogate (PR-opening delegates)
    |
    +-- DIRECT SKILL ROUTING: cross-cutting rules
    |   |
    |   +-- Understand a nontrivial change -> how
    |   +-- Design across functions ------> architect
    |   +-- Parallel work ----------------> swarm
    |   +-- Competing designs -----------> arena
    |   +-- Contested design -------------> interrogate
    |   +-- Prose ------------------------> unslop
    |   +-- Docs / PR / commit prose ------> technical-writing
    |   +-- Auditable long work ----------> show-me-your-work
    |   +-- Large task / no playbook ------> figure-it-out
    |
    +-- SKILL DEPENDENCIES
    |   |
    |   +-- [S] poteto-agent
    |   |   +-- Reads poteto-mode again
    |   |       +-- Same routing and principles
    |   |
    |   +-- [S] how
    |   |   +-- Simple question
    |   |   |   +-- [A] 1 explainer (explores and explains) -- GPT-6-Astra xhigh
    |   |   +-- Complex question
    |   |       +-- [A] 2-4 explorers in parallel -- GPT-6.1-Sol xhigh
    |   |       +-- Then [A] 1 explainer -- GPT-6-Astra xhigh
    |   |
    |   +-- [S] why
    |   |   +-- [A] Source-control investigator -- GPT-6.1-Sol xhigh
    |   |   +-- [A] Available-source investigators (parallel) -- GPT-6.1-Sol xhigh
    |   |   +-- Then [A] 1 synthesizer -- GPT-6-Astra xhigh
    |   |
    |   +-- [S] architect
    |   |   +-- [S] how
    |   |   +-- [S] why (ownership / layering changes)
    |   |   +-- [S] arena
    |   |   |   +-- [A] Read-only design candidates (Worker profile) -- GPT-6.1-Sol xhigh
    |   |   |   +-- Then [A] 1 cross-judge -- GPT-6-Astra xhigh
    |   |   +-- [S] interrogate (design pressure)
    |   |   +-- [P] exhaust-the-design-space
    |   |   +-- [P] foundational-thinking
    |   |   +-- [P] outcome-oriented-execution
    |   |   +-- [P] redesign-from-first-principles
    |   |   +-- [P] fix-root-causes
    |   |   +-- [P] subtract-before-you-add
    |   |
    |   +-- [S] arena
    |   |   +-- [A] N read-only design candidates (default 3) -- GPT-6.1-Sol xhigh
    |   |   +-- Then [A] 1 read-only cross-judge -- GPT-6-Astra xhigh
    |   |   +-- Parent selects, combines, and checks the design
    |   |   +-- [P] separate-before-serializing-shared-state
    |   |   +-- [P] laziness-protocol
    |   |   +-- [P] redesign-from-first-principles
    |   |   +-- [P] prove-it-works
    |   |
    |   +-- [S] swarm
    |   |   +-- [A] N workers in parallel -- GPT-6.1-Sol xhigh
    |   |   |   +-- Own worktree unless local access needed
    |   |   +-- Parent aggregates results
    |   |
    |   +-- [S] interrogate
    |   |   +-- [A] 1 Judge -- GPT-6-Astra xhigh
    |   |   |   +-- Intent, rubric, and code-quality lens
    |   |   +-- Parent judges and synthesizes findings
    |   |
    |   +-- [S] teach
    |   |   +-- [S] how / why / unslop
    |   |
    |   +-- [S] recall
    |   |   +-- [A] Parallel history readers -- GPT-6.1-Sol xhigh
    |   |   |   +-- Skip fan-out for 1-2 chats
    |   |   +-- [S] why (shared-record investigators)
    |   |   +-- [S] unslop
    |   |
    |   +-- [S] technical-writing
    |   |   +-- [S] unslop
    |   |
    |   +-- [S] figure-it-out
    |   |   +-- Reads poteto-mode's principles index
    |   |   +-- [S] architect (one-way design choices)
    |   |   +-- [S] show-me-your-work
    |   |   +-- [P] prove-it-works
    |   |   +-- [P] never-block-on-the-human
    |   |   +-- [P] foundational-thinking
    |   |   +-- [P] laziness-protocol
    |   |   +-- [P] separate-before-serializing-shared-state
    |   |   +-- [P] sequence-verifiable-units
    |   |   +-- [P] encode-lessons-in-structure
    |   |
    |   +-- [S] show-me-your-work
    |   |   +-- [S] unslop
    |   |   +-- [A] 1 trail Judge -- GPT-6-Astra xhigh
    |   |   +-- [P] encode-lessons-in-structure
    |   |
    |   +-- [S] automate-me
    |   |   +-- [A] Parallel history miners (when mining) -- GPT-6.1-Sol xhigh
    |   |   +-- Reads poteto-mode as a shape reference
    |   |   +-- External skill authoring if installed
    |   |   +-- [S] unslop
    |   |
    |   +-- [S] setup-pstack
    |   |   +-- Writes Explorer, Worker, and Judge profiles
    |   |
    |   +-- [S] create-verification-skill
    |   |   +-- Generates and proves a project skill
    |   |   +-- Suggests maintain-verification-skill
    |   |
    |   +-- [S] maintain-verification-skill
    |   |   +-- Reads an existing project verify skill
    |   |   +-- [A] 1 read-only reader per feature -- GPT-6.1-Sol xhigh
    |   |   +-- Parent drives live verification
    |   |
    |   +-- [S] tdd ------> failing test -> fix -> rerun
    |   +-- [S] unslop ---> writing rules
    |   +-- [S] bro ------> plain-language restatement
    |
    +-- PRINCIPLE ROUTING: read relevant leaf skills
        |
        +-- Core
        |   +-- laziness-protocol
        |   +-- foundational-thinking
        |   +-- redesign-from-first-principles
        |   +-- attack-the-premise
        |   +-- subtract-before-you-add
        |   +-- minimize-reader-load
        |   +-- outcome-oriented-execution
        |   +-- experience-first
        |   +-- exhaust-the-design-space
        |   +-- build-the-lever
        |
        +-- Architecture
        |   +-- model-the-domain
        |   +-- boundary-discipline
        |   +-- type-system-discipline
        |   +-- make-operations-idempotent
        |   +-- migrate-callers-then-delete-legacy-apis
        |   +-- separate-before-serializing-shared-state
        |
        +-- Verification
        |   +-- prove-it-works
        |   +-- fix-root-causes
        |   +-- sequence-verifiable-units
        |   +-- test-behavior-not-implementation
        |
        +-- Delegation
        |   +-- guard-the-context-window
        |   +-- never-block-on-the-human
        |
        +-- Meta
        |   +-- encode-lessons-in-structure
        |
        +-- References between principles and skills
            |
            +-- type-system-discipline
            |   +-- [P] boundary-discipline
            |   +-- [P] encode-lessons-in-structure
            |
            +-- sequence-verifiable-units
            |   +-- [P] prove-it-works
            |   +-- [P] build-the-lever
            |
            +-- prove-it-works
                +-- [S] show-me-your-work
                    (large / complex audited work)
```

## Setup

Run these commands on the machine where your agents run:

```sh
npx skills add getpaseo/paseo
npx skills add khoi/paseo-pstack
```

The first installs the Paseo reference skill; the second installs pstack. Select the providers you use. For project-only installation, run from the repository root and choose project scope.

You need a running Paseo daemon and at least one installed, signed-in provider. Check providers with `paseo provider ls`.

Enable Paseo tools under **Settings → your host → Orchestration → Enable Paseo tools**, or set this in `~/.paseo/config.json`:

```json
{ "daemon": { "mcp": { "injectIntoAgents": true } } }
```

Run `paseo reload` after changing the config. Restart existing agents so they receive the tools.

### Configure models

Invoke `setup-pstack` in a Paseo agent:

```text
/setup-pstack
```

It detects available models and writes six Agent profiles. `pstack-explorer` and `pstack-worker` default to GPT-6.1 Sol xhigh; `pstack-judge` always uses Astra xhigh. `pstack-arena-runner-1`, `pstack-arena-runner-2`, and `pstack-arena-runner-3` default to Astra, GPT-6.1 Sol, and Opus 5.5, all xhigh. Arena and architect launch one candidate per runner by default, followed by one Judge. Explorer, Worker, and Arena Runner model settings are configurable.

Edit profiles under **Settings → your host → Agent profiles**, or rerun setup. Keep the six profile IDs unchanged because skills look them up by ID. Judge model and effort remain fixed. Without profiles, skills use their documented defaults. See [setup-pstack](./skills/setup-pstack/SKILL.md) for the role IDs and defaults.

## Usage

Start with a goal and a checkable result:

```text
/poteto-mode retries write duplicate rows. reproduce it, fix the cause, and verify the original repro passes.
```

Use your provider's skill invocation syntax, such as `/poteto-mode` or `$poteto-mode`.

The mode selects a playbook, copies its steps into a task list, and invokes supporting skills as needed. It stays active across turns until you opt out. Say "new task" to explicitly close the current playbook and select another.

Most skills require explicit invocation. Once invoked, the mode loads its dependencies itself. The delegate skill `poteto-agent` is also discoverable by name.

For a specific procedure, invoke the skill directly:

```text
/how does cancellation propagate through the worker?
/interrogate review this branch for correctness and maintainability.
/swarm check each package with its check.sh. one worker per package.
```

The [guide](./docs/guide/README.md) walks through setup, investigation, design, implementation, and verification.

## Playbooks

| Playbook | Purpose |
|---|---|
| [investigation](./skills/poteto-mode/playbooks/investigation.md) | Answer a read-only question with evidence. |
| [bug-fix](./skills/poteto-mode/playbooks/bug-fix.md) | Reproduce a defect, fix its cause, and verify. |
| [feature](./skills/poteto-mode/playbooks/feature.md) | Design, implement, and verify new behavior. |
| [refactoring](./skills/poteto-mode/playbooks/refactoring.md) | Change structure while proving behavior is preserved. |
| [opening-a-pr](./skills/poteto-mode/playbooks/opening-a-pr.md) | Prepare ordered commits and a focused PR. |

## Skills

Principles guide decisions, skills provide reusable procedures, and playbooks sequence work. Principles are packaged as skills too; their index and application rules live in [poteto-mode](./skills/poteto-mode/SKILL.md#principles).

| Skill | Purpose |
|---|---|
| [poteto-mode](./skills/poteto-mode/SKILL.md) | Coordinate playbooks, skills, and principles. |
| [how](./skills/how/SKILL.md) | Explain architecture and runtime behavior. |
| [why](./skills/why/SKILL.md) | Investigate decisions and history across available sources. |
| [teach](./skills/teach/SKILL.md) | Combine how and why into a plain explanation. |
| [recall](./skills/recall/SKILL.md) | Reconstruct recent work from history and current evidence. |
| [architect](./skills/architect/SKILL.md) | Compare interfaces and module designs before implementation. |
| [arena](./skills/arena/SKILL.md) | Compare read-only design proposals and synthesize the strongest result. |
| [swarm](./skills/swarm/SKILL.md) | Distribute work across parallel workers and aggregate results. |
| [interrogate](./skills/interrogate/SKILL.md) | Review a change with one independent Astra xhigh Judge. |
| [tdd](./skills/tdd/SKILL.md) | Prove a regression test fails before the fix and passes after. |
| [figure-it-out](./skills/figure-it-out/SKILL.md) | Design a custom workflow for large or unmatched tasks. |
| [show-me-your-work](./skills/show-me-your-work/SKILL.md) | Keep and independently review a decision trail. |
| [create-verification-skill](./skills/create-verification-skill/SKILL.md) | Generate and prove a project-specific verification harness. |
| [maintain-verification-skill](./skills/maintain-verification-skill/SKILL.md) | Check a verification skill against source and live behavior. |
| [unslop](./skills/unslop/SKILL.md) | Remove formulaic language and filler. |
| [bro](./skills/bro/SKILL.md) | Restate the last response plainly. |
| [technical-writing](./skills/technical-writing/SKILL.md) | Structure and edit technical prose. |
| [automate-me](./skills/automate-me/SKILL.md) | Create a personal mode from recurring working preferences. |
| [setup-pstack](./skills/setup-pstack/SKILL.md) | Configure Explorer and Worker; keep Judge fixed at Astra xhigh. |

[poteto-agent](./skills/poteto-agent/SKILL.md) serves as the delegate entry point and loads the full mode before working.

## Execution

Paseo launches the subagents shown in the graph. You can inspect and steer them in the app. Completion notifications return control to the parent, which handles results according to the invoking skill.

Parallel writers use separate worktrees or output directories. `swarm` allows the current workspace when a worker needs local state unavailable in a fresh worktree. Permission modes come from role profiles and remain subject to session instructions.

Most workflow rules are Markdown instructions followed by the agent. They are not enforced by a central workflow engine.

## Optional dependencies

Some workflows use tools or skills installed separately:

- `deslop` for code cleanup before committing.
- `create-skill` for skill authoring when installed.
- A browser, desktop, or terminal driver for verifying behavior on the relevant surface.

The required Paseo reference skill is installed in the setup steps above.

## Troubleshooting

| Symptom | Check |
|---|---|
| `create_agent` is unavailable | Enable Paseo tools and restart the agent. |
| Judge cannot launch | Check Astra xhigh availability with `paseo provider ls` and model discovery; no alternate judge is selected. |
| A configured model is rejected | Rerun `setup-pstack` to detect available models. |
| A delegate is waiting | Inspect its activity and permission prompts in the app. |

For daemon issues, inspect `~/.paseo/daemon.log`.

## Update or remove

```sh
npx skills update
```

Remove selected skills by name:

```sh
npx skills remove poteto-mode setup-pstack
```

Delete unwanted `pstack-*` profiles under **Settings → your host → Agent profiles**. Rerunning setup updates the profile set using current choices as its starting point.

## Credits and license

Adapted from pstack by [poteto](https://x.com/poteto) (Lauren Tan) to run on [Paseo](https://paseo.sh). This fork retains five playbooks and 23 principles.

[MIT](./LICENSE).
