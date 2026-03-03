# Single Agent Workflow (No Subagents, No Teams)

This workflow is for environments where neither Agent Teams nor subagents are available. The PL handles all research, analysis, and deliverable creation directly.

---

## When to Use This Workflow

Use this workflow when:
- `TeamCreate` tool is NOT available (no Agent Teams)
- `Agent` tool is NOT available (no subagents)
- You're the only agent working on the engagement

**Detection:** Check your available tools at the start of Phase 0. If neither `TeamCreate` nor `Agent` is present, use this workflow.

---

## Key Differences from Team Workflows

| Aspect | Team/Subagent Workflow | Single Agent Workflow |
|--------|------------------------|----------------------|
| Research | Parallel experts | Sequential by PL |
| Fact-checking | Dedicated agent | PL inline |
| Partner review | Separate agent | PL self-review |
| Deliverable | Deliverable Advisor | PL builds |
| Meetings | Team coordination | Internal checkpoints |
| Process files | Multiple YAML files | Consolidated files |

---

## Workflow Overview

```
Phase 0 → Phase 1 → Phase 2 → Phase 3 → Phase 4 → Phase 5 → Phase 6
(setup)  (scope)   (prelim)   (deep)    (checkpoint) (deliver) (next steps)
```

**All phases executed by PL directly.**

---

## Phase 0: Environment Check

**READ:** None (this is the first phase)

Same as other workflows, but:
- Skip Agent Teams detection
- Skip subagent permission checks
- Note to user: "I'll handle all research directly — no parallel agents available in this environment."

Create project folder as usual:
```bash
PROJECT_DIR="<slug-from-topic>"
mkdir -p "$PROJECT_DIR/process"
cd "$PROJECT_DIR"
```

**UPDATE:** After Phase 0 completes, create `process/engagement-state.yaml` with initial state:
```yaml
current_phase: "Phase 1"
last_updated: "<ISO 8601>"
checkpoints_completed: []
workflow_type: "single-agent"
```

---

## Phase 1: Scope & Clarification

**READ:** None (Phase 1 is scoping)

Same as other workflows. Use `AskUserQuestion` for format/length selection.

**Length interpretation for single agent:**
- `3min` → 1-2 workstreams, 5-8 slides
- `5min` → 2-3 workstreams, 10-15 slides
- `10min` → 3-4 workstreams, 20-25 slides
- `10min+` → 4-5 workstreams, 30+ slides

Adjust expectations: "Since I'm working solo, I'll focus on the most critical questions rather than comprehensive coverage."

**UPDATE:** After Phase 1 completes, update `process/engagement-state.yaml`:
```yaml
current_phase: "Phase 2"
last_updated: "<ISO 8601>"
scope_confirmed: true
```

---

## Phase 2: Preliminary Research

**READ:**
- `process/engagement-state.yaml` (check current phase)
- `references/methodology/partner-guide.md` (for self-review criteria)
- `references/templates/yaml-formats.md` (for output format)

### Build Issue Tree

Create `process/issue-tree.yaml` (or `.md` if YAML fails) as usual, but with fewer branches:
- 3min: 1-2 branches
- 5min: 2-3 branches
- 10min: 3-4 branches
- 10min+: 4-5 branches

### Sequential Research

Instead of spawning experts, **you do the research sequentially** for each workstream:

1. **For each branch of the issue tree:**
   - Research the key question using WebFetch, Bash curl, search MCPs
   - Collect data, benchmarks, case studies
   - Write findings to `process/preliminary-<workstream>.yaml` (or `.md`)
   - Include all standard fields: key_findings, data_points, implications, sources

2. **Consolidate as you go:**
   - After each workstream, update `process/issue-tree.yaml` with findings
   - Note cross-workstream connections
   - Flag contradictions or gaps

### Inline Fact-Checking

After completing all preliminary research:
1. Review all `process/preliminary-*.yaml` files
2. Sanity check key data points (source_type, source_url presence)
3. Cross-check for contradictions
4. Flag high model_estimate ratios (>10%)

### Self-Review (Partner Role)

Since there's no Partner agent, **you perform a self-review**:

**Read `references/methodology/partner-guide.md`** and apply the review criteria to your own work:
- Storyline coherence: Do the pieces fit together?
- Evidence gaps: Where is analysis thin?
- Logic issues: Any conflicting assumptions?
- Skeptic challenges: What would a critic ask?

Write self-review to `process/self-review-phase2.yaml` (or `.md`):
```yaml
self_review:
  phase: "preliminary"
  timestamp: "<ISO 8601>"
  assessment:
    storyline_coherence: "<evaluation>"
    evidence_gaps:
      - "<gap 1>"
      - "<gap 2>"
    logic_issues:
      - "<issue 1>"
    skeptic_challenges:
      - "<challenge 1>"
  action_items:
    - "<what to address in Phase 3>"
  overall_verdict: "Ready for user checkpoint" | "Needs more work on [X]"
```

If you identify issues, address them before proceeding.

**Communication Discipline:** Do NOT show findings to the user until the formal checkpoint below. Complete internal work first.

### Preliminary Findings Checkpoint

Present findings to user as usual. Wait for feedback before Phase 3.

**UPDATE:** After checkpoint, update `process/engagement-state.yaml`:
```yaml
current_phase: "Phase 3"
last_updated: "<ISO 8601>"
checkpoints_completed: ["preliminary_findings"]
user_feedback: "<summary of user feedback>"
self_review_phase2_complete: true
```

---

## Phase 3: Deep Problem Solving

**READ:**
- `process/engagement-state.yaml` (check user feedback from Phase 2)
- All `process/preliminary-*.yaml` files (build on Phase 2 findings)
- `references/methodology/partner-guide.md` (for self-review criteria)

### Sequential Deep Research

Based on user feedback, go deeper on each workstream sequentially:

1. **For each branch requiring deep validation:**
   - Conduct targeted research
   - Validate hypotheses thoroughly
   - Write findings to `process/deep-<workstream>.yaml` (or `.md`)
   - Update `process/issue-tree.yaml` with validated hypotheses

2. **Cross-workstream synthesis:**
   - After completing all deep research, synthesize findings
   - Resolve contradictions
   - Connect insights across workstreams

### Inline Fact-Checking

Same as Phase 2, but more thorough:
- Verify ALL key data points
- Cross-check consistency with Phase 2 findings
- Validate critical assumptions

### Pivot Check

After deep research, ask yourself:
1. Does what I learned change the issue tree?
2. Do any findings invalidate other assumptions?
3. Is the core question still right?

If YES to any → loop back and address.

### Self-Review (Partner Role)

Perform final self-review using `references/methodology/partner-guide.md`:

Write self-review to `process/self-review-phase3.yaml` (or `.md`):
```yaml
self_review:
  phase: "deep"
  timestamp: "<ISO 8601>"
  assessment:
    storyline_coherence: "<evaluation>"
    evidence_strength: "<strong/moderate/weak areas>"
    insightfulness: "<obvious vs non-obvious insights>"
    creativity: "<fresh thinking or conventional?>"
    problem_solving: "<does this solve the user's problem?>"
  final_findings:
    - "<key finding 1>"
    - "<key finding 2>"
  recommendations_aligned: true|false
  overall_verdict: "Ready for Phase 4" | "Needs revision"
```

Address any issues before proceeding.

**UPDATE:** After Phase 3 completes, update `process/engagement-state.yaml`:
```yaml
current_phase: "Phase 4"
last_updated: "<ISO 8601>"
deep_research_complete: true
self_review_phase3_complete: true
```

---

## Phase 4: Final Checkpoint

**READ:**
- `process/engagement-state.yaml` (check Phase 3 completion status)
- All `process/deep-*.yaml` files (for deliverable structure planning)

Present deliverable structure to user as usual. Get sign-off before building.

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

### Read BCG Patterns

**Read `references/methodology/bcg-patterns.md`** before building deliverable.

### Choose Output Format

Based on user's format choice:
- **Slides (HTML):** Read `deliverable-tools/frontend-slides/slide-tool.md` using Read tool
- **Slides (PPTX):** Read `deliverable-tools/pptx/pptx-tool.md` using Read tool
- **Document (DOCX):** Read `deliverable-tools/docx/docx-tool.md` using Read tool
- **Markdown:** Built-in, no skill needed
- **Notion:** Use Notion MCP

### Build Deliverable

**You build the deliverable directly** (no Deliverable Advisor):

1. **Structure proposal:** Draft outline, present to user for approval
2. **Build deliverable:** Follow the nested skill instructions
3. **Self-review:** Check against `references/workflow/pre-delivery-checklist.md`
4. **Iterate if needed:** Max 2-3 versions

**For slides, ALWAYS produce BOTH:**
1. Slides (html or pptx) — for presenting
2. Markdown document (.md) — for depth and reading

### Pre-Delivery Checklist

Run through `references/workflow/pre-delivery-checklist.md` before presenting.

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

Same as other workflows:
- Category A: Researchable now (provide URLs, queries)
- Category B: Human action required (generate ready-to-use documents)
- Save `process/engagement-state.yaml` for resumability

**UPDATE:** After Phase 6 completes, update `process/engagement-state.yaml`:
```yaml
current_phase: "Complete"
last_updated: "<ISO 8601>"
engagement_complete: true
next_steps_documented: true
```

---

## Process Files Structure

```
<project-folder>/
├── slides.html (or report.md)     # Final deliverable
├── process/
│   ├── engagement-state.yaml      # Phase tracking
│   ├── issue-tree.yaml            # Living issue tree
│   ├── preliminary-*.yaml         # Phase 2 findings (one per workstream)
│   ├── deep-*.yaml                # Phase 3 findings (one per workstream)
│   ├── self-review-phase2.yaml    # PL self-review after Phase 2
│   ├── self-review-phase3.yaml    # PL self-review after Phase 3
└── next-steps/
    ├── interview-guide-*.md
    ├── survey-*.md
    └── data-request-*.md
```

---

## Quality Considerations

### Strengths of Single Agent Workflow
- **Coherence:** Single voice, consistent analysis
- **Efficiency:** No coordination overhead
- **Simplicity:** Straightforward execution

### Limitations
- **Coverage:** Less breadth than parallel experts
- **Perspective:** Single viewpoint, less diversity
- **Speed:** Sequential research takes longer
- **Self-review:** Less rigorous than independent Partner review

### Mitigation Strategies
1. **Focus ruthlessly:** Prioritize highest-impact questions
2. **Leverage external sources:** Use case studies, expert quotes for diverse perspectives
3. **Structured self-review:** Follow Partner guide rigorously
4. **User checkpoints:** Get feedback early and often
5. **Acknowledge limitations:** Be transparent about single-agent constraints

---

## Time Expectations

Adjust user expectations for single-agent work:

| Length | Team Workflow | Single Agent Workflow |
|--------|---------------|----------------------|
| 3min | 30-45 min | 45-60 min |
| 5min | 60-90 min | 90-120 min |
| 10min | 2-3 hours | 3-4 hours |
| 10min+ | 4-6 hours | 5-7 hours |

**Tell the user upfront:** "Since I'm working solo, this will take longer than with parallel agents. I'll focus on the most critical questions to deliver maximum value."

---

## Resumability

Single-agent workflow is fully resumable:
- `process/engagement-state.yaml` tracks progress
- `--resume` flag picks up where you left off
- All process files preserved for continuity

---

## When to Recommend Enabling Agents

If the user has a complex, multi-faceted problem (4+ key questions), suggest enabling Agent Teams or subagents:

> "This problem has multiple dimensions that would benefit from parallel research. If you enable Agent Teams (add to settings.json), I can deploy specialized experts to cover more ground faster. See `references/workflow/setup-guide.md` for instructions."

Don't block on this — proceed with single-agent workflow if user declines.
