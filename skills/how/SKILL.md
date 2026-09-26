---
name: how
description: "Use for \"how does X work\", code walkthroughs before changing something, and placement / ownership / layering questions (\"where should this live\", \"which package owns this\", \"is this the right layer\"). Explains subsystem architecture, runtime flow, onboarding mental models. Use why for motivation."
disable-model-invocation: true
---

# How

Explore the codebase to answer "how does X work?" questions. Produce architectural explanations at the level of a senior engineer onboarding onto a subsystem, enough to build a working mental model, not so much that it reads like annotated source code.

Each spawn below is a Paseo `create_agent` call that names an override profile, a tier profile, and a default. Set the `create_agent` launch (`provider/model`, `thinkingOptionId`, `modeId`, features) from the pstack Agent profiles (read them with `list_profiles`): the role's override profile if it exists, else its tier profile, else the default. Copy the profile's `featureValues` into `features`. If `create_agent` rejects a model, use the default and say so. If it rejects the default, use the closest valid model of the same provider from `list_models`.

## Step 1. Assess Complexity

If the scope is ambiguous, state your interpretation and explore. The user can redirect.

- **Simple** (a single module, a small utility, a narrow question such as "how does function X work"): no explorers. One explainer explores and explains in a single pass. Go to Step 2b.
- **Complex** (a subsystem spanning multiple files or services, a cross-cutting feature, a full architectural overview): spawn parallel explorers first, then hand off to the explainer. Go to Step 2a.

When in doubt, take the simple path.

## Step 2a. Explore (complex questions only)

Decompose the question into 2 to 4 exploration angles, each a distinct slice of the subsystem. Spawn all explorers in a single message:

- profile: `pstack-how-explorer`, else `pstack-code`, default `codex/gpt-6-sol` xhigh
- read-only: say so in the prompt (no edits)

Each explorer gets the prompt in `references/explorer-prompt.md` with its angle filled in. Then go to Step 3.

## Step 2b. Direct Explain (simple questions)

Spawn one Paseo subagent that explores and explains in one pass:

- profile: `pstack-how-explainer`, else `pstack-judgment`, default `claude/claude-opus-5-5` max
- read-only: say so in the prompt (no edits)

Build its prompt from `references/explainer-prompt.md` without the explorer-findings section. Go to Step 4.

## Step 3. Synthesize (complex questions only)

Once all explorers have returned, spawn one Paseo subagent to synthesize their findings into one explanation:

- profile: `pstack-how-explainer`, else `pstack-judgment`, default `claude/claude-opus-5-5` max
- read-only: say so in the prompt (no edits)

Build its prompt from `references/explainer-prompt.md` with every explorer's findings filled in.

## Step 4. Present

Present the explainer's output to the user. Light edits for clarity or context from the conversation are fine. Do not substantially rewrite it.

## Output Format

The explanation uses the sections defined in `references/explainer-prompt.md`, dropping any that do not apply: Overview, Key Concepts, How It Works, Where Things Live, Gotchas.
