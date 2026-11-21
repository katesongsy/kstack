---
name: k-help
description: Guides users through kstack setup, k-mode, and picking the skill, playbook, or principle for a task. Use for /k-help or "which kstack skill should I use".
---

# K help

Answer the user's question about kstack, hand them a prompt they can send, and link the file the answer came from. For a help question, don't start the work. The user asked how, and a kstack run spends real tokens, so let them send the prompt.

A message that asks for work, such as "use kstack to fix this bug", is not a help question. Read [`k-mode`](../k-mode/SKILL.md), do the work under it, and mention once that `/k-mode` starts the next task.

This file maps questions to the skills and guide pages that hold the answers. Those files own the details. Read the file you route to before you quote it, and trust it when it disagrees with this map. Links here are relative to the installed package, so name the file by path when the user cannot open one.

## Command names

Pi loads a skill with `/skill:<name>`, for example `/skill:arena`. Two entry points also ship as prompt templates, so they need no prefix: `/k-mode <task>` and `/k-help <question>`. Pi also loads a skill on its own when a task matches its description, which is how `k-mode` reaches the skills it routes to.

## Find out what they need

Infer the need from the message and the conversation. A named situation, such as "which skill reviews a PR?", goes straight to its section. If the need is still unclear, ask one multiple-choice question with `ask_user`, then answer only the section they pick:

- Get set up
- Start a task with `/k-mode`
- Pick a skill for a situation
- Fix a run that went wrong
- Make kstack your own

Check the state that changes the answer, and mention it only when it does:

- No kstack section in `~/.pi/agent/AGENTS.md` means `/skill:setup-kstack` hasn't run for this user, so every role uses the session model.
- No `verify-*` skill or other app harness in the project means agents have no scripted way to drive the app. Mention `create-verification-skill` when the question is about proving a change works.
- No formal verification skill in the project means no property is checked past what tests reach. Mention `create-formal-verification-skill` when the question is about a race, an invariant, or another case tests rarely hit. When one exists and the code it covers changed after the commit it records as last in sync, mention `maintain-formal-verification-skill` if the question touches that code.

When the model block is missing and it matters, ask whether the user wants to pick a model for each role and a thinking budget now. It matters when the user is new, the question is about setup or cost, or the answer depends on which models run. Ask at most once per chat. If the need is also unclear, ask both questions together. Offer two choices:

- Now: give them `/skill:setup-kstack` to type, and answer their question too.
- Later: answer their question, and add one line saying every role runs on the session model until they run `/skill:setup-kstack`.

## Get set up

1. Install the package:

   ```bash
   pi install git:github.com/<you>/kstack
   ```

   A local checkout installs with `pi install ./kstack`, and `pi -e ./kstack` tries it for one run. Pi lists what it loaded with `pi list`, and `/reload` picks up a new install in an open session.
2. Run [`/skill:setup-kstack`](../setup-kstack/SKILL.md). It asks for a thinking budget, maps a model to each role you want to pin, and writes a kstack section into `~/.pi/agent/AGENTS.md`. That block applies to new sessions. Then it offers once each to generate a verification skill and a formal verification skill for the project, unless the project already has one.
3. Start a real task with `/k-mode`, a goal, and a check that can pass or fail.

Installing changes nothing until the user invokes a skill. The [README](../../README.md) and [guide page 1](../../docs/guide/01-setup.md) have the details. Offer to word their first prompt with them, per [`references/prompting.md`](references/prompting.md).

If cost is the worry, say where the tokens go and how to spend fewer. kstack spends extra tokens on subagents and review panels. Rerun `/skill:setup-kstack` and pick a smaller budget or cheaper models. A role left on the session model costs nothing extra. A shorter panel list runs fewer subagents, one for each entry. Save `/k-mode` for work that needs rigor.

Every skill here uses the Agent Skills format, so another agent tool can read the files even without pi. What it loses is the pi specifics: the `subagent` tool, the `ask_user` tool, and the model block in `AGENTS.md`. Anything that spawns delegates needs a replacement for those.

## Start a task with `/k-mode`

`/k-mode` matches the task to a playbook, copies the playbook's steps into the plan, and runs the other skills as the steps need them. A good prompt states the goal and how to tell it's done. It doesn't list skills, because a hand-written sequence tends to drop or reorder steps the playbook would keep. Read [`references/prompting.md`](references/prompting.md) before you help word one. [Guide page 2](../../docs/guide/02-k-mode.md) has examples.

Whether k-mode stays on depends on how the user starts it:

- `/k-mode <task>` expands a prompt template, so the skill's instructions enter the conversation for that turn.
- A session keeps what it loaded, so k-mode keeps applying across later turns of the same session. Compaction can drop it. Start the next task with `/k-mode` again.
- To have it on in every session without typing it, add one line to `AGENTS.md`, such as "use k-mode for non-trivial engineering work". Pi loads `AGENTS.md` from the working directory and from `~/.pi/agent/AGENTS.md` on every session.

Mid-chat, "new task" makes the mode match a fresh playbook. `/k-mode` already uses the `k-agent` subagent for the delegates its playbook steps spawn. To get the same style from a delegate of your own, spawn `k-agent`.

## Pick a skill

The default answer is `/k-mode`, which runs most of the others when its steps need them. Name a skill directly when the user wants more or less of something than the playbook gives. Read the skill before you recommend it, and give one example prompt.

| The user wants to | Skill |
|---|---|
| Do any non-trivial task with rigor | [`k-mode`](../k-mode/SKILL.md) |
| Check that the agent understood their goal and the problem before it starts | [`in-your-own-words`](../in-your-own-words/SKILL.md) |
| Get a prompt rewritten deeper, not bigger, before sending it: what a top professional would add, their raw material and methods, and an outside check. It returns the prompt and doesn't do the task | [`sharpen`](../sharpen/SKILL.md) |
| Know how code works now, or where new code should live | [`how`](../how/SKILL.md) |
| Know why code is shaped this way, or where a number came from | [`why`](../why/SKILL.md) |
| Understand a change or subsystem, explained plainly | [`teach`](../teach/SKILL.md) |
| Catch up on their own recent work on a topic | [`recall`](../recall/SKILL.md) |
| Catch up on what changed in this session since their last message, in plain words and small visuals, with every item waiting on them listed and decidable from the reply alone | [`ketchup`](../ketchup/SKILL.md) |
| Catch up like `ketchup`, in a short subtitled video of slides, with the waiting calls as text under it | [`ketchup-video`](../ketchup-video/SKILL.md) |
| Know what a small diff could break outside itself | [`blast-radius`](../blast-radius/SKILL.md) |
| Settle types and module shape before code that crosses a function boundary | [`architect`](../architect/SKILL.md) |
| Get several attempts at one brief, merged into the best one | [`arena`](../arena/SKILL.md) |
| Run parallel checks over slices, or race workers, each in its own worktree | [`swarm`](../swarm/SKILL.md) |
| Have different models review a diff and try to break it | [`interrogate`](../interrogate/SKILL.md) |
| Fix a bug test-first when a cheap local test exists | [`tdd`](../tdd/SKILL.md) |
| Apply TypeScript rules to `.ts` or `.tsx` work | [`typescript-best-practices`](../typescript-best-practices/SKILL.md) |
| Strip comments before review, using a reviewer that didn't write them | [`no-comments`](../no-comments/SKILL.md) |
| Clean AI tells out of prose | [`unslop`](../unslop/SKILL.md) |
| Write docs, an RFC, a README, a PR description, or a commit message to a standard | [`technical-writing`](../technical-writing/SKILL.md) |
| Hear the last reply again in plain words | [`bro`](../bro/SKILL.md) |
| Give agents a scripted way to drive the app and prove behavior | [`create-verification-skill`](../create-verification-skill/SKILL.md) |
| Bring a verification skill and its feature map back in line with the app | [`maintain-verification-skill`](../maintain-verification-skill/SKILL.md) |
| Formally verify a race, invariant, or other property that tests rarely reach | [`create-formal-verification-skill`](../create-formal-verification-skill/SKILL.md) |
| Bring a formal verification back in line after the code it covers changed, and learn whether a failure is drift or a bug | [`maintain-formal-verification-skill`](../maintain-formal-verification-skill/SKILL.md) |
| Show a PM or designer what the user sees as a chart checked by a TLA+ model, with the shortest path to each state that breaks a rule | [`statechart`](../statechart/SKILL.md) |
| Vet a performance number before reporting or acting on it | [`benchmark-checklist`](../benchmark-checklist/SKILL.md) |
| Run a large or cross-cutting change, or one to review after stepping away | [`figure-it-out`](../figure-it-out/SKILL.md) |
| Keep a decision log during a run, and review it afterward | [`show-me-your-work`](../show-me-your-work/SKILL.md) |
| Pick a model for each role and a thinking budget | [`setup-kstack`](../setup-kstack/SKILL.md) |
| Turn their own working habits into a personal mode skill | [`automate-me`](../automate-me/SKILL.md) |
| Turn what a finished task taught into skill edits | [`reflect`](../reflect/SKILL.md) |
| Stop agents from repeating the same mistakes in this repo | [`correct`](../correct/SKILL.md) |
| Build a page whose buttons wake an agent run through a webhook | [`make-bot-ui`](../make-bot-ui/SKILL.md) |
| Find their way around kstack | `/k-help` |

If a skill directory next to this one is missing from the table, read its frontmatter and route by its description. The `principle-*` directories are covered under principles below.

Close calls:

- `how` explains what the code does. `why` explains the reasons. `teach` runs one or both and explains the result plainly.
- `arena` gives every worker the same brief and merges the best parts. `swarm` splits work into slices or a race and returns one report.
- `interrogate` reviews the diff. `blast-radius` looks for breakage outside the diff and proves the one fact that makes the change safe.
- `recall` rebuilds context across recent sessions. `ketchup` covers only this session since the user's last message. Resuming one specific session or branch is the Session pickup playbook.
- `figure-it-out` designs one rigorous run. The Autonomous run playbook drives one task to a finish condition.

Not in kstack:

- There is no `deslop`, `control-cli`, or `control-ui`. `no-comments` covers the comment strip before review, and `create-verification-skill` covers driving a real app.
- Pi has no built-in skill author to route through, so the Authoring a skill playbook owns that job.
- Pi has no built-in PR watcher. The Babysit playbook owns PR status, and [`k-mode/scripts/watch-pr`](../k-mode/scripts/watch-pr) polls one.

## Playbooks and principles

Playbooks are step lists inside `k-mode`, not skills, so they have no command of their own. Inside `/k-mode`, describing the task picks one, and these phrases name one directly:

- "babysit this pr" or "check on pr 123" runs Babysit. It drives the PR to merge-ready and stops there. It doesn't merge unless the user asks to merge, land, or ship.
- "land the stack" runs Shipping.
- "take over this branch" runs Session pickup.
- "pause safely" runs Pause safely.
- "full autopilot on this queue" runs Autopilot-full. "autopilot, don't merge" runs Autopilot-stack.
- "run the eval playbook" runs Eval.

The Playbooks section of [`k-mode`](../k-mode/SKILL.md) lists every playbook. [Guide page 6](../../docs/guide/06-verify-and-ship.md) covers opening, babysitting, and landing a PR.

kstack has no planning skill. Pi has no plan mode either, so a plan is a reply section or a file. For work that spans phases or stacked PRs, asking `/k-mode` for a plan runs the [Multi-phase plan playbook](../k-mode/playbooks/multi-phase-plan.md), which writes the plan and doesn't implement it. For a design question, the Prototype playbook or `architect` settles it in code first.

Principles are one-rule skills that `k-mode` reads and cites in its replies. The user rarely invokes one. They steer with the names instead, as in "apply prove it works. show me the real output." `/skill:principle-<name>` still loads one on demand. [Guide page 8](../../docs/guide/08-principles.md) lists them.

## Fix a run that went wrong

| Symptom | Fix |
|---|---|
| The mode stopped applying after a compaction | Start the task with `/k-mode` again, or put one line about it in `AGENTS.md`. |
| A question got treated as the next step of the last task | Say "new task", or say the turn doesn't need the mode. |
| A new model choice had no effect | The kstack block in `AGENTS.md` applies to new sessions. Start one, or run `/reload`. |
| Runs cost more than expected | See the cost paragraph under Get set up. |
| A skill didn't load on its own | Pi loads a skill when a task matches its description, and it may miss. Type `/skill:<name>`, or let `/k-mode` route to it. |
| Two agents overwrote each other | Spawn each with `worktree: true`, so each gets its own checkout. |
| An overnight run moved but finished nothing | The finish condition needs a check that can pass or fail, not a duration. See [guide page 7](../../docs/guide/07-overnight.md). |
| The reply claims success from a green build | Ask for the real command, flow, stored value, or profile. That's the prove-it-works principle. |
| A subagent ignored the operator's rules | Its kickoff didn't open with `Read <path>/worker.md before anything else.`. |

For a run that drifts, [`references/prompting.md`](references/prompting.md) has one-line steers. [Guide page 10](../../docs/guide/10-recipes-and-pitfalls.md) has more pitfalls and the recipes worth copying.

## Make kstack your own

- [`automate-me`](../automate-me/SKILL.md) drafts a personal mode skill from the user's own history, to use alongside `k-mode`.
- [`reflect`](../reflect/SKILL.md) after a session turns its lessons into skill edits the user approves.
- `/k-mode write a skill for <workflow>` runs the authoring playbook. The eval playbook tests a skill change blind.
- Fix a misbehaving skill in its own PR, not inside the feature work where it went wrong.

[Guide page 9](../../docs/guide/09-make-it-yours.md) covers each of these.

## Reply

Lead with the answer. Give at most one example prompt in a code block, adapted from [`references/recipes.md`](references/recipes.md) when one fits, then the link to that file. Keep it short unless the user asked for the whole map.
