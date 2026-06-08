---
name: sb-create-whiteboard
description: "Create a Stable Baseline whiteboard, either by hand (stencils, architecture icons, sticky notes, embedded diagrams, freedraw) or with the premium AI multi-agent designer. Use when user wants to create a whiteboard, design a board, sketch an architecture or flow visually, or run sb-create-whiteboard."
---

# Stable Baseline — Create Whiteboard

$ARGUMENTS

Create a freeform whiteboard in Stable Baseline using MCP.

If you don't know IDs yet: use `listWorkspaces` -> `listProjects` to discover them, or read `.sb/config.json` for cached IDs. Read `getWhiteboardGuide` (or the `sb://whiteboard-guide` resource) before authoring for the current element schema and best practices.

There are two ways to build a board. Pick based on what the user wants:

## A) AI multi-agent design (premium, fastest for a polished board)
- Call `designWhiteboard` with a clear goal in plain English. It runs a planner/layouter/builder/critic team and returns a finished, rendered board.
- COST + CONSENT: `designWhiteboard` costs 50 credits per run. State the cost and get explicit user confirmation BEFORE calling it. Always offer the manual path (free) as an alternative.
- If unsure there are enough credits, check `getCreditBalance` first.

## B) Manual authoring (free, full control)
1) `createWhiteboard` with { projectId, title }. The title is required.
2) Add content with `addWhiteboardElements`. Prefer the RICHEST representation that fits, not plain rectangles:
   - Stencils: browse with `listWhiteboardStencils`, then place symbols and templates.
   - Architecture icons: look them up with `listArchitectureIcons` (one query per technology, e.g. "docker", "aws s3") and use the returned `iconPath` EXACTLY. Do not invent icon paths.
   - Embedded diagrams: `insertWhiteboardDiagram` drops a rendered diagram (Mermaid, D2, infographic, etc.) onto the board.
   - Images: `insertWhiteboardImage`.
   - Sticky notes, shapes, connectors, and freedraw for annotation and emphasis.
3) Render with `getWhiteboardImage` to verify, then iterate.

## Hard rules
- Whiteboard titles are mandatory.
- Never call `designWhiteboard` without first confirming the 50-credit cost with the user.
- After creating, cache the board's title -> id in `.sb/config.json` (`cache.whiteboards`) and set `cache.lastUpdated` to the current ISO 8601 timestamp.

## Reference
- `getWhiteboardGuide` / `sb://whiteboard-guide`
- `listWhiteboardStencils`, `listArchitectureIcons`
