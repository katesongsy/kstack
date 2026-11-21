# Set up kstack

In this page you install the package, pick which models kstack uses, and run your first task. Setup is one command plus a short conversation.

## Install the package

kstack is a pi package. From a shell:

```bash
pi install git:github.com/<you>/kstack
```

A local checkout installs with `pi install ./kstack`, and `pi -e ./kstack` tries it for one run without adding it. `pi list` confirms what is installed, and `/reload` picks it up in a session that is already open.

## Pick your models

Run:

```text
/skill:setup-kstack
```

[`setup-kstack`](../../skills/setup-kstack/SKILL.md) detects the models pi can reach, asks for a thinking budget, shows you each role (code delegates, judgment, verification, the review panels), and asks what you want. Answer the questions. It writes a small block into `~/.pi/agent/AGENTS.md` between two `kstack:models` markers, and every kstack skill reads it.

Every role runs on your session model by default, so kstack needs no configuration at all to work, on any provider. Pinning a role buys two things: a cheaper or faster model on mechanical work, and a second provider on a panel.

The budget sets the thinking level. `large` gives hard roles `xhigh` and the rest `high`, `medium` gives `high` and `medium`, and `small` gives `medium` and `low`. Nothing runs above `max`, and kstack never writes `max`.

You only override what you care about. A role with no line runs on the session model. To restore that, delete the role's line. A rerun of `/skill:setup-kstack` keeps any role whose model differs from the default. When a default changes, a block written before the change still pins the old value until you run `/skill:setup-kstack` again. That run replaces the old defaults it recognizes, tells you which, and keeps your other choices. To drop a pin it does not recognize, delete that line or the block first.

## Accept the verification offers, or don't

At the end of setup, `/skill:setup-kstack` looks for a way to prove app behavior in your project, either a `verify-*` skill or an existing harness. If it finds neither, it offers once to generate one with [`/skill:create-verification-skill`](../../skills/create-verification-skill/SKILL.md).

Say yes and it writes `.pi/skills/verify-<app>/`, a project-local skill that teaches agents to drive your app the way a user does. It proves the skill works once before handing it over. Say no and setup moves on. You can run `/skill:create-verification-skill` yourself any time. [Verify and ship](./06-verify-and-ship.md#create-a-project-verification-skill) covers it in depth.

If you're new to kstack, say yes. An agent that can check its own work keeps going until the check passes. An agent that can't hands every result back to you to check by hand. Of everything in this guide, the verification skill pays off the most.

Then setup offers once to generate a formal verification skill with [`/skill:create-formal-verification-skill`](../../skills/create-formal-verification-skill/SKILL.md), unless the project already has one. Tests check the cases someone thought to write. A formal verification skill checks one key property of your code, such as "a job never runs twice at once", in every case within a stated scope. Say yes when your code has concurrency, retries, a state machine, or a parser, where a bug is costly and tests rarely reach the case that breaks. Otherwise say no, and run it later if that changes. [Verify and ship](./06-verify-and-ship.md#prove-a-property-with-a-formal-verification-skill) covers it.

After setup, start a new session, or run `/reload`. The block applies to new sessions.

## Keep the cost in check

kstack spends extra tokens on subagents and review panels. That's the price of the rigor. To spend fewer:

- Rerun `/skill:setup-kstack` and pick a smaller reasoning budget or cheaper models. A strong model in the main session with cheaper, faster models in the code roles is a good split.
- Leave a role on the session model so it costs nothing beyond the session itself.
- Shorten a panel list. Each entry runs one subagent.
- Save `/k-mode` for work that needs rigor. A small, obvious edit doesn't.

## Run your first task

Pick something real but small, and describe it the way you'd describe it to a colleague:

```text
/k-mode add a --json flag to this command. text output stays byte-identical. verify both.
```

Watch the plan. Its first items are the matched playbook's steps copied in, the Feature playbook for this prompt.

From here you can type normal follow-ups. The session keeps what `/k-mode` loaded, so the style holds for later turns, but compaction can drop it. Start the next task with `/k-mode` again, or put one line in `AGENTS.md`, such as "use k-mode for non-trivial engineering work", to have it every session.

Next: [Route work through `/k-mode`](./02-k-mode.md).
