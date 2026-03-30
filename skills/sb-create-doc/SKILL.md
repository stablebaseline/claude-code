---
name: sb-create-doc
description: "Create a new Stable Baseline document (CDMD) with optional follow-up diagram/image insertion. Use when user wants to create a document, write docs, or run sb-create-doc."
---

# Stable Baseline — Create Document

$ARGUMENTS

Create a Stable Baseline document using MCP tools.

If you don't know IDs yet: use `listWorkspaces` → `listProjects` → `listFolders` to discover them. Or read `.sb/config.json` for cached IDs.

Rules:
- Write CDMD. Keep it readable and structured.
- Do NOT include any diagram/image markers in the `cdmd` passed to `createDocument`.
- If the user wants diagrams: create the doc first, then call `insertDiagramInDocument`.
- If the user wants images: create the doc first, then call `insertImageInDocument` (or use `createImageUploadSession` + PUT + `insertImageInDocument`).

Steps:
1) Call `createDocument` with { projectId, folderId (optional), title, cdmd }.
2) Update `.sb/config.json` `cache.documents` with the new document's title → `{ "id": "<doc-id>", "folderId": "<folder-id-or-null>" }` mapping, and set `cache.lastUpdated` to the current ISO 8601 timestamp.
3) If diagrams are requested:
   - Call `listDiagramTypes` / `getDiagramTypeGuide`
   - Call `insertDiagramInDocument` with afterLine placements.
4) If images are requested:
   - Prefer upload-session flow.

Reference resources:
- `sb://cdmd-guide` for syntax
