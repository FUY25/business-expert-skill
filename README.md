# Business Expert Skill

A structured approach to business strategy that helps you analyze markets, evaluate opportunities, and make informed decisions. It asks the right questions, does the research, and delivers actionable recommendations backed by evidence.

## What This Skill Does

This isn't a chatbot that summarizes Wikipedia articles. It's a systematic way to work through business problems:

- **Starts with the right question** — Most business problems are poorly framed. It spends time upfront making sure you're solving the real problem, not just the stated one.
- **Builds hypotheses, then tests them** — Rather than boiling the ocean with research, it forms a point of view early and seeks evidence to prove or disprove it. Faster, sharper, more decisive.
- **Delivers complete packages** — You get both the presentation (slides for stakeholders) and the full analysis (detailed document for those who want depth). No more choosing between brevity and rigor.
- **Maintains quity gates** — Work gets reviewed before you see it. If the analysis is thin, the logic is weak, or the evidence doesn't support the conclusion, it gets sent back. You only see work that meets the bar.

## When You'd Use This

### Market Entry & Expansion
- "Should we enter the EU B2C market?"
- "Which Southeast Asian market should we prioritize?"
- "Is this acquisition target worth pursuing?"

### Pricing & Positioning
- "How should we price our new product?"
- "Can we raise prices without losing customers?"
- "Where do we sit in the competitive landscape?"

### Growth Strategy
- "How do we grow from $10M to $50M ARR?"
- "Should we expand horizontally or go deeper in our core?"
- "What's our path to market leadership?"

### Operational Decisions
- "Should we build or buy this capability?"
- "Where can we cut costs without hurting growth?"
- "Is our go-to-market strategy working?"

### Investment & Due Diligence
- "Is this market attractive enough to invest?"
- "What's the realistic TAM/SAM/SOM?"
- "What are the real risks here, not just the obvious ones?"

## How It Works

The skill uses structured thinking principles adapted from strategy consulting to break down complex problems systematically:

**Hypothesis-driven approach.** Instead of gathering all available data first, it forms a point of view about what's likely true, then seeks evidence to validate or refute it. This moves faster without sacrificing rigor.

**MECE decomposition.** Every problem gets broken down into components that are Mutually Exclusive (no overlap) and Collectively Exhaustive (nothing missing). This prevents analysis paralysis and ensures nothing gets missed.

**Pyramid principle.** Lead with the answer, then support it with storylines, then back each storyline with evidence. You get the recommendation upfront, then can drill down into the reasoning.

**"So what?" test.** Every data point must answer "so what does this mean for the decision?" If it doesn't drive the recommendation, it's noise.

### The 6-Phase Workflow

**Phase 1: Scope & Clarification**
Restates your problem and asks clarifying questions. What decision is this informing? Who's the audience? What constraints matter? This prevents wasted work downstream.

**Phase 2: Landscape Research & Preliminary Findings**
Initial exploration to establish baseline understanding. Researches the market, competitive landscape, trends, and develops preliminary insights. This is internally iterative — refines as it learns. At the end, you get a checkpoint: "Here's what we've learned so far. Does this framing make sense?"

**Phase 3: Hypothesis Validation & Recommendations**
Based on your feedback, goes deeper on specific areas. Validates hypotheses with targeted research, develops recommendations, iterates until the analysis is solid. Another checkpoint before building the deliverable.

**Phase 4: Final Checkpoint**
You see the complete storyline with evidence map before anything gets built. This is your last chance to redirect. It's far cheaper to restructure an outline than to redo a 20-slide deck.

**Phase 5: Deliverable Creation**
Builds both the slides (for presenting) and the full document (for reading). Contextual file names like `Market_Entry_Strategy_EU_Paint_v1.md`, not generic `report.md`. Multiple revision rounds with quality checks.

**Phase 6: Next Steps & Resumability**
Specific follow-up actions, not vague "do more research." If a next step involves publicly available data, it tries to find it or gives you the exact lookup path. If it requires human action (interviews, internal data), it generates ready-to-use guides.

### The Team Structure

**The skill coordinates multiple specialized agents:**

- **Project Lead (PL)** — Manages the engagement, synthesizes findings, owns the narrative arc. This is the main Claude session.
- **Partner** — Quality gate. Reviews all work before you see it. Can send work back if it doesn't meet the bar.
- **Business Experts** — Each owns a problem-scoped question (not a data domain). They research, analyze, and deliver findings.
- **Deliverable Advisor** — Builds the final output with format-specific expertise (slides, documents, etc.).

This team structure ensures quality (Partner review), depth (Experts go deep on specific questions), and coherence (PL synthesizes into a clear story).

## Key Concepts

### Internally Iterative Phases

Phase 2 and Phase 3 aren't one-shot. They iterate internally until the work is solid:
- **Phase 2:** Research → insights → refine → repeat until preliminary findings are ready
- **Phase 3:** Validate → discover gaps → research deeper → repeat until satisfied

The PL and Partner decide when to move forward. This prevents premature conclusions and ensures depth.

### The Square Analogy

Think of research space as 2D: X-axis = breadth, Y-axis = depth.
- **Phase 2** covers 1/4 of the square (moderate breadth, shallow depth) — preliminary findings
- **Phase 3** can go anywhere in the remaining space (deep on specific areas, or broader if needed)

The PL decides where to focus Phase 3 based on your feedback from Phase 2. This flexibility prevents wasted effort.

### Issue Tree: Two Exposure Levels

The issue tree has one version but two exposure levels:
- **Internal** (saved to process files): Key questions + detailed hypotheses (H1, H2, H3)
- **User-facing**: Key questions + verbal context (no granular hypotheses)

You see "What are the key questions we'll answer?" not "Here are our hypotheses H1, H2, H3 that we'll test." Less methodology, more substance.

### Mandatory Checkpoints

Three checkpoints ensure alignment:
1. **Scope Checkpoint (Phase 1):** Confirm we're solving the right problem
2. **Preliminary Findings Checkpoint (Phase 2):** Share initial research, get feedback before deep dive
3. **Final Checkpoint (Phase 4):** Sign off on storyline before deliverable

These aren't bureaucracy — they prevent wasted work. It's far cheaper to redirect after Phase 2 than to redo the entire analysis.

### Problem Delegation, Not Task Delegation

Business Experts receive problems, not tasks. Instead of "Go research the EU paint market and give me a report," they get "Can our cost advantage survive tariffs and shipping to remain competitive?" The Expert owns the how, the PL owns the what.

This approach gets better results. You give smart agents hard problems and let them figure out the approach. The PL reviews their plan before they execute.

### Always Both Formats

When you request slides, you get both:
1. **Slides** (pptx or html) — for presenting to stakeholders
2. **Markdown document** (.md) — for reading and depth

Slides are great for presentations but lack detail. The companion document provides the full analysis for anyone who wants to understand the reasoning.

## Professional Standards

### Content Density

**Documents:** Every major section must have a framing paragraph, 3+ pieces of evidence with sources, at least one comparison or benchmark, and explicit implications for your decision.

**Slides:** 60-80% of slides should be evidence slides (headline + 2-3+ evidence + source). The rest can be framing, transitions, or recommendations. No sparse slides with just a headline and 2 generic bullets.

### Minimum Output by Length

| Length | Slides | Words | Team Size | Use Case |
|--------|--------|-------|-----------|----------|
| 3min | 5-8 | 800-1200 | 2-3 experts | Quick decisions, focused questions |
| 5min | 10-15 | 2000-2500 | 3-4 experts | Typical strategy work |
| 10min | 20-25 | 4000-5000 | 4-5 experts | Detailed evidence, benchmarking |
| 10min+ | 30+ | 6000+ | 5-6 experts | Full strategic review |

These are minimums. If the research produces 20K words of findings, the deliverable should reflect that coverage, not compress it into a thin summary.

### Business Strategy Professional Mindset

**Action-oriented headlines.** Slide headlines state conclusions, not topics. "Market is attractive for new entrants" not "Market Overview."

**Implications after every benchmark.** Don't just say "Competitor X has 47% margin." Say what it means for your decision. "Their 47% margin vs. our 32% suggests we're underpricing or have a cost structure problem."

**Specific numbers, not vague language.** "15% CAGR from '23 to '30" not "significant growth." Precision builds credibility.

**Named competitors, not "industry peers."** Identify 3-5 specific companies. "Uniqlo, Heilan, Nike" not "industry peers."

**Show your sources early.** Include a "Basis of Perspectives" slide showing what research you did, what sources you used, and any limitations. This builds credibility before you make any claims.

**"From... To..." framing for transformation.** When recommending change, show current state vs. future state. Makes the recommendation concrete.

**Ranges over false precision.** "$590M-$650M" not "$623M." Acknowledge uncertainty rather than pretending you know the exact number.

## Installation

```bash
# Clone the repository
git clone https://github.com/FUY25/business-expert-skill.git

# Copy to Claude skills directory
cp -r business-expert-skill ~/.claude/skills/business-expert
```

## Usage

```bash
# Basic usage
claude "Should we enter the EU B2C paint market?"

# With parameters
claude "Analyze the EU market entry opportunity --format pptx --length 10min --level executive"

# Resume previous engagement
claude "Continue the market analysis --resume"
```

### Parameters

- `--format` — Output format: `md` (markdown), `slides` (HTML), `pptx` (PowerPoint), `docx` (Word), `notion`
- `--length` — Content coverage: `3min` (focused), `5min` (standard), `10min` (comprehensive), `10min+` (extensive)
- `--level` — Audience: `executive` (high-level, decisions), `analyst` (balanced detail), `technical` (full detail)
- `--sources` — Source credibility: `strict` (tier-1 only), `balanced` (default), `broad` (includes blogs, case studies)
- `--lang` — Output language: `en`, `zh`, etc.

## Directory Structure

```
business-expert/
├── SKILL.md                    # Main skill definition
├── README.md                   # This file
├── references/
│   ├── workflow/               # How to execute
│   │   ├── phases-overview.md  # 6-phase workflow details
│   │   ├── setup-guide.md      # Installation and configuration
│   │   ├── data-sources.md     # Available MCPs, APIs, libraries
│   │   └── pre-delivery-checklist.md  # Quality gates
│   ├── methodology/            # Consulting principles
│   │   ├── agent-teams-guide.md  # Agent teamwork and coordination
│   │   ├── partner-guide.md    # Partner review standards
│   │   └── bcg-patterns.md     # Presentation patterns
│   └── templates/              # Formats and schemas
│       ├── yaml-formats.md     # YAML schemas for process files
│       ├── output-documents.md # Document structure and standards
│       └── output-slides.md    # Slide structure and standards
├── scripts/
│   └── validate_process.py     # Data quality validation
├── skills/                     # Nested output skills
│   ├── frontend-slides/        # HTML slides generation
│   ├── pptx/                   # PowerPoint generation
│   └── docx/                   # Word document generation
└── evals/
    └── evals.json              # Test cases
```

## Output Formats

### Markdown (.md)
Always produced. Provides full analysis with depth and detail. Good for reading, sharing with team, archiving.

### HTML Slides (.html)
Animated, runs in browser, zero dependencies. Good for remote presentations, sharing via link.

### PowerPoint (.pptx)
Traditional slide deck. Editable by user after generation. Good for corporate environments.

### Word Document (.docx)
Formal report format. Good for official documentation, compliance, archiving.

### Notion
Collaborative, team-editable. Good for living documents that evolve over time.

**Important:** When you request slides (HTML or PPTX), you get BOTH the slides and a companion .md document. Slides for presenting, .md for depth.

## What Makes This Different

**It's not a research assistant.** Research assistants gather information. This solves problems. There's a difference between "here's what I found about the EU paint market" and "you should enter via Amazon DTC with a phased UK-first approach because..."

**It's not a report generator.** Report generators summarize data. This builds arguments. Every section connects to the recommendation. If it doesn't drive the decision, it's cut.

**It's not a single-shot tool.** You get checkpoints throughout. Scope checkpoint, preliminary findings checkpoint, final checkpoint. This prevents wasted work and ensures alignment.

**It's not just slides or just documents.** You get both. Slides for presenting, documents for depth. No more choosing between brevity and rigor.

**It's not a solo agent.** Multiple specialized agents work together (PL, Partner, Experts, Deliverable Advisor). The Partner ensures quality. The Experts go deep. The PL synthesizes.

## Contributing

This skill is actively developed based on real business strategy practice. Contributions welcome — especially from those with strategy consulting or business analysis experience.

## License

MIT License

## Credits

Built using structured thinking principles from strategy consulting. Designed for practitioners who need more than summaries — they need decisions.
