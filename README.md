# Claude Skills

A collection of Claude Code skills for enhanced AI-assisted development workflows.

## Available Skills

| Skill | Description |
|-------|-------------|
| [scope-check](skills/scope-check/SKILL.md) | Context window evaluation and chunking strategy tool. Prevents mid-implementation context collapse by validating document size before writing large design documents or implementation plans. |
| [benji-taylor-design-eval](skills/benji-taylor-design-eval/SKILL.md) | Structured design critique based on Benji Taylor's six principles (progressive disclosure, fluidity, simplicity/delight triad, craft, trust through clarity, accessibility). Evaluates any product or interface against these dimensions. |

## Installation

### Via Claude Code Marketplace

```bash
/plugin marketplace add dvdarkin/claude-skills
/plugin install scope-check@dvdarkin
```
Restart Claude Code to pick up the new skill.

### Manual Installation

Clone the repository and copy desired skills to your Claude skills directory:

```bash
git clone https://github.com/dvdarkin/claude-skills.git
cp -r claude-skills/skills/scope-check ~/.claude/skills/
```

## Skills Overview

### scope-check

Evaluates specification documents against Claude Code's context window constraints. Provides grounded estimations of whether a spec will fit within safe context bounds and recommends chunking strategies when it won't.

**Features:**
- Automatic activation before writing design documents or implementation plans
- Traffic light system (GREEN/YELLOW/ORANGE/RED) for context budget status
- Interactive chunking recommendations for large specs
- Research-backed expansion estimates based on language and TDD level

**Quick Start:**
```bash
# Basic check
/scope-check docs/feature-spec.md

# With explicit overrides
/scope-check docs/feature-spec.md --stack typescript --tdd comprehensive
```

**Example Output:** See [scope-check-example.md](scope-check-example.md) for a real-world example analyzing a complex AI Virtual Car Showroom spec.

**Research Foundation:**
- QSM Database: Function Point to LOC ratios (2,192 projects)
- IFPUG ISO/IEC 20926:2009: Function Point counting standards
- Liu et al. (2024): "Lost in the Middle" - context position bias
- Chroma Research: "Context Rot" study across 18 LLMs
- RULER Benchmark: Long-context model evaluation

See [scope-check documentation](skills/scope-check/SKILL.md) for full details.

### benji-taylor-design-eval

Evaluates a product or interface against six design principles drawn from Benji Taylor's work (Family wallet, Base, X). Produces a structured critique covering progressive disclosure, fluidity, simplicity/fluidity/delight triad, craft, trust through clarity, and accessibility without depth sacrifice.

**Best for:** Consumer products and prosumer tools. Less directly applicable to pure developer tooling or enterprise software (flag and adapt rather than skip).

**Quick Start:**
```
Invoke the skill when reviewing any product design, UI screenshots, or mockups.
Ask: "evaluate this design" or "what would a great designer change here"
```

## Contributing

Contributions welcome. If you have a skill that could benefit the Claude Code community, feel free to submit a PR.

Issues and PRs: https://github.com/dvdarkin/claude-skills

## License

MIT License - see [LICENSE](LICENSE) for details.

## Author

Denis Darkin (dvdarkin@gmail.com)
