# MyPowerAgents

A personal library of opencode skills/agents, checked in here for reuse and version history.

> **Full usage docs:** See [README.md](README.md) for installation, running, and architecture details.

## Repo layout

Each agent lives in its own top-level folder, named to match the skill:

```
MyPowerAgents/
├── README.md              # installation, usage, and architecture docs
├── CLAUDE.md              # this file — repo-wide conventions
└── Job-Finder/
    └── SKILL.md            # the job-finder skill definition
```

As more agents are added, give each its own folder (e.g. `Job-Application-Assistant/`, `Morning-Brief/`) with a `SKILL.md` at minimum, and update the index below.

## Agent index

| Folder | Skill name | Purpose |
|---|---|---|
| `Job-Finder/` | `job-finder` | Searches LinkedIn, Indeed, ZipRecruiter, Dice, Google Jobs, and company career portals (Greenhouse, Lever, corporate ATS) for open roles; optionally ranks matches against a resume; builds a single-file interactive HTML dashboard. Can also expand/refresh a prior search and audit an existing dashboard for closed/removed listings. |
| `Interview-Practice-Coach/` | `interview-practice-coach` | Runs a mock behavioral/technical interview from a pasted job description: asks one competency-based question at a time, coaches answers into STAR-format stories against a consistent rubric, and pressure-tests them with likely follow-ups. Model-agnostic workflow. |

## Conventions

- Each `SKILL.md` uses standard Claude skill frontmatter (`name`, `description`) followed by numbered `## Step N` sections describing the workflow, plus a closing `## Notes` section for cross-cutting caveats.
- Skill files should stay a faithful, current copy of what's actually running — when a skill is updated in a live session (e.g. via `save_skill`), check the update back into this repo so the two don't drift.
- Keep source-reliability notes (which job boards/sites fetch cleanly vs. which are JS-rendered/unreliable) up to date in the relevant `SKILL.md` — these are hard-won and expensive to rediscover.
- Commit messages should note *what changed in the agent's behavior*, not just "update skill" (e.g. "job-finder: add Step 5 for verifying listings are still open").

## Change log

- **2026-08-14** — Added `Job-Finder/SKILL.md`: full job-search-and-dashboard skill, including a new Step 5 for auditing an existing dashboard and removing closed/stale listings (with explicit notes on which sources can/can't be reliably re-verified: Built In/Greenhouse/Lever/Dice fetch cleanly, ZipRecruiter redirect URLs and JS-rendered ATS portals like Mastercard/Adyen cannot be confirmed via automated fetch).
- **2026-08-17** — Added README.md with installation, running, architecture, and source-reliability docs.
- **2026-08-18** — Added `Interview-Practice-Coach/SKILL.md`: mock-interview skill that reads a pasted JD, asks one competency question at a time, and coaches answers into STAR stories; reformatted from its packaged `.skill` export into this repo's `## Step N` / `## Notes` convention and made explicitly model-agnostic (no hardcoded model name).
