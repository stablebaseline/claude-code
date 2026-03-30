---
name: sb-manage-images
description: "Upload, insert, update, or delete images in a Stable Baseline document using the upload-session flow. Use when user wants to manage images, add screenshots, or run sb-manage-images."
---

# Stable Baseline — Manage Images

$ARGUMENTS

Manage images in a Stable Baseline document using MCP tools.

If you don't know the documentId: use `listWorkspaces` → `listProjects` → `listDocuments` or `getProjectHierarchy` to find it. Or read `.sb/config.json` for cached IDs.

Supported formats: PNG, JPEG, GIF, WebP, SVG (max 10MB).

## Inserting a new image (recommended flow)

1) Call `getDocument` to get the current `versionTimestamp` and line numbers.

2) **Upload-session flow** (best practice — avoids base64 issues):
   a. Call `createImageUploadSession` with `{ documentId, fileName, mimeType }`.
      → Returns `uploadUrl` and `assetUrl`.
   b. PUT the raw file bytes to `uploadUrl` with header `Content-Type: <mimeType>`.
      The URL is single-use and expires in 5 minutes.
   c. Call `insertImageInDocument` with `imageUrl` set to the `assetUrl` from step (a).
      Include: `documentVersionTimestamp`, `nlDescription`, `caption`, `afterLine`.
      The tool will finalize the already-uploaded file without re-uploading.

3) **Alternative: direct insert** (for small images or URLs):
   - Call `insertImageInDocument` with one of:
     - `imageUrl`: a public URL to fetch from
     - `imageBase64`: base64-encoded image data
   - Include: `documentVersionTimestamp`, `nlDescription`, `caption`, `afterLine`.

## Updating image metadata

- Call `getDocument` to find the IMAGE_OMITTED marker and note the `imageId`.
- Call `updateImageInDocument` with the `imageId`, `documentVersionTimestamp`, and fields to change:
  `alt`, `caption`, `nlDescription`, `width`, `height`, `align`.
- This does NOT replace the image file — to swap the image, delete and re-insert.

## Deleting an image

- Call `getDocument` to find the IMAGE_OMITTED marker and note the `imageId`.
- Call `deleteImageInDocument` with the `imageId`.
- This removes the marker, database record, and storage file.

## Replacing an image

1) Delete the existing image (`deleteImageInDocument`).
2) Re-read the document (`getDocument`) to get the updated versionTimestamp and line numbers.
3) Insert the new image at the desired position using the upload-session flow above.

Hard rules:
- NEVER hand-edit IMAGE_OMITTED markers in CDMD.
- ALWAYS use `getDocument` before any operation to get the current `versionTimestamp`.
- If an operation fails due to versionTimestamp mismatch, re-read the document and retry.
- Write descriptive `nlDescription` values — they power semantic search.
