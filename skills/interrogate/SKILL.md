---
name: interrogate
description: "Use for \"interrogate\", \"adversarial review\", \"challenge this\", \"stress test this code\", \"find blind spots\", or \"tear this apart\". One Astra xhigh Judge challenges changes against the stated intent."
disable-model-invocation: true
---

# Interrogate

Spawn one independent Judge to adversarially review code changes against the intent and rubric.

The deliverable is a synthesized verdict. Do NOT auto-apply changes.

## Step 1, Determine Scope

Identify what to review from context:

- If the user points at specific files or a diff, use that
- If on a feature branch, run `git diff main...HEAD` (or the appropriate base branch) for the full changeset
- If the user's message references recent work, gather the relevant files

Package the diff (or file contents) plus any surrounding context files the Judge needs to understand the code.

## Step 2, State the Intent

Before spawning the Judge, state the intent explicitly. Derive this from:

- The user's message
- Commit messages
- PR description if one exists
- The code itself

Write one clear paragraph. If you're unsure about the intent, ask the user before proceeding.

## Step 3, Spawn the Judge

Read `list_profiles` and launch one Paseo `create_agent` using `pstack-judge`. Always set provider/model to `codex/gpt-6-astra`, `settings.thinkingOptionId` to `xhigh`, `settings.modeId` to `full-access`, and `notifyOnFinish` to `true`. Keep the task read-only in the prompt. If Astra xhigh is unavailable, report the blocked review without substituting another model or effort.

Read `references/reviewer-prompt.md` and fill in the template with:
1. The stated intent
2. The diff or file contents
3. The review rubric from `references/rubric.md`
4. The code-quality lens from `references/code-quality-review.md`

Give the filled template to the Judge, including the code-quality lens.

## Step 4, Validate Findings

Read the Judge's findings, trace each claimed failure through the code, and deduplicate overlapping issues. Weight evidence and impact rather than model agreement. Distinguish confirmed issues from uncertainties.

## Step 5, Lead Judgment

You are the lead reviewer, a pragmatic senior engineer, not a neutral aggregator.

Read `references/lead-judgment.md` for the full framework.

Categorize every finding using these buckets:

- **Act on**. Real issues affecting correctness, security, or maintainability given the actual goals. These would block a real PR.
- **Consider**. Legitimate points, but you're not sure they outweigh the cost of addressing them right now. Worth the user's attention.
- **Noted**. Technically valid but not actionable. Context-dependent, premature optimization, or low-impact given the current stage.
- **Dismissed**. Wrong, nitpicky, or missing context. Brief explanation why.

For each finding, include:
- The supporting code or evidence
- The category (act on / consider / noted / dismissed)
- A one-line rationale for the categorization

## Output Format

Present the verdict in this structure:

### Intent
> [The stated intent paragraph from Step 2]

### Judge
GPT-6-Astra xhigh: [N findings]

### Act On
[Findings that should be addressed. For each: description, supporting evidence, why it matters.]

### Consider
[Findings worth thinking about. For each: description, supporting evidence, tradeoff involved.]

### Noted
[Valid but low-priority. Brief list.]

### Dismissed
[Rejected findings with brief rationale.]

### Uncertainties
[Claims that could not be verified and the evidence needed to settle them.]
