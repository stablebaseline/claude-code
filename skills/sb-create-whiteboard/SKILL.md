---
name: sb-create-whiteboard
description: "Create a Stable Baseline whiteboard, either by hand (stencils, architecture icons, sticky notes, embedded diagrams, freedraw) or with the premium AI multi-agent designer. Use when user wants to create a whiteboard, design a board, sketch an architecture or flow visually, or run sb-create-whiteboard."
---

# Stable Baseline: Create Whiteboard

Create a freeform whiteboard in Stable Baseline using MCP.

If you don't know IDs yet: use `listWorkspaces` -> `listProjects` to discover them, or read `.sb/config.json` for cached IDs. Read `getWhiteboardGuide` (or the `sb://whiteboard-guide` resource) before authoring for the current element schema and best practices.

There are two ways to build a board. Pick based on what the user wants:

## A) AI multi-agent design (premium, fastest for a polished board)
- Call `autoDesignWhiteboard` with a clear goal in plain English. It composes the whole board, renders it and reviews the picture.
- COST + CONSENT: it costs credits (78 a board at the time of writing; the first call always returns the exact cost). The tool enforces consent itself with a two-call handshake, so follow it exactly:
  1. Call `autoDesignWhiteboard` with `{ goal }` and WITHOUT `confirm`. It returns the cost, the current credit balance and a single-use `quoteToken`. It does no work and spends nothing.
  2. Show the user the quote, and always offer the manual path in section B as the free alternative.
  3. Only after the user agrees, call again with the same arguments plus `confirm: true` and that `quoteToken`. The token works once, within 10 minutes.

  A call with `confirm: true` but without the `quoteToken`, or with arguments changed since the quote, starts nothing and charges nothing: it returns a fresh quote, which you show the user again. A call without `confirm` returns a quote and nothing else, so treating that as a failure is the most common mistake here.
- It runs in the BACKGROUND. The second call returns a `sessionId`, so tell the user it is building rather than waiting for a finished board in the response. A flowchart, mind map, org chart, kanban board, SWOT, timeline, wireframe and the other kinds `getWhiteboardGuide` lists fill in within seconds. Other boards take roughly 1 to 3 minutes. If the server fails part way, the credits are refunded automatically.
- With `documentId` it designs into an existing board and never moves or changes what is already there. The kinds above are drawn in a frame of their own beside it, and any other kind goes in free space below it. Clear the board only when the user asks, with `replaceExisting: true` (send it on the quote call too, so the quote says what will happen). Only those kinds can clear a board, and the result says whether it did: never promise a clear it does not report. `undoClear: true` with `confirm: true` and the `documentId` brings back what the newest design cleared, at no cost.
- To change a design that is already on the board, never start a new one. Call again with `changeDesign: true`, the `documentId`, the change in the user's own words as the `goal` (for "make it all green" the goal is "make it all green") and the `designId` from that design's result (the newest design if you leave it out). A change to how it looks alone is free: call it with `confirm: true` and no `quoteToken`. A change to what it holds is quoted and confirmed like a design. `undoChange: true` with `confirm: true` and the `documentId` puts the design back as it was, at no cost.
- Keep the `goal` to what should be drawn, and say nothing in it about what is already on the board or where the design goes. It may say how the design should look (colours, compact or spacious, left to right or top to bottom). A design carries figures only when the goal gives them or asks for figures, numbers, data or examples.
- Useful optional arguments: `title`, `projectId`, `documentId`, `brandKitId`, and `designProfile`, which is one of `standard` (default), `branded-executive`, `illustrated`, `image`, `agentic` or `agentic-deck`.
- `sourceTranscript` bills differently: 7 credits for every 5 minutes of transcript, with a 10-minute minimum, not the flat board price. The first call quotes it.

## B) Manual authoring (free, full control)
1) `createWhiteboard` with { projectId, title }. The title is required.
2) Add content with `addWhiteboardElements`. Prefer the RICHEST representation that fits, not plain rectangles:
   - Stencils: browse with `listWhiteboardStencils`, then place symbols and templates.
   - Architecture icons: look them up with `listArchitectureIcons` (one query per technology, e.g. "docker", "aws s3") and use the returned `iconPath` EXACTLY, or for a library icon with no `iconPath` its `iconUrl` as `imageUrl`. Do not invent icon paths.
   - Embedded diagrams: `insertWhiteboardDiagram` drops a rendered diagram (Mermaid, D2, infographic, etc.) onto the board.
   - Images: `insertWhiteboardImage`.
   - Sticky notes, shapes, connectors, and freedraw for annotation and emphasis.
3) Render with `getWhiteboardImage` to verify, then iterate.

## Hard rules
- Whiteboard titles are mandatory.
- Never send `confirm: true` to `autoDesignWhiteboard` before the user has seen the quote and agreed to its cost, and always send the `quoteToken` from that quote with it. The free calls are the exception: a look-only change, `undoClear` and `undoChange` take `confirm: true` without a token.
- After creating, cache the board's title -> id in `.sb/config.json` (`cache.whiteboards`) and set `cache.lastUpdated` to the current ISO 8601 timestamp.

## Reference
- `getWhiteboardGuide` / `sb://whiteboard-guide`
- `listWhiteboardStencils`, `listArchitectureIcons`
