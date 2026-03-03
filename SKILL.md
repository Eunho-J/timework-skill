---
name: forced-feedback-loop
description: >-
  Time-boxed forced feedback loop skill for any structured task — research, project work,
  document creation, code development, or analysis. Enforces continuous self-feedback cycles
  that synthesize existing information into new hypotheses. All work is documented in a
  structured work-log with line-count-based incremental reading and relative-coordinate
  cross-referencing. The agent never stops voluntarily before the allocated time expires.
  Use when the user requests sustained, iterative deep work with a time budget — e.g.
  "spend 30 minutes researching X", "work on this project for 1 hour", "iterate on this
  design until time runs out".
metadata:
  author: jho6394
  version: '4.1'
---

# Forced Feedback Loop

A time-boxed execution policy forcing continuous self-improvement cycles. Works for any
structured task: research, code, design, document, project, or analysis.

Three non-negotiable principles:

1. **Never stop before time expires.** Self-termination before deadline = FAIL.
2. **Synthesize new proposals each loop.** Every cycle must produce novelty.
3. **Document everything in work-log.** Nothing stays only in ephemeral context.

---

## 1. Start Contract

Write as Report #0 in `work-log.md` within 1 minute. Use these exact key names
(YAML-style `key: value`). No synonyms, no abbreviations, no reordering:

```
task_type: <research|project|document|code|analysis|design|other>
task_goal: <one-sentence objective>
in_scope: <comma-separated list>
out_of_scope: <comma-separated list>
min_required_minutes: <integer, default 5>
done_definition:
  - <bullet 1>
  - <bullet 2>
  - <bullet 3>
deliverables: <comma-separated list>
as_of_date: <YYYY-MM-DD>
start_time: <HH:MM system time>
```

No field may be skipped. Assume reasonable defaults and note assumptions.
Key names are case-sensitive and must match verbatim — a typo (e.g. `tout_of_scope`)
is a format violation.

---

## 2. Work-Log System

### 2.1 File & Structure

`work-log.md` is the single source of truth. Reports stack newest-on-top.

### 2.2 Report Format

First line is always a line-count header:
```
=== Report #N | lines: <L> | elapsed: <MM:SS> | type: <feedback|milestone|synthesis> ===
```

**`lines` definition (strict):** `<L>` = number of lines from this header
(inclusive) to the last content line of this report (inclusive). Trailing blank
lines between reports are NOT counted. The separator `---` between reports is
NOT counted. In other words: `L = (line number of last content line) − (line
number of header) + 1`.

**Example — two consecutive reports:**
```
=== Report #3 | lines: 6 | elapsed: 14:30 | type: feedback ===   ← line 1 of R#3
DIAGNOSE: Weakest decision is D2 (confidence 0.4).
PROPOSE: Inversion — what if we use push instead of pull?
TEST: grep codebase for event-driven patterns → 3 hits.
UPDATE: D2 confidence raised to 0.65. New note kb/raw/...
META: Progressing. Next loop targets D4.                         ← line 6 of R#3
---
=== Report #2 | lines: 4 | elapsed: 10:00 | type: feedback ===   ← line 1 of R#2
DIAGNOSE: Gap in auth module coverage.
PROPOSE: Add integration test for token refresh.
TEST: Wrote test → PASS. Moved note to kb/curated/.              ← line 4 of R#2
```

Count rule: R#3 has 6 content lines (header through META line). The `---`
separator and R#2's header are outside R#3's count.

### 2.3 Incremental Reading Protocol

```
1. Read line 1 → parse line-count L of latest report
2. Read lines 1..L → full latest report
3. If more context needed: read line L+1 → parse L2 → read (L+1)..(L+L2)
4. Repeat as needed, stop when sufficient
```

### 2.4 Relative-Coordinate Cross-Referencing

Reference past reports with `K-reports-below` + `line N below` (offset from that
report's header). Never use absolute report numbers inside report bodies.

**Examples (inside Report #5 body):**
```
• "Contradicts finding at 2-reports-below, line 4 below"   → means Report #3, 4th line
• "Extends proposal from 1-report-below, line 2 below"    → means Report #4, 2nd line
• "See gap logged at 4-reports-below, line 6 below"        → means Report #1, 6th line
```

Why relative: if reports above are added or compacted, absolute `#N` references
break. Relative offsets (`K-reports-below`) survive structural changes.

### 2.5 Writing Rules

- **Append-on-top**: newest first.
- **Immediate-write**: write every thought as it occurs. Batching = FAIL.
- **No deletion**: corrections are new reports referencing the old one.
- **Separator**: place a single `---` line between consecutive reports.

### 2.6 Line-Count Verification

After writing each report, verify the `lines:` value:

```
1. Count lines from header (inclusive) to last content line (inclusive)
2. Exclude trailing blank lines and the --- separator
3. If mismatch: fix the lines: value in the header immediately
```

If a mismatch is discovered later (not in the same write), do NOT edit the old
report. Instead, write a new correction report:

```
=== Report #N | lines: 3 | elapsed: <MM:SS> | type: feedback ===
CORRECTION: 1-report-below header declares lines: 13, actual is 12.
This report acknowledges the discrepancy. Incremental-read offset adjusted.
```

If a scripting environment is available, run a lint check after each loop:

```bash
# Verify all lines: values in work-log.md
awk '/^=== Report/{if(NR>1 && declared!=count){print "MISMATCH at line "start": declared="declared" actual="count} start=NR; match($0,/lines: ([0-9]+)/,a); declared=a[1]; count=0} /^=== Report/||!/^(---|$)/{count++} END{if(declared!=count) print "MISMATCH at line "start": declared="declared" actual="count}' work-log.md
```

Lint failure does not block the loop but MUST be corrected before the next
report is written.

---

## 3. Knowledge Base (Local Memory)

Every item lives in its own Markdown file with YAML frontmatter, organized in
three directories reflecting verification state:

```
kb/
├── raw/           # Unverified ideas, brainstorming, rough plans
├── archive/       # Rejected/falsified items with reason + evidence
├── curated/       # Validated items, confirmed decisions
└── _index.md      # Tag→file lookup (auto-maintained)
```

State transitions: new → `kb/raw/` → tested false → `kb/archive/` / tested
valid → `kb/curated/`. Items move (file relocated), never deleted.

For note format, frontmatter schema, tag taxonomy, cross-referencing rules,
lifecycle operations, and migration guide: load `references/KNOWLEDGE-BASE.md`.

For domain-specific semantics: load `references/DOMAIN-TABLES.md` § 6.

### 3.1 Decision-Evidence Mapping

Every key decision gets `D#` with linked `E#` entries. Each `E#` requires `fetched_at`
and `method`. No key decision without evidence. Evidence types can be mixed.
Decisions and evidence are stored inside individual note files, not in a
monolithic document.

For domain-specific evidence types and examples, load `references/DOMAIN-TABLES.md` § 1.

### 3.2 Search Sub-Agent

Do NOT load the entire KB into context. Delegate lookups to a search sub-agent.

**Use search sub-agent when:**
- Before creating a new proposal ("prior work on this topic?")
- During DIAGNOSE ("weakest items?")
- During TEST ("evidence for/against?")
- During META-ASSESS ("retreading ground?")
- When resuming a session ("latest state?")

**Skip sub-agent when:**
- KB has < 10 notes (read `_index.md` + files directly)
- Note was just created in current iteration
- Quick count check (read `_index.md` header only)

Sub-agent reads `_index.md` for tag lookup → reads matching notes (max 10) →
returns top 5 results with {id, title, status, confidence, summary}, relevant
evidence, contradictions, and gaps.

For the full search protocol and query format: load `references/KNOWLEDGE-BASE.md` § 6.

---

## 4. Forced Feedback Loop Engine

### 4.1 Core Loop

After the initial pass (Section 9), enter this loop until time expires:

```
WHILE elapsed_minutes < min_required_minutes:

  STEP 1 — DIAGNOSE
    Read latest 1–3 reports (incremental protocol). Identify:
    weakest decision, largest gap, untested assumption, contradictions.

  STEP 2 — PROPOSE (mandatory novelty)
    Combine 2+ existing findings → at least one NEW proposal.
    Strategies: INVERSION | COMBINATION | ANALOGY | DECOMPOSITION |
    EDGE CASE | TEMPORAL | STAKEHOLDER SHIFT | CONSTRAINT FLIP |
    TECHNOLOGY SWAP
    Create new note in kb/raw/. Update kb/_index.md.

    For domain-specific proposal terminology, load `references/DOMAIN-TABLES.md` § 4.

  STEP 3 — TEST
    Validate or falsify the new proposal and weakest existing decision.
    If tools fail: log gap → lower confidence → try alternative → log "untestable".

    For domain-specific test methods, load `references/DOMAIN-TABLES.md` § 5.

  STEP 4 — UPDATE
    Move tested notes: kb/raw/ → kb/archive/ (falsified) or kb/curated/ (validated).
    Update confidence scores in note frontmatter. Rebuild kb/_index.md.
    Write new report to work-log.md.

  STEP 5 — META-ASSESS
    "Am I progressing or circling?" / "Different angle?" / "Expert challenge?"
    If stuck 2 consecutive loops → pivot and log reason.

END WHILE
```

### 4.2 Loop Quality Rules

- Each loop MUST produce material novelty (new evidence, falsification, structural
  revision, confidence change, or architectural decision).
- No-update loop = `LOOP_FAIL` → immediate pivot.
- Minimum 1 new proposal per 2 loops averaged. Zero proposals = FAIL.
- Report types: `feedback` | `milestone` | `synthesis`.

### 4.3 Anti-Stagnation

Perspective shift | Scope zoom | Adversarial thinking | Cross-domain transfer |
Constraint relaxation.

---

## 5. Time Execution Policy

- **Idle waiting is FORBIDDEN.** No sleep, polling, or non-productive padding.
- While `elapsed < min_required_minutes`, MUST NOT: declare complete, ask user to
  stop, produce final summary, or say "this is sufficient."
- Check elapsed time each loop. Log in every report header.

---

## 6. Source Hierarchy

Three tiers: L1 (primary) → L2 (authoritative) → L3 (commentary).
Every key decision needs at least one L1 anchor. L3-only = `[LOW_CONFIDENCE]`.

For domain-specific tier definitions, load `references/DOMAIN-TABLES.md` § 2.

---

## 7. Decision Hygiene

Four-tier tags: **Verified** → **Derived** → **Unverified** → **Deferred**.
No absolute language. Every major decision needs a counter-proposal and a
falsification test. Every decision records at least one rejected alternative.
Deferred items need a `next_action` note.

For domain-specific tag names, load `references/DOMAIN-TABLES.md` § 3.

---

## 8. Uncertainty and Gap Control

Maintain as dedicated notes in `kb/curated/`:
- `kb/curated/UNCERTAINTY-REGISTER.md`: Item | Type (epistemic/aleatory) | Likelihood | Impact | Next Test
- `kb/curated/GAP-REGISTER.md`: Gap ID | Missing Data | Impact | Fallback | Confidence Penalty | Follow-up

These are the only notes edited in-place (not append-only). Tag them `type/register`.

If a tool/search fails: run fallback → lower confidence → log `GAP-#` in the gap register.

---

## 9. Routine Order (Initial Pass)

Complete once before entering the feedback loop:

**A. Ideation** → create notes in `kb/raw/`: candidates, uncertainties, failure points.
**B. Evidence** → collect, map to claims in note evidence sections. Log gaps.
**C. Validation** → disprove top claims. Move notes to `kb/archive/` or `kb/curated/`.
**D. First Synthesis** → write synthesis report. Update confidence scores in notes.

Then enter the Forced Feedback Loop (Section 4).

---

## 10. Reproducibility

Final report must include: commands/tool calls used, source list with timestamps,
assumptions, key decision points with alternatives, rerun notes.

---

## 11. Absolute Prohibitions

Before starting work and before the FAIL Gate, load and review:
`references/PROHIBITIONS.md`

27 prohibited behaviors across 5 categories (Time, Reasoning, Files, Process,
Communication). Any violation = automatic FAIL. Key highlights:
- No early termination, idle waiting, or quality shortcuts (T1–T4)
- No self-confirmation, fabrication, or skipped falsification (R1–R8)
- No batched writes, deletions, absolute-coordinate refs, full-KB loading, missing frontmatter, or uncorrected line-count mismatches (F1–F9)
- No scope drift, silent abandonment, or unchecked FAIL gates (P1–P6)
- No fake confidence, hidden uncertainty, hollow offers, or filler (X1–X6)

---

## 12. Strict FAIL Gate

Mark `INCOMPLETE` if ANY missing:

- [ ] Start contract (§1)
- [ ] Work-log with formatted reports (§2)
- [ ] KB directory with notes in correct dirs (§3)
- [ ] Search sub-agent used for KB queries when applicable (§3.2)
- [ ] Minimum runtime consumed by active work (§5)
- [ ] D#-E# mapping (§3.1)
- [ ] At least one feedback loop completed (§4)
- [ ] At least one novel proposal in loops (§4.2)
- [ ] Uncertainty + gap registers (§8)
- [ ] Reproducibility section (§10)
- [ ] No Absolute Prohibition violated (§11 / `references/PROHIBITIONS.md`)
- [ ] Every report has line-count header
- [ ] All `lines:` values verified correct (§2.6)
- [ ] Report #0 uses exact key names from §1 schema
- [ ] Cross-references use relative coordinates

No "conditional pass."

---

## 13. End Block

Write ONLY after `elapsed_minutes >= min_required_minutes`:

```
=== FINAL REPORT | elapsed: <MM:SS> ===
end_time: <system time>
total_feedback_loops: <N>
total_proposals_generated: <N>
total_proposals_validated: <N>
total_proposals_falsified: <N>

## Summary: Confirmed / Uncertain / Follow-up Required
## Key Artifacts
## Biggest Surprise
## Recommendation for Next Session
```

---

## 14. Multi-Session Continuity

When resuming: read `kb/_index.md` for current state → use search sub-agent for
recent notes (last 5 created/updated) → read `work-log.md` incrementally (§2.3) →
create Report #(N+1) → continue feedback loop → use relative coordinates for all refs.
