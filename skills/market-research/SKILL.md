---
name: market-research
description: Research a company, person, or sector to produce client intelligence for a sales meeting, engagement, or proposal. Covers company profiling, leadership analysis, competitive landscape, sector context, and use case identification. Outputs structured markdown documents to a GitHub repo.
metadata: {"clawdbot":{"emoji":"🔍"}}
---

# market-research — Client Intelligence Research

Produces a structured client intelligence suite from public sources. Used to prepare for sales meetings, client engagements, proposals, or partnership conversations.

## When to use

- Preparing for a first meeting with a prospective client
- Building a profile of a partner, competitor, or collaborator
- Researching a sector before pitching into it
- Producing background intelligence for an engagement

## Output structure

Research is written to a GitHub repo with this structure:

```
client-<name>/
├── README.md                    ← Engagement overview, quick links, status
└── research/
    ├── 00-index.md              ← Research index, one-para brief, quick reference table
    ├── 01-company-overview.md   ← Who they are, origin, structure, tech stack, culture
    ├── 02-leadership.md         ← Key people, signals, meeting room dynamics
    ├── 03-services-clients.md   ← Services depth + full client/product portfolio
    ├── 04-ai-stance.md          ← Their published views on AI + what it signals (if relevant)
    ├── 05-<partner>.md          ← Any key partners/third parties in the engagement
    ├── 06-sector-context.md     ← Industry landscape, competitive dynamics, disruption forces
    ├── 07-forge-labs-fit.md     ← How Forge Labs maps to their pain points
    ├── 08-use-cases.md          ← Prioritised use cases (tiered: quick wins / medium / strategic)
    ├── 09-meeting-prep.md       ← Agenda, opening line, discovery questions, objection handling
    ├── 10-team-profiles.md      ← Forge Labs team profiles relevant to this engagement
    └── 11-<person>-profile.md   ← Individual profiles for key Forge Labs people (if needed)
```

Files are numbered for consistent ordering. README.md links to all research files.

## Research process

### Phase 1 — Planning

Before fetching anything, build a research plan:
1. What is the engagement? (first meeting, proposal, ongoing client?)
2. Who are the key people? (prospect contacts, partners, our own team)
3. What sources are available? (website, LinkedIn, GitHub, case studies, news, sector reports)
4. What are the key questions to answer?

### Phase 2 — Data gathering (parallel where possible)

Use browser and curl to gather from all available public sources simultaneously:

**Company:**
- Website (all pages: about, team, work/portfolio, services, insights/blog, case studies)
- LinkedIn company page (often auth-walled — try anyway, note if blocked)
- GitHub (if tech company)
- News and press mentions
- Job listings (reveals tech stack, growth areas, pain points)

**Leadership:**
- Team page on website — capture exact quotes, role titles, "currently learning" notes, anything personal
- LinkedIn profiles (often auth-walled — try, note if blocked)
- GitHub profiles
- Conference talks, podcast appearances, published writing
- Canterbury Tech / local community board memberships

**Sector:**
- How is AI/technology changing this sector?
- Who are the key competitors?
- What are the global players doing that local players haven't caught up with yet?
- What are the biggest pain points for businesses in this sector?

**Our team:**
- Match Forge Labs capabilities to their specific pain points
- Identify which team members are most relevant and why
- Note any shared community connections (Canterbury Tech, meetups, etc.)

### Phase 3 — Synthesis and writing

Write each document in order (00 → 11). Key principles:

**Fact vs inference:**
- Verified facts: stated directly on public sources — present as fact with citation
- Inferences: interpretations, motivations, predictions — label with `> 💡 **Inference:**`
- Unverified: claims you can't source — label with `> ⚠️ **Unverified:**`

**Document standards:**
- Every file: confidence metadata block at top, TOC with backlinks, Sources section at bottom
- ASCII diagrams for org charts, competitive landscapes, workflow maps
- Folder trees for structure diagrams
- Tables for comparisons, team rosters, client portfolios

**Confidence metadata block (top of every file):**
```markdown
> **Confidence:** 🟢 High / 🟡 Medium / 🔴 Low
> **Last verified:** YYYY-MM-DD
> **Primary sources:** [Source](URL), [Source](URL)
```

**Sources section (bottom of every file):**
```markdown
---
## Sources
[^1]: [Source name](URL)
[^2]: [Source name](URL)
```

**TOC backlink pattern (after every `##` heading):**
```markdown
## Section Name
[↑ Top](#table-of-contents)

content...
```

### Phase 4 — Use case analysis

For every engagement, produce a tiered use case list:

```
Tier 1 — Quick Wins (0–30 days)
  High value, low friction, fast ROI
  5 use cases max

Tier 2 — Medium-term (1–3 months)
  Higher complexity, proven value building on tier 1
  5 use cases max

Tier 3 — Strategic (3–12 months)
  Structural, partnership-level, or new revenue streams
  5 use cases max
```

For each use case: the problem, the solution (with workflow diagram if helpful), impact estimate, relevant Forge Labs services.

Include a prioritisation matrix (effort vs impact grid).

### Phase 5 — Meeting prep

`09-meeting-prep.md` is the most important file for an upcoming meeting. It must include:

- Meeting details (date, time, location, attendees)
- Our objectives (ranked)
- Their likely objectives
- Suggested agenda with timings
- Opening line (specific, researched, disarming)
- Discovery questions (10–15, categorised)
- Key talking points (3–5 specific, quotable)
- Objection handling table (objection | response)
- What success looks like (minimum / good / great)

## Git workflow

```bash
# Create repo
flgh repo create client-<name> --desc "Forge Labs engagement with <Name>"
flgh clone forgelabs-sierra/client-<name>
cd /workspace/client-<name>
git config user.email "sierra@forgelabs.nz"
git config user.name "Sierra"

# Fix remote after clone (removes embedded token)
flgh remote set-url forgelabs-sierra/client-<name>

# Commit and push
git add .
git commit -m "research: <description>"
flgh push
```

**Important:** Always run `flgh remote set-url` after cloning — the clone command embeds a token in the remote URL that the Squid proxy will block on push.

## Subagent usage — lessons learned

### Validation pass via subagent

After the main research is written, a validation pass (adding citations, TOC backlinks, confidence ratings, inference labels) can be delegated to a subagent. However:

**Timeout risk:** A single subagent tasked with processing 10+ files will likely time out (10 min limit) if it tries to analyse deeply before editing. The first attempt timed out having only completed the analysis phase.

**What works:**
- Give the subagent explicit, mechanical instructions — "do these 4 specific things to each file, in order, save and move on immediately"
- Explicitly say: do NOT re-read or over-analyse — fast and complete beats slow and perfect
- Provide all verified facts in the task prompt so the subagent doesn't need to fetch URLs
- Keep the task tightly scoped — one pass, one commit, done

**What doesn't work:**
- Asking the subagent to "validate and improve" with latitude — it will spend its budget on analysis
- Asking it to fetch and cross-check URLs during the pass — adds time, rarely changes outcome for a first pass
- A single subagent for more than ~10 files at depth

**Recovery:** If a subagent times out mid-task, spawn a second one with tighter instructions targeting only the remaining work. Check git log first to see what was already committed.

## Quality standards

Before considering research complete:

- [ ] All facts cited to a source URL
- [ ] All inferences labelled
- [ ] Leadership profiles include motivations for attending the meeting
- [ ] Use cases mapped to specific Forge Labs services
- [ ] Meeting prep includes a specific opening line (not generic)
- [ ] Confidence ratings reflect actual source quality
- [ ] README.md has quick links to all key files
- [ ] All files committed and pushed

## Tone and style

- Analytical, not promotional
- Honest about what is verified vs inferred
- Opinionated where evidence supports it — don't hedge everything
- Use ASCII diagrams liberally — they render well in GitHub markdown
- Tables for structured comparisons
- Keep individual sections scannable — bullets over paragraphs where appropriate
