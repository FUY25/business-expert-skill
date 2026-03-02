# Workflow Phases Overview

The engagement follows six phases (plus Phase 0 setup). This file provides detailed instructions for each phase.

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

3. **Agent Teams** — Check if `TeamCreate` is in tools. See `references/agent-teams-guide.md` for setup.

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

---

## Phase 1: Scope & Clarification ⚠️ REQUIRED

The most important phase. Misaligned scope wastes everything downstream.

### What to Do

1. Restate user's problem in your own words
2. Ask clarifying questions — don't assume
3. Align on scope: what's in, what's out, what decision is being made
4. Skip questions answered by flags (`--format`, `--length`, `--level`)

### Mandatory Questions (unless flags set)

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

### Example Clarification Questions

- "The core question is [X]. Is that accurate?"
- "Are we looking at this from a [company/investor/regulator] perspective?"
- "What geographies are in scope?"
- "Do you have internal data I should factor in?"
- "Who's the audience, and what decision will this inform?"
- Ask if they have internal data, proprietary research, or specific datasets
- Ask about the user's background and what they care about — this shapes the analysis

### Partner Consultation (Agent Teams)

Before presenting the scope checkpoint to the user, the PL should message the Partner for a quick sanity check on the proposed scope:

```
SendMessage(
  type="message",
  recipient="partner",
  content="Before I confirm scope with the user, here's what I'm proposing: [scope summary]. Any red flags? Is this the right question to answer?",
  summary="Scope sanity check"
)
```

The Partner may flag:
- Scope too broad or too narrow for the `--length` setting
- Missing a critical angle the user probably cares about
- Framing that will lead to an unhelpful answer

This is a quick check, not a formal review. If the Partner has concerns, adjust before presenting to the user.

### ★ SCOPE CHECKPOINT (mandatory)

**Before presenting this checkpoint, verify you have explicitly asked the user about output format (unless `--format` was set). If you haven't asked yet, ask now — do not present the checkpoint with an assumed format.**

User confirms problem statement, scope, and output format. Do not proceed without this.

---

## Phase 2: Landscape Research & Preliminary Findings

This is the initial phase of the project. You establish baseline understanding through broad exploration, develop preliminary insights, and form hypotheses based on findings. This phase is **internally iterative** — research, analyze, refine, repeat until preliminary findings are solid.

### What Phase 2 Covers (PL Decides)

The PL decides what topics to cover based on the engagement. Common areas include:
- Market context (sizing, growth, trends, segmentation)
- Competitive landscape (players, dynamics, positioning)
- Customer insights (needs, willingness to pay, behavior)
- Regulatory/legal environment
- Cost structure and economics
- Strategic fit and capabilities
- Technology and innovation trends
- Risk factors

**The PL selects 3-5 areas most relevant to the engagement.** These are not prescriptive — choose what's needed to establish baseline understanding and form the issue tree.

### Agent Deployment by Length

- `--length 3min`: Use model knowledge + 1-2 targeted searches
- `--length 5min`: 2-3 focused agents (default)
- `--length 10min`: 3-4 agents covering more ground
- `--length 10min+`: 4-5 agents, broader coverage

### Every Agent Must Write YAML

Include in every agent prompt:
> "Before you return, write your findings to `process/<workstream-name>.yaml` using the format in `references/templates/yaml-formats.md`."

After agents return: `ls process/*.yaml` — re-dispatch any agent that didn't write its file.

### Phase 2 is Internally Iterative

```
Research → Insights → Form hypotheses → More research needed?
   ↑                                            ↓
   └────────────── Yes, refine ────────────────┘
                      ↓
                   No, ready for checkpoint
```

**PL and Partner decide when to iterate:**
- Do we have enough baseline understanding?
- Are the preliminary findings solid enough to present?
- Do we need more research before forming the issue tree?

Iterate within Phase 2 until satisfied, then proceed to checkpoint.

### Phase 2 Output: Preliminary Findings (1/4 of the Square)

Think of the research space as a 2D square: X-axis = breadth, Y-axis = depth.

**Phase 2 covers 1/4 of the square:** Moderate breadth, shallow depth. You establish baseline understanding across selected areas, but these findings are **preliminary** — room to go deeper in Phase 3.

Example preliminary findings:
- Market: "EU decorative paint is €12B, growing 3.2% CAGR" (preliminary - based on 2-3 sources)
- Competitive: "Top 3 players hold 45% share" (preliminary - high-level view)
- Insight: "Gap in mid-tier eco segment" (preliminary - pattern observed)
- Internal: "Your cost advantage might survive tariffs" (preliminary - rough calculation)

### Build MECE Issue Tree (Internal Structure)

The issue tree has one version with two exposure levels:

**Internal structure** (saved to `process/issue-tree.yaml`):
```yaml
core_question: "Should we launch B2C paint in EU/US?"
branches:
  - key_question: "Is the market attractive?"
    hypotheses:
      - id: H1
        statement: "Decorative segment growing >3% CAGR"
      - id: H2
        statement: "Specialty niches have room for new entrants"
  - key_question: "Does our cost advantage survive?"
    hypotheses:
      - id: H3
        statement: "Margins positive after tariffs + shipping"
```

**User-facing presentation** (ASCII tree with verbal context):
```
Core question: Should we launch B2C paint in EU/US?

├── Market viability
│   We're testing whether the decorative paint segment is growing fast
│   enough to support new entrants, and whether specialty eco-friendly
│   niches have room for a new player like you.
│
├── Cost advantage survivability
│   We're checking if your 38% manufacturing cost advantage can survive
│   after we factor in tariffs, shipping, and EU compliance costs.
│
└── Channel feasibility
    We're evaluating two paths: Amazon DTC (lower upfront cost) vs.
    retail partnerships (higher reach but more complex).
```

**Key principle:** Show the structure and directional context to the user, but keep granular hypotheses (H1, H2, H3) internal.

### Partner Review (MANDATORY)

Partner reviews preliminary findings and issue tree. Write to `process/partner-review-phase2.yaml`.

**For detailed Partner review questions and quality criteria, see `references/methodology/partner-guide.md`.**

### PL Consults Partner (Agent Teams)

Before presenting to the user, the PL should message the Partner:

```
SendMessage(
  type="message",
  recipient="partner",
  content="Here are the preliminary findings and issue tree: [summary]. Does this framing make sense? Are we asking the right questions?",
  summary="Phase 2 review"
)
```

The Partner may flag:
- MECE violations (overlapping or missing branches)
- Wrong level of abstraction
- Missing critical angles
- Findings too thin to present

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

**Example:**
```
Preliminary Findings (Phase 2 of 6)

Market Context:
• EU decorative paint market is €12B, growing 3.2% CAGR
• Specialty eco-friendly segment growing 2x market rate (6.8% CAGR)
• US market is larger but more fragmented

Competitive Landscape:
• Top 3 players hold 45% share, focused on mass market
• Limited eco-friendly offerings in mid-tier segment
• DTC channel still nascent (only 8% of sales)

Initial Insights:
• There's a gap in the mid-tier eco-friendly segment
• Incumbents are slow to reformulate (2-3 year cycles)
• Amazon DTC channel growing 25% YoY in home improvement

Key Questions We'll Investigate Next:
├── Can your cost advantage survive tariffs and shipping?
├── Is Amazon DTC viable with your budget constraints?
└── What's the risk to your OEM relationships?

Does this make sense? Should we adjust focus before diving deeper?
```

Wait for user feedback before proceeding to Phase 3.

---

## Phase 3: Hypothesis Validation & Recommendations

Based on user feedback from Phase 2, you now go deeper on specific areas. This phase is **internally iterative** — validate, discover gaps, research deeper, refine, repeat until PL and Partner are satisfied with the depth and recommendations.

### Phase 3 Can Go Anywhere in the Square

Phase 2 covered 1/4 of the square (moderate breadth, shallow depth). Phase 3 can go anywhere in the remaining space:
- **Go deep** on specific areas (e.g., "Need precise cost advantage numbers")
- **Go broader** if needed (e.g., "Should we also look at adjacent markets?")
- **Validate hypotheses** with targeted research
- **Develop recommendations** based on validated findings

**The PL decides where to focus based on:**
- User feedback from Phase 2 checkpoint
- What's most critical to the decision
- Where preliminary findings were weakest

### Phase 3 is Internally Iterative

```
Validate → Discover gaps → More research needed?
   ↑                              ↓
   └────────── Yes, dive deeper ──┘
                  ↓
              No, ready for final checkpoint
```

**PL and Partner decide when to iterate:**
- Are the hypotheses validated with sufficient evidence?
- Do we have enough depth to make recommendations?
- Are there gaps that need more research?

Iterate within Phase 3 until satisfied, then proceed to Phase 4.

### Validation Round

1. Deploy targeted agents based on user feedback and PL priorities
2. Each writes to `process/validation-<hypothesis-name>.yaml`
3. Analyze: support or refute each hypothesis
4. **Cross-workstream contradiction check (MANDATORY).** Before proceeding, compare findings across all experts. If Expert A's finding contradicts Expert B's assumption — e.g., cost expert says "15-25% advantage" but channel expert's model assumes "30% advantage" — this is not a footnote. The downstream conclusions are built on a false premise. Resolve it: spawn a follow-up agent, have the PL reconcile directly, or re-dispatch the affected expert with corrected assumptions. Update both workstream YAMLs with the resolution.
5. Verify files exist: `ls process/*.yaml`

### Pivot Check (MANDATORY after each round)

After agents return and cross-expert contradictions are resolved, ask these three questions before proceeding:

1. **"Does what we just learned change the issue tree?"**
   - Did a major branch get refuted? → Remove or replace it in `process/issue-tree.yaml`
   - Did we discover a new dimension the tree doesn't cover? → Add a new branch with new hypotheses
   - Did the relative importance of branches shift? → Re-prioritize
   - If YES to any → update `process/issue-tree.yaml` (increment version, log the change), spawn new experts for new branches, and run another validation round on the new hypotheses. Tell the user: "The [X] analysis revealed something that changes the picture — [explain]. I'm updating the issue tree and running additional research on [new branch]."
   - If NO to all → proceed to Partner review

2. **"Do any validated findings invalidate other experts' assumptions?"**
   - Check each expert's key assumptions against other experts' findings
   - If an expert used an assumption that's now contradicted → re-dispatch that expert with corrected inputs
   - This is the cross-pollination step that prevents siloed analysis

3. **"Is the core question still the right question?"**
   - Sometimes validation reveals the user's real problem is different from what was scoped. E.g., "should we enter B2C?" might really be "should we diversify away from OEM dependency?"
   - If the core question has shifted → **loop back to Phase 1**. Tell the user: "Based on what I've found, I think the real question might be [X] rather than [Y]. Can we re-scope?"
   - This is rare but important. Don't silently pivot without the user's buy-in.

### Verify Agent Outputs

Run the validation script to catch common data quality issues:

```bash
python scripts/validate_process.py process/
```

This script checks:
- All expected agents wrote YAML files
- Every key data point has a `source_type` field (verified, model_estimate, or derived)
- `model_estimate` ratio ≤10% per agent (if higher, agent needs more specific search instructions)
- No stale sources (older than domain-specific thresholds)
- Cross-agent discrepancies for the same metric

Review the warnings and decide whether to re-dispatch agents or proceed. The Partner review is the final quality gate.

Manual verification if script unavailable:
```bash
ls process/*.yaml
```

Check each YAML for:
- Missing `source_type` on data points → send agent back to label every data point
- `model_estimate` ratio >10% for any agent → re-dispatch that agent with more specific search instructions
- Stale sources (older than domain threshold) → flag for Partner review

Before proceeding to Partner review, confirm all expected agents have written their files and data is labeled.

### Partner Review (MANDATORY)

**Partner review (MANDATORY).** After validation and pivot check, the Partner reviews ALL findings.

**For detailed Partner review questions and quality criteria, see `references/methodology/partner-guide.md`.**

**The Partner can trigger a restructure.** If the Partner judges that the framing itself is wrong — not just weak evidence, but wrong questions — they can send the engagement back to Phase 2 with a `action: "restructure"` directive in their review YAML. This is different from sending an individual agent back. A restructure means: the issue tree needs fundamental revision, new experts need to be defined, and the existing validation may need to be re-evaluated under the new framing.

Write review to `process/partner-review-validation.yaml`. **Verify the file exists** before proceeding. If the Partner says restructure → go back to Phase 2. If the Partner says findings don't solve the problem → do another validation round.

**When to iterate (do another round):**
- The Partner flags that key hypotheses are unresolved or the evidence is weak
- A finding contradicts the overall framing and the issue tree needs restructuring
- The user provided new information that changes the problem
- `--length 10min+`: always do at least 2 validation rounds

**When to stop:**
- The Partner's verdict is "findings are solid, ready for user checkpoint"
- All high-priority hypotheses are validated or clearly refuted with evidence
- Diminishing returns — another round won't materially change the recommendation

**Working checkpoints (optional):** During validation, share findings whenever meaningful or when you need user input. These are lightweight — "here's what I'm seeing, any reactions?" Use your judgment on when these are helpful.

### Fact-Check Step

Models tend to generate plausible-sounding numbers from training data without verifying them. This step catches that. After agents return their findings and after `validate_process.py` runs:

1. **Identify the top 5-8 most impactful data points** — the numbers that drive the recommendation. Focus on: market sizes, growth rates, competitor financials, cost figures, and any number used in a calculation.
2. **For each `verified` data point:** Attempt to access the cited `source_url`. If the URL is reachable, check that the number actually appears in the source (or is reasonably close). If the URL is dead or the number doesn't match, downgrade to `model_estimate` and flag it.
3. **For each `model_estimate` data point:** Do one more targeted search to try to find a real source. If found, upgrade to `verified` with the URL. If not, keep as `model_estimate` — but if this number is critical to the recommendation, flag it as a risk in the Partner review.
4. **Cross-check numbers across agents.** If two agents cite different values for the same metric (e.g., different market size figures), flag the discrepancy and determine which source is more authoritative.
5. **Write results to `process/fact-check-phase{N}.yaml`** (format in `references/templates/yaml-formats.md`).

The fact-check results feed into the Partner review. The Partner should pay special attention to any `downgraded` or `discrepancy` items and decide whether they weaken the storyline enough to require another research round.

---

## Phase 4: Final Checkpoint ⚠️ REQUIRED

Last collaborative moment before deliverable.

**Partner review (MANDATORY).** One final Partner review before presenting the storyline to the user. The Partner checks the complete narrative arc — do the storylines connect? Does the evidence support the recommendation? **Write to `process/partner-review-final.yaml`** using the same Partner review YAML structure. **Verify the file exists** before presenting to the user.

**For detailed Partner review questions and quality criteria, see `references/methodology/partner-guide.md`.**

### Present to User

- Full storyline outline (3-5 storylines)
- Key conclusions with evidence
- Evidence map as ASCII tree:
```
Recommendation: Go via Amazon DTC, phased launch
├── Storyline 1: Market is attractive
│   ├── [HIGH] EU decorative paint €12B, 3.2% CAGR ← Euromonitor 2025
│   ├── [MED]  Specialty segment growing 2x market ← industry interviews
│   └── [GAP]  US specialty segment size — no reliable source found
├── Storyline 2: Cost advantage holds after tariffs
│   ├── [HIGH] 38% cost gap survives tariffs ← customs data + model
│   └── [LOW]  Shipping costs stable ← freight index (volatile)
└── Storyline 3: Amazon DTC is feasible under $500K
    ├── [HIGH] Launch cost ~$180K ← 3 comparable case studies
    └── [MED]  Breakeven at 14 months ← financial model
```

Use `[HIGH]`, `[MED]`, `[LOW]` for confidence and `[GAP]` for missing data. This lets the user see exactly where the analysis is strong and where it's thin before signing off.

### ★ USER SIGN-OFF (mandatory)

User must approve before deliverable generation.

**The user must sign off before you proceed to deliverable generation.** This is non-negotiable. It's far cheaper to restructure an outline than to redo a 20-slide deck.

---

## Phase 5: Deliverable Creation (iterative)

The Deliverable Advisor builds the final output as a consulting-style narrative. This is NOT a one-shot handoff — the PL and Partner review the deliverable and send it back for revision until it meets the quality bar.

**Before building the deliverable, the Deliverable Advisor MUST read `references/methodology/bcg-patterns.md`.** This contains structural patterns, headline styles, data presentation conventions, and narrative frameworks distilled from real BCG decks. Follow these patterns — especially: action-oriented headlines (not topic-only), "Implications to [client]" after every benchmark, "Basis of Perspectives" slide showing sources/interviews, and "From... To..." framing for transformation narratives.

### Deliverable Requirements

**IMPORTANT: Always produce BOTH formats when user requests slides:**

1. **Slides** (pptx or html) — for presenting
   - Follows slide standards: modular, visual, concise
   - Good for presentations and quick scanning

2. **Markdown document** (.md) — for depth and reading
   - Follows document standards: linear narrative, detailed analysis, full evidence
   - Good for reading, understanding depth, sharing with stakeholders who want detail
   - This is ALWAYS produced alongside slides, even if user only asked for slides

**File naming:** Use contextual, professional names based on the engagement topic:
- ✅ `Market_Entry_Strategy_EU_Paint_v1.md` and `Market_Entry_Strategy_EU_Paint_v1.pptx`
- ✅ `Pricing_Strategy_Analysis_CompanyX_v1.md` and `Pricing_Strategy_Analysis_CompanyX_v1.html`
- ❌ `report.md` and `slides.pptx` (too generic)

Use underscores or hyphens for file names, include version number (v1, v2, etc.), and keep naming consistent between the .md and slides files.

### Structure Proposal First

**Structure proposal (lightweight approval before building):**

Before building the full deliverable, the Deliverable Advisor proposes the structure to the PL for quick approval. This catches structural issues early — it's much cheaper to restructure an outline than to redo a 20-slide deck.

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

The PL reviews and either approves or suggests adjustments (e.g., "Combine market viability and cost into one section, expand channel strategy"). This is a quick check, not a formal plan approval — the Deliverable Advisor doesn't need `mode="plan"`.

### Review Loop (3 versions max)

**Deliverable review loop (3 versions max):**
1. **Version 1:** Deliverable Advisor produces first draft → PL does full review → lists ALL issues at once
2. **Version 2:** Deliverable Advisor fixes all issues → PL re-reviews affected sections → catches anything new or introduced by fixes
3. **Version 3:** Final polish → if still issues, present to user with noted imperfections and let them decide

After 3 versions, diminishing returns kick in. Present what you have. Only exceed 3 versions if the user explicitly requests further refinements.

The business-expert skill owns the **content structure** (what goes where, narrative arc, which evidence supports which claim). The nested skill file owns the **visual execution** (layout, fonts, styling, chart rendering). The Lead's review bridges both — ensuring the visual output faithfully represents the analytical work.

**Output structure (Pyramid Principle):**
1. **Executive Summary** — 5-15 key conclusions and insights as bullet points. Lead with the answer.
2. **Situation / Context** — Where things stand today.
3. **Storylines** — 3-5 connected storylines. Each has:
   - A headline — at the problem/domain level, state a clear "so what"; individual slides can be descriptive
   - Evidence: data, charts, benchmarks, business cases
   - Connection to the next storyline
4. **Recommendations** — Specific, actionable next steps with prioritization
5. **Risks & Mitigations** *(optional)* — Include when the decision carries meaningful downside. Not every deliverable needs this.
6. **Appendix** — Detailed data, methodology, additional analysis

**Charts:** Use wherever they make data more digestible. Every chart answers a specific question. Label axes, include units, add a one-line takeaway caption.

**Adapting charts to the medium:**
- **Markdown:** Use ASCII tables, Mermaid diagrams, or describe chart data in structured tables
- **HTML slides:** Use inline SVG, Chart.js, or D3-style visualizations
- **PPTX:** Use pptxgenjs chart objects (bar, line, pie, scatter)
- **DOCX:** Use well-formatted tables or embedded chart images
- **Notion:** Use formatted tables, or generate chart images and embed them

**Citations:** Group sources into a references section per slide or per storyline. Don't scatter inline. Every claim traceable, narrative flows clean.

**Nested skill loading rule:** Do NOT read any nested skill files until Phase 5 (Deliverable Creation). During Phases 0-4, only reference the output formats table to confirm the chosen format is available. When Phase 5 begins, the Deliverable Advisor reads ONLY the single nested skill file for the chosen format using the Read tool (NOT the Skill tool) — never load multiple. The Skill tool cannot find nested skills at `skills/*/SKILL.md` paths.

### Pre-Delivery Checklist ⚠️ REQUIRED

See `references/workflow/pre-delivery-checklist.md` — verify every item before presenting.

---

## Phase 6: Next Steps & Resumability

### Next Steps Categories

**Category A: Things that CAN be researched now.** If a next step involves publicly available data — shipping rate databases, regulatory filing portals, industry association reports, government trade data — don't just say "get shipping quotes." Actually try to find them during the engagement, or at minimum provide the exact URLs, databases, and search queries the user would need. Examples:
  - "Get real container rates from Freightos (https://fbx.freightos.com) for Vietnam→UK route" — not just "check shipping costs"
  - "Look up REACH registration fees at ECHA (https://echa.europa.eu/fees-for-registration) for your specific substance list" — not just "budget for REACH"
  - "Pull Amazon UK PPC benchmarks from Jungle Scout or Helium 10 for the 'eco paint' keyword cluster"

If the skill had web access during the engagement, these should already be partially answered. If not, provide the exact lookup path so the user can do it in 10 minutes, not 10 hours.

**Category B: Things that require human action.** Interviews, site visits, internal data pulls, partnership conversations. For each of these, **generate a ready-to-use document** saved to the project folder:

  - **Expert/stakeholder interview guides** → save as `next-steps/interview-guide-<topic>.md`
    - 8-12 questions, ordered by priority
    - Opening context paragraph the interviewer can read aloud
    - Follow-up probes for each question
    - "What to listen for" notes (what answers would change the recommendation)

  - **Customer survey drafts** → save as `next-steps/survey-<topic>.md`
    - Screener questions to qualify respondents
    - 10-15 core questions with response scales
    - Open-ended questions for qualitative insight

  - **Internal data request specs** → save as `next-steps/data-request-<topic>.md`
    - Exact fields/metrics needed
    - Time period and granularity
    - Who likely owns this data (finance, ops, sales)
    - How it will be used in the analysis

  - **Meeting agendas** → save as `next-steps/meeting-agenda-<topic>.md`
    - Objective, attendees, time allocation
    - Key questions to resolve
    - Pre-read materials (reference the deliverable sections)

**Create a `next-steps/` subfolder** in the project directory for all these materials. The user should be able to hand these documents directly to their team without editing.

Save the summary to `process/next-steps-proposal.yaml` and include as an appendix in the deliverable.

### Save Engagement State

Write `process/engagement-state.yaml` for resumability.

### Framing

**Resumability framing:** Frame the deliverable as complete on its own, but with a clear pull:

> "Here's the best analysis I can build with what's available. To take this further, these are the areas where better data would sharpen the conclusions — [specific items]. If you come back with any of these, I can show you how the picture changes."

**Engagement state:** Save `process/engagement-state.yaml` tracking:
- Current phase and status
- Validated hypotheses
- Pending work and data gaps
- Output format and deliverable location
- Parameters used: format, length, level, sources, lang

When the skill triggers in a new session, Phase 0 handles resumability — check for `--resume`/`--new` flags or prompt the user if `process/` exists. When the user returns with new data (interview transcripts, survey results, expert notes), read them as new inputs, update relevant hypotheses, and generate an updated deliverable — showing what changed.

---

## Content Coverage Pattern

For Phase 2 (hypothesis formation) or when you need to cover more ground during Phase 3, adjust your approach based on `--length`:

**Focused exploration (`--length 3min`):**
Deploy 1-2 targeted searches or use model knowledge to quickly form hypotheses. Appropriate when the question is narrow and well-defined.

**Standard exploration (`--length 5min`):**
Deploy 2-3 Business Experts in parallel, each covering a specific question. This is the default approach for typical strategy questions.

**Broader exploration (`--length 10min` or `10min+`):**
Deploy 3-5 Business Experts to cover more ground. This is appropriate when entering an unfamiliar domain, when the question has multiple dimensions, or when the user wants comprehensive coverage.

**When to use which:**
- Phase 2 (with `--length 10min+`) → Broader exploration to cover more questions
- Phase 3 → Targeted validation. You know what you're looking for.
- If a checkpoint reveals a blind spot → Targeted deep dive on that specific area.

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
