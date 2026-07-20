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
/plugin install conductor@conductor-dev
```

`conductor-dev` is the marketplace name (from `.claude-plugin/marketplace.json`);
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
/plugin install conductor@conductor-dev
```

Update to the latest pushed version any time with `/plugin marketplace update conductor-dev`.

## Skills

| Skill | Purpose |
|-------|---------|
| `using-conductor` | Index/map of the workflow arc and where it meets superpowers. |
| `orchestrating-executors` | Reviewer/executor split, per-task checkpoint protocol, quota-aware executor selection. |
| `checkpoint-verification` | Refuses green tests as proof; inspect call-site + drive the real runtime path. |
| `adversarial-review-to-go` | External adversarial reviewer in converging rounds to GO; re-verify every finding. |
| `convention-commit-gate` | Centralized enums, no magic literals, project commit style. |

## The Arc

```
brainstorming (SP) → writing-plans (SP)
  → orchestrating-executors ⇄ checkpoint-verification ⇄ convention-commit-gate  (per task)
  → adversarial-review-to-go
  → finishing-a-development-branch (SP)
```

## Porting to another environment

Executor command syntax is NOT baked into the skills. It lives in
`skills/orchestrating-executors/references/executor-roster.md`. Edit that one
file for your machine/agents; the skills stay unchanged.

## Relationship to superpowers

Conductor is a delta. It assumes superpowers is installed for the
brainstorm / plan / finish bookends. The four delta skills also work standalone.
