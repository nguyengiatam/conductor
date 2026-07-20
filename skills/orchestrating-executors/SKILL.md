---
name: orchestrating-executors
description: Use when a plan is ready and implementation will be delegated to an external coding agent (not a Claude subagent) — establishes the reviewer/executor split, the per-task checkpoint protocol, and quota-aware executor selection.
---

# Orchestrating External Executors

You are the architect and reviewer. You do not write business code. External coding agents (the "executors") implement the plan one task at a time; you gate every task before the next begins.

This is different from `superpowers:subagent-driven-development`, which dispatches Claude subagents. Here the executors are separate tools with their own quotas, quirks, and failure modes. Their exact invocation lives in [references/executor-roster.md](references/executor-roster.md) — read it before dispatching. Keep this skill's body agent-agnostic; all machine-specific commands stay in the roster.

## Roles

- **Claude (you):** design the plan (via `superpowers:writing-plans`), pick the executor, hand off one task, then review and verify the result. Write code yourself only for foundation, concurrency-critical, or verification code where precision matters more than delegation.
- **Executor:** implements exactly one task, commits it, and stops. Never let an executor run multiple tasks unreviewed.
- **Adversarial reviewer:** a second external agent that stress-tests the result — see `adversarial-review-to-go`.

## The Checkpoint Protocol

A **checkpoint** is the reviewable unit: **one plan task → one commit on a feature branch → executor stops → you review.** Enforce all of:

1. One task per handoff. The prompt to the executor names the single task, the current HEAD, the baseline test state, and the safety rules for this repo.
2. The executor commits its own work with the task's commit message, then halts.
3. You review before releasing the next task. Reviewing means running `checkpoint-verification`, then `convention-commit-gate`, on the actual diff — not trusting the executor's summary.
4. Verification output is real and pasted. "Tests pass" without the run output is not acceptance.

If an executor violated the protocol (ran ahead, skipped verify, edited another repo), stop and reconcile before continuing — do not paper over it.

## Selecting an Executor

**Check quota across the whole roster before assigning heavy work.** An executor that is out of quota can fail silently (exit 0, empty output) and burn your time. The roster documents the quota-check command for each.

Selection order is quota-and-strength-based, not fixed:
- Prefer the executor with the most remaining quota that is strong at the task type.
- Route review-heavy, concurrency-sensitive work to the reviewer-strong agent (see roster).
- **Purely mechanical work** (enum/field/error-code changes already spelled out in the plan) — do it yourself. Arguing with a weak executor costs more than typing it.
- Reserve some quota on at least one agent as a fallback / second opinion.

## Handoff Prompt Checklist

Every executor prompt includes:
- The single task and its acceptance criteria (copy from the plan).
- Current HEAD SHA and the baseline test count/state.
- The instruction to reconcile against real code/enums before hardcoding anything (executors catch plan errors this way — when one stops to ask, take it seriously).
- The convention requirements from `convention-commit-gate` for any new code.
- Repo safety rules (branch, ports/DB isolation if parallel, no killing processes by pattern).

## The Loop

```
for each task in plan:
    select executor (quota + strength)  → roster
    hand off one task
    executor implements + commits + stops
    checkpoint-verification   (inspect call-site + real runtime path)
    convention-commit-gate    (enums, no magic literals, commit style)
    fix or re-dispatch if a gate fails
when a risky area is complete, before merge:
    adversarial-review-to-go  (converge findings to GO)
then:
    superpowers:finishing-a-development-branch
```

## Red Flags

| Thought | Reality |
|---------|---------|
| "The executor said tests pass, ship it" | Run `checkpoint-verification` yourself. Summaries hide skipped links. |
| "Let it do the next task too, this one looks fine" | One task per checkpoint. Compounding unreviewed work compounds bugs. |
| "I'll just assign it, quota is probably fine" | Check quota first. Silent quota failure looks like 'did nothing'. |
| "This tiny mechanical change — delegate it" | If it's fully specified, you're faster than the round-trip. |
| "The executor is wrong, override it" | When an executor stops to question the plan, it's often right. Verify against source before dismissing. |
