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

```
superpowers:brainstorming        → design the change
superpowers:writing-plans        → task-by-task plan with real code

   ┌─ orchestrating-executors    → pick executor (roster), hand off ONE task
   │     checkpoint-verification  → inspect call-site + drive real runtime path
   │     convention-commit-gate   → enums, no magic literals, commit style
   └─  (loop per task; fix or re-dispatch if a gate fails)

adversarial-review-to-go         → external reviewer, converge findings to GO
superpowers:finishing-a-development-branch → merge / PR / cleanup
```

## When to Use Which Skill

| Situation | Skill |
|-----------|-------|
| Deciding what to build | `superpowers:brainstorming` |
| Turning a spec into tasks | `superpowers:writing-plans` |
| Handing a task to an external agent | `orchestrating-executors` |
| Accepting an executor's result | `checkpoint-verification` |
| Committing at a checkpoint | `convention-commit-gate` |
| Hardening a risky area before merge | `adversarial-review-to-go` |
| Integrating the finished branch | `superpowers:finishing-a-development-branch` |

## Principles That Hold Across All Stages

- Claude does not write business code — it designs, reviews, and verifies.
  (Exception: foundation/concurrency/verification code where precision beats
  delegation.)
- Every finding from any reviewer is re-verified on real source before it is
  applied — never blindly.
- Green tests are never acceptance; the call-site and real runtime path are.
- The executor roster is machine-specific and lives in one file; the skills are
  portable.

If superpowers is not installed, the four delta skills still work standalone —
you just lose the brainstorm/plan/finish bookends this map references.
