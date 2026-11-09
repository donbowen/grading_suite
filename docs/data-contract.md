# Shared data contract

Use UTF-8 JSON and stable IDs. The agent may extend these records for a particular exam, but must update this contract in its private working copy and migrate all consumers together. Arrays are ordered unless otherwise stated. All paths are project-relative. All mutable files have a monotonically increasing integer `revision`; schema changes increment `schema_version`.

## Files and ownership

| File in workspace/ | Required content |
| --- | --- |
| state.json | schema_version, revision, current_stage, source_fingerprints (path → SHA-256), completed_stages, approvals (stage, timestamp, teacher decision), blockers, next_actions |
| manifest.json | schema_version, revision, submissions: version, test_id, ordered pages (page_id, source_path, sha256), question_pages (question_id → page_id array), issues |
| questions.json | schema_version, revision, questions: version, question_id, prompt, possible, bonus, key, source_paths, optional equivalence_group |
| transcripts.raw.json | schema_version, revision, answers; immutable after a run, preserve old runs in history/ |
| transcription_reviews.json | schema_version, revision, append-only events: answer_id, base_hash, draft_text, accepted_text, status, note, timestamp |
| transcripts.reviewed.json | schema_version, revision, answers materialized from raw plus accepted corrections; never drafts |
| rubric.json | schema_version, revision, status (draft/approved), questions: version, question_id, possible, criteria, accepted_alternatives, partial_credit_rules, common_errors, dependencies, bonus, unresolved_questions; criteria each have criterion_id, description, points |
| calibration.json | schema_version, revision, rounds: sample_ids, independent_agent_scores, teacher_scores, discrepancies, decisions, resulting_rubric_revision, teacher_approved |
| grades.json | schema_version, revision, rubric_revision, transcript_revision, items, totals; snapshot previous runs in history/ |
| grade_reviews.json | schema_version, revision, append-only events: item_id, before, after, rationale, status, timestamp, rubric_revision, transcript_hash; separate individual overrides from shared rule events |
| issues.json | schema_version, revision, issues: issue_id, type, severity, affected_ids, evidence, suggested_action, status, resolution |

## Answer record

```json
{
  "answer_id": "version_A::TEST_001::Q1",
  "version": "version_A",
  "test_id": "TEST_001",
  "question_id": "Q1",
  "text": "The student’s retained answer, exactly as read.",
  "source_pages": ["version_A::TEST_001::p001"],
  "visual_evidence": [],
  "reading_status": "clear",
  "review_status": "pending",
  "notes": [],
  "source_hash": "sha256-of-source-and-transcription"
}
```

`reading_status`: clear, uncertain, blank, missing_source, or visual_review_required. `review_status`: pending, draft, deferred, or accepted. A genuinely blank answer on a present page is distinct from a missing/unreadable page. Use `[UNCLEAR]` for uncertain retained writing; preserve drawings/tables/code with source links and visual descriptions, never invented geometry. `source_hash` is computed, not this example string. Encode ID components or reject separators within them to prevent collisions.

## Grade item

```json
{
  "item_id": "version_A::TEST_001::Q1",
  "answer_id": "version_A::TEST_001::Q1",
  "version": "version_A",
  "test_id": "TEST_001",
  "question_id": "Q1",
  "earned": 3,
  "possible": 5,
  "bonus": false,
  "criterion_scores": [{"criterion_id": "Q1.c1", "earned": 3, "possible": 5}],
  "rationale": "Evidence and the specific rubric rule supporting this score.",
  "evidence": ["Student’s exact relevant text or source page reference"],
  "flags": [],
  "rubric_revision": 1,
  "transcript_hash": "computed-hash",
  "status": "proposed"
}
```

Grade status: proposed, needs_review, reviewed, or stale. A blocked score is `earned: null`, excluded from totals and exports until resolved. Decimal points are allowed if the teacher's rubric allows them. Award no more than possible points per item; track bonus separately. Calculate sums in code; do not trust totals generated in prose.

## Revisions and approval

Keep source scans and raw transcriptions unchanged. Save corrections and individual grade decisions as events, materialize current views, and keep prior snapshots. Accept saves only against the revision the teacher viewed; reject stale writes with a reload message. Drafts must survive reload but never affect accepted transcripts or final grades.

A transcript change marks corresponding grades stale. A shared rubric change increments the rubric revision, identifies every affected answer (including reviewed answers), and invalidates their review acceptance. Deterministic rules may update all affected scores together after an impact preview. Rules requiring judgment queue an agent regrade; show pending/stale status until it finishes. Preserve individual overrides with an explicit conflict flag if a new rule affects them. Show before/after totals and a history entry. Never claim a free-text rule has instantly regraded everyone just because its wording was saved.

Use SHA-256 fingerprints to detect input changes on resume. New papers, keys, and versions require appropriate preparation, review, and calibration. Final export requires complete coverage, current grades, resolved blocking issues, teacher approval, and agreement between exported totals and reviewed item scores.
