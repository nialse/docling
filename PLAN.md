# Plan for Implementing MBOX File Support

## Overview
Docling currently supports a variety of document formats. To extend its capabilities we plan to add support for parsing `mbox` e‑mail archives. Each e‑mail should become a `DoclingDocument` including attachments and metadata.

## Architecture
1. **InputFormat extension**
   - Add `InputFormat.MBOX` to `docling.datamodel.base_models.InputFormat`.
   - Update `FormatToExtensions`, `FormatToMimeType` and `MimeTypeToFormat` mappings accordingly (e.g. `"mbox"`, `"application/mbox"`, `"application/x-mbox"`).
2. **Backend implementation**
   - Create `docling/backend/mbox_backend.py` implementing `DeclarativeDocumentBackend`.
   - Use Python’s `mailbox` and `email` libraries for parsing.
   - Each e‑mail in the archive will be converted into a `DoclingDocument`.
3. **Metadata handling**
   - Extract headers such as `From`, `To`, `Cc`, `Bcc`, `Subject`, `Date`, `Message‑Id`, `In‑Reply‑To` and store them in a metadata block. The metadata can be placed inside `DocumentOrigin` or a new metadata structure if available.
4. **Attachment processing**
   - For each MIME part with a `Content-Disposition: attachment`:
     - Save the payload to a temporary file or `BytesIO`.
     - Determine its format via existing `InputFormat` detection.
     - Recursively invoke `DocumentConverter` to parse the attachment when possible.
   - Represent attachments in the e‑mail document as children under a dedicated group (e.g. `GroupLabel.ATTACHMENTS`).
5. **Pipeline integration**
   - Register the backend and default pipeline in `DocumentConverter` so `InputFormat.MBOX` can be used seamlessly.
   - Support pagination if needed (one page per e‑mail or treat each message as a page).

## Implementation Steps
1. **Model updates**
   - Extend `InputFormat` enumeration and mapping dictionaries.
   - Add unit tests verifying that mbox files are recognized and mapped correctly.
2. **Backend creation**
   - Implement `MboxDocumentBackend` with methods `is_valid`, `supported_formats`, `convert`, and `unload`.
   - During conversion iterate over messages, create a new `DoclingDocument` per message, extract body text and populate metadata.
   - Add optional flag to include/exclude attachments.
3. **Attachment conversion utility**
   - Write helper to convert MIME attachments into `InputDocument` instances.
   - Use `DocumentConverter` to parse them; fallback to storing raw bytes if unsupported.
4. **Metadata enrichment**
   - Define function `_add_metadata(doc, message)` to insert header information at the start of the document (similar to JATS backend).
5. **Update documentation**
   - Document new format in `docs/usage/supported_formats.md` and provide an example.
6. **Testing**
   - Create sample `.mbox` file with a few messages and attachments under `tests/data`.
   - Add end‑to‑end tests ensuring messages and attachments are correctly parsed.
7. **Release tasks**
   - Update `CHANGELOG.md` once implementation is ready.

## Technology Choices
- **mailbox** / **email** modules from Python standard library for parsing.
- Reuse existing `DoclingDocument` and converter APIs to minimize duplication.
- Temporary storage for attachments via `tempfile` or in‑memory streams.
- Unit tests with `pytest` following existing patterns.

This plan ensures that e‑mails contained in mbox archives become first‑class inputs for Docling while preserving metadata and attachments for further processing.
