# The pi adapter

kstack is written for [pi](https://pi.dev), the coding agent that provides the skills, the subagents, and the model block its skills read. This page is the contract between the two. Read it once when you change a kstack skill, and again when you port kstack to another harness.

## Install and discovery

kstack is a pi package. Pi discovers its resources from the manifest in `package.json`:

```json
{
  "pi": {
    "skills": ["./skills"],
    "prompts": ["./prompts/*.md"],
    "subagents": { "agents": ["./agents"] }
  }
}
```

| Resource | Where it lands in a session |
|---|---|
| `skills/<name>/SKILL.md` | The skill catalog, loaded by name with `/skill:<name>` or by the model when a task matches its description. |
| `prompts/*.md` | Slash commands, here `/k-mode` and `/k-help`. |
| `agents/*.md` | Subagent definitions, spawned with `subagent({ agent: "<name>" })`. |

`pi list` shows the installed packages. `pi -e ./kstack` tries a local checkout for one run. `/reload` picks up edits in an open session, and `/skill:k-mode` forces a load when the model missed the description.

## Names

| kstack name | pi form |
|---|---|
| run a task in k mode | `/k-mode <task>`, or `/skill:k-mode` |
| ask about kstack | `/k-help <question>` |
| any other skill | `/skill:<name>` |
| the delegate agent | `subagent({ agent: "k-agent", task: "..." })` |
| the comment reviewer | `subagent({ agent: "comment-sicko", task: "..." })` |
| the model block | `~/.pi/agent/AGENTS.md`, between the `kstack:models` markers |

## Skills, and where the model finds them

Bundled skills sit next to `k-mode`, so a k-mode skill refers to a sibling as `../<name>/SKILL.md`. Pi tells the model the absolute path of every skill it can see, and hides skills marked `disable-model-invocation: true` from that list, so those load only through `/skill:<name>`. `k-mode` and `k-help` stay visible because `/k-mode` and `/k-help` point at them by name.

Project skills belong in `.pi/skills/` or `.agents/skills/`. User skills belong in `~/.pi/agent/skills/` or `~/.agents/skills/`.

## Subagents

Pi spawns children with the `subagent` tool. The fields kstack uses:

| Field | Meaning in kstack |
|---|---|
| `agent` | `k-agent` for a delegate a playbook step spawns, or the agent a routed skill names. |
| `task` | One self-contained brief. The first line is always `Read <path>/worker.md before anything else.` |
| `async` | `true` for fan-out, so the children run while the session keeps working and the session is woken when they finish. |
| `model` | The role's line from the model block, as `provider/model` or `provider/model:level`. Omitted when the role runs on the session model. |
| `worktree` | `true` for any child that writes, so two children never share a checkout. |
| `tools` | A tool allowlist for a read-only child, such as `read, grep, find, ls`. |

Pi has no cloud agents. Isolation comes from a git worktree, and a long run keeps its liveness record in a mission.

## The model block

`skills/setup-kstack/SKILL.md` writes this block into `~/.pi/agent/AGENTS.md`, which pi loads into every session:

```markdown
<!-- kstack:models:start -->
# budget: large (xhigh for hard roles, high for the rest)
feature, refactoring: inherit-parent
...
<!-- kstack:models:end -->
```

Every skill that spawns children reads its role's line. A role with no line, or a line reading `inherit-parent` or `auto`, runs on the session model, so kstack needs no configuration at all to work. A pinned role carries a pi model id with an optional `:off`, `:minimal`, `:low`, `:medium`, `:high`, `:xhigh`, or `:max` suffix. `subagents.defaultThinking` in `~/.pi/agent/settings.json` carries the budget to the unpinned roles.

## Asking, and not asking

Pi's `ask_user` tool is the only way a kstack skill asks a question. `worker.md` forbids a delegate from using it at all, because a child cannot see the human's answer in a way the session can act on. A step that would block on a person becomes a choice you can undo, stated in the reply.

## Plans, missions, and the decision trail

Pi has no todo tool and no plan mode. k-mode writes the matched playbook's steps into its reply and, for long work, into `.audit/plan-<task-slug>.md`. Missions record why a long run exists, the runs it made, and its receipts. A decision trail is `decisions.tsv`, from the `show-me-your-work` skill.

## Transcripts

Pi stores sessions under `~/.pi/agent/sessions/`:

```text
~/.pi/agent/sessions/--<path with / replaced by ->--/<timestamp>_<session-id>.jsonl
```

One JSON object per line. A skill that reads history, such as `recall`, `automate-me`, `eval`, or `show-me-your-work`, reads only the directory for the active workspace and never globs across the others.

## Overnight and scheduled work

Pi has no loop command. An async subagent wakes the session when it finishes, which covers an event to watch. A recurring re-check uses a pi-subagents schedule:

```ts
subagent({ action: "schedule.create", id: "nightly-sweep", workflow: true, options: { every: "24h" } })
```

For a hard stop, bound the run with a timeout in the prompt and a check that can fail, not a duration that has to pass.

## Porting kstack to another harness

Read the table in Names and replace each pi form with the harness's own. Then check the four things a harness can break:

1. **Skill discovery.** If the harness cannot see `skills/`, every relative link in k-mode breaks. Point it at the folder instead, or install the skills into its own directory.
2. **Delegation.** A harness with no subagent tool turns every fan-out step into serial work, which changes the cost, not the intent.
3. **The model block.** A harness that cannot read `AGENTS.md` needs the block fed to it another way, or every role falls back to the session model.
4. **The plan.** Any place that names a todo tool, a plan file, or a decision log must name the harness's equivalent, or the step does nothing.
