---
name: setup-benny
description: Configure Benny and prepare its triage and repro schedules. Use when installing Benny or changing its Slack, tracker, repository, routing, control, model, or budget settings.
disable-model-invocation: true
---

# Set up Benny

Benny ships as a dormant automation pack inside pstack. pstack installs only its normal skill root; this file and the two operational files are not slash skills.

The human enters setup by pointing a Paseo agent at the pack's `FOR_AGENTS.md`. The bootstrap flow copies the whole pack into the target repository, then reads this file directly at `.paseo/automations/benny/skills/setup-benny/SKILL.md`.

Benny needs external configuration and two live Paseo schedules. Paseo schedules start a fresh agent on a cron and have no Slack trigger, so each schedule polls the source channel. Use `run_schedule_once` for an on-demand run, including one fired from an external webhook.

Do not create or update a schedule until the user explicitly asks. Never put a secret value in pack files, prompts, or committed configuration.

## 1. Copy the pack and enable shared pstack skills

Do this before asking for Benny configuration and before calling `create_schedule`.

Ask which repository will run the schedules. The source pack is the directory containing `FOR_AGENTS.md`. The destination is `<target-repository>/.paseo/automations/benny/`.

Merge the entire source pack into the destination:

1. Create the destination when it is absent.
2. Copy every source file to the same relative path.
3. Preserve destination-only files. Never delete unrelated files during install or refresh.
4. Keep user-owned configuration, feature maps, and routing maps outside the destination. Never overwrite them.
5. When an existing source-managed file differs, inspect the diff and merge without discarding local edits. If ownership is ambiguous, stop and ask before replacing it.
6. Verify that the destination contains `FOR_AGENTS.md`, this setup file, both operational files, their references, and the templates.

If this file is already being read from the target destination, treat the copy as complete and run the same verification before continuing.

Install pstack into the target repository's `.agents/skills/`. Run this from the target repository root:

```bash
npx skills add khoi/paseo-pstack
```

Preserve every unrelated skill in `.agents/skills/`. If pstack is already installed there, update it in place. Verify the pstack skill directories exist after installing.

Start a fresh Paseo agent rooted in the target project. Verify that these shared pstack skills resolve from project scope:

- `how`
- `why`
- `tdd`
- `unslop`
- `principle-separate-before-serializing-shared-state`
- `principle-minimize-reader-load`
- `principle-guard-the-context-window`
- `principle-sequence-verifiable-units`
- `principle-fix-root-causes`
- `principle-prove-it-works`

Do not count a skill loaded from the current session or a user-scoped install. The check must show that a fresh agent in the target repository receives pstack through its project skills.

If project-scoped skill installation is unavailable or any shared dependency does not resolve, stop and explain the failure.

The Benny files are read directly from `.paseo/automations/benny/`. Do not copy that directory into `.agents/skills/` or expect its `SKILL.md` files to appear in the slash-skill list.

Tell the user that `.agents/skills/`, `.paseo/automations/benny/`, and any referenced secret-free configuration must be committed before either schedule is enabled. Do not commit them unless the user asks.

Once this check passes, live schedule prompts may read the committed operational files by their stable repository-relative paths. They must not embed a skill install path outside the repository or copy the file contents.

## 2. Adapt the configuration

Open these copied examples:

- `../../templates/configuration.example.yaml`
- `../reproduce-and-fix-issues/references/feature-map.example.md`

Create user-owned copies outside `.paseo/automations/benny/`. These are configuration files, not pack files. Example locations:

- Project config, such as `.paseo/benny/configuration.yaml`
- Project feature map, such as `.paseo/benny/feature-map.md`
- Project routing map, such as `.paseo/benny/routing.md`
- User config, such as `~/.config/benny/configuration.yaml`
- User feature map, such as `~/.config/benny/feature-map.md`

Fill one feature-map section for every user-facing feature the automation may reproduce. Keep it at the user point of view. Do not freeze implementation details or current code paths in the map.

Do not edit the copied examples. Pack refreshes may update source-managed files after conflict review, but they must never touch the user-owned copies.

Prefer committed, secret-free files in the target repository when a fresh schedule run must read them. Otherwise paraphrase the required values into the live prompt. Reference a repository file only after you confirm that the file is committed in the repository at the schedule's `cwd`.

Use stable repository-relative paths for committed pack and configuration files. Never reference the pstack source checkout or a skill install path outside the repository from a live schedule.

## 3. Fill the required choices

Ask for or confirm:

- Source Slack channel ID
- Optional operations or status channel ID
- Repository URL and default branch
- Triage identity or Slack user ID
- Issue tracker type, team, project, labels, and intake status
- Tracker adapter skill or MCP actions
- Optional routing map path
- Required control skill name
- Required user-facing feature-map path
- Status emoji strings
- Pull request URL format
- Cron cadence for each schedule
- Polling and effort budgets
- `provider/model` for triage, repro, code work, and media review

Use only `claude/...` or `codex/...` models that Paseo's `list_models` shows as available. Do not guess a model and do not carry over a private default.

The source channel, triage identity, repository, tracker adapter, control skill, and feature map must be explicit. Fail setup if any required value stays ambiguous.

Use pstack's `unslop` skill on the final schedule names and prompt shims before saving them.

## 4. Check integration capabilities

The triage automation needs:

- Read access to the configured source Slack channel and its threads
- Thread-reply access in that channel
- Attachment metadata and file download access when reports include media
- Search, read, create, and update access through the configured issue-tracker adapter

The repro automation needs:

- Read access to the source thread
- Thread-reply access in the source channel
- Optional post and edit access in the configured operations channel
- Repository read and history access
- A pull request action that can open a draft pull request
- The configured control-adapter skill

Prefer configured Slack MCP actions for reads and posts. Configure the Slack and tracker MCP servers for the provider each schedule runs on. The optional `BENNY_SLACK_BOT_TOKEN` may fill a narrow gap such as editing one operations status message or downloading an attachment. Store the value in a secret manager or environment, not in YAML.

Do not use undocumented integration endpoints.

## 5. Prepare the routing map

If the user wants reroutes or owner pings:

1. Copy `../triage-issue-reports/references/routing.example.md` outside `.paseo/automations/benny/`.
2. Replace every placeholder with public or organization-local values.
3. Keep owner pings off by default.
4. Allow a ping only for a configured feature owner or a confirmed likely regression author.

If no routing map is configured, triage may classify a report but must not guess a destination or owner.

## 6. Verify the control adapter

Read `../reproduce-and-fix-issues/references/control-adapter.md` and the user's completed feature map.

Confirm that the named skill can:

- Bring up the target app
- Navigate every mapped feature through the real UI
- Exercise mapped states through declared adapter actions
- Inspect state without forcing the result
- Capture screenshots
- Start and stop a recording
- Clean up its processes and temporary data

If any capability is missing, leave the repro automation disabled. It must fail closed rather than claim a reproduction it did not perform.

## 7. Prepare the live schedules

Ask whether this is first-time creation or configuration of existing schedules. Check with `list_schedules`.

Read `../../FOR_AGENTS.md` from the copied pack as the primary user-intent source for either path. Use it to understand the two triggers, tools, instructions, outcomes, and shared rules.

### First-time creation

Create one schedule at a time.

For each schedule:

1. Read the matching copied prompt template as secondary internal source material.
2. Turn `FOR_AGENTS.md`, the finished Benny configuration, and the template intent into a complete natural-language request.
3. Tell the live prompt to read and follow its exact committed operational file under `.paseo/automations/benny/`.
4. Use the stable repository-relative path, not a skill install path outside the repository. Do not copy the operational file contents into the live prompt.
5. Confirm that the Slack and tracker MCP servers are reachable from an agent on the schedule's provider.
6. Confirm that the copied pack and any referenced configuration files are committed in the repository at the schedule's `cwd`.
7. Show the draft: name, cron, provider, `cwd`, isolation, and the full prompt. Obtain approval.
8. Call `create_schedule`, then `pause_schedule` right away so it stays disabled until the thread-safety test passes.
9. Finish this schedule before starting the next one.

Give `create_schedule` this complete triage intent, filled from configuration:

- Name `benny-triage`.
- Read and follow `.paseo/automations/benny/skills/triage-issue-reports/SKILL.md` for every run.
- Run on the configured triage cron with the configured triage `provider/model`, `cwd` at the target repository, and `isolation: "local"`.
- Poll for new top-level reports in the configured source Slack channel.
- Read the triggering thread and reply only inside it.
- Use the configured issue-tracker integration.
- Classify, inspect evidence, trace cause, dedupe, and create only clear new bugs.
- End one thread-only verdict with the configured `[benny:bug]`, `[benny:performance]`, or `[benny:other]` marker and optional tracker URL.
- Never post a source-channel root message.

After the triage schedule is created and paused, give `create_schedule` this complete repro and fix intent:

- Name `benny-reproduce`.
- Read and follow `.paseo/automations/benny/skills/reproduce-and-fix-issues/SKILL.md` for every run.
- Run on the configured repro cron with the configured repro `provider/model`, `cwd` at the target repository, and `isolation: "worktree"` so each run gets its own Paseo worktree.
- Poll for the same new top-level reports in the configured source Slack channel.
- Use the configured repository and default branch.
- Read the source thread and reply only inside it.
- Include pull request creation and the configured tracker, control-adapter, and feature-map requirements. Paraphrase mapped user paths and states unless you confirmed an eligible committed file in the same repository.
- Wait for a trusted triage marker before acting.
- Reproduce the exact symptom twice through the mapped real UI and capture evidence.
- Verify an existing fix without authoring over it.
- Attempt an optional bounded fix only after confirmed repro, then open a draft pull request when proof and checks pass.
- Never post a source-channel root message.

### Existing schedules

Do not call `create_schedule` for a schedule that already exists. Find it with `list_schedules` and `inspect_schedule`.

Finish configuration, routing, control-adapter, and feature-map validation. Then give the user this concise update checklist.

For the existing triage schedule, update:

- Name
- Direct instruction to read `.paseo/automations/benny/skills/triage-issue-reports/SKILL.md`
- Cron, provider, `cwd`, and the source-channel poll
- Slack thread read and reply capabilities
- Issue-tracker integration
- Paraphrased triage instructions, thread-only rule, and Benny verdict markers

For the existing repro schedule, update:

- Name
- Direct instruction to read `.paseo/automations/benny/skills/reproduce-and-fix-issues/SKILL.md`
- Cron, provider, `cwd`, and the matching source-channel poll
- Repository and default branch
- Slack thread read and reply capabilities
- Pull request action
- Tracker, control-adapter, and feature-map requirements
- Paraphrased marker wait, evidence, verification, and bounded-fix instructions

After the user approves the checklist, change each existing schedule in place with `update_schedule`. Do not create replacements or duplicates.

### Creation boundary

Never write the daemon's schedule storage directly. For new schedules, the only finish path is a reviewed `create_schedule` call.

Do not resume either schedule until the thread-safety test passes. Drive test runs with `run_schedule_once` and read them with `schedule_logs`.

## 8. Test thread safety

Use a test channel or a harmless test report.

Before testing, confirm that the target repository's `.agents/skills/`, `.paseo/automations/benny/`, and every referenced secret-free configuration file are committed on the branch checked out at each schedule's `cwd`. Confirm that both live prompts point at their exact committed operational files. If any check fails, stop. Tell the user that the schedule cannot be enabled yet.

Verify:

1. Triage stores the root `thread_ts` and posts exactly one verdict as a reply.
2. The verdict contains one configured marker.
3. Repro accepts the marker only from the configured triage identity.
4. Repro keeps the same immutable source coordinates.
5. No source-channel root message appears.
6. A delegated worker cannot use any Slack write action.
7. Missing coordinates, a deleted parent, or a failed preflight produces no post and no tracker issue.

Enable normal traffic with `resume_schedule` only after all seven checks pass.
