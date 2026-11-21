---
name: arena
description: "Spawn N parallel candidates at the same task, pick a base, graft the strongest parts of the losers into it. Use for /skill:arena, 'arena this', 'throw it in the arena', or when one attempt at a non-trivial artifact would lock in the wrong shape."
disable-model-invocation: true
---

# Arena

Open the plan with one entry per phase (Frame, Fan out, Cross-judge, Pick, Graft, Verify) before launching anything.

## Phase A: Frame

Every candidate gets the same prompt, so the prompt is the contract.

1. State the artifact each candidate is producing.
2. Derive the rubric, 3-6 concrete gradeable criteria for what success looks like on *this* task. It is the picker's tool in Phase D. Candidates only see the task.
3. Pick the runners from the `arena runners` line in the kstack block in `~/.pi/agent/AGENTS.md`. With no block, every runner uses the session model, which makes the arena single-model. An `inherit-parent` or `auto` entry is the session model too, so omit `model` for it. Put each seat on a different provider where you can, because a second provider is what a real bakeoff and a cross-judge buy. If a launch rejects a model, use the nearest accepted model of the same family or drop the seat, keep at least two runners, and say so. Two models on one provider are one family. Spawn more for multiple design directions, and repeat one model when the work is generation-bound, not judgment-sensitive. Pick each subagent's thinking level per the Subagents section of the **k-mode** skill.
4. Give each candidate its own output location. Use `worktree: true` on the launch where the candidate writes to the repo, otherwise `/tmp/arena-<slug>/candidate-<n>/`, per the **separate-before-serializing-shared-state** principle skill.

## Phase B: Fan out

Spawn all N subagents from one `subagent({ workflow: true })` call with `async: true`, each with the agent, the task, the path to the shared grounding, its own output path, and instructions to produce the artifact and a short rationale naming the alternatives it considered and what it rejected.

If a candidate produces no output, proceed with N-1 and note the dropout.

## Phase C: Cross-judge

After all Phase B candidates complete, spawn one judge on a read-only tool set from the `arena cross-judge pool` line in the kstack block in `~/.pi/agent/AGENTS.md`. The judge is never from the parent's provider, so take the first entry that names another provider. An `inherit-parent` or `auto` entry is the session model, which shares the parent's provider. When every seat shares the parent's provider, skip the judge and say the arena ran single-vendor. If a launch rejects a model, use the nearest accepted model of the same family, then the next such entry, or skip the judge, and say so. While you read in Phase D, the judge scores each candidate, by path label, on each rubric criterion and recommends a base.

## Phase D: Pick a base

Read every candidate end to end before picking. Score each against the rubric criterion by criterion, not on holistic feel, and compare with the cross-judge when Phase C ran one. Agreement on the base confirms the pick. Disagreement means one of you is biased or the rubric was ambiguous. Read both rationales before deciding.

Pick the base a future maintainer can extend most easily without breaking invariants. When two feel tied, prefer the cleaner boundary or smaller API, per the Laziness Protocol.

## Phase E: Graft

Walk each losing candidate once more for what is worth porting into the base, usually one or two things per candidate, not most of it. Fold each graft in by hand, per the **redesign-from-first-principles** principle skill, so the result stays coherent under one mental model. Don't paste mechanically.

If the candidates converge on one shape, that is strong agreement. Note it and ship the consensus shape. No graft is needed. If they wildly diverge, Phase A was under-specified. Reframe and re-run rather than averaging the divergence.

## Phase F: Verify

Verify the synthesized artifact like any other output, per the **prove-it-works** principle skill. If verification finds a problem the arena missed, either Phase A was wrong (re-frame and re-run) or a candidate caught it and you missed the graft (go back to Phase E). Don't paper over it.

## Outputs

One synthesized artifact, with a short synthesis note alongside naming the base and why, the cross-judge's verdict or that Phase C skipped the judge, each graft with its source candidate, what was rejected and why, any convergence or dropouts, and the verification result.
