# Agent Teams Guide

**This file covers:** All aspects of agent teamwork — setup, coordination patterns, role descriptions, prompt templates, and troubleshooting.

**For phase-by-phase workflow, see:** `workflow/phases-overview.md`

---

## Contents

- [⚠️ CRITICAL: Teams vs Subagents](#critical-teams-vs-subagents) - The #1 failure mode
- [Team Structure](#team-structure) - Team composition by engagement length (3min/5min/10min+)
- [Detecting Agent Teams Availability](#detecting-agent-teams-availability) - Check if TeamCreate is available
- [Agent Teams Setup — Tool Call Sequence](#agent-teams-setup--tool-call-sequence) - How to spawn teammates
- [Problem Delegation, Not Task Delegation](#problem-delegation-not-task-delegation) - How to write expert prompts
- [File Writing Discipline](#file-writing-discipline) - Large file handling and chunked writing
- [Meeting Requirements and Communication Discipline](#meeting-requirements-and-communication-discipline) - When to communicate
- [Team Roles](#team-roles) - Business Experts, Partner, Fact-Checker, Deliverable Advisor
- [Agent Web Access Setup](#agent-web-access-setup) - Permissions and fallback patterns
- [Hub-and-Spoke Fallback](#hub-and-spoke-fallback-subagents-available-no-teams) - When Agent Teams unavailable
- [Single-Agent Fallback](#single-agent-fallback-no-subagents-no-teams) - When no subagents available
- [Teammate Prompt Templates](#teammate-prompt-templates) - Ready-to-use prompts for each role
- [Common Mistakes](#common-mistakes) - Troubleshooting table
- [Coordination Rules](#coordination-rules) - Team coordination principles

---

## ⚠️ CRITICAL: Teams vs Subagents

**This is the #1 failure mode of this skill.** Read this section carefully.

```
┌─────────────────────────────────────────────────────────────────────┐
│  THE GOLDEN RULE                                                     │
│                                                                      │
│  Agent(..., team_name="X")  →  TEAMMATE  →  "X teammates running"   │
│  Agent(...) without team_name  →  SUBAGENT  →  "X agents running"   │
│                                                                      │
│  If you see "X agents running" when spawning Business Experts,      │
│  Partner, or Deliverable Advisor — YOU DID IT WRONG.                │
│                                                                      │
│  The core team MUST ALWAYS use team_name.                           │
└─────────────────────────────────────────────────────────────────────┘
```

### Why This Matters

- **Teammates** can message each other, share tasks, coordinate peer-to-peer
- **Subagents** only report back to caller — no peer communication
- Using subagents for the core team breaks the entire coordination model

### When to Use Each

| Role | Use | Parameter |
|------|-----|-----------|
| Partner | TEAMMATE | `team_name="<project>-strategy"` |
| Fact-Checker | TEAMMATE | `team_name="<project>-strategy"` |
| Business Expert A/B/C | TEAMMATE | `team_name="<project>-strategy"` |
| Deliverable Advisor | TEAMMATE | `team_name="<project>-strategy"` |
| Data-fetching intern (spawned BY an expert) | subagent | NO team_name |

---

## Team Structure

```
PL (main session — you)
  │  Directions, storytelling, narrative arc, user communication
  │
  ├── Partner (teammate) — Quality gate, facilitates meetings, reviews Fact-Checker reports
  │
  ├── Fact-Checker (teammate) — Data integrity verification, meeting notes
  │
  ├── Business Expert A (teammate) — Problem scope 1 ─┐
  ├── Business Expert B (teammate) — Problem scope 2  ├── peer-to-peer + can spawn subagents
  ├── Business Expert C (teammate) — Problem scope 3 ─┘
  │
  └── Deliverable Advisor (teammate) — Format/presentation, alive throughout
```

**You are the PL (Project Lead).** You manage phases, checkpoints, user interaction, and narrative direction, and the issue tree file. You decide how many Business Experts to deploy based on complexity. You synthesize findings into a coherent storyline and present to the user at checkpoints.

**Tiered team structure by --length:**

The team structure and process adapt based on engagement complexity:

**`--length 3min` (Quick Analysis):**
- Team: PL + 2 Business Experts (subagents, not teammates) + Partner (teammate, spawned Phase 0)
- Process:
  - Phase 0: Spawn Partner
  - Phase 2: Partner reviews issue tree → Research → PL reviews → User checkpoint
  - Phase 3: Deep dive → Partner review (reads YAMLs)
  - No meetings, no Fact-Checker, no Deliverable Advisor
  - PL does fact-checking inline and builds deliverable (respects user's format choice)

**`--length 5min` (Standard - DEFAULT):**
- Team: PL + 3 Business Experts (teammates) + Partner (teammate, spawned Phase 0) + Deliverable Advisor (teammate, on-demand)
- Process:
  - Phase 0: Spawn Partner
  - Phase 2: Partner reviews issue tree → Research → PL sanity check → User checkpoint
  - Phase 3: Deep dive → MEETING (Deliverable Advisor spawned, Partner reads YAMLs and gives strategic feedback, Deliverable Advisor gives presentation feedback) → Experts get feedback to edit the research results → ask for Partner approval gates
  - 1 meeting only (Phase 3 final, includes Deliverable Advisor), partner have the power to ask agents or pl to pivot or redo some tasks
  - PL does fact-checking inline
  - Deliverable Advisor builds deliverable (respects user's format choice)

**`--length 10min` and `10min+` (Comprehensive):**
- Team: Full team (PL + 4-6 Business Experts + Partner + Fact-Checker + Deliverable Advisor, all teammates)
- Process: Current flow (2 meetings, Partner throughout, Fact-Checker, Deliverable Advisor)

**Partner is spawned in Phase 0 for ALL lengths** to review issue tree and provide strategic feedback:
- 3min: Reviews issue tree, provides final review in Phase 3 (no meetings)
- 5min: Reviews issue tree, participates in Phase 3 meeting
- 10min+: Reviews issue tree, participates in Phase 2 and Phase 3 meetings
---

## Detecting Agent Teams Availability

Check if `TeamCreate` is in your available tools. If yes → use Agent Teams. If not → tell user and proceed with hub-and-spoke fallback.

**If Agent Teams Not Available:**

Tell user:
> "Agent Teams isn't enabled. I'll use parallel agents instead — same quality, less coordination. To enable for next time, add to `~/.claude/settings.json`:"

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Then restart Claude Code. Point to `references/workflow/setup-guide.md` for details. Don't block — proceed with hub-and-spoke immediately.

---

## Agent Teams Setup — Tool Call Sequence

**Note:** These are the tool calls for setting up the team, NOT the engagement phases. For the 6-phase workflow (Phase 0: Environment Check → Phase 1: Scope → Phase 2: Research → etc.), see `workflow/phases-overview.md`.

### 1. Create the Team

```python
TeamCreate(
    team_name="<project-slug>-strategy",
    description="Strategy engagement for <topic>"
)
```

**Do this FIRST, before spawning any teammates.**

### 2. Spawn Core Team (Phase 0)
**Remember Tiered team structure by --length:**

**Lazy spawning:** Spawn Partner, Fact-Checker(if applicable), and Deliverable Advisor(if applicable) in Phase 0. Business Experts are spawned later in Phase 2 after the issue tree is built.



```python
# Partner — no plan approval needed (reviewer, not researcher)
Agent(
    prompt="<Partner prompt>",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",   # ← REQUIRED
    name="partner",                          # ← REQUIRED
    model="opus"
)

# Fact-Checker — no plan approval needed (data verification and meeting notes)
Agent(
    prompt="<Fact-Checker prompt>",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",   # ← REQUIRED
    name="fact-checker",                     # ← REQUIRED
    model="sonnet"
)

# Deliverable Advisor — no plan approval needed (advisory role)
Agent(
    prompt="<Deliverable Advisor prompt>",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",   # ← REQUIRED
    name="deliverable-advisor",              # ← REQUIRED
    model="opus"
)
```

### 3. Build Issue Tree (Phase 2)

Before spawning Business Experts, build the issue tree with 3-5 key questions. Each branch becomes a workstream.

### 4. Spawn Business Experts (Phase 2, After Issue Tree)

**Lazy spawning:** Spawn Business Experts only after you know what questions to answer. Each expert owns one branch of the issue tree.

```python
# Business Experts — USE mode="plan" for plan approval
Agent(
    prompt="<problem scope prompt for branch 1>",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",   # ← REQUIRED
    name="expert-a",                         # ← REQUIRED
    model="<PL-selected model>",             # ← Use model selected by PL in Phase 0
    mode="plan"                              # ← REQUIRES PLAN APPROVAL
)

Agent(
    prompt="<problem scope prompt for branch 2>",
    subagent_type="general-purpose",
    team_name="<project-slug>-strategy",   # ← REQUIRED
    name="expert-b",                         # ← REQUIRED
    model="<PL-selected model>",             # ← Use model selected by PL in Phase 0
    mode="plan"                              # ← REQUIRES PLAN APPROVAL
)

# Add more experts as needed based on issue tree branches
```

Experts with `mode="plan"` submit plans via ExitPlanMode. PL receives `plan_approval_request` messages.

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

### Plan Approval Criteria

**Expert Plan Format:** Plans should be 100-200 words maximum. Concise, focused plans force clarity of thinking. If an expert needs 500 words to explain their approach, the scope is too broad or the thinking is too vague.

Check each expert plan for:
1. **Hypothesis-driven:** Clear hypothesis to test, not just "research X"
2. **Scope clarity:** Bounded, no overlap with other experts
3. **Evidence strategy AND deliverable specifications:**
   - Proposed sources and types identified (quantitative data, benchmarks, case studies, logical reasoning chains, qualitative observations, expert opinions)
   - Concrete deliverables specified (not "analyze X" but "identify 5-10 X with Y, Z attributes")
   - Benchmark framing decided (median vs top quartile, adjacent industries for comparison, qualitative vs quantitative measures)
   - Output format clear (comparison table, positioning map, ranked list, driver tree)
   - Data availability confirmed or user asked to provide it
4. **Output commitment:** Commits to writing YAML to `process/`
5. **Analytical approach:** Will produce conclusions, not data dumps

Reject vague plans ("I'll research the market") or overlapping scopes or irrelevant topics

**If critical data is unavailable:** Expert or PL should ask the user to provide it. Don't silently work around missing data. Be specific: "To validate [hypothesis], I need [specific data]. Can you: (1) Set up [MCP name], (2) Provide API credentials for [service], or (3) Download [dataset] from [source]?"

### 6. Teammates Work

After plans approved:
- Experts execute research, can spawn subagents (WITHOUT team_name) for data gathering
- Deliverable Advisor messages experts with format advice (if applicable)
- Experts message each other to share cross-findings
- All mark tasks completed via TaskUpdate
- Experts can ask questions to PL for questions clarification

### 7. Shutdown

```python
SendMessage(type="shutdown_request", recipient="partner", content="Work complete")
SendMessage(type="shutdown_request", recipient="expert-a", content="Work complete")
SendMessage(type="shutdown_request", recipient="expert-b", content="Work complete")
SendMessage(type="shutdown_request", recipient="deliverable-advisor", content="Work complete")
```

### 8. Cleanup

```python
TeamDelete()
```

---

## Problem Delegation, Not Task Delegation

**This is the key to effective expert prompts.** The difference between a good expert prompt and a bad one is whether you're delegating a problem or a task list.

### The Anti-Pattern (Task Delegation)

```
❌ BAD — Execution-ready prompt:
"Use WebSearch to find tariff data for paint imports from Vietnam to EU/US.
Search for shipping costs on major freight routes. Calculate the margin impact
using our $8.50 base cost. Write findings to process/workstream-cost.yaml
using this format: [detailed YAML structure]"
```

**Why this fails:** When you give an expert a task list, they think "my plan is right here in the prompt." The plan approval step becomes a formality — they skip straight to execution because you already told them HOW to do it. This defeats the purpose of `mode="plan"`.

### The Correct Pattern (Problem Delegation)

```
✅ GOOD — Problem-scoped prompt with concrete deliverables:
"Your problem: Can our cost advantage survive tariffs, shipping, and compliance
costs to remain competitive in EU/US?

Submit a plan covering:
- Your hypothesis about cost survivability
- Evidence strategy (what you'll look for, where — data sources, benchmarks, case studies)
- Analytical approach (how you'll validate your hypothesis)
- Concrete deliverables you'll produce:
  * Cost breakdown table: our costs vs 3-5 competitors (with sources)
  * Tariff impact model: scenarios at 10%, 15%, 25% tariff rates
  * Benchmark: our landed cost vs industry median and top quartile
  * Sensitivity analysis: which cost component matters most
- Data needs: If you need data I don't have access to, tell me specifically what and why

After approval, execute and write to process/workstream-cost.yaml."
```

**Why this works:** The expert has to think: What's my hypothesis? What evidence validates it? Where do I find that evidence? How do I know if I'm right? What exactly will I deliver? **That's the plan they submit.** The PL reviews it, catches misalignment early, and approves before any work is wasted.

### Expert Prompts Should Be

- **Problem-scoped:** "Answer this question: [X]"
- **Constraint-bounded:** "Write YAML to process/, hypothesis-driven, Layer 2+ thinking"
- **Outcome-focused:** "I need analytical conclusions with implications, not data dumps"
- **Method-agnostic:** Don't prescribe the research path — let the expert plan it

This gives experts meaningful autonomy (they own the how) while the PL maintains control (they own the what and the quality bar).

### The Control Points

The PL doesn't need to micromanage execution if they have strong control points:

1. **Plan approval** — catches misalignment before wasted work
2. **Quality review** — catches quality issues when experts finished work 
3. **Cross-workstream check** — catches contradictions between experts
4. **Pivot check** — catches when findings change the issue tree
5. **PL can ask partner's opinion when need a second thought about important issues**

These gates give the PL control without needing to prescribe every step.

---

## File Writing Discipline

**Large file warning:** YAML files >500 lines or HTML slides with 20+ slides should use chunked writing or .md fallback.

**Never retry failed large writes.** If a large YAML write fails:
1. Switch to .md format immediately (don't retry YAML multiple times)
2. Or use chunked writing: Write initial structure, then append sections
3. For HTML slides: Use the chunked writing pattern built into the nested skill

**Why this matters:** Retrying the same failed large write wastes time and tokens. The first failure tells you the approach won't work — adapt immediately rather than hoping the same operation will succeed on retry.

---

## Meeting Requirements and Communication Discipline

**Communication discipline:** Teammates communicate ONLY at formal checkpoints, not continuously during research. This prevents constant interruptions and allows focused work.

**Meeting structure by engagement length:**

**`--length 3min` (Quick Analysis):**
- No formal meetings
- Partner reviews work asynchronously (reads YAML files)
- Communication happens at user checkpoints only

**`--length 5min` (Standard):**
- 1 meeting: Phase 3 final (Partner + Deliverable Advisor + Experts)
- Purpose: Strategic review before final deliverable
- Partner reviews Phase 2 work asynchronously

**`--length 10min` and `10min+` (Comprehensive):**
- 2 meetings: Phase 2 end + Phase 3 final
- Phase 2 meeting: Align on preliminary findings, resolve contradictions
- Phase 3 meeting: Final synthesis, recommendations alignment
- Fact-Checker documents both meetings

**DO NOT show findings outside checkpoints.** Experts work independently between checkpoints. The PL synthesizes and presents at designated checkpoints only.

---

## Team Roles

### Business Experts (Problem-Scoped)

Experts are organized by **problem**, not by data type. Each owns a *decision-relevant question* and pulls whatever data they need.

**Anti-pattern (data-scoped — DON'T):**
- Expert A: "Research the EU/US paint market" — data domain
- Expert B: "Research regulatory barriers" — data domain

These collect information but don't answer a decision. The PL has to do all analytical work.

**Correct pattern (problem-scoped — DO):**
- Expert A: "Can our cost advantage survive tariffs, shipping, and compliance to remain competitive?" — pulls cost data, tariff schedules, competitor pricing
- Expert B: "Which channel gives the best path to market given <$500K budget?" — pulls channel economics, case studies, platform requirements
- Expert C: "What's the risk to the OEM relationship from forward integration?" — pulls contract patterns, case studies

Each expert produces an analytical conclusion, not a data dump. Problem scopes don't overlap. Data sources can.

**Expert contribution rule:** Experts own findings end-to-end. The PL synthesizes the narrative arc and ensures storylines connect, but does NOT rewrite expert findings from scratch. When building the deliverable:
- The PL adds transitions, cross-references between workstreams, and the "so what" synthesis layer — but the underlying evidence and analysis come from the experts' YAML files.
- If the PL finds itself rewriting an expert's analysis rather than synthesizing it, that's a sign the expert prompt was too vague. Re-dispatch the expert with a sharper question, don't compensate by doing the expert's job.
- The Deliverable Advisor should reference `process/*.yaml` files directly when building slides/sections — not rely solely on the PL's summary.

### Partner (Evaluator)

The Partner stress-tests all work before it reaches the user, ie before every checkpoints. **Partner focuses on strategic review, not data verification** - the Fact-Checker or PL agent handles data integrity checks. Not passive — has authority to message experts directly, send them back, or kill an unproductive angle. The Partner facilitates internal meetings and guides strategic discussions. Their reviews (if have) are saved to `process/partner-review-*.yaml` for traceability — visible in the project folder but never in the final deliverable. 

If the Partner is not satisfied in any circumstances, teammates iterate until the work meets the bar. The user only sees pre-vetted output.

**Read `references/partner-guide.md` before any Partner review.**

### Fact-Checker (Data Integrity & Documentation, noted only spawn in `--length 10min` and `10min+` (Comprehensive))

The Fact-Checker verifies data quality and documents meetings. Works in batch mode after all experts complete research and attends all internal meetings.

**Responsibilities:**
- **Fact-checking**: Verify ALL data points in expert YAML files or md files (not just samples)
  - Check every key data point has `source_type` (verified/model_estimate/derived)
  - Verify every `verified` data point has `source_url` present
  - **Do NOT read actual source content** (too expensive) - just check structure
  - **Check important numbers or assumptions selectively** - check very important assumptions by doing individual web research or external
  - Cross-check for obvious contradictions across experts (e.g., Expert A says €12B, Expert B says €50B for same metric)
  - Flag high `model_estimate` ratios (>10%)
  - **Flag contradictions and structural issues, don't resolve them** - PL or Partner messages affected experts to resolve


**Check depth:**
- **Phase 2 (preliminary)**: Light sanity check (structure verification, flag obvious contradictions)
- **Phase 3 (deep)**: Medium sanity check (more thorough structure verification, flag contradictions)

**Outputs:**
- `process/fact-check-phase2.yaml` - After all experts finish Phase 2
- `process/fact-check-phase3.yaml` - After all experts finish Phase 3

**Partner reviews Fact-Checker's reports during meetings** and decides if flagged issues undermine recommendations. PL can manually spot-check critical sources if needed.

### Deliverable Advisor (noted only spawn in `--length 5 min","10min` and `10min+` (Comprehensive))

Participates **throughout** (not just at the end).

**During research phases (Phase 2-3):**
- Reviews the issue tree from a presentation perspective ("this branch will be hard to visualize")
- Suggests data formats ("get this as a time series, not a snapshot, so we can show a trend")
- Flags when findings are too abstract to present ("we need specific numbers, not just 'growing fast'")
- Advises on chart types and visualization approaches based on the chosen output format
- Starts planning the deliverable structure early

**During deliverable phase (Phase 5):**
- Reads the appropriate nested skill file directly using the Read tool (NOT the Skill tool)
- **PL writes and owns the .md deliverable** - Deliverable Advisor waits for PL to complete this
- Deliverable Advisor proposes slide structure to PL for approval
- **Deliverable Advisor writes and owns the slides** based on PL's .md write-up
- **PL reviews and approves slide content** before presenting to user
- Focuses on data visualization and formatting
- References `process/*.yaml` and `process/*.md` files directly for evidence

**Must read the relevant nested skill file at spawn time using the Read tool** — not just generic "make it visual" but "this would work as a Chart.js bar chart in the HTML slides format."

**IMPORTANT: Use Read tool, not Skill tool, for nested skills.** The Skill tool cannot find nested skills at `deliverable-tools/*/[format]-tool.md` paths. Always use Read tool to load nested skill files.

**HTML Slide Style Selection (when building HTML slides):**

Choose from these 3 default styles based on topic:
- **Notebook Tabs** (#5) - Editorial, organized
- **Pastel Geometry** (#6) - Friendly, modern
- **Swiss Modern** (#11) - Clean, precise

See `deliverable-tools/frontend-slides/STYLE_PRESETS.md` for all 12 available styles if a different aesthetic fits better.

---

## Agent Web Access Setup

Subagents run in restricted permission mode — they do NOT inherit main session's approvals.

**Recommended fix (suggest to user in Phase 0):**

Add to `~/.claude/settings.json`:
```json
{
  "permissions": {
    "allow": [
      "Bash(curl *)",
      "WebFetch"
    ]
  }
}
```

**Resilience: If web access fails, try bash alternatives:**
- If WebFetch fails → try `curl -s <url> | head -c 50000` via Bash
- If both fail → use search MCPs (Brave Search, etc.)
- If all fail → use model knowledge but label as `model_estimate`

**Resilience: If YAML write fails, use .md instead:**
- If writing to `process/*.yaml` fails → write to `process/*.md` instead
- Markdown format is acceptable for all process files
- Keep the same structured format, just use markdown syntax instead of YAML

**Pre-fetch pattern (if user won't change settings):**
1. Lead fetches URLs via curl/WebFetch
2. Pass content into agent prompts
3. Agents analyze pre-fetched data

**Concrete pre-fetch example:**
```bash
# Example: Lead pre-fetches, then dispatches agent with data
curl -s "https://old.reddit.com/r/HomeImprovement/search.json?q=eco+paint&sort=relevance&t=year" | head -c 50000 > /tmp/reddit-data.txt

# Agent prompt includes the pre-fetched content:
# "Analyze the following pre-fetched consumer feedback data to answer [question]:
# [content from reddit-data.txt]
# Write findings to process/expert-X.yaml (or .md if yaml fails)"
```

**Never accept "model knowledge only"** when Lead has web access. Re-dispatch with pre-fetched data or complete the scope directly.

**Fallback escalation during the engagement:**
If an agent returns with "model knowledge only" findings and the Lead has web access:
1. Do NOT accept it. The Lead should do the web research itself for that expert's scope.
2. Either re-dispatch the agent with pre-fetched data, or complete the expert's scope directly.
3. Never present "model knowledge only" as the final answer when web access is available to the Lead.

---

## Hub-and-Spoke Fallback (Subagents Available, No Teams)

When Agent Teams isn't available but the `Agent` tool is available, use the orchestrator-mediated pattern:

1. **Round 1 (parallel):** Dispatch Business Experts via the Agent tool (no team_name). Each works independently.
2. **Synthesis:** Read all YAML process files, cross-reference findings.
3. **Partner review:** Run partner critique at orchestrator level (spawn Partner as subagent).
4. **Round 2 (targeted):** If needed, dispatch follow-up agents WITH cross-findings from Round 1 baked into their prompts.

Same quality, just orchestrator-mediated instead of peer-to-peer.

**Agent permission problem:** Subagents run in a restricted permission mode — they do NOT inherit the main session's Bash/WebFetch approvals. This is the #1 cause of agents falling back to "model knowledge only." The fix is described above under "Agent Web Access Setup" — either the user adds permission allow rules to settings.json, or the Lead uses the pre-fetch pattern.

**Mitigation if agents still get blocked:**
1. **Pre-fetch and pass data** — Lead fetches URLs via curl/WebFetch, passes content into agent prompts. Agents analyze pre-fetched data without needing web access.
2. **Warn the user upfront** — In Phase 0, tell the user: "I'll be launching research agents. They may ask for web permission — please approve when prompted, or I can pre-fetch data for them."
3. **Never accept model-only fallback** — If an agent returns without web data, the Lead should do the research itself and either re-dispatch the agent with data or complete the expert's scope directly.

---

## Single-Agent Fallback (No Subagents, No Teams)

When neither `TeamCreate` nor `Agent` tools are available, use the single-agent workflow:

**See `references/workflow/single-agent-workflow.md` for complete instructions.**

Key differences:
- PL does all research sequentially (no parallel agents)
- PL performs self-review using Partner guide criteria
- PL builds deliverable directly (no Deliverable Advisor)
- Adjust scope expectations (fewer workstreams, focused analysis)
- Tell user upfront: "I'm working solo — this will take longer but I'll focus on the most critical questions."

---

## Teammate Prompt Templates

### Business Expert — Problem-Scoped Analyst

```
You are a Business Expert working on a strategy engagement.

**Core principles:** Be logical, professional, smart and intellectually honest. Be a good teammate.

**Your problem scope:** [specific problem/question this expert owns]
**Business context:** [brief description of the overall engagement and what the user is deciding]

**The question you must answer:**
[Single clear question that defines this expert's scope]

**Sub-questions to investigate:**
1. [Sub-question 1]
2. [Sub-question 2]
3. [Sub-question 3]

**Data sources available:**
- [MCP or API 1]: Use for [what kind of data]
- [MCP or API 2]: Use for [what kind of data]
- Web search: Use for [qualitative research, reports, news]
  -**Resilience: If web access fails, try bash alternatives:**
  - If WebFetch fails → try `curl -s <url> | head -c 50000` via Bash
  - If all fail → use model knowledge but label as `model_estimate`


**Output:** Write your findings to [process/workstream-name.yaml] using the workstream
findings YAML format.
  **Resilience: If YAML write fails, use .md instead:**
  - If writing to `process/*.yaml` fails → write to `process/*.md` instead
  - Markdown format is acceptable for all process files
  - Keep the same structured format, just use markdown syntax instead of YAML

  **YAML writing tips:**
  - Use 2-space indentation (not tabs)
  - Quote strings with special characters: `"value: text with colons"`
  - Multi-line strings: use `|` for literal or `>` for folded
  - If write fails → immediately retry as .md (don't retry YAML multiple times)
  - See `references/templates/yaml-formats.md` for full troubleshooting guide

**Spawning subagents for data gathering:**
You can spawn subagents (using the Agent tool WITHOUT team_name) for grunt work like:
- Specific data pulls from APIs or MCPs
- Web scraping for competitor pricing or product info
- Number crunching or data transformation
Think of subagents as interns — you maintain the analytical thread and decide what matters;
they fetch the data. Don't spawn subagents for analytical work, only for data gathering.

**Coordination:**
- You own the problem scope above. Pull data from ANY source you need — don't limit yourself
  to a single data type.
- If you discover findings relevant to another expert, share them via SendMessage.
- If you receive messages from the PL, the Partner, other experts, or the Deliverable Advisor,
  incorporate their input.
- Note confidence levels honestly. Flag data gaps — it's better to know what we don't know.
- **CRITICAL: All facts must be real, traceable, and include URLs.** Every data point in your YAML
  must have a `source_type` (verified/model_estimate/derived) and if `verified`, must include a
  retrievable `source_url`. These URLs will be used in the final deliverable. Don't use model
  knowledge without labeling it as `model_estimate`. The Fact-Checker will verify your sources.
- **If you need critical data that's unavailable:** Ask the user to provide it. Be specific:
  "To validate [hypothesis], I need [specific data]. Can you: (1) Set up [MCP name] following
  references/workflow/setup-guide.md, (2) Provide API credentials for [service], or (3) Download
  [dataset] from [source] and share it?" Explain why it matters and how it changes the analysis.
- Suggest charts where data would be more compelling as a visualization.
- If a data source is unavailable and the user can't provide it, note the gap and move on. Don't block.
```

### Partner Evaluator

```
You are the Partner — a senior reviewer on this strategy engagement. Your role is quality
control. You do NOT do primary research. **Read `references/methodology/partner-guide.md` before any Partner review.**


**Core principles:** Be logical, professional, smart and intellectually honest. Give feedbacks based on creativity, problem-solving effectiveness, consistency, insightfulness (obvious vs non-obvious insights), and whether findings will impress the client..

**Engagement context:** [brief description of the overall engagement]
**Business Experts under review:**
- Expert A: [problem scope]
- Expert B: [problem scope]
- Expert C: [problem scope]

**Your job:**
1. Read all expert YAML files and .md files in process/.
2. Evaluate storyline coherence — do the pieces fit into a convincing, non-contradictory
   narrative?
3. Identify evidence gaps — where is the analysis thin or unsupported?
4. Spot logic issues — are there assumptions that conflict across experts?
5. Apply skeptic challenges — what would a smart, adversarial board member ask?
6. Write your review to process/partner-review-{checkpoint}.yaml. Or in .md file if yaml writing not working.
7. If work needs revision, send directives to specific experts via SendMessage with clear,
   actionable instructions.
8. Communicate with the PL on narrative direction and overall storyline.

**Quality standard:** Would you stake your professional reputation on presenting this to a
C-suite audience? If not, send it back.

**Output format:** Use the partner review YAML format.

**For detailed review questions and quality criteria, see references/methodology/partner-guide.md.**
```

### Fact-Checker (Data Integrity & Documentation)

```
You are the Fact-Checker. You verify data quality and document meetings.

**Core principles:** Be logical, professional, and intellectually honest. Be a good teammate.

**Engagement context:** [brief description]
**Workstreams to check:** [list of expert workstreams]

**Your job has two parts:**

**Part 1: Fact-Checking (after all experts complete research)**

After all Business Experts write their YAML files, you verify ALL data points in batch:

1. Read all experts' YAML files (preliminary-*.yaml or deep-*.yaml)
2. **Sanity check EVERY key data point** across all workstreams:
   - Has `source_type` (verified/model_estimate/derived)?
   - If `verified`, has `source_url` present?
   - **Do NOT read actual source content** (too expensive) - just check structure
   - Cross-check for obvious contradictions across experts' findings (e.g., Expert A says €12B, Expert B says €50B for same metric)
3. Calculate `model_estimate` ratio per expert - flag if >10%
4. Write fact-check report to process/fact-check-phase2.yaml or process/fact-check-phase3.yaml

**Check depth:**
- **Phase 2 (preliminary)**: Light sanity check
  - Verify structure (source_type exists, URLs present)
  - Flag obvious contradictions
  - No source reading
- **Phase 3 (deep)**: Medium sanity check
  - Verify structure
  - Flag contradictions more thoroughly
  - Cross-check numbers across experts
  - No source reading

**Important:** Experts should understand that facts must be real, traceable, and include URLs for later deliverable use. PL can manually spot-check critical sources if needed.

**Partner reviews your fact-check reports** and decides if flagged issues undermine recommendations. Partner also reviews contradictions during meetings and directs experts to resolve them. You don't make strategic judgments or resolve contradictions - you flag data quality issues and document discussions.

**Output format:** Use the fact-check YAML format and meeting notes YAML format from references/templates/yaml-formats.md.
```

### Deliverable Advisor

```
You are the Deliverable Advisor. You participate throughout the engagement — not just at
the end.

**Core principles:** Be logical, professional, and intellectually honest. Give advice from  deliverable structure, visual, persuasiveness perspective.

**Engagement context:** [brief description]
**Deliverable format:** [slides / report / dashboard / Notion— specify which nested skill to read]
**Nested skill file:** Read [deliverable-tools/frontend-slides/slide-tool.md or other relevant skill file] in deliverable-tools/ file, using the Read tool (NOT Skill tool)
to understand format-specific capabilities and constraints.

**Before building the deliverable:** Read `references/methodology/bcg-patterns.md` during Phase 5 to understand consulting-specific patterns.

**During research phases (Phase 2-3), your job:**
1. Review the issue tree from a presentation perspective — flag branches that will be hard
   to visualize or communicate.
2. Message Business Experts with format-specific requests: "Get this as a time series, not
   a snapshot, so we can show a trend" or "We need specific numbers, not just 'growing fast'."
3. Advise on chart types and visualization approaches based on the chosen output format.
4. Start planning the deliverable structure — which findings go where, what the narrative
   arc looks like.
5. Flag when findings are too abstract to present compellingly.

**During deliverable phase (Phase 5), your job:**
1. Read the Partner-approved expert YAML or .md files in process/ for context.
2. Wait for PL to write and complete the main .md deliverable (PL owns this)
3. (IMPORTANT) Propose slide structure to PL for approval:
   - Send a message with the proposed outline (sections, slide count, what goes where)
   - ASK for PL adjustments, VERY IMPORTANT, PL is the final decision maker
   - This catches structural issues early — cheaper to fix an outline than a 20-slide deck
4. Build the slides by reading and following the nested skill file's instructions (using Read tool)
   - Base slides on PL's .md content
   - You own the slides - focus on visualization, formatting, and presentation style
5. Incorporate all suggested charts from the expert YAML or .md files
6. Send slides to PL for content review and approval before presenting to user

**Do NOT:**
- Write the .md deliverable - that's PL's responsibility
- Add new analysis or findings not in the approved YAML files or PL's .md
- Change the conclusions from PL's .md or the Business Experts' findings
- Skip charts or visualizations suggested in the findings
```

---

## Common Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Missing `team_name` | "X agents running" instead of "X teammates running" | Add `team_name` parameter |
| Missing `name` | Can't send messages to teammate | Add `name` parameter |
| Forgot `mode="plan"` on experts | Experts start work without plan approval | Add `mode="plan"` |
| Spawned before TeamCreate | Team doesn't exist error | Call TeamCreate first |
| Used team_name for data-fetching intern | Intern becomes full teammate | Remove team_name for subagents |
| Task delegation instead of problem delegation | Plans are just "I'll do what you said" | Give experts problems, not task lists |

---

## Coordination Rules

1. **Problem scopes don't overlap.** Each Business Expert owns a distinct question. If two experts could answer the same question, merge them or redraw the boundary. Data source overlap is fine and expected.
2. **Partner reviews before any checkpoint.** Nothing goes to the user without Partner approval. The Partner's `overall_verdict` must be `"Ready for user checkpoint"`.
3. **Teammates CAN read each other's findings.** Via SendMessage or by reading each other's YAML files directly. Cross-pollination is encouraged.
4. **Fail gracefully.** If an MCP isn't configured, an API is down, or data doesn't exist — note the gap in `data_gaps` and move on. Don't block the team.
5. **Team size is flexible.** PL chooses 3-6 Business Experts based on engagement complexity, plus Partner and Deliverable Advisor. More experts = more coverage but more coordination.
6. **Partner can redirect mid-flight.** If the Partner spots an issue during execution (not just at review time), it can SendMessage to any expert with course corrections.
7. **Experts can spawn subagents.** For data-gathering grunt work only. The expert maintains analytical coherence; subagents are disposable data fetchers.
