---
name: sb-edit-doc
description: "Edit an existing Stable Baseline document with anchor or line patches, or bulk find-and-replace, with proper optimistic locking. Use when user wants to edit a document, update docs, or run sb-edit-doc."
---

# Stable Baseline: Edit Document

Edit an existing Stable Baseline document using MCP tools.

If you don't know the documentId: use `listWorkspaces` -> `listProjects` -> `listDocuments` or `getProjectHierarchy` to find it. Or read `.sb/config.json` for cached IDs. `listDocuments` with `query` also finds the text to change, line by line.

Choose the right approach:

**A) Anchor patches** (`editDocument`, recommended): replace an exact snippet of the document. They do not drift when someone else edits the document, so no fresh line numbers are needed.
**B) Line patches** (`editDocument`): replace, insert or delete whole lines by number, when an anchor will not do.
**C) Bulk find-and-replace** (`findAndReplaceTextInDocument`): rename a term, fix a repeated typo or update a brand name across the whole document.
**D) Title or folder move only**: pass an empty patches array with the new title or folderId.

Never mix anchor patches and line patches in one call.

Steps:

1) Call `getDocument` with the documentId (or use the `versionTimestamp` a `listDocuments` result carries).
   - Note the `versionTimestamp`: you MUST pass it on every edit call.
   - Read the line-numbered content to find what to change. The number and tab before each line are for display only.

2) For **anchor patches** (`editDocument`):
   - Each patch is `{ oldText, newText }`. `oldText` must match the document exactly and occur once.
   - If it occurs more than once, make `oldText` longer, or add `before` or `after` (the exact text just before or after it).
   - `newText: ""` deletes the anchored text.
   - The `text` of a `listDocuments` content match can be pasted straight in as `oldText`.
   - A no-match answers with nearby text and an ambiguous match with the count: read it and fix the anchor rather than retrying blindly.

3) For **line patches** (`editDocument`):
   - Each patch has `startLine`, `endLine` (1-based, inclusive), and `replacement`.
   - To replace line 5: `{ startLine: 5, endLine: 5, replacement: "new content" }`
   - To delete lines 10 to 12: `{ startLine: 10, endLine: 12, replacement: "" }`
   - To insert after line 7: `{ startLine: 8, endLine: 7, replacement: "inserted content" }`
   - NEVER include line number prefixes in your replacement text.

4) For **bulk find-and-replace** (`findAndReplaceTextInDocument`):
   - Provide `find` (exact text to match) and `replace` (replacement text).
   - Case-sensitive by default; set `caseSensitive: false` if needed.
   - It says how many it replaced.

5) For a large or structural edit, call `editDocument` with `dryRun: true` first and show the user the result.

6) If the edit fails with a versionTimestamp mismatch:
   - Re-call `getDocument` to get the latest versionTimestamp.
   - Re-apply your patches against the updated content.

Hard rules:
- ALWAYS read the current text first (`getDocument`, or `listDocuments` with `query`): never edit blind.
- NEVER hand-edit DIAGRAM_OMITTED or IMAGE_OMITTED markers. Use dedicated diagram/image tools.
- Keep patches small and precise. Avoid replacing large document sections unnecessarily.
- Include a `changeSummary` to document what changed (appears in version history).

Reference:
- `sb://cdmd-guide` for CDMD syntax
