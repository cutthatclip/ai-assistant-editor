# Setup guide

The workflow exports are sanitized portfolio artifacts. They do not contain working credentials, Google resource IDs, or webhook IDs, and they import as inactive workflows.

## Prerequisites

- An n8n instance that supports the nodes in the exports
- A Google account with access to Google Drive and Google Sheets
- A Google Cloud project with the Google Drive API and Google Sheets API enabled
- Google OAuth credentials configured for the n8n instance
- A Gemini API key
- A Gemini model that accepts video input and can return JSON text
- Premiere Pro and Adobe Media Encoder only if using the professional linked-proxy preparation described by this project

The exact n8n version used for the original build was not recorded in the repository. Node labels and options can differ between releases.

## 1. Create Google resources

### Drive

Create an intake folder for analysis copies of completed proxy files. Do not point the workflow at a folder containing original camera masters unless that is an intentional, tested decision.

### Google Sheets

Create one spreadsheet with these tabs:

- `Clip Catalog`
- `Review Queue`
- `Error Log`

Add the exact headers from [data-contract.md](data-contract.md) to row 1 of each tab.

## 2. Configure credentials

1. In Google Cloud, enable the Google Drive API and Google Sheets API.
2. Configure the OAuth consent screen appropriate to the account.
3. Create an OAuth client whose redirect URI exactly matches the URI shown by the n8n Google credential screen.
4. In n8n, create and authorize Google Drive and Google Sheets credentials.
5. In Google AI Studio or the supported Google interface, create a Gemini API key.
6. In n8n, create a Gemini credential with that key.

Never commit the client secret, OAuth token, Gemini API key, or n8n credential export.

## 3. Import the AI footage logger

Import:

```text
workflows/ai-footage-logger.sanitized.json
```

Reconnect credentials for:

- Google Drive Trigger
- Download file
- Analyze video
- each Google Sheets append node

Replace the placeholders:

```text
REPLACE_WITH_YOUR_CREDENTIAL_ID
REPLACE_WITH_YOUR_GOOGLE_DRIVE_FOLDER_ID
REPLACE_WITH_YOUR_GOOGLE_SHEETS_DOCUMENT_ID
REPLACE_WITH_YOUR_CLIP_CATALOG_TAB_ID
REPLACE_WITH_YOUR_REVIEW_QUEUE_TAB_ID
REPLACE_WITH_YOUR_ERROR_LOG_TAB_ID
```

After choosing the spreadsheet and tabs from n8n's lists, refresh the Google Sheets columns and confirm every mapping against the headers.

The exported Gemini node references `models/gemini-3.1-flash-lite`. If unavailable, choose a video-capable Gemini model and rerun the full test matrix. Do not assume another model will follow the same prompt equally well.

## 4. Verify logger mappings

### Drive trigger

- Watch the intended intake folder.
- Trigger only for new files.
- Confirm the top-level Drive file `id` is used downstream, not a permission or parent-folder ID.

### Download

- Download by the current item's Drive file ID.
- Store the binary file in the field expected by the Gemini node (the original workflow uses `data`).

### Edit Fields

The `analysis` object is parsed with:

```javascript
{{ $json.content.parts[0].text.parseJson() }}
```

### Google Sheets routes

- `Append row in sheet` â†’ `Clip Catalog`
- `Append row in sheet1` â†’ `Review Queue`
- `Append row in sheet2` â†’ `Error Log`

Rename the nodes after import if desired; if code references a node by name, update the reference before renaming.

## 5. Test the logger before publishing

Use non-sensitive, short sample proxies with clearly different content:

1. Intentional dialogue scene
2. Interview/direct-address clip
3. Deliberate B-roll
4. Obvious false start or aborted take
5. Empty or ambiguous clip

Confirm:

- one new Drive file produces one execution;
- a completed clip reaches the catalog;
- a questionable clip reaches the review queue;
- a forced invalid output reaches the Error Log;
- a repeated Drive file ID is stopped before download/Gemini;
- filenames and timecodes belong to the current clip;
- no node uses pinned test data.

Publish the workflow only after those checks pass.

## 6. Import the search workflow

Import:

```text
workflows/footage-search-assistant.sanitized.json
```

Reconnect the Google Sheets and Gemini credentials. Select the `Clip Catalog` spreadsheet tab in `Get row(s) in sheet` and refresh its columns.

Confirm the form contains a required text field labeled exactly:

```text
What footage are you looking for?
```

If that label changes, update the field reference in `Build Search Catalog`.

## 7. Test search before publishing

Run at least these searches:

- an exact spoken phrase present in `Spoken Terms (Timecoded)`;
- a visual concept present only in summaries/keywords;
- a combined subject-and-action query;
- an intentionally impossible request.

Verify that search:

- returns no more than five unique proxy files;
- copies clip names and timecodes from the catalog;
- distinguishes spoken and visual matches;
- excludes `Processing Error` rows;
- marks review-required results with a warning;
- produces a no-match page for unsupported requests.

Publish the workflow and use its production form URL only after testing. Do not commit production or signed test URLs.

## 8. Optional QC workflow

`workflows/testing/ai-footage-logger-qc-test.sanitized.json` is an inactive copy intended for controlled validation experiments. Connect it only to test resources. Avoid pointing two active logger workflows at the same intake folder.

## Planned hard-failure route

The current setup does not append Gemini node timeouts or hard API failures to the Error Log sheet. A future version should:

1. enable the Gemini node's error output or continue-on-error behavior appropriate to the installed n8n version;
2. map the error into the Error Log contract;
3. append `Failed Stage = Gemini Video Analysis` and a safe error message;
4. preserve the proxy filename and n8n execution link when available;
5. test with a deliberate invalid credential, timeout, or unsupported file in a nonproduction workflow.

That route is future work, not part of the completed export.

