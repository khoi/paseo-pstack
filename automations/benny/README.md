# benny

benny gives you two paseo schedules for slack issue reports. one triages each report. the other reproduces confirmed bugs and may prepare a small draft fix.

the files in this directory are dormant setup and automation sources. they do not appear as slash skills.

## set it up

1. point a paseo agent at [`FOR_AGENTS.md`](./FOR_AGENTS.md) and name the target repository.
2. let setup merge this whole directory into the target at `.paseo/automations/benny/`. it must preserve destination-only files and review conflicts instead of overwriting local edits.
3. let setup install pstack into the target repository's `.agents/skills/` for shared dependencies:

```bash
npx skills add khoi/paseo-pstack
```

4. keep user-owned configuration outside the copied pack, for example in `.paseo/benny/`. adapt [`configuration.example.yaml`](./templates/configuration.example.yaml) and [`feature-map.example.md`](./skills/reproduce-and-fix-issues/references/feature-map.example.md).
5. commit `.agents/skills/`, `.paseo/automations/benny/`, and any secret-free configuration before enabling either schedule.
6. review each new schedule draft before setup creates it, or approve in-place updates to existing schedules. then send a harmless test report and verify every source-channel post stays in the original thread.
