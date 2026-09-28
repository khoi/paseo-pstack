### Feature

**You own the design and implementation. Plan, review, verify.**

1. `how` over the affected subsystem.
2. `architect` for parallel design exploration.
3. Implement directly with a specific scope (file paths, named data shape and its organizing structure per **principle-model-the-domain**, a state machine over scattered booleans, a table/registry over branching, a typed model over repeated shape assumptions, chosen before writing logic, and success criteria). Comments per **Comments**. Surgical edits, re-ground against the source for upstream-derived files. Port shared-primitive improvements to all consumers and verify each. Commit liberally.
4. Verify on the matching surface. "Inconclusive" or wrong-surface is not a pass. Flag it.
5. Rebase into small, ordered commits. Stack follow-ups.
   Use the **sequence-verifiable-units** principle skill, building, verifying, and committing each small unit before the next.
6. If the design is contested, `interrogate` before shipping.
7. Run **Opening a PR**.

**Reply:** what you built, what you chose and why, open decisions. Tables for design alternatives.
