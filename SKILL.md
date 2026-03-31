---
name: code-architecture
description: >-
  Plans, documents, and reviews code architecture for software projects. Produces structured
  architecture documents with Mermaid diagrams through a collaborative interview workflow.
  Triggers on architecture planning, project structuring, folder organization, system design,
  component design, data flow planning, codebase restructuring, or architecture review requests.
  Also handles evolving existing codebases by analyzing provided code, folder structures, or
  reference projects. Has deep reference patterns for Node.js/Express, Unity (C#), and C++/CMake
  but works for any technology stack. Supports greenfield projects, restructuring existing code,
  architecture reviews, and style-reference-based planning.
---

# Code Architecture Planner

Plans and documents software architecture before writing code. Produces two deliverables: a
**Mermaid diagram** visualizing the system and a **written architecture document** covering all
key decisions.

## Workflow Overview

1. **Detect Stack** — Identify technology and project mode
2. **Tech Discovery** *(if stack unclear)* — Analyze project needs, recommend technologies
3. **Interview** — Gather requirements (max 2 rounds)
4. **Generate** — Architecture document + Mermaid diagram
5. **Consistency Check** — 9-point internal audit
6. **Review & Iterate** — Targeted revisions with cascade tracking
7. **Implementation Spec** *(optional)* — AI-ready output for coding tools

## Step 1: Detect Stack

Identify the technology stack from the user's request. Look for explicit mentions of frameworks,
languages, and tools.

**Known stacks** (load the corresponding reference file):
- **Node.js / Express** → read `references/nodejs-express.md`
- **Unity (C#)** → read `references/unity.md`
- **C++ / CMake** → read `references/cpp-cmake.md`

**Unrecognized stack**: Proceed without loading a reference file. The core workflow works for any
stack. Use general knowledge of that technology instead.

A project can span multiple stacks. Load all relevant references.

**No clear stack specified**: If the user describes the project by purpose or domain without
naming specific technologies → proceed to Step 2 (Tech Discovery).

**Clear stack specified**: Skip Step 2, proceed directly to Step 3 (Interview).

If the stack is ambiguous and it matters for the architecture, ask as part of the interview.

**Determine the project mode** — this selects the interview workflow:

- **Greenfield** (building from scratch) → Step 2 or Step 3
- **Greenfield with style reference** (new project, but user provided example code they like) → Step 2 or Step 3. Analyze the reference code for patterns and conventions, then apply them to the new project.
- **Restructuring** (user has existing code that IS the project, wants it reorganized) → Step 3R
- **Architecture Review** (user wants an honest assessment of their existing project) → Step 3A

If ambiguous, ask: "Is this the project you want me to architect, an example of a style you'd
like me to follow, or would you like me to review what you have?"

## Step 2: Tech Discovery

**Triggers when** the user describes a project by purpose or domain without specifying a clear
technology stack. For example: "I want to build a personal dashboard" or "I need a tool to
manage my home lab."

**Skip when** the user has already named their technologies: "I want to build a React app with
an Express backend and PostgreSQL."

For the full Tech Discovery workflow — including the requirements analysis framework,
how to present recommendations, and handling partial stack specifications — see
[references/INTERVIEW.md](references/INTERVIEW.md#tech-discovery).

### Summary

1. **Gather project purpose.** If the user's initial message is clear, move straight to analysis.
   If vague, ask 1-2 clarifying questions about what the project does (not how to build it).
2. **Identify technical requirements.** Analyze the project description for capability needs:
   real-time updates, data persistence, authentication, external API integration, file handling,
   background tasks, etc. Be specific about *why* each need exists in this project.
3. **Recommend a stack.** Present a cohesive technology recommendation — not individual tools
   in isolation, but a stack that works together. Justify each choice by tying it to a specific
   project requirement identified in the previous step.
4. **Wait for approval.** Do not proceed to the interview until the user confirms the stack or
   requests changes. If they push back on specific choices, adjust and re-present.

After approval, load any relevant reference files for the confirmed stack, then proceed to
Step 3 (Interview).

## Step 3: Requirements Interview

**Always begin by interviewing the user.** For detailed interview guidance including handling
user-provided context, stack-specific questions, pattern choices, and interview depth rules,
see [references/INTERVIEW.md](references/INTERVIEW.md).

### Core Questions (ask what isn't already answered):

1. **Project summary** — What is this project? What problem does it solve?
2. **Scale & scope** — Solo project, small team, production-scale?
3. **Core features** — What are the 3-5 most important features or systems?
4. **Tech stack preferences** — Technologies already decided, or open to recommendations?
5. **Constraints** — Timeline, platform targets, performance, hosting limitations?

Ask these as a single concise batch. Skip anything the user's prompt already answers.
**If Tech Discovery was completed**, skip questions 1 and 4 entirely — those are already resolved.

### Interview Pacing

1. **Initial batch**: Core questions + relevant stack-specific questions. Usually 3-6 questions.
2. **One follow-up round** (if needed): Pattern choices and/or up to 3 clarifying questions.
3. **Then generate.** Make reasonable assumptions for unclear details and note them.

If the user signals impatience, skip pattern choices and generate with reasonable defaults.

## Step 3R: Restructuring Interview

For users with existing code they want reorganized. See [references/INTERVIEW.md](references/INTERVIEW.md)
for the full restructuring workflow.

**Key steps:**
1. Analyze the provided code — map structure, identify problems, identify what's working.
2. Present analysis before asking questions.
3. Ask about: pain points, growth direction, migration tolerance, constraints.
4. Generate with before/after comparisons and migration tasks.

## Step 3A: Architecture Review

For users wanting an honest assessment of their existing architecture. See
[references/INTERVIEW.md](references/INTERVIEW.md) for the full review workflow.

**Key steps:**
1. Map the current architecture in your own words.
2. Lead with what's working well (be specific).
3. Flag concerns ranked by impact with actionable recommendations.
4. Give a brief, honest verdict.
5. Deliver inline in conversation, then offer next steps.

## Step 4: Generate Architecture Document

Produce a structured markdown architecture document. For complete section-by-section guidance
including output scaling rules, see [references/DOC-TEMPLATE.md](references/DOC-TEMPLATE.md).

### Output Scaling

- **Small projects**: Collapse Sections 2, 7, 8 into a single "Notes" section. ~500-1000 words. 5-8 tasks.
- **Medium projects**: Full sections. ~1500-2500 words. 10-20 tasks.
- **Large projects**: Detailed sections, multiple diagrams. ~2500-4000 words. 15-25 tasks.

### Document Sections

1. **Project Overview** — Goals and requirements summary
2. **Scope Boundaries** — In scope, out of scope with upgrade paths
3. **Tech Stack** — Each technology with justification (incorporates Tech Discovery decisions if applicable)
4. **Folder & File Structure** — Annotated directory tree
5. **Components & Systems** — Responsibility, interface, dependencies
6. **Data Flow & Relationships** — End-to-end data movement
7. **Tradeoffs & Pitfalls** — 3-5 realistic risks with mitigations
8. **Simplicity Check** — Self-audit for over-engineering (apply findings before presenting)
9. **Implementation Tasks** — Ordered milestones with dependencies and effort estimates
10. **Appendix** — Glossary, reference links, configuration reference

Deliver as a markdown file saved to the outputs directory.

## Step 5: Generate Mermaid Diagram

### Stack-Specific Defaults

- **Node.js / Express / Web APIs** → Flowchart (graph TD or LR): request flow through layers
- **Unity / Game projects** → Class diagram (primary) + Flowchart (secondary) for game state
- **C++ / CMake** → Flowchart: module/library dependency graph
- **Restructuring** → Two diagrams: "before" and "after"
- **Unrecognized stacks** → Flowchart (graph TD) as default

### Diagram Guidelines

- Label nodes with component/system names
- Label edges with relationship nature ("HTTP request", "emits event", "reads from")
- Use subgraphs where it improves clarity
- Split diagrams with more than ~15 nodes into focused subsystem diagrams

Use the Mermaid Chart tool to render if available.

## Step 6: Consistency Check

Before presenting, run a 9-point audit. For the complete checklist, see
[references/CONSISTENCY.md](references/CONSISTENCY.md).

**Quick summary** — verify alignment between:
1. File tree ↔ component descriptions
2. Tech stack ↔ actual usage
3. Data flow ↔ components
4. Simplicity Check ↔ proposed architecture
5. Feasibility of proposed solutions
6. Diagram ↔ document
7. Scope boundaries ↔ architecture
8. Implementation tasks ↔ components
9. Appendix completeness

Fix inconsistencies silently before presenting.

## Step 7: Review & Iterate

After presenting, ask:
- Does this match your mental model?
- Any systems missing or over-engineered?
- Want to drill deeper into any specific component?

### Handling Feedback

- **Targeted revisions**, not full regeneration
- **Cascade changes** intentionally — check downstream sections
- **Advocate, don't steamroll** — make your case once, then respect the decision
- **Support "what if" exploration** — generate focused comparisons
- **Drill-down requests** — produce focused sub-architectures
- **Re-run consistency check** after every revision

## Output Format

Architecture document → markdown file saved to outputs directory.
Mermaid diagram → rendered inline using Mermaid Chart tool if available.

### Implementation Spec (Optional, AI-Ready Output)

After architecture approval, offer to generate a stripped-down spec for AI coding tools:

1. **File tree** — complete folder/file structure, copy-paste ready
2. **Interfaces** — public function signatures in actual code syntax
3. **Schemas** — database schemas, data models, API contracts
4. **Implementation task list** — ordered tasks with dependencies (no effort estimates)
5. **Constraints** — non-obvious rules the AI must follow

Omit: Project Overview, Scope Boundaries, "why" explanations, Tradeoffs, Simplicity Check, glossary.

Save as a separate markdown file (e.g., `project-name-spec.md`).

## Important Principles

- **Explain the "why"** — Justify decisions, not just list structure.
- **Don't reinvent the wheel** — Use industry-standard patterns. Novel architecture is for novel problems.
- **Simpler until proven otherwise** — Every abstraction justifies its existence at this project's scale.
- **Be a collaborator, not a yes-man** — Advocate for better approaches. Respect the user's final decision.
- **Favor patterns the user already knows** — But introduce unfamiliar patterns when meaningfully better, with sufficient explanation.
