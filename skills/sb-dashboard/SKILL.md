---
name: sb-dashboard
description: "Build and refine live dashboards on a Stable Baseline database table: suggested visuals, pages of cards, charts and tables, a picture of each page to check, and a PDF download. Use when user wants a dashboard, KPIs or charts over database rows, or run sb-dashboard."
---

# Stable Baseline: Dashboards

Build dashboards over Stable Baseline database tables using MCP tools. A dashboard is live: its numbers update as the rows change. For a chart inside a document, use the sb-create-diagram skill instead.

If you don't know IDs yet: read `.sb/config.json` (`cache.databases`), or find the database with `listDatabases` and read its tables with `getDatabase`.

Steps:
1) Read the table: `getDatabase` with `databaseId` and `table`. Note the field names and types. `getDatabase` also lists the database's dashboards: change an existing one rather than making a near-copy.
2) Get suggestions: `getDatabase` with `table` and `profile: true` (and `suggest`, up to 12) returns facts about each field and ready visuals, each with a `candidate` ref.
3) Plan the pages with the user: the questions each page answers, and the visuals for them. A good page leads with cards and KPIs, then trends and breakdowns, then a table for the detail. Keep each page to one purpose.
4) For a visual the suggestions do not cover: `listDashboardVisualTypes` (optionally one `group`), then `getDashboardVisualGuide` with the `type` for its wells, its settings and a worked example. Copy the example's spec and change it.
5) Save: `saveDatabaseDashboard` with `databaseId`, `name` and `pages`. Each page has a `name` and `visuals`; each visual is a `candidate` ref used as it is, or a `spec`, with an optional `title` and `size` (small, medium, large or full). `layout: "tidy"` lays a page out again in the order its visuals are listed. On a big change, call with `check: true` first: it checks everything and saves nothing. A dashboard only the user sees takes `personal: true`.
6) Look at every page you changed: `getDashboardImage` with `databaseId`, `dashboard` and `page`. Check the numbers are plausible, no visual is empty, titles fit, and the page reads in order. The answer names any visual with nothing to show.
7) Adjust and look again: `saveDatabaseDashboard` with `dashboard` and the `version` you read. A visual with an `id` is changed, or taken out with `remove`; one without an `id` is added. Repeat until the page looks right.
8) Report the dashboard's name and its link. If the user wants a file, `getDashboardImage` with `format: "pdf"` (and `pages: "all"` for every page) answers a download link.

Undo and reading:
- Every save's answer says how to undo it: `saveDatabaseDashboard` with `restore` (the version id from that answer) and `dashboard` puts it back.
- To read an existing dashboard first: `getDatabase` with `dashboard`, and `visual` for one visual's numbers.

Limits: a dashboard holds at most 20 pages (fewer on smaller plans) and 30 visuals (8 on Free), 24 on one page. One call takes 20 pages, so build a bigger dashboard over several calls.

Hard rules:
- Look before you report: call `getDashboardImage` after every save, for each page you changed.
- Never invent numbers or labels. Visuals work their numbers out from the rows; titles say what is shown.
- A shared dashboard is seen by everyone who can open the database. Use `personal: true` when the user wants one just for themselves.
- Ask before deleting a dashboard (`deleteDatabase` with `dashboard`; it waits in the trash for 30 days) or replacing a page someone built by hand.

Reference:
- `getDatabaseGuide` with topic dashboards, `listDashboardVisualTypes`, `getDashboardVisualGuide`
- Tables, fields and rows: the sb-database skill.
