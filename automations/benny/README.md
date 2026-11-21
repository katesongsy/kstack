# benny

benny gives you two headless jobs for slack issue reports. one triages each report. the other reproduces confirmed bugs and may prepare a small draft fix.

the files in this directory are dormant setup and job sources. they do not appear as skills the model loads on its own.

## set it up

1. point pi at [`FOR_AGENTS.md`](./FOR_AGENTS.md) and name the target repository.
2. let setup merge this whole directory into the target at `.pi/automations/benny/`. it must preserve destination-only files and review conflicts instead of overwriting local edits.
3. let setup enable kstack in the target repository's `.pi/settings.json` for shared dependencies:

```json
{
	"packages": ["git:github.com/<owner>/kstack"]
}
```

4. keep user-owned configuration outside the copied pack, for example in `.pi/benny/`. adapt [`configuration.example.yaml`](./templates/configuration.example.yaml) and [`feature-map.example.md`](./skills/reproduce-and-fix-issues/references/feature-map.example.md).
5. commit `.pi/settings.json`, `.pi/automations/benny/`, and any secret-free configuration before enabling either job.
6. wire each job: a command that runs `pi -p` and reads its operational file, plus a trigger that starts it from a new report. then send a harmless test report and verify every source-channel post stays in the original thread.

## how a job runs

there is no hosted automation product in the loop. a job is one headless pi run:

```bash
pi -p "Read and follow .pi/automations/benny/skills/triage-issue-reports/SKILL.md for this run. The configuration is at <path>. The report is channel <id>, thread <ts>."
```

a slack event relay in CI, a cron poll of the channel, or a small local listener decides when to run it. the skill file decides what it does.

## files

- [`FOR_AGENTS.md`](./FOR_AGENTS.md) is the intent, written for the agent that installs the pack.
- [`templates/`](./templates/) holds the configuration example and the two prompt sources.
- [`skills/setup-benny/`](./skills/setup-benny/SKILL.md) is the install procedure.
- [`skills/triage-issue-reports/`](./skills/triage-issue-reports/SKILL.md) and [`skills/reproduce-and-fix-issues/`](./skills/reproduce-and-fix-issues/SKILL.md) are the two operational files.
