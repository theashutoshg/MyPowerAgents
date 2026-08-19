---
name: github-profile-builder
description: "Creates or updates a GitHub Pages profile site (index.html) hosted at <user>.github.io, OR a GitHub Profile README bio page (README.md) that appears at github.com/<user>. Use this whenever the user wants to build, refresh, or redesign their public GitHub profile page, add a new section (featured work, skills, experience, stats), create a profile README bio, or regenerate either page from their resume/LinkedIn data — even if they don't mention GitHub Pages or profile README by name."
---

# GitHub Profile Builder

Creates a single-file, self-contained `index.html` profile page for a GitHub Pages site hosted at `<username>.github.io`. The page is a responsive, modern HTML card layout with inline CSS — no build tools, no external dependencies beyond Google Fonts and Font Awesome CDN.

## Step 0: Determine mode — create vs. update

- **GitHub Pages site (create)** — the user does not yet have a `<username>.github.io` repo. Guide them through setup (see Step 6) and then generate the page.
- **GitHub Pages site (update)** — the user already has a profile page and wants to change content, add sections, or restyle it. Read the existing `index.html`, identify what to change, and apply edits in place. Always preserve the existing design language (colors, gradients, card styles) unless the user explicitly asks for a redesign.
- **Profile README (create/update)** — the user wants a Markdown bio on their GitHub profile page (`github.com/<username>`). See Step 7 for setup and generation instructions.
- **Both** — the user wants a complete GitHub presence: profile README for the quick bio + Pages site for the full portfolio.

Ask the user which mode they need if it isn't obvious from context.

## Step 1: Gather profile data

Collect the following from the user (skip any they don't want to include):

- **Name** — displayed as the page heading.
- **Title / tagline** — e.g. "AI Leader · Engineering Director · Generative AI & LLM Architectures".
- **Location** — city / metro area.
- **Contact links** — LinkedIn URL, email, Twitter/X, personal website, etc.
- **Bio / About** — 1-3 paragraphs summarising who they are and what they do. If the user pastes a resume or LinkedIn export, extract and condense this from the summary/experience sections. Never copy blocks verbatim — paraphrase for a public-facing profile.
- **Stats / Achievements** — quantified highlights (years of experience, revenue impact, cost savings, team sizes, scale metrics). Each stat needs a number and a short label.
- **Skills** — grouped into categories (e.g. "Leadership & Domain Expertise", "Technical Skills"). Use short tag-style labels (2-5 words each).
- **Experience** — role title, company, date range, and a 2-3 sentence summary per position. Most recent first.
- **Featured Work** — cards linking to case studies, blog posts, repos, calculators, or any artifacts. Each needs a title, short description (1 sentence), icon class (Font Awesome), and URL.
- **GitHub repos to feature** — if the user wants to showcase specific repos, pull the repo description from GitHub and add a card linking to it.
- **Social links** — LinkedIn, GitHub profile, Twitter, etc. displayed as buttons at the bottom.

If the user provides a resume file (`.docx`, `.pdf`, `.txt`), read and extract data from it. For `.docx` files, use a Python snippet to parse `word/document.xml` for `<w:t>` text runs. For `.pdf` files, try `pdftotext` or a Python PDF library.

## Step 2: Choose layout and design

Use the existing profile page template as the baseline unless the user requests a different design:

- **Header** — dark gradient background (`#1a1a2e` to `#16213e`), name in gradient text, tagline, contact info row.
- **Content** — white card with sections separated by `section-title` headings with a left-border accent.
- **Stats** — grid of gradient cards with large numbers and labels.
- **Skills** — flex-wrapped pill/tag badges.
- **Experience** — timeline layout with dots and bordered content cards.
- **Featured Work** — grid of gradient cards with icons, titles, and descriptions. External links open in `new tab`.
- **Social Links** — centered gradient pill buttons with icons.
- **Footer** — dark background, copyright line.

Design tokens (default palette):
- Primary gradient: `linear-gradient(135deg, #667eea 0%, #764ba2 100%)`
- Header bg: `linear-gradient(135deg, #1a1a2e 0%, #16213e 100%)`
- Card bg: `white`
- Body bg: same primary gradient
- Font: Inter (Google Fonts)
- Icons: Font Awesome 6.4 CDN

If the user provides their own colour scheme or design reference, adapt accordingly.

## Step 3: Generate the page

Write the complete `index.html` to the repo root. Requirements:

- Single-file, self-contained — all CSS is inline in a `<style>` block in `<head>`. No external CSS files.
- Responsive — media query at 768px for mobile (smaller heading, single-column stats, reduced padding).
- Semantic HTML — use `<section>`, `<h2>`, `<p>`, `<a>` appropriately.
- External links in Featured Work and Social Links must have `target="_blank"`.
- Never hardcode secrets, API keys, or private URLs.
- Keep the file under 400 lines when possible — collapse repeated tag spans onto fewer lines if the skills list is very long.
- Include proper `<!DOCTYPE html>`, `<meta charset>`, and viewport meta tag.

## Step 4: Validate the output

After writing the file:

1. Verify the HTML is well-formed — check for unclosed tags, matching quotes, and correct nesting. A quick `grep -c '<div' index.html` vs `grep -c '</div>' index.html` should match.
2. Confirm all internal links (e.g. `Case_Study.html`, `ROI_Calculator.html`) reference files that exist in the repo. Use `ls` to check. If a linked file is missing, note it in the summary but don't block on it — the user may plan to add it later.
3. Confirm external URLs are well-formed (start with `https://`).

## Step 5: Summarise changes

Tell the user:
- What was created or changed (sections added/modified).
- The file path (`index.html` at repo root).
- Any missing linked files they may need to add.
- Remind them to commit and push to GitHub for the page to go live.

## Step 6: GitHub Profile Page Setup Instructions

If the user does not yet have a GitHub Pages profile repo, walk them through this one-time setup. Provide these instructions clearly and offer to execute the git commands for them.

### What is a GitHub Profile Page?

GitHub lets you create a special repository named `<your-username>.github.io`. The `index.html` at the root of the `main` branch in this repo is automatically served as a public website at `https://<your-username>.github.io/`. This is your personal profile page — separate from your GitHub profile README.

### One-Time Setup

**Option A: Create via GitHub CLI (recommended)**

```bash
# Replace <username> with your GitHub username
gh repo create <username>.github.io --public --description "My GitHub Pages profile"
```

Then clone it locally:
```bash
git clone https://github.com/<username>/<username>.github.io.git
cd <username>.github.io
```

**Option B: Create via GitHub Web UI**

1. Go to https://github.com/new
2. Repository name: `<your-username>.github.io` (must match exactly)
3. Set to **Public**
4. Check "Add a README file"
5. Click "Create repository"

Then clone it:
```bash
git clone https://github.com/<username>/<username>.github.io.git
cd <username>.github.io
```

### Enable GitHub Pages

1. Go to your repo on GitHub → **Settings** → **Pages** (left sidebar)
2. Under **Source**, select "Deploy from a branch"
3. Branch: `main`, folder: `/ (root)`
4. Click **Save**
5. Wait 1-2 minutes. Your site is live at `https://<your-username>.github.io/`

### Making Updates

After editing `index.html` locally:
```bash
git add index.html
git commit -m "Update profile page: <describe change>"
git push origin main
```

Changes appear on the live site within 1-2 minutes.

### Optional: Custom Domain

If you own a domain, add a `CNAME` file to the repo root containing your domain, then configure DNS to point to `<username>.github.io`.

## Step 7: GitHub Profile README Bio Page

In addition to the GitHub Pages site (`<username>.github.io`), GitHub supports a **profile README** — a special `README.md` that appears at the top of your public profile page (`github.com/<username>`). This is separate from the Pages site and is great for a quick bio, badges, and pinned content.

### What is a GitHub Profile README?

GitHub has a hidden feature: if you create a **public repository with the exact same name as your username** (e.g. `theashutoshg/theashutoshg`), the `README.md` in that repo is rendered on your GitHub profile page at `https://github.com/<username>`. It appears above your pinned repos.

This is the fastest way to add a bio to your GitHub presence — no hosting, no DNS, no build. Just a Markdown file.

### One-Time Setup

**Option A: Create via GitHub CLI**

```bash
# Create a repo matching your username
gh repo create <username>/<username> --public --description "My profile README"
```

Then clone it:
```bash
git clone https://github.com/<username>/<username>.git
cd <username>
```

**Option B: Create via GitHub Web UI**

1. Go to https://github.com/new
2. Repository name: **exactly** `<your-username>` (e.g. `theashutoshg`)
   - GitHub will show a hint: *"theashutoshg/<theashutoshg> is a special repository. Its README.md will appear on your GitHub profile!"*
3. Set to **Public**
4. Check "Add a README file"
5. Click "Create repository"

Then clone it:
```bash
git clone https://github.com/<username>/<username>.git
cd <username>
```

### What to Put in the Profile README

A good profile README is concise (under 80 lines of Markdown). Recommended structure:

```markdown
# Hi there, I'm <Name> 👋

## About Me

- **Role:** <Title> at <Company>
- **Focus:** <1-2 sentence summary of what you do>
- **Location:** 📍 <City>
- 🔭 Currently working on <project or focus area>
- 🌱 Learning <what you're exploring>
- 💬 Ask me about <topics you're happy to discuss>
- 📫 How to reach me: [LinkedIn](https://linkedin.com/in/<you>) | [Email](mailto:<you>@<domain>)

## GitHub Stats

<!-- Use https://github.com/anuraghazra/github-readme-stats for live stats -->
![Ashutosh's GitHub stats](https://github-readme-stats.vercel.app/api?username=<username>&show_icons=true&theme=radical)

## Featured Projects

| Project | Description |
|---------|-------------|
| [Repo Name](https://github.com/<you>/repo) | Short description |

## Tech Stack

`Java` `Spring Boot` `AWS` `Kafka` `Python` `LLMs` `RAG` `Agentic AI`
```

### Using Profile Readme Generator Tools

These free tools generate profile README content from a form:

- **https://rahuldkjain.github.io/github-profile-readme-generator/** — fill in fields, get copy-paste Markdown
- **https://www.canopas.com/github-profile-readme-generator** — alternative generator with more layout options
- **https://github.com/abbiepalac13/Profile-README-Generator** — simple and clean

### Adding Live GitHub Stats

Use these badge generators to add auto-updating stats to the README:

- **GitHub Readme Stats:** `https://github-readme-stats.vercel.app/api?username=<username>&show_icons=true&theme=radical`
- **GitHub Streak Stats:** `https://github-readme-streak-stats.herokuapp.com/?user=<username>&theme=radical`
- **GitHub Top Skills:** `https://github-readme-stats.vercel.app/api/top-langs/?username=<username>&layout=compact`

### Making Updates

After editing `README.md` locally:
```bash
git add README.md
git commit -m "Update profile README: <describe change>"
git push origin main
```

Changes appear on your GitHub profile page within seconds.

### Profile README vs. GitHub Pages Site

| Feature | Profile README | GitHub Pages Site |
|---------|---------------|-------------------|
| **URL** | `github.com/<username>` | `<username>.github.io` |
| **Format** | Markdown (`README.md`) | HTML (`index.html`) |
| **Styling** | GitHub-flavored Markdown + badge images | Full CSS control, custom design |
| **Hosting** | Automatic (part of GitHub) | GitHub Pages (free static hosting) |
| **Best for** | Quick bio, stats badges, pinned repos | Full portfolio with custom layout |
| **Setup time** | 5 minutes | 15-30 minutes |

**Recommendation:** Set up both. The profile README gives visitors a quick overview on GitHub itself. The Pages site is your full portfolio with custom design, case studies, and featured work.

## Notes

- The skill produces a **single HTML file** — no build step, no frameworks. This keeps the page fast, portable, and easy to edit by hand.
- The page uses CDN-hosted Google Fonts and Font Awesome. If the user needs offline support, offer to inline the font files (larger file size).
- For users with many repos to feature, suggest a "Projects" or "Open Source" section using the same card grid pattern.
- If the user asks for a dark-mode toggle or animations beyond basic hover effects, implement them as progressive enhancements in inline `<script>` — but keep the page functional without JavaScript.
- When updating an existing page, always `Read` the file first. Never overwrite without understanding the current structure.
