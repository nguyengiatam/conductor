# Conductor

A Claude Code plugin packaging a **multi-agent delivery discipline**: Claude
architects and reviews while external coding agents (agy, Codex, kiro,
opencode, …) implement — under adversarial review-to-GO and real-runtime
verification. It extends [superpowers](https://github.com/obra/superpowers)
rather than replacing it.

## Install

From GitHub (recommended):

```
/plugin marketplace add https://github.com/nguyengiatam/conductor.git
/plugin install conductor@conductor-marketplace
```

`conductor-marketplace` is the marketplace name (from `.claude-plugin/marketplace.json`);
the `@<marketplace>` qualifier is **required** on install. `conductor` is the
plugin name.

The GitHub `owner/repo` shorthand also works, but it clones over SSH by default —
set `CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1` to clone over HTTPS instead:

```
/plugin marketplace add nguyengiatam/conductor
```

From a local clone (no network):

```
/plugin marketplace add /path/to/conductor
/plugin install conductor@conductor-marketplace
```

Update to the latest pushed version any time with `/plugin marketplace update conductor-marketplace`.

## Skills

| Skill | Purpose |
|-------|---------|
| `pointer-handoff` | One short pointer file per project: current state + next action. Read on resume, written before the session ends. |
| `lessons-ledger` | Per-project lessons indexed by code area and work type, so only the relevant ones load — and get quoted into executor prompts. |
| `concept-briefing` | Locks a user-confirmed system profile, tiers each request, and routes it to the right amount of process — including a phased roadmap for layered work. |
| `using-conductor` | Index/map of the workflow arc and where it meets superpowers. |
| `orchestrating-executors` | Reviewer/executor split, per-task checkpoint protocol, quota-aware executor selection. |
| `executor-context` | One fixed context file the coordinator maintains, so handoffs point at it instead of retyping conventions. |
| `checkpoint-verification` | Refuses green tests as proof; inspect call-site + drive the real runtime path. |
| `adversarial-review-to-go` | External adversarial reviewer in converging rounds to GO; re-verify every finding. |
| `convention-commit-gate` | Centralized enums, no magic literals, project commit style. |

## The Arc

The arc is **elastic** — `concept-briefing` tiers each request and the tier decides
which steps run. Below is the full T2/T3 path:

```
pointer-handoff (resume) → lessons-ledger (what applies here?)
  → concept-briefing → brainstorming (SP) → writing-plans (SP)
  → orchestrating-executors ⇄ checkpoint-verification ⇄ convention-commit-gate  (per task)
  → adversarial-review-to-go
  → finishing-a-development-branch (SP)
  → pointer-handoff (record state + next action)
```

| Tier | What runs |
|------|-----------|
| **T0** mechanical (typo, constant, rename) | Do it directly → `convention-commit-gate` |
| **T1** one obvious way, 1–3 files | No spec, no plan file → executor → verification gates |
| **T2** ≥2 approaches, or touches schema/API | Full arc; adversarial review only for risky areas |
| **T3** money/settled figures, broken boundaries | Full arc, nothing skipped |

Skipping steps is always the user's call — asked once, batched. Verification gates
are never skipped when real code gets written.

Large layered work gets a **roadmap** first: phases from foundation upward, each
standing on the last, with detailed plans written per phase rather than in advance.

## Porting to another environment

Executor command syntax is NOT baked into the skills. It lives in
`skills/orchestrating-executors/references/executor-roster.md`. Edit that one
file for your machine/agents; the skills stay unchanged.

## Relationship to superpowers

Conductor is a delta. It assumes superpowers is installed for the
brainstorm / plan / finish bookends. The eight delta skills also work standalone.
