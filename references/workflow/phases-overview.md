# Workflow Phases Overview

The engagement follows six phases (plus Phase 0 setup). The workflow adapts based on engagement complexity.

---

## Tiered System by --length

The team structure and process adapt based on engagement complexity. Partner is present for ALL lengths to provide strategic feedback, but involvement varies.

### Quick Reference

| Length | Team Size | Meetings | Fact-Checking | Deliverable Builder | Workflow File |
|--------|-----------|----------|---------------|---------------------|---------------|
| `3min` | PL + 2 experts (subagents) + Partner (on-demand) | None | PL inline | PL | `workflow-3min.md` |
| `5min` | PL + 3 experts + Partner + Deliverable Advisor (on-demand) | 1 (Phase 3) | PL inline | Deliverable Advisor | `workflow-5min.md` |
| `10min` / `10min+` | PL + 4-6 experts + Partner + Fact-Checker + Deliverable Advisor | 2 (Phase 2 + 3) | Fact-Checker | Deliverable Advisor | `workflow-10min-plus.md` |

### Which Workflow to Use

**If `--length 3min`:** Read `workflow-3min.md` for the quick analysis workflow.

**If `--length 5min` (DEFAULT):** Read `workflow-5min.md` for the standard workflow.

**If `--length 10min` or `10min+`:** Read `workflow-10min-plus.md` for the comprehensive workflow.

---

## Common Phases (All Tiers)

Some phases work the same across all tiers:

### Phase 1: Scope & Clarification ⚠️ REQUIRED

The most important phase. Misaligned scope wastes everything downstream.

**What to Do:**
1. Restate user's problem in your own words
2. Ask clarifying questions — don't assume
3. Align on scope: what's in, what's out, what decision is being made
4. Skip questions answered by flags (`--format`, `--length`, `--level`)

**Mandatory Questions (unless flags set):**

**You MUST ask the user about output format before presenting the scope checkpoint.** This is not optional — do not assume markdown. Do not decide the format on the user's behalf. Ask it as its own standalone question, separate from other clarifications. Use this exact phrasing:

  > "What format would you like the final deliverable in?"
  > 1. Markdown report (.md) — simple, editable, no setup needed
  > 2. HTML slides — animated, runs in browser, zero dependencies
  > 3. PowerPoint (.pptx) — traditional slide deck
  > 4. Word document (.docx) — formal report format
  > 5. Notion page — collaborative, team-editable

  Only skip this question if `--format` was explicitly set in the arguments. "The user didn't mention a format" is NOT the same as "the user chose markdown." You must ask.

**You MUST ask the user about report length before presenting the scope checkpoint.** This is separate from the format question. Use this phrasing:

  > "How much content should the final deliverable cover?"
  > 1. Focused analysis (~5-8 slides / ~1000 words) — 1-2 key questions, for quick decisions
  > 2. Standard analysis (~10-15 slides / ~2500 words) — 3-4 key questions, typical strategy work
  > 3. Comprehensive analysis (~20-25 slides / ~4500 words) — 4-5 key questions, detailed evidence and benchmarking
  > 4. Extensive analysis (~30+ slides / ~6000+ words) — 6-7 key questions, full strategic review

  Only skip this question if `--length` was explicitly set in the arguments. Do not default to "standard" without asking.

**Example Clarification Questions:**
- "The core question is [X]. Is that accurate?"
- "Are we looking at this from a [company/investor/regulator] perspective?"
- "What geographies are in scope?"
- "Do you have internal data I should factor in?"
- "Who's the audience, and what decision will this inform?"
- Ask if they have internal data, proprietary research, or specific datasets
- Ask about the user's background and what they care about — this shapes the analysis

**★ SCOPE CHECKPOINT (mandatory):**

User confirms problem statement, scope, and output format. Do not proceed without this.

---

## Data Sources

Use available MCPs and APIs to pull real data. The skill adapts to whatever tools the user has configured.

**Sourcing priority:**
1. User's internal data — always ask first
2. Configured MCPs (fastest, most structured)
3. Free public APIs via Python scripts
4. Web search for reports and qualitative data
5. If data doesn't exist — note the gap transparently, use logical reasoning or analogies

See `references/workflow/data-sources.md` for the full catalog of MCPs, APIs, and Python libraries.
See `references/workflow/setup-guide.md` for installation and configuration instructions.
