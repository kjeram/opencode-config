## Development loop

Work follows an evidence-first loop, coordinated by `orchestrator-agent` (the
default primary agent). Each stage has one owner and hands off explicitly.

```
Requirement
  -> Understand & clarify ....... research-agent (read-only evidence gathering)
  -> Define acceptance criteria . spec-agent    -> openspec/changes/{name}/proposal.md + specs/**
  -> Design smallest solution ... planning-agent (smallest viable design + trade-offs)
  -> Plan / break into tasks .... planning-agent -> plans/{feature}/plan.md (commit-sized steps)
  -> [gate] Review the plan ..... reviewer-agent  -> Verdict: solid | needs changes | unsafe
  -> Implement .................. implementation-agent (exact plan) OR apply-agent (RED->GREEN)
  -> Test locally ............... implementation-agent + test-fixer-agent (minimal root-cause fixes)
  -> Review ..................... verifier-agent   -> Verdict: pass | needs fixes | rollback
  -> Done? -- no --> iterate ..... orchestrator routes back to planning/test-fixer
  -> Document / close out ....... summarize what changed and what was verified
```

Only implementation-class agents (`spec-agent`, `planning-agent`,
`implementation-agent`, `apply-agent`, `test-fixer-agent`) may edit files.
`orchestrator-agent`, `research-agent`, `reviewer-agent`, and `verifier-agent`
are read-only.

### Lanes (chosen by the orchestrator based on risk)

- **Fast**: research -> planning -> implementation. Review/verify only when behavior changes.
- **Standard**: research -> planning -> review -> implementation -> verify.
- **High-Risk**: research -> planning -> review -> approval gate -> implementation -> verify.
- **Spec-First**: spec -> approval gate -> planning -> review -> approval gate -> apply -> verify.

### Gate signals

- `reviewer-agent` blocks implementation on `needs changes` / `unsafe`.
- `verifier-agent` verdict is the `Done?` decision: `pass` closes out,
  `needs fixes` routes back narrowly, `rollback` halts and reports.

### Artifacts (shared state between agents)

- `openspec/changes/{name}/proposal.md` and `specs/**` — spec + acceptance criteria.
- `plans/{feature}/plan.md` — implementation-ready, commit-structured plan.

## Prefer built-in tools over shell commands

Always use the dedicated built-in tools instead of invoking equivalent
functionality through `bash` or `python`. The built-in tools are safer, faster,
and provide a better experience.

- File search: use the **Glob** tool, not `find`, `ls`, or shell globbing
- Content search: use the **Grep** tool, not `grep`, `rg`, `awk`, or `sed`
- Reading files: use the **Read** tool, not `cat`, `head`, or `tail`
- Editing files: use the **Edit** tool, not `sed`, `awk`, or `python` scripts
- Writing files: use the **Write** tool, not `echo >`, `cat <<EOF`, or `python`
- Task delegation and exploration: use the **Task** tool

Reserve `bash` for genuine system and terminal operations that have no built-in
equivalent — for example `git`, package managers, build tools, test runners, and
process management.

Reserve `python` for actual program execution or computation, never as a
substitute for file reading, editing, searching, or text manipulation that a
built-in tool already covers.
