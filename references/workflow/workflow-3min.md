# Quick Analysis Workflow (3min)

This workflow is optimized for focused, time-sensitive analysis with 1-2 key questions.

---

## Team Structure

**Team:**
- PL (you)
- 2 Business Experts (subagents, no team_name)
- Partner (teammate, on-demand in Phase 3)

**Process:**
- Phase 0: No team spawn (Partner spawned on-demand in Phase 3)
- Phase 2: Research → PL reviews → User checkpoint
- Phase 3: Deep dive → Partner review (spawned on-demand, reads YAMLs, gives feedback) → Deliverable
- No meetings, no Fact-Checker, no Deliverable Advisor
- PL does fact-checking inline
- PL builds deliverable (respects user's format choice)

---

## Phase 0: Environment Check ⛔ BLOCKING

**This phase is not optional.** Before restating the problem, before asking questions, before doing anything analytical — run this check first.

### First Message Template

Start with environment status:
> "Before we dive in — quick setup check: I have web access and file permissions. I don't see Octagon MCP configured, which would help with competitive data — I can work without it, but here's how to set it up if you want: [link to setup-guide.md]."

### Resume/New Check

- `--resume` → look for `process/engagement-state.yaml`. If found, brief user and skip to appropriate phase.
- `--new` → ignore existing `process/`, start fresh.
- Neither flag + `process/` exists → ask user: "Resume or start fresh?"

### Create Project Folder

```bash
PROJECT_DIR="<slug-from-topic>"  # e.g., "b2c-paint-market-entry"
mkdir -p "$PROJECT_DIR/process"
cd "$PROJECT_DIR"
```

Tell user the path: "I've created `~/Desktop/b2c-paint-market-entry/` — all files go there."

If the user already specified a working directory or the current directory looks like an existing project (has `process/` in it), use it as-is — don't nest folders.

### Required Checks

1. **File read/write permissions** — stop if missing
2. **Web access** — test with this fallback chain:
   - Priority 1: WebFetch on `https://en.wikipedia.org/wiki/Main_Page`
   - Priority 2: `curl -s -o /dev/null -w "%{http_code}" https://en.wikipedia.org/wiki/Main_Page --max-time 10`
   If curl returns 200 but WebFetch failed, tell the user: "WebFetch isn't working in this session, but network access is fine — I'll use curl instead." For all agent prompts, include: "Use `curl -s <url> | head -c 50000` via Bash for web fetches instead of WebFetch."
   - Priority 3: Search MCPs (Brave Search, etc.)
   - Priority 4: Model knowledge only (flag all data as `model_estimate`)

3. **Output dependencies** — Markdown always works. HTML/PPTX/DOCX bundled. Notion needs MCP.

### Check and Recommend

- **Output-specific dependencies:** Markdown (.md) is always available with zero setup. HTML slides, PPTX, and DOCX are bundled — no external skills needed. For Notion output → check Notion MCP is configured (see `references/workflow/setup-guide.md`).
- **Data MCPs:** List what's available (Octagon, FRED, AkShare, Yahoo Finance, Brave Search) with brief value explanation. Don't overwhelm — just note what would help.

### Quality Settings Recommendation

Suggest to user if not configured:
```json
{
  "model": "claude-opus-4-6",
  "env": {
    "CLAUDE_CODE_EFFORT_LEVEL": "max",
    "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "64000"
  }
}
```

- `effort_level: max` unlocks the deepest reasoning mode (Opus 4.6 only). This is the single biggest lever for analysis quality.
- `max_output_tokens: 64000` (default is 32K) gives more room for detailed deliverables.
- Note: these settings increase cost significantly. Only recommend for strategy engagements, not quick questions.

When dispatching subagents, use `model: "opus"` for Business Experts that need deep analysis. Use `model: "haiku"` for simple tasks like file formatting or data extraction.

### Post-Scope MCP Recommendation

After Phase 1 (post-scope), make a SPECIFIC MCP recommendation based on the problem domain. Point to `references/workflow/setup-guide.md`.

### No Team Spawn in Phase 0

For 3min engagements, do NOT spawn any teammates in Phase 0. Partner will be spawned on-demand in Phase 3 only.

---

## Phase 1: Scope & Clarification ⚠️ REQUIRED

See `phases-overview.md` for complete Phase 1 instructions (same across all tiers).

---

## Phase 2: Landscape Research & Preliminary Findings

This is the initial phase of the project. You establish baseline understanding through broad exploration, develop preliminary insights, form hypotheses, and conduct preliminary validation.

**Mindset: Insightful overview with preliminary validation**
- Understand the landscape, identify patterns, see what's there
- Include how/why at an overview level (not just "what")
- Generate insights from patterns (what do trends mean? preliminary implications?)
- Conduct preliminary validation of key hypotheses
- NOT exhaustive deep problem solving yet - that's Phase 3

### Build Initial Issue Tree (Before Deploying Experts)

**Create the issue tree BEFORE Phase 2 research begins.** The issue tree guides which experts to deploy and what questions they should answer.

**The issue tree is the single source of truth** for key questions, hypotheses, and progress state. **PL must keep it updated** throughout the engagement.

**Internal structure** (saved to `process/issue-tree.yaml` or `.md` if yaml fails):
```yaml
core_question: "Should we launch B2C paint in EU/US?"
version: 1
last_updated: "<ISO 8601>"
branches:
  - id: "market-viability"
    key_question: "Is the market attractive?"
    workstream: "market"
    hypotheses:
      - id: H1
        statement: "Decorative segment growing >3% CAGR"
        status: pending
        evidence_summary: ""
      - id: H2
        statement: "Specialty niches have room for new entrants"
        status: pending
        evidence_summary: ""
  - id: "cost-advantage"
    key_question: "Does our cost advantage survive?"
    workstream: "cost"
    hypotheses:
      - id: H3
        statement: "Margins positive after tariffs + shipping"
        status: pending
        evidence_summary: ""
changelog:
  - version: 1
    timestamp: "<ISO 8601>"
    change: "Initial tree created before Phase 2 research"
```

**User-facing presentation** (ASCII tree with verbal context - questions only, not answers):
```
Core question: Should we launch B2C paint in EU/US?

├── Is the market attractive?
│   We're testing whether the decorative paint segment is growing fast
│   enough to support new entrants, and whether specialty eco-friendly
│   niches have room for a new player like you.
│
└── Does our cost advantage survive?
    We're checking if your 38% manufacturing cost advantage can survive
    after we factor in tariffs, shipping, and EU compliance costs.
```

**Key principle:** Show the KEY QUESTIONS and directional context to the user, but keep granular hypotheses (H1, H2, H3) internal.

### Spawn Business Experts (Subagents)

For 3min engagements, spawn 2 Business Experts as **subagents** (no team_name parameter):

```python
# Expert A - Problem scope 1
Agent(
    prompt="<problem scope prompt for branch 1>",
    subagent_type="general-purpose",
    model="opus"
    # NO team_name - this is a subagent
)

# Expert B - Problem scope 2
Agent(
    prompt="<problem scope prompt for branch 2>",
    subagent_type="general-purpose",
    model="opus"
    # NO team_name - this is a subagent
)
```

Each expert owns one branch of the issue tree.

### Every Agent Must Write YAML

Include in every agent prompt:
> "Before you return, write your findings to `process/preliminary-<workstream-name>.yaml` using the format in `references/templates/yaml-formats.md`."

After agents return: `ls process/preliminary-*.yaml` — re-dispatch any agent that didn't write its file.

### PL Does Inline Fact-Checking

After all Business Experts complete their preliminary research, the PL does inline fact-checking:

1. Read all experts' `process/preliminary-*.yaml` files
2. **Sanity check key data points**:
   - Has `source_type` (verified/model_estimate/derived)?
   - If `verified`, has `source_url` present?
   - Cross-check for obvious contradictions across experts' findings
3. Flag high `model_estimate` ratios (>10%)
4. If issues found, re-dispatch affected experts with corrections

**No separate fact-check YAML file** - PL handles this inline.

### ⚠️ PRELIMINARY FINDINGS CHECKPOINT (Mandatory)

**Present preliminary findings to the user — as its own message.** This is a mandatory checkpoint between Phase 2 and Phase 3.

**What to share:**
1. **Context:** "Here's what we've learned in our initial research (Phase 2 of 6). These findings are preliminary — we'll go deeper in Phase 3 based on your feedback."

2. **Preliminary findings:** Share 3-5 key insights from the research:
   - Market context (size, growth, trends)
   - Competitive landscape
   - Initial insights and patterns
   - What this means for the client

3. **Issue tree:** Show the key questions you'll investigate (ASCII tree with verbal context, not granular hypotheses)

4. **Ask for feedback:** "Does this framing make sense? Should we adjust focus before diving deeper?"

Wait for user feedback before proceeding to Phase 3.

---

## Phase 3: Deep Problem Solving

Based on user feedback from Phase 2, you now go deeper on specific areas.

**Mindset: Deep insightful problem solving with thorough validation**
- Go as deep as needed to actually solve the problem
- Validate hypotheses thoroughly, understand mechanisms deeply, analyze trade-offs
- Develop actionable recommendations with confidence

### Deploy Targeted Experts

Based on user feedback, deploy the same 2 experts (or new ones) for deep validation. Each writes to `process/deep-<workstream-name>.yaml`.

### PL Does Inline Fact-Checking

After all Business Experts complete deep research, the PL does inline fact-checking (same process as Phase 2, but more thorough):

1. Read all experts' `process/deep-*.yaml` files
2. **Sanity check ALL key data points**:
   - Has `source_type` (verified/model_estimate/derived)?
   - If `verified`, has `source_url` present?
   - Cross-check for contradictions across experts
3. Flag high `model_estimate` ratios (>10%)
4. **Cross-workstream contradiction check**: If Expert A's finding contradicts Expert B's assumption, resolve before proceeding

### Pivot Check (MANDATORY after validation)

After experts return and contradictions are resolved, ask:

1. **"Does what we just learned change the issue tree?"**
   - If YES → update `process/issue-tree.yaml`, spawn new experts, run another round
   - If NO → proceed to Partner review

2. **"Do any validated findings invalidate other experts' assumptions?"**
   - If YES → re-dispatch affected experts with corrected inputs

3. **"Is the core question still the right question?"**
   - If NO → loop back to Phase 1 with user

### Partner Review (On-Demand)

Spawn Partner as a teammate for final review:

```python
Agent(
    prompt="<Partner review prompt>",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",   # ← Partner is a teammate
    name="partner",
    model="opus"
)
```

Partner prompt should include:
- "Read all expert YAML files in process/ (preliminary-*.yaml and deep-*.yaml)"
- "Read `references/methodology/partner-guide.md` for review criteria"
- "Evaluate storyline coherence, insightfulness, creativity, problem-solving effectiveness"
- "Give feedback on each expert's work"
- "Write review to process/partner-review-phase3.yaml"

**No meeting** - Partner reads YAMLs and provides written feedback.

If Partner flags issues, address them before proceeding to Phase 4.

---

## Phase 4: Final Checkpoint ⚠️ REQUIRED

Present deliverable structure and direction to user.

**What to present:**
- Section outline (what sections will be covered)
- Narrative arc (how sections connect)
- Key direction (what will be recommended)
- Evidence strength assessment (where evidence is strong vs weak)

**Example:**
```
Final Checkpoint (Phase 4 of 6)

Deliverable structure:
1. Executive Summary - Key recommendation and rationale
2. Market Viability - EU decorative paint market, eco segment opportunity
3. Cost Advantage Analysis - Tariff/shipping impact, competitive positioning
4. Recommendations - Phased approach with timeline
5. Risks & Mitigations

Direction: The deliverable will recommend Amazon DTC launch with phased approach,
based on strong market evidence and validated cost advantage.

Evidence strength: Strong on market size and cost structure, moderate on channel
economics (limited comparable data)

Ready to proceed with building this?
```

### ★ USER SIGN-OFF (mandatory)

User must approve before deliverable generation.

---

## Phase 5: Deliverable Creation

The PL builds the final output as a consulting-style narrative.

**Before building the deliverable, the PL MUST read `references/methodology/bcg-patterns.md`.** This contains structural patterns, headline styles, data presentation conventions, and narrative frameworks.

### Deliverable Requirements

**PL builds deliverable in user's chosen format:**
- If user chose slides → build slides (html or pptx)
- If user chose markdown → build .md document
- If user chose docx → build .docx document
- If user chose Notion → build Notion page

**For slides, ALWAYS produce BOTH formats:**
1. Slides (pptx or html) — for presenting
2. Markdown document (.md) — for depth and reading (ALWAYS produced alongside slides)

**File naming:** Use contextual, professional names based on engagement topic (e.g., `Market_Entry_Strategy_EU_Paint_v1.md` and `Market_Entry_Strategy_EU_Paint_v1.pptx`)

### Output Structure (Pyramid Principle)

1. **Executive Summary** — 5-15 key conclusions and insights as bullet points. Lead with the answer.
2. **Situation / Context** — Where things stand today.
3. **Storylines** — 2-3 connected storylines. Each has:
   - A headline — at the problem/domain level, state a clear "so what"
   - Evidence: data, charts, benchmarks, business cases
   - Connection to the next storyline
4. **Recommendations** — Specific, actionable next steps with prioritization
5. **Risks & Mitigations** *(optional)* — Include when the decision carries meaningful downside
6. **Appendix** — Detailed data, methodology, additional analysis

### Pre-Delivery Checklist ⚠️ REQUIRED

See `references/workflow/pre-delivery-checklist.md` — verify every item before presenting.

---

## Phase 6: Next Steps & Resumability

### Next Steps Categories

**Category A: Things that CAN be researched now.** If a next step involves publicly available data, provide exact URLs, databases, and search queries.

**Category B: Things that require human action.** For each, **generate a ready-to-use document** saved to `next-steps/`:
- Expert/stakeholder interview guides → `next-steps/interview-guide-<topic>.md`
- Customer survey drafts → `next-steps/survey-<topic>.md`
- Internal data request specs → `next-steps/data-request-<topic>.md`
- Meeting agendas → `next-steps/meeting-agenda-<topic>.md`

### Save Engagement State

Write `process/engagement-state.yaml` for resumability.

### Framing

Frame the deliverable as complete on its own, but with a clear pull:

> "Here's the best analysis I can build with what's available. To take this further, these are the areas where better data would sharpen the conclusions — [specific items]. If you come back with any of these, I can show you how the picture changes."
