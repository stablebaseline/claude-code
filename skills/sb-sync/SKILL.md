---
name: sb-sync
description: "Generate or update the Stable Baseline section in AGENTS.md for cross-IDE auto-sync. Preserves all existing AGENTS.md content. Use when user wants to sync AGENTS.md, regenerate sync rules, or run sb-sync."
---

# Stable Baseline — Sync AGENTS.md

$ARGUMENTS

Add or update the Stable Baseline section in the repository's `AGENTS.md` file. This is the cross-IDE standard for agent instructions — it works in Cursor, VS Code Copilot, Claude Code, OpenCode, Windsurf, Zed, Warp, Roo Code, Aider, and all other AGENTS.md-compatible tools.

If you don't have IDs yet, read `.sb/config.json` or call `listWorkspaces` → `listProjects` to discover them.

CRITICAL — PRESERVE EXISTING CONTENT:
- If `AGENTS.md` already exists, READ IT FIRST. You MUST NOT remove, replace, or overwrite ANY existing content.
- ONLY append or update the "Stable Baseline — Documentation Sync" section.
- If that section already exists from a previous run, update it in-place. Leave all other sections untouched.
- If `AGENTS.md` does not exist, create a new file with ONLY the Stable Baseline section below (do not invent other rules).

BEFORE writing the section, you MUST analyse the repository to understand:
- What languages, frameworks, and tools are used
- What the key directories and file patterns are
- What areas of the codebase are architecturally significant
- What existing documentation exists in Stable Baseline (call `getProjectHierarchy`)

Then generate the Stable Baseline section with ALL of the following sub-sections:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 1 — Project Documentation Link
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Under the heading `## Stable Baseline — Documentation Sync`, state that this project's documentation lives in Stable Baseline.
Include the workspaceId, projectId, and MCP server name (`sb`).

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 2 — Auto-Sync Rules
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

This is the CRITICAL section. Create a table mapping file patterns to documentation actions.

The agent reading this file must know: "When I change file X, I need to update document Y in Stable Baseline."

Rules for building the table:
- File patterns MUST be specific to THIS repo (based on your analysis)
- Map each pattern to the specific Stable Baseline document that covers it
- Include the action: `editDocument` for updates, `createDocument` for new docs
- Cover ALL architecturally significant areas

Categories to consider (include only what's relevant to THIS repo):
- **Architecture changes** — system design files, service boundaries, API schemas
- **Security changes** — auth config, authorization rules, secrets management
- **Data model changes** — migrations, schema files, ORM models
- **Infrastructure changes** — Dockerfiles, CI/CD configs, IaC templates, deploy scripts
- **API contract changes** — OpenAPI specs, protobuf, GraphQL schemas, route definitions
- **Dependency changes** — package.json, go.mod, Cargo.toml, requirements.txt
- **Configuration changes** — environment configs, feature flags, build configs
- **Business logic changes** — core domain logic, business rules, workflow definitions

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 3 — Sync Workflow
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Explain the step-by-step workflow the agent must follow when a sync trigger fires:

1. Read `.sb/config.json` to get workspaceId, projectId, AND the `cache` object
2. Look up the target document in `cache.documents` by title
   - **Cache HIT**: Call `getDocument` directly with the cached document ID — skip `getProjectHierarchy` entirely
   - **Cache MISS or 404**: Call `getProjectHierarchy` to find the document, then update `cache.documents` with the discovered ID so future syncs are fast
3. Read the document's current content and get its `versionTimestamp`
4. Determine what changed and how the documentation should be updated
5. Call `editDocument` with targeted patches (NOT full rewrites)
6. If no matching document exists, call `createDocument` in the appropriate folder, then add it to `cache.documents`
7. Update `cache.lastUpdated` in `.sb/config.json` with the current ISO 8601 timestamp

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 4 — MCP Tool Rules
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Include these universal rules:

- Use `sb` tools for ALL document, folder, diagram, and image operations
- NEVER manually write or edit `DIAGRAM_OMITTED`, `IMAGE_OMITTED`, or `<!-- DIAGRAM: ... -->` markers
- ALWAYS call `getDocument` before `editDocument` (versionTimestamp is required for optimistic locking)
- Create documents first, THEN insert diagrams/images with dedicated tools
- For bulk text substitutions use `findAndReplaceTextInDocument`
- Prefer `editDocument` with small patches over full document rewrites

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 5 — Documentation Principles
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Include a brief section on documentation philosophy:

- Document the "why" behind decisions, not just the "what"
- Technical and business documentation must be consistent and reference each other
- Documentation should prevent architectural drift by recording rationale
- Every document should be immediately useful to a developer joining tomorrow

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 6 — Resources
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

List the MCP resources:
- `sb://cdmd-guide` — CDMD syntax reference
- `sb://diagram-types` — Available diagram types and their renderers
- `sb://server-info` — Server capabilities, tools, and prompts
- `sb://projects/{projectId}/hierarchy` — Project folder/document structure

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

OUTPUT: If `AGENTS.md` already exists, read it first, then output the UPDATED file preserving ALL existing content and appending/updating ONLY the Stable Baseline section. If it does not exist, output a new file containing ONLY the Stable Baseline section. The Stable Baseline section must be self-contained — an agent reading it with zero prior context must understand what to do.
