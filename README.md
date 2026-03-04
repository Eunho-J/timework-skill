# Forced Feedback Loop

**[English](README.md)** | **[한국어](README.ko.md)** | **[日本語](README.ja.md)** | **[中文](README.zh.md)**

An agent skill that forces continuous self-feedback loops until the budget (time or loop count) is exhausted, steadily improving work quality without ever stopping voluntarily.

## Problems This Solves

- AI does one shallow pass and declares "done"
- Hollow offers like "I can also do X next" instead of actually doing it
- Work history lives only in ephemeral context and vanishes after the session
- Confirmation bias — the agent validates its own conclusions without real testing
- Search space narrows over time when the agent only recombines existing information
- Knowledge files grow monolithic and become unsearchable as ideas accumulate

## Core Principles

1. **Never stop before the budget is spent** — active work only during `min_required_minutes` and/or `min_required_loops`
2. **Generate new proposals every loop** — both convergent (recombine known) and divergent (explore unknown)
3. **Write everything to files** — reverse-chronological reports in `work-log.md`, session continuity guaranteed
4. **Manage knowledge as individual notes** — Obsidian-style KB with per-idea files, searched by a dedicated sub-agent

## Skill File Structure

```
forced-feedback-loop/
├── SKILL.md                        # Core execution policy (loaded on activation)
└── references/
    ├── DOMAIN-TABLES.md            # Domain-specific tables (lazy loaded)
    ├── PROHIBITIONS.md             # 27 prohibited behaviors (lazy loaded)
    └── KNOWLEDGE-BASE.md           # KB system: note format, tags, search protocol
```

## Runtime Artifacts

The agent creates these files in the working directory during execution:

```
<working-directory>/
├── work-log.md                     # Full work history (reverse-chronological report stack)
└── kb/                             # Knowledge Base (Obsidian-like)
    ├── _index.md                   # Tag → file auto-index
    ├── raw/                        # Unverified ideas, drafts
    │   ├── 20260303-143000-xxx.md
    │   └── ...
    ├── archive/                    # Rejected items (with reason)
    │   ├── 20260303-150000-yyy.md
    │   └── ...
    └── curated/                    # Validated items
        ├── 20260303-160000-zzz.md
        ├── UNCERTAINTY-REGISTER.md # Uncertainty register
        └── GAP-REGISTER.md         # Data gap register
```

### Why Individual Notes?

A monolithic 3-file approach grows unwieldy as ideas accumulate:

- Loading everything into context wastes tokens
- No way to retrieve only relevant items
- Search efficiency degrades as sessions get longer

The KB system stores each idea/decision/evidence as an individual Markdown file with YAML frontmatter (tags, status, confidence, etc.), so the search sub-agent can retrieve exactly what's needed.

### Search Sub-Agent

When the agent looks up information in the KB, it delegates to a search sub-agent instead of reading everything:

1. Tag-based lookup via `_index.md`
2. Read matching note files (max 10)
3. Return top 5 results as {id, title, status, confidence, summary}
4. Include relevant evidence, contradictions, and gaps

If the KB has fewer than 10 notes, the agent reads files directly without a sub-agent.

## Supported Domains

| task_type | Use case |
|---|---|
| `research` | Investigation, literature review |
| `code` | Development, engineering |
| `design` | UI/UX design |
| `document` | Writing, documentation |
| `analysis` | Data analysis |
| `project` | Project management |
| `other` | Anything else |

Evidence types, source hierarchy, tag systems, and test methods adapt automatically per domain. See `references/DOMAIN-TABLES.md` for detailed mappings.

## Usage

### Installation

Extract the downloaded zip and place it in your agent's skill directory.

| Agent | Path |
|---|---|
| Perplexity | Upload via User Settings |
| OpenAI Codex | `~/.codex/skills/forced-feedback-loop/` |
| Claude Code | `~/.claude/skills/forced-feedback-loop/` |
| Augment | `~/.augment/skills/forced-feedback-loop/` |
| VS Code (Copilot) | `.agents/skills/forced-feedback-loop/` (project root) |

### Examples

**Research:**
```
Spend 30 minutes researching "EV battery market trends in South Korea".
```

**Code:**
```
Spend 20 minutes improving this API server's response time. Current p99 latency is 500ms.
```

**Design:**
```
Spend 15 minutes improving the usability of this dashboard layout.
```

**Document:**
```
Spend 25 minutes strengthening the argument structure of this technical proposal.
```

**Count-based (no time limit):**
```
Run 10 feedback loops to improve this function's error handling.
```

**Both (time + count):**
```
Spend at least 20 minutes AND run at least 5 loops optimizing this query.
```

Specify a time budget, a loop count, or both. The agent activates the skill automatically and runs feedback loops until the termination condition is met.

### Termination Modes

| `min_required_minutes` | `min_required_loops` | Behavior |
|---|---|---|
| set | — | **Time-based**: run until time expires |
| — | set | **Count-based**: run exactly N loops |
| set | set | **Both**: run until both thresholds are met |

If neither is specified, defaults to 5 minutes.

### Convergent × Divergent Dual Proposals

Each loop's proposal step alternates between two modes:

| Mode | Direction | Description |
|---|---|---|
| **Convergent** | Known → new hypothesis | Combine 2+ existing findings from the KB into a new proposal |
| **Divergent** | Unknown → new hypothesis | Identify an adjacent question the KB can't answer yet, then actively search for new information to test it |

At least 1 divergent proposal must appear in every 3 consecutive loops. Pure convergence narrows the search space.
Divergent proposals must pass a relevance check against `task_goal`, so they stay on-topic.

### Reading work-log.md

Reports stack newest-on-top. The first line of each report declares its line count, enabling incremental reading:

```
=== Report #3 | lines: 6 | elapsed: 14:30 | type: feedback ===   ← line 1
DIAGNOSE: Weakest decision is D2 (confidence 0.4).
PROPOSE: Inversion — what if we use push instead of pull?
TEST: grep codebase for event-driven patterns → 3 hits.
UPDATE: D2 confidence raised to 0.65.
META: Progressing. Next loop targets D4.                         ← line 6
---
=== Report #2 | lines: 4 | elapsed: 10:00 | type: feedback ===
...
```

**`lines:` rule:** Count from the header line (inclusive) to the last content line (inclusive). The `---` separator and blank lines between reports are excluded.
The agent verifies this value after writing each report and runs a lint check in scripting environments.

Cross-references use relative coordinates (`K-reports-below`, `line N below`), so references survive log compaction.

## Multi-Session Continuity

If `work-log.md` and the `kb/` directory remain from a previous session, the agent picks up where it left off. It reads `kb/_index.md`, uses the search sub-agent to review recent notes, restores context via incremental reading, and continues the feedback loop.

```
Continue the battery market research from yesterday for another 20 minutes.
```

## Compatibility

Follows the [Agent Skills open spec](https://agentskills.io/specification) and works with any agent that supports this spec.
