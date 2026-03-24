# Architecture Document Template

Detailed section-by-section guidance for generating the architecture document.

## Output Scaling

Scale the document to the project's complexity:

- **Small projects** (personal tool, single-purpose API, game jam): Merge Scope Boundaries, Tradeoffs, and Simplicity Check into a single "Notes" section. Skip "Considered and passed over" in Tech Stack. Appendix only includes terms the user genuinely might not know. Total: ~500-1000 words. 5-8 implementation tasks. If the folder tree is obvious, the doc might be overkill — offer to scaffold instead.
- **Medium projects** (portfolio piece, multi-system app, semester project): Full sections with meaningful detail. Total: ~1500-2500 words. 10-20 tasks.
- **Large projects** (production app, team project, complex game): Detailed sections, multiple diagrams, thorough tradeoff analysis. Total: ~2500-4000 words. 15-25 tasks (or flag that a separate planning session is needed).

When in doubt, start leaner — it's easier to expand than trim.

## Section 1: Project Overview

One paragraph summarizing the project, its goals, and key requirements from the interview.

## Section 2: Scope Boundaries

Define what this version includes and excludes.

- **In scope**: Features and systems being architected. 3-5 items.
- **Out of scope (with upgrade path)**: Features deliberately excluded. 2-4 items. For each:
  - Why it's excluded now.
  - What changes in the architecture to support it later (the upgrade path).

Scope boundaries aren't walls — they're thresholds with a known cost to cross.

## Section 3: Tech Stack

- List each technology and **why** it was chosen.
- If recommending something the user didn't specify, explain the reasoning.
- Note alternatives considered and why they were passed over.

## Section 4: Folder & File Structure

Present as an indented tree with inline comments:

```
project-root/
├── src/
│   ├── components/    # Reusable UI components
│   ├── services/      # Business logic and API calls
│   └── utils/         # Shared helper functions
├── tests/
└── config/
```

Go 2-3 levels deep for large projects. For restructuring projects, present as before/after comparison.

## Section 5: Components & Systems

List each major component. For each:
- **Responsibility**: What it owns and does.
- **Interface**: How other systems interact with it (public methods, events, API endpoints).
- **Dependencies**: What it relies on.

## Section 6: Data Flow & Relationships

- How data moves through the system end-to-end.
- Direction of dependencies.
- Event-driven, pub/sub, or observer patterns.
- This section should directly correspond to the Mermaid diagram.

## Section 7: Tradeoffs & Pitfalls

3-5 realistic risks, tradeoffs, or pitfalls for this specific architecture. For each:
- What the risk is.
- Why it matters for this project.
- A mitigation strategy or alternative approach.

Be honest and specific — generic advice like "write tests" doesn't belong.

## Section 8: Simplicity Check

Review the proposed architecture and flag areas where a simpler approach could work. For each:
- **What was proposed**: The component, pattern, or technology.
- **Simpler alternative**: A less complex way to achieve the same goal.
- **When to upgrade**: The trigger that would justify the more complex approach.

Look for:
- Abstractions with only one implementation.
- Separate files/modules that could merge at this scale (<30 lines = probably doesn't need its own file).
- External dependencies where the standard library would suffice.
- Patterns designed for team/production scale applied to a solo project.
- Premature infrastructure before the simple version has proven insufficient.

**Critical: Apply your own findings.** If something should be simpler, simplify it in Sections 4-6 before presenting. The Simplicity Check should only show items where complexity is genuinely justified. If you can't articulate a concrete, project-specific reason to keep the complexity, simplify it.

## Section 9: Implementation Tasks

Ordered list of implementation tasks grouped into **milestones** (usable vertical slices).

For each task:
- **What to build**: One clear deliverable.
- **Depends on**: Which tasks must complete first.
- **Estimated effort**: T-shirt size (small / medium / large).

Order so the user gets a working (if minimal) system as early as possible. First milestone = smallest thing that proves the architecture works.

10-20 tasks typical. More than 25 → flag that a separate planning session is needed.

For restructuring: frame as migration tasks that keep the system functional at every step.

## Appendix

Include at the bottom of every architecture document:

- **Glossary**: Define terms the specific user might not know. Scale to audience. Don't patronize.
- **Reference links**: Official docs for libraries, APIs, or tools mentioned by name.
- **Configuration reference**: Environment variables, config files, API keys. Brief description and where to get each one.

Only include terms and references that actually appear in the document.

## Implementation Spec (AI-Ready, Optional)

Stripped-down spec for AI coding tools. Contains only actionable information:

1. **File tree** — complete, copy-paste ready.
2. **Interfaces** — every public function signature, struct/class definition in actual code syntax.
3. **Schemas** — database schemas, data models, API contracts.
4. **Implementation task list** — ordered tasks with dependencies, one-line descriptions. No effort estimates.
5. **Constraints** — non-obvious rules: "use the SQLite amalgamation," "dates as YYYY-MM-DD strings," etc.

Omit: Project Overview, Scope Boundaries, "why" explanations, Tradeoffs, Simplicity Check, glossary.

Save as separate file (e.g., `project-name-spec.md`).

Only generate if the user asks or if they plan to use AI tooling. Prompt: "Want me to generate an implementation spec you can hand to Claude Code?"
