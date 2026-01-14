# Context Budget Assumptions

**Purpose:** Configurable baseline assumptions for scope-check calculations. Users can customize these values based on their actual Claude Code setup.

## Default Configuration

```yaml
# Total context window for Claude Sonnet 4.5
total_window: 200000  # tokens

# Fixed overhead (always present, unavailable for work)
fixed_overhead: 27000  # tokens
  # Breakdown:
  # - System prompt: ~3,000 tokens
  # - System tools: ~16,700 tokens
  # - Custom agents: ~250 tokens
  # - Memory files: ~5,000 tokens (varies by CLAUDE.md size)
  # - Skills: ~1,600 tokens (varies by loaded skills)

# Auto-compaction buffer (reserved, triggers compaction)
autocompact_buffer: 45000  # tokens (22.5% of total)

# Calculated working budget
working_budget: 128000  # tokens (total - overhead - buffer)
```

## Status Thresholds

Based on `working_budget` (128K default):

| Status | Threshold | % of Working Space | Meaning |
|--------|-----------|-------------------|---------|
| GREEN  | <50K | <39% | Safe single-pass, proceed freely |
| YELLOW | 50-75K | 39-58% | Fits but tight, include context management |
| ORANGE | 75-100K | 58-78% | Chunking recommended |
| RED    | >100K | >78% | Must decompose |

## Customization Instructions

### Step 1: Check Your Actual Context Usage

Run `/context` in Claude Code to see your actual overhead:

```
/context
  ⛁ System prompt: 3.0k tokens
  ⛁ System tools: 16.7k tokens
  ⛁ Custom agents: 247 tokens
  ⛁ Memory files: 4.9k tokens
  ⛁ Skills: 1.6k tokens
  ⛝ Autocompact buffer: 45.0k tokens
```

### Step 2: Calculate Your Overhead

```
Your overhead = System prompt + Tools + Agents + Memory + Skills
Example: 3.0k + 16.7k + 0.2k + 4.9k + 1.6k = 26.4k
```

### Step 3: Update This File

Edit the values above to match your setup:

```yaml
fixed_overhead: 26400  # your calculated overhead
working_budget: 128600 # 200000 - 26400 - 45000
```

### Step 4: Adjust Thresholds (Optional)

If you want more/less conservative thresholds, recalculate:

```python
# Conservative (more safety margin)
GREEN:  working_budget * 0.30  # 38.4K
YELLOW: working_budget * 0.50  # 64K
ORANGE: working_budget * 0.70  # 89.6K
RED:    working_budget * 0.70+ # >89.6K

# Moderate (default)
GREEN:  working_budget * 0.39  # 50K
YELLOW: working_budget * 0.58  # 75K
ORANGE: working_budget * 0.78  # 100K
RED:    working_budget * 0.78+ # >100K

# Aggressive (use with caution)
GREEN:  working_budget * 0.50  # 64K
YELLOW: working_budget * 0.70  # 89.6K
ORANGE: working_budget * 0.85  # 108.8K
RED:    working_budget * 0.85+ # >108.8K
```

## Important Notes

### Auto-Compaction is Not Guaranteed

The 45K buffer triggers automatic summarization when context fills up, providing an **unpredictable safety margin**. Don't rely on it for planning - use `working_budget` as your baseline.

**Why unpredictable:**
- Compaction effectiveness varies by content type
- Some sessions compact better than others (narrative > code snippets)
- Compaction introduces latency and potential information loss
- Better to plan within budget than depend on emergency cleanup

### Overhead Variability

Your actual overhead depends on:
- **Memory files:** Large CLAUDE.md files increase overhead
- **Loaded skills:** More active skills = more overhead
- **Custom agents:** Project-specific agents add tokens
- **Model version:** System prompts may change between updates

**Recommendation:** Check `/context` periodically and update this file if overhead drifts significantly (>5K difference).

### Model-Specific Budgets

Different Claude models have different context windows:

| Model | Total Window | Typical Overhead | Typical Working Budget |
|-------|-------------|------------------|----------------------|
| Sonnet 4.5 | 200K | ~27K | ~128K |
| Opus 4.5 | 200K | ~27K | ~128K |
| Haiku 3.5 | 200K | ~27K | ~128K |

Update `total_window` if using a different model or if Anthropic changes limits.
