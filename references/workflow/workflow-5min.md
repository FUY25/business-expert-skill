# Standard Analysis Workflow (5min)

This is the default workflow for typical strategy work with 3-4 key questions.

---

## Team Structure

**Team:**
- PL (you)
- 3 Business Experts (teammates with mode="plan")
- Partner (teammate, on-demand in Phase 3)
- Deliverable Advisor (teammate, on-demand in Phase 3)

**Process:**
- Phase 0: No team spawn (Partner + Deliverable Advisor spawned on-demand in Phase 3)
- Phase 2: Research → PL sanity check → User checkpoint
- Phase 3: Deep dive → MEETING (Partner + Deliverable Advisor spawned, Partner reads YAMLs and gives strategic feedback, Deliverable Advisor gives presentation feedback) → Deliverable
- 1 meeting only (Phase 3 final, includes Deliverable Advisor)
- PL does fact-checking inline
- Deliverable Advisor builds deliverable (respects user's format choice)

---

## Phase 0: Environment Check ⛔ BLOCKING

**This phase is not optional.** Before restating the problem, before asking questions, before doing anything analytical — run this check first.

### First Message Template

Start with environment status:
> "Before we dive in — quick setup check: I have web access and file permissions. I'll use Agent Teams for parallel research. I don't see Octagon MCP configured, which would help with competitive data — I can work without it, but here's how to set it up if you want: [link to setup-guide.md]."

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

3. **Agent Teams** — Check if `TeamCreate` is in tools. See `references/methodology/agent-teams-guide.md` for setup.

4. **Output dependencies** — Markdown always works. HTML/PPTX/DOCX bundled. Notion needs MCP.

### Check and Recommend

- **Agent Teams:** Check if `TeamCreate` is in your available tools. If yes, use it. If not, tell the user it's not enabled and proceed with hub-and-spoke. See `references/methodology/agent-teams-guide.md` for setup.
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

For 5min engagements, do NOT spawn any teammates in Phase 0. Partner and Deliverable Advisor will be spawned on-demand in Phase 3 only.

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
  - id: "cost-advantage"
    key_question: "Does our cost advantage survive?"
    workstream: "cost"
    hypotheses:
      - id: H3
        statement: "Margins positive after tariffs + shipping"
        status: pending
        evidence_summary: ""
  - id: "channel-strategy"
    key_question: "Which channel is most feasible?"
    workstream: "channel"
    hypotheses:
      - id: H5
        statement: "Amazon DTC viable with <$500K budget"
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
├── Does our cost advantage survive?
│   We're checking if your 38% manufacturing cost advantage can survive
│   after we factor in tariffs, shipping, and EU compliance costs.
│
└── Which channel is most feasible?
    We're evaluating two paths: Amazon DTC (lower upfront cost) vs.
    retail partnerships (higher reach but more complex).
```

**Key principle:** Show the KEY QUESTIONS and directional context to the user, but keep granular hypotheses (H1, H2, H3) internal.

### Create Team

Before spawning Business Experts, create the team:

```python
TeamCreate(
    team_name="<project-slug>-strategy",
    description="Strategy engagement for <topic>"
)
```

### Spawn Business Experts (Teammates with Plan Approval)

For 5min engagements, spawn 3 Business Experts as **teammates** with `mode="plan"`:

```python
# Expert A - Problem scope 1
Agent(
    prompt="<problem scope prompt for branch 1>",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",   # ← REQUIRED for teammates
    name="expert-a",                         # ← REQUIRED for teammates
    model="opus",
    mode="plan"                              # ← REQUIRES PLAN APPROVAL
)

# Expert B - Problem scope 2
Agent(
    prompt="<problem scope prompt for branch 2>",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",
    name="expert-b",
    model="opus",
    mode="plan"
)

# Expert C - Problem scope 3
Agent(
    prompt="<problem scope prompt for branch 3>",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",
    name="expert-c",
    model="opus",
    mode="plan"
)
```

Each expert owns one branch of the issue tree.

### Plan Approval

Experts with `mode="plan"` submit plans via ExitPlanMode. PL receives `plan_approval_request` messages.

**Check each expert plan for:**
1. **Hypothesis-driven:** Clear hypothesis to test, not just "research X"
2. **Scope clarity:** Bounded, no overlap with other experts
3. **Evidence strategy AND deliverable specifications:**
   - Specific sources and types identified
   - Concrete deliverables specified (not "analyze X" but "identify 5-10 X with Y, Z attributes")
   - Benchmark framing decided (median vs top quartile, adjacent industries, qualitative vs quantitative)
   - Output format clear (comparison table, positioning map, ranked list, driver tree)
4. **Output commitment:** Commits to writing YAML to `process/`
5. **Analytical approach:** Will produce conclusions, not data dumps

**To APPROVE:**
```python
SendMessage(
    type="plan_approval_response",
    request_id="<from the request>",
    recipient="expert-a",
    approve=True
)
```

**To REJECT with feedback:**
```python
SendMessage(
    type="plan_approval_response",
    request_id="<from the request>",
    recipient="expert-b",
    approve=False,
    content="Your scope overlaps with Expert A. Focus only on cost structure."
)
```

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

Based on user feedback, deploy the same 3 experts for deep validation. Each writes to `process/deep-<workstream-name>.yaml`.

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
   - If NO → proceed to meeting

2. **"Do any validated findings invalidate other experts' assumptions?"**
   - If YES → re-dispatch affected experts with corrected inputs

3. **"Is the core question still the right question?"**
   - If NO → loop back to Phase 1 with user

### Phase 3 Final Meeting (MANDATORY)

Spawn Partner and Deliverable Advisor as teammates for the final meeting:

```python
# Partner
Agent(
    prompt="<Partner prompt>",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",
    name="partner",
    model="opus"
)

# Deliverable Advisor
Agent(
    prompt="<Deliverable Advisor prompt>",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",
    name="deliverable-advisor",
    model="opus"
)
```

**Meeting structure:**
1. **Partner reads expert YAMLs during the meeting** and gives comments/feedback to each agent:
   - Reads each expert's deep-*.yaml file live
   - Reviews each expert's validation work in real-time
   - Checks for insightfulness, creativity, problem-solving effectiveness
   - Identifies any remaining gaps or issues
   - Gives specific feedback to each agent

2. **Each agent responds** and commits to addressing Partner's comments

3. **Deliverable Advisor participates** to ensure findings support final deliverable:
   - Reviews findings from presentation perspective
   - Confirms data formats work for visualization
   - Identifies any gaps that would weaken deliverable
   - Starts planning deliverable structure

4. **PL shares thoughts** after reviewing detailed docs:
   - Synthesizes findings across workstreams
   - Assesses storyline coherence
   - Proposes recommendations

**Discussion topics:**
- Review final findings across all workstreams
- Ensure recommendations are defensible
- Resolve any remaining contradictions
- Assess storyline coherence
- Confirm deliverable structure (Deliverable Advisor input)

**No separate Partner review file** - the meeting discussion captures all feedback.

If Partner flags issues during the meeting, address them before proceeding to Phase 4.

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
4. Channel Strategy - Amazon DTC feasibility, launch economics
5. Recommendations - Phased approach with timeline
6. Risks & Mitigations

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

The Deliverable Advisor builds the final output as a consulting-style narrative.

**Before building the deliverable, the Deliverable Advisor MUST read `references/methodology/bcg-patterns.md`.** This contains structural patterns, headline styles, data presentation conventions, and narrative frameworks.

### Structure Proposal First

Before building the full deliverable, the Deliverable Advisor proposes the structure to the PL for quick approval:

```
SendMessage(
  type="message",
  recipient="<PL name or main session>",
  content="Here's my proposed structure for the deliverable:

  1. Executive Summary (1 slide) — [key recommendation]
  2. Basis of Perspectives (1 slide) — sources and methodology
  3. Market Viability (3 slides) — [H1, H2 findings]
  4. Cost Advantage Analysis (3 slides) — [H3, H4 findings]
  5. Channel Strategy (4 slides) — [H5, H6 findings + benchmarking]
  6. Recommendations (2 slides) — phased approach with timeline
  7. Risks & Mitigations (1 slide)
  8. Appendix (3 slides) — detailed data tables

  Total: ~18 slides. Does this structure work, or should I adjust?

  Note: I'll also produce a companion .md document with the full analysis for reading and depth.",
  summary="Deliverable structure proposal"
)
```

The PL reviews and either approves or suggests adjustments.

### Deliverable Requirements

**Deliverable Advisor builds deliverable in user's chosen format:**
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
3. **Storylines** — 3-4 connected storylines. Each has:
   - A headline — at the problem/domain level, state a clear "so what"
   - Evidence: data, charts, benchmarks, business cases
   - Connection to the next storyline
4. **Recommendations** — Specific, actionable next steps with prioritization
5. **Risks & Mitigations** *(optional)* — Include when the decision carries meaningful downside
6. **Appendix** — Detailed data, methodology, additional analysis

### Review Loop (3 versions max)

1. **Version 1:** Deliverable Advisor produces first draft → PL does full review → lists ALL issues at once
2. **Version 2:** Deliverable Advisor fixes all issues → PL re-reviews affected sections → catches anything new
3. **Version 3:** Final polish → if still issues, present to user with noted imperfections

After 3 versions, diminishing returns kick in. Present what you have.

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

---

## Shutdown

After Phase 6, gracefully shut down teammates:

```python
SendMessage(type="shutdown_request", recipient="partner", content="Work complete")
SendMessage(type="shutdown_request", recipient="expert-a", content="Work complete")
SendMessage(type="shutdown_request", recipient="expert-b", content="Work complete")
SendMessage(type="shutdown_request", recipient="expert-c", content="Work complete")
SendMessage(type="shutdown_request", recipient="deliverable-advisor", content="Work complete")
```

Then cleanup:

```python
TeamDelete()
```
