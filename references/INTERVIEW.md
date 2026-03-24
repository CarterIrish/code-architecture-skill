# Interview Reference

Detailed guidance for the interview phase of the architecture workflow. This file covers
all project modes: greenfield, restructuring, and architecture review.

## Handling User-Provided Context

When the user provides existing code, folder structures, file uploads, or examples:

1. **Analyze first, don't ignore.** Acknowledge what was provided. Call out specific patterns, naming conventions, and structural decisions observed.
2. **Weigh it heavily, but don't treat it as law.** Use it to inform the architecture. If a standard pattern or simpler approach would serve better, say so and explain why.
3. **Identify what to keep, what to evolve, and what to rethink.** Be specific: "Your route/controller split is solid and I'd keep it. Your data layer could be simplified at this scale."
4. **Adapt the interview.** Skip questions the provided context already answers.
5. **For large codebases**, focus on structural patterns (folder layout, module boundaries, naming conventions, dependency direction) rather than line-by-line analysis.

## Stack-Specific Interview Questions

Add these follow-ups based on the detected stack. Only ask what's relevant and unanswered.

**Unity / Game projects:**
- Target platform (PC, mobile, web, console)?
- Art style or asset approach (pixel art, 3D, procedural, purchased assets)?
- Target scope — game jam, semester project, portfolio piece, commercial release?
- Any existing art, music, or design docs?
- Multiplayer or single-player?

**Node.js / Express projects:**
- Will there be a frontend consuming this API, or is it standalone?
- Expected traffic — personal use, small group, public-facing?
- Deployment target — home lab, cloud (Heroku, Railway, AWS), containerized?
- Any existing data or databases to integrate with?

**C++ / CMake projects:**
- Is this a library, an application, or both?
- Target platforms and compilers (GCC, Clang, MSVC, cross-platform)?
- Any required third-party dependencies already known?

For unrecognized stacks, use the core questions only and let the user's answers guide follow-ups.

## Pattern Choices

Each stack reference file contains a **Pattern Choices** section with 2-3 options for key architectural decisions. These represent decisions where multiple approaches are equally valid.

**When to present:**
- After the initial interview batch, once you know the project's scale and features.
- Present only choices relevant to this project. 1-3 per project maximum.
- Skip choices the user's provided context already answers.

**How to present:**
- Use short labels (A, B, C) with a one-sentence description.
- Always include a clarification escape hatch ("I'd like more detail on this before deciding").
- Don't dump code examples during the interview.

**If the user doesn't care:** Pick the simplest option that fits the project's scale. Note the choice and reasoning in the architecture doc.

## Handling User Unfamiliarity

When the user indicates they haven't used a system, pattern, or technology before:

1. **Prefer accessible patterns over optimal ones.** The "best" architecture is the one the user can actually build.
2. **Flag the learning curve in Tradeoffs (Section 7).** Be specific about what to expect.
3. **Sequence implementation tasks to build skills progressively.** Don't stack unfamiliar concepts into Milestone 1.
4. **Include reference links in the Appendix** — one or two high-quality links per unfamiliar concept.

## Offering Choices with a Clarification Escape Hatch

When presenting multiple-choice options, always include a clarification option. The user may not understand implications.

If the user asks for clarification, explain options with concrete examples of what each would mean for *their specific project*. Then re-present the choices.

---

## Restructuring Interview (Step 2R)

For users with existing code they want to reorganize.

### Analysis Phase

Before asking questions, analyze what the user provided:

1. **Map the current structure.** Identify files, responsibilities, and relationships. For monoliths, identify logical sections. For multi-file projects, map the dependency graph.
2. **Identify structural problems.** Name specific issues: "server.js handles routing, auth, DB queries, and config in one 400-line file." Be concrete.
3. **Identify what's working.** Call out good patterns. This builds trust and prevents unnecessary churn.

Present this analysis before asking questions.

### Restructuring Questions

Ask only what the analysis didn't answer:
1. **Pain points** — What specifically feels messy or hard to work with?
2. **Growth direction** — Adding features soon, or stable code you just want cleaner?
3. **Migration tolerance** — Gradual refactor or clean restructure?
4. **Constraints** — Anything that can't change?

One batch, one follow-up max, then generate.

### Restructuring Output Adjustments

Modify the standard doc template for restructuring:

- **Section 1 (Project Overview)**: Focus on what the project already does and what restructuring aims to achieve.
- **Section 2 (Scope Boundaries)**: Frame as "what's changing vs. what's staying."
- **Section 4 (Folder Structure)**: Present as a **before/after comparison** with annotations.
- **Section 9 (Implementation Tasks)**: Frame as **migration tasks**. Each task keeps the system functional. Include "verify nothing broke" after each step.

Mermaid diagram shows the **proposed** architecture. Optionally produce a "before" diagram too.

---

## Architecture Review (Step 2A)

For users wanting an honest assessment of their existing architecture.

### Deep Analysis

Read everything provided — code, folder structure, config files, package.json, README. Then produce:

1. **Architecture map.** Describe the current architecture in your own words: layers, data flow, boundaries. For large projects, focus on top-level structure and one representative vertical slice.

2. **What's working well.** Lead with strengths. Be specific — "Your separation of routes and controllers is clean" not "looks good overall."

3. **Concerns.** Ranked by impact. For each:
   - **What the issue is** — concrete, not vague.
   - **Why it matters for this project** — tied to specific scale, goals, or growth direction.
   - **What you'd recommend** — a specific, actionable change.

4. **Recommendations.** Broader "nice-to-have" suggestions that aren't problems.

5. **Verdict.** Brief, honest summary. Sound architecture? Over-engineered? Under-engineered?

### Review Interview (Minimal)

Ask only:
1. **Context** (if not obvious) — Production? In development? Side project?
2. **Focus** (optional) — Specific area of concern, or general review?

One round max. If just "review this" with no other context, proceed — the code provides enough.

### Review Output Format

Deliver **inline in the conversation**, not as a separate file. Use a Mermaid diagram if it helps illustrate concerns.

After the review, offer next steps:
- "Want me to restructure any of these?" (→ Step 2R)
- "Want me to architect a plan for adding [feature]?" (→ Step 2)
- "Want me to go deeper on any of these concerns?"
