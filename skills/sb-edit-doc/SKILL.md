---
name: sb-edit-doc
description: "Edit an existing Stable Baseline document using line-based patches or bulk find-and-replace, with proper optimistic locking. Use when user wants to edit a document, update docs, or run sb-edit-doc."
---

# Stable Baseline — Edit Document

$ARGUMENTS

Edit an existing Stable Baseline document using MCP tools.

If you don't know the documentId: use `listWorkspaces` → `listProjects` → `listDocuments` or `getProjectHierarchy` to find it. Or read `.sb/config.json` for cached IDs.

Choose the right approach:

**A) Line-based patches** (`editDocument`) — for targeted edits: replacing specific lines, inserting new sections, deleting blocks.
**B) Bulk find-and-replace** (`findAndReplaceTextInDocument`) — for renaming terms, fixing typos, or updating brand names across the entire document.
**C) Title/folder move only** — pass empty patches array with the new title or folderId.

Steps:

1) Call `getDocument` with the documentId.
   - Note the `versionTimestamp` — you MUST pass it on every edit call.
   - Read the line-numbered content to identify what to change.

2) For **line-based patches** (`editDocument`):
   - Each patch has `startLine`, `endLine` (1-based, inclusive), and `replacement`.
   - To replace line 5: `{ startLine: 5, endLine: 5, replacement: "new content" }`
   - To delete lines 10-12: `{ startLine: 10, endLine: 12, replacement: "" }`
   - To insert after line 7: `{ startLine: 8, endLine: 7, replacement: "inserted content" }`
   - Prefer small, targeted patches. Do NOT replace the entire document.
   - NEVER include line number prefixes in your replacement text.

3) For **bulk find-and-replace** (`findAndReplaceTextInDocument`):
   - Provide `find` (exact text to match) and `replace` (replacement text).
   - Case-sensitive by default; set `caseSensitive: false` if needed.
   - Good for: renaming variables, updating URLs, fixing repeated typos.

4) If the edit fails with a versionTimestamp mismatch:
   - Re-call `getDocument` to get the latest versionTimestamp.
   - Re-apply your patches against the updated content.

Hard rules:
- ALWAYS call `getDocument` first — never edit blind.
- NEVER hand-edit DIAGRAM_OMITTED or IMAGE_OMITTED markers. Use dedicated diagram/image tools.
- Keep patches small and precise. Avoid replacing large document sections unnecessarily.
- Include a `changeSummary` to document what changed (appears in version history).

Reference:
- `sb://cdmd-guide` for CDMD syntax
