# Executor Roster (machine-specific — edit per environment)

This file is the ONLY machine-specific part of the plugin. Replace its contents
with your own executors when moving to a new environment. The
`orchestrating-executors` skill stays unchanged.

Each executor entry documents: **invoke syntax · quota check · strengths · gotchas.**

> The entries below are the market-report / WSL setup (as of 2026-07). Treat
> command paths and quirks as examples, not universal truth.

## Priority order (when multiple have quota)

1. **agy** (Antigravity CLI) — first choice when it has quota.
2. **Codex** / **kiro** — when they have quota; Codex is the review specialist.
3. **opencode + deepseek** — only when everything else is exhausted; split tasks
   smaller and review harder ("not very smart, control tightly").

Codex is preferred for REVIEW (its strength), not for coding when another
executor is available.

## Always check quota for ALL executors before assigning heavy work

Silent quota exhaustion is the classic trap: the tool runs, prints a line or
two of preamble, exits 0 with near-empty output — looks like "ran but did
nothing." Confirm quota first.

- **agy:** `timeout 300 agy --print --dangerously-skip-permissions "Reply one word: ok"`
  — out of quota prints `Error: Individual quota reached... Resets in Xh`.
  Individual quota resets ~every 4–5h.
- **kiro:** `kiro-cli chat --no-interactive "/usage"` — prints % monthly + daily
  credit and reset date (free; overage disabled, so it hard-stops at zero).
- **Codex:** read latest snapshot
  `ls -t ~/.codex/sessions/2026/*/*/rollout-*.jsonl | head -1` then grep
  `"rate_limits"`; `primary.used_percent` is the WEEKLY window. For fresh
  numbers ping `codex exec --json --skip-git-repo-check "ok"` (cheap) then grep.

## agy (executor — mechanical/docs, general implementation)

- **Invoke:** write the task to a scratch file, then
  `agy -c --prompt="read file <path> and do it"` (run via Bash with
  `dangerouslyDisableSandbox: true`, from the repo cwd, long timeout).
- **Critical:** bind the message with `--prompt=` (the `=` matters). Positional
  message args get swallowed into context and agy goes off researching the flag
  instead of doing the task. `-c` continues the existing project session.
- **Model:** `--model "Claude Sonnet 4.6 (Thinking)"` (or current).
- **Gotcha:** headless `--print` with multi-step tasks is unreliable; prefer
  `-c --prompt=` form. Long inline prompts return empty — hand off via file.

## Codex (reviewer-first; heavy-logic executor when needed)

- **Invoke (review/impl):**
  `codex exec --cd <dir> --sandbox danger-full-access --skip-git-repo-check "<prompt>"`.
  Do NOT use the companion wrapper for reviews — its token snapshot goes stale.
- **Strength:** concurrency/TOCTOU/crash-gap and migration-on-deploy findings
  that e2e never surfaces. Run review as a converging multi-round loop (see
  `adversarial-review-to-go`); each round ~50–145k tokens.
- **Gotcha (WSL):** the workspace-write sandbox blocks AF_VSOCK and unshare-net
  (docker/psql/e2e fail). Use `danger-full-access`. Avoid bare backticks in
  prompts (host command-substitution fires before Codex sees them).

## kiro-cli + Sonnet 5 (executor)

- **Invoke:** `kiro-cli chat --model claude-sonnet-5 --trust-all-tools --no-interactive "<prompt>"`,
  run as a background Bash job of the main session. Use a NEW session per task
  (resume a specific session only to continue in-progress context).
- **Gotcha — self-kill:** the prompt text lives in argv; forbid killing
  processes by pattern, and never embed a kill command string in the prompt.
- **Gotcha — silent hang (no token spend):** kiro writing a multi-line
  server-start script, or a `node -e` that touches the DB without
  `process.exit(0)`, holds a pipe waiting for EOF and goes silent 15+ min.
  Require single-line background commands ending in `&`, health-check with a
  SEPARATE curl, and `process.exit(0)` in a finally for any DB script. If kiro
  is silent >5 min, inspect children of `kiro-cli-chat` and kill the pipe holder.
- **Env:** `/mnt/c` (NTFS) has no inotify — watch does not reload; restart the
  dev server after edits. Source `.env.local` before e2e.

## opencode + deepseek-v4 (last resort)

- **Invoke:** `opencode run -m opencode/deepseek-v4-flash-free "<msg>"`
  (`-c`/`--session` to continue, `--agent` to pick agent).
- **Discipline:** split tasks smaller, review harder, never hand it a large block.
