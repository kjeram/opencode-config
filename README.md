# opencode-config

A configuration for [opencode](https://opencode.ai) that turns a single AI assistant
into a coordinated team of specialist agents. Each agent owns exactly one stage of an
evidence-first development loop — research, spec, plan, review, implement, test, and
verify — with explicit handoffs and approval gates between them.

The goal is simple: no code is written without evidence, no plan is executed without
review, and no work is closed out without verification.

## What this repository contains

```
.opencode/
  opencode.jsonc          # opencode config: default agent, LSP, permissions
  AGENTS.md               # canonical development loop, lanes, and tooling rules
  agents/                 # one Markdown file per agent (prompt + permissions)
  skills/                 # specialized skill packs loaded on demand
    golang-pro/
    python-pro/
    sql-pro/
AGENTS.md                 # top-level note pointing config into .opencode
```

Each file in `.opencode/agents/` defines one agent through YAML frontmatter (name,
mode, temperature, and per-tool permissions) followed by the agent's system prompt.

## Configuration

`.opencode/opencode.jsonc`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "default_agent": "orchestrator-agent",
  "lsp": true,
  "permission": {
    "lsp": "allow"
  }
}
```

- **`default_agent`**: `orchestrator-agent` — the primary agent every session starts with.
- **`lsp`**: language-server integration is enabled and allowed.

## The development loop

Work follows an evidence-first loop, coordinated by `orchestrator-agent`. Each stage
has one owner and hands off explicitly:

```
Requirement
  -> Understand & clarify ....... research-agent   (read-only evidence gathering)
  -> Define acceptance criteria . spec-agent       -> openspec/changes/{name}/proposal.md + specs/**
  -> Design smallest solution ... planning-agent    (smallest viable design + trade-offs)
  -> Plan / break into tasks .... planning-agent   -> plans/{feature}/plan.md (commit-sized steps)
  -> [gate] Review the plan ..... reviewer-agent    Verdict: solid | needs changes | unsafe
  -> Implement .................. implementation-agent (exact plan) OR apply-agent (RED->GREEN)
  -> Test locally ............... implementation-agent + test-fixer-agent (minimal root-cause fixes)
  -> Review ..................... verifier-agent     Verdict: pass | needs fixes | rollback
  -> Done? -- no --> iterate ..... orchestrator routes back to planning/test-fixer
  -> Document / close out ....... summarize what changed and what was verified
```

The canonical definition of this loop lives in [`.opencode/AGENTS.md`](.opencode/AGENTS.md).

### Read-only vs. editing agents

A hard boundary separates agents that can change files from agents that cannot:

- **May edit files**: `spec-agent`, `planning-agent`, `implementation-agent`,
  `apply-agent`, `test-fixer-agent`, `readme-architect`.
- **Read-only**: `orchestrator-agent`, `research-agent`, `reviewer-agent`,
  `verifier-agent`.

This keeps coordination, evidence gathering, and review structurally incapable of
mutating the repository — the review gates cannot be shortcut.

## Lanes

The orchestrator selects a lane by scope and risk. Higher risk adds review, verification,
and explicit approval gates:

| Lane | Flow |
|------|------|
| **Fast** | research → planning → implementation (review/verify only when behavior changes) |
| **Standard** | research → planning → review → implementation → verify |
| **High-Risk** | research → planning → review → **approval gate** → implementation → verify |
| **Spec-First** | spec → **approval gate** → planning → review → **approval gate** → apply → verify |

### Gate signals

- **`reviewer-agent`** blocks implementation on a `needs changes` or `unsafe` verdict.
- **`verifier-agent`** produces the `Done?` decision: `pass` closes out, `needs fixes`
  routes back narrowly, `rollback` halts and reports.

### Shared artifacts

Agents pass state to each other through files on disk rather than conversation alone:

- `openspec/changes/{name}/proposal.md` and `specs/**` — spec and acceptance criteria (owned by `spec-agent`).
- `plans/{feature}/plan.md` — the implementation-ready, commit-structured plan (owned by `planning-agent`).

## The agents

### orchestrator-agent — the coordinator

- **Mode:** primary (the session default) · **Temperature:** 0.2 · **Edits files:** no
- **Delegates to:** every specialist agent (via the `task` tool)

Routes each request to the right specialist and preserves phase boundaries. It classifies
the request, picks a lane by scope and risk, and moves work through
evidence → plan → review → approved execution → verification, checking that each phase
produced enough signal before advancing. It never implements code itself, never assigns
work outside a specialist's role, and never lets implementation start before the plan is
explicit enough to execute. If verification returns `rollback`, it stops and reports
instead of routing more work.

### research-agent — evidence gathering

- **Mode:** all · **Temperature:** 0.1 · **Edits files:** no
- **Tools:** read, web search/fetch, and read-only git (`git log`, `git ls-files`, `git show`)

Gathers concise, evidence-based repository and tooling context so another agent can decide
what command to run, which files matter, or what pattern applies. It operates in one of
three modes — **Test Context Discovery**, **Failure-Specific Research**, or
**Feature/Implementation Research** — and returns findings tied to concrete evidence
(exact paths, line ranges, commands, URLs). It never invents facts, never writes plans or
code, and stops at roughly 80% confidence, marking anything unresolved under
`Open Questions`. This is the read-only front end that feeds `spec-agent`,
`planning-agent`, `test-fixer-agent`, and `readme-architect`.

### spec-agent — spec-first artifacts

- **Mode:** all · **Temperature:** 0.1 · **Edits files:** yes (spec artifacts only)
- **Delegates to:** `research-agent`

Turns feature descriptions or change requests into structured, spec-first artifacts:
`proposal.md` (what and why) and one capability spec per capability, each with testable,
observable acceptance criteria. It writes to `openspec/changes/{change-name}/` and appends
new domain terms to `GLOSSARY.md`. It writes **no** implementation code, plans, or design
decisions — those belong downstream — and presents its artifacts for approval before
anything else happens. This is the entry point for the **Spec-First** lane.

### planning-agent — the smallest viable plan

- **Mode:** all · **Temperature:** 0.2 · **Edits files:** yes (plan files)
- **Delegates to:** `research-agent`

Transforms a request (plus research and any spec) into an implementation-ready plan sized
for a single pull request, where each step is a meaningful, reviewable, testable unit
corresponding to one commit. It designs the smallest solution that satisfies the
requirement, makes trade-offs explicit, and avoids speculative abstraction or new
dependencies. Genuinely blocking unknowns are marked `[NEEDS CLARIFICATION]` and must be
resolved before the final plan is saved to `plans/{feature-name}/plan.md`. It does not
write production code.

### reviewer-agent — adversarial plan review

- **Mode:** all · **Temperature:** 0.1 · **Edits files:** no
- **Tools:** read and read-only git (`git diff`, `git log`, `git show`, `git status`)

Stress-tests a plan *before* any code is written. It checks pattern fit, scope discipline,
and reuse; simulates realistic failures (null/undefined inputs, empty states, partial
updates, race conditions, downstream breakage); and rewrites ambiguous instructions into
explicit, deterministic, testable steps. Every finding carries evidence, and weak or
speculative objections are removed. Output begins with a verdict —
`solid | needs changes | unsafe` — that gates whether implementation may begin.

### implementation-agent — exact plan execution

- **Mode:** all · **Temperature:** 0.1 · **Edits files:** yes
- **Tools:** read, edit, bash, question

Executes an approved plan exactly as written, producing production-ready code. It does not
skip, merge, redesign, or reinterpret steps, and does not introduce new tools, libraries,
or patterns unless the plan requires them — no TODOs or placeholders. Git access is
**read-only**: it never commits, pushes, amends, or rebases; if the plan asks for a git
write it stops and asks. It runs targeted tests at checkpoints and hands off to
`test-fixer-agent` when a failure is unclear, non-local, repeated, or out of plan scope.

### apply-agent — RED→GREEN playbook execution

- **Mode:** all · **Temperature:** 0.1 · **Edits files:** yes
- **Tools:** read, edit, bash, question

Executes an implementation playbook (`implementation.md`) with strict test-driven
discipline: write a failing test (**RED**), confirm the failure is a real assertion
failure, write the minimal code to pass (**GREEN**), verify it passes, then move on. It
follows steps in exact order with zero improvisation and never designs architecture. Every
git commit requires explicit user approval, and destructive commands are forbidden. This
is the execution agent for the **Spec-First** lane.

### test-fixer-agent — minimal root-cause test repair

- **Mode:** all · **Temperature:** 0.1 · **Edits files:** yes
- **Delegates to:** `research-agent`

Makes a failing test suite pass through correct, minimal, root-cause fixes without
sacrificing intended behavior. It runs the narrowest useful test first and broadens only
after it is green; classifies each failure (product bug, test bug, stale mock, async/timing,
snapshot, selector, environment, dependency, flaky, or known exception); and never deletes,
skips, weakens, or blindly re-snapshots tests. When behavior changed intentionally it
updates the test; when behavior should stay the same it fixes the product code; when intent
is unclear it stops and asks. It reassesses after two failed attempts on the same failure.

### verifier-agent — post-implementation audit

- **Mode:** all · **Temperature:** 0.1 · **Edits files:** no
- **Tools:** read and read-only git (`git diff`, `git log`, `git show`, `git status`)

Audits the finished implementation against the approved plan, assuming it is incorrect,
incomplete, or unsafe until proven otherwise. It checks plan compliance, behavioral
consistency beyond happy paths, realistic edge cases, regression risk, and test coverage —
every defect backed by concrete evidence (file path, diff hunk, plan step, or command
output). Its verdict is the loop's exit gate: `pass`, `needs fixes`, or `rollback`. It never
claims tests passed without evidence and reports missing validation explicitly.

### readme-architect — documentation

- **Mode:** all · **Temperature:** 0.2 · **Edits files:** yes (docs and docstrings only)
- **Delegates to:** `research-agent`

Creates and maintains `README.md` files and source-code docstrings (Python docstrings,
JSDoc/TSDoc, Go doc comments), adapting depth and structure to project type and audience.
When editing source files it touches **only** documentation comments — never logic,
control flow, signatures, or imports. It invents no commands, URLs, or credentials, using
clear placeholders for anything unconfirmed. This agent sits outside the core loop and is
invoked directly for documentation work.

### Disabled built-ins

`plan.md` and `build.md` set `disable: true`, turning off opencode's default `plan` and
`build` agents so this configuration's orchestrated pipeline is used instead.

## Skills

`.opencode/skills/` holds specialized skill packs that agents load on demand via the
`skill` tool when a task matches the skill's description, rather than carrying that
guidance in every prompt:

- **golang-pro** — idiomatic, concurrent, high-performance Go.
- **python-pro** — type-safe, production-ready, modern async Python.
- **sql-pro** — query optimization and schema design across PostgreSQL, MySQL, SQL Server, and Oracle.

## Tooling conventions

All agents prefer opencode's built-in tools over shell equivalents (see
[`.opencode/AGENTS.md`](.opencode/AGENTS.md)):

- File search → **Glob** (not `find`/`ls`)
- Content search → **Grep** (not `grep`/`rg`/`awk`/`sed`)
- Reading files → **Read** (not `cat`/`head`/`tail`)
- Editing files → **Edit** (not `sed`/`awk`)
- Writing files → **Write** (not `echo >`/`cat <<EOF`)
- Delegation and exploration → **Task**

`bash` is reserved for genuine system operations (git, package managers, build tools, test
runners); `python` is reserved for actual computation, never as a substitute for the
file/search/edit tools.

## Usage

This repository is an opencode configuration, not a standalone application. To use it,
open a project with opencode configured against these agents. Sessions start on
`orchestrator-agent`, which classifies the request, picks a lane, and delegates through
the loop above. You interact primarily with the orchestrator; it routes to specialists and
surfaces approval gates and verification verdicts back to you.

## Extending the configuration

- **Add an agent:** create `.opencode/agents/{name}.md` with YAML frontmatter (`name`,
  `mode`, `temperature`, `permission`) and a system prompt, then grant the orchestrator
  `task` permission for it if it should be part of the loop.
- **Add a skill:** create `.opencode/skills/{name}/SKILL.md` with a clear description so
  the right agent loads it on demand.
- **Change routing or lanes:** edit [`.opencode/AGENTS.md`](.opencode/AGENTS.md), the
  canonical source for the loop, lanes, and gate signals.
