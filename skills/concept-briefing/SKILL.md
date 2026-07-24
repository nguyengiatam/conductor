---
name: concept-briefing
description: Use as the first step before superpowers:brainstorming on any new request — captures a short scale/business-criticality signal so design depth and executor handoffs calibrate to the actual size of the work instead of treating every request the same.
---

# Concept Briefing

Before design begins, capture one thing: how big is this, really? A short,
persisted calibration signal — scale, business-criticality, and what the
requester actually expects — that `superpowers:brainstorming` and
`orchestrating-executors` both read so their depth matches the work, not a
fixed default.

This is deliberately narrow. It does NOT capture shared vocabulary,
architecture decisions, or task dependencies — those already live in the
spec/plan that `superpowers:brainstorming`/`writing-plans` produce, later.
Duplicating them here would just create a second, staler copy.

## What Goes In the Brief

- **Quy mô ước lượng:** nhỏ / vừa / lớn, with one reason.
- **Mức độ nghiệp vụ:** cơ học/ít rủi ro | có nghiệp vụ | business-critical, with one reason.
- **Kỳ vọng người yêu cầu:** any expectation the request itself signals (fast/minimal vs thorough/robust); if silent, default to what the scale line implies.
- **Hàm ý hiệu chỉnh** for each downstream stage: how many approaches / how deep the questions for brainstorming; how tight the checkpoint loop and how much Claude should write itself for orchestrating-executors; whether adversarial-review-to-go needs multiple convergence rounds.

## Creating It

Ask 1–3 lightweight questions — shallower than brainstorming's own
clarifying-question loop — or skip questions entirely and write the
one-line calibration yourself when the request is unambiguous (e.g. "fix a
typo in the README" needs zero questions). This step must never become
overhead on small requests; that would defeat its own purpose.

Confirm with a single line to the user (e.g. "Quy mô: vừa, nghiệp vụ trung
bình. Đúng chưa?") — not a spec-review gate. This is triage, not a design
decision.

## File Location

`docs/superpowers/plans/YYYY-MM-DD-<topic>-concept-brief.md`. Pick `<topic>`
yourself — brainstorming/writing-plans haven't run yet, so no slug exists.
When they do run, reuse the same slug for the spec/plan filenames so the
trio is easy to find together.

## Updating It

When a later checkpoint (`checkpoint-verification`, `adversarial-review-to-go`,
or a mid-plan discovery) reveals the actual scope diverges from this
estimate — a task assumed mechanical turns out to need a real redesign —
update this file yourself, without waiting to be asked, then carry the new
calibration into subsequent handoff prompts.

## Handoff to the Rest of the Arc

- `superpowers:brainstorming` reads the calibration to decide how many
  approaches to propose and how much to probe before presenting a design.
- `orchestrating-executors` quotes the calibration line in every handoff
  prompt (see its Handoff Prompt Checklist). If no brief exists (Conductor
  adopted mid-project), infer calibration from the plan's Architecture
  section or ask the user directly — a soft fallback, not a blocker.

## Red Flags

| Thought | Reality |
|---------|---------|
| "This is obviously small, I'll just start brainstorming" | Say so in one line and move on — but write the line. An unstated assumption is what drifts. |
| "I'll capture the vocab/architecture/dependency map here too" | Out of scope. Those belong in the spec/plan, not this file — don't duplicate them. |
| "The estimate was wrong, I'll just proceed" | Update the brief first. A stale calibration miscalibrates every handoff after it. |
| "Small task, but let me ask 5 questions to be safe" | Match the questions to the scale. Over-asking on small work is the failure mode this skill exists to prevent. |
