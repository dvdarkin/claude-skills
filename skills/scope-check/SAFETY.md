# Context Pressure Safety Recommendations

Include these recommendations when scope-check returns YELLOW or higher status. Based on empirically validated research from production systems.

## The Core Problem

AI agents degrade predictably as context fills:
- **Effective context is ~50-65% of advertised** - a 200K window degrades significantly beyond 130K
- **"Exam rushing" behavior** - agents skip verification, take shortcuts, miss instructions
- **U-shaped attention** - middle content gets 10-25% less attention than start/end
- **Non-linear collapse** - quality often drops suddenly at thresholds, not gradually

**Key insight**: Prevention beats detection. Proactive context management dramatically outperforms reactive approaches.

## During Implementation

### Prompt Structure (Validated +30% improvement)

**Dual instruction placement:**
Place critical instructions at BOTH the beginning AND end of long contexts.

```markdown
# System Instructions (START)
You are implementing [feature]. Key requirements:
- [Critical requirement 1]
- [Critical requirement 2]

[... bulk content: specs, code context, etc. ...]

# Reminder (END)
Remember the key requirements:
- [Critical requirement 1]
- [Critical requirement 2]

Now implement [specific task].
```

**Query positioning:**
Always place the specific ask at the very END, after all context documents.

**Format choice:**
- Use XML tags for document structure (performs well)
- Avoid JSON for document collections (performs poorly in long context)
- Use clear transition phrases: "Based on the information above..."

### Context Management Triggers

**At 80% utilization (~160K tokens):**
- Strip tool outputs already saved to files
- Replace verbose outputs with "Output saved to [path]"
- Summarize older conversation turns (keep last 3-5 raw)

**At 90% utilization (~180K tokens):**
- Force compaction before continuing
- Consider checkpoint and restart with summary

**At 95% utilization (~190K tokens):**
- Stop and restart with fresh context
- Preserve: architectural decisions, unresolved bugs, critical implementation details

### Compaction Hierarchy (in order of preference)

1. **Strip retrievable info** (lossless): Remove file contents that exist on disk and can be re-read
2. **Observation masking**: Clear verbose tool outputs after processing (50%+ cost reduction)
3. **Turn summarization** (lossy): Compress older turns, preserve recent 3-5 for "rhythm"
4. **Full trajectory summary**: Last resort, loses nuance

### Checkpoint Often

- Commit working code frequently
- Each commit = potential clean restart point
- If degradation occurs, can resume from last good commit with fresh context

## Degradation Detection

### Behavioral Signals (watch for these)

- Increased refusals ("I cannot do that")
- "I cannot find the information" when data clearly exists
- Skipped verification steps
- Shorter, less detailed responses
- Increased latency
- Repeating previous responses

### When Detected

1. **Don't push through** - degradation compounds
2. **Compact immediately** - strip tool outputs, summarize older turns
3. **If persistent** - checkpoint and restart with:
   - Summary of progress so far
   - Current task state
   - Key decisions made
   - Last 3-5 files touched

## Multi-Agent Safety

When using subagents for chunked execution:

### Subagent Best Practices

- Each subagent starts with **clean context** (no accumulated history)
- Pass only **essential shared context** (types, constants, interfaces)
- Return **condensed summaries** (1-2K tokens max)
- Include: what was built, key decisions, interface contracts
- Exclude: implementation details retrievable from files

### Coordinator Protection

- Monitor coordinator's accumulated context
- If >25% budget used by summaries: consider sub-coordinators
- "Share memory by communicating; don't communicate by sharing memory"

### Shared Context Header Template

```markdown
## Shared Context for [Chunk N]

### Types (from previous chunks)
- User: { id: UUID, email: string, ... }
- AuthToken: { token: string, expires: Date }

### Constants
- API_BASE: "/api/v1"
- AUTH_ROUTES: ["/login", "/logout", "/refresh"]

### Key Decisions
- Using JWT for auth (not sessions)
- All IDs are UUIDs (not auto-increment)

### Dependencies
- Chunk N depends on: [list previous chunks]
- Implements interfaces from: [list]
```

## Validated Agentic Instructions

These three instructions showed **+20% improvement on SWE-bench** when used together:

### 1. Persistence Instruction
> "You are an agent - please keep going until the user's query is completely resolved before ending your turn and yielding back to the user. Only terminate when you are sure the problem is solved."

### 2. Anti-Guessing Instruction
> "If you are not sure about file content or codebase structure pertaining to the user's request, use your tools to read files and gather the relevant information - do NOT guess or make up an answer."

### 3. Deliberation Instruction
> "You MUST plan extensively before each function call, and reflect extensively on the outcomes of the previous function calls. DO NOT do this entire process by making function calls only, as this can impair your ability to solve the problem."

## Output Reservation

Always reserve buffer for output generation:

```
safe_input_tokens = (context_window × 0.8) - (expected_output × 1.2)
```

For 200K window expecting 20K output:
```
safe_input = (200K × 0.8) - (20K × 1.2) = 160K - 24K = 136K
```

## References

- Liu et al. (2024): "Lost in the Middle" - position bias documentation
- Chroma "Context Rot": Degradation patterns across 18 LLMs
- RULER Benchmark: Real context capacity vs. advertised
- Anthropic: "Effective context engineering for AI agents"
- OpenAI GPT-4.1 Prompting Guide: Long context best practices
- LangGraph/CrewAI/AutoGPT: Production memory management patterns
