# Submission scans

Organize scans as **test version / test ID / page images**:

```text
inputs/submission_scans/
  version_A/
    TEST_001/
      001.jpg
      002.jpg
      003.jpg
    TEST_002/
      001.jpg
      002.jpg
  version_B/
    TEST_003/
      001.jpg
      002.jpg
```

This mirrors the original workflow: each exam version had a folder, each paper had its own subfolder, and the phone's JPG files stayed in filename/capture-date order, which gave the page order of that test. Sort by filename when filenames preserve capture order; otherwise use capture dates to establish order, then record it in a manifest. Filesystem modification dates can change during syncing and are not reliable capture dates.

For new scans, zero-padded page names (`001.jpg`, `002.jpg`, …) are easiest. Existing phone filenames are fine. The agent must record the exact ordered filenames and map each question to its source pages. Never assume question number equals photo number. Extra pages, mixed batches, rescans, and reordered pages happen; verify the printed exam version and flag conflicts instead of silently assigning pages to the wrong test.

Write an anonymous test ID on the cover and first scanned page before taking photos. Omit the name-bearing cover. Keep the ID-to-name list elsewhere. Check remaining pages for names or other identifiers too. IDs only need to be unique together with their version.

Photograph pages in reading order, capture margins and reverse sides that contain answers, and keep the original files. A phone synced with Dropbox or Google Drive works; a scanner or PDF works too. For PDFs, the agent should make page images in workspace/ and preserve the original PDF. Don't merge two students into one document.

The nested folder is an empty example. No student submissions are distributed. Scan contents are ignored by Git.
