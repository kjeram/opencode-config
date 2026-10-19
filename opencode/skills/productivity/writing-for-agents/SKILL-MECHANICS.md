# Skill mechanics

The skill-specific branch of [`writing-for-agents`](SKILL.md): what changes when the document is a skill (frontmatter, the invocation choice, and router skills). Everything else about writing it is the universal reference in `SKILL.md`.

## Invocation

Two choices, trading the two loads:

- A **model-invoked** skill keeps a `description`, so the agent can discover and load it autonomously. The user can also load it explicitly with `@skill-id`. The description is its top-level context pointer, available to the model for discovery; a model-invoked skill whose content is all reference can also be shared reference that other skills load. Mechanics: write a model-facing description carrying the trigger branches (the pointer-writing rules in `SKILL.md` apply in full).
- A **user-invoked** skill sets `disable-model-invocation: true`. OpenCode then omits it from the model's available skill list while keeping it explicitly loadable by ID, including through a user `@skill-id` mention. This prevents automatic discovery; it is not an access-control boundary. Keep its `description` as a human-facing one-line summary with trigger lists stripped.

Pick model-invocation only when the agent must reach the skill on its own, or another skill must. If it only ever fires by hand, make it user-invoked and pay no context load.

For shared reference that should be automatically discoverable by multiple skills, use a model-invoked skill. A user-invoked skill is hidden from automatic discovery, though another skill can still load it explicitly if its ID is named.

## Splitting by invocation

The invocation cut of splitting (the sequence cut lives in `SKILL.md`): split off a model-invoked skill when you have a distinct leading word that should trigger it on its own (a trigger word you actually use in your prompts), or another skill must reach it. You pay context load for the new always-loaded description, so that independent reach has to be worth it.

## Router skills

When user-invoked skills multiply past what you can remember, that piled-up cognitive load is cured by a **router skill**: one user-invoked skill that names the others and when to reach for each, so the human has one skill to remember instead of many. It can only hint, never fire them: user-invoked skills have no description, so nothing but the human can reach them.
