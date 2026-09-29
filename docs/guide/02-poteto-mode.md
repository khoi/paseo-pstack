# Route work through `/poteto-mode`

`/poteto-mode` is the front door. You give it a goal, it matches one of five playbooks, copies that playbook's steps into the todo list, and calls the other skills as the steps need them. In this page you learn what a good prompt looks like, and how little of one you actually need.

![A dispatcher pulls a switch lever to route robots on rail handcars toward lit gates, under a /poteto-mode departure board listing BUG FIX, FEATURE, and INVESTIGATION.](./images/router.jpg)

## What happens to your prompt

```mermaid
flowchart TD
    A[Your prompt] --> B[poteto-mode]
    B --> C[Read the Principles section]
    C --> D{Match the task}
    D -->|Read-only question| E[Investigation]
    D -->|Defect| F[Bug fix]
    D -->|New behavior| G[Feature]
    D -->|Structure only| H[Refactoring]
    D -->|Large work or no match| J[figure-it-out]
    E --> K[Verify and report]
    F --> K
    G --> K
    H --> K
    J --> K
```

The fifth playbook, [Opening a PR](../../skills/poteto-mode/playbooks/opening-a-pr.md), handles completed code changes. The [playbook directory](../../skills/poteto-mode/playbooks/) has the full set.

## Say the goal, not the ceremony

You don't write a spec. You say what's wrong or what you want, plus anything you already know that saves the agent time:

```text
/poteto-mode users get two notifications after a retry. repro first, then fix and verify.
```

That's a Bug fix prompt. "repro first" is a real constraint, not politeness, and the playbook honors it. Watch the todo list fill with the Bug fix steps. A skipped step stays visible with `skip: <reason>`.

When the conversation already carries the context, the prompt shrinks to almost nothing. All of these are enough:

```text
/poteto-mode do it
```

```text
continue
```

```text
keep going until done
```

Short works because the mode is sticky and the playbook holds the structure. Your words carry the intent, and the skill carries the rigor.

## Switch tasks with "new task"

A long chat accumulates context from the last task. When you change subjects, say so:

```text
/poteto-mode new task. figure out why the cache entry survives logout. don't change any code yet.
```

"new task" makes `/poteto-mode` close the prior playbook and re-match from scratch. It usually re-matches on its own when your question plainly isn't the next step, so the phrase matters most when your question could pass for one. "don't change any code yet" pins this one to Investigation. Without it, "figure out why" tends to end in a committed fix.

Switch after the current turn finishes. A message sent to a running Paseo tab interrupts it, and the agent drops the step it was on. For unrelated work, open a new tab instead.

## Give parallel work its own worktree

If you run several agents against one repository, they will fight over the working tree. Ask for isolation up front:

```text
/poteto-mode new task. branch off <base> in a fresh worktree, then port the parser change there.
```

Each task in its own branch and worktree means no agent stomps another's files. The [Opening a PR playbook](../../skills/poteto-mode/playbooks/opening-a-pr.md) already works from a worktree for code changes, so mostly you only say this when a specific base or location matters.

**Pitfall:** don't enumerate skills in your prompt ("use /how, then /architect, then /arena..."). The playbook already sequences them, and a hand-written sequence usually reorders or drops steps the playbook would have kept. Name a skill only when you want to override a specific choice.

Read [`poteto-mode`](../../skills/poteto-mode/SKILL.md) itself for the full routing rules.

Next: [Understand the code](./03-understand.md).
