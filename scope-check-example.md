# Scope Check Example

This is a real example of the scope-check skill analyzing a complex spec document.

---

I see there's a spec document for an "AI Virtual Car Showroom" project. Before implementing, I'm using the scope-check skill to evaluate the specification complexity against context window constraints.

● Skill(scope-check:scope-check)
  ⎿  Successfully loaded skill

● I'm running scope-check to evaluate the specification before implementation. Let me first read the spec and analyze it.

● Read(AI Virtual Car Showroom Project Vision and Scope.md)
  ⎿  Read 406 lines

● Search(pattern: "package.json")
  ⎿  Found 0 files

● Search(pattern: "pyproject.toml")
  ⎿  Found 0 files

● Search(pattern: "*.csproj")
  ⎿  Found 0 files

● Now let me complete the scope-check analysis.

```
┌─────────────────────────────────────────────────────────────────────────┐
│ SCOPE CHECK:    RED - Must decompose before implementation              │
├─────────────────────────────────────────────────────────────────────────┤
│ Spec: AI Virtual Car Showroom (~5.5K tokens, 405 lines)                 │
│ Stack: Python (FastAPI) + TypeScript (React) + PyTorch - mixed stack    │
│ TDD Level: Standard (assumed, no existing test infrastructure)          │
│                                                                         │
│ Estimated output: 140-170K tokens                                       │
│ Working budget: 128K (200K - 27K overhead - 45K buffer)                 │
│ Usage: ~155K (121% of working budget)                                   │
│                                                                         │
│ → MUST decompose into smaller chunks                                    │
│ → High coupling density - requires careful boundary selection           │
└─────────────────────────────────────────────────────────────────────────┘
```

**DETAILED ANALYSIS**

Complexity Metrics:
- NL-CC: HIGH (multi-phase pipeline, multiple AI models, full-stack)
- Coupling: HIGH (AI ↔ Backend ↔ Frontend ↔ Infrastructure tightly integrated)
- Function points: ~40+ UFP (significant application)

Expansion Calculation:
- Base: 5.5K × 1.3 (mixed stack) × 15 = ~107K tokens
- With TDD (1.3×): ~139K tokens
- Conservative (+20%): ~167K tokens

---

**CHUNKING ANALYSIS**

This spec describes a full-stack AI-powered SaaS application. I've identified 4 natural boundaries with relatively lower coupling between them:

| Chunk | Scope | Estimated Output | Dependencies |
|-------|-------|------------------|--------------|
| 1. AI/ML Pipeline | Segmentation, detail enhancement, background replacement, lighting | ~50K tokens | None (core) |
| 2. Backend API | FastAPI, endpoints, queue, storage, webhooks | ~35K tokens | Chunk 1 interfaces |
| 3. Frontend UI | React, upload, config, status, download | ~30K tokens | Chunk 2 API |
| 4. Infrastructure | AWS, GPU, S3, CDN, Redis | ~25K tokens | All (deployment) |

**Recommended Implementation Order:**
1. Chunk 1 (AI/ML) → Core value, defines interfaces
2. Chunk 2 (Backend) → Wraps AI pipeline in API
3. Chunk 3 (Frontend) → Consumes API
4. Chunk 4 (Infrastructure) → Deploys everything

---

Your spec is RED status. Options:

- **[A]** Split into 4 chunks as proposed above and implement sequentially
- **[B]** Suggest different boundaries (tell me your preference)
- **[C]** Focus on MVP only (Phase 1 proof-of-concept with minimal scope)
- **[D]** Override with explicit acknowledgment (not recommended)

Which approach would you like to take?
