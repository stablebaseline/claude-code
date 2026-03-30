---
name: sb-create-diagram
description: "Generate and insert a diagram into an existing Stable Baseline document. Use when user wants to create a diagram, add a diagram, insert a chart, or run sb-create-diagram."
---

# Stable Baseline — Create Diagram

$ARGUMENTS

Insert a diagram into a Stable Baseline document using MCP.

Steps:
1) If documentId is missing: ask for it or locate via `listDocuments` / hierarchy resources. Or read `.sb/config.json` for cached IDs.
2) Call `getDocument` to get the current versionTimestamp and line numbering.
3) Decide diagram type:
   - If user specified a type, use it.
   - Otherwise call `listDiagramTypes` and pick the best match.
4) Call `getDiagramTypeGuide` for DSL instructions.
5) **Icon lookup (required for systemsarchitecture / d2)**:
   - Before writing any DSL that uses icons, call `listArchitectureIcons` with a `query` for each technology/service you plan to include (e.g. query="docker", query="aws ec2").
   - Use the `iconPath` values returned EXACTLY as-is in your `icon:` fields. Do NOT guess or invent icon paths.
   - If you cannot find an icon for a node, omit the `icon:` property rather than guessing.
   - The server will reject diagrams with invalid icon paths, so always look up icons first.
6) Generate `diagramCode`.
7) Call `insertDiagramInDocument` with:
   - documentId
   - documentVersionTimestamp (from getDocument)
   - type
   - diagramCode
   - prompt + nlDescription
   - afterLine (choose sensible placement if not provided)

Hard rules:
- Do not hand-edit diagram markers in CDMD.
- If insertion fails due to versionTimestamp mismatch, re-run `getDocument` and retry with the new timestamp.

Reference:
- `sb://diagram-types`
