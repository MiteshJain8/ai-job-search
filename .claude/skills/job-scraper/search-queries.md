# Search Queries for Job Scraper

## Save every search to Notion

User instruction: store searched results in Notion after every job-search run.
- Database: https://app.notion.com/p/41c4e0fd96334efe9fd746bd9c7507d6
- Data source: collection://06f04a51-bf8f-4138-9366-1040d4741550
- Fetch the schema and query existing rows before writing. Match by normalized Job URL or employer requisition ID; update existing matches rather than duplicating them.
- Store presented opportunities with company, role, location, source URL, experience requirement, search-relative rank, actual last-check date, fit notes, gaps, and shift caveats. Do not fabricate scores or dates.
- Preserve user-maintained application statuses. Record rank context in content when updating ranks from another search. Do not mark a role Applied merely because it was saved.
- Continue local scrape-state deduplication and storage as defined by the canonical scraper workflow; Notion supplements it.
- Report the Notion link and saved/updated counts. If syncing fails, report it explicitly and keep local results available for retry.

<!-- SETUP: Customize these queries based on your skills, target roles, and location -->

## Installed portal CLIs (primary for `/scrape`)

`/scrape` discovers every portal skill under `.agents/skills/*/SKILL.md` and runs its CLI first. Shipped country-agnostic CLIs include `linkedin-search` and `freehire-search`; Danish demos and any skill you add with `/add-portal` are included the same way. You do **not** need a matching `site:` line below for those CLIs to run.

The `site:` query templates in this file are the **WebSearch fallback** — for portals without a CLI, company career pages, or when a CLI fails.

**Language scope:** write every query category in every language listed in your CLAUDE.md Languages table (typically 1-2, sometimes more). A posting requiring a language you have *not* declared, as a job condition, is excluded before scoring; a posting requiring a *higher level* than you declared in a language you *do* work in is flagged for your own judgment, not excluded — see `04-job-evaluation.md`'s Language Gate, the single source of truth for this rule. Translate each category's keywords rather than machine-translating word-for-word (e.g. "Frontend Developer" -> "Desarrollador Frontend", not a literal word-for-word translation) if you work in more than one language.

## Search Sites

Primary (your market's job boards - scaffold one with `/add-portal`):
- **naukri.com** - India general job board
- **linkedin.com/jobs** - LinkedIn job listings (filter: India / Bengaluru); also covered by `linkedin-search` CLI
- **wellfound.com/jobs** - startup roles (optional)
- **instahyre.com** - India technical roles (optional)

Secondary (company career pages via Google):
- Direct Google searches with `site:` filters for known target companies

## Query Categories

Queries are grouped by priority. Write **each category in every language from your Languages table** (see Language scope above). Combine each query with your location terms (e.g. your city, region, or metro area) where the site supports it.

**Organize by function, not job title.** The same underlying work carries different titles across companies and markets (a "Data Scientist" role at one employer may be posted as "Insights Analyst" or "Data Consultant" at another). Name each priority category after the function it covers, and list several plausible job titles as query variants within that category rather than betting an entire priority tier on one exact title string.

### Priority 1: Equal-priority entry-level engineering tracks

Backend, applied AI, full-stack, and DevOps/platform roles are equally ranked.

```
site:naukri.com "backend engineer" Bengaluru
site:naukri.com "software engineer" Bengaluru
site:naukri.com "applied ai engineer" India
site:linkedin.com/jobs "software engineer" India
```

### Priority 2: Backend and applied-AI systems

These match the strongest documented technical evidence.

```
site:naukri.com "python fastapi" Bengaluru OR India
site:naukri.com "java backend" India
site:linkedin.com/jobs "rag engineer" Bengaluru India
```

### Priority 3: Full-stack and platform engineering

Adjacent roles supported by the documented project and internship experience.

```
site:naukri.com "full stack developer" React Bengaluru
site:naukri.com "devops engineer" Docker India
```

### Priority 4: Broader Technical / Consulting

Wider net for general technical roles.

```
site:naukri.com "graduate engineer trainee" Bengaluru
site:linkedin.com/jobs "junior software engineer" India
site:naukri.com "software developer" Bengaluru
```

## Location Filter

When evaluating results, verify the job location is within reasonable commute distance from your home. Define acceptable areas:
- Bengaluru and surrounding areas
- Remote roles based in India
- India-wide roles
- International relocation (open, assess visa eligibility before applying)
- Night-shift work (exclude)

## Language Filter

Your working languages and levels are in CLAUDE.md's Languages table. When filtering scraped results, apply `04-job-evaluation.md`'s Language Gate: a posting requiring a language you haven't declared at all is excluded; a posting requiring a higher level than you declared in a language you do work in is not excluded, flag it clearly instead (see `job-scraper/SKILL.md`'s Step 3 "Quick Fit Assessment" for how the flag surfaces in `/scrape` output). Postings simply *written* in a language you don't work in, that don't require it on the job, are fine.

## Date Filter

Only include jobs posted within the last 14 days, or with an application deadline that has not yet passed. If a posting date cannot be determined, include it but flag as "date unknown".

## Adapting Queries

If the user specifies a focus area, select queries from the matching category and also generate 2-3 custom queries for that focus. For example:
- "/scrape [focus_area]" -> relevant category queries + custom focus-specific queries
