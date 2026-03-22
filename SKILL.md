---
name: code-architecture
description: >
  Plan, document, and review code architecture for software projects. Trigger whenever the user
  wants to architect, plan, design, structure, or restructure a codebase — web apps, games, APIs,
  CLI tools, or any software. Trigger on: "architect," "plan the structure," "how should I organize,"
  "system design," "project structure," "folder structure," "component design," "data flow,"
  "separation of concerns," "restructure my project," "refactor this layout," or "here's my current
  code, help me plan the next phase." Also handles evolving existing codebases — analyzes provided
  code, folder structures, or reference projects and builds on them. Covers web dev (Node.js,
  Express, REST, databases), game dev (Unity, C#, ScriptableObjects), and C++/CMake (CLI tools,
  libraries, systems programming). Loads domain-specific reference files based on context.
---

# Code Architecture Planner

This skill helps the user plan and document software architecture before writing code. It produces
two deliverables: a **Mermaid diagram** visualizing the system and a **written architecture document**
covering all key decisions.

## Workflow

### Step 1: Detect Domain

Determine the project's domain from the user's request. Look for cues:

- **Web**: mentions of APIs, endpoints, databases, Express, Node, React, auth, REST, middleware, deployment
- **Game**: mentions of Unity, Unreal, C#, game systems, ECS, player mechanics, scenes, prefabs, ScriptableObjects
- **C++ / CMake**: mentions of C++, CMake, CLI tools, systems programming, header/source files, libraries, FetchContent, vcpkg

A project can span multiple domains (e.g., a game with a Node.js backend). Load all relevant references in that case.

If ambiguous, ask the user.

Once determined, read the appropriate reference file(s) for domain-specific patterns:
- Web projects → read `references/webdev.md`
- Game projects → read `references/gamedev.md`
- C++ / CMake projects → read `references/cpp-cmake.md`

### Step 2: Requirements Interview

**Always begin by interviewing the user.** Ask focused questions to understand scope before generating anything. Tailor the interview to the detected domain.

#### Handling User-Provided Context

The user may provide existing context alongside their request — code snippets, folder structures, file uploads, a previous project's layout, or examples of patterns they like. When this happens:

1. **Analyze first, don't ignore.** Before asking interview questions, explicitly acknowledge what was provided. Call out specific patterns, naming conventions, and structural decisions you observe in their code or examples.
2. **Weigh it heavily, but don't treat it as law.** User-provided context reveals how they think and what they're comfortable with. That's valuable signal — use it to inform the architecture. But if an industry-standard pattern or simpler approach would serve the project better, say so. Explain *why* the alternative is stronger and let the user decide. The goal is a great architecture, not a copy of what they already have.
3. **Identify what to keep, what to evolve, and what to rethink.** Be specific: "Your route/controller split is solid and I'd keep it. Your data layer could be simplified from three model files to one at this scale — here's why." This makes the skill a collaborator, not a rubber stamp.
4. **Adapt the interview.** Skip questions that the provided context already answers. If they pasted an Express app with PostgreSQL, don't ask about tech stack preferences — confirm what you see and ask about what's missing.
5. **If the user uploads files or references a large codebase**, focus on the structural patterns (folder layout, module boundaries, naming conventions, dependency direction) rather than line-by-line code analysis. The goal is architecture, not code review.

#### Core Interview Questions

Core questions (always ask what isn't already answered by provided context):
1. **Project summary** — What is this project? What problem does it solve or what experience does it create?
2. **Scale & scope** — How big is this? Solo project, small team, production-scale? How many users/players?
3. **Core features** — What are the 3-5 most important features or systems?
4. **Tech stack preferences** — Any technologies already decided on, or open to recommendations?
5. **Constraints** — Timeline, platform targets, performance requirements, hosting limitations?

Ask these as a single concise batch — don't drip-feed one question at a time.

If the user's initial message already answers some of these clearly, acknowledge what you know and only ask about the gaps.

#### Offering choices with a clarification escape hatch

When presenting the user with multiple-choice options (e.g., "how modular should integrations be?"), **always include a clarification option** alongside the substantive choices. The user may not understand the implications of an option, or the question itself may be ambiguous. A choice like "I'd like more detail on this before deciding" lets them get clarification on that specific question without being forced to pick blindly or stall the entire interview.

If the user asks for clarification, explain the options with concrete examples of what each would mean for *their specific project* — not generic definitions. Then re-present the choices. Continue with any remaining questions that weren't blocked by the clarification.

### Step 3: Generate Architecture Document

After the interview, produce a **structured markdown architecture document** with these sections in order:

#### Section 1: Project Overview
- One paragraph summarizing the project, its goals, and key requirements gathered from the interview.

#### Section 2: Scope Boundaries
- Explicitly define what this version of the project includes and what it does not.
- **In scope**: List the features and systems being architected. These are the commitments for this version.
- **Out of scope (with upgrade path)**: List features deliberately excluded. For each, briefly note:
  - Why it's excluded now (not needed yet, adds complexity, unclear requirements).
  - What changes in the architecture to support it later (the "upgrade path"). This is the key differentiator — scope boundaries aren't walls, they're thresholds. The user should know the cost of crossing each one.
- This section prevents scope creep during development while acknowledging that scope evolves. If a feature is out of scope, the user shouldn't feel guilty about skipping it — and they should know exactly what it takes to add it when the time comes.
- Keep it concise. 3-5 in-scope items, 2-4 out-of-scope items. If the project is small enough that scope is obvious, this section can be brief.

#### Section 3: Tech Stack
- List each technology and **why** it was chosen.
- If recommending something the user didn't specify, explain the reasoning.
- Note alternatives considered and why they were passed over.

#### Section 4: Folder & File Structure
- Present as an indented tree (using code block formatting).
- Include brief inline comments explaining what each directory/file is responsible for.
- Only go as deep as is useful — top 2-3 levels for large projects.

```
project-root/
├── src/
│   ├── components/    # Reusable UI components
│   ├── services/      # Business logic and API calls
│   └── utils/         # Shared helper functions
├── tests/
└── config/
```

#### Section 5: Components & Systems
- List each major component or system.
- For each, describe:
  - **Responsibility**: What it owns and does.
  - **Interface**: How other systems interact with it (public methods, events, API endpoints).
  - **Dependencies**: What it relies on.

#### Section 6: Data Flow & Relationships
- Describe how data moves through the system end-to-end.
- Identify the direction of dependencies.
- Call out any event-driven, pub/sub, or observer patterns.
- This section should directly correspond to the Mermaid diagram (Step 4).

#### Section 7: Tradeoffs & Pitfalls
- List 3-5 realistic risks, tradeoffs, or pitfalls for this specific architecture.
- For each, briefly explain:
  - What the risk is.
  - Why it matters for this project.
  - A mitigation strategy or alternative approach.
- Be honest and specific — generic advice like "write tests" doesn't belong here.

#### Section 8: Simplicity Check
- Review the entire proposed architecture and actively flag areas where a simpler approach could work.
- For each flagged area, present:
  - **What was proposed**: The component, pattern, or technology in the architecture.
  - **Simpler alternative**: A less complex way to achieve the same goal.
  - **When to upgrade**: The specific trigger or threshold that would justify adopting the more complex approach.
- This section is an honest self-audit. Look for patterns like:
  - Abstractions with only one implementation (do you really need an interface for that?).
  - Separate services or modules that could be a single file at this scale.
  - External dependencies where the standard library would suffice.
  - Patterns designed for team/production scale applied to a solo/personal project.
  - Premature infrastructure (caching layers, message queues, microservices) before the simple version has proven insufficient.
- The goal is NOT to strip the architecture down — it's to give the user clear, honest signals about where they can start simpler and scale up later, versus where the proposed complexity is justified from day one.

#### Section 9: Implementation Tasks
- Break the architecture into a concrete, ordered list of implementation tasks. This is the "what do I build first?" section — it bridges the gap between the architecture plan and actually writing code.
- Group tasks into **milestones** that represent a usable vertical slice of the project. Each milestone should produce something that works end-to-end, even if incomplete. For example: Milestone 1 might be "auth + one static page behind login," not "build the entire database layer."
- For each task:
  - **What to build**: One clear deliverable (e.g., "Set up Express server with session auth middleware").
  - **Depends on**: Which tasks must be completed first. This defines the build order.
  - **Estimated effort**: Rough T-shirt size (small / medium / large) so the user can plan their time. Don't fake precision — an afternoon vs. a weekend vs. a week is useful; "4.5 hours" is not.
- Order tasks so the user gets a working (if minimal) system as early as possible. The first milestone should be the smallest thing that proves the architecture works — auth + one endpoint + one widget rendering data, not "set up the entire plugin system."
- Keep it practical. 10-20 tasks is typical. More than 25 means the project scope is large enough that the task list should be a separate planning session, not embedded in the architecture doc.
- This section is NOT a project management tool. It doesn't track status, assignees, or due dates. It answers one question: "In what order should I build this?"

#### Appendix
- Include at the bottom of every architecture document. This keeps the main sections clean while making the doc self-contained.
- **Glossary**: Define every acronym and technical term used in the document that isn't common English. If you wrote "FK," "TTL," "FIFO," "RLS," "CRUD," "MFA," "ORM," or any domain-specific term, define it here. Assume the reader is technically literate but may not share your exact vocabulary. Format as a simple term → definition list.
- **Reference links**: If the architecture references specific libraries, APIs, or tools, link to their official documentation. The reader shouldn't have to search for "what is CLI11" — give them a URL.
- **Configuration reference**: If the architecture involves environment variables, config files, or API keys that need to be set up, list them here with a brief description of what each one does and where to get it. For example: `DUO_IKEY` — Duo integration key, found in the Duo Admin Panel under Applications.
- Keep it practical. Only include terms and references that actually appear in the document. Don't pad it with generic definitions.

### Step 4: Generate Mermaid Diagram

After (or alongside) the written doc, produce a **Mermaid diagram** that visualizes the architecture. Choose the diagram type that best fits:

- **Flowchart (graph TD/LR)** — Good for showing system components and their relationships. Use this as the default.
- **Sequence diagram** — Good for showing request/response flows or time-ordered interactions between systems.
- **Class diagram** — Good for showing inheritance hierarchies or data models in game dev.

Guidelines for the diagram:
- Label nodes clearly with component/system names.
- Label edges with the nature of the relationship (e.g., "HTTP request", "emits event", "reads from").
- Group related components using subgraphs where it improves clarity.
- Keep it readable — if the system is complex, create multiple focused diagrams rather than one overwhelming one.

Use the Mermaid Chart tool to render the diagram if available. Otherwise, output the Mermaid code in a fenced code block.

### Step 5: Consistency check

Before presenting the architecture to the user, do a final pass across the entire output to catch internal contradictions and inaccuracies. This step exists because the architecture document has many interdependent sections — a decision in one section has downstream effects on others, and it's easy for stale references, phantom files, or contradictory claims to survive between iterations.

Check for:

1. **File tree vs. component descriptions**: Every file mentioned in Section 4 (Folder Structure) must correspond to a component in Section 5, and vice versa. No phantom files (listed in the tree but never described), no ghost components (described in Section 5 but absent from the tree).
2. **Tech stack vs. actual usage**: Every technology in Section 3 must actually appear in the architecture. If a technology was considered and rejected, it should not show up in the file tree, components, or data flow. Conversely, if a library appears in the implementation details, it should be listed in the tech stack with a rationale.
3. **Data flow vs. components**: The flows described in Section 6 must only reference components and interfaces that exist in Section 5. If a flow mentions "the WebSocket server routes the message," there must be a WebSocket server component.
4. **Simplicity Check vs. proposed architecture**: Section 8 flags areas that could be simpler. Verify the *proposed* architecture (Sections 4-6) actually reflects the simpler approach where the Simplicity Check recommends it. If the Simplicity Check says "don't create a separate filter class" but Section 4 still lists `NoteFilter.h`, that's a contradiction.
5. **Feasibility of proposed solutions**: Verify that what's being recommended is actually possible with the stated technologies. Check for things like: APIs that don't exist or work differently than described, library features that require a different version, patterns that don't apply to the stated framework, or configuration that the stated tool doesn't support.
6. **Diagram vs. document**: Every node in the Mermaid diagram should correspond to a component in Sections 4-5. Every arrow should correspond to a relationship described in Section 6. No orphan nodes, no undocumented connections.
7. **Scope boundaries vs. architecture**: Features listed as "out of scope" in Section 2 should not appear in the file tree, components, or task list. Features listed as "in scope" must be covered by at least one component and at least one implementation task.
8. **Implementation tasks vs. components**: Every component in Section 5 should be created by at least one task in Section 9. Every task in Section 9 should reference components that exist in the architecture. Task dependencies should form a valid ordering — no circular dependencies, no task depending on something that comes after it.
9. **Appendix completeness**: Scan the entire document for acronyms and technical terms. Every one must appear in the glossary. Every library or tool mentioned by name should have a reference link. Every environment variable or config value referenced in the architecture should appear in the configuration reference.

If inconsistencies are found, fix them silently before presenting to the user. Don't call out the self-correction — just deliver clean output.

### Step 6: Review & Iterate

After presenting both deliverables, ask the user:
- Does this match your mental model?
- Any systems missing or over-engineered?
- Want to drill deeper into any specific component?

#### Handling Feedback

The user will likely want changes. Handle feedback efficiently:

1. **Targeted revisions, not full regeneration.** If the user says "the folder structure is wrong but the rest is good," only revise Section 4 (and any downstream sections it affects, like the diagram). Don't regenerate the entire doc — that wastes time and risks losing parts the user already approved.

2. **Cascade changes intentionally.** Some changes ripple. If the user restructures the folder layout, check whether Section 5 (Components), Section 6 (Data Flow), and the Mermaid diagram still match. Call out what else changed and why: "I updated the folder structure as you asked. This also affected the data flow section because the dependency direction shifted."

3. **Track what's approved.** Mentally note which sections the user has signed off on. Don't re-propose changes to approved sections unless the user's new feedback creates a contradiction.

4. **Advocate, don't steamroll.** If the user requests a pattern you think is suboptimal, don't silently comply and don't lecture. Make your case once, clearly: state what you'd recommend instead, *why* it's stronger for this specific project, and what the user's approach risks. Then respect their decision. If they insist after hearing the tradeoff, implement it — they've made an informed choice. The skill's job is to make sure the user has the information to decide well, not to override them.

5. **Support "what if" exploration.** The user may want to compare approaches: "What would it look like if I used SQLite instead of PostgreSQL?" Generate a focused comparison of the affected sections (Tech Stack, Folder Structure, relevant Components) rather than a whole new document.

6. **Drill-down requests.** If the user asks to "go deeper on the auth system" or "expand the crafting system," produce a focused sub-architecture for that component: its own internal structure, data model, state management, and edge cases. This is a mini version of the full doc, scoped to one system.

7. **Update the diagram.** Whenever the written architecture changes structurally (new components, removed components, changed relationships), regenerate the Mermaid diagram to match. The doc and diagram must always agree.

8. **Re-run the consistency check after revisions.** Every targeted revision can introduce contradictions — a removed component still mentioned in the data flow, a renamed file still referenced in the old form, a rejected technology lingering in the tech stack. Run Step 5's checklist against the revised sections before presenting the update.

## Output Format

The architecture document should be delivered as a **markdown file** saved to the outputs directory so the user can download it. The Mermaid diagram should be rendered inline using the Mermaid Chart tool if available.

## Important Principles

- **Explain the "why"** — Don't just list structure; justify decisions. The user wants to understand the reasoning, not just receive a blueprint.
- **Don't reinvent the wheel** — Lean on industry-standard patterns and proven approaches. If Express middleware, Unity ScriptableObjects, or CMake's target-based model already solve the problem well, use them. Novel architecture should be reserved for novel problems.
- **Simpler until proven otherwise** — Default to the lightest-weight solution that meets the requirements. The Simplicity Check (Section 8) exists to enforce this. Every layer of abstraction, every additional dependency, and every separated module needs to justify its existence for *this* project at *this* scale — not for a hypothetical future version.
- **Be a collaborator, not a yes-man** — The user's input is the most important signal, but the skill's job is to produce a *great* architecture, not just a familiar one. When a standard approach is stronger than what the user proposed, advocate for it — explain the benefit, acknowledge the tradeoff, and let them decide. When the user's approach is sound, say so and build on it.
- **Favor patterns the user already knows** — If you know the user's tech stack or experience level, lean into familiar patterns rather than introducing unnecessary new concepts. But if an unfamiliar pattern is meaningfully better for the project, introduce it with enough explanation for the user to evaluate it.
