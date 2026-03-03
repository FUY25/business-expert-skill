# Comprehensive Analysis Workflow (10min / 10min+)

This workflow is for detailed strategic work with 4-7 key questions, requiring rigorous validation and comprehensive analysis.

---

## Team Structure

**Team:**
- PL (you)
- 4-6 Business Experts (teammates with mode="plan")
- Partner (teammate, throughout)
- Fact-Checker (teammate)
- Deliverable Advisor (teammate)

**Process:**
- Phase 0: Spawn Partner + Fact-Checker + Deliverable Advisor
- Phase 2: Research → Meeting (all team) → User checkpoint
- Phase 3: Deep dive → Meeting (all team) → Deliverable
- 2 meetings (Phase 2 + Phase 3)
- Fact-Checker verifies data quality
- Deliverable Advisor builds deliverable

**Already optimized** with Option B changes (Partner reads YAMLs during meetings, Fact-Checker does sanity check only)

---

## Phase 0: Environment Check ⛔ BLOCKING

**This phase is not optional.** Before restating the problem, before asking questions, before doing anything analytical — run this check first.

**READ:** None (this is the first phase)

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

### Spawn Core Team in Phase 0

For 10min/10min+ engagements, spawn Partner, Fact-Checker, and Deliverable Advisor in Phase 0:

```python
# Create team first
TeamCreate(
    team_name="<project-slug>-strategy",
    description="Comprehensive strategy engagement for <topic>"
)

# Partner
Agent(
    prompt="You are the Partner on this engagement. Read `references/methodology/partner-guide.md` for your role. You'll participate in Phase 2 and Phase 3 meetings to provide strategic oversight and quality control.",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",
    name="partner",
    model="opus"
)

# Fact-Checker
Agent(
    prompt="You are the Fact-Checker. Read `references/methodology/fact-checker-guide.md` for your role. You'll verify data quality and source reliability after Phase 2 and Phase 3 research.",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",
    name="fact-checker",
    model="sonnet"
)

# Deliverable Advisor
Agent(
    prompt="You are the Deliverable Advisor. Read `references/methodology/deliverable-advisor-guide.md` for your role. You'll participate in meetings and build the final deliverable in Phase 5.",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",
    name="deliverable-advisor",
    model="opus"
)
```

**UPDATE:** After Phase 0 completes, create `process/engagement-state.yaml` with initial state:
```yaml
current_phase: "Phase 1"
last_updated: "<ISO 8601>"
checkpoints_completed: []
team_spawned: ["partner", "fact-checker", "deliverable-advisor"]
```

---

## Phase 1: Scope & Clarification ⚠️ REQUIRED

**READ:** None (Phase 1 is scoping)

See `phases-overview.md` for complete Phase 1 instructions (same across all tiers).

**UPDATE:** After Phase 1 completes, update `process/engagement-state.yaml`:
```yaml
current_phase: "Phase 2"
last_updated: "<ISO 8601>"
scope_confirmed: true
```

---

## Phase 2: Landscape Research & Preliminary Findings

**READ:**
- `process/engagement-state.yaml` (check current phase and team status)
- `references/methodology/partner-guide.md` (for Partner review criteria)
- `references/templates/yaml-formats.md` (for expert output format)
- `references/methodology/fact-checker-guide.md` (for Fact-Checker role)

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
  - id: "channel-strategy"
    key_question: "Which channel is most feasible?"
    workstream: "channel"
    hypotheses:
      - id: H5
        statement: "Amazon DTC viable with <$500K budget"
        status: pending
        evidence_summary: ""
  - id: "competitive-positioning"
    key_question: "How do we differentiate?"
    workstream: "positioning"
    hypotheses:
      - id: H7
        statement: "Eco-friendly positioning resonates with target segment"
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
├── Which channel is most feasible?
│   We're evaluating two paths: Amazon DTC (lower upfront cost) vs.
│   retail partnerships (higher reach but more complex).
│
└── How do we differentiate?
    We're assessing whether eco-friendly positioning creates meaningful
    competitive advantage in the target segment.
```

**Key principle:** Show the KEY QUESTIONS and directional context to the user, but keep granular hypotheses (H1, H2, H3) internal.

### Partner Reviews Issue Tree

After building the issue tree, have Partner review it:

```python
SendMessage(
    type="message",
    recipient="partner",
    content="I've built the initial issue tree at process/issue-tree.yaml. Please review it and let me know if you see any issues or suggest improvements.",
    summary="Issue tree review request"
)
```

Wait for Partner's feedback. If Partner suggests changes, update the issue tree before deploying experts.

### Spawn Business Experts (Teammates with Plan Approval)

For 10min/10min+ engagements, spawn 4-6 Business Experts as **teammates** with `mode="plan"`:

```python
# Expert A - Problem scope 1
Agent(
    prompt="<problem scope prompt for branch 1>",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",
    name="expert-a",
    model="opus",
    mode="plan"
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

# Expert D - Problem scope 4
Agent(
    prompt="<problem scope prompt for branch 4>",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",
    name="expert-d",
    model="opus",
    mode="plan"
)

# For 10min+, add Expert E and F as needed
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

### Fact-Checker Verification

After all Business Experts complete their preliminary research, dispatch the Fact-Checker:

```python
SendMessage(
    type="message",
    recipient="fact-checker",
    content="All experts have completed Phase 2 research. Please verify data quality across all preliminary-*.yaml files. Check for:

    1. Source reliability (verified vs model_estimate ratios)
    2. Cross-workstream contradictions
    3. Missing source URLs for verified claims
    4. Suspicious data points that need validation

    Write your findings to process/fact-check-phase2.yaml with any issues flagged.",
    summary="Phase 2 fact-check request"
)
```

Wait for Fact-Checker to complete. If issues found, re-dispatch affected experts with corrections.

**Communication Discipline:** Do NOT show findings to the user until the formal checkpoint below. Work internally with the team first.

### Phase 2 Meeting (MANDATORY)

After fact-checking is complete, conduct Phase 2 meeting with all team members:

**Meeting structure:**
1. **PL presents synthesis** of preliminary findings across all workstreams
2. **Each expert presents** their key findings and preliminary insights
3. **Partner provides strategic feedback:**
   - Reads expert YAMLs during the meeting
   - Evaluates storyline coherence and insightfulness
   - Identifies gaps or areas needing deeper investigation
   - Challenges assumptions and hypotheses
4. **Fact-Checker highlights** any data quality concerns
5. **Deliverable Advisor notes** what findings will translate well to final deliverable
6. **Team discusses** what to prioritize in Phase 3 deep dive

**Discussion topics:**
- Are preliminary findings directionally correct?
- Which hypotheses need deeper validation?
- Are there new questions we should investigate?
- Should we adjust the issue tree?

### ⚠️ PRELIMINARY FINDINGS CHECKPOINT (Mandatory)

**Present preliminary findings to the user — as its own message.** This is a mandatory checkpoint between Phase 2 and Phase 3.

**What to share:**
1. **Context:** "Here's what we've learned in our initial research (Phase 2 of 6). These findings are preliminary — we'll go deeper in Phase 3 based on your feedback."

2. **Preliminary findings:** Share 4-6 key insights from the research:
   - Market context (size, growth, trends)
   - Competitive landscape
   - Initial insights and patterns
   - What this means for the client
   - Data quality assessment

3. **Issue tree:** Show the key questions you'll investigate (ASCII tree with verbal context, not granular hypotheses)

4. **Ask for feedback:** "Does this framing make sense? Should we adjust focus before diving deeper?"

Wait for user feedback before proceeding to Phase 3.

**UPDATE:** After checkpoint, update `process/engagement-state.yaml`:
```yaml
current_phase: "Phase 3"
last_updated: "<ISO 8601>"
checkpoints_completed: ["preliminary_findings"]
user_feedback: "<summary of user feedback>"
phase2_meeting_complete: true
```

---

## Phase 3: Deep Problem Solving

**READ:**
- `process/engagement-state.yaml` (check user feedback from Phase 2)
- All `process/preliminary-*.yaml` files (build on Phase 2 findings)
- `references/methodology/partner-guide.md` (for Partner review criteria)
- `references/methodology/fact-checker-guide.md` (for Fact-Checker role)

Based on user feedback from Phase 2, you now go deeper on specific areas.

**Mindset: Deep insightful problem solving with thorough validation**
- Go as deep as needed to actually solve the problem
- Validate hypotheses thoroughly, understand mechanisms deeply, analyze trade-offs
- Develop actionable recommendations with confidence

### Deploy Targeted Experts

Based on user feedback, deploy the same 4-6 experts for deep validation. Each writes to `process/deep-<workstream-name>.yaml`.

### Fact-Checker Verification

After all Business Experts complete deep research, dispatch the Fact-Checker again:

```python
SendMessage(
    type="message",
    recipient="fact-checker",
    content="All experts have completed Phase 3 deep research. Please verify data quality across all deep-*.yaml files. This is the final validation before recommendations. Check for:

    1. Source reliability and verification
    2. Cross-workstream contradictions
    3. Data consistency with Phase 2 findings
    4. Any claims that need additional validation

    Write your findings to process/fact-check-phase3.yaml with any issues flagged.",
    summary="Phase 3 fact-check request"
)
```

Wait for Fact-Checker to complete. If issues found, re-dispatch affected experts with corrections.

### Pivot Check (MANDATORY after validation)

After experts return and fact-checking is complete, ask:

1. **"Does what we just learned change the issue tree?"**
   - If YES → update `process/issue-tree.yaml`, spawn new experts, run another round
   - If NO → proceed to meeting

2. **"Do any validated findings invalidate other experts' assumptions?"**
   - If YES → re-dispatch affected experts with corrected inputs

3. **"Is the core question still the right question?"**
   - If NO → loop back to Phase 1 with user

### Phase 3 Final Meeting (MANDATORY)

Conduct Phase 3 meeting with all team members:

**Meeting structure:**
1. **Each expert presents** deep findings and validated conclusions
2. **Partner reads expert YAMLs during the meeting** and gives strategic feedback:
   - Reviews each expert's validation work in real-time
   - Checks for insightfulness, creativity, problem-solving effectiveness
   - Identifies any remaining gaps or issues
   - Evaluates recommendation quality
   - Gives specific feedback to each agent
3. **Fact-Checker presents** data quality assessment and any remaining concerns
4. **Deliverable Advisor participates** to ensure findings support final deliverable:
   - Reviews findings from presentation perspective
   - Confirms data formats work for visualization
   - Identifies any gaps that would weaken deliverable
   - Starts planning deliverable structure
5. **PL synthesizes** findings across workstreams:
   - Connects insights across branches
   - Assesses storyline coherence
   - Proposes recommendations
   - Identifies risks and mitigations

**Discussion topics:**
- Review final findings across all workstreams
- Ensure recommendations are defensible
- Resolve any remaining contradictions
- Assess storyline coherence
- Confirm deliverable structure (Deliverable Advisor input)
- Validate evidence strength for each recommendation

If Partner or Fact-Checker flags issues during the meeting, address them before proceeding to Phase 4.

**UPDATE:** After Phase 3 completes, update `process/engagement-state.yaml`:
```yaml
current_phase: "Phase 4"
last_updated: "<ISO 8601>"
deep_research_complete: true
partner_review_complete: true
fact_check_complete: true
phase3_meeting_complete: true
```

---

## Phase 4: Final Checkpoint ⚠️ REQUIRED

**READ:**
- `process/engagement-state.yaml` (check Phase 3 completion status)
- All `process/deep-*.yaml` files (for deliverable structure planning)

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
5. Competitive Positioning - Differentiation strategy and brand positioning
6. Recommendations - Phased approach with timeline and resource requirements
7. Risks & Mitigations - Key risks and mitigation strategies
8. Appendix - Detailed data, methodology, benchmarking

Direction: The deliverable will recommend Amazon DTC launch with phased approach,
based on strong market evidence and validated cost advantage. Eco-friendly positioning
creates meaningful differentiation in target segment.

Evidence strength: Strong on market size, cost structure, and competitive landscape.
Moderate on channel economics (limited comparable data). Weak on brand positioning
effectiveness (requires customer research to validate).

Ready to proceed with building this?
```

### ★ USER SIGN-OFF (mandatory)

User must approve before deliverable generation.

**UPDATE:** After user sign-off, update `process/engagement-state.yaml`:
```yaml
current_phase: "Phase 5"
last_updated: "<ISO 8601>"
checkpoints_completed: ["preliminary_findings", "final_checkpoint"]
deliverable_structure_approved: true
```

---

## Phase 5: Deliverable Creation

**READ:**
- `process/engagement-state.yaml` (confirm user approval)
- `references/methodology/bcg-patterns.md` (MUST read before building deliverable)
- All `process/deep-*.yaml` files (source material for deliverable)
- `references/workflow/pre-delivery-checklist.md` (before presenting)

The Deliverable Advisor builds the final output as a consulting-style narrative.

**Before building the deliverable, the Deliverable Advisor MUST read `references/methodology/bcg-patterns.md`.** This contains structural patterns, headline styles, data presentation conventions, and narrative frameworks.

### Structure Proposal First

Before building the full deliverable, the Deliverable Advisor proposes the structure to the PL for quick approval:

```
SendMessage(
  type="message",
  recipient="<PL name or main session>",
  content="Here's my proposed structure for the deliverable:

  1. Executive Summary (1-2 slides) — [key recommendation]
  2. Basis of Perspectives (1 slide) — sources and methodology
  3. Market Viability (4-5 slides) — [H1, H2 findings with detailed evidence]
  4. Cost Advantage Analysis (4-5 slides) — [H3, H4 findings with benchmarking]
  5. Channel Strategy (5-6 slides) — [H5, H6 findings + detailed economics]
  6. Competitive Positioning (3-4 slides) — [H7, H8 findings + positioning map]
  7. Recommendations (3-4 slides) — phased approach with timeline, resources, KPIs
  8. Risks & Mitigations (2-3 slides) — key risks with mitigation strategies
  9. Appendix (5-7 slides) — detailed data tables, methodology, benchmarking

  Total: ~30-35 slides. Does this structure work, or should I adjust?

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

1. **Executive Summary** — 7-15 key conclusions and insights as bullet points. Lead with the answer.
2. **Situation / Context** — Where things stand today.
3. **Storylines** — 4-5 connected storylines. Each has:
   - A headline — at the problem/domain level, state a clear "so what"
   - Evidence: data, charts, benchmarks, business cases
   - Connection to the next storyline
4. **Recommendations** — Specific, actionable next steps with prioritization, timeline, and resource requirements
5. **Risks & Mitigations** — Key risks with specific mitigation strategies
6. **Appendix** — Detailed data, methodology, additional analysis, benchmarking

### Review Loop (3 versions max)

1. **Version 1:** Deliverable Advisor produces first draft → PL + Partner do full review → list ALL issues at once
2. **Version 2:** Deliverable Advisor fixes all issues → PL + Partner re-review affected sections → catch anything new
3. **Version 3:** Final polish → if still issues, present to user with noted imperfections

After 3 versions, diminishing returns kick in. Present what you have.

### Pre-Delivery Checklist ⚠️ REQUIRED

See `references/workflow/pre-delivery-checklist.md` — verify every item before presenting.

**UPDATE:** After deliverable is complete, update `process/engagement-state.yaml`:
```yaml
current_phase: "Phase 6"
last_updated: "<ISO 8601>"
deliverable_complete: true
deliverable_files: ["<list of files created>"]
```

---

## Phase 6: Next Steps & Resumability

**READ:**
- `process/engagement-state.yaml` (check deliverable completion)
- All process files (for next steps recommendations)

### Next Steps Categories

**Category A: Things that CAN be researched now.** If a next step involves publicly available data, provide exact URLs, databases, and search queries.

**Category B: Things that require human action.** For each, **generate a ready-to-use document** saved to `next-steps/`:
- Expert/stakeholder interview guides → `next-steps/interview-guide-<topic>.md`
- Customer survey drafts → `next-steps/survey-<topic>.md`
- Internal data request specs → `next-steps/data-request-<topic>.md`
- Meeting agendas → `next-steps/meeting-agenda-<topic>.md`
- Pilot program specifications → `next-steps/pilot-program-<topic>.md`

### Save Engagement State

Write `process/engagement-state.yaml` for resumability.

### Framing

Frame the deliverable as complete on its own, but with a clear pull:

> "Here's the best analysis I can build with what's available. To take this further, these are the areas where better data would sharpen the conclusions — [specific items]. If you come back with any of these, I can show you how the picture changes."

**UPDATE:** After Phase 6 completes, update `process/engagement-state.yaml`:
```yaml
current_phase: "Complete"
last_updated: "<ISO 8601>"
engagement_complete: true
next_steps_documented: true
```

---

## Shutdown

After Phase 6, gracefully shut down teammates:

```python
SendMessage(type="shutdown_request", recipient="partner", content="Work complete")
SendMessage(type="shutdown_request", recipient="fact-checker", content="Work complete")
SendMessage(type="shutdown_request", recipient="deliverable-advisor", content="Work complete")
SendMessage(type="shutdown_request", recipient="expert-a", content="Work complete")
SendMessage(type="shutdown_request", recipient="expert-b", content="Work complete")
SendMessage(type="shutdown_request", recipient="expert-c", content="Work complete")
SendMessage(type="shutdown_request", recipient="expert-d", content="Work complete")
# Add expert-e, expert-f if spawned
```

Then cleanup:

```python
TeamDelete()
```
