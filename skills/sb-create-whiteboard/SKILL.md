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
- Call `autoDesignWhiteboard` with a clear goal in plain English. It runs a planner/layouter/builder/critic team and returns a finished, rendered board.
- COST + CONSENT: it costs 50 credits per run. The tool enforces this itself with a two-call handshake, so follow it exactly:
  1. Call `autoDesignWhiteboard` with `{ goal }` and WITHOUT `confirm`. It returns a cost quote and the current credit balance. It does no work and spends nothing.
  2. Show the user the quote, and always offer the manual path in section B as the free alternative.
  3. Only after the user agrees, call again with the same arguments plus `confirm: true`.

  A single call with `confirm: true` up front skips the user's decision. A single call without it returns a quote and nothing else, so treating that as a failure is the most common mistake here.
- It runs in the BACKGROUND. The second call returns a `sessionId` and the board fills in over roughly 1 to 3 minutes, so tell the user it is building rather than waiting for a finished board in the response. If the server fails part way, the credits are refunded automatically.
- Useful optional arguments: `title`, `projectId`, `documentId`, `brandKitId`, and `designProfile`, which is one of `standard` (default), `branded-executive`, `illustrated`, `image`, `agentic` or `agentic-deck`.
- `sourceTranscript` bills differently: 2 credits per minute with a 10-minute minimum, NOT the flat 50.

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
- Never send `confirm: true` to `autoDesignWhiteboard` before the user has seen the quote and agreed to the 50-credit cost.
- After creating, cache the board's title -> id in `.sb/config.json` (`cache.whiteboards`) and set `cache.lastUpdated` to the current ISO 8601 timestamp.

## Reference
- `getWhiteboardGuide` / `sb://whiteboard-guide`
- `listWhiteboardStencils`, `listArchitectureIcons`
