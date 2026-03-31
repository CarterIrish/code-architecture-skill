# Interview Reference

Detailed guidance for the interview phase of the architecture workflow. This file covers
all project modes: greenfield, restructuring, architecture review, and tech discovery.

## Tech Discovery

### When It Triggers

Tech Discovery activates when the user describes a project by **what it does** rather than
**what it's built with**. Signals:

- The user describes a purpose or domain: "a personal dashboard," "a tool to manage recipes,"
  "a home lab monitoring system."
- No frameworks, languages, or databases are named.
- The user explicitly asks for recommendations: "what should I build this with?"

**Skip Tech Discovery when:**
- The user names specific technologies: "React frontend with Express API."
- The user provides existing code (→ restructuring or review mode).
- The user provides a style reference with clear tech choices.

**Partial stack specification:** If the user specifies some technologies but leaves gaps
(e.g., "I want to use PostgreSQL but I'm not sure about the frontend"), run Tech Discovery
only for the unspecified parts. Acknowledge what's already decided and build around it.

### Phase 1: Understand the Project Purpose

Start from the user's initial description. If it's clear enough to analyze, move straight to
Phase 2. If it's too vague, ask **1-2 purpose-focused questions** — not technical questions.

Good clarifying questions (purpose-focused):
- "What's the core thing you'll use this for day-to-day?"
- "Is this for just you, or will other people use it too?"
- "Are there any specific services or data sources it needs to connect to?"

Bad clarifying questions (too technical too early):
- "Do you want a REST API or GraphQL?"
- "Have you considered using WebSockets?"
- "What database are you thinking about?"

The goal is to understand the **problem space**, not to quiz the user on technologies they
may not know yet.

### Phase 2: Identify Technical Requirements

Analyze the project description and extract what the project **needs to do**. Frame each
requirement in terms of the user's project, not abstract capabilities.

**Requirements to look for:**

| Capability Need | Project Signal | Example |
|---|---|---|
| Real-time UI updates | Status indicators, live data, dashboards, chat | "I want to see my server status update live" |
| Data persistence | User data, settings, history, content management | "I need to store recipes and tag them" |
| Authentication | Multiple users, private data, admin features | "Only I should be able to access the admin panel" |
| External API integration | Third-party services, data fetching, webhooks | "Pull weather data and display it" |
| Server-side rendering | SEO, fast initial load, content-heavy pages | "A blog that search engines can index" |
| Background/scheduled tasks | Cron jobs, polling, periodic updates | "Check server status every 5 minutes" |
| File handling | Uploads, image processing, document generation | "Users can upload profile photos" |
| Complex client-side state | Multi-step forms, drag-and-drop, interactive editors | "A kanban board with drag-and-drop" |
| Offline support | PWA, local-first, unreliable connectivity | "Should work when my phone has no signal" |
| Multi-platform | Web + mobile, desktop app, CLI | "Web app now, maybe a mobile app later" |

**Be specific about why.** Don't just list "needs a database." Say: "Your recipe collection
needs persistent storage with search and tagging — that points toward a relational database
where you can query by ingredient or category."

**Don't over-discover.** Only surface requirements that are evident from the project description
or reasonable to infer. Don't ask the user to confirm every possible capability. If something
might be needed later, note it as a future consideration, not a current requirement.

### Phase 3: Recommend a Stack

Present a **cohesive stack recommendation** — technologies that work well together, not a
shopping list of "best in class" tools evaluated independently.

**Structure the recommendation as:**

1. **Requirements summary.** Briefly restate what the project needs (from Phase 2). This shows
   the user you understood their project before jumping to tech choices.
2. **Recommended stack.** Each technology with a one-line justification tied to a specific
   requirement. Group by layer (frontend, backend, database, infrastructure) when applicable.
3. **Key reasons this stack fits.** 2-3 sentences on why these technologies work well *together*
   for *this specific project*. Not generic praise — concrete fit.
4. **Alternatives worth mentioning.** Only if there's a genuinely competitive alternative for a
   key choice. Keep to 1-2 alternatives max. Explain the tradeoff, not just the option.

**Example (personal dashboard project):**

> Based on what you've described, your dashboard needs: live-updating status indicators
> (real-time UI), data from external services like your home lab APIs (API integration),
> and persistent storage for configuration and history (database).
>
> **Recommended stack:**
> - **React** — Your status indicators and server metrics need to update without page
>   refreshes. React's component model makes it straightforward to build self-updating
>   widgets that each manage their own data polling or WebSocket connection.
> - **Express (Node.js)** — Lightweight API layer to proxy your home lab services and
>   handle auth. Node.js is a natural fit since you're already in the JavaScript ecosystem
>   with React.
> - **SQLite** — For a personal dashboard, SQLite gives you persistent storage with zero
>   setup. No separate database server to manage. If you later need multi-user access,
>   PostgreSQL is a straightforward upgrade.
> - **Socket.io** — For the live status indicators. Pushes updates from your server to the
>   dashboard instead of the browser polling every few seconds.
>
> This stack keeps the infrastructure simple (no separate database server, one language
> across frontend and backend) while giving you real-time capabilities where your project
> actually needs them.
>
> **Alternative worth considering:** If you'd prefer a batteries-included framework over
> assembling Express middleware yourself, **Next.js** would give you React + API routes +
> server-side rendering in one package. Tradeoff: more opinionated, slightly steeper
> learning curve if you haven't used it, but less configuration.

**Principles for good recommendations:**

- **Tie every choice to a project requirement.** If you can't point to a specific need it
  serves, reconsider whether it belongs.
- **Prefer familiar technologies when the difference is marginal.** If the user mentions
  experience with a technology that would work, lean toward it even if something else is
  slightly "better" on paper.
- **Don't over-stack.** A personal tool doesn't need Kubernetes, Redis, and a message queue.
  Recommend the minimum stack that covers the requirements. The Simplicity Check later will
  catch over-engineering, but it's better to not introduce it in the first place.
- **Name the upgrade path.** If you're recommending a simpler choice now (SQLite over
  PostgreSQL, polling over WebSockets), briefly note when the user would outgrow it.
- **Be honest about tradeoffs.** Don't sell a technology; explain what it gives and what it
  costs. Let the user make an informed choice.

### Phase 4: Wait for Approval

After presenting the recommendation, **stop and wait.** Do not proceed to the architecture
interview until the user responds.

**If the user approves:** Load any relevant reference files for the confirmed stack. Proceed
to the interview (Step 3). Skip questions about project summary and tech stack — those are
already resolved.

**If the user pushes back on specific choices:** Adjust those parts of the recommendation.
Don't re-present the entire stack — just address what changed and confirm.

**If the user wants a completely different direction:** Accept it. Note any concerns once
("SQLite might struggle if you need concurrent writes from multiple users"), then respect the
decision and proceed.

**If the user asks for more detail on a recommendation:** Explain further. Provide concrete
examples of how that technology would work in *their* project. Then re-confirm.

---

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

## Restructuring Interview (Step 3R)

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

## Architecture Review (Step 3A)

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
- "Want me to restructure any of these?" (→ Step 3R)
- "Want me to architect a plan for adding [feature]?" (→ Step 3)
- "Want me to go deeper on any of these concerns?"
