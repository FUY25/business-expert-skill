# YAML Templates for Process Files

All agents must write structured YAML files (or .md files if YAML fails) to `process/`. Use these exact formats.

**Resilience: If YAML write fails, use .md instead:**
- If writing to `process/*.yaml` fails → write to `process/*.md` instead
- Markdown format is acceptable for all process files
- Keep the same structured format, just use markdown syntax instead of YAML

---

## Contents

- [YAML Writing Tips & Troubleshooting](#yaml-writing-tips--troubleshooting) - Common errors, validation, fallback patterns
- [Workstream Findings (Business Experts)](#workstream-findings-business-experts) - Preliminary and deep research formats
- [Fact-Check Results (Fact-Checker Agent)](#fact-check-results-fact-checker-agent) - Data verification format
- [Meeting Notes (Fact-Checker Agent)](#meeting-notes-fact-checker-agent) - Phase 2 and Phase 3 meeting documentation
- [Issue Tree](#issue-tree) - Strategic question tracking and hypothesis validation
- [Partner Review](#partner-review) - Quality gate assessment format
- [Engagement State (Resumability)](#engagement-state-resumability) - Session state tracking for resume capability
- [Next Steps Proposal](#next-steps-proposal) - Post-engagement action items

---

## YAML Writing Tips & Troubleshooting

### Common YAML Syntax Errors

**1. Indentation (CRITICAL):**
- Use **2 spaces** for indentation (NOT tabs)
- Consistent indentation is required - mixing spaces/tabs will fail
```yaml
# ✅ CORRECT
key_findings:
  - finding: "Market growing"
    confidence: high

# ❌ WRONG (tabs or 4 spaces)
key_findings:
	- finding: "Market growing"
    confidence: high
```

**2. Special Characters in Strings:**
- Quote strings containing: `:` `,` `#` `&` `*` `!` `|` `>` `%` `@` `` ` ``
```yaml
# ✅ CORRECT
finding: "Market size: $12B"
source: "Report from 2025: Q4 analysis"

# ❌ WRONG (unquoted colons)
finding: Market size: $12B
source: Report from 2025: Q4 analysis
```

**3. Multi-line Strings:**
- Use `|` for literal (preserves newlines) or `>` for folded (wraps lines)
```yaml
# ✅ CORRECT
implications: |
  This finding suggests three things:
  1. Market is large enough
  2. Growth is accelerating
  3. Competition is fragmented

# ❌ WRONG (unquoted multi-line)
implications: This finding suggests three things:
  1. Market is large enough
```

**4. Empty Values:**
- Use `""` for empty strings, `[]` for empty arrays
```yaml
# ✅ CORRECT
evidence_summary: ""
data_gaps: []

# ❌ WRONG (missing values)
evidence_summary:
data_gaps:
```

### When to Use .md Fallback

Use markdown format if:
1. YAML write fails with syntax error
2. File permission issues
3. Content is very large (>500 lines)
4. You're unsure about YAML syntax

**Markdown equivalent:**
```markdown
# Workstream: market-viability

**Phase:** preliminary
**Agent:** expert-market
**Status:** complete

## Key Findings

### Finding 1
- **Finding:** Market growing at 3.2% CAGR
- **Confidence:** high
- **Source:** Euromonitor 2025
- **Implications:** Large enough to support new entrants
```

### Validation Before Writing

Before writing YAML, mentally check:
- [ ] All indentation is 2 spaces (no tabs)
- [ ] Strings with `:` or special chars are quoted
- [ ] Multi-line strings use `|` or `>`
- [ ] No empty values without `""` or `[]`
- [ ] Lists use `- ` (dash + space)

### If Write Fails

1. **Check error message** - often indicates line number
2. **Try .md fallback immediately** - don't retry YAML multiple times
3. **Keep same structure** - just use markdown headings/bullets instead of YAML keys

---

## Workstream Findings (Business Experts)

**Business Experts create TWO files per workstream - one for each phase:**

### Preliminary Research (Phase 2)

File: `process/preliminary-<workstream-name>.yaml`

```yaml
workstream: "<name of this workstream>"
phase: "preliminary"
agent: "<agent identifier>"
problem_scope: "<the question this agent is answering>"
timestamp: "<ISO 8601>"
status: "complete"
key_findings:
  - finding: "<one-sentence conclusion>"
    confidence: high|medium|low
    source: "<where this came from>"
    implications: "<what this means for the client's specific decision>"
    data_points:
      - metric: "<what was measured>"
        value: "<number or range>"
        year: <year>
        source_type: verified|model_estimate|derived
        source_url: "<url if verified, empty if model_estimate>"
data_gaps:
  - "<what we couldn't find>"
sources:
  - name: "<source name>"
    type: "<source type>"
    url: "<url if available>"
```

### Implications Field (MANDATORY)

Every agent must include an `implications` field in their key findings — what does this finding mean for the client's specific decision? This forces the thinking to happen during research, not just during synthesis.

```yaml
key_findings:
  - finding: "EU decorative paint market is €12B, growing 3.2% CAGR"
    confidence: high
    source: "Euromonitor 2025"
    implications: "Market is large enough to support a niche entrant — even 0.1% share = €12M revenue"
```

### `source_type` Rules (MANDATORY)

- `verified` — number comes from a specific, citable source. MUST have `source_url`.
- `model_estimate` — from model knowledge, not verified. Must be explicitly labeled. Treated as hypothesis, not evidence.
- `derived` — calculated from other data points. Show the formula.

**Data sourcing workflow:**
1. Search for the number using web/MCP/API
2. If found → cite as `verified` with URL
3. If not found after genuine search → label `model_estimate` and note in `data_gaps`

**Quality gate:** Agent with >10% `model_estimate` ratio has not done enough research. Re-dispatch with more specific search instructions.

### Case Study Standards in YAML

A one-line mention ("Clare raised $7.5M") is NOT a case study. For each comparable, collect at minimum:

**Case study depth:** For the 1-2 most relevant case studies, collect all 4 dimensions below. For supporting comparables, a brief mention with source is acceptable.

```yaml
case_studies:
  - company: "Clare Paint"
    strategy: "DTC-only eco paint via Shopify + Instagram"
    outcomes:
      - metric: "Revenue"
        value: "$7.5M Series A (2020)"
        source_type: verified
        source_url: "https://..."
      - metric: "Market position"
        value: "Top 5 DTC paint brand in US"
        source_type: model_estimate
    what_worked: "Strong brand identity, Instagram-first marketing, simplified SKU count"
    what_didnt: "Struggled with B2B/contractor channel, high CAC"
    lesson_for_client: "DTC paint can work but requires strong brand investment; contractor channel is a separate problem"
```

Each case study must have all 4 dimensions:
1. What they did (strategy/approach)
2. Key outcomes (quantitative metrics where available, qualitative results where not — both are valid)
3. What worked and what didn't
4. The specific lesson for this client's situation

### Deep Research (Phase 3)

File: `process/deep-<workstream-name>.yaml`

```yaml
workstream: "<name of this workstream>"
phase: "deep"
agent: "<agent identifier>"
problem_scope: "<the question this agent is answering>"
timestamp: "<ISO 8601>"
status: "complete"
hypotheses_validated:
  - hypothesis_id: "H1"
    statement: "<hypothesis from Phase 2>"
    status: supported|refuted|revised
    evidence_strength: strong|moderate|weak
key_findings:
  - finding: "<one-sentence conclusion>"
    confidence: high|medium|low
    source: "<where this came from>"
    implications: "<what this means for the client's specific decision>"
    data_points:
      - metric: "<what was measured>"
        value: "<number or range>"
        year: <year>
        source_type: verified|model_estimate|derived
        source_url: "<url if verified, empty if model_estimate>"
data_gaps:
  - "<what we couldn't find>"
sources:
  - name: "<source name>"
    type: "<source type>"
    url: "<url if available>"
```

**Key difference from preliminary:** Deep research includes `hypotheses_validated` section showing which hypotheses from Phase 2 were proven/refuted.

---

## Fact-Check Results (Fact-Checker Agent)

**Note: Fact-Checker only spawned for `--length 10min` and `10min+` engagements. For 3min and 5min, PL does inline fact-checking.**

**Fact-Checker creates ONE file per phase (batch mode):**

### After Preliminary Research

File: `process/fact-check-phase2.yaml`

```yaml
fact_check:
  phase: "preliminary"
  timestamp: "<ISO 8601>"
  workstreams_checked: ["market", "competitive", "gtm"]
  checked_data_points: <number>
  check_depth: "light-to-medium"
  results:
    - workstream: "market"
      metric: "EU decorative paint market size"
      claimed_value: "€12B"
      claimed_source: "Euromonitor 2025"
      source_url: "https://..."
      verification: verified|downgraded|upgraded|discrepancy|inaccurate
      actual_value: "€11.8B"
      notes: "Close enough — rounding difference"
    - workstream: "competitive"
      metric: "Specialty segment CAGR"
      claimed_value: "6.2%"
      claimed_source: "model knowledge"
      source_url: ""
      verification: "upgraded"
      actual_value: "5.8%"
      notes: "Found in Grand View Research 2025 report"
    - workstream: "gtm"
      metric: "Amazon DTC growth rate"
      claimed_value: "25% YoY"
      claimed_source: "Industry report"
      source_url: "https://..."
      verification: "inaccurate"
      actual_value: "18% YoY"
      notes: "Source says 18%, not 25%. Expert misread or misremembered."
  summary:
    verified_count: 5
    downgraded_count: 1
    upgraded_count: 1
    discrepancy_count: 1
    inaccurate_count: 1
    model_estimate_ratio_by_expert:
      market: 0.08
      competitive: 0.12
      gtm: 0.05
    risk_flags:
      - "Shipping cost estimate ($8.50/unit) is model_estimate and critical to margin calculation"
      - "Amazon growth rate was inaccurate (claimed 25%, actually 18%) - affects channel projections"
  cross_check_findings:
    - issue: "Market size definition inconsistency"
      workstreams: ["market", "gtm"]
      details: "Market expert uses €12B (DIY only), GTM expert uses €15B (includes professional)"
      resolution_needed: true
      resolution_owner: "PL or Partner to message affected experts"
```

**Verification types:**
- `verified` — Fact is accurate, source confirms the claim
- `downgraded` — URL dead or inaccessible, downgraded to model_estimate
- `upgraded` — Was model_estimate, found real source
- `discrepancy` — Multiple sources give different values
- `inaccurate` — Source exists but claimed number doesn't match what source actually says

### After Deep Research

File: `process/fact-check-phase3.yaml`

Same format as preliminary, but with:
- `phase: "deep"`
- `check_depth: "medium-to-heavy"`
- More thorough source verification (50%+ spot-checked, actually read for accuracy)
- More cross-checking across workstreams

---

## Meeting Notes (Fact-Checker Agent)

**Note: Meeting notes only created for `--length 10min` and `10min+` engagements. For 3min and 5min, no formal meeting notes are captured.**

**Fact-Checker captures meeting notes from SendMessage transcript:**

### Phase 2 Meeting

File: `process/meeting-phase2.yaml`

```yaml
meeting:
  phase: 2
  timing: "end"
  purpose: "Align on preliminary findings before user checkpoint"
  attendees: ["expert-market", "expert-competitive", "expert-gtm", "pl", "partner", "fact-checker"]
  timestamp: "<ISO 8601>"

  key_discussions:
    - topic: "Market size discrepancy"
      summary: "Expert A says €12B (DIY only), Expert B says €15B (includes professional). Resolved: €12B DIY is target market."

    - topic: "Cost advantage assumptions"
      summary: "Expert C's channel model assumes 30% advantage, but Expert A validated only 25%. Expert C will rerun model."

  decisions:
    - "Issue tree approved as-is, no restructuring needed"
    - "Phase 3 will focus 70% on cost/channel validation, 30% on market validation"

  action_items:
    - agent: "expert-competitive"
      action: "Clarify market size scope in preliminary YAML (DIY vs total)"
    - agent: "expert-gtm"
      action: "Rerun channel model with 25% cost advantage assumption in Phase 3"

  contradictions_resolved:
    - issue: "Market size definition"
      resolution: "Aligned on €12B DIY as target market, €15B total for context"

  issue_tree_updates:
    - change: "None - issue tree approved as-is"
```

### Phase 3 Start Meeting (Optional)

File: `process/meeting-phase3-start.yaml` (only created if user gives major change request)

```yaml
meeting:
  phase: 3
  timing: "start"
  purpose: "Address user's change request before Phase 3 validation"
  trigger: "User requested scope change between Phase 2 and Phase 3"
  attendees: ["expert-market", "expert-competitive", "expert-gtm", "pl", "partner", "fact-checker"]
  timestamp: "<ISO 8601>"

  user_change_request:
    summary: "<what the user asked for>"
    impact: "major|minor"

  key_discussions:
    - topic: "How does this change affect our issue tree?"
      summary: "<discussion summary>"

  decisions:
    - "<decisions made>"

  action_items:
    - agent: "<agent>"
      action: "<what needs to be done>"

  issue_tree_updates:
    - change: "<what changed in issue tree>"
```

### Phase 3 Final Meeting

File: `process/meeting-phase3-final.yaml`

```yaml
meeting:
  phase: 3
  timing: "final"
  purpose: "Final synthesis before Phase 4 checkpoint"
  attendees: ["expert-market", "expert-competitive", "expert-gtm", "pl", "partner", "fact-checker"]
  timestamp: "<ISO 8601>"

  key_discussions:
    - topic: "<discussion topic>"
      summary: "<summary>"

  decisions:
    - "<decisions made>"

  final_findings:
    - "<key finding>"

  recommendations_aligned:
    - "<recommendation>"

  contradictions_resolved:
    - issue: "<issue>"
      resolution: "<resolution>"

  storyline_coherence:
    assessment: "coherent|needs-work"
    notes: "<partner's assessment>"
```

---

## Issue Tree

**The issue tree is the single source of truth** for key questions, hypotheses, and progress state. **PL owns and must keep it updated** throughout the engagement. **All teammates refer to this file** to understand current strategic direction.

**Created at start of Phase 2 (before deploying experts).** Each branch maps to a workstream.

File: `process/issue-tree.yaml` (or `.md` if YAML fails)

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
        status: pending|supported|refuted|revised
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

**PL responsibilities:**
- Create initial issue tree at start of Phase 2 (before deploying experts)
- **Own and maintain this file** - it's the internal tracking document everyone refers to
- Each branch maps to a workstream (expert assignment)
- Update hypothesis statuses as findings come in (pending → supported/refuted/revised)
- Add/remove branches when pivot checks reveal changes needed
- Increment version and log changes in changelog
- **All teammates (Partner, Experts, Deliverable Advisor) look at this file** to understand current strategic direction
- This file is the engagement's strategic backbone

**Living document:** When validation adds, removes, or restructures branches:
1. Increment version
2. Update hypothesis statuses
3. Log the change in changelog
4. User sees current tree at each checkpoint, not the original

---

## Partner Review

**Partner focuses on strategic review, not data verification. Partner reviews expert findings + fact-check reports together.**

File: `process/partner-review-<checkpoint>.yaml`

```yaml
reviewer: "Partner"
checkpoint: "phase2" | "validation" | "final"
timestamp: "<ISO 8601>"
status: "approved" | "needs-revision"

# Phase 2 Review: Strategic Framing
# Focus: Is the issue tree MECE? Are we asking the right questions?
# Partner reads: preliminary-*.yaml files + fact-check-*-preliminary.yaml files

# Phase 3 Review: Evidence Quality
# Focus: Are conclusions supported? Is evidence solid?
# Partner reads: deep-*.yaml files + fact-check-*-deep.yaml files

assessment:
  storyline_coherence: "<are the hypotheses well-framed?>"
  evidence_gaps:
    - "<what's missing>"
  logic_issues:
    - "<what doesn't hold up>"
  skeptic_challenges:
    - "<what would a critic say>"

directives:
  - agent: "<which agent>"
    action: "<specific instruction>"

overall_verdict: "Ready for user checkpoint" | "Needs more work on [X]"
```

**Note:** Partner does NOT verify data points themselves - that's Fact-Checker's job. Partner reviews the fact-check reports and focuses on strategic assessment.

### Partner Restructure Directive

When the framing itself is wrong (not just weak evidence):

```yaml
directives:
  - agent: "lead"
    action: "restructure"
    reason: "The issue tree treats channel and positioning as independent, but validation shows they're deeply coupled — DTC pricing forces premium positioning, Amazon pricing enables value positioning."
    suggested_framing: "Restructure around positioning-channel combinations"
```

This sends the engagement back to Phase 2 for fundamental revision.

---

## Engagement State (Resumability)

File: `process/engagement-state.yaml`

```yaml
engagement:
  topic: "<user's original question>"
  project_folder: "<path>"
  created: "<ISO 8601>"
  last_updated: "<ISO 8601>"

current_phase: 3
phase_status:
  phase_0: complete
  phase_1: complete
  phase_2: complete
  phase_3: in_progress
  phase_4: pending
  phase_5: pending
  phase_6: pending

parameters:
  format: slides
  length: 10min
  level: analyst
  depth: standard
  sources: balanced
  lang: en

hypotheses:
  supported:
    - H1: "Market growing >3% CAGR"
    - H3: "Margins positive after tariffs"
  refuted:
    - H6: "Retail partnership viable under $500K"
  pending:
    - H2: "Specialty niches accessible"

data_gaps:
  - "Partner B financials — private company"
  - "Real freight quotes for Vietnam→UK route"

issue_tree_updates:
  - version: 1
    timestamp: "<ISO 8601>"
    change: "Initial tree created before Phase 2 research"
  - version: 2
    timestamp: "<ISO 8601>"
    change: "Added branch for positioning-channel coupling after Phase 2 findings"

findings_status:
  preliminary_complete: true
  deep_complete: false
  fact_check_phase2_complete: true
  fact_check_phase3_complete: false
  partner_review_phase2: "approved"
  partner_review_phase3: "pending"

checkpoints_completed:
  - checkpoint: "phase1_scope"
    timestamp: "<ISO 8601>"
    user_feedback: "Approved, proceed with research"
  - checkpoint: "phase2_preliminary"
    timestamp: "<ISO 8601>"
    user_feedback: "Good direction, focus more on cost analysis in Phase 3"

loop_back_count: 1
loop_back_history:
  - from_phase: 3
    to_phase: 2
    reason: "Positioning analysis revealed channel-positioning coupling"
    timestamp: "<ISO 8601>"
```

---

## Next Steps Proposal

File: `process/next-steps-proposal.yaml`

```yaml
next_steps:
  category_a_researchable:
    - action: "Get real container rates from Freightos"
      url: "https://fbx.freightos.com"
      route: "Vietnam→UK"
      priority: high
    - action: "Pull Amazon UK PPC benchmarks"
      tool: "Jungle Scout or Helium 10"
      keywords: "eco paint"
      priority: medium

  category_b_human_action:
    - action: "Interview 3 potential retail partners"
      materials_generated: "next-steps/interview-guide-retail-partners.md"
      priority: high
    - action: "Customer survey on eco-paint preferences"
      materials_generated: "next-steps/survey-eco-preferences.md"
      priority: medium
```
