# Expansion Multipliers Reference

Reference tables for scope-check calculations. Based on empirical research from QSM Database (2,192 projects), IFPUG standards, and context degradation studies.

## Language Multipliers

Derived from Lines of Code per Function Point ratios, normalized to Python baseline.

| Language | LOC/FP | Multiplier | Tokens/LOC | Notes |
|----------|--------|------------|------------|-------|
| **Python** | ~21 | 1.0 | ~10 | Baseline, most concise |
| **Go** | ~35 | 1.8 | ~8 | Explicit but compact |
| **TypeScript** | ~47 | 2.2 | ~7 | Type annotations add bulk |
| **JavaScript** | ~47 | 2.2 | ~7 | Similar to TS without types |
| **Rust** | ~50 | 2.4 | ~9 | Ownership + explicit errors |
| **Java** | ~53 | 2.5 | ~10 | Verbose OOP patterns |
| **C#** | ~54 | 2.6 | ~10 | Similar to Java |
| **C++** | ~50-186 | 4.0 | ~11 | High variance, use conservative |

**Usage in formula:**
```
base_output_tokens = spec_tokens × language_multiplier × 15
```

The `× 15` represents middle-of-range expansion from spec token to output tokens (accounting for 10-20 LOC per spec token, then ~10 tokens per LOC).

## TDD Multipliers

Test code adds to total output. Multiplier based on test-to-implementation ratio.

| TDD Level | Test:Impl Ratio | Multiplier | Description |
|-----------|-----------------|------------|-------------|
| **none** | 0:1 | 1.0 | No tests generated |
| **minimal** | 1:1 | 2.0 | Basic happy path tests |
| **standard** | 2:1 | 3.0 | Happy + error + edge cases |
| **comprehensive** | 3:1 | 4.0 | Full coverage + integration tests |

**Detection heuristics:**
```
If CLAUDE.md declares tdd: → use declared value
Else if project has test infrastructure:
  - jest.config, pytest.ini, tests/ dir → "standard"
  - test_file_count / source_file_count > 0.3 → "comprehensive"
  - test patterns exist but sparse → "minimal"
Else → "minimal" (conservative default)
```

## Spec Type Variance

Different spec types have different interpretation latitude, affecting output variability.

| Spec Type | Variance | Description | Detection |
|-----------|----------|-------------|-----------|
| **vision** | ±50% | High-level goals, requires architectural decisions | Contains: "goals", "vision", "scope", "objectives" |
| **design** | ±25% | Detailed with wireframes/models, constrained output | Contains: ASCII diagrams, data models, code samples |
| **mixed** | ±35% | Combination of vision and design sections | Default when unclear |

**Usage in formula:**
```
conservative_estimate = with_tdd × (1 + variance)
aggressive_estimate = with_tdd × (1 - variance × 0.5)
```

## Context Budget Thresholds

Fixed 200K context window with safety margins based on degradation research.

| Threshold | Tokens | % of 200K | Status | Action |
|-----------|--------|-----------|--------|--------|
| Safe | <100K | <50% | GREEN | Proceed freely |
| Caution | 100-130K | 50-65% | YELLOW | Monitor, include safety recs |
| Warning | 130-160K | 65-80% | ORANGE | Recommend chunking |
| Critical | >160K | >80% | RED | Must decompose |

**Research basis:**
- "Lost in the Middle" (Liu et al., 2024): U-shaped attention, 10-25% accuracy drops
- RULER benchmark: Only 50% of models maintain quality at claimed limits
- Chroma "Context Rot": 13.9-85% degradation as length increases
- Anthropic guidance: Operate at 70-80% maximum, reserve 10-20% for output

## Complexity Thresholds

### NL-CC (Natural Language Cyclomatic Complexity)

Count decision points in spec text:
```
NL_CC = 1 + conditionals + alternatives + iterations + exceptions

Conditionals: "if", "when", "in case of", "provided that"
Alternatives: "or", "either...or", "alternatively"
Iterations: "for each", "while", "until", "repeated"
Exceptions: "unless", "except when", "failure to"
```

| NL-CC | Complexity | Chunking Impact |
|-------|------------|-----------------|
| 1-10 | Low | Single chunk OK |
| 11-20 | Moderate | Consider splitting at natural boundaries |
| 21-50 | High | Likely needs 2-4 chunks |
| >50 | Extreme | Requires significant decomposition |

### Coupling Ratio

```
coupling_ratio = shared_entities / total_entities
```

| Ratio | Chunkability | Strategy |
|-------|--------------|----------|
| <0.3 | Highly chunkable | Semantic chunking with 15% overlap |
| 0.3-0.5 | Moderately chunkable | Hierarchical chunking with shared context header |
| >0.5 | Resists chunking | Architectural decomposition before generation |

## Token Counting Quick Reference

| Content Type | Formula | Example |
|--------------|---------|---------|
| English prose | chars / 4 | 1000 chars ≈ 250 tokens |
| Code | lines × 10 | 100 lines ≈ 1000 tokens |
| Tables/structured | chars / 3 | 900 chars ≈ 300 tokens |
| Mixed document | (prose + code + tables) × 1.15 | Add 15% buffer |

**Rough equivalents:**
- 200K tokens ≈ 500 pages of text
- 200K tokens ≈ 20,000 lines of code
- 1 token ≈ 4 characters ≈ 0.75 words

## Coordinator Load Formula

For multi-agent (chunked) execution:

```
coordinator_load = (
    original_spec_tokens +           # Still needs spec for orchestration
    (num_chunks × 2000) +            # ~2K per chunk summary
    (num_chunks × 500) +             # ~500 tokens handoff overhead
    5000                             # ~5K conversation buffer
)
```

**Safety thresholds:**
- <15% of budget: Safe, proceed with multi-agent
- 15-25%: Monitor coordinator context
- >25%: Consider hierarchical orchestration (sub-coordinators)

## References

- QSM Database: Function Point to LOC ratios, 2,192 projects
- IFPUG ISO/IEC 20926:2009: Function Point counting standards
- Liu et al. (2024): "Lost in the Middle" - https://arxiv.org/abs/2307.03172
- Chroma Research: "Context Rot" study across 18 LLMs
- RULER Benchmark: Long-context model evaluation
- Anthropic: "Effective context engineering for AI agents"
