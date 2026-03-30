---
name: sb-manage-data
description: "Upload, reference, and manage data files (CSV, JSON, TSV) for Vega/Vega-Lite diagrams in a Stable Baseline document. Use when user wants to upload data files, create charts with data, or run sb-manage-data."
---

# Stable Baseline — Manage Data Files

$ARGUMENTS

Manage data files for Vega/Vega-Lite diagrams in a Stable Baseline document.

If you don't know the documentId: use `listWorkspaces` → `listProjects` → `listDocuments` or `getProjectHierarchy` to find it. Or read `.sb/config.json` for cached IDs.

Data files are **sidebar attachments** — they are NOT embedded in the document body. They are referenced by URL in Vega/Vega-Lite diagram specs.

Supported formats: CSV (.csv), JSON (.json), TSV (.tsv), plain text (.txt). Max 10MB.

## Uploading a data file and creating a Vega diagram

1) Call `createVegaDataUploadSession` with `{ documentId, fileName }`.
   - Content type is auto-detected from the file extension (.csv → text/csv, etc.).
   - You can override with the `contentType` parameter if needed.
   → Returns `uploadUrl` and `assetUrl`.

2) PUT the raw file bytes to `uploadUrl` with header `Content-Type: <contentType>`.
   - The URL is single-use and expires in 5 minutes.

3) Use the `assetUrl` in your Vega or Vega-Lite spec:
   ```json
   {
     "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
     "data": { "url": "<assetUrl>" },
     "mark": "bar",
     "encoding": { ... }
   }
   ```

4) Call `insertDiagramInDocument` with:
   - `type`: the Vega or Vega-Lite diagram type (check `listDiagramTypes` for the exact type name)
   - `diagramCode`: the JSON spec referencing the assetUrl
   - `documentId`, `documentVersionTimestamp` (from `getDocument`)
   - `afterLine`, `prompt`, `nlDescription`, etc.

## Deleting a data file

- Call `deleteVegaDataFile` with `{ documentId, attachmentId }`.
- This removes the storage file and database record.
- NOTE: Any diagrams referencing the deleted file's `assetUrl` will break. Update or remove those diagrams first.

## Replacing a data file

1) Upload the new file using the upload-session flow above (step 1-2).
2) Update any existing Vega/Vega-Lite diagrams to point to the new `assetUrl`:
   - Call `getDocument` to find DIAGRAM_OMITTED markers.
   - Call `getDiagramInDocument` to read the current spec.
   - Call `updateDiagramInDocument` with updated `diagramCode` referencing the new assetUrl.
3) Delete the old data file with `deleteVegaDataFile`.

## Multiple data files in one diagram

A single Vega spec can reference multiple data sources. Upload each file separately, then reference all their assetUrls in the spec.

Hard rules:
- ALWAYS upload data files via `createVegaDataUploadSession` + PUT. Never embed large datasets inline in diagram code.
- The `assetUrl` is permanent and JWT-authenticated — it requires a valid user session to access.
- Data files are NOT part of the document body. They won't appear in `getDocument` output.
- Clean up unused data files with `deleteVegaDataFile` to avoid orphaned storage.

Reference:
- `sb://diagram-types` to find exact Vega/Vega-Lite type names
- `sb://cdmd-guide` for document syntax
