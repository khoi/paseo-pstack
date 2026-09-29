---
name: arena
description: "Spawn N parallel read-only design candidates at the same task, pick a base, graft the strongest parts of the losers into it. Use for /arena, 'arena this', 'throw it in the arena', or when one attempt at a non-trivial design would lock in the wrong shape."
disable-model-invocation: true
---

# Arena

Fan out N parallel read-only design proposals for the same task. Read every candidate end to end. Pick the strongest as the base. Graft the best ideas from the others into it. Verify the synthesized result.

Candidates and the judge may inspect code and evidence, but must not edit files, run mutating commands, implement changes, commit, open PRs, or spawn agents. Return proposals in the response, including code sketches as text. The parent owns synthesis and any later implementation. These are task constraints, regardless of the launch permission mode.

## Start

Open a todolist with one entry per phase before launching anything.

1. Frame
2. Fan out
3. Cross-judge
4. Pick
5. Graft
6. Verify

## Phase A: Frame

The N candidates will receive the same prompt, so the prompt is the contract.

1. State the design proposal each candidate is producing.
2. Derive the rubric. State what success looks like for *this* task, then turn it into 3-6 concrete gradeable criteria. The rubric is the picker's tool in Phase D. Candidates only see the task.
3. Pick the runners. Read `list_profiles` and launch one candidate per Arena Runner profile below by default. For an explicit candidate count, cycle through runners 1, 2, 3 until that count is reached. Explicit per-candidate model choices override this rotation. Each candidate independently explores the same brief and returns a read-only proposal.
4. Assign candidate labels. Candidates share read-only access to the relevant source and return their proposals in their responses. No candidate worktrees or output files.

| Profile ID | Default provider/model | Effort |
|---|---|---|
| `pstack-arena-runner-1` | `codex/gpt-6-astra` | `xhigh` |
| `pstack-arena-runner-2` | `codex/gpt-6.1-sol` | `xhigh` |
| `pstack-arena-runner-3` | `claude/claude-opus-5-5` | `xhigh` |

Copy each profile's `provider/model`, `thinkingOptionId`, and `featureValues` into `create_agent.provider`, `settings.thinkingOptionId`, and `settings.features`. Set `settings.modeId` to `full-access` for Codex or `bypassPermissions` for Claude unless the user requested another mode.

A missing profile uses that seat's default, never `pstack-worker`. Validate missing-profile defaults or explicit model overrides with `list_models`; if unavailable, report the seat as a dropout rather than silently substituting a model.

## Phase B: Fan out

Spawn all N Paseo subagents in one message with `create_agent` and `notifyOnFinish: true` (the default, so don't poll), each with the task, the path to the shared grounding, its candidate label, the read-only constraints above, and instructions to return both the design proposal and a short rationale.

Each rationale names the alternatives the candidate considered and what it rejected.

If a candidate fails to produce output, proceed with the remaining candidates and note the dropout in the synthesis record. If none finish, report the arena as blocked. Architect still requires at least two structurally distinct candidates before synthesis.

## Phase C: Cross-judge

After all Phase B candidates complete, launch exactly one Judge using `pstack-judge`: always `codex/gpt-6-astra`, `thinkingOptionId: xhigh`, and `modeId: full-access`. Read the profile with `list_profiles`, but never change the Judge model or effort or select by provider diversity. If Astra xhigh is unavailable, report the blocked judgment step. Say read-only in the prompt. The Judge sees the rubric and complete candidate responses by label, scores each criterion, and recommends a base with rationale. It runs in parallel with the parent's reading in Phase D, after the candidate responses are complete.

## Phase D: Pick a base

Read every candidate end to end before picking.

Score each candidate against the rubric criterion by criterion, not on holistic feel. Compare against the cross-judge. Agreement on the base confirms the pick. Disagreement means one of you is biased or the rubric was ambiguous. Read both rationales before deciding.

Pick the base on which candidate a future maintainer can extend most easily without breaking invariants. Prefer the cleaner boundary or smaller API when two feel tied, per the Laziness Protocol.

Record the pick and the reason in a short synthesis note alongside the base proposal, including the cross-judge's verdict.

## Phase E: Graft

Walk each losing candidate once more and identify what is worth porting into the base. The signal is usually one or two things per candidate, not most of it.

Fold each design idea into the proposal, per the **redesign-from-first-principles** principle skill. Don't paste mechanically. The result has to remain coherent under one mental model.

Record what was grafted, from which candidate, and what was rejected and why.

When N candidates converge on the same shape, that is a strong agreement signal. Note the convergence in the record and return the consensus shape. No graft is needed. When N candidates wildly diverge, Phase A was under-specified. Reframe and re-run rather than averaging the divergence.

## Phase F: Verify

Check the synthesized design against the source, constraints, and caller examples, per the **prove-it-works** principle skill. Distinguish facts verified by inspection from assumptions that need implementation or runtime proof. Design review is not proof that an implementation works.

If verification surfaces a problem the arena did not catch, either Phase A was wrong (re-frame and re-run) or one candidate caught it and you missed the graft (go back to Phase E). Don't paper over.

## Outputs

One synthesized design proposal returned to the caller, not an implementation. One short synthesis note alongside, naming the base, the grafts (with source candidate), the rejections, the dropouts if any, and the verification result.
