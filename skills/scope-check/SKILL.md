---
name: scope-check
description: Use before implementing specs or when authoring design documents - evaluates specification complexity against context window constraints, provides grounded expansion estimates, and recommends chunking strategies when specs exceed safe bounds. CRITICAL - Activate BEFORE writing any plan, design, or spec document to prevent mid-implementation context collapse.
---

# Scope Check

## Overview

Evaluate specification documents against Claude Code's context window constraints. Provides grounded estimations of whether a spec will fit within safe context bounds and recommends chunking strategies when it won't.

**Two modes:**
- **Standalone**: Run `/scope-check path/to/spec.md` before implementation
- **Automatic gate**: Self-activates when detecting planning workflows before artifacts are written

## Activation Protocol

**When to activate (automatic detection):**

You MUST activate this skill when you detect ANY of these workflow signals:

**Planning workflow indicators:**
- About to write a design document to `docs/plans/` or `docs/design/`
- Phrases: "write the validated design", "saving design to", "design document"
- Phrases: "writing plan to", "implementation plan", "plan document header"
- User says: "help me design", "let's plan", "write a spec for", "brainstorm this feature"

**Pre-write detection:**
- About to call Write tool with a file path containing: `plans/`, `design/`, `spec/`, `architecture/`
- Estimated content size >5K characters
- Creating documentation for implementation

**Other planning skill activity:**
- Conversation mentions: "brainstorming skill", "writing-plans skill", "planning workflow"
- You are in the process of creating comprehensive design or planning artifacts

**Interception workflow:**

When ANY trigger detected:
1. BEFORE writing the file, announce: "Running scope-check to validate document size and context budget"
2. Estimate document size based on conversation content so far
3. Run the analysis (detect stack, estimate expansion, calculate status)
4. Present traffic light status (GREEN/YELLOW/ORANGE/RED)
5. Only proceed to write if:
   - GREEN/YELLOW status, OR
   - User explicitly acknowledges and overrides RED/ORANGE warning

**Why this matters:**
- Prevents writing specs that exceed Read tool's 25K token limit (cannot be read in single pass)
- Ensures implementation stays within context budget (prevents mid-work context collapse)
- Forces chunking decisions during design phase when architectural judgment is fresh

This is a GATE skill - you actively assert yourself into planning workflows to prevent context budget issues.

## Quick Start

```bash
# Basic check
/scope-check docs/feature-spec.md

# With explicit overrides
/scope-check docs/feature-spec.md --stack typescript --tdd comprehensive
```

## The Analysis Process

### Step 1: Detect Project Context

**Stack detection (in order):**
1. CLAUDE.md `stack:` declaration (if present)
2. `package.json` → TypeScript/JavaScript
3. `pyproject.toml` / `requirements.txt` → Python
4. `*.csproj` / `*.sln` → C#
5. `Cargo.toml` → Rust
6. `go.mod` → Go
7. Fallback: Python (most conservative expansion ratio)

**TDD detection:**
1. CLAUDE.md `tdd:` declaration (if present)
2. Scan for test infrastructure:
   - `pytest.ini`, `jest.config`, `*.test.ts`, `tests/` → "standard" assumed
   - High test file ratio (>0.3 test files per source) → "comprehensive"
   - No test patterns found → "minimal" assumed
3. **Confirm with user**: "Detected [X] TDD level from project. Correct for this spec?"

### Step 2: Analyze Specification

**Token estimation:**
```
Prose sections: character_count / 4
Code blocks: line_count × 10
Tables/structured: character_count / 3
Total + 15% buffer for formatting variance
```

**Complexity metrics (no model calls required):**
- **NL-CC (Natural Language Cyclomatic Complexity)**: Count conditionals, alternatives, loops, exceptions
- **Coupling density**: Entity references across sections (flags chunking resistance)
- **Function points**: Count inputs, outputs, inquiries, data entities

### Step 3: Calculate Expansion Estimates

```
base_output = spec_tokens × language_multiplier × 15
with_tdd = base_output × tdd_multiplier
conservative = with_tdd × (1 + spec_type_variance)
```

See MULTIPLIERS.md for language, TDD, and spec type values.

### Step 4: Determine Status

**Two constraints to check:**

#### Constraint 1: Spec File Size (Read Tool Limit)

Claude Code's Read tool has a hard **25,000 token limit** (~100K characters). Specs exceeding this cannot be read in a single pass.

| Spec Size | Status | Action |
|-----------|--------|--------|
| <20K tokens (~80K chars) | OK | Spec readable in single pass |
| 20-25K tokens (80-100K chars) | WARNING | Near limit, consider splitting |
| >25K tokens (~100K+ chars) | **MUST SPLIT** | Cannot write as single file |

**If spec exceeds 25K tokens**: Block writing single file. Split into multiple spec files with descriptive slug names based on semantic boundaries:

```
spec-auth-flow.md           # Authentication and session management
spec-data-models.md         # Core entities and relationships
spec-api-endpoints.md       # REST/GraphQL interface definitions
spec-ui-components.md       # Frontend component specifications
```

Use coupling analysis to determine boundaries - high-coupling sections stay together, low-coupling sections split naturally.

#### Constraint 2: Implementation Context Budget

**See CONTEXT-BUDGET.md for configurable assumptions and customization instructions.**

**Default baseline (Sonnet 4.5):**
- Total window: 200K tokens
- Fixed overhead: ~27K (system, tools, memory, skills)
- Autocompact buffer: 45K (reserved, triggers compaction)
- **Working budget: ~128K tokens** (200K - 27K - 45K)

**Status thresholds (% of 128K working budget):**

| Status | Threshold | % Used | Meaning |
|--------|-----------|--------|---------|
| GREEN  | <50K | <39% | Safe single-pass, proceed freely |
| YELLOW | 50-75K | 39-58% | Fits but tight, include context management recommendations |
| ORANGE | 75-100K | 58-78% | Chunking recommended, offer interactive refinement |
| RED    | >100K | >78% | Must decompose, block without override |

**Note:** Users can customize thresholds in CONTEXT-BUDGET.md based on their actual `/context` overhead and risk tolerance.

## Output Format

### Traffic Light Summary (always shown)

```
┌─────────────────────────────────────────────────────┐
│ SCOPE CHECK: [STATUS EMOJI] [STATUS] - [One-liner] │
├─────────────────────────────────────────────────────┤
│ Spec: [path] ([X] tokens)                           │
│ Stack: [Language] + [TDD level] TDD                 │
│ Estimated output: [range] tokens                    │
│ Working budget: 128K (200K - 27K overhead - 45K buffer) │
│ Usage: [Y]K ([Z]% of working budget)                │
│                                                     │
│ → [Primary recommendation]                          │
│ → [Secondary recommendation if applicable]          │
└─────────────────────────────────────────────────────┘
```

### Detailed Analysis (on request or ORANGE/RED)

```
DETAILED ANALYSIS
─────────────────
Token breakdown:
  Prose: [X] tokens ([Y] chars)
  Code samples: [X] tokens ([Y] lines)
  Tables: [X] tokens ([Y] chars)

Complexity metrics:
  NL-CC: [X] ([low/moderate/high])
  Coupling ratio: [X] ([chunkable/resists chunking])
  Function points: [X] UFP

Expansion calculation:
  Base: [formula] = [X] tokens
  With TDD ([multiplier]×): [Y] tokens
  Conservative (+[Z]%): [W] tokens
```

## Interactive Chunking (ORANGE/RED status)

When chunking is recommended, enter interactive mode:

### Step 1: Propose Boundaries

```
CHUNKING ANALYSIS
─────────────────
Your spec has coupling ratio [X] ([chunkable/resistant]).
Detected [N] natural boundaries:

  Chunk 1: Lines [X-Y] "[Section Name]"
           ~[X] tokens → ~[Y]K output
           Dependencies: [none/list]

  Chunk 2: Lines [X-Y] "[Section Name]"
           ~[X] tokens → ~[Y]K output
           Dependencies: [Chunk N]

Recommended order: [sequence with parallelization notes]

Does this split align with your architecture?
  [A] Looks good, generate execution plan
  [B] Adjust boundaries (tell me what to move)
  [C] Different split strategy (describe your preference)
```

### Step 2: Refine Based on Feedback

If B or C selected, ask targeted follow-ups:
- "Should [X section] stay with [Y], or move to a separate chunk?"
- "You mentioned [Z] depends on [W] - should we reorder?"

Recalculate estimates after each adjustment.

### Step 3: Generate Execution Plan

```
EXECUTION PLAN
──────────────
Shared context header (include in each chunk):
  - Core types: [list]
  - Constants: [list]
  - ~[X] tokens overhead per chunk

Chunk 1 → implement → verify → commit
Chunk 2 → implement → verify → commit
[Parallel chunks marked with ┬┘ notation]

Total estimated API calls: [range]
```

### Coordinator Load Analysis

For multi-agent execution, calculate coordinator burden:

```
SUBAGENT ANALYSIS
─────────────────
Chunks: [N]
Estimated summaries: [X]K tokens ([N] × 2K)
Handoff overhead: [Y]K tokens
Coordinator total: ~[Z]K tokens

[✓/⚠️] [Assessment of coordinator budget safety]
```

If coordinator load >25% of budget, recommend:
- Hierarchical orchestration (sub-coordinators per 4 chunks)
- Reduce chunk count by grouping related sections

## Integration Gate Mode

When invoked by `brainstorming` or `writing-plans` before writing artifacts:

### Check 1: File Size Constraint (Hard Block)

**Before writing ANY spec**, estimate output size. If spec would exceed 25K tokens (~100K chars):

**BLOCKED - Must split first:**
```
"Spec would be ~[X]K tokens, exceeding 25K read limit.
Must split into semantic chunks:

Proposed split:
  spec-[slug-1].md  (~[X]K tokens) - [description]
  spec-[slug-2].md  (~[X]K tokens) - [description]
  spec-[slug-3].md  (~[X]K tokens) - [description]

Proceed with this split? [A] Yes [B] Suggest different boundaries"
```

No override allowed - this is a hard tooling constraint.

### Check 2: Context Budget Status

**GREEN/YELLOW**: Proceed to write, include status in document header
```markdown
<!-- scope-check: GREEN - 45K estimated, safe for single-pass -->
```

**ORANGE**: Show chunking recommendations, ask:
"Spec is ORANGE status. Options:
  [A] Split now using recommended boundaries
  [B] Write as-is with warning header
  [C] Show detailed analysis first"

**RED**: Block write, require resolution:
"Spec is RED status ([X]K estimated, exceeds 100K safe limit).
Must either:
  [A] Enter interactive chunking to split spec
  [B] Override with explicit acknowledgment"

**Override syntax**: User says "Write anyway, I understand the risk"
```markdown
<!-- scope-check: RED (override) - 245K estimated, chunking recommended -->
```

## Context Pressure Recommendations

Include with YELLOW+ status. See SAFETY.md for full details.

**Key mitigations:**
- Dual instruction placement (start AND end of prompts)
- Query positioning (specific asks at very end)
- Compaction triggers at 80% utilization
- Checkpoint often (commit working code frequently)

## Key Principles

- **Prevention over detection**: Right-size specs before implementation starts
- **Grounded estimates**: Use empirically-validated expansion ratios, not guesses
- **Interactive refinement**: Chunking decisions need human architectural judgment
- **Safe defaults**: Conservative estimates prevent mid-implementation context collapse
- **Transparency**: Show the math so users can calibrate their own intuitions
