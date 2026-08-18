---
name: job-finder
description: "Searches LinkedIn, Indeed, ZipRecruiter, Dice, Google Jobs, company career portals (Greenhouse, Lever, corporate ATS), and job aggregators for open roles and builds an interactive HTML dashboard of results. Use this whenever the user asks to find jobs, search for openings, look for leadership/IC/technical-product roles, wants a job search dashboard, wants to expand/refresh a previous search, or wants roles matched against their resume — even if they don't name a specific site."
---

# Job Finder

Searches multiple job boards for open roles matching the user's target role(s) and location, optionally ranks/highlights matches against a resume, and renders the results as an interactive HTML dashboard artifact.

## Step 0: Resuming or expanding a previous search

If the user references a prior job-finder run ("resume the session", "refresh the dashboard", "find more jobs", "increase your search"):
- Use `mcp__session_info__list_sessions` / `read_transcript` to recover the original role/location/resume context if it isn't in the current conversation.
- If a dashboard HTML file from a prior run is referenced but not directly readable (different session's output folder), don't block on it — just re-derive parameters from the session transcript and re-run the search fresh. Never assume old results are cached anywhere.
- For "find more" / "increase search" requests: pull **additional pages** from paginated sources you already used (e.g. `?page=2`, `?page=3` on Built In or Wellfound listing pages), add **new query variants** (synonyms like "Head of Engineering", "Engineering Director" vs "Director of Engineering"), and/or add **new sources** you hadn't tried yet (see Step 2). Read the existing dashboard's jobs array back out (`grep`/`node -e` for `const jobs = [...]`) before appending, so you can dedupe by URL/company+title and avoid repeats.
- When appending to an existing dashboard file, prefer `Edit` to insert new entries before the closing `];`, then re-run a quick Node/bash script to parse the array back out and assert the total count and unique-URL count — don't trust it silently.

## Step 1: Gather search parameters

Before searching, make sure you have:
- **Role/keywords** — e.g. "Engineering Manager", "Product Manager", "Software Engineer". If the user hasn't specified, ask, or default to a broad sweep across their likely categories: **Leadership/management**, **Individual contributor**, and **Technical Product** roles based on context clues (their resume, past conversation, or explicit request).
- **Location** — city/remote preference. Ask if not given; don't assume.
- **Resume (optional)** — only use resume-based filtering/ranking if the user has attached or previously shared a resume in this conversation. Do NOT ask them to upload one if they haven't offered — just search broadly without ranking if no resume is present.
  - If a resume file is referenced, read it. For `.docx` files the `Read` tool will fail on the binary — extract text via a quick Python snippet in the sandbox instead: unzip the docx and parse `word/document.xml` for `<w:t>` run text (`zipfile` + `xml.etree.ElementTree`, joining paragraph `<w:p>` text nodes). Don't give up after one failed `Read` attempt.
  - Extract: target titles, seniority level, key skills, years of experience, industry/domain specialization (e.g. payments, healthcare, adtech), and notable achievements (leadership scope, team size, revenue/scale impact) to use for matching later.

## Step 2: Search each source

Search across these sources. Not every source needs to return results — skip any that come back empty rather than forcing results. Note reliability characteristics below; they determine how much you can trust a raw `web_fetch`.

- **ZipRecruiter** — a connected MCP tool (`search_jobs`) is available; load via `tool_search` (e.g. query "ziprecruiter jobs search"). Plain `job_role` queries on senior titles return a lot of noise (unrelated industries/levels). Narrow with `seniority_classes: ["SENIOR"]` and a `salary_min` floor (e.g. 180000 for VP/Director-level searches) to cut junk results. Run several calls varying `job_role` phrasing (e.g. "VP Engineering", "VP Software Engineering", "Head of Engineering", "Director of Engineering") and `location`/`location_types` (metro + REMOTE) rather than one broad call. Capture the exact `job_redirect_url` (with `match_token`) per listing — don't substitute a generic search-page URL for multiple jobs, they won't be unique. Note: these `job_redirect_url` values are very long and can exceed `web_fetch`'s URL-length limit, so they generally can't be re-verified later (see Step 5).
- **Built In / Wellfound** — these are server-rendered and fetch cleanly with `web_fetch`. Use their search/filter URLs (e.g. `builtinsf.com/jobs/dev-engineering/search/<role-slug>`, `wellfound.com/role/l/<role>/<location>`) and paginate with `?page=2`, `?page=3` for more volume. This is also the most reliable source to re-verify later: closed/removed postings render "Sorry, this job was removed" instead of the job body.
- **Greenhouse (`job-boards.greenhouse.io`) and Lever (`jobs.lever.co`)** — search `site:job-boards.greenhouse.io "<role>"` / `site:jobs.lever.co "<role>"` to find company career-portal listings directly; these render server-side and fetch reliably with full JD text and pay ranges. This is the best path for "search company career pages" requests, including for companies not explicitly named by the user (search by role + industry instead, e.g. "VP Engineering payments site:job-boards.greenhouse.io").
- **LinkedIn** — `web_search` with `site:linkedin.com/jobs` surfaces real postings and aggregate counts, but **individual `linkedin.com/jobs/view/...` pages are JS-rendered and return empty via `web_fetch`** — don't keep retrying them. Use the aggregate search snippets (titles, companies, counts) as directional signal, and say explicitly in your summary that LinkedIn line-items couldn't be pulled if that's the case, rather than fabricating listing details.
- **Indeed, Dice, Monster, SimplyHired, Glassdoor** — Dice's search results pages fetch fine with individual listings. Indeed/Monster/Glassdoor/SimplyHired job-view pages are mostly JS-rendered too (empty shells) or return oversized listing pages that need `grep`/chunked reading rather than a raw dump; treat them the same as LinkedIn — good for aggregate market signal (job counts, salary ranges, notable companies hiring), unreliable for pulling specific current listings. Don't spend more than one or two fetch attempts per aggregator before moving on.
- **Corporate ATS portals (Workday, Adyen/Mastercard/PayPal-style custom career sites)** — often JS-rendered SPAs; a direct `web_fetch` on a specific `careers.<company>.com/job/...` URL frequently returns just the site shell with "0 jobs found". When `web_search` snippets already contain enough detail (title, location, comp range, req ID) to be confident the listing is current, you can include it citing the search snippet — but if a listing shows "job was removed" or you can't fetch it directly, treat it as stale and drop it rather than guessing.
- **Company career pages (general)** — if the user names specific target companies, search `"<company> careers <role>"` and try the Greenhouse/Lever path first (most reliable), falling back to the corporate site.

For each job found, capture what's available (not all fields exist for every source):
- Title, company, short description (1-2 sentences, paraphrased — never quote job postings verbatim per copyright rules)
- Pay range (if listed)
- Location / remote status
- Source portal
- Easy Apply available (true/false) — LinkedIn/ZipRecruiter often expose this
- Apply link (prefer the direct/official link; fall back to the portal listing link) — must be a unique, specific-listing URL, not a generic search page
- Common connections — only surfaces from LinkedIn data; omit this field entirely for other sources rather than guessing

Aim for a reasonably broad sweep (roughly 15-30 jobs across sources) for a first pass; expand well beyond that (multiple pages, more query variants, more sources) when the user explicitly asks to see more or increase the search.

## Step 3: Match against resume (if provided)

If a resume was provided, score **every** job (not just new ones) with a tiered match, using domain-aware heuristics rather than just title overlap:

- **Strong** — the job's title/company/description overlaps with the resume's specific industry/domain (e.g. payments, healthcare, adtech, security) using precise keywords tied to that domain (e.g. for payments: `payment(s)`, `fintech`, `banking`, `sanctions`, `settlement`, `clearing`, `billing`, `transaction`, `fraud` — but avoid generic words like bare "compliance" or "risk" that false-positive across unrelated industries).
- **Good** — the role is a strong seniority/skills-level match (same leadership level, overlapping tech stack — e.g. language/cloud/distributed-systems match) but not in the resume's specific domain, OR it overlaps with a distinct specialty called out in the resume (e.g. AI/LLM/GenAI expertise) even without domain overlap.
- **Stretch** — leadership-level match only, with a domain or functional mismatch (e.g. resume is a software engineering leader but the posting is hardware/robotics/electrical/manufacturing, or is a presales/solutions-engineering/customer-success role rather than core engineering delivery).

Implementation tip: do this as a small scoring pass (keyword regex per tier, applied to `company + title + description`) rather than judging each job individually by eye — it's more consistent across a large job list and easy to re-run when new jobs are appended. Keep the keyword lists domain-specific to *this* resume, not generic.

Also:
- Tag each job with **domain-fit badges** (e.g. "Payments/Fintech fit", "AI/LLM fit") separate from the tier, so the user can see *why* it matched.
- Call out specifically if the role is a **leadership/management position** the resume qualifies for (recurring ask) — surface it prominently.
- Don't fabricate a match — if overlap is weak, say so or tag it Stretch rather than inflating fit.

If no resume was provided, skip ranking; just present all results sorted by recency/relevance.

## Step 4: Build the dashboard

Before writing any code, read `/mnt/skills/public/frontend-design/SKILL.md` for styling guidance if available.

Create a single-file HTML artifact (`Write`, saved to the outputs folder) with:
- A card view of all jobs: title, company, pay, location, source badge, easy-apply badge, level badge (VP/SVP vs Director/Sr. Director etc.), match-tier badge and domain-fit badge(s) if a resume was used, and an "Apply" button/link (opens the apply link in a new tab)
- Filters: by level, location, source, easy-apply only, and **resume match tier** if resume was used
- Sort options: match strength (default sort when a resume was used), pay (high/low), recency
- Keep all data inline in the HTML/JS as a single `const jobs = [...]` array (no browser storage — see artifact rules); this is a one-time snapshot, not a persisted tracker
- Every job object should use a consistent, unique, specific-listing `url` — never a generic search-page URL substituted across multiple jobs (breaks dedup and misleads the user).
- After writing, validate before presenting: parse the `jobs` array back out with `node -e` (or similar) and assert the count, unique-URL count, and match-tier distribution look sane — catches accidental duplicates or broken JSON from manual edits.

Present the file with `present_files` and give a short spoken summary (job count, standout leadership/domain matches if any, and any sources you had to skip due to fetch limitations) — don't repeat the full list in chat text since the dashboard already shows it.

## Step 5: Verifying listings are still open (on request)

If the user asks to check whether existing dashboard listings are still open / accepting applications ("remove closed positions", "check if these are still live"):

- Read the existing dashboard's `jobs` array back out first (`node -e` with a regex match + `eval`/`JSON.parse`) and group by `source`, since verifiability differs sharply by source:
  - **Built In / Wellfound, Greenhouse, Lever, Dice** — fetch reliably with `web_fetch`; a closed/filled listing shows explicit text ("Sorry, this job was removed", a 404, or a redirect to the company's generic career-page root instead of the specific listing) rather than the job body. Treat that as closed and remove it.
  - **ZipRecruiter `job_redirect_url` links** — these are very long (with an embedded `match_token`) and routinely exceed `web_fetch`'s URL-length limit, returning an error rather than content. This is a tool limitation, not evidence the listing is closed — don't remove these based on a fetch failure; disclose the limitation instead.
  - **JS-rendered corporate ATS pages (Mastercard, Adyen, PayPal, Workday-style portals)** — return an empty shell regardless of whether the job is open, per Step 2. Same rule: a failed/empty fetch here is inconclusive, not evidence of closure.
- Given the cost of fetching full job pages, checking all listings in one pass may not be practical for very large dashboards (60+ jobs). Prioritize: (1) any listing whose URL looks malformed or generic (fix or verify these first — they're often a leftover data-quality bug from a hasty append), (2) a representative cross-section of every source type present, rather than exhaustively fetching one dominant source (e.g. Built In) end-to-end if that would blow the context budget.
- When you find a job with a broken/generic URL (e.g. a ZipRecruiter search-page URL reused across multiple entries) but the same role clearly still exists under a specific, fetchable URL from another source you encountered during checking (e.g. it also appears as a live Built In listing for the same company/title/pay range), fix the URL/source in place rather than deleting the entry outright.
- After edits, re-run the count/unique-URL validation from Step 4, then report back plainly: how many were confirmed live, how many were removed as closed (name them), and which ones could not be verified due to tool limitations (name the source and why) — don't imply full certainty you don't have.

## Notes

- Never reproduce full job description text verbatim from listings — always paraphrase per copyright rules, and keep descriptions short (1-2 sentences).
- If a source's listings require login/authentication, are JS-rendered SPAs that return empty shells, or can't be fetched for any other reason, say so explicitly in your summary rather than guessing at content or silently omitting the source.
- Re-run this same workflow whenever the user asks for a refreshed search — don't assume old results are cached anywhere, but do reuse recoverable context (role, location, resume) from prior sessions when the user is clearly continuing one.
