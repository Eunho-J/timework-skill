# Domain-Specific Reference Tables

Load this file when you need domain-specific details for evidence types, source
hierarchy, decision tags, test methods, or proposal terminology.

---

## 1. Decision-Evidence Types by Domain

Common fields (required for ALL evidence entries):
- `fetched_at` timestamp
- `method` — how the evidence was obtained

| Domain | Evidence Type | Required Fields |
|---|---|---|
| Research / Analysis | `[source]` | `source_url`, `key_quote` (1–3 lines) |
| Code / Engineering | `[test_result]` | `test_command`, `pass/fail`, `output_summary` |
| | `[benchmark]` | `benchmark_tool`, `metric`, `value`, `baseline` |
| | `[code_review]` | `file_path#line`, `issue_or_confirmation` |
| Design / UX | `[mockup]` | `mockup_path`, `variant_id`, `rationale` |
| | `[user_test]` | `participants`, `task_completion_rate`, `key_observation` |
| | `[heuristic_eval]` | `heuristic_name`, `severity`, `finding` |
| Document / Writing | `[draft_review]` | `section`, `reviewer_perspective`, `verdict` |
| | `[consistency_check]` | `rule_applied`, `pass/fail`, `location` |
| Project / General | `[stakeholder_input]` | `stakeholder_role`, `position_summary` |
| | `[constraint_check]` | `constraint`, `met/unmet`, `impact` |

### Example (code task)
```
D1: "Use connection pooling for DB access"
  E1.1: [benchmark] method=locust_load_test fetched_at=12:05
        benchmark_tool=locust metric=p99_latency value=45ms baseline=120ms
  E1.2: [source] method=web_search fetched_at=12:06
        source_url=https://docs.sqlalchemy.org/... key_quote="pool_size default is 5"
```

### Example (design task)
```
D1: "Use bottom navigation instead of hamburger menu"
  E1.1: [heuristic_eval] method=nielsen_heuristic fetched_at=12:10
        heuristic_name="Recognition over recall" severity=major
        finding="Hamburger hides primary actions, increasing interaction cost"
  E1.2: [source] method=web_search fetched_at=12:11
        source_url=https://nngroup.com/... key_quote="Hamburger menus reduce discoverability by 21%"
```

---

## 2. Source & Authority Hierarchy

| Tier | Research / Analysis | Code / Engineering | Design / UX | Document / Writing | Project / General |
|---|---|---|---|---|---|
| **L1** (primary) | Official law, regulator, standard, raw data | Official docs, language spec, RFC, API reference | Design system spec, platform HIG (Apple/Google/Material), WCAG standard | Style guide, official policy, primary source | Contract, SLA, approved budget, signed-off scope |
| **L2** (authoritative) | Research institutes, academic papers, vendor technical docs | Established libraries' docs, authoritative tutorials (e.g. official guides), published benchmarks | NN/g research, Baymard studies, peer-reviewed UX research, major design case studies | Authoritative references, published templates, domain exemplars | Industry benchmarks, PMI/standard methodologies, comparable project post-mortems |
| **L3** (commentary) | News, blog, community, secondary commentary | Blog posts, Stack Overflow, forum answers, personal opinions | Dribbble/Behance examples, design blog posts, individual portfolio pieces | Blog posts, opinion pieces, informal reviews | Anecdotal experience, informal advice, unverified estimates |

Rules:
- Every key conclusion or decision must include at least one L1 anchor.
- L3-only conclusions are tagged `[LOW_CONFIDENCE]` and flagged for follow-up.
- When L1 sources conflict, log the conflict explicitly and investigate further.
- For code: passing tests alone is L2 at best. Pair with L1 documentation to confirm
  intended behavior, not just observed behavior.

---

## 3. Decision Hygiene Tags

| Tag | Research | Code | Design | Document | Project |
|---|---|---|---|---|---|
| **Verified** | `[FACT]` | `[TESTED]` | `[VALIDATED]` | `[CONFIRMED]` | `[APPROVED]` |
| **Derived** | `[INFERENCE]` | `[REASONED]` | `[PATTERN]` | `[INTERPRETED]` | `[ESTIMATED]` |
| **Unverified** | `[HYPOTHESIS]` | `[ASSUMED]` | `[EXPLORATORY]` | `[DRAFT]` | `[PROPOSED]` |
| **Deferred** | `[FOLLOW_UP]` | `[TODO]` | `[NEEDS_TEST]` | `[TBD]` | `[BLOCKED]` |

Tier definitions:
1. **Verified**: Backed by L1/L2 evidence or passing tests
2. **Derived**: Logical conclusion from verified items, but not directly tested
3. **Unverified**: Untested or partially tested proposition
4. **Deferred**: Recognized but cannot be resolved in the current loop

Rules:
- No numeric superiority claim without measurable basis.
- Avoid absolute language ("always", "never", "best").
- Include at least one counter-proposal and one falsification/stress test per major decision.
- Every decision must record at least one rejected alternative with rationale.
- `Deferred` items must have a `next_action` note describing what unblocks them.

---

## 4. Proposal Terminology by Domain

| Domain | Proposal is called | Example |
|---|---|---|
| Research | Hypothesis | "Market size may be 2x larger when including adjacent segments" |
| Code | Design alternative | "Replace polling with WebSocket for real-time updates" |
| Design | Variant / Option | "Card layout variant with progressive disclosure" |
| Document | Draft angle | "Restructure argument to lead with cost impact" |
| Project | Approach | "Outsource module B to reduce timeline by 3 weeks" |
| Analysis | Interpretation | "Correlation may be driven by confounding variable Z" |

---

## 5. Test Methods by Domain

| Domain | Primary test methods |
|---|---|
| Research | Web search, academic search, data analysis, source cross-validation |
| Code | Run tests, execute benchmarks, build & check errors, lint, type-check |
| Design | Create mockup/wireframe, heuristic evaluation, compare with design systems, accessibility check |
| Document | Re-read for coherence, fact-check claims, consistency pass, audience-perspective review |
| Project | Constraint check (timeline, budget, skills), dependency analysis, risk assessment |
| Analysis | Statistical test, sensitivity analysis, reproduce with different parameters |

Multi-method testing is encouraged: e.g., for a code decision, run both a
benchmark AND check official docs for best practices.

---

## 6. Knowledge Base Semantics by Domain

| Concept | Research | Project/Code | Document/Design |
|---|---|---|---|
| `kb/raw/` notes | Hypotheses | Design options | Draft sections |
| `kb/archive/` notes | Falsified claims | Rejected approaches | Discarded drafts |
| `kb/curated/` notes | Validated findings | Confirmed decisions | Final sections |

Each note is an individual Markdown file with YAML frontmatter.
See `references/KNOWLEDGE-BASE.md` for full note format and lifecycle.
