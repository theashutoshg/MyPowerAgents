# MyPowerAgents

A personal library of opencode skills/agents, checked in here for reuse and version history.

## Repo layout

Each agent lives in its own top-level folder, named to match the skill:

```
MyPowerAgents/
├── README.md                # this file
├── CLAUDE.md                # repo-wide conventions
├── Job-Finder/
│   └── SKILL.md             # the job-finder skill definition (source of truth)
└── outputs/                 # generated HTML dashboards land here
```

As more agents are added, give each its own folder with a `SKILL.md` at minimum, and update the index below.

## Agent index

| Folder | Skill name | Purpose |
|---|---|---|
| `Job-Finder/` | `job-finder` | Searches LinkedIn, Indeed, ZipRecruiter, Dice, Google Jobs, and company career portals (Greenhouse, Lever, corporate ATS) for open roles; optionally ranks matches against a resume; builds a single-file interactive HTML dashboard. Can also expand/refresh a prior search and audit an existing dashboard for closed/removed listings. |

---

## How to install the skill

The skill is a single `SKILL.md` file placed in opencode's global skills directory:

```
~/.config/opencode/skills/job-finder/SKILL.md
```

### Quick setup (copy from this repo)

```bash
# From the repo root:
mkdir -p ~/.config/opencode/skills/job-finder
cp Job-Finder/SKILL.md ~/.config/opencode/skills/job-finder/SKILL.md
```

That's it. opencode auto-discovers skills from `~/.config/opencode/skills/<name>/SKILL.md` — no registration, no config file changes needed.

### Verifying it's loaded

After copying, restart opencode (or start a new session). The skill should appear in the agent's available skills list. You can confirm by asking:

> "What skills do you have available?"

The `job-finder` skill should be listed with its description.

---

## How to run the skill

### Basic usage (interactive)

Start an opencode session and use natural language. The agent will recognize the intent and load the skill automatically:

```
> Find me senior engineering jobs in San Francisco Bay Area

> Search for Director of Engineering roles, remote preferred

> I'm looking for VP of Engineering positions in fintech

> Find jobs matching my resume and rank them
```

The skill triggers on keywords like: *find jobs*, *search for openings*, *job search dashboard*, *look for roles*, *hiring*, etc. You don't need to name the skill explicitly.

### What the agent will ask

If you haven't specified details, the agent will ask for:

1. **Role/keywords** — e.g. "Engineering Manager", "VP Engineering", "Software Engineer"
2. **Location** — city/area or "Remote"
3. **Resume (optional)** — only if you attach or paste one; not required

### What it produces

An interactive HTML dashboard (`outputs/job-dashboard.html`) with:
- Card view of all found jobs (title, company, pay, location, source, apply link)
- Filters by level, source, location, easy-apply
- Sort by pay (high/low), recency, or company name
- All data inline — open the HTML file in any browser, no server needed

### Example prompts

| Prompt | What happens |
|---|---|
| `Find senior engineering roles in SF` | Broad search across all sources, returns ~30-50 roles |
| `Director of Engineering, remote only` | Filters to director-level remote roles |
| `VP Engineering at fintech companies` | Searches Greenhouse/Lever/Built In for fintech VP roles |
| `Find more jobs` (after a previous search) | Expands with additional pages, new query variants, new sources |
| `Refresh the dashboard` | Re-runs the full search fresh (no cached results) |
| `Check if these listings are still open` | Re-verifies each URL, removes closed listings |
| `Rank these against my resume` | Adds match tiers (Strong/Good/Stretch) and domain-fit badges |

### Resume-based matching

When you provide a resume (paste text, `.txt`, `.md`, or `.docx`), the agent:
1. Extracts your seniority level, key skills, industry/domain, and achievements
2. Scores every job with a **match tier**: Strong / Good / Stretch
3. Tags each job with **domain-fit badges** (e.g. "Payments/Fintech fit", "AI/LLM fit")
4. Surfaces **leadership positions** prominently
5. Sorts by match strength by default

### Refreshing / expanding a search

Use these prompts to build on a prior run:
- `"Find more jobs"` — pulls additional pages, new query synonyms, new sources
- `"Search for product manager roles too"` — adds a new role category to the same dashboard
- `"Remove closed positions"` — re-verifies listings and drops stale ones

---

## How the skill works (architecture)

The skill is a workflow definition, not executable code. opencode's agent reads the `SKILL.md` and follows the steps:

1. **Gather parameters** — role, location, resume (if provided)
2. **Search each source** — parallel `web_search` and `web_fetch` calls across:
   - **Built In / Wellfound** — server-rendered, fetch cleanly, most reliable
   - **Greenhouse / Lever** — direct career-portal listings, fetch reliably with full JD text
   - **LinkedIn** — aggregate search snippets only (individual pages are JS-rendered)
   - **Dice** — search results pages fetch fine
   - **ZipRecruiter** — via MCP tool if available
   - **Indeed / Glassdoor / Monster** — aggregate signal only (JS-rendered)
3. **Match against resume** (if provided) — keyword scoring per tier
4. **Build dashboard** — single-file HTML with inline data, filters, sort
5. **Validate** — parses the `jobs` array back out with `node -e` to assert count and unique URLs
6. **Verify on request** — re-fetches URLs to check if listings are still open

### Source reliability

| Source | Fetches cleanly? | Re-verify later? | Notes |
|---|---|---|---|
| Built In | Yes | Yes | "Sorry, this job was removed" for closed listings |
| Wellfound | Yes | Yes | Same removal text |
| Greenhouse | Yes | Yes | Full JD text and pay ranges |
| Lever | Yes | Yes | Full JD text |
| Dice | Yes | Yes | Search results with individual listings |
| LinkedIn | Snippets only | No | JS-rendered individual pages return empty |
| ZipRecruiter | MCP tool | No | `job_redirect_url` links exceed URL-length limits |
| Indeed/Glassdoor | Snippets only | No | JS-rendered, good for market signal only |
| Corporate ATS (Workday, etc.) | No | No | SPA shells, empty on fetch |

---

## Conventions

- Each `SKILL.md` uses standard opencode skill frontmatter (`name`, `description`) followed by numbered `## Step N` sections describing the workflow, plus a closing `## Notes` section.
- Skill files should stay a faithful, current copy of what's actually running — when a skill is updated in a live session, check the update back into this repo so the two don't drift.
- Keep source-reliability notes up to date in the relevant `SKILL.md` — these are hard-won and expensive to rediscover.
- Commit messages should note *what changed in the agent's behavior*, not just "update skill" (e.g. "job-finder: add Step 5 for verifying listings are still open").

## Change log

- **2026-08-14** — Added `Job-Finder/SKILL.md`: full job-search-and-dashboard skill, including Step 5 for auditing an existing dashboard and removing closed/stale listings.
- **2026-08-17** — Added this README with installation, usage, and architecture docs.
