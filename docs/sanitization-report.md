# Sanitization report

## Scope

This report covers the files prepared for the public portfolio review copy. The repository has not been published by this preparation step.

## Replaced in workflow exports

The three n8n JSON files were exported from the working local instance and then sanitized. The review copy replaces or removes:

- Google Drive and Google Sheets credential IDs and credential names;
- Gemini credential IDs and credential names;
- Google Drive intake folder ID and URL;
- Google Sheets document ID and URL;
- Google Sheets tab IDs and cached URLs;
- form webhook IDs;
- n8n workflow IDs, active-version IDs, version IDs, and source-workflow IDs;
- n8n project ownership, creator, last-updated-by, and sharing metadata;
- personal name and email metadata;
- execution/static/pinned data and cached result URLs.

Credentials and Google resources are represented by `REPLACE_WITH_YOUR_...` placeholders. Workflows are marked inactive.

## Excluded from the repository

- Gemini API keys, Google client secrets, OAuth tokens, and n8n credential exports
- Source videos and proxy media
- The capstone slide deck because its embedded browser screenshots included temporary signed localhost form URLs
- Raw search-result and no-match screenshots containing temporary form signatures
- A Clip Catalog screenshot containing a child's spoken name, location, and age
- A raw Gemini-response screenshot containing an unnecessary opaque thought signature
- Internal build scripts with absolute local user paths
- Inspection artifacts, local logs, and unrelated notes

## Included screenshots

The included screenshots were selected because they show project evidence without the known private IDs or signed browser URLs:

- footage logger workflow
- footage search workflow
- structured AI output
- human review queue
- validation Error Log

## Automated audit checks

Before review, the repository is scanned for:

- email-address patterns;
- personal name and known account strings;
- Google Drive/Docs URLs;
- localhost URLs and form signatures;
- known workflow, credential, project, folder, spreadsheet, and webhook identifiers;
- malformed JSON;
- broken Markdown image links;
- README-listed files that do not exist.

Automated scans reduce risk but do not replace a manual review before publication.

## Publication checklist

- [ ] Confirm repository name and owner.
- [ ] Decide whether to add an open-source license.
- [ ] Confirm whether the GitHub profile should display a personal name or only `cutthatclip`.
- [ ] Review every included image at full resolution.
- [ ] Search the final repository for secrets and personal data again.
- [ ] Import sanitized workflows into a disposable n8n instance if possible.
- [ ] Confirm no private source footage is added later.
- [ ] Obtain explicit approval before creating or publishing the GitHub repository.

