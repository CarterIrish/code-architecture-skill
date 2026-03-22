# Code Architecture Planner

A Claude skill that plans and documents software architecture through a collaborative, iterative process. It produces a structured architecture document with Mermaid diagrams — not by generating a generic template, but by interviewing you, challenging assumptions, and advocating for the simplest approach that meets your requirements.

## What makes this different

Most architecture skills generate a document and hand it to you. This one argues with you first.

- **Simplicity Check** — Every architecture is audited for over-engineering. The skill actively flags where a simpler approach would work and tells you exactly when you'd need to upgrade. A solo project doesn't get microservices just because that's the "standard" pattern.
- **Scope boundaries as thresholds** — Out-of-scope features aren't just listed, they come with upgrade paths. You know exactly what it costs to add multi-user support or drag-and-drop later.
- **Collaborative pushback** — If you suggest a pattern that's suboptimal for your project, the skill makes its case once, explains why, and lets you decide. It's a collaborator, not a yes-man or a rubber stamp.
- **Consistency checking** — Before presenting the architecture, the skill runs a 9-point internal audit catching contradictions between sections (phantom files in the tree, rejected tech lingering in the stack, diagram nodes with no matching component).
- **Multi-domain reference files** — The skill loads domain-specific patterns based on your project type rather than applying web dev conventions to a game or vice versa.

## Supported domains

| Domain | Reference file | Covers |
|--------|---------------|--------|
| **Web** | `references/webdev.md` | Node.js, Express, REST APIs, PostgreSQL, auth patterns, middleware, deployment |
| **Game** | `references/gamedev.md` | Unity, C#, ScriptableObjects, state machines, component architecture, ECS |
| **C++ / CMake** | `references/cpp-cmake.md` | Modern CMake, library/application split, FetchContent, CLI tools, systems programming |

The skill auto-detects the domain from your request. Projects spanning multiple domains (e.g., a multiplayer game with a Node.js backend) load all relevant references.

## Architecture document sections

The skill produces a single markdown document with 9 sections plus an appendix:

1. **Project Overview** — What the project is and what it solves
2. **Scope Boundaries** — What's in, what's out, and the cost of crossing each boundary
3. **Tech Stack** — Every technology with a justification, plus alternatives considered
4. **Folder & File Structure** — Annotated directory tree
5. **Components & Systems** — Each component's responsibility, interface, and dependencies
6. **Data Flow & Relationships** — How data moves end-to-end
7. **Tradeoffs & Pitfalls** — Realistic risks with mitigations specific to this project
8. **Simplicity Check** — Where to start simpler, with concrete upgrade thresholds
9. **Implementation Tasks** — Ordered milestones with dependencies and effort estimates
10. **Appendix** — Glossary, reference links, configuration reference

Plus a Mermaid diagram visualizing the system architecture.

## Workflow

1. **Detect domain** → Load the right reference file(s)
2. **Interview** → Focused questions with clarification options. Adapts to user-provided context (existing code, folder structures, reference projects)
3. **Generate** → Full architecture doc + Mermaid diagram
4. **Consistency check** → Internal audit for contradictions
5. **Review & iterate** → Targeted revisions, not full regeneration. Tracks approved sections.

## Installation

### Claude Code (recommended)

Add the marketplace and install with two commands:

```
/plugin marketplace add CarterIrish/code-architecture-skill
/plugin install code-architecture@code-architecture-marketplace
```

Or via terminal:

```bash
claude plugin marketplace add CarterIrish/code-architecture-skill
claude plugin install code-architecture@code-architecture-marketplace
```

### Claude.ai

Download `code-architecture.skill` from [Releases](../../releases) and drag it into a Claude conversation, or upload it in **Settings > Customize > Skills**.

### Manual install

Copy the skill directory into your personal skills folder:

```bash
# macOS / Linux
cp -r code-architecture ~/.claude/skills/code-architecture

# Windows (PowerShell)
Copy-Item -Recurse code-architecture "$HOME\.claude\skills\code-architecture"
```

## Example prompts

- *"I want to architect a REST API for a personal dashboard with auth and PostgreSQL"*
- *"Help me plan the architecture for a Unity survival horror roguelike with crafting and inventory"*
- *"I'm building a C++ CLI tool with CMake — Phase 1 is CLI, Phase 2 adds a Dear ImGui GUI"*
- *"Here's my current project structure — help me plan the next phase"*
- *"I'm building a multiplayer game with a Node.js backend and a Unity client"*

## Design philosophy

- **Don't reinvent the wheel** — Use industry-standard patterns. Novel architecture is for novel problems.
- **Simpler until proven otherwise** — Every abstraction justifies its existence at this project's scale, not a hypothetical future version.
- **Be a collaborator, not a yes-man** — Advocate for better approaches. Explain the tradeoff. Respect the user's final decision.
- **Explain the why** — Don't just list structure; justify every decision.

## Built with

This skill was built iteratively using the [skill-creator](https://github.com/anthropics/skill-creator) framework — drafted, tested against 4 domain-specific prompts plus a real project, revised based on feedback, and packaged.

## License

MIT
