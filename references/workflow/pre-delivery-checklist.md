# Pre-Delivery Checklist

Verify EVERY item before presenting deliverable to user.

---

## Structure & Narrative

- [ ] Executive summary presents 5-15 key conclusions as bullet points
- [ ] Storyline headlines at problem/domain level state clear "so what"
- [ ] Narrative flows — each storyline connects logically to the next
- [ ] Deliverable length matches user's requested reading time (`--length`)

## Evidence & Sources

- [ ] Every claim has traceable source in references section
- [ ] Every key data point has a **retrievable URL** (not just source name)
  - ✅ "Euromonitor 2025 (https://www.euromonitor.com/paint-market-eu-2025)"
  - ❌ "Euromonitor 2025" (not retrievable)
- [ ] Every data slide has source footnotes at bottom
- [ ] For slides: "Sources & References" slide at end with full URLs for all major claims
- [ ] For documents: Inline citations with URLs + "References" section at end with full bibliography
- [ ] For paywalled sources: Include URL + note "(subscription required)" so user knows where to access
- [ ] For model estimates: Explicitly labeled as "Model estimate (not verified)" in source line
- [ ] No orphan evidence — every chart/trend connects to storyline or recommendation
- [ ] Citations grouped per slide/section, not scattered inline

## Charts & Visualizations

- [ ] Every chart has labeled axes with units
- [ ] Every chart has one-line takeaway caption
- [ ] Every chart has source attribution

## Content Density

- [ ] 60-80% of slides are "evidence slides" (specific numbers, charts, quotes, sourced observations)
- [ ] No sparse slides — each carries enough substance to justify existence
- [ ] Evidence slides have: (1) insight headline, (2) 2-3+ evidence pieces, (3) source footnote

## Benchmarking & Case Studies

- [ ] At least one multi-dimensional benchmarking table (3+ competitors × 3+ dimensions)
  - Dimensions can be quantitative (margin, market share, revenue) or qualitative (positioning approach, channel strategy, trust-building mechanism)
  - Single-metric bar charts are acceptable as supplementary visuals, but the core benchmarking must be multi-dimensional
- [ ] At least 1-2 comparable case studies with depth (strategy, outcome, lesson)
  - Not every comparable needs full depth — pick the 1-2 most relevant ones
  - Each gets a half-slide or full paragraph deep-dive: what they did, the outcome, and the transferable lesson
- [ ] Every benchmark ends with "so what for this client" line
  - Not just "Competitor X has 47% margin" but "Competitor X's 47% margin suggests pricing room exists, but their margin includes brand premium that a new entrant won't have for 2-3 years."

## Financial Analysis (if applicable)

- [ ] Driver tree included if recommendation depends on financial viability
- [ ] Unit economics shown as waterfall chart (under 8 line items) or table

## Implications Pattern

- [ ] Section-level implications: each major analysis section has at least one "so what for this client" callout
- [ ] Not every slide needs implications — use judgment based on whether the section's findings change the recommendation
- [ ] In YAML files: Every agent includes an `implications` field — what does this finding mean for the client's decision?
- [ ] In deliverable: 1-2 "so what does this mean for you" slides or callout boxes per major section are enough
- [ ] The Lead decides during synthesis which implications are strong enough to surface and where they land in the narrative

## Basis of Perspectives

- [ ] "Basis of Perspectives" slide included (optional for quick/brief engagements)
  - Place early in the deck — builds credibility before any claims are made
  - Shows: number of sources analyzed (e.g., "12 public filings, 8 industry reports"), named databases and research firms used, case studies examined, any limitations (e.g., "no primary interviews conducted, model knowledge used for X")
  - Skip only for quick/executive-brief engagements where the overhead isn't justified

## Recommendations

- [ ] Recommendations are specific and actionable
- [ ] If risk section included, it has mitigations — not just list of risks

---

## Quick Reference: Content Density Standards

### Slides (HTML, PPTX)

| Slide Type | Required Elements |
|------------|-------------------|
| Evidence slide | (1) Insight headline, (2) 2-3+ evidence pieces, (3) Source footnote |
| Chart slide | Labeled axes, units, takeaway caption, source attribution |
| Section divider | Exempt from density requirements |
| Agenda slide | Exempt from density requirements |

### Documents (MD, DOCX)

| Section (H2) | Required Elements |
|--------------|-------------------|
| Every section | (1) Framing paragraph with key insight, (2) 3+ evidence points with sources, (3) At least one comparison/benchmark |

---

## Output Length Standards

| Length Flag | Deliverable Size |
|-------------|------------------|
| `3min` | 5-8 slides / 800-1200 words |
| `5min` | 10-15 slides / 2000-2500 words |
| `10min` | 20-25 slides / 4000-5000 words |
| `10min+` | 30+ slides / 6000+ words |

**Rule:** Deliverable must meet the minimum for engagement depth (3000-4000 words for standard, 5000-7000 for deep) OR 60% of research volume, whichever is higher. If shorter, Deliverable Advisor is over-compressing.

---

## Content Ownership

The business-expert skill owns the **content structure** (what goes where, narrative arc, which evidence supports which claim). The output skill/format owns the **visual execution** (layout, fonts, styling, chart rendering). The Lead's review bridges both — ensuring the visual output faithfully represents the analytical work.

---

## Process Documentation Reminder

**Process files are not optional.** Every agent — Business Expert, Partner, and PL — must write YAML files to `process/` as specified. If a phase completes without its expected files, something went wrong. The `process/` directory is the engagement's source of truth — without it, there's no audit trail, no resumability, and no way for the Partner to review structured findings.
