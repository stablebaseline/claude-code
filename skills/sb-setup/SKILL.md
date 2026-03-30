---
name: sb-setup
description: "Onboard a repository into Stable Baseline: discover IDs, analyse the repo, create a bespoke documentation structure, and augment AGENTS.md with sync rules. Use when user wants to set up Stable Baseline, onboard a project, generate initial documentation, or run sb-setup."
---

# Stable Baseline — Project Setup

$ARGUMENTS

You are connected to the Stable Baseline MCP server. Your job is to fully onboard this repository.

IMPORTANT — Tool Usage:
All MCP tools (listWorkspaces, listProjects, getProjectHierarchy, createFolder, createDocument, insertDiagramInDocument, etc.) are ALREADY available. Call them directly by name. Do NOT use `searchTools` to discover or activate them — `searchTools` is a convenience meta-tool, not an activation mechanism. If a tool call fails with "tool not found", retry the call directly — do NOT search for alternatives.

Follow every step in order. Do NOT skip any step. Steps 1 and 2 are PREREQUISITES — if either fails, STOP and tell the user what to fix. Do NOT continue to Step 3+.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 1 — Verify MCP connection
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Test the connection by calling `listWorkspaces`.

If the call FAILS (error, timeout, auth failure, or tool not found):
→ STOP. Tell the user exactly this:
  "The Stable Baseline MCP server (`sb`) is not connected or not authenticated.

   **What to do:**
   1. Follow the setup guide: https://stablebaseline.io/docs/mcp/setup
   2. Once the MCP connection is configured and working, come back here and type **continue**."
→ Wait for the user to say "continue" (or equivalent). When they do, restart from Step 1.
→ Do NOT proceed to any further steps until the connection succeeds.

If the call SUCCEEDS, the connection is working. Continue to Step 2.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 2 — Resolve workspace and project
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Using the result from Step 1:

a) **No workspaces returned** (empty list):
   → STOP. Tell the user exactly this:
     "No workspaces found on your Stable Baseline account.

      **What to do:**
      1. Go to https://app.stablebaseline.io/ and create a workspace (and a project inside it).
      2. Once done, come back here and type **continue**."
   → Wait for the user to say "continue" (or equivalent). When they do, restart from Step 2.
   → Do NOT proceed until workspaces exist.

b) **Multiple workspaces returned**:
   → ASK the user which workspace to use. List them by name. Wait for their answer before continuing.

c) **Exactly one workspace**:
   → Use it automatically.

Once you have a workspaceId, call `listProjects` with that workspaceId:

d) **No projects returned** (empty list):
   → STOP. Tell the user exactly this:
     "No projects found in this workspace.

      **What to do:**
      1. Go to https://app.stablebaseline.io/ and create a project in your workspace.
      2. Once done, come back here and type **continue**."
   → Wait for the user to say "continue" (or equivalent). When they do, re-call `listProjects` and continue from Step 2d.
   → Do NOT proceed until a project exists.

e) **Multiple projects returned**:
   → ASK the user which project to use. List them by name. Wait for their answer before continuing.

f) **Exactly one project**:
   → Use it automatically.

Store the workspaceId and projectId — you will need them for every subsequent call.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 3 — Create local config
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Create `.sb/config.json` in the repository root:

```json
{
  "workspaceId": "<workspace-id>",
  "projectId": "<project-id>",
  "defaultFolderId": null,
  "mcp": {
    "server": "sb"
  },
  "cache": {
    "documents": {},
    "folders": {},
    "lastUpdated": "<ISO-8601-timestamp>"
  }
}
```

The `cache` object stores document and folder IDs so that future sync operations (`sb-sync`, `sb-update`) can skip `getProjectHierarchy` and jump directly to `getDocument(docId)`. You will populate it in Step 7 as you create docs and folders.

- `cache.documents`: Maps document titles to `{ "id": "<doc-id>", "folderId": "<folder-id-or-null>" }`
- `cache.folders`: Maps folder names to their folder IDs
- `cache.lastUpdated`: ISO 8601 timestamp of the last cache update

Add `.sb/` to `.gitignore` if not already present.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 4 — Check for existing documentation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Call `getProjectHierarchy` with the projectId from Step 2 to check if documentation already exists.

**A) Project is EMPTY** (no folders or documents):
→ This is a fresh project. Continue to Step 5.

**B) Project has EXISTING documentation** (folders and/or documents found):
→ STOP. Show the user what already exists — list the folder tree and document titles.
→ Then ASK the user to choose one of these options:

  1. **Augment** — "Keep your existing documentation. I'll analyse your repo, identify gaps, and only create what's missing or needs updating. Existing documents won't be modified."
  2. **Replace** — "Delete all existing folders and documents, then create a fresh documentation set from scratch. This is destructive — version history will be lost."
  3. **Cancel** — "Stop here so you can review your existing documentation first. Re-run `/sb-setup` when ready."

Wait for the user to choose before proceeding.

If the user chooses **Augment**:
- In Step 6, you MUST account for existing docs. Read each existing document (call `getDocument`) to understand what's already covered.
- Design your structure around what already exists — fill gaps, don't duplicate.
- In Step 7, only create NEW folders/documents for areas not yet covered. Do NOT recreate or overwrite existing docs.
- In Step 8, include BOTH existing and new documents in the AGENTS.md auto-sync table.

If the user chooses **Replace**:
- Delete all existing folders and documents (call `deleteFolder` for root folders, `deleteDocument` for root-level documents).
- Then continue to Step 5 as if the project were empty.

If the user chooses **Cancel**:
- STOP. Tell the user: "No changes made. Your existing documentation is untouched. Run `/sb-setup` again when you're ready."
- Do NOT proceed to any further steps.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 5 — Analyse the repository
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Before creating any documentation, you MUST deeply understand this repository. Analyse:

a) **Tech stack** — languages, frameworks, runtimes, package managers, build tools
b) **Architecture** — monolith vs microservices, frontend/backend split, API patterns, data layer
c) **Directory structure** — how is the code organised? what are the key directories?
d) **Entry points** — where does execution start? what are the main modules?
e) **Dependencies** — critical third-party libraries, internal shared packages
f) **Infrastructure** — deployment targets (cloud, containers, serverless), CI/CD, IaC
g) **Data & storage** — databases, caches, message queues, file storage
h) **Auth & security** — authentication providers, authorization model, secrets management
i) **Testing** — test frameworks, test structure, coverage approach
j) **Existing documentation** — READMEs, wiki, docs folder, inline comments quality
k) **Domain context** — what business problem does this solve? who are the users?
l) **Pain points** — what areas look under-documented, inconsistent, or risky?

Read key files: README, package.json/Cargo.toml/go.mod/etc., config files, directory listings, a sample of core source files. Be thorough.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 6 — Design a bespoke documentation structure
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Based on your analysis, design a folder-and-document structure in Stable Baseline that is **specific to this repository**. There is NO fixed template. Every repo gets a different structure.

PRINCIPLES:

- **Technical Standards & Coding Patterns** — How code should be written, naming conventions, style guides, code review checklists relevant to THIS stack
- **Architectural Patterns** — System design, component relationships, data flows, API contracts, integration points specific to THIS codebase
- **Security Patterns** — Auth flows, authorization model, data protection, secrets management, vulnerability handling relevant to THIS system
- **Operational & Maintenance** — Deployment procedures, monitoring, alerting, incident response, runbooks for THIS infrastructure
- **Compliance & Governance** — Regulatory requirements, audit trails, data retention policies applicable to THIS domain
- **Business Context** — Business rules, domain models, stakeholder requirements, product decisions, user personas for THIS product
- **Drift Prevention** — Document the "why" behind every significant decision so future developers don't deviate without understanding the consequences

RULES for structure design:
- Create folders and nesting that match THIS repo's natural boundaries
- Group documents by how people will FIND them, not by type
- Every document must have a clear purpose — no placeholder docs
- Include architecture diagrams where visual representation adds value
- Do NOT create generic folders like "Miscellaneous" or "Other"
- Keep nesting shallow (3 levels max) unless the repo is genuinely complex

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 7 — Create the documentation in Stable Baseline
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Using the structure you designed:

1. Call `createFolder` for each folder (use parentId for nesting).
   → After EACH `createFolder` call, update `.sb/config.json` `cache.folders` with the folder name → folder ID mapping.
2. Call `createDocument` for each document with substantive CDMD content.
   - Write REAL content based on what you learned in Step 5. Not stubs, not TODOs.
   - Use the CDMD format (call `sb://cdmd-guide` resource if unsure of syntax).
   - Do NOT include DIAGRAM or IMAGE markers in the cdmd body.
   → After EACH `createDocument` call, update `.sb/config.json` `cache.documents` with the document title → `{ "id": "<doc-id>", "folderId": "<folder-id-or-null>" }` mapping.
3. After creating EACH document, you MUST generate diagrams. This is NOT optional.

   **Diagram generation is MANDATORY.** Every document that covers architecture, data flow, system design, component relationships, API contracts, deployment topology, state machines, data models, sequence flows, pipelines, or infrastructure MUST include at least one diagram.

   For each document:
   a) Decide the best diagram type. Call `getDiagramTypeGuide` for the type you pick to learn the correct DSL syntax.
   b) Write the diagram DSL following the guide.
   c) Call `insertDiagramInDocument` with appropriate afterLine placement.

   The ONLY documents exempt from diagrams are pure text documents like glossaries, coding style guides, or checklists.

Write documents that are immediately useful. A developer joining tomorrow should be able to read these docs and understand the system.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 8 — AGENTS.md (augment or create)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

`AGENTS.md` is the cross-IDE standard for agent instructions (supported by Cursor, VS Code Copilot, Claude Code, OpenCode, Windsurf, Zed, Warp, Roo Code, Aider, and 15+ other tools).

CRITICAL: Check if `AGENTS.md` already exists in the repository root BEFORE writing.

**If AGENTS.md ALREADY EXISTS:**
- You MUST NOT replace, overwrite, or remove ANY existing content.
- Read the entire file first.
- APPEND the Stable Baseline section below to the END of the existing file.
- If a "Stable Baseline" section already exists from a previous run, update ONLY that section in-place.

**If AGENTS.md does NOT exist:**
- Create it with the Stable Baseline section below.

Use the `/sb-sync` skill to generate the Stable Baseline section, OR write it directly following this structure:

```markdown
## Stable Baseline — Documentation Sync

This project's documentation lives in Stable Baseline.
- Workspace: `<workspace-id>`
- Project: `<project-id>`
- MCP Server: `sb`

### Auto-Sync Rules

When you make changes to any of the following, update the corresponding
Stable Baseline documentation using MCP tools:

| File Pattern | Documentation Impact | Action |
|---|---|---|
| <patterns specific to THIS repo> | <what docs to update> | <editDocument / createDocument> |

### MCP Tool Usage

- Use MCP tools for ALL document/folder/diagram/image operations
- Never manually write DIAGRAM_OMITTED, IMAGE_OMITTED, or marker HTML
- Always call `getDocument` before `editDocument` (need versionTimestamp)
- Create documents first, then insert diagrams/images with dedicated tools
- For bulk text changes use `findAndReplaceTextInDocument`

### Resources

- `sb://cdmd-guide` — CDMD syntax reference
- `sb://diagram-types` — Available diagram types
- `sb://server-info` — Server capabilities
```

The file patterns in the Auto-Sync Rules table MUST be specific to THIS repo.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 9 — Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

When done, provide a summary:
- Workspace and project IDs used
- Folder structure created (tree view)
- Documents created (with titles and brief descriptions)
- Diagrams inserted (list each diagram with its type and which document it belongs to)
- AGENTS.md location and key auto-sync rules
- `.sb/config.json` cache status — confirm all created folders and documents are cached
- Suggested next steps for the user

After the summary, update `.sb/config.json` `cache.lastUpdated` to the current ISO 8601 timestamp.
