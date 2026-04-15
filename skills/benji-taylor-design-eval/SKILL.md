---
name: benji-taylor-design-eval
description: >
  Use when reviewing, critiquing, or auditing a product's design quality, UX, or interface decisions.
  Trigger on "is this well designed", "what's wrong with this UI", "how would a great designer evaluate this",
  or when the user wants principled design feedback on their own project.
---

# Benji Taylor Design Evaluation

## Overview

Structured design critique based on the principles Benji Taylor has articulated and demonstrated
through his work on the Family self-custody wallet, Base (Coinbase L2), and X. Surfaces real
tensions in a design and where it resolves or fails them -- not a scorecard with arbitrary numbers.

## When to Use

- User asks to critique, review, audit, or score a product's design
- User asks "is this well designed" or "how would a great designer look at this"
- User wants to improve the design quality of their own product or feature
- Reviewing screenshots or mockups for design principles compliance

**When NOT to use:**
- Pure developer tooling (delight and fluidity have different meanings in CLI/API contexts)
- Enterprise software (progressive disclosure is often inverted by necessity)
- Flag and adapt if the product falls in these categories rather than skipping entirely

## Quick Reference

| Dimension | Core Question |
|---|---|
| Progressive Disclosure | Does complexity appear only when needed? |
| Fluidity | Does motion communicate state changes? |
| Simplicity/Fluidity/Delight | Are all three present, not just one? |
| Craft | Are edge cases designed with equal care? |
| Trust Through Clarity | Are high-stakes actions unambiguous? |
| Accessibility Without Depth | Can novices and experts coexist? |

---

## The Six Evaluation Dimensions

### 1. Progressive Disclosure

**The principle**: Fundamentals at your fingertips. Everything else appears as it becomes relevant.
Never dump all features at once.

**What to look for**:
- Does the first screen show only what a new user needs?
- Are advanced features gated behind discovery (not hidden, but not foregrounded)?
- Is there a coherent mental model for what is always visible vs. what appears contextually?

**Failure modes**:
- Settings exposed before the user has done anything
- Feature parity between power and novice users baked into primary navigation
- Long lists of options when a single default would serve 80% of users

---

### 2. Fluidity as Semantic Communication

**The principle**: Motion is not decoration. It is a channel for communicating state changes,
transitions, and the weight of an action.

**What to look for**:
- Do animations signal what just happened or what is about to happen?
- Are irreversible or high-stakes actions marked by a perceptible transition (not just a dialog)?
- Is there a consistent motion language, or is animation applied randomly?

**Failure modes**:
- Instant state changes with no transition for consequential actions
- Decorative animations that do not communicate anything about the system state
- Inconsistent timing -- some things animate, others snap

---

### 3. The Simplicity / Fluidity / Delight Triad

Each pillar resolves a specific failure mode, and all three must be present:

| Pillar | Resolves | Test question |
|---|---|---|
| Simplicity | Cognitive overload | Can a first-time user orient themselves in under 10 seconds? |
| Fluidity | Experience fragmentation | Does the product feel like one continuous thing or a set of screens? |
| Delight | Emotional disengagement | Is there at least one moment where the product surprises the user pleasantly? |

**What to look for**:
- Products often have one or two of these but sacrifice the third
- Simplicity without fluidity = sterile
- Fluidity without simplicity = overwhelming
- Delight without either = gimmicky

---

### 4. Craft as Competitive Positioning

**The principle**: Polish is not a finishing coat. It is a deliberate choice to slow down and do fewer
things better, treated as a strategic differentiator rather than a luxury.

**What to look for**:
- Are typography, spacing, and color decisions consistent or ad hoc?
- Are edge cases (empty states, loading states, errors) designed with the same care as the happy path?
- Does the product feel like it was built by one mind or assembled from parts?

**Failure modes**:
- Empty states that are just blank with a spinner
- Error messages in default OS styling
- Inconsistent component behaviour across screens (button that works one way here, differently there)

---

### 5. Trust Through Clarity in High-Stakes Interactions

**The principle**: When an action is irreversible, consequential, or involves real-world assets (money,
data, identity), the interface must make the stakes unambiguous -- before, during, and after.

This is Taylor's most hard-won principle from crypto product design. It generalises to any interaction
where the user might regret an action.

**What to look for**:
- Is the user clear on what will happen before they tap Confirm?
- Are destructive actions visually distinguishable from non-destructive ones?
- Does confirmation feel like a ritual (i.e., a moment of attention), not just a speed bump?

**Failure modes**:
- Confirm dialogs with identical styling to regular dialogs
- Action labels that are vague ("Continue" instead of "Delete Account")
- No post-action feedback -- the user cannot tell if the action completed

---

### 6. Accessibility Without Depth Sacrifice

**The principle**: Making a product approachable for newcomers must not strip power from experienced
users. The tension is resolved through layering, not simplification.

**What to look for**:
- Is the entry path clear for someone who has never used this category of product?
- Can a power user access advanced functionality without fighting through the simplified surface?
- Is the information architecture coherent for both audiences simultaneously?

**Failure modes**:
- Tutorial overlays that disappear and never return (no re-discovery path)
- Advanced features buried in settings rather than surfaced contextually
- Onboarding that oversimplifies to the point of being misleading about product depth

---

## Output Format

Structure your evaluation as follows:

### [Product Name] - Design Evaluation

**Context**: [One sentence on what the product is and who it is for]

For each of the six dimensions, write:
- **Verdict**: Passes / Partially Passes / Fails
- **Observation**: What you see in the product (specific, not generic)
- **Tension**: The underlying design tension this reveals
- **Recommendation**: One concrete change that would move it toward the principle (only if Fails or Partially Passes)

End with:

**Strongest dimension**: [which one and why]
**Most critical gap**: [which one and why -- this is where design investment would have highest leverage]
**Overall character**: [one paragraph on the product's design identity -- what kind of designer built this, what values are implicit in the choices]

---

## How to Run This Evaluation

**Input the evaluator needs** (ask for what is missing):
- Product description or URL
- Target user (novice / power user / both)
- Key interactions to evaluate (if known)
- Screenshots or description of the interface
- Any known design constraints (platform, team size, timeline)

**If screenshots are not provided**: Evaluate based on the product description and any publicly
available information. Be explicit about what is inferred vs. observed.
