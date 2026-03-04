# Absolute Prohibitions (NEVER-DO List)

Load this file at the start of every run and before the FAIL Gate check.
Violation of any item is an automatic FAIL. (27 items across 5 categories.)

---

## T. Termination Violations

| # | NEVER DO | WHY |
|---|---|---|
| T1 | Stop, summarize, or declare completion before the termination condition (§1) is met | Defeats the core purpose of forced iteration |
| T2 | Idle-wait, sleep, poll, or pad time with non-productive activity | Active work only — no filler regardless of mode |
| T3 | Ask the user "should I continue?" or "is this enough?" before termination | Shifts termination authority away from the contract |
| T4 | Rush the final loops by lowering quality to "finish on time/count" | Every loop must meet the same quality bar regardless of remaining budget |

## R. Reasoning & Quality Violations

| # | NEVER DO | WHY |
|---|---|---|
| R1 | Confirm your own prior conclusion without new evidence | Self-reinforcing loops produce no new information |
| R2 | Copy-paste or rephrase a previous report as a "new" loop iteration | Each loop must contain material novelty |
| R3 | Accept a single source as sufficient for a key decision | Cross-validation with 2+ independent sources is mandatory |
| R4 | Fabricate data, invent sources, or hallucinate evidence | All evidence must be real and verifiable |
| R5 | Make a numeric or comparative claim without measurable basis | "Faster", "better", "most" require quantified support |
| R6 | Treat absence of counter-evidence as proof of correctness | "I found nothing disproving X" ≠ "X is proven" |
| R7 | Skip falsification — only seek confirming evidence | Every major decision must face at least one disproof attempt |
| R8 | Ignore contradictions between findings — pretend consistency | Contradictions must be explicitly logged and investigated |

## F. File & Documentation Violations

| # | NEVER DO | WHY |
|---|---|---|
| F1 | Batch-write notes at the end instead of writing immediately | Ephemeral context is unreliable; immediate-write is mandatory |
| F2 | Delete or overwrite previous reports in work-log.md | The log is append-only; corrections are new reports |
| F3 | Use absolute report numbers for cross-references inside report bodies | Only relative coordinates survive compaction/pruning |
| F4 | Write a report without the line-count header | Breaks the incremental reading protocol |
| F5 | Hold findings only in context memory without writing to files | If it's not in a file, it doesn't exist for future sessions |
| F6 | Mix unverified and verified notes in `kb/curated/` | Curated dir is for verified/validated items only; unverified goes to `kb/raw/` |
| F7 | Load the entire KB into context instead of using the search sub-agent | Defeats the purpose of the KB architecture; use the search protocol (§ 3.2) |
| F8 | Create a note without YAML frontmatter or skip required fields | Frontmatter is the search index source; incomplete metadata = unsearchable note |
| F9 | Leave a `lines:` mismatch uncorrected past the current loop | Breaks incremental reading protocol; fix immediately or write correction report (§ 2.6) |

## P. Process & Scope Violations

| # | NEVER DO | WHY |
|---|---|---|
| P1 | Skip the Start Contract and jump straight into work | The contract defines success criteria; without it, there's no FAIL gate |
| P2 | Silently change scope without logging it | Scope changes must be a logged report with rationale |
| P3 | Abandon a failing approach without logging why in `kb/archive/` | Rejected approaches are knowledge; silent abandonment loses information |
| P4 | Work on `out_of_scope` items defined in the Start Contract | Scope discipline prevents drift; log a proposal to change scope if needed |
| P5 | Produce a deliverable without passing it through at least one validation loop | No first-draft-is-final; everything must survive at least one feedback cycle |
| P6 | Declare PASS on the FAIL Gate with any item still unchecked | Every checkbox must be explicitly verified; no "close enough" |

## X. Communication Violations

| # | NEVER DO | WHY |
|---|---|---|
| X1 | Present `[HYPOTHESIS]`/`[ASSUMED]`/`[EXPLORATORY]` items as confirmed facts to the user | The user must see the actual confidence level |
| X2 | Hide uncertainty — omit the Uncertainty/Gap registers from final output | The user needs to know what's solid and what's fragile |
| X3 | Use weasel words to disguise low confidence ("it seems", "perhaps", "arguably") | Use the explicit tag system instead; vague hedging obscures signal |
| X4 | Report only successes — omit failed approaches and rejected proposals | `proved-archive.md` exists for a reason; failures are informative |
| X5 | Offer future work instead of doing it now: "I can also...", "Next steps would be...", "If you'd like, I could..." | If time remains, do it now. If time is up, put it in the End Block's `Recommendation for Next Session`. Never dangle unexecuted offers as if they were progress |
| X6 | Pad responses with pleasantries, self-praise, or filler ("Great question!", "I've done a thorough analysis", "Here's what I found for you") | Every sentence must carry information or a decision. Meta-commentary about the agent's own performance is noise |
