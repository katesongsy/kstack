---
name: setup-benny
description: Configure Benny and prepare its triage and repro jobs. Use when installing Benny or changing its Slack, tracker, repository, routing, control, model, or budget settings.
disable-model-invocation: true
---

# Set up Benny

Benny ships as a dormant automation pack inside kstack. The package manifest exposes only kstack's normal skill roots; this file and the two operational files are not skills the model loads on its own.

The human enters setup by pointing pi at the pack's `FOR_AGENTS.md`. The bootstrap flow copies the whole pack into the target repository, then reads this file directly at `.pi/automations/benny/skills/setup-benny/SKILL.md`.

Benny needs external configuration and two live jobs that start a headless `pi -p` run.

Do not create or update a job until the user explicitly asks. Never put a secret value in package files, prompts, or committed configuration.

## 1. Copy the pack and enable shared kstack skills

Do this before asking for Benny configuration and before wiring either job.

Ask which repository will run the jobs. The source pack is the directory containing `FOR_AGENTS.md`. The destination is `<target-repository>/.pi/automations/benny/`.

Merge the entire source pack into the destination:

1. Create the destination when it is absent.
2. Copy every source file to the same relative path.
3. Preserve destination-only files. Never delete unrelated files during install or refresh.
4. Keep user-owned configuration, feature maps, and routing maps outside the destination. Never overwrite them.
5. When an existing source-managed file differs, inspect the diff and merge without discarding local edits. If ownership is ambiguous, stop and ask before replacing it.
6. Verify that the destination contains `FOR_AGENTS.md`, this setup file, both operational files, their references, and the templates.

If this file is already being read from the target destination, treat the copy as complete and run the same verification before continuing.

Add kstack to the target repository's `.pi/settings.json`. If the file or the `.pi` directory does not exist, create it.

Merge this entry into the existing JSON or JSONC:

```json
{
	"packages": ["git:github.com/<owner>/kstack"]
}
```

Preserve every unrelated top-level setting and every other package entry. If the kstack source already appears in `packages`, leave it as it is and remove the duplicate. Preserve comments and valid JSONC syntax when the file uses JSONC. Validate the file after editing it.

Trust the project in a fresh pi session rooted there, or run `/reload` in an open one. Verify that these shared kstack skills resolve from project scope:

- `why`
- `tdd`
- `unslop`
- `principle-separate-before-serializing-shared-state`
- `principle-minimize-reader-load`
- `principle-guard-the-context-window`
- `principle-sequence-verifiable-units`
- `principle-fix-root-causes`
- `principle-prove-it-works`

Do not count a skill loaded from the current session or a user-scoped install. The check must show that a fresh session in the target repository receives kstack through project settings.

If project-scoped packages are unavailable or any shared dependency does not resolve, stop and explain the failure.

The Benny files are read directly from `.pi/automations/benny/`. Do not add that directory to the package manifest or expect its `SKILL.md` files to appear in the skill catalog.

Tell the user that `.pi/settings.json`, `.pi/automations/benny/`, and any referenced secret-free configuration must be committed before either job is enabled. Do not commit them unless the user asks.

Once this check passes, the job prompts may read the committed operational files by their stable repository-relative paths. They must not embed a package cache path or copy the file contents.

## 2. Adapt the configuration

Open these copied examples:

- `../../templates/configuration.example.yaml`
- `../reproduce-and-fix-issues/references/feature-map.example.md`

Create user-owned copies outside `.pi/automations/benny/`. These are configuration files, not pack files. Example locations:

- Project config, such as `.pi/benny/configuration.yaml`
- Project feature map, such as `.pi/benny/feature-map.md`
- Project routing map, such as `.pi/benny/routing.md`
- User config, such as `~/.config/benny/configuration.yaml`
- User feature map, such as `~/.config/benny/feature-map.md`

Fill one feature-map section for every user-facing feature the job may reproduce. Keep it at the user point of view. Do not freeze implementation details or current code paths in the map.

Do not edit the copied examples. Pack refreshes may update source-managed files after conflict review, but they must never touch the user-owned copies.

Prefer committed, secret-free files in the target repository when a fresh job checkout must read them. Otherwise paraphrase the required values into the job prompt. Reference a repository file only after confirming that it is committed on the branch the job checks out.

Use stable repository-relative paths for committed pack and configuration files. Never reference the package source directory or a package cache path from a job prompt.

## 3. Fill the required choices

Ask for or confirm:

- Source Slack channel ID
- Optional operations or status channel ID
- Repository URL and default branch
- Triage identity or Slack user ID
- Issue tracker type, team, project, labels, and intake status
- Tracker adapter skill or MCP actions
- Optional routing map path
- Required control skill name
- Required user-facing feature-map path
- Status emoji strings
- Pull request URL format
- Polling and effort budgets
- Model slug for triage, repro, code work, and media review

Use only model ids that `pi --list-models` reports. Do not guess a slug and do not carry over a private default.

The source channel, triage identity, repository, tracker adapter, control skill, and feature map must be explicit. Fail setup if any required value stays ambiguous.

Use kstack's `unslop` skill on the final job names and prompts before saving them.

## 4. Check integration capabilities

The triage job needs:

- Read access to the configured source Slack channel and its threads
- Thread-reply access in that channel
- Attachment metadata and file download access when reports include media
- Search, read, create, and update access through the configured issue-tracker adapter

The repro job needs:

- Read access to the source thread
- Thread-reply access in the source channel
- Optional post and edit access in the configured operations channel
- Repository read and history access
- A pull request action that can open a draft pull request
- The configured control-adapter skill

Prefer the Slack MCP tools for reads and posts. The optional `BENNY_SLACK_BOT_TOKEN` may fill a narrow gap such as editing one operations status message or downloading an attachment. Store the value in a secret manager or environment, not in YAML.

Do not use undocumented integration endpoints.

## 5. Prepare the routing map

If the user wants reroutes or owner pings:

1. Copy `../triage-issue-reports/references/routing.example.md` outside `.pi/automations/benny/`.
2. Replace every placeholder with public or organization-local values.
3. Keep owner pings off by default.
4. Allow a ping only for a configured feature owner or a confirmed likely regression author.

If no routing map is configured, triage may classify a report but must not guess a destination or owner.

## 6. Verify the control adapter

Read `../reproduce-and-fix-issues/references/control-adapter.md` and the user's completed feature map.

Confirm that the named skill can:

- Bring up the target app
- Navigate every mapped feature through the real UI
- Exercise mapped states through declared adapter actions
- Inspect state without forcing the result
- Capture screenshots
- Start and stop a recording
- Clean up its processes and temporary data

If any capability is missing, leave the repro job disabled. It must fail closed rather than claim a reproduction it did not perform.

## 7. Prepare the two jobs

Each job is two things: a command that starts a headless run, and a trigger that decides when to start it.

The command reads its operational file and nothing else it has to guess at:

```bash
pi -p "Read and follow .pi/automations/benny/skills/triage-issue-reports/SKILL.md for this run. The configuration is at <configuration path>. The report is channel <channel id>, thread <thread ts>."
```

The trigger is one of these. Ask which the user has.

- **A Slack relay in CI.** A Slack event subscription forwards a new top-level report to a CI workflow, and the workflow runs the command with the thread coordinates as inputs. This is the closest match to the original design, and it fires per report.
- **A cron poll.** A scheduled job runs the command every few minutes with a "poll the source channel for new reports" instruction, and the operational file's own polling budget decides what counts as new. It needs no inbound webhook, and it costs a run per interval.
- **A local listener.** A small server on the user's machine accepts the Slack event and runs the command. Good for a trial, and only alive while the machine is.

Fill each job's prompt from `FOR_AGENTS.md`, the finished configuration, and the matching template under `templates/`. Keep the prompt short: the operational file owns the behavior.

### The triage job

- Name `benny-triage`.
- Reads and follows `.pi/automations/benny/skills/triage-issue-reports/SKILL.md` on every run.
- Triggers on each new top-level report in the configured source Slack channel.
- Reads the triggering thread and replies only inside it.
- Uses the configured issue-tracker integration.
- Classifies, inspects evidence, traces cause, dedupes, and creates only clear new bugs.
- Ends with one thread-only verdict carrying the configured `[benny:bug]`, `[benny:performance]`, or `[benny:other]` marker and an optional tracker URL.
- Never posts a source-channel root message.

### The repro job

- Name `benny-reproduce`.
- Reads and follows `.pi/automations/benny/skills/reproduce-and-fix-issues/SKILL.md` on every run.
- Triggers on the same new top-level reports, or on the trusted triage marker when the relay can filter on it.
- Uses the configured repository and default branch.
- Reads the source thread and replies only inside it.
- Carries the pull-request action and the configured tracker, control-adapter, and feature-map requirements. Paraphrase mapped user paths and states unless an eligible committed file exists on the branch the job checks out.
- Waits for a trusted triage marker before acting.
- Reproduces the exact symptom twice through the mapped real UI and captures evidence.
- Verifies an existing fix without authoring over it.
- Attempts an optional bounded fix only after a confirmed repro, then opens a draft pull request when the proof and checks pass.
- Never posts a source-channel root message.

Finish one job end to end, including the thread-safety test in step 8, before wiring the second.

### Changing an existing job

Do not create a second job for the same purpose. Find the existing trigger and command, then edit them:

- The command's operational file path, when the pack moved.
- The model, when the configuration changed it.
- The Slack capabilities and the tracker integration.
- The paraphrased triage or repro instructions, when the operational file changed.
- The trigger's channel filter and marker filter.

Ask the user to make the change where the trigger lives, such as the workflow file or the crontab. Then rerun step 8 before enabling traffic.

### Boundary

Do not enable either job until the thread-safety test passes. Do not give a job a Slack token that a worker subagent can read, and do not let a job post outside the coordinates it froze.

## 8. Test thread safety

Use a test channel or a harmless test report.

Before testing, confirm that the target repository's `.pi/settings.json`, `.pi/automations/benny/`, and every referenced secret-free configuration file are committed on the branch the job checks out. Confirm that both live prompts point at their exact committed operational files. If any check fails, stop. Tell the user that the job cannot be enabled yet.

Verify:

1. Triage stores the root `thread_ts` and posts exactly one verdict as a reply.
2. The verdict contains one configured marker.
3. Repro accepts the marker only from the configured triage identity.
4. Repro keeps the same immutable source coordinates.
5. No source-channel root message appears.
6. A delegated worker cannot use any Slack write action.
7. Missing coordinates, a deleted parent, or a failed preflight produces no post and no tracker issue.

Enable normal traffic only after all seven checks pass.
