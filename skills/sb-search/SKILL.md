---
name: sb-search
description: "Find anything in Stable Baseline by its name or id, count things in one call, find what a person owns or created, see what changed this week, and ask the knowledge graph (the shared company brain) questions with cited answers. Use when user wants to search docs, find a document, plan, task, board or database by name, count documents, tasks or plans, list their own tasks, ask the company brain, see what changed, find related documents, or run sb-search."
---

# Stable Baseline: Search

Find content in Stable Baseline and answer questions from it using MCP tools. Every document, diagram, plan and board can feed the organisation's knowledge graph, where it is switched on.

Steps:
1) To FIND something by its name, friendly id (DOC-123, PLN-4, DB-12) or uuid, of any kind: `kg_search` with `artefactMetadataOnly: true`, narrowed with `artefactTypes` (document, whiteboard, plan, task, improvement, compliance, database, row) when you know the kind. It works on every plan, with or without the knowledge graph.
2) To search inside documents: `listDocuments` with `query`. It greps every document body and returns `contentMatches` with line numbers; use `isRegex` for a pattern.
3) To ANSWER a question from the content: `kg_search` without that flag, with `projectId` or `workspaceId`. Use `local` for a specific fact, `global` for themes and overviews, `graph` and `path` for how things connect. For how something stood on a past day, or how it changed, add `asOf` (a day) or `history: true`, with the user's `timeZone`; they cover database records too. These knowledge modes need the organisation's knowledge graph (Enterprise plans); where it is off the call says so, so fall back to step 2.
4) To see what changed: `listDocuments`, `listWhiteboards`, `listImprovements` and `listPlans` with `changedSince` (today, this_week, last_7_days, a date and so on) and the user's `timeZone`, scoped with `projectId`, `workspaceId` or neither for the whole organisation. For how one document changed: `listDocumentVersions` with `documentId`, `changedSince` and `compare: true`.
5) To COUNT ("how many"): add `countOnly: true` to `listDocuments`, `listImprovements`, `listPlans`, `listTasks` or `listMembers`. It answers just `{ count }`, for every filter the same call would apply, however many it would list. It counts only what the user may open. Never page through a list to count it.
6) To list what a person made or owns: `createdBy` on `listDocuments`, `listImprovements`, `listPlans` and `listTasks`, and `ownerId` on `listImprovements` and `listTasks` (a plan has no owner). Pass the person's user id (from `listMembers` or `listAssignablePrincipals`), or "me" for the user: "my tasks" is `ownerId: "me"`. A task owned by a team does not match a person's `ownerId`. Someone outside the organisation matches nothing, and the answer is empty, not an error. These filters work with `countOnly` and with `changedSince`.
7) For tasks and plans: `getTask`, `listTasks`, `getPlan` and `getPlanHierarchy` name each task's owner (`owner_name`, beside `owner_id`) and carry `start_date`, `end_date` and `target_date`. A task's due date is its `end_date`, or its `target_date` when it has no end date. An empty owner can mean a team owns the task (`getTask` shows `owner_team_id`). Say a task has no due date only when both dates are empty.
8) For related content: `kg_related_documents`, `kg_backlinks`, `kg_get_entity` and `kg_get_wiki_page`; for good starting questions, `kg_suggest_sample_questions`.
9) Summarise the findings and cite each source (its title and the link the tool returned) so the user can open it.

## Notes
- Results are limited to what the caller may open, and to what the API key reaches.
- Counts, totals and rows of a database table: the sb-database skill.
- If `kg_search` returns little, the graph may still be building for recently added docs; fall back to `listDocuments`.

## Reference
- `kg_search`, `listDocuments`, `listDocumentVersions`, `kg_get_wiki_page`, `kg_related_documents`, `kg_get_entity`
- `listImprovements`, `listPlans`, `listTasks`, `listMembers`, `getTask`, `getPlan`, `getPlanHierarchy`
