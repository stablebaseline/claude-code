---
name: sb-search
description: "Search the Stable Baseline knowledge graph (the shared company brain) to find documents, answer questions, surface related content, and look up entities across the workspace. Use when user wants to search docs, ask the company brain, find related documents, or run sb-search."
---

# Stable Baseline — Search the Knowledge Graph


Every document, diagram, plan, and board in Stable Baseline feeds a self-learning knowledge graph that is shared across the workspace. Use it to answer questions and find content from Claude Code.

Steps:
1) For a natural-language question or topic, call `kg_search` (semantic search) with the query.
2) To read a synthesized wiki page for a topic or entity, call `kg_get_wiki_page`.
3) To find documents related to one you already have, call `kg_related_documents`.
4) For a specific entity, call `kg_get_entity`; for what links to it, call `kg_backlinks`.
5) If you only need to locate a document by title or folder, `listDocuments` and the project-hierarchy resource are faster than semantic search.
6) Summarize the findings and cite the source documents (title + id) so the user can open them.

## Notes
- The knowledge graph is scoped to the caller's workspace and permissions.
- If `kg_search` returns little, the graph may still be building for recently added docs; fall back to `listDocuments`.
- `kg_suggest_sample_questions` offers good starting questions for a workspace.

## Reference
- `kg_search`, `kg_get_wiki_page`, `kg_related_documents`, `kg_get_entity`
