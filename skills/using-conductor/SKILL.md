---
name: using-conductor
description: Use when starting delegated multi-agent delivery on a project — maps the full workflow arc and points to the right conductor skill and superpowers skill at each stage.
---

# Using Conductor

Conductor is the delivery discipline for work where **Claude architects and
reviews while external coding agents implement.** It is a thin delta over
superpowers — it does not replace `brainstorming`, `writing-plans`, or
`finishing-a-development-branch`; it slots the delegation-and-review loop
between them.

## The Arc

**The arc is elastic.** `concept-briefing` tiers the request and that tier decides
which steps below actually run. What follows is the T2/T3 path — the full one.

```
conductor:pointer-handoff        → (ongoing project) read the pointer, reconcile with reality
conductor:lessons-ledger         → pull only the lessons matching this area + kind of work
conductor:concept-briefing       → confirm system profile, tier the request, route
superpowers:brainstorming        → design the change (steered by the profile)
superpowers:writing-plans        → task-by-task plan with real code

   ┌─ executor-context           → keep the fixed context file current; handoffs point at it
   ├─ orchestrating-executors    → pick executor (roster), hand off ONE task
   │     checkpoint-verification  → inspect call-site + drive real runtime path
   │     convention-commit-gate   → enums, no magic literals, commit style
   └─  (loop per task; fix or re-dispatch if a gate fails; re-tier if scope diverges)

adversarial-review-to-go         → external reviewer, converge findings to GO
superpowers:finishing-a-development-branch → merge / PR / cleanup
conductor:pointer-handoff        → write state + next action before the session ends
```

`pointer-handoff` brackets the whole arc: read at the start, written at the end
and whenever something significant surfaces mid-session. `lessons-ledger` is
consulted before working in an area, and written the moment a diagnosis proves
wrong or verification catches what the tests missed; its project-wide lessons get
crystallized into the `executor-context` file rather than pasted into prompts.

Large layered work inserts one step: `concept-briefing` produces a **roadmap** of
phases from foundation upward, then each phase runs the arc above on its own,
with its own tier. Detailed plans are written per phase, never in advance.

### What runs at each tier

| Tier | Path |
|------|------|
| **T0** mechanical | Do it yourself → `convention-commit-gate`. Nothing else. |
| **T1** small, obvious | No spec, no plan file → executor → `checkpoint-verification` → `convention-commit-gate` |
| **T2** medium | Full arc; `adversarial-review-to-go` only if it touches a risky area |
| **T3** large / critical | Full arc, nothing skipped |

Skipping a step at T0/T1 requires the user's OK — `concept-briefing` asks once,
batched. Verification gates are never skipped when real code gets written.

## When to Use Which Skill

| Situation | Skill |
|-----------|-------|
| Resuming an ongoing project, or closing a session | `pointer-handoff` |
| Starting work in an area, or recording a wrong diagnosis | `lessons-ledger` |
| Profiling the system + tiering the request before design | `concept-briefing` |
| Planning large work as phases from foundation upward | `concept-briefing` (roadmap) |
| Deciding what to build | `superpowers:brainstorming` |
| Turning a spec into tasks | `superpowers:writing-plans` |
| Handing a task to an external agent | `orchestrating-executors` |
| Writing project context executors reuse every dispatch | `executor-context` |
| Accepting an executor's result | `checkpoint-verification` |
| Committing at a checkpoint | `convention-commit-gate` |
| Hardening a risky area before merge | `adversarial-review-to-go` |
| Integrating the finished branch | `superpowers:finishing-a-development-branch` |

## Principles That Hold Across All Stages

- Process depth matches the work. Mechanical work gets no spec; critical work
  gets everything. Measuring size without changing what runs is just ceremony.
- The system profile is confirmed by the user, not inferred and assumed. Scale,
  users, and what wins a trade-off decide architecture — a machine guess there
  propagates into every downstream prompt.
- Claude does not write business code — it designs, reviews, and verifies.
  (Exception: foundation/concurrency/verification code where precision beats
  delegation.)
- Every finding from any reviewer is re-verified on real source before it is
  applied — never blindly.
- Green tests are never acceptance; the call-site and real runtime path are.
- The executor roster is machine-specific and lives in one file; the skills are
  portable.

If superpowers is not installed, the eight delta skills still work standalone —
you just lose the brainstorm/plan/finish bookends this map references.
