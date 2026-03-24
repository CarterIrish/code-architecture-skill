# Consistency Check

Run this 9-point audit before presenting the architecture to the user, and again after every
revision. Fix inconsistencies silently — don't call out the self-correction.

## Checklist

1. **File tree vs. component descriptions**: Every file in Section 4 must correspond to a component in Section 5, and vice versa. No phantom files, no ghost components.

2. **Tech stack vs. actual usage**: Every technology in Section 3 must appear in the architecture. Rejected tech must not show up in file tree, components, or data flow. If a library appears in implementation details, it must be in the tech stack.

3. **Data flow vs. components**: Flows in Section 6 must only reference components and interfaces from Section 5. If a flow mentions "the WebSocket server," there must be a WebSocket server component.

4. **Simplicity Check vs. proposed architecture**: If Section 8 recommends simplifying something, Sections 4-6 must reflect the simpler approach. Don't flag "this could be one file" and then leave two files in the tree.

5. **Feasibility of proposed solutions**: Verify recommendations are actually possible with the stated technologies. Check for: APIs that don't exist or work differently, library features requiring a different version, patterns that don't apply to the stated framework, unsupported configuration.

6. **Diagram vs. document**: Every Mermaid diagram node must correspond to a component in Sections 4-5. Every arrow must correspond to a relationship in Section 6. No orphan nodes, no undocumented connections.

7. **Scope boundaries vs. architecture**: Out-of-scope features (Section 2) must not appear in the file tree, components, or task list. In-scope features must be covered by at least one component and one implementation task.

8. **Implementation tasks vs. components**: Every component in Section 5 must be created by at least one task in Section 9. Every task must reference existing components. Task dependencies must form a valid ordering — no circular dependencies, no task depending on a later task.

9. **Appendix completeness**: Scan for acronyms and technical terms — all must appear in the glossary. Every library or tool mentioned by name must have a reference link. Every environment variable or config value must appear in the configuration reference.
