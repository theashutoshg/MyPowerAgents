# Interview-Practice-Coach

Folder-level conventions for this agent. Repo-wide conventions live in the root [CLAUDE.md](../CLAUDE.md); this file only covers things specific to `interview-practice-coach`.

## What this is

A mock-interview skill: given a pasted job description, the agent plays interviewer — one question at a time — then coaches each answer into a tight STAR-format story against a fixed rubric. See [SKILL.md](SKILL.md) for the full step-by-step workflow, and [README.md](README.md) for installation/usage.

## Conventions specific to this skill

- The skill must stay **model-agnostic**: no hardcoded references to a specific model or vendor in the prompt text. It describes a workflow (read JD → ask → coach → iterate), not model-specific behavior.
- Never let the coaching step invent facts, numbers, or examples the user hasn't actually given — if a story is too thin to draft, the workflow asks clarifying questions first (Step 3) rather than fabricating detail.
- The "push back on self-focused motivation framing" rule (Notes in SKILL.md) is a deliberate, recurring behavior — don't soften it to a one-time correction.

## Change log

- **2026-08-18** — Converted from the packaged `conventions/interview-practice-coach.skill` export into this repo's `## Step N` / `## Notes` SKILL.md convention; removed the hardcoded model reference to make the skill model-agnostic.
