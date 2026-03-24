# Code Architecture Planner

A Claude skill that plans and documents software architecture through a collaborative, iterative process. It produces a structured architecture document with Mermaid diagrams — not by generating a generic template, but by interviewing you, challenging assumptions, and advocating for the simplest approach that meets your requirements.

## What makes this different

Most architecture skills generate a document and hand it to you. This one argues with you first.

- **Pattern choices** — Instead of defaulting to whatever Claude thinks is "best practice," the skill presents 2-3 valid architectural options (routing style, error handling, state machines, etc.) and lets you pick. Your style preference matters more than convention.
- **Simplicity Check with teeth** — Every architecture is audited for over-engineering, and the findings are *applied* before you see the doc. If a file would be 20 lines, it gets merged. The Simplicity Check doesn't just flag problems — it fixes them.
- **Restructuring workflow** — Not just greenfield projects. Provide existing code and the skill analyzes it, identifies structural problems and strengths, produces before/after comparisons, and generates migration tasks that keep your app functional at every step.
- **Style references** — Provide an existing project you like the structure of, and the skill extracts the patterns (routing style, code conventions, folder layout) and applies them to your new project. No more fighting Claude's default opinions.
- **Output scaling** — A personal CLI tool doesn't get the same 2000-word treatment as a production multiplayer game. Small projects get a collapsed template; large projects get the full treatment.
- **Scope boundaries as thresholds** — Out-of-scope features aren't just listed, they come with upgrade paths. You know exactly what it costs to add multi-user support or drag-and-drop later.
- **Collaborative pushback** — If you suggest a pattern that's suboptimal for your project, the skill makes its case once, explains why, and lets you decide.
- **Consistency checking** — A 9-point internal audit catches contradictions between sections before you see the doc.
- **AI-ready implementation specs** — After you approve the architecture, optionally generate a stripped-down spec (file tree, function signatures, schemas, task list) optimized for handing to Claude Code or Cursor.

## Supported stacks

| Stack | Reference file | Covers |
|-------|---------------|--------|
| **Node.js / Express** | `references/nodejs-express.md` | MVC patterns, Express routing, PostgreSQL/MongoDB/SQLite, auth, middleware, WebSockets, rate limiting, API versioning, monorepos. **Pattern choices**: routing style, code style (functional vs class), error handling, module system. |
| **Unity (C#)** | `references/unity.md` | Component architecture, ScriptableObjects, state machines, proc-gen (cellular automata, BSP, WFC), day/night cycles, save systems, audio architecture. **Pattern choices**: inter-system communication, state machine style, data architecture. |
| **C++ / CMake** | `references/cpp-cmake.md` | Modern CMake, library/app split, FetchContent/vcpkg, testing with GoogleTest/Catch2. **Pattern choices**: project organization, error handling, dependency management. |
| **Any other stack** | *(none — uses general knowledge)* | The core workflow (interview, architecture doc, diagram, consistency check) is stack-agnostic. Works for Django, Go, Rust, Unreal, Godot, etc. — just without stack-specific pattern suggestions. |

The skill auto-detects the stack from your request. Projects spanning multiple stacks (e.g., a multiplayer game with a Node.js backend) load all relevant references.

## Architecture document sections

The skill produces a markdown document scaled to your project's complexity:

1. **Project Overview** — What the project is and what it solves
2. **Scope Boundaries** — What's in, what's out, and the cost of crossing each boundary
3. **Tech Stack** — Every technology with a justification, plus alternatives considered
4. **Folder & File Structure** — Annotated directory tree (before/after comparison for restructuring)
5. **Components & Systems** — Each component's responsibility, interface, and dependencies
6. **Data Flow & Relationships** — How data moves end-to-end
7. **Tradeoffs & Pitfalls** — Realistic risks with mitigations specific to this project
8. **Simplicity Check** — Where complexity earned its place, after simplifications were already applied
9. **Implementation Tasks** — Ordered milestones with dependencies and effort estimates
10. **Appendix** — Audience-aware glossary, reference links, configuration reference

Small projects collapse sections 2, 7, and 8 into a single "Notes" section. Plus a Mermaid diagram (stack-specific defaults: flowcharts for APIs, class diagrams for games, before/after pairs for restructuring).

## Workflow

1. **Detect stack + project mode** → Load reference file(s), route to greenfield / style reference / restructuring
2. **Interview** → Core questions + stack-specific follow-ups + pattern choices. Adapts to user-provided context. Max 2 rounds.
3. **Generate** → Architecture doc scaled to project complexity + Mermaid diagram
4. **Consistency check** → 9-point internal audit
5. **Review & iterate** → Targeted revisions with cascade tracking. Supports "what if" comparisons and drill-downs.
6. **Implementation spec** *(optional)* → AI-ready output for Claude Code / Cursor

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
- *"Help me plan the architecture for a Unity 2D roguelike with proc-gen and crafting"*
- *"I'm building a C++ CLI tool with CMake — I've never set up CMake before"*
- *"Here's my 400-line server.js monolith — help me break it apart"*
- *"Use this MVC example as a style reference for my new project"*
- *"I'd rather use MongoDB and Redis for sessions"* (mid-conversation revision)

## Design philosophy

- **Simpler until proven otherwise** — Every abstraction justifies its existence at this project's scale, not a hypothetical future version.
- **Don't reinvent the wheel** — Use industry-standard patterns. Novel architecture is for novel problems.
- **Be a collaborator, not a yes-man** — Advocate for better approaches. Explain the tradeoff. Respect the user's final decision.
- **Respect user preferences** — Pattern choices let the user pick their style. Style references carry forward. Claude's "best practice" opinion doesn't override user comfort.
- **Explain the why** — Don't just list structure; justify every decision.

## Built with

This skill was built iteratively using the [skill-creator](https://github.com/anthropics/skill-creator) framework — drafted, tested against multiple domain-specific prompts (greenfield, restructuring, style reference), revised based on performance evaluation, and packaged.

## License

MIT
