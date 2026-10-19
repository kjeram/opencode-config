# OpenCode workflows

The `opencode` directory contains the OpenCode configuration and a toolkit of engineering and productivity workflows. The built-in `build` agent is the primary agent; the custom agents below are focused subagents it can delegate to.

## How the pieces fit

- **Commands** are explicit `/slash-command` entry points. They run in the current `build` session and load a skill with the same name.
- **Skills** are reusable procedures and reference material. Use them directly as `@skill-name` when appropriate.
- **Agents** are focused subagents for delegated work. They are not slash commands; the main agent starts them when a workflow calls for their expertise.

The workflow guide below is organized by the job you are trying to do, not by file type. The inventories link to the source definitions for details.

## Workflows at a glance

| Need | Start here | What it does |
| --- | --- | --- |
| Find the right process | [`/ask`](commands/ask.md) | Routes a situation to a suitable workflow in this toolkit. |
| Clarify an idea or decision | [`/grill-me`](commands/grill-me.md), [`/grill-with-docs`](commands/grill-with-docs.md), or [`/to-questionnaire`](commands/to-questionnaire.md) | Interview to clarify a plan; optionally capture agreed glossary terms and ADRs, or prepare questions for an expert. |
| Plan a large, uncertain effort | [`/wayfinder`](commands/wayfinder.md) | Build and resolve a shared map of decisions before committing to execution. |
| Prepare a repo or incoming work | [`/setup-skills`](commands/setup-skills.md), then [`/triage`](commands/triage.md) | Configure tracker/domain conventions, then move issues or external PRs through triage. |
| Investigate a problem or design | [`/improve-codebase-architecture`](commands/improve-codebase-architecture.md), or use `@diagnosing-bugs`, `@research`, or `@prototype` | Explore architecture, establish a bug feedback loop, research primary sources, or build a throwaway prototype. |
| Turn intent into actionable work | [`/to-spec`](commands/to-spec.md) → [`/to-tickets`](commands/to-tickets.md) | Publish a spec, then break it into dependency-linked, independently verifiable tickets. |
| Build a ticket or complete spec | [`/implement`](commands/implement.md) or [`/implement-spec`](commands/implement-spec.md) | Implement one piece of work, or coordinate a ticket graph across isolated worktrees. |
| Check and communicate a change | [`/code-review`](commands/code-review.md), then `@pr` as needed | Review standards and spec fit independently; shape a concise PR body with evidence and merge risk. |
| Learn, improve, or hand off | [`/teach`](commands/teach.md), [`/retro`](commands/retro.md), or [`/handoff`](commands/handoff.md) | Learn across sessions, improve the agent environment after a session, or pass context to another agent. |

## Distinct workflows

### 1. Orient, clarify, and decide

- [`/ask`](commands/ask.md) / `ask`: route a situation to a fitting skill or workflow.
- [`/grill-me`](commands/grill-me.md) / `grill-me`: sharpen a plan or idea with one question at a time; does not create project documents.
- [`/grill-with-docs`](commands/grill-with-docs.md) / `grill-with-docs`: the same conversational decision process, while capturing agreed terminology in a glossary and offering ADRs for durable decisions.
- [`/to-questionnaire`](commands/to-questionnaire.md) / `to-questionnaire`: create a questionnaire for a person who has information the user needs.
- [`/wayfinder`](commands/wayfinder.md) / `wayfinder`: map a large, unclear effort as decision tickets and resolve them until the route to a defined destination is clear. This is planning, not implementation by default.

### 2. Configure a project and handle incoming work

- [`/setup-skills`](commands/setup-skills.md) / `setup-skills`: configure the issue tracker, triage labels, and domain-document conventions expected by the engineering workflows. Run it before first use of those workflows in a repo.
- [`/triage`](commands/triage.md) / `triage`: categorize and progress issues or supported external PRs, gather missing information, and prepare agent-ready briefs.
- `wizard`: generate a guided shell script for manual human steps such as provisioning, credential setup, or a one-off migration. Use when the agent cannot perform the steps itself.

### 3. Investigate, design, and diagnose

- [`/improve-codebase-architecture`](commands/improve-codebase-architecture.md) / `improve-codebase-architecture`: scan for evidence-backed opportunities to deepen modules, produce a visual report, and discuss a selected opportunity. It delegates read-only exploration to `architecture-scout`.
- `diagnosing-bugs`: build a tight feedback loop for hard bugs or performance regressions, then use experiments to isolate the cause.
- `research`: investigate from primary sources and save cited findings in the repo; delegates the reading and write-up to `researcher`.
- `prototype`: make throwaway code to answer a logic/state-model or UI design question before committing to a production implementation.
- `codebase-design`: shared reference vocabulary for module interfaces, depth, seams, adapters, leverage, and locality. Consult it when designing or deepening a module.
- `suggest`: request evidence-backed solution options with fit, benefits, and trade-offs; this is a subagent rather than a skill or command.

### 4. Turn intent into a spec and ticket graph

1. [`/to-spec`](commands/to-spec.md) / `to-spec` synthesizes the current context into a spec and publishes it to the configured tracker. It checks the proposed test seams with the user.
2. [`/to-tickets`](commands/to-tickets.md) / `to-tickets` breaks a spec or plan into tracer-bullet tickets with blocking relationships.
3. For an effort too large or uncertain to specify yet, use [`/wayfinder`](commands/wayfinder.md) first to resolve the decisions that make a useful spec possible.

These workflows expect tracker configuration from `setup-skills`.

### 5. Implement work

- [`/implement`](commands/implement.md) / `implement`: implement a scoped piece of work from a spec or ticket in the current working context. It uses test-driven development, checks the work, runs a code review, and commits the result.
- [`/implement-spec`](commands/implement-spec.md) / `implement-spec`: implement an entire spec by coordinating its ticket dependency graph. Independent tickets use separate worktrees and `implementer` subagents; completed work is integrated by `merger`. If worktrees are unavailable, proceed serially rather than sharing a checkout.
- `tdd`: reusable red-green-refactor guidance for a behavior change; also the implementation discipline used by the implementation workflows.
- `implementer`: implements one scoped ticket in an isolated worktree using TDD.
- `merger`: integrates a completed ticket branch into the integration branch.

### 6. Review and prepare a change for sharing

- [`/code-review`](commands/code-review.md) / `code-review`: review changes from a fixed point on two independent axes: repository standards and fidelity to the originating spec. It uses `standards-reviewer` and `spec-reviewer` in parallel.
- `standards-reviewer`: read-only review for documented standards violations and code smells.
- `spec-reviewer`: read-only review for whether the change meets its spec; reports when no spec is available.
- `pr`: guidance for writing a PR body with a compact change summary, before/after evidence, and an explicit merge-danger assessment.

### 7. Learn, reflect, and transfer context

- [`/teach`](commands/teach.md) / `teach`: create a workspace for learning a topic over multiple sessions.
- [`/retro`](commands/retro.md) / `retro`: review a coding session and suggest improvements to the agent environment, such as navigation, checks, standards, or tooling—not changes to the product code.
- [`/handoff`](commands/handoff.md) / `handoff`: write a concise handoff document for another session or agent, saved outside the workspace.
- [`/wait-what`](commands/wait-what.md) / `wait-what`: re-explain a message that did not land, using simpler language and project terminology where available.
- `writing-for-agents`: reference for writing skills and other documents consumed by agents.

## Complete command list

All 17 commands run in the current `build` session and explicitly load the matching skill.

| Command | Purpose |
| --- | --- |
| [`/ask`](commands/ask.md) | Route to a workflow. |
| [`/code-review`](commands/code-review.md) | Review a diff against standards and its spec. |
| [`/grill-me`](commands/grill-me.md) | Clarify a plan without creating documents. |
| [`/grill-with-docs`](commands/grill-with-docs.md) | Clarify a plan and record agreed domain docs. |
| [`/handoff`](commands/handoff.md) | Create a context handoff. |
| [`/implement`](commands/implement.md) | Implement scoped work. |
| [`/implement-spec`](commands/implement-spec.md) | Coordinate implementation of a ticket graph. |
| [`/improve-codebase-architecture`](commands/improve-codebase-architecture.md) | Find and discuss architectural deepening opportunities. |
| [`/retro`](commands/retro.md) | Retrospect on a session and improve the agent environment. |
| [`/setup-skills`](commands/setup-skills.md) | Configure repo conventions for the engineering workflows. |
| [`/teach`](commands/teach.md) | Teach a topic across sessions. |
| [`/to-questionnaire`](commands/to-questionnaire.md) | Prepare an expert questionnaire. |
| [`/to-spec`](commands/to-spec.md) | Create and publish a spec. |
| [`/to-tickets`](commands/to-tickets.md) | Create dependency-linked tickets. |
| [`/triage`](commands/triage.md) | Triage an issue or supported external PR. |
| [`/wait-what`](commands/wait-what.md) | Re-explain something more clearly. |
| [`/wayfinder`](commands/wayfinder.md) | Map and resolve decisions for a large effort. |

## Complete skill list

The 25 skills live under [`skills/`](skills/). Some are normally invoked explicitly by a command; others are reusable references or procedures that can be selected by their descriptions.

| Workflow area | Skills |
| --- | --- |
| Clarify and plan | [`ask`](skills/ask/SKILL.md), [`grill-me`](skills/grill-me/SKILL.md), [`grill-with-docs`](skills/grill-with-docs/SKILL.md), [`to-questionnaire`](skills/to-questionnaire/SKILL.md), [`wayfinder`](skills/wayfinder/SKILL.md) |
| Configure and operate | [`setup-skills`](skills/setup-skills/SKILL.md), [`triage`](skills/triage/SKILL.md), [`wizard`](skills/wizard/SKILL.md), [`writing-for-agents`](skills/writing-for-agents/SKILL.md) |
| Investigate and design | [`codebase-design`](skills/codebase-design/SKILL.md), [`diagnosing-bugs`](skills/diagnosing-bugs/SKILL.md), [`improve-codebase-architecture`](skills/improve-codebase-architecture/SKILL.md), [`prototype`](skills/prototype/SKILL.md), [`research`](skills/research/SKILL.md) |
| Specify and implement | [`to-spec`](skills/to-spec/SKILL.md), [`to-tickets`](skills/to-tickets/SKILL.md), [`implement`](skills/implement/SKILL.md), [`implement-spec`](skills/implement-spec/SKILL.md), [`tdd`](skills/tdd/SKILL.md) |
| Review and share | [`code-review`](skills/code-review/SKILL.md), [`pr`](skills/pr/SKILL.md) |
| Learn and close the loop | [`handoff`](skills/handoff/SKILL.md), [`retro`](skills/retro/SKILL.md), [`teach`](skills/teach/SKILL.md), [`wait-what`](skills/wait-what/SKILL.md) |

## Complete subagent list

The seven focused agents live under [`agents/`](agents/). The primary `build` agent delegates to them when a workflow benefits from isolated context or specialized review.

| Agent | Role |
| --- | --- |
| [`architecture-scout`](agents/architecture-scout.md) | Read-only codebase exploration for evidence-backed architectural opportunities. |
| [`implementer`](agents/implementer.md) | Implements one scoped ticket in an isolated worktree using TDD. |
| [`merger`](agents/merger.md) | Integrates a completed ticket branch into its integration branch. |
| [`researcher`](agents/researcher.md) | Researches primary sources and produces cited findings. |
| [`spec-reviewer`](agents/spec-reviewer.md) | Checks a supplied diff against its originating spec. |
| [`standards-reviewer`](agents/standards-reviewer.md) | Checks a supplied diff against documented standards. |
| [`suggest`](agents/suggest.md) | Recommends evidence-grounded solution approaches and trade-offs. |
