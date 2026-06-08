---
name: sb-edit-whiteboard
description: "Update an existing Stable Baseline whiteboard: add, move, restyle, or remove elements, embed diagrams or images, and re-render. Use when user wants to edit a whiteboard, update a board, change something on a board, or run sb-edit-whiteboard."
---

# Stable Baseline — Edit Whiteboard

$ARGUMENTS

Update an existing whiteboard in Stable Baseline using MCP.

Steps:
1) Resolve the whiteboardId: ask the user, find it via `listWhiteboards`, or read `.sb/config.json` (`cache.whiteboards`).
2) Call `getWhiteboard` to load the current scene (elements and their ids) so you edit against the live state, never stale ids.
3) Apply the changes:
   - Add elements: `addWhiteboardElements` (stencils, architecture icons, sticky notes, connectors, embedded diagrams via `insertWhiteboardDiagram`, images via `insertWhiteboardImage`).
   - Move, restyle, or delete existing elements, or replace the scene: `updateWhiteboardScene`. Pass `deleteIds` to remove elements; you do not need to re-send every element to delete or tweak a few.
   - Duplicate a group: `duplicateWhiteboardElements`.
4) Render with `getWhiteboardImage` to verify, then iterate.

## Hard rules
- Always `getWhiteboard` first and edit against current element ids.
- Read `getWhiteboardGuide` for the element schema if unsure.
- Preserve existing content unless the user asked to remove it.

## Reference
- `getWhiteboardGuide` / `sb://whiteboard-guide`
