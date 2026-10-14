---
name: sb-create-diagram
description: "Generate and insert a diagram into an existing Stable Baseline document. Use when user wants to create a diagram, add a diagram, insert a chart, or run sb-create-diagram."
---

# Stable Baseline: Create Diagram

Insert a diagram into a Stable Baseline document using MCP.

Steps:
1) If documentId is missing: ask for it or locate via `listDocuments` / hierarchy resources. Or read `.sb/config.json` for cached IDs.
2) Call `getDocument` to get the current versionTimestamp and line numbering.
3) Decide diagram type:
   - If user specified a type, use it.
   - Otherwise call `listDiagramTypes` with a `query` for the intent. Each type lists the diagram families it draws, each with its default renderer: use a family's default renderer as the type unless the user named a language. `default` is the platform default renderer, for architecture, process, org chart and other box-and-line diagrams.
4) Call `getDiagramTypeGuide` for DSL instructions (it takes a type or a family's name).
5) **Icon lookup (for any diagram with icons)**:
   - Before writing any DSL that uses icons, call `listArchitectureIcons` with a `query` for each technology/service you plan to include (e.g. query="docker", query="aws ec2").
   - Use the field the answer's `usage` names for your diagram type, exactly as returned: `iconKey` as the icon in a platform default diagram (`default`), `d2IconPath` in D2. For another type, follow the icon example `getDiagramTypeGuide` gives. Do NOT guess or invent icons.
   - A result without an `iconKey` cannot go in a platform default diagram.
   - If you cannot find an icon for a node, leave its icon out rather than guessing; an unknown icon is refused or drawn without its picture.
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
