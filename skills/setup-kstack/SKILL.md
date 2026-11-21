---
name: setup-kstack
description: Configure which model kstack runs each role on and at what thinking level. Detects the models this pi can reach and writes a kstack section into ~/.pi/agent/AGENTS.md. Use for /skill:setup-kstack, "configure kstack models", "kstack budget", or changing kstack's model choices.
---

# Setup kstack

Write a kstack section into `~/.pi/agent/AGENTS.md`, the file pi loads into every session. That section maps each role kstack runs to a model and a thinking level.

## Steps

### 1. Detect available models

Run `pi --list-models`. Its rows are `provider`, `model`, and whether the model takes a thinking level. Inside a session, `subagent({ action: "models" })` lists the registry ids the launch path accepts. Use both, and prefer the registry ids when the two disagree, because those are the ids a spawn resolves.

Never write a model this detection has not confirmed. If neither source works, ask the user to paste their `pi --list-models` output. `inherit-parent` and `auto` are always valid, though neither is a model id.

### 2. Load current state

Read `~/.pi/agent/AGENTS.md` if it exists. kstack owns only the block between `<!-- kstack:models:start -->` and `<!-- kstack:models:end -->`, and everything outside those markers belongs to the user and never changes. Inside the block, the `# budget` line and the role lines are the current choices. A line whose role is not one of the eight in step 5, such as `hardest tasks`, `architect runners`, or `why investigators`, comes from a retired role. Drop it.

A block without the `# defaults: 4` line came from an older setup that pinned real models, so treat its values as defaults rather than choices and offer the step 3 options over them. Tell the user which lines you dropped or replaced.

### 3. Ask for a budget

Use `ask_user`, with these labels, naming the current budget when the block records one. With no block, say that `large` matches the skill defaults.

- `large — xhigh for hard roles, high for the rest`
- `medium — high for hard roles, medium for the rest`
- `small — medium for hard roles, low for the rest`

The budget sets the thinking level of every pinned model and, if the user agrees in step 6, the level of unpinned roles too. The ladder, least to most, is `off`, `minimal`, `low`, `medium`, `high`, `xhigh`, `max`. Never write `max` for a role.

### 4. Ask which roles to pin

Every role runs on the session model by default, which is why kstack works the same on any provider. Pinning buys two things: a cheaper or faster model on mechanical work, and a second provider on a panel.

Ask with `ask_user`, multiple answers allowed:

- Nothing, keep every role on the session model (fastest setup)
- Code roles (`feature, refactoring`, `bug-fix`, `perf-issue`, `hillclimb`)
- `judgment and prose`
- `mechanical edits` and `verification`
- `swarm workers`
- The panels (`arena runners`, `arena cross-judge pool`, `interrogate reviewers`)

For each role the user picks, offer the detected models that take the role's thinking level, grouped by provider, plus `inherit-parent`. A panel role takes a comma-separated list, one seat per entry, and one subagent runs per seat. For a panel, offer one model per provider the user has. A panel whose seats all share one provider still runs, and its cross-judge is skipped with a note, because a judge from the parent's own provider adds nothing.

### 5. Show the roles and confirm

Show every role, its model, and its thinking level, marking any id that detection did not confirm as needing a choice. Show each line you dropped or replaced. Ask whether to accept as-is or change specific roles.

The eight roles:

| role | runs |
|---|---|
| `feature, refactoring` | the feature and refactoring playbooks |
| `bug-fix` | the bug fix playbook |
| `perf-issue` | the perf and runtime forensics playbooks |
| `hillclimb` | the hillclimb playbook |
| `judgment and prose` | replies, docs, review, and any task that turns on taste |
| `mechanical edits` | renames, formatting, and other low-judgment edits |
| `verification` | any subagent whose job is to prove a change works |
| `swarm workers` | `/skill:swarm` workers and the default for other fan-out |
| `arena runners` | `/skill:arena` candidates, one request per entry |
| `arena cross-judge pool` | `/skill:arena` judge candidates, tried in order |
| `interrogate reviewers` | `/skill:interrogate` adversaries, one request per entry |

### 6. Ask about unpinned roles

If any role keeps the session model, offer once to write `subagents.defaultThinking` in `~/.pi/agent/settings.json` with the budget's level. That is what carries the budget to those roles. Ask before editing the file, keep every other key untouched, and skip the write when the key already holds the same value.

### 7. Validate

Every model written must be a detected id at or below `max`, with a `:level` suffix the model supports. `inherit-parent` and `auto` always pass. If a chosen id is not available, stop and ask again rather than guessing a near match.

### 8. Write

Write the block into `~/.pi/agent/AGENTS.md`, creating the file if it is missing and appending the block at the end if the markers are absent. Preserve every byte outside the markers, and overwrite the block whole so a re-run is idempotent. Shape:

```markdown
<!-- kstack:models:start -->
# kstack model configuration. One line per role. Delete a line to run that role on the session model.
# `inherit-parent` or `auto` means the session model. Alias entries in a panel list still count toward its fan-out.
# defaults: 4
# budget: large (xhigh for hard roles, high for the rest)
feature, refactoring: inherit-parent
bug-fix: inherit-parent
perf-issue: inherit-parent
hillclimb: inherit-parent
judgment and prose: inherit-parent
mechanical edits: inherit-parent
verification: inherit-parent
swarm workers: inherit-parent
arena runners: inherit-parent, inherit-parent, inherit-parent
arena cross-judge pool: inherit-parent
interrogate reviewers: inherit-parent, inherit-parent
<!-- kstack:models:end -->
```

A role pinned to a real model reads `provider/model` or `provider/model:level`. Keep the labels exactly as written, because the skills parse them.

### 9. Confirm

Tell the user the block was written, that it applies to new sessions, and to run `/reload` in an open session. Re-running this skill updates it. A skill that cannot find the block runs every role on the session model, so a missing block is a working default, not an error.

### 10. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with `/skill:create-verification-skill`." On yes, load that skill. On no, move on without pushing.

### 11. Offer a formal verification skill (optional)

Check whether the project already has a formal verification skill (a project skill that runs a model checker, a proof, or another verifier against the code). If not, offer once: "want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with `/skill:create-formal-verification-skill`." On yes, load that skill. On no, move on without pushing.
