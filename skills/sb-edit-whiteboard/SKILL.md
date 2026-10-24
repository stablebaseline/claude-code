---
name: sb-edit-whiteboard
description: "Update an existing Stable Baseline whiteboard: add, move, restyle, or remove elements, change or undo an AI design, embed diagrams or images, re-render, and export it as PowerPoint. Use when user wants to edit a whiteboard, update a board, change something on a board, turn a board into slides, or run sb-edit-whiteboard."
---

# Stable Baseline: Edit Whiteboard


Update an existing whiteboard in Stable Baseline using MCP.

Steps:
1) Resolve the whiteboardId: ask the user, find it via `listWhiteboards`, or read `.sb/config.json` (`cache.whiteboards`).
2) Call `getWhiteboard` to load the current scene (elements and their ids) so you edit against the live state, never stale ids.
3) Apply the changes:
   - Add elements: `addWhiteboardElements` (stencils, architecture icons, sticky notes, connectors, embedded diagrams via `insertWhiteboardDiagram`, images via `insertWhiteboardImage`).
   - Move, restyle, or delete existing elements, or replace the scene: `updateWhiteboardScene`. Pass `deleteIds` to remove elements; you do not need to re-send every element to delete or tweak a few.
   - Duplicate a group: `duplicateWhiteboardElements`.
   - A design `autoDesignWhiteboard` drew: to change how it looks or what it holds, or to undo a change or a clear, call `autoDesignWhiteboard` again with `changeDesign`, `undoChange` or `undoClear` (the sb-create-whiteboard skill has the rules and the cost). Never start a new design to change one that is already there.
4) Render with `getWhiteboardImage` to verify, then iterate.
5) To hand the board out as slides: `exportFromWhiteboard` with `documentId` and `kind: "canvas"` makes an editable PowerPoint in which each frame is a slide, in the board's slide order (a board with no frames is one slide). A large board finishes in the background: call again with the same arguments until `status` is `completed`. The download link lasts an hour.

## Hard rules
- Always `getWhiteboard` first and edit against current element ids.
- Read `getWhiteboardGuide` for the element schema if unsure.
- Preserve existing content unless the user asked to remove it.

## Reference
- `getWhiteboardGuide` / `sb://whiteboard-guide`
