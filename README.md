# kstack

> Adapted from [pstack](https://github.com/poteto/pstack) by [poteto](https://x.com/poteto), retargeted from Cursor to the [pi coding agent](https://pi.dev). The ideas, the playbooks, and most of the words are hers.

kstack is a rigorous engineering stack for pi. It gives pi a way to work that aims at shipping less code, of higher quality, with proof that it works.

AI writes too much slop. Throughput without quality is not a goal worth having. If you want to go fast, go deep first. kstack is that stance, turned into skills a coding agent can follow: every task routes through a playbook, every claim carries evidence, and every change gets verified against the real artifact.

Two things follow from it:

- **Fewer, better changes.** The stack biases toward deletion, the smallest change that solves the problem, and code whose data shapes are named before any logic is written.
- **Fearless parallelism.** Once one agent can be trusted to write verifiable code, you can run several with confidence. `k-mode` spawns `k-agent` delegates, each in its own worktree, and every one of them applies the same principles and the same bar.

Every frontier model is good at something different, and kstack is model-agnostic. It runs on whatever provider you have configured in pi, and `/skill:setup-kstack` lets you pin a different model to each role when you want a mix.

Fork it, improve it, make it yours. PRs are welcome.

## install

kstack is a pi package. Install it from git, npm, or a local checkout:

```bash
pi install git:github.com/<you>/kstack
pi install ./kstack          # a local checkout
pi -e ./kstack               # try it for one run, without adding it
```

`pi list` shows what is installed, and `/reload` picks up a new install in an open session.

## get started

Two steps:

1. **Pick your models.** Run `/skill:setup-kstack`. It detects the models pi can reach, asks for a thinking budget, and writes a small block into `~/.pi/agent/AGENTS.md` mapping each role to a model. Answer as little or as much as you like. With no pins, every role runs on your session model and nothing else is needed.
2. **Start a task.** Use `/k-mode` for anything that needs rigor:

   ```text
   /k-mode add a --json flag to this command. text output stays byte-identical. verify both.
   ```

New here? The [kstack guide](./docs/guide/README.md) walks through a first real task, from setup and prompting through verification and overnight runs. Stuck, or unsure which skill fits? Ask `/k-help`.

The rest of the skills are situational, and `k-mode` runs them for you as its steps need them.

## usage

`/k-mode` is the main entry point. It reads your request, matches it to a playbook, copies that playbook's steps into its plan, and runs the other skills as the steps fire.

```text
/k-mode this pr has a subtle bug where the scroll drifts every 750ms even when idle. repro first,
then fix and verify.
```

```text
/k-mode i'm going to bed. land the stack even if ci flakes. i want everything merged by morning.
```

<details>
<summary>the twenty-two playbooks</summary>

| playbook | for |
|---|---|
| [investigation](./skills/k-mode/playbooks/investigation.md) | a read-only question. how does x work, why was y built this way, are we sure. |
| [bug fix](./skills/k-mode/playbooks/bug-fix.md) | reproduce a defect, root-cause it, and fix with runtime evidence. |
| [perf](./skills/k-mode/playbooks/perf-issue.md) | trace a measured slowness and improve it against a baseline. |
| [hillclimb](./skills/k-mode/playbooks/hillclimb.md) | sustained, scientific improvement of one metric against a target, looping hypotheses with before/after measurement and one commit per accepted win. |
| [runtime forensics](./skills/k-mode/playbooks/runtime-forensics.md) | diagnose a live symptom (leak, idle-cpu spin, glitch) from instrumentation. |
| [trace forensics](./skills/k-mode/playbooks/trace-forensics.md) | diagnose a captured profiling artifact (cpuprofile, trace, spindump, heap snapshot). |
| [feature](./skills/k-mode/playbooks/feature.md) | new or changed behavior, built from a named data shape. |
| [refactoring](./skills/k-mode/playbooks/refactoring.md) | a behavior-preserving change to structure or shape. |
| [prototype](./skills/k-mode/playbooks/prototype.md) | a throwaway sketch to make a design or behavioral decision cheaply, or to settle an empirical fork by observing it. |
| [visual parity](./skills/k-mode/playbooks/visual-parity.md) | pixel-exact ui equivalence between two implementations. |
| [authoring a skill](./skills/k-mode/playbooks/authoring-a-skill.md) | writing or editing a SKILL.md. |
| [eval](./skills/k-mode/playbooks/eval.md) | test how a skill or prompt change affects agent behavior, blinded. |
| [babysit](./skills/k-mode/playbooks/babysit.md) | drive a pr or a stack to merge-ready: conflicts, review threads, ci. |
| [shipping](./skills/k-mode/playbooks/shipping.md) | independently verify a green stack, then land the contiguous verified run bottom-up through github by default or origin when available. |
| [autonomous run](./skills/k-mode/playbooks/autonomous-run.md) | drive a long task to completion without stopping. |
| [autopilot-full](./skills/k-mode/playbooks/autopilot-full.md) | run independent prs to merged with one owner per pr. a swarm verifies each code-ready head, and every later change to the patch gets a fresh check. |
| [autopilot-stack](./skills/k-mode/playbooks/autopilot-stack.md) | autopilot-full without the merge. build and verify prs for the operator to review and land, stacking a pr only on a pr it builds on. |
| [session pickup](./skills/k-mode/playbooks/session-pickup.md) | resume or take over a prior agent's in-flight work. |
| [pause safely](./skills/k-mode/playbooks/pause-safely.md) | suspend in-flight work cleanly so it can be resumed later. |
| [multi-phase plan](./skills/k-mode/playbooks/multi-phase-plan.md) | work that spans phases or stacked PRs. |
| [worktree cleanup](./skills/k-mode/playbooks/worktree-cleanup.md) | reclaim disk by pruning merged or abandoned worktrees and stale ios simulators, safety-gated. |
| [opening a pr](./skills/k-mode/playbooks/opening-a-pr.md) | open a ready pr from small ordered commits with a conventional commits title and a briefing-style body. invoked at the end of every other playbook. |

</details>

When invoked, it:

1. matches your task to a [playbook](./skills/k-mode/playbooks/) and writes its steps as a numbered plan, copied in verbatim. For long runs the same plan lands in `.audit/plan-<task-slug>.md`.
2. routes to the other skills as the steps fire.
3. writes unslopped replies framed for the consumer and the maintainer.

The full rules and playbooks live in [`skills/k-mode/SKILL.md`](./skills/k-mode/SKILL.md).

`/k-mode` is a prompt template, so it costs one `/k-mode` per task. What a session loaded stays with it, but compaction can drop it, so start the next task with `/k-mode` again. To have the style every session, add one line to `AGENTS.md`, for example "use k-mode for non-trivial engineering work".

## skills

`k-mode` runs most of these for you when a step needs them (`why`, `architect`, `arena`, `swarm`, `interrogate`, `unslop`, `no-comments`, `technical-writing`, `tdd`, and the principles). Every skill other than `k-mode` and `k-help` loads with `/skill:<name>`. The table below is for when you want one directly.

```text
/skill:how do we cancel runs? do we have an n+1 when we look up every run to cancel?
```

```text
/skill:interrogate review this pr.
```

<details>
<summary>all skills</summary>

| skill | use it when |
|---|---|
| [`k-mode`](./skills/k-mode/SKILL.md) | default entry point for any non-trivial task. also `/k-mode`. |
| [`k-help`](./skills/k-help/SKILL.md) | you're new to kstack, or unsure which skill, playbook, or principle fits. finds out what you're trying to do, answers that part, and hands you a prompt to type. also `/k-help`. |
| [`in-your-own-words`](./skills/in-your-own-words/SKILL.md) | you want the agent to play back your goal and the problem before it starts. |
| [`sharpen`](./skills/sharpen/SKILL.md) | you want a prompt made deeper, not bigger, before you send it: what a top professional would do that you didn't ask for, the raw material they'd start from, their methods and tools, and an outside check that the result is right. hands back the rewritten prompt, which asks the agent to prototype any choice an experiment can settle and leaves product and preference choices to you. doesn't do the task. |
| [`how`](./skills/how/SKILL.md) | you want a walkthrough of how a subsystem works. |
| [`why`](./skills/why/SKILL.md) | you want to know why something was built this way. answers from the record, not the code: the commits and pr discussions that introduced it and each later change, and other sources when that trail is thin. cites what the authors wrote, marks its own inferences, and says what the record does not answer. |
| [`recall`](./skills/recall/SKILL.md) | you're starting or resuming work and want your recent context on a topic rebuilt from your own session history and the shared record, handed back as a tight current-state brief. |
| [`ketchup`](./skills/ketchup/SKILL.md) | you want what changed in this session since your last message, in plain words and small visuals, with every item waiting on you listed and decidable from the reply alone, even when that makes the reply long. |
| [`ketchup-video`](./skills/ketchup-video/SKILL.md) | you want the `ketchup` catch-up as a short subtitled video of slides, with the calls waiting on you as text under it. |
| [`blast-radius`](./skills/blast-radius/SKILL.md) | you have a small-looking change and want to know what else it could break, with the one fact it's safe because of proven by running code, not asserted. |
| [`architect`](./skills/architect/SKILL.md) | you're about to write code that crosses a function boundary and want the caller's usage, types, and module shape settled first. |
| [`arena`](./skills/arena/SKILL.md) | you want N parallel attempts at the same thing, then to grab the best parts of each. |
| [`swarm`](./skills/swarm/SKILL.md) | you want N parallel workers across different slices or races, then one aggregated report. |
| [`interrogate`](./skills/interrogate/SKILL.md) | you have a diff and want different models to try to break it, keeping only findings a run proves. |
| [`automate-me`](./skills/automate-me/SKILL.md) | you want your own `-mode` skill, drafted from how you've actually worked. |
| [`make-bot-ui`](./skills/make-bot-ui/SKILL.md) | you want a page or dashboard whose buttons wake an agent run over a webhook. |
| [`setup-kstack`](./skills/setup-kstack/SKILL.md) | you want to pick which models kstack uses per role and at what thinking budget. detects your models and writes a config block. |
| [`reflect`](./skills/reflect/SKILL.md) | a long task landed and you want the recipe captured as a skill edit. |
| [`correct`](./skills/correct/SKILL.md) | you keep correcting agents for the same mistakes. mines history for mistake classes, fixes each at the highest level that works (architecture, then types, lint, and ci, then tests, with docs last), and keeps a table pairing each rule with what enforces it. |
| [`teach`](./skills/teach/SKILL.md) | you want to actually understand a change or subsystem, not just have it summarized. runs how + why and weaves one plain explanation, built up diagram by diagram. |
| [`tdd`](./skills/tdd/SKILL.md) | you're fixing a bug and there's a cheap local test path. write the failing test first, then the fix. |
| [`benchmark-checklist`](./skills/benchmark-checklist/SKILL.md) | you ran a benchmark or measured a speedup or regression. vets the number (limiter, tuning, errors, repeat runs, end-to-end relevance) before you report or act on it. |
| [`no-comments`](./skills/no-comments/SKILL.md) | strip comments before review; spawns Comment Sicko, fixes accepted findings, offers encodings for claimed constraints. |
| [`typescript-best-practices`](./skills/typescript-best-practices/SKILL.md) | you're reading or editing typescript. grounds the type-system-discipline principle in syntax. |
| [`figure-it-out`](./skills/figure-it-out/SKILL.md) | no bundled playbook fits. designs a rigorous, auditable playbook for the task. |
| [`show-me-your-work`](./skills/show-me-your-work/SKILL.md) | you want a reviewable decision trail. logs decisions to a tsv you can commit. |
| [`create-verification-skill`](./skills/create-verification-skill/SKILL.md) | your project has no scripted way to prove app behavior. generates a project-local verify skill with a feature map, for any language or platform. |
| [`maintain-verification-skill`](./skills/maintain-verification-skill/SKILL.md) | your verify skill's feature map has drifted from the app. source wave + one live pass, at most one PR of proven corrections. |
| [`create-formal-verification-skill`](./skills/create-formal-verification-skill/SKILL.md) | tests can't reach the case that breaks a race, retry, state machine, parser, or unsafe code. verifies one property with the tool that fits and generates a project-local skill to recheck it on every change. |
| [`maintain-formal-verification-skill`](./skills/maintain-formal-verification-skill/SKILL.md) | code your formal verification covers has changed since it was last checked. updates the model, harness, or properties to match the code, reruns, and says whether each failure is drift in the verification or a bug in the code. |
| [`statechart`](./skills/statechart/SKILL.md) | someone who owns the behavior needs to see it, not read a spec. checks a tla+ model, draws what the user sees as a mermaid statechart, marks each state that breaks a rule, and gives the shortest path to each one. |
| [`unslop`](./skills/unslop/SKILL.md) | you're cleaning up writing. removes AI tells. |
| [`bro`](./skills/bro/SKILL.md) | you want the last message restated in plain human language, no jargon. |
| [`technical-writing`](./skills/technical-writing/SKILL.md) | layered doc standard (Diátaxis + Google developer style + STE + Global English) for docs, RFCs, readmes, PR descriptions, commit messages. |

</details>

### examples

Most of the time, type `/k-mode` at the start of a task and let it route to a playbook. The other skills fire as the steps need them.

<details>
<summary>all the examples</summary>

```text
bug fix:           /k-mode this pr has a subtle bug where the scroll drifts every 750ms even
                   when idle. repro first, then fix and verify.
perf:              /k-mode a big list takes a second or two to load even though we virtualize.
                   run a cpu trace and tell me why.
feature:           /k-mode build a small feature behind a feature flag. verify it really works.
prototype:         /k-mode build two prototypes of the markdown renderer so we can compare.
                   spawn an agent for each.
multi-phase:       /k-mode open source these skills as a package. nothing internal leaks, work
                   in a temp dir, show me the dependency graph first.
overnight run:     /k-mode i'm going to bed. land the stack even if ci flakes. i want
                   everything merged by morning.
babysit:           /k-mode check on pr 123. anything outstanding?
visual parity:     /k-mode the row spacing is too tall when this flag is on. the second image
                   is correct. repro and fix until it matches.
figure it out:     /k-mode i'm stepping away. migrate every caller from the synchronous store
                   to the new async one, keeping behavior identical. i want to trust it was
                   done right when i'm back.
how:               /skill:how do we cancel runs? do we have an n+1 when we look up every run?
why:               /skill:why is this feature flag not on yet?
architect:         design this instrumentation to be high signal with no false positives.
                   /skill:architect this first.
arena:             /skill:arena take my prompt to the arena verbatim. i want to compare their
                   proposals with yours.
swarm:             /skill:swarm check every package under packages/ against its check.sh. one
                   worker per package. one report.
interrogate:       /skill:interrogate review this pr.
tdd:               /skill:tdd implement
unslop:            can we unslop and tighten the new changes?
reflect:           /skill:reflect that took too long. capture what we learned so the next run
                   doesn't repeat it.
correct:           /skill:correct
show-me-your-work: /skill:show-me-your-work keep a decision trail i can review when i'm back.
automate-me:       /skill:automate-me
help:              /k-help which skill should i use to review this branch?
```

</details>

## the `k-agent` and Comment Sicko subagents

[`k-mode`](./skills/k-mode/SKILL.md) runs in the session that receives it. That session owns the playbook, the plan, and every spawn.

kstack also ships [`k-agent`](./agents/k-agent.md) as the wrapper for delegates spawned during a run. It reads `k-mode` in full, including its inline principles index, before doing any work. Substituting a general-purpose delegate skips that read and drifts. Do not spawn `k-agent` to run the playbook itself.

kstack also ships [Comment Sicko](./agents/comment-sicko.md), a read-only comment reviewer. Usually invoke it through [`no-comments`](./skills/no-comments/SKILL.md), not directly.

## principles

Twenty-four short skills, one principle each. `k-mode` indexes them inline and reads that index at task start. The standalone files are there so other skills can reference a principle by name, and so the index can point at the full rule for each.

<details>
<summary>all twenty-four principles</summary>

| principle | group | rule |
|---|---|---|
| [laziness-protocol](./skills/principle-laziness-protocol/SKILL.md) | core | Bias toward deletion and the smallest change that solves the problem. |
| [foundational-thinking](./skills/principle-foundational-thinking/SKILL.md) | core | Apply before writing logic: choosing core types and data structures, sequencing scaffold-vs-feature work, asking what concurrent actors share. Get the data structures right so downstream code becomes obvious. |
| [redesign-from-first-principles](./skills/principle-redesign-from-first-principles/SKILL.md) | core | Redesign as if the requirement had been a foundational assumption from day one, instead of bolting it on. |
| [attack-the-premise](./skills/principle-attack-the-premise/SKILL.md) | core | Apply when two or more fixes that share one premise have failed the same gate. Write the premise down and test it directly before you write another fix that assumes it. |
| [subtract-before-you-add](./skills/principle-subtract-before-you-add/SKILL.md) | core | Remove dead weight, redundant validators, and stub references first, then build on the simpler base. |
| [minimize-reader-load](./skills/principle-minimize-reader-load/SKILL.md) | core | Count layers between question and answer, and hidden state in the reader's head; collapse one-caller wrappers and shrink mutable scope. |
| [outcome-oriented-execution](./skills/principle-outcome-oriented-execution/SKILL.md) | core | Apply during planned rewrites and migrations with explicit phase boundaries. Converge on the target architecture; don't preserve smooth intermediate states with throwaway compatibility code. |
| [experience-first](./skills/principle-experience-first/SKILL.md) | core | Choose user delight over implementation convenience; ship fewer polished features over more rough ones. |
| [exhaust-the-design-space](./skills/principle-exhaust-the-design-space/SKILL.md) | core | Build 2-3 competing prototypes and compare side by side before committing. |
| [build-the-lever](./skills/principle-build-the-lever/SKILL.md) | core | Apply to any non-trivial work, not just bulk work: edits, migrations, analyses, checks. Build the tool that does it or proves it (codemod, script, generator, or a skill your subagents follow) instead of working by hand. The tool is the artifact a reviewer can rerun. |
| [model-the-domain](./skills/principle-model-the-domain/SKILL.md) | architecture | Encode the domain in a structure instead of scattered conditionals. |
| [boundary-discipline](./skills/principle-boundary-discipline/SKILL.md) | architecture | Concentrate guards at system boundaries (CLI, config, network, external APIs); trust internal types and keep business logic in pure functions. |
| [type-system-discipline](./skills/principle-type-system-discipline/SKILL.md) | architecture | Make illegal states unrepresentable, brand semantic primitives, parse external data at boundaries, exhaust variants, derive from authoritative schemas. |
| [make-operations-idempotent](./skills/principle-make-operations-idempotent/SKILL.md) | architecture | Converge to the same end state regardless of partial prior runs. |
| [migrate-callers-then-delete-legacy-apis](./skills/principle-migrate-callers-then-delete-legacy-apis/SKILL.md) | architecture | Migrate callers and delete the old API in the same wave instead of preserving compatibility layers. |
| [separate-before-serializing-shared-state](./skills/principle-separate-before-serializing-shared-state/SKILL.md) | architecture | Eliminate the sharing first; serialize structurally only when one shared writer is a real invariant. |
| [prove-it-works](./skills/principle-prove-it-works/SKILL.md) | verification | Apply after completing a task, before declaring done. Verify against the real artifact (run the feature, read the actual value, inspect the diff), not a proxy, self-report, or 'it compiles.'. |
| [fix-root-causes](./skills/principle-fix-root-causes/SKILL.md) | verification | Trace each symptom back to the first place the system went wrong and fix it there. A guard is a fix only when the state it handles is legitimate. |
| [sequence-verifiable-units](./skills/principle-sequence-verifiable-units/SKILL.md) | verification | Apply to multi-step work (sweeps, migrations, runs of similar edits) and to how you stack commits and PRs. Break work into small units that each end in a verifiable state, check each before the next, and order delivery so the sequence proves itself to a reviewer. |
| [test-behavior-not-implementation](./skills/principle-test-behavior-not-implementation/SKILL.md) | verification | Apply when you write, change, or keep a test. Assert what the code's users can observe, so the test fails when that behavior breaks and keeps passing when only the internals change. |
| [explain-the-number](./skills/principle-explain-the-number/SKILL.md) | verification | Apply before you trust, report, or act on a number you measured: a speedup, a regression, a throughput, a latency, or an eval result. Find what limits it, and rule out that it measured something other than the work you think. |
| [guard-the-context-window](./skills/principle-guard-the-context-window/SKILL.md) | delegation | Route bulk to subagents; keep summaries in the main thread, not raw payloads. |
| [never-block-on-the-human](./skills/principle-never-block-on-the-human/SKILL.md) | delegation | Proceed, present the result, let the human course-correct after the fact; reserve confirmation for irreversible actions. |
| [encode-lessons-in-structure](./skills/principle-encode-lessons-in-structure/SKILL.md) | meta | Encode the rule as a lint, metadata flag, runtime check, or script instead of more text. |

</details>

## not shipped here

A few things `k-mode` used to lean on, and what replaces them:

- **A comment stripper and app drivers.** `no-comments` covers the comment pass before review. `create-verification-skill` gives agents a scripted way to drive a real app.
- **Cloud agents.** Pi runs children on your machine, isolated by a git worktree. A long run keeps its liveness record in a mission.
- **A loop command.** An async subagent wakes the session when it finishes, and a pi-subagents schedule covers recurring work. See the [pi adapter](./docs/harness/pi.md).
- **A built-in skill author.** The [authoring playbook](./skills/k-mode/playbooks/authoring-a-skill.md) owns writing and editing a `SKILL.md`.

## why are there no planning skills?

A separate planning stage tends to produce a document that is wrong in ways nobody discovers until the code is written. The best spec is code. If you do want a plan, [`k-mode`](./skills/k-mode/SKILL.md) covers it with the [multi-phase plan playbook](./skills/k-mode/playbooks/multi-phase-plan.md), but it is not a default.

## make it yours

`k-mode` is one opinionated style. You may not want exactly that.

Run [`automate-me`](./skills/automate-me/SKILL.md). It mines your recent sessions, drafts a `<your-name>-mode` skill from how you've actually worked, and routes through kstack underneath. You keep kstack as the base and end up with your own routing skill alongside `k-mode`.

Models are configurable too. Run [`setup-kstack`](./skills/setup-kstack/SKILL.md). It detects the models pi can reach and writes a block into `~/.pi/agent/AGENTS.md` mapping each role to a model and thinking level. Every skill reads it and falls back to the session model when a role has no line, so you override only what you want.

When a default changes, a block written before the change still pins the old value until you run `/skill:setup-kstack` again. That run replaces the old defaults it recognizes and keeps any other role whose model differs from the default. To drop a pin it does not recognize, delete that role's line or the block first.

## license

MIT. Copyright (c) 2026 Kate Song.
