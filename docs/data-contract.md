# Data contract

## Footage-analysis JSON

Gemini is instructed to return one JSON object with this structure:

```json
{
  "overall_category": "scene_dialogue",
  "content_summary": "A child sets a table while interacting with an adult.",
  "visual_description": "A child arranges dishes and cutlery in a home setting.",
  "spoken_content_summary": "They discuss setting the table.",
  "searchable_keywords": ["child", "table", "dinner", "cutlery"],
  "visual_keywords": ["child", "table", "dishes", "home"],
  "spoken_terms": [
    {
      "term": "set the table",
      "timestamp": "00:00:07"
    }
  ],
  "people_or_subjects": ["child", "adult"],
  "locations_or_settings": ["home", "dining area"],
  "mistake_detected": false,
  "mistake_summary": "",
  "empty_or_unusable": false,
  "human_review_required": false,
  "review_reason": "",
  "confidence_score": 92,
  "usable_ranges": [
    {
      "start_time": "00:00:00",
      "end_time": "00:01:21",
      "reason": "Coherent interaction with useful dialogue."
    }
  ],
  "segments": [
    {
      "start_time": "00:00:00",
      "end_time": "00:01:21",
      "segment_type": "scene_dialogue",
      "summary": "Child and adult set the table.",
      "spoken_terms": ["set the table"],
      "visual_keywords": ["table", "dishes"],
      "usable": true,
      "issue": ""
    }
  ]
}
```

The example illustrates the schema; it is not a guaranteed output.

## Allowed overall categories

- `interview`
- `scene_dialogue`
- `b_roll`
- `performance_take`
- `false_start_or_mistake`
- `empty_or_unusable`
- `mixed`
- `needs_human_review`

## Validation requirements

- `overall_category`, `content_summary`, and `visual_description` must be nonempty text.
- `spoken_content_summary`, `mistake_summary`, and `review_reason` must be text, but may be empty.
- keyword, subject, location, spoken-term, usable-range, and segment fields must be arrays.
- mistake, unusable, and human-review fields must be true/false values.
- `confidence_score` must normalize to a whole number from 0 to 100.
- spoken timestamps and range boundaries must use `HH:MM:SS`.
- each usable range must have a nonzero end time and a reason.

## Clip Catalog columns

Create these headers exactly, in row 1:

```text
Original Clip Name
Proxy Clip Name
Clip Category
Content Summary
Visual Description
Spoken Terms (Timecoded)
Searchable Keywords
Mis-Takes
Potentially Usable Sections
Confidence Score
Processing Status
Error
```

## Review Queue columns

```text
Original Clip Name
Proxy Clip Name
Clip Category
Content Summary
Review Reason
Confidence Score
Potentially Usable Sections
Review Status
```

## Error Log columns

```text
Timestamp
Workflow
Proxy Clip Name
Failed Stage
Error Message
Execution URL
Status
```

The completed workflow uses this log for structured validation failures. The planned Gemini hard-failure route will reuse these columns.

## Search request

The editor submits one text field:

```text
What footage are you looking for?
```

## Search-result JSON

```json
{
  "search_query": "child setting a table",
  "no_match": false,
  "result_count": 1,
  "matches": [
    {
      "rank": 1,
      "original_clip_name": "EXAMPLE_001",
      "proxy_clip_name": "EXAMPLE_001_Proxy.mov",
      "category": "scene_dialogue",
      "relevance_score": 95,
      "matching_timecode": "00:00:00-00:01:21",
      "matched_by": ["content_summary", "visual_description"],
      "match_reason": "The child is visibly setting a dining table.",
      "processing_status": "Complete",
      "warning": ""
    }
  ],
  "message": ""
}
```

No-match responses use an empty `matches` array, `no_match: true`, `result_count: 0`, and the message `No relevant footage was found.`

