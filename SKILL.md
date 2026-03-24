---
name: code-architecture
description: >
  Plan, document, and review code architecture for software projects. Trigger whenever the user
  wants to architect, plan, design, structure, or restructure a codebase — web apps, games, APIs,
  CLI tools, or any software. Trigger on: "architect," "plan the structure," "how should I organize,"
  "system design," "project structure," "folder structure," "component design," "data flow,"
  "separation of concerns," "restructure my project," "refactor this layout," or "here's my current
  code, help me plan the next phase." Also handles evolving existing codebases — analyzes provided
  code, folder structures, or reference projects and builds on them rather than starting from scratch.
  Has deep reference patterns for Node.js/Express, Unity (C#), and C++/CMake, but works for ANY
  technology stack — the core workflow (interview, architecture doc, diagram, consistency check) is
  stack-agnostic. Detect the stack from context and load the appropriate reference file if one exists.
---

# Code Architecture Planner

This skill helps the user plan and document software architecture before writing code. It produces
two deliverables: a **Mermaid diagram** visualizing the system and a **written architecture document**
covering all key decisions.

## Workflow

### Step 1: Detect Stack

Identify the specific technology stack from the user's request. Look for explicit mentions of
frameworks, languages, and tools — not broad categories.

**Known stacks** (load the corresponding reference file):
- **Node.js / Express**: Express, Node, npm, middleware, REST API with JavaScript/TypeScript → read `references/nodejs-express.md`
- **Unity**: Unity, C#, MonoBehaviour, ScriptableObject, prefab, scene, game development with C# → read `references/unity.md`
- **C++ / CMake**: C++, CMake, header files, FetchContent, vcpkg, systems programming → read `references/cpp-cmake.md`

**Unrecognized stack**: If the project uses a technology without a reference file (Django, Flask, Go, Unreal, Godot, Rust, React Native, etc.), proceed without loading a reference. The core workflow — interview, architecture doc, diagram, consistency check — works for any stack. Use your general knowledge of that technology instead. Don't apologize for the lack of a reference file; just produce the architecture.

A project can span multiple stacks (e.g., a Unity game with a Node.js backend). Load all relevant references.

If the stack is ambiguous and it matters for the architecture (e.g., the user says "a web app" but hasn't specified a framework), ask as part of the interview — don't make it a separate step.

Also determine the **project mode** — this affects which workflow to follow. The user may provide existing code, but the intent behind providing it matters:

- **Greenfield** (building from scratch, no existing code provided) → proceed to Step 2 (Requirements Interview)
- **Greenfield with style reference** (building something new, but the user provided existing code as an example of patterns they like — "use this project as a reference," "I like how this is structured," "here's an example of the style I want") → proceed to Step 2 (Requirements Interview). The reference code is input *context*, not the project being architected. Analyze it during the "Handling User-Provided Context" phase of Step 2 to extract patterns, naming conventions, and structural preferences, then apply them to the new project's architecture.
- **Restructuring** (the user has existing code that IS the project, and wants it reorganized — "my code is messy," "break this apart," "how should I restructure this") → proceed to Step 2R (Restructuring Interview)

The distinction between "style reference" and "restructuring" is about ownership and intent. If the user says "here's my Express app, it's a mess, help me fix it" — that's restructuring. If the user says "here's an MVC example I like, I want to build a new project using this style" — that's greenfield with a style reference. If ambiguous, ask: "Is this the project you want me to architect, or an example of a style you'd like me to follow?"

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

#### Stack-Specific Interview Questions

After the core questions, add stack-specific follow-ups based on the detected stack. Only ask what's relevant and unanswered.

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

For unrecognized stacks, use the core questions only and let the user's answers guide any follow-ups.

#### Pattern Choices

Each reference file contains a **Pattern Choices** section with 2-3 options for key architectural decisions (e.g., routing style, error handling, state machine approach). These represent decisions where multiple approaches are equally valid and the "right" answer depends on user preference, not best practice.

**When to present pattern choices:**
- After the initial interview batch, once you know the project's scale and features. Some choices only matter for certain project types (e.g., routing style only matters if there are multiple routes).
- Present only the choices that are relevant to this specific project. A simple CLI tool doesn't need a state machine choice; a game with 3 enemy types does.
- Present 1-3 pattern choices per project. More than that overwhelms the interview. Pick the ones that most affect the architecture's shape.

**How to present them:**
- Use the short labels from the reference file (A, B, C) with a one-sentence description of each option. Don't dump code examples during the interview — those are reference material for when you're generating the architecture.
- Always include a clarification escape hatch ("I'd like more detail on this before deciding").
- If the user provided a style reference (code example), check whether it already answers a pattern choice. If their example uses functional closures with no classes, don't ask about code style — just use functional.

**If the user doesn't care:**
- If the user says "whatever you think is best" or "I don't have a preference," pick the simplest option that fits the project's scale. Note your choice in the Tech Stack or Notes section so the user knows what was chosen and why.

#### Interview Depth Rules

The interview should be efficient, not exhaustive. Follow this pacing:

1. **Initial batch**: Ask the core questions + relevant stack-specific questions as a single message. Skip anything the user's prompt already answers. This is usually 3-6 questions.
2. **One follow-up round** (if needed): After the user responds, present relevant pattern choices and/or ask up to 3 clarifying questions. Pattern choices and clarifying questions can be combined in one message. Only ask follow-ups that would meaningfully change the architecture — not nice-to-haves.
3. **Then generate.** After at most two rounds of questions, you have enough to produce the architecture. If minor details are still unclear, make a reasonable assumption, state it in the Project Overview, and let the user correct it during review. Don't hold the architecture hostage to perfect information.

If the user signals impatience ("just build it," "that's enough questions," short answers), skip pattern choices and generate immediately with reasonable defaults (simplest option per choice).

#### Handling User Unfamiliarity

When the user indicates they haven't used a system, pattern, or technology before (e.g., "I've never built a crafting system," "I'm new to WebSockets," "first time using CMake"), adapt the architecture:

1. **Prefer accessible patterns over optimal ones.** If the user has never built proc-gen, recommend cellular automata or BSP (well-documented, easy to visualize) over wave function collapse (powerful but harder to debug). The "best" architecture is the one the user can actually build.
2. **Flag the learning curve in Tradeoffs (Section 7).** Be specific: "You mentioned you haven't built a save system before. The ISaveable pattern recommended here is straightforward but requires understanding C# interfaces — here's what to expect."
3. **Sequence implementation tasks to build skills progressively.** In Section 9, order tasks so the user encounters unfamiliar concepts one at a time, with working feedback loops in between. Don't stack three new concepts into Milestone 1.
4. **Include reference links in the Appendix.** For each unfamiliar system, add a link to the best learning resource (official docs, a well-known tutorial, a reference implementation). Not a reading list — one or two high-quality links per unfamiliar concept.

#### Offering choices with a clarification escape hatch

When presenting the user with multiple-choice options (e.g., "how modular should integrations be?"), **always include a clarification option** alongside the substantive choices. The user may not understand the implications of an option, or the question itself may be ambiguous. A choice like "I'd like more detail on this before deciding" lets them get clarification on that specific question without being forced to pick blindly or stall the entire interview.

If the user asks for clarification, explain the options with concrete examples of what each would mean for *their specific project* — not generic definitions. Then re-present the choices. Continue with any remaining questions that weren't blocked by the clarification.

### Step 2R: Restructuring Interview (Existing Codebases Only)

When the user has existing code they want to reorganize (detected in Step 1), use this alternate interview flow instead of Step 2's greenfield interview. The user already knows what their project does — they need help organizing it better.

#### Analysis Phase

Before asking any questions, analyze what the user provided:

1. **Map the current structure.** Identify files, their responsibilities, and how they relate. For a monolith file, identify the logical sections (routes, middleware, queries, config). For a multi-file project, map the dependency graph.
2. **Identify structural problems.** Name the specific issues: "server.js is handling routing, auth, DB queries, and config in one 400-line file" or "your controllers are importing each other circularly." Be concrete, not vague.
3. **Identify what's working.** Not everything needs to change. Call out good patterns: "Your middleware chain is clean," "Your naming conventions are consistent." This builds trust and prevents unnecessary churn.

Present this analysis to the user before asking questions. It shows you actually read their code and gives them a chance to correct misunderstandings.

#### Restructuring Interview Questions

Ask only what the analysis didn't answer:
1. **Pain points** — What specifically feels messy or hard to work with? (The user knows where it hurts.)
2. **Growth direction** — Are you adding features soon, or is this stable code you just want cleaner?
3. **Migration tolerance** — Do you want a gradual refactor (move one piece at a time) or a clean restructure (reorganize everything at once)?
4. **Constraints** — Anything that can't change? (e.g., "the DB schema stays," "I can't rename the main file because PM2 is configured to watch it")

Follow the same depth rules as Step 2: one batch, one follow-up max, then generate.

#### Restructuring-Specific Output Adjustments

When generating the architecture doc for a restructuring project, modify the standard template:

- **Section 1 (Project Overview)**: Focus on what the project *already does* and what the restructuring aims to achieve. Don't describe the project as if it's new.
- **Section 2 (Scope Boundaries)**: Frame as "what's changing vs. what's staying." In-scope = systems being restructured. Out-of-scope = systems left as-is (with notes on when they'd need attention).
- **Section 4 (Folder Structure)**: Present as a **before/after comparison**. Show the current structure on the left and the proposed structure on the right, with arrows or annotations showing where each piece of the old structure ends up. This is more useful than a standalone tree for restructuring.
- **Section 9 (Implementation Tasks)**: Frame as **migration tasks**, not build tasks. Each task should move one piece of the old structure to the new one without breaking the rest. Include a task for "verify nothing broke" after each migration step. Order tasks so the system is functional after every step — no "tear everything apart and reassemble" plans.

The Mermaid diagram should show the **proposed** architecture (not the current one). Optionally produce a second diagram showing the current state if the contrast is helpful.

### Step 3: Generate Architecture Document

After the interview, produce a **structured markdown architecture document** with these sections in order.

#### Output Scaling

Not every project needs the same depth. Scale the architecture document to the project's complexity:

- **Small projects** (personal tool, single-purpose API, game jam): **Collapse the template.** Merge Scope Boundaries, Tradeoffs, and Simplicity Check into a single "Notes" section — short bullets, not full subsections. Skip "Considered and passed over" in Tech Stack. The Appendix should only include terms the user genuinely might not know (don't define "GCC" for a C++ developer). Total doc: ~500-1000 words. Implementation tasks: 5-8. If the project is small enough that the folder tree is obvious, the architecture doc might be overkill — say so and offer to just scaffold the project instead.
- **Medium projects** (portfolio piece, multi-system app, semester project): Full sections with meaningful detail. Total doc: ~1500-2500 words. Implementation tasks: 10-20.
- **Large projects** (production app, team project, complex game with many systems): Detailed sections, multiple diagrams, thorough tradeoff analysis. Total doc: ~2500-4000 words. Implementation tasks: 15-25 (or flag that a separate planning session is needed).

Use the scale determined during the interview (Question 2: Scale & scope). When in doubt, start leaner — it's easier to expand a section the user wants more detail on than to trim a bloated doc.

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
  - Separate files or modules that could be merged at this scale. If a file would be <30 lines, it probably doesn't need to exist separately.
  - External dependencies where the standard library would suffice.
  - Patterns designed for team/production scale applied to a solo/personal project.
  - Premature infrastructure (caching layers, message queues, microservices) before the simple version has proven insufficient.
- **Apply your own findings.** This is critical. If the Simplicity Check identifies something that should be simpler, go back and actually simplify it in Sections 4-6 before presenting the document. Don't flag "this could be one file instead of two" and then leave two files in the tree. The Simplicity Check should only show items where the complexity is genuinely justified despite looking suspicious — cases where you considered simplifying and decided the current approach earns its weight. If you can't articulate a concrete, project-specific reason to keep the complexity, simplify it.
- The goal is NOT to strip the architecture down — it's to make sure every piece of complexity earns its place. A file that exists "for future extensibility" when there's no concrete plan to extend it is dead weight.

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
- **Glossary**: Define acronyms and technical terms that the *specific user* might not know. Scale to the audience — don't define "GCC" for a C++ developer or "REST" for someone who just told you about their Express API. Define terms that are specific to the architecture's recommendations (e.g., "amalgamation" if you recommended the SQLite amalgamation approach, or "PIMPL" if you used that pattern). When in doubt, include it — but a glossary full of terms the user clearly knows is patronizing, not helpful.
- **Reference links**: If the architecture references specific libraries, APIs, or tools, link to their official documentation. The reader shouldn't have to search for "what is CLI11" — give them a URL.
- **Configuration reference**: If the architecture involves environment variables, config files, or API keys that need to be set up, list them here with a brief description of what each one does and where to get it. For example: `DUO_IKEY` — Duo integration key, found in the Duo Admin Panel under Applications.
- Keep it practical. Only include terms and references that actually appear in the document. Don't pad it with generic definitions.

### Step 4: Generate Mermaid Diagram

After (or alongside) the written doc, produce a **Mermaid diagram** that visualizes the architecture.

#### Stack-Specific Diagram Defaults

Choose the diagram type based on the project's stack. These are defaults — override if the project's specific architecture calls for something different.

**Node.js / Express / Web APIs** → **Flowchart (graph TD or LR)**
Show the request flow through layers: Client → Route → Controller → Service → Database. Use subgraphs to group middleware, routes, and data layers. Label edges with the nature of the interaction ("HTTP request", "SQL query", "session check"). This is the most useful diagram for understanding how a request moves through the system.

**Unity / Game projects** → **Class diagram** (primary) + **Flowchart** (secondary)
The class diagram shows system composition: which managers exist, what ScriptableObjects define data, how systems reference each other. This is usually the most valuable diagram for game architecture because it shows ownership and data flow between systems. Add a secondary flowchart for game state flow (MainMenu → Loading → Playing → Paused → GameOver) if the project has meaningful state transitions.

**C++ / CMake projects** → **Flowchart (graph TD)**
Show the module/library dependency graph: which targets link against which, where the public interfaces are. Use subgraphs for library vs. application vs. test targets. Label edges with "links against" or "includes."

**Restructuring projects** → **Two diagrams**
Produce a "before" diagram showing the current structure and an "after" diagram showing the proposed architecture. This makes the transformation concrete and lets the user see exactly what changes. If the current structure is a monolith (everything in one file), the "before" diagram can be a simple box showing the tangle; the "after" shows the clean separation.

**Unrecognized stacks** → **Flowchart (graph TD)** as default. Adapt based on what makes sense for the architecture.

#### General Diagram Guidelines
- Label nodes clearly with component/system names.
- Label edges with the nature of the relationship (e.g., "HTTP request", "emits event", "reads from").
- Group related components using subgraphs where it improves clarity.
- Keep it readable — if the system is complex, create multiple focused diagrams rather than one overwhelming one. A diagram with more than ~15 nodes is usually too dense; split into subsystem diagrams.

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

### Implementation Spec (AI-Ready Output)

After the user approves the architecture, offer to generate an **implementation spec** — a stripped-down, structured version of the architecture optimized for feeding into AI coding tools (Claude Code, Cursor, etc.). The architecture doc is for the human to read and decide; the implementation spec is for an AI to execute.

The implementation spec should contain only actionable information, no prose explanations:

1. **File tree** — the complete folder/file structure, copy-paste ready.
2. **Interfaces** — every public function signature, struct/class definition, and type. Use actual code syntax for the target language, not prose descriptions. For example: `int calculateStreak(const std::vector<std::string>& completionDates);` not "a function that calculates the streak."
3. **Schemas** — database schemas (SQL), data models (JSON shape), API contracts (endpoint + request/response shape). Copy-paste ready.
4. **Implementation task list** — the ordered task list from Section 9, but stripped to just task name, dependencies, and a one-line description. No effort estimates or milestone framing — the AI doesn't need to plan its time.
5. **Constraints** — any non-obvious rules the AI must follow: "use the SQLite amalgamation, not system SQLite," "all dates stored as YYYY-MM-DD strings in local time," "CSV output must follow RFC 4180 escaping."

Omit from the implementation spec: Project Overview, Scope Boundaries, "why" explanations, Tradeoffs, Simplicity Check, Appendix glossary. These served their purpose during the architecture review — the decisions are made, the AI just needs to build.

Save the implementation spec as a separate markdown file (e.g., `project-name-spec.md`). Don't combine it with the architecture doc — they serve different audiences.

This is optional — only generate it if the user asks or if you've detected they plan to use AI tooling for implementation. A natural prompt: "Want me to generate an implementation spec you can hand to Claude Code?"

## Important Principles

- **Explain the "why"** — Don't just list structure; justify decisions. The user wants to understand the reasoning, not just receive a blueprint.
- **Don't reinvent the wheel** — Lean on industry-standard patterns and proven approaches. If Express middleware, Unity ScriptableObjects, or CMake's target-based model already solve the problem well, use them. Novel architecture should be reserved for novel problems.
- **Simpler until proven otherwise** — Default to the lightest-weight solution that meets the requirements. The Simplicity Check (Section 8) exists to enforce this. Every layer of abstraction, every additional dependency, and every separated module needs to justify its existence for *this* project at *this* scale — not for a hypothetical future version.
- **Be a collaborator, not a yes-man** — The user's input is the most important signal, but the skill's job is to produce a *great* architecture, not just a familiar one. When a standard approach is stronger than what the user proposed, advocate for it — explain the benefit, acknowledge the tradeoff, and let them decide. When the user's approach is sound, say so and build on it.
- **Favor patterns the user already knows** — If you know the user's tech stack or experience level, lean into familiar patterns rather than introducing unnecessary new concepts. But if an unfamiliar pattern is meaningfully better for the project, introduce it with enough explanation for the user to evaluate it.
