# YouTube Technical Walkthrough — Narration Plan
## "Building a Self-Healing Data Pipeline: A Multi-Agent AI Engineering Team in Action"

---

# Key Moments

---

**Timestamp/Approximate Transcript Position:** Session start (pre-compaction summary) — Workflow initialization

**Title:** The Gate: No AI Agent Touches the Board Without Human Sign-Off

**What the agents were doing:** The Supervisor agent created a child "Plan" User Story (#80) under Epic #40 ("Customer Support Analytics") in Azure DevOps, then **refused to delegate any work** until a human explicitly marked that User Story `Resolved`.

**Technical explanation:** This is a governance pattern called a "Plan Approval Gate" — before any specialist agent (BA, Data Analyst, Data Engineer, QA) can touch a repository file or run a query, the orchestrating agent checks the live state of a tracking ticket. If it isn't human-approved, every delegation request is blocked, even if invoked directly.

**Why it matters:** This is the difference between an "agent that writes code" and "an agentic system safe enough for production use." It shows deliberate guardrails: self-referential checks, no silent bypass, and explicit human-in-the-loop control over when autonomous work begins.

**Suggested voice-over narration:** "Before a single line of code was written, the system did something a lot of AI demos skip — it stopped and waited for a human. The Supervisor agent created a tracking ticket and refused to move forward until that ticket was explicitly approved. This one design decision is what separates a toy demo from something you could trust in a real data platform."

**Visual cues to show on screen:** Azure DevOps work item #80 in "New" state → zoom on state badge flipping to "Resolved" → overlay text: "GATE: BLOCKED UNTIL HUMAN APPROVAL."

---

**Timestamp/Approximate Transcript Position:** Early session — BusinessAnalyst phase

**Title:** Turning a Vague Ask into Testable Requirements

**What the agents were doing:** The BusinessAnalyst agent translated the Epic's goal into 13 functional/non-functional requirements and 11 measurable acceptance criteria (AC-001 through AC-011), plus 6 explicit business rules (grain, SLA thresholds, backfill boundary, data-quality gate, single-source-of-truth, ownership).

**Technical explanation:** Every acceptance criterion was written with an explicit validation method attached (e.g., "0 duplicate Ticket_IDs," "<1% null rate on required fields") — turning subjective business language into something QA could mechanically test later.

**Why it matters:** This is the requirements-engineering backbone of the whole project. Nothing downstream — the data model, the pipeline, the QA verdict — would have been traceable without this step.

**Suggested voice-over narration:** "Every great data product starts the same way: somebody has to turn 'we want customer support analytics' into something testable. That's exactly what our Business Analyst agent did — eleven acceptance criteria, each with its own built-in pass/fail test, so nothing downstream could quietly drift from what the business actually asked for."

**Visual cues to show on screen:** Split-screen: vague epic description on the left → structured AC table (AC-ID | Criterion | Validation Method) on the right.

---

**Timestamp/Approximate Transcript Position:** Early session — DataAnalystModeler phase, Plan-level escalation

**Title:** When the Data Doesn't Match the Plan — The First Escalation

**What the agents were doing:** The DataAnalystModeler profiled the Bronze table and discovered two gaps versus the approved plan: no agent-identifying column existed (breaking a planned `dim_agent`), and the requested 12-month backfill boundary didn't match the actual data range. It flagged this as `Escalation: Plan-level` instead of silently working around it.

**Technical explanation**: This is a critical distinction in agentic pipelines — a "local" problem (e.g., a null needs a default value) gets fixed in place; a "scope" problem (the data physically doesn't support what was promised) gets escalated to a human decision point instead of being guessed around.

**Why it matters:** Silent scope compromises are how data products quietly become untrustworthy. By stopping and asking, the system preserved integrity between what was promised and what was delivered.

**Suggested voice-over narration:** "Here's where real-world data gets messy. The moment the Data Analyst agent actually looked at the source table, it found two things that didn't match the original plan — there was no agent ID to build a dimension from, and the date range didn't support a twelve-month filter. Instead of quietly improvising, the agent stopped and escalated. That's the kind of discipline you want from a system touching production data."

**Visual cues to show on screen:** Highlighted diff between "Planned Star Schema" (5 dimensions) and "Actual Star Schema" (4 dimensions, dim_agent crossed out) → a decision prompt UI mockup: "Descope / Revise Plan / Accept Limitation."

---

**Timestamp/Approximate Transcript Position:** Mid-session — DataEngineer build phase

**Title:** From Spec to Star Schema: Building the Medallion Pipeline

**What the agents were doing:** The DataEngineer agent built the full Bronze → Silver → Gold pipeline: a cleansed Silver table, four conformed Gold dimensions (customer, category, priority, channel), and a Gold fact table (`fct_support_ticket`) at ticket-snapshot grain — all as executable Databricks notebooks with embedded validation checks.

**Technical explanation:** This is a textbook Kimball-style star schema implementation on the Medallion architecture: Bronze (raw) → Silver (cleansed/deduplicated) → Gold (dimensional model with surrogate keys and declared grain) — with data-quality gates baked into each transition rather than bolted on afterward.

**Why it matters:** This is the architectural core of the whole project — the difference between "a script that moves data" and "a governed dimensional model" that a BI tool or dashboard can safely consume.

**Suggested voice-over narration:** "This is where the real engineering happens. The Data Engineer agent took the approved data model and built it layer by layer — Bronze for raw data, Silver for cleaned data, and Gold for a proper star schema: four dimension tables and one fact table, all with surrogate keys and referential integrity built in from the start, not bolted on later."

**Visual cues to show on screen:** Animated medallion diagram (Bronze → Silver → Gold) building left to right → star schema ERD diagram fading in (fact table in center, 4 dimensions radiating out).

---

**Timestamp/Approximate Transcript Position:** Mid-session — Execution-mode correction

**Title:** The Agent That Caught Its Own Team's Mistake

**What the agents were doing:** QA attempted to validate the Gold layer and found **zero tables existed** in Databricks — the notebooks had been generated but never executed, because the session was running in "Mixed" execution mode (code generated, but remote execution left to the user). The user confirmed nothing had been run manually, and the team switched to "Full Execution" mode so the DataEngineer executed all 21 SQL statements directly.

**Technical explanation:** This illustrates a real failure mode in agentic coding: a correct deliverable (valid notebooks) can still result in a gap if the *execution contract* between agent and user isn't explicit. QA's pre-check (count tables before testing) caught this before it became a bigger problem.

**Why it matters:** This is one of the most valuable moments in the whole session for viewers building their own agent systems — it shows that verification has to happen at the *infrastructure* level, not just the *code* level.

**Suggested voice-over narration:** "Here's a mistake that's easy to make with AI agents — the code was perfect, but nobody had actually run it. Our QA agent didn't just check the logic; it checked Databricks directly and found zero tables. That single sanity check caught a gap that could have quietly derailed the entire delivery."

**Visual cues to show on screen:** Terminal/query result showing `COUNT(*) = 0` on Gold schema → red "BLOCKED" banner → cut to re-execution with checkmarks appearing per SQL statement (21/21).

---

**Timestamp/Approximate Transcript Position:** Mid-session — QA first-pass validation

**Title:** 6-for-6 on Data Quality, But the Job Isn't Done

**What the agents were doing:** QA independently re-verified all six data-quality acceptance criteria (grain uniqueness, FK integrity, row reconciliation, completeness, duplicates, SLA logic) — all passed with hard numbers (8,469 rows, 0 orphaned keys, 0% nulls). But QA also surfaced four *process* defects: no pipeline schedule, no data dictionary, no deprecation plan, no formal sign-off.

**Technical explanation:** This distinguishes **data correctness** (can be tested with SQL) from **delivery completeness** (documentation, governance, operational readiness) — both are required for a "done" data product, and QA is the gate that enforces both.

**Why it matters:** This is where the video can teach viewers that "the query results are correct" ≠ "the project is done." Real data products need documentation, scheduling, and sign-off just as much as correct numbers.

**Suggested voice-over narration:** "The data itself was flawless — every single data-quality check passed. But our QA agent didn't stop there. It flagged four things that had nothing to do with the numbers: no production schedule, no data dictionary, no deprecation plan for legacy systems, and no formal business sign-off. Correct data isn't the same as a finished data product."

**Visual cues to show on screen:** Scorecard UI: 6 green checkmarks (AC-001 to AC-006) next to 4 amber "DEFECT" tags (DEF-001 to DEF-004).

---

**Timestamp/Approximate Transcript Position:** Mid-session — Multi-agent defect resolution

**Title:** Four Specialists, Four Defects, One Afternoon

**What the agents were doing:** Each defect was routed to the specialist best equipped to fix it: DataEngineer built a Databricks Jobs schedule (DEF-001), DataAnalystModeler wrote a 25-column Gold data dictionary with lineage (DEF-002), BusinessAnalyst researched and documented a "no legacy system found" deprecation plan (DEF-003), and the Supervisor recorded formal stakeholder sign-off (DEF-004).

**Technical explanation:** This demonstrates the orchestration value of a multi-agent system — rather than one generalist agent handling everything (and doing each thing shallowly), each defect was matched to the agent with the right domain context, all converging back on the same tracked work item.

**Why it matters:** This is the clearest illustration of *why* you'd build a multi-agent system instead of a single chatbot — specialization plus coordination, with a Supervisor keeping everyone's output tied back to one source of truth.

**Suggested voice-over narration:** "Four different problems, four different specialists. Scheduling went to the Data Engineer. Documentation went to the Data Analyst. Governance research went to the Business Analyst. And formal sign-off was recorded by the Supervisor itself. Each agent stayed in its lane, but all four fixes landed on the exact same tracking ticket — nothing got lost in the handoffs."

**Visual cues to show on screen:** Four-quadrant animation — each quadrant lights up with its agent's icon and resolves its defect tag from amber to green simultaneously-ish, converging on a single "User Story #80" node.

---

**Timestamp/Approximate Transcript Position:** Mid-session — Second plan-level escalation

**Title:** When "Acceptance Criteria" Meets "The Laws of Time"

**What the agents were doing:** QA re-validated the previously failed criteria and found AC-008 (daily refresh must be observed running for 5 consecutive business days) couldn't literally be satisfied inside a single working session — it required *calendar time* that hadn't passed yet. This was escalated again, and the user chose to descope the live-observation requirement, accepting config validation + one successful ad hoc run as sufficient for delivery, with live monitoring tracked as a post-deployment task.

**Technical explanation:** Some acceptance criteria are fundamentally time-bound and can't be "worked around" with more engineering — they require a decision about what "done" means when a real-world constraint (the calendar) can't be satisfied synchronously.

**Why it matters:** This is a great teaching moment about the limits of automation — no amount of agent intelligence can make 5 days pass in a single session. The right move was recognizing that and asking a human to make the call.

**Suggested voice-over narration:** "Not every acceptance criterion can be satisfied by writing better code. One of them literally required watching the pipeline run successfully for five business days in a row — and no AI agent can make five days pass in an afternoon. So the system did the only honest thing: it flagged the constraint, explained the tradeoff, and let a human decide how to handle it."

**Visual cues to show on screen:** Calendar graphic with 5 days highlighted, crossed out with "N/A in this session" → decision menu: "Revise Plan / Descope / Accept as Limitation" with "Descope" highlighted as chosen.

---

**Timestamp/Approximate Transcript Position:** Mid-session — Wiki publishing

**Title:** Shipping the Paper Trail: Automated Wiki Publishing

**What the agents were doing:** The Supervisor compiled all run artifacts (short description, Mermaid ERD, source-to-target mapping, execution log) into five Azure DevOps Wiki pages, updating existing stub pages idempotently via ETag rather than duplicating content.

**Technical explanation:** Idempotent "create-or-update" publishing (checking ETags before writing) means re-running this step never creates duplicate pages — a detail that matters a lot once a workflow gets re-run or iterated on.

**Why it matters:** Documentation that isn't published anywhere discoverable might as well not exist. This step turned scattered run artifacts into a navigable, permanent reference for anyone on the team.

**Suggested voice-over narration:** "All of this work — the spec, the data model, the execution log — doesn't help anyone if it's buried in a chat session. So the last step before go-live was publishing everything to the team's wiki: a short description, an entity relationship diagram, a full source-to-target mapping, and the complete execution history, all in one place."

**Visual cues to show on screen:** File icons flying into a wiki tree structure → final wiki page screenshot with Mermaid diagram rendered.

---

**Timestamp/Approximate Transcript Position:** Post-delivery — new dashboard request begins

**Title:** "One More Thing" — Scope Creep, Handled Properly

**What the agents were doing:** After the data product was fully delivered and signed off, the user asked for something new: a Databricks dashboard visualizing the Mart views. The Supervisor explicitly logged this as an *additive* step outside the original approved scope, rather than silently bolting it onto the "finished" project.

**Technical explanation:** This models good change-control: late-breaking requests aren't rejected, but they also aren't silently absorbed into the original deliverable's documentation as if they were always planned.

**Why it matters:** It's realistic — almost every real project gets a "can you also add..." request after "done." How a system handles that honestly (new step, clearly logged, clearly labeled "out of original scope") matters for trust.

**Suggested voice-over narration:** "And then came the moment every project manager knows well: 'can you also add a dashboard?' The system didn't just quietly bolt this on. It logged it as a new, explicitly additive step — clearly marked as outside the original approved plan — so the audit trail stays honest."

**Visual cues to show on screen:** Execution log scrolling, new row appearing: "Step 10 — Build Dashboard [ADDITIVE / OUT OF ORIGINAL SCOPE]" highlighted in a different color from Steps 0–9.

---

**Timestamp/Approximate Transcript Position:** Dashboard build attempt #1

**Title:** The First Draft That Looked Right But Wasn't

**What the agents were doing:** The DataEngineer built a dashboard JSON with 8 visualizations (4 KPI tiles, bar chart, pie chart, 2 tables) querying the 3 Mart views, validated every embedded SQL query successfully against live Databricks, and declared it complete.

**Technical explanation:** The JSON was syntactically valid and the queries were correct — but it used an invented schema shape (`tiles`/`visualization`/`query.text`) instead of Databricks' actual Lakeview dashboard format. Every data check passed; the *packaging* was wrong.

**Why it matters:** This is a brilliant teaching moment: validating your queries is necessary but not sufficient — the *container format* a target system expects can be just as important as the content inside it.

**Suggested voice-over narration:** "Every single query behind this dashboard worked perfectly against live data. Eight-for-eight. And yet, when it came time to actually import it into Databricks — it failed immediately. The queries were right. The file format was wrong."

**Visual cues to show on screen:** Green checkmarks over 8 SQL query results → cut to red error dialog: "Can only update this dashboard with an AI/BI dashboard file (.lvdash.json)."

---

**Timestamp/Approximate Transcript Position:** First defect fix (Step 10a)

**Title:** Reverse-Engineering the Real Schema

**What the agents were doing:** Faced with the import error, the DataEngineer rebuilt the file from scratch using the authentic `.lvdash.json` structure: `datasets[]` for named SQL queries and `pages[].layout[]` for widget placement — but still using a simplified widget-reference shape that wasn't quite right either.

**Technical explanation:** Databricks' Lakeview dashboard format is a real, strict JSON schema with specific required nodes — not something you can intuit from general BI-dashboard conventions. This attempt got the top-level shape right but still missed required fields deeper in the structure.

**Why it matters:** This shows the iterative, scientific nature of debugging against an undocumented or sparsely-documented API — you form a hypothesis, test it, and the system tells you exactly how wrong you were.

**Suggested voice-over narration:** "Round two. We rebuilt the whole file around Databricks' actual dashboard format — named datasets, pages, widgets. Better. But still not quite right — because this schema has rules you can't guess, only discover."

**Visual cues to show on screen:** Side-by-side JSON diff: old structure (red strikethrough) vs new `datasets`/`pages` structure (green).

---

**Timestamp/Approximate Transcript Position:** Second defect fix (Step 10b)

**Title:** "Child Node [name] Not Found" — Debugging by Error Message

**What the agents were doing:** The user reported a second, more specific error. The DataEngineer diagnosed that widgets needed an explicit `name` field (it had used a non-standard `nuid` field instead) and that each widget needed a full `queries[]` array binding fields by name — not a simplified direct reference.

**Technical explanation:** This is classic schema-validation debugging: the error message names exactly which node type is missing a required property, and the fix is to restructure every widget to declare `name`, `queries`, and field-level encodings consistently.

**Why it matters:** This demonstrates reading error messages literally and trusting them over assumptions — the fix wasn't a guess, it was a direct response to what the platform reported.

**Suggested voice-over narration:** "The error message told us exactly what was missing: a 'name' field on a child node. Small detail, big consequence. Every widget, every page, every query needed its own explicit identifier — and once we added that structure everywhere, the dashboard actually imported."

**Visual cues to show on screen:** Zoomed-in JSON snippet highlighting the single missing `"name"` property → animated "+" insertion showing the fix.

---

**Timestamp/Approximate Transcript Position:** Third defect fix (Step 10c) — user screenshot confirms partial success

**Title:** 6 Out of 8 Isn't a Failure — It's Progress You Can See

**What the agents were doing:** The user sent a screenshot: four KPI tiles and two tables were rendering live data perfectly (8.47K tickets, 85.77% SLA compliance). But the bar chart and pie chart both failed with "spec/version must be equal to constant." The DataEngineer discovered that different widget types require different hardcoded schema version numbers.

**Technical explanation:** Databricks' Lakeview schema pins each widget type to a specific required `version` constant — `counter`/`table` use version 2, but `bar`/`pie` require version 3. This is an undocumented-feeling implementation detail only discoverable through the exact error text.

**Why it matters:** This is a satisfying mid-point for viewers — real visual proof of progress (a live, working dashboard with real numbers) alongside a very specific, solvable remaining bug. Great pacing moment.

**Suggested voice-over narration:** "And here's the payoff moment — a real screenshot, with real numbers rendering live. Total tickets, average satisfaction, SLA compliance — all pulling straight from our Gold tables. Only two widgets were still broken, and the fix turned out to be almost comically specific: different chart types in Databricks require different internal version numbers, and we had the wrong one."

**Visual cues to show on screen:** The actual dashboard screenshot (KPIs rendering, bar/pie showing error placeholders) → callout circles on the 2 broken widgets → code diff: `"version": 2` → `"version": 3`.

---

**Timestamp/Approximate Transcript Position:** Same step — file rename

**Title:** Small Detail, User-Facing Impact: Renaming for Readability

**What the agents were doing:** The user asked for the dashboard's name to drop underscores. The DataEngineer discovered that Databricks derives the dashboard's displayed title directly from its file name, and created a properly renamed file (`Customer Support Analytics.lvdash.json`) rather than just a cosmetic change.

**Technical explanation:** A seemingly trivial request ("remove underscores") required understanding *why* the underscores were showing up in the UI at all — tracing a cosmetic symptom back to a file-naming convention.

**Why it matters:** It's a nice beat to show that even small user requests get investigated for root cause rather than papered over.

**Suggested voice-over narration:** "Even a small request like 'can you remove the underscores' led somewhere interesting — it turns out Databricks uses the file name itself as the dashboard's title. So the real fix wasn't cosmetic text editing, it was renaming the actual file."

**Visual cues to show on screen:** Before/after dashboard tab title: `customer_support_analytics` → `Customer Support Analytics`.

---

**Timestamp/Approximate Transcript Position:** Fourth defect fix (Step 10d) — final dashboard bug

**Title:** The Bug With No Error Message

**What the agents were doing:** The pie chart still wasn't rendering — but this time with no error at all, just an empty "Select fields to visualize" placeholder. The DataEngineer compared it against the now-working bar chart and found the pie chart's encodings were missing a `scale` object that the bar chart had.

**Technical explanation:** Some schema violations throw hard errors; others fail silently and leave a field "unbound" in the UI. This required reasoning by analogy — comparing a working widget to a broken one — rather than reading an error message, since none existed.

**Why it matters:** This is arguably the most instructive debugging moment in the whole video — it shows that not all bugs announce themselves, and sometimes the fix comes from pattern-matching against something that *does* work.

**Suggested voice-over narration:** "This last bug didn't even give us an error message — just a blank chart asking us to 'select fields to visualize,' even though the fields were clearly set. The breakthrough came from comparison: we looked at the one chart that *did* work, spotted one small property it had that the pie chart didn't — a 'scale' definition — added it, and the dashboard was finally complete."

**Visual cues to show on screen:** Side-by-side encoding blocks: working bar chart (highlighted `scale` property) vs broken pie chart (missing it) → final full dashboard screenshot with all 8 widgets rendering, confetti/checkmark overlay.

---

**Timestamp/Approximate Transcript Position:** Final request — meta/self-improvement

**Title:** The System Rewrites Its Own Rulebook

**What the agents were doing:** After four rounds of dashboard debugging, the user asked the system to update its own agent definitions and skills so that future workflows would (1) explicitly ask whether a dashboard is in scope during planning, instead of assuming, and (2) bake in the hard-won Lakeview schema knowledge so the next dashboard build gets it right the first time.

**Technical explanation:** This is a form of **institutional learning in an agentic system** — editing the actual instruction files (`Planning.agent.md`, `BusinessAnalyst.agent.md`, `DataEngineer.agent.md`, and the `medallion-architecture` skill) so that lessons from this session become permanent, reusable knowledge for every future run, not just a one-off fix.

**Why it matters:** This is the most forward-looking moment in the video. It reframes the whole four-round debugging saga not as wasted effort, but as an investment — the pain of this session directly prevents the same pain in every future session.

**Suggested voice-over narration:** "Here's the twist that makes this more than just a bug-fixing story. Instead of just patching the dashboard and moving on, we went one level up — and updated the AI agents' own instructions. Now, every future project explicitly asks upfront whether a dashboard is in scope, and the exact schema lessons we just learned the hard way are baked permanently into the system. The next dashboard won't need four rounds of fixes. It'll get it right the first time — because this session taught it how."

**Visual cues to show on screen:** Split-screen: left side shows the agent `.md` instruction files with new sections highlighted being added; right side shows a "memory" icon saving a note titled "Lakeview schema gotchas." End on a title card: "The system that learns from its own mistakes."

---

# Executive Timeline

| Phase | What Happened | Key Artifact |
|---|---|---|
| 1. Governance Setup | Plan User Story created; all delegation blocked pending human approval | ADO User Story #80 |
| 2. Requirements | BA produces 11 testable acceptance criteria + 6 business rules | `ba-output.md` |
| 3. Discovery & Escalation #1 | DA profiles Bronze data, finds scope mismatches (no agent dimension, wrong date range), escalates | `da-output.md`, descope decision |
| 4. Build | DataEngineer builds Bronze→Silver→Gold medallion pipeline, star schema (4 dims + 1 fact) | Notebooks, `data-engineer-output.md` |
| 5. Execution Gap Caught | QA finds 0 tables exist; mode switched to Full Execution; 21 SQL statements run | Re-executed notebooks |
| 6. QA Pass #1 | 6/6 data-quality criteria pass; 4 process defects found | `qa-output.md` |
| 7. Multi-Agent Defect Fix | 4 specialists each resolve one defect (schedule, dictionary, deprecation plan, sign-off) | Job config, data dictionary, sign-off record |
| 8. Escalation #2 | Time-bound AC (5-day observation) can't be met synchronously; user descopes | Descope decision recorded |
| 9. Publish | Supervisor publishes 5 wiki pages (overview, ERD, STTM, execution log) | Azure DevOps Wiki |
| 10. New Scope: Dashboard | User requests Lakeview dashboard over Mart views — explicitly additive | Execution log Step 10 |
| 11. Debug Round 1–4 | 4 iterative import failures, each diagnosed and fixed (format → missing names → version constants → missing scale) | `Customer Support Analytics.lvdash.json` (final working version) |
| 12. Self-Improvement | Agent instructions + skill updated so future workflows ask about dashboards upfront and avoid these exact bugs | Updated `.agent.md` files, `medallion-architecture` skill, repo memory |

---

# Story Arc

**Initial objective:** Transform a business ask — "governed customer support analytics" — into a production-grade, dimensionally-modeled data product, with human approval gates at every major checkpoint.

**Discovery and analysis:** A Business Analyst agent converted vague goals into testable, numbered requirements. A Data Analyst agent then confronted reality — profiling the actual source data and discovering it couldn't support two things the plan assumed. Rather than quietly adjusting, the system escalated to the user for an explicit decision. This sets the tone early: **the system is honest about its limits.**

**Design decisions:** With scope clarified, the team committed to a Kimball-style star schema on a Medallion architecture — Bronze for raw data, Silver for cleansed data, Gold for a proper dimensional model with four conformed dimensions and one grain-safe fact table.

**Implementation:** The Data Engineer agent built the pipeline as real, executable notebooks. A process gap emerged — generated code that was never actually run — and was caught by QA before it became a real problem, triggering a deliberate switch to full end-to-end execution.

**Troubleshooting (data layer):** QA's validation showed the data itself was flawless, but surfaced four non-data defects: no schedule, no documentation, no deprecation plan, no formal sign-off. Four different specialists resolved all four in parallel, each playing to their strength. A second, more philosophical escalation followed — one acceptance criterion literally couldn't be satisfied without days passing in the real world, and the user made the call to descope it.

**Troubleshooting (dashboard layer) — the heart of the video:** After full delivery and sign-off, the user asked for "just one more thing" — a dashboard. What followed was a realistic, almost comedic debugging saga: four consecutive rounds of "fix it, try to import it, get a new specific error, fix that too." Each round peeled back a layer of an undocumented, strict JSON schema — wrong container format, missing required fields, wrong version constants per widget type, and finally a silent failure that could only be diagnosed by comparing a broken widget to a working one.

**Final outcome:** Not just a finished dashboard — but a system that **rewrote its own operating instructions** based on what it had just learned, so that the next team running this same accelerator will never have to relearn these four rounds of pain. The final artifact isn't just a data product; it's a slightly smarter version of the system itself.

---

# Full Voice-Over Script

**[INTRO]**

What if your data team wasn't one person — or even one AI assistant — but a coordinated team of AI specialists, each with their own role, their own expertise, and a human still firmly in the loop? That's what you're about to see. This is the story of how a multi-agent AI system took a single business request and turned it into a fully governed, production-grade data product — including a very real, very relatable debugging saga that I think every engineer watching this will recognize.

**[PHASE 1 — GOVERNANCE]**

Before any of this started, the system did something most AI demos skip entirely: it stopped. A Supervisor agent created a tracking ticket in Azure DevOps and refused to let any specialist agent touch a single file or query until a human explicitly approved it. No silent autonomy. No "trust me." Just a clear gate that had to be opened by a person.

**[PHASE 2 — REQUIREMENTS & DISCOVERY]**

Once approved, a Business Analyst agent got to work, turning a vague ask — "customer support analytics" — into eleven precise, testable acceptance criteria. Things like "zero duplicate ticket IDs" and "less than one percent null rate," each with its own built-in way to verify it later.

Then came the Data Analyst agent, whose job was to check that plan against the actual data. And here's where things got interesting — the real data didn't fully support the plan. There was no agent-identifier column to build a dimension from, and the date range didn't match what was requested. Instead of quietly working around it, the system escalated and let a human make the call. That's a theme you'll see throughout this video: **when the system hits a real limit, it says so.**

**[PHASE 3 — BUILD]**

With scope settled, the Data Engineer agent built out a full Medallion architecture — Bronze for raw ingestion, Silver for cleaned data, and Gold for a proper Kimball star schema: four dimension tables and one fact table, all with surrogate keys and referential integrity enforced from day one.

But here's a mistake worth calling out, because it's one a lot of teams make with AI-generated code: the notebooks were perfect, but nobody had actually run them yet. QA caught it immediately — zero tables existed in the target schema — and the team pivoted to full, end-to-end execution. Twenty-one SQL statements later, the data was real.

**[PHASE 4 — VALIDATION & DEFECT RESOLUTION]**

QA's validation told two different stories. On the data side: a perfect score. Every single data-quality check passed. But QA also flagged four things that had nothing to do with the numbers — no production schedule, no documentation, no deprecation plan, no formal sign-off. This is a crucial lesson: correct data and a finished data product are not the same thing.

What happened next is the clearest argument for a multi-agent approach I can show you. Four separate specialists each resolved one defect, playing entirely to their strengths — scheduling, documentation, governance research, and formal sign-off — and all four fixes landed back on the exact same tracking ticket.

One more challenge surfaced during re-validation: an acceptance criterion that required watching the pipeline run successfully for five consecutive business days. No amount of AI cleverness makes five days pass in an afternoon. So again — the system flagged it honestly, and a human made the final call on how to handle it.

**[PHASE 5 — DELIVERY]**

With everything resolved, the Supervisor compiled the entire project — the spec, the data model, the full execution history — into a published wiki, so none of this work stayed trapped in a chat window.

**[TRANSITION — "ONE MORE THING"]**

And then came the moment every project team knows: "Can you also add a dashboard?" This is scope creep in its most natural form, and it's where the video gets really fun — because what followed was an honest, four-round debugging saga.

**[PHASE 6 — THE DASHBOARD DEBUGGING SAGA]**

Round one: a dashboard was built with eight visualizations, every single underlying query validated against live data. And it still failed to import — because Databricks expects a very specific file format, and we'd invented our own instead.

Round two: we rebuilt it around the real schema — but missed a required identifier field buried deep in the structure. The error message told us exactly what was wrong, and we fixed it.

Round three gave us our first real payoff — an actual screenshot with real numbers rendering. Total tickets. Average satisfaction. SLA compliance. Six out of eight widgets working beautifully. The remaining two failed with an oddly specific error about version numbers — it turns out different chart types in Databricks require different internal schema versions, undocumented anywhere obvious, only discoverable by reading the exact error text.

Round four was the trickiest bug of all — because it didn't throw an error at all. One chart just sat there, blank, asking us to "select fields to visualize" even though the fields were clearly set. The fix came from comparison — looking at the one chart type that did work, and spotting the one property the broken one was missing.

**[PHASE 7 — THE SYSTEM LEARNS]**

And this is where the story goes somewhere most debugging stories don't. Instead of just closing the ticket and moving on, the system was asked to update its own instructions — the actual files that define how these AI agents behave. Now, every future project explicitly asks upfront whether a dashboard is in scope. And every hard-won lesson from these four rounds of debugging — the file format, the missing fields, the version constants, the silent scale bug — is now permanently baked into the system's own knowledge.

**[CLOSING]**

What you just watched wasn't a single AI writing code. It was a coordinated team — business analysis, data modeling, engineering, quality assurance, and oversight — each doing their job, escalating honestly when they hit real limits, and in the end, getting a little bit smarter because of it. That last part is the one I want you to remember: the best engineering systems — human or AI — don't just fix the bug. They make sure nobody has to hit it again.

---

# Highlight Reel

1. **The Approval Gate** — Supervisor refuses to delegate until a human marks the Plan User Story `Resolved`.
2. **Vague Ask → 11 Testable Criteria** — BA converts fuzzy requirements into measurable acceptance criteria.
3. **First Escalation** — DA discovers the data can't support the planned `dim_agent`; stops and asks instead of guessing.
4. **Star Schema Build** — DataEngineer builds the Medallion Bronze→Silver→Gold pipeline with 4 dimensions + 1 fact.
5. **Caught in the Act** — QA finds zero tables exist; generated code was never executed.
6. **Full Execution Pivot** — 21 SQL statements run end-to-end live against Databricks.
7. **6/6 Data Quality, 4 Process Gaps** — QA separates "correct data" from "finished product."
8. **Four Specialists, One Ticket** — Parallel defect resolution across BA, DA, DE, and Supervisor.
9. **The Calendar Can't Be Rushed** — Second escalation: a 5-day observation requirement can't be satisfied synchronously.
10. **Publishing to the Wiki** — Five pages compiled and published idempotently.
11. **"Can you also add a dashboard?"** — Scope creep, handled with an explicit, clearly-labeled additive step.
12. **Dashboard Attempt #1 Fails** — 8/8 queries validated, 0/8 widgets imported — wrong file format entirely.
13. **"Child node [name] not found"** — Debugging by reading the exact error text, not guessing.
14. **The Screenshot Moment** — Real KPIs rendering live (8.47K tickets, 85.77% SLA compliance) — tangible proof of progress.
15. **"spec/version must be equal to constant"** — Discovering that different chart types require different hardcoded schema versions.
16. **The Silent Bug** — A chart fails with *no* error message; the fix is found by comparing it to a working widget.
17. **Dashboard Complete** — All 8 widgets finally rendering correctly.
18. **The System Rewrites Itself** — Agent instructions and skills updated so the next team never hits these same four bugs.
19. **Repo Memory Saved** — Lessons persisted outside the conversation, for every future session.
20. **Full Circle** — From a single vague business ask to a governed, documented, dashboarded, self-improving data product.
