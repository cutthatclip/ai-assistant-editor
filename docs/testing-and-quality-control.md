# Testing and quality control

## Purpose

The project uses both model prompting and deterministic checks. Prompting improves editorial interpretation; code prevents malformed output from silently entering the clean catalog.

## Test cases completed

| Test | What it evaluated | Result |
|---|---|---|
| Intentional dialogue clip | classification, summary, search terms, usable range | Completed catalog row |
| Direct-address child clip | interview-like intent and spoken timestamps | Completed catalog row |
| Short shaky accidental-looking recording | production-intent reasoning | Initially misclassified; prompt revised; later classified `false_start_or_mistake` and routed to review |
| Duplicate Drive event | API cost and duplicate-row prevention | Previously processed Drive file ID discarded before Gemini |
| Confidence returned as `0.9` | resilient confidence handling | Normalized to `90` |
| Deliberately invalid confidence in QC copy | validation failure route | Error Log row created |
| Query for visible/spoken table content | semantic search and ranking | Relevant clips returned with reasons/timecodes |
| Query for content absent from catalog | hallucination resistance | No relevant footage response |

## Important learning from the false-start test

The first prompt over-weighted describable content: understandable speech and a visible person caused the model to treat an obvious mistimed recording as a real scene. The revised prompt added a production-intent check and explicit rules that speech alone does not make a scene or interview.

This does not prove the model will always detect bad takes. It demonstrates an iterative evaluation loop and why human review remains necessary.

## Deterministic validation

The logger validates:

- presence of the analysis object;
- required text fields;
- allowed category;
- expected arrays;
- expected true/false values;
- normalized 0â€“100 confidence;
- spoken term and timestamp structure;
- usable range timestamps and reasons.

Invalid structure produces:

```text
validation.valid = false
validation.processing_status = Processing Error
```

and a semicolon-delimited error message for the Error Log.

## Human-review rules

A valid response produces `Needs Review` if:

- the model requests review;
- confidence is below 80;
- the clip is mixed or unclear;
- the clip is a likely false start/mistake;
- a mistake flag is set;
- the clip may be empty or unusable;
- no usable range is identified.

These are product rules, not proof that the model's creative judgment is correct.

## Search quality rules

The search prompt:

- requires relevance of at least 60;
- limits output to five matches;
- deduplicates proxy files before prompting;
- distinguishes visible concepts from explicitly spoken phrases;
- requires catalog-supported filenames and timecodes;
- excludes `Processing Error`;
- warns about `Needs Review`;
- requires an empty array for no-match.

The result formatter escapes HTML before displaying model-provided strings.

## Evidence files

- `../assets/screenshots/structured-ai-output.png`
- `../assets/screenshots/human-review-queue.png`
- `../assets/screenshots/validation-error-log.png`

Raw videos are excluded for privacy, copyright, and repository size. Signed local form-result screenshots are also excluded because their browser URLs contained temporary private signatures.

## Test limits

- The sample is too small for statistical accuracy claims.
- No frame-accurate timecode benchmark was performed.
- No load, concurrency, rate-limit, or long-duration test is documented.
- Search was evaluated qualitatively, not with a labeled information-retrieval dataset.
- Gemini hard API failures were observed through n8n behavior but are not yet routed to the Error Log sheet.

## Recommended next evaluation

Build a permission-cleared test set with labeled categories and ground-truth timestamps. Record:

- category precision/recall;
- false-start detection rate;
- spoken-term accuracy;
- timestamp error tolerance;
- percentage routed to review;
- search precision at 1 and 5;
- false-positive rate for no-match queries;
- cost and processing time by clip duration.

