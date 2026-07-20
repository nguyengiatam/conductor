---
name: adversarial-review-to-go
description: Use after a risky area is implemented and before merge — runs an external adversarial reviewer in converging rounds until findings reach zero (GO), while independently re-verifying every finding on real source before acting.
---

# Adversarial Review to GO

For design-sensitive or concurrency-sensitive work, one review pass is not
enough. Run an external adversarial reviewer (the reviewer-strong agent in the
roster) in **converging rounds**: each round produces findings, you patch the
valid ones minimally, re-verify, and re-review — until a round produces zero
surviving findings ("GO"). Convergence looks like 5 → 2 → 1 → 0.

This is the delegated-reviewer counterpart to
`superpowers:receiving-code-review` — apply that skill's discipline (act on
valid feedback, push back with reasoning on invalid feedback) to an external
agent's output.

## The Golden Rule

**Always re-verify each finding against the real source yourself. Never apply
findings blindly.** The external reviewer is fast and catches things e2e can't
(crash-gap, TOCTOU, migration-on-deploy), but it is also sometimes wrong or
wrong about severity. For each finding:

1. Confirm it on the actual code (reproduce the reasoning at the cited site).
2. If valid — patch minimally (prefer a CAS/guard at the exact contention point
   over a broad rewrite), then re-verify with `checkpoint-verification`.
3. If invalid or overstated — push back with the code/test that disproves it,
   and record why it was rejected.
4. Surface rejected findings to the user for a decision when they involve a
   real trade-off (e.g. distributed primitive vs in-process guard).

## The Round Loop

```
round = 1
repeat:
    dispatch external reviewer on the current diff (self-contained prompt:
        point at plan + diff scope; fresh thread if prior context is huge)
    for each finding: re-verify on source → patch-if-valid / rebut-if-not
    re-run checkpoint-verification (2 consecutive green where applicable)
    round += 1
until a round yields zero surviving findings  → GO
```

## Practical Notes

- Keep each review prompt self-contained (plan pointer + diff scope) so a fresh
  reviewer thread works — don't rely on a giant resumed context.
- The reviewer may run out of quota mid-loop. A purely formal confirmation round
  can be skipped if the user agrees; already-verified mechanical fixes don't
  need another round.
- Record the convergence (e.g. "4 rounds, 5→2→1→0") and the accepted residual
  trade-offs in the merge commit or plan notes.

## Red Flags

| Thought | Reality |
|---------|---------|
| "Reviewer flagged it, just apply it" | Re-verify on source first. Reviewers are sometimes wrong. |
| "One round was clean enough" | Converge to zero. The last round is where subtle ones surface. |
| "Rewrite the whole thing to be safe" | Patch minimally at the contention point. Broad rewrites add risk. |
| "Rejecting this finding, moving on" | If it's a real trade-off, the user decides — surface it. |
