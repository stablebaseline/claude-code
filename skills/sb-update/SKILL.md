---
name: sb-update
description: "Update Stable Baseline documentation with changes, decisions, and knowledge from the current conversation. Use mid-conversation when architectural changes, key decisions, or important implementation details need to be captured. Use when user says sb-update, update docs, sync docs, or capture changes."
---

# Stable Baseline — Update Documentation

$ARGUMENTS

You are updating Stable Baseline documentation to reflect changes, decisions, or knowledge from this conversation.

Follow this workflow EXACTLY:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 1 — Read Configuration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Read `.sb/config.json` to get the workspaceId, projectId, AND the `cache` object (`cache.documents`, `cache.folders`, `cache.lastUpdated`). If the file doesn't exist, ask the user which workspace and project to update.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 2 — Analyse Conversation Context
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Review what was changed or decided in this conversation. Identify:
- **Code changes**: Files modified, features added, bugs fixed, refactors performed
- **Architectural decisions**: Design choices, tradeoffs considered, patterns adopted or rejected
- **Key decisions**: Why something was done a certain way (the rationale, not just the what)
- **New knowledge**: Discovered constraints, gotchas, limitations, or important context
- **Schema changes**: Database migrations, new tables/columns, RLS policy changes
- **API changes**: New endpoints, modified contracts, auth flow changes

For each item, determine which Stable Baseline document it belongs to.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 3 — Discover Existing Documentation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Use a **cache-first** approach to find existing documents:

1. Check `cache.documents` from `.sb/config.json` for each change area from Step 2.
   - **Cache HIT**: Call `getDocument` directly with the cached document ID — skip `getProjectHierarchy`.
   - **Cache MISS or 404**: Fall back to `getProjectHierarchy` with the projectId, find the document, then update `cache.documents` with the discovered ID.

2. If `cache.documents` is empty or missing, call `getProjectHierarchy` once, map all documents, and populate the cache.

Map each change area from Step 2 to an existing document. If no document exists for a change area, note it for creation in Step 5.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 4 — Update Existing Documents
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

For each document that needs updating:

1. Call `getDocument` to read the current content and get the `versionTimestamp` (required for optimistic locking)
2. Determine what sections need to be added, updated, or expanded
3. Call `editDocument` with **targeted patches** — NOT full rewrites

Rules for editing:
- **Patch, don't rewrite**: Only change the sections that are affected. Leave everything else untouched.
- **Capture rationale**: For architectural decisions, document WHY the decision was made, what alternatives were considered, and what tradeoffs were accepted.
- **Be specific**: Include file paths, function names, configuration keys — concrete references that a developer can follow.
- **Update, don't duplicate**: If the document already covers a topic, update that section in-place rather than adding a new section that repeats it.
- **Timestamp decisions**: For significant architectural decisions, note when they were made (use the current date).
- **Update cache**: After all edits in this step, update `.sb/config.json` `cache.lastUpdated` to the current ISO 8601 timestamp.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 5 — Create New Documents (if needed)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

If a change area has no matching document:

1. Identify the most appropriate existing folder for the new document
2. Call `createDocument` in that folder with well-structured content
3. Follow CDMD format (read `sb://cdmd-guide` if unsure about syntax)
4. Update `.sb/config.json` `cache.documents` with the new document's title → `{ "id": "<doc-id>", "folderId": "<folder-id-or-null>" }` mapping

Only create new documents for substantial new areas. Minor changes should be folded into existing documents.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 6 — Generate or Update Diagrams (if applicable)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

If the changes affect system architecture, data flow, component relationships, or any visual structure:

1. Check if existing diagrams need updating (read `sb://diagram-types` for available types)
2. Use `insertDiagramInDocument` to add new diagrams where they aid understanding
3. Prefer updating existing diagrams over creating duplicates

Diagram types to consider:
- **Architecture changes** → C4 Context/Container diagrams, component diagrams
- **Data model changes** → Entity-relationship diagrams (ERD)
- **Flow changes** → Sequence diagrams, flowcharts, BPMN
- **API changes** → Sequence diagrams showing request/response flows

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STEP 7 — Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

After all updates are complete, provide a summary:
- List each document updated (with document name and what was changed)
- List any new documents created
- List any diagrams added or updated
- Note any areas that may need future documentation but weren't covered

IMPORTANT RULES:
- Use `sb` tools for ALL operations
- ALWAYS call `getDocument` before `editDocument` (versionTimestamp is required)
- NEVER manually write or edit `DIAGRAM_OMITTED`, `IMAGE_OMITTED`, or `<!-- DIAGRAM: ... -->` markers — use the dedicated diagram/image tools
- Prefer `editDocument` with small patches over full document rewrites
- For bulk text substitutions use `findAndReplaceTextInDocument`
- Document the "why" behind decisions, not just the "what"
