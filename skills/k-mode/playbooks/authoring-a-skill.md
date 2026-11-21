### Authoring or modifying a skill

**You own the skill's voice.**

1. Read Pi's skill format before writing anything: frontmatter with `name` and `description`, then direct instructions, then the support files the skill needs. A project skill lives in `.pi/skills/<name>/SKILL.md` or `.agents/skills/<name>/SKILL.md`, and a user skill lives in `~/.pi/agent/skills/<name>/SKILL.md` or `~/.agents/skills/<name>/SKILL.md`. The description decides when the model loads the skill, so state what it does and when it applies.
2. Validate the skill: frontmatter has `name` and `description`, referenced files exist, cross-skill links resolve.
3. Test cases if structural. Skip if subjective.
4. Run **Opening a PR**.

When in doubt, delete. Keep only prose that changes a decision. Tell it to do the thing and skip the reason. Explain only when the rule is confusing without one. Match tone to scope. Point at structural sources (types, READMEs, config) per the **encode-lessons-in-structure** principle skill. Delegate to other skills by path. Don't restate. A workflow you keep hitting but isn't captured → propose a new skill.

**Reply:** summary of the skill, key design decisions, validation notes.
