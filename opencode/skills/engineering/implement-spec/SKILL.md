---
name: implement-spec
description: "Implement the result of /to-spec and /to-tickets in code."
disable-model-invocation: true
---

You have been provided a spec. This spec should have tickets associated with it, describing how to implement the spec.

The issue tracker should have been provided to you. If not, tell the user to run `/setup-skills`.

The goal is the entire spec implemented on a single **integration branch**, with every ticket resolved the way the issue tracker closes work.

The tickets are not a list of steps. They are a **task graph** with blocking relationships between them. This means there is always a **frontier** of tickets which are ready to be grabbed.

Communication to and from subagents should be sparse. Communicate primarily through **context pointers**: to the spec, tickets, research notes, and previous commits. Don't duplicate information already available via pointers.

**Implementer subagents** should be run in the background where possible for maximum concurrency.

## Steps

1. Read the spec and tickets to understand the task graph.

2. (optional) Use the `architecture-scout` subagent to conduct exploration required by the tickets: inspect relevant code or external documentation and save concise notes outside the repository, accessible to later subagents. This lets implementers focus on implementation.

3. Create the integration branch. If the issue tracker closes work through PRs, or the user asks for one, open a draft PR after the first merge in step 5 (a branch with no commits ahead of main can't open one), marked as closing the spec and tickets.

4. Use the `implementer` subagent for each independent ready ticket, in its own OpenCode worktree and branch. Do not run parallel implementers in the same checkout. If separate worktrees are unavailable, work serially. Each implementer:
   - confirms its worktree is based on the integration branch before starting, and resets onto it if not;
   - loads skill ID `tdd` with the skill tool to build the ticket;
   - merges the integration branch tip into its own branch before reporting done

5. Once an implementer completes, merge its work to the integration branch with the `merger` subagent.

6. If this changes the **frontier** of available tickets, kick off more **implementer subagents** to work on the new tickets. This allows for maximum concurrency.

7. Once all tickets are complete, load skill ID `code-review` with the skill tool on the integration branch. Fix review findings with a single `implementer` subagent.

8. If a draft PR exists, mark it ready for review. Otherwise, resolve each ticket the way the issue tracker closes work, and report the integration branch.

9. Clean up all **implementer subagent** worktrees.
