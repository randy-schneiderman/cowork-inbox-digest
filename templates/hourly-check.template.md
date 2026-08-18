# Hourly check template

Copy this, fill in every `{{PLACEHOLDER}}` from your `INTERVIEW.md` answers and your filled-in `connector-capability-map.md`, then paste the result as the `prompt` when you create the scheduled task in `SETUP.md`.

---

Hourly inbox check for {{PERSON_NAME}}'s {{TOPIC_SLUG}} digest. {{PERSONA_DESCRIPTION}}

CONNECTOR: {{CONNECTOR}}. Use the tool calls from your connector capability map for every SCAN, READ, MARK_PROCESSED, and SEND/DRAFT step below — they're referenced here by capability name, not hardcoded.

APPROVAL MODE: {{APPROVAL_MODE}}

WORKING FOLDER: {{WORKING_FOLDER}} (if your connector has no STORAGE capability, skip any step that references it). Every hourly run that produces an artifact saves a copy here; the end-of-day run sweeps this folder clean once the day's digest has consolidated everything, so files here are always same-day and disposable.

MAX RETRIES: {{MAX_RETRIES}}

FIRST: Run in bash:
  date
  TZ='{{TIMEZONE}}' date '+%Y-%m-%d %H %Z'

This gives you the current local date, hour (0-23), and timezone label. Store the local date as LOCAL_DATE and the hour as LOCAL_HOUR. LOCAL_HOUR prints zero-padded (e.g. `08`, `09`); if you compare it using bash arithmetic (`((...))`) instead of a POSIX test, bash reads a leading zero as octal and `08`/`09` will error out. Strip the leading zero, or force base 10 (`10#$LOCAL_HOUR`), or just use `[ "$LOCAL_HOUR" -lt N ]` style comparisons, which handle zero-padded values correctly without any of this.

OPERATING WINDOW: If LOCAL_HOUR is less than {{OPERATING_WINDOW_START_HOUR}} or greater than {{OPERATING_WINDOW_END_HOUR}}, stop immediately. No output, no file, no notification.

ERROR HANDLING (applies to every step below that calls an external tool):
- Transient failure (rate limit, timeout, 5xx, a connector hiccup): retry up to {{MAX_RETRIES}} times with a short pause between attempts.
- Permanent failure (auth error, permission denied, malformed request): do not retry, treat as a hard failure immediately.
- A hard failure on SCAN (the step itself errors out, not "zero results"): stop the run and send a PushNotification (routine_summary tags) stating what failed and at which step. This is the one exception to "stay silent" — silence must only ever mean nothing new, never that the run broke.
- A hard failure reading one specific message: skip that message, note the count in the digest footer as "N messages excluded due to read errors," continue with the rest.
- A hard failure on DELIVER (artifact) or SEND/DRAFT: after exhausting retries, do NOT proceed to the mark-processed step. Leave every message exactly as found so the next run picks it up again. Send a failure PushNotification.

STEP 1 — SCAN

Use your connector's SCAN capability to find unread inbox messages from the last day. Previous hourly runs mark processed mail via MARK_PROCESSED, so a well-scoped scan should return only new arrivals since the last run. Paginate if needed.

If the result is empty: stop immediately with no output. (Not an error, see ERROR HANDLING above for the distinction.)

STEP 2 — CLASSIFY

For each thread assign a bucket:

BUCKET A — INCLUDE IN DIGEST: {{BUCKET_A_DEFINITION}}

BUCKET B — SKIPPED (read to confirm, note in footer, mark processed): {{BUCKET_B_DEFINITION}} Include in the footer as: "Skipped: [subject] — [one-line reason]." Mark processed after delivery. If a Bucket B email contains something that genuinely belongs in Bucket A, promote it.

BUCKET C — IGNORE ENTIRELY: {{BUCKET_C_LIST}} Do not mention, do not mark processed.

If there are no Bucket A or B messages after classification: stop immediately with no output.

STEP 3 — READ

For every Bucket A and Bucket B message, use your connector's READ capability to get the full plain-text body. Do not summarize from snippets or subject lines. Keep a list of message IDs read and their bucket. Apply the per-message error handling above for any message that fails to read.

Some messages are aggregator or "daily digest" emails by design (a platform's own roundup linking to several unrelated articles, each with only a one-line teaser, no full body to read). These aren't a read error, there's genuinely nothing fuller to fetch, so don't retry them and don't force them into Bucket A or B on the strength of the teaser alone. Exclude them from classification, and count them in the footer separately as "N aggregator/digest emails excluded (snippet-only, no full body available)."

STEP 4 — BUILD UPDATE DIGEST

Write a compact self-contained HTML file. For the filename, convert {{TOPIC_SLUG}} to a filename-safe slug first: lowercase, spaces and punctuation to hyphens (e.g. "AI Digest" becomes "ai-digest"). Name the file `{LOCAL_DATE}_{slug}-update-{LOCAL_HOUR}h.html`. Use {{TOPIC_SLUG}} in its original casing everywhere else (headers, subject lines).

Style: light and dark mode via prefers-color-scheme, max-width 820px, system font stack, all CSS inline in a style block, no external assets, no localStorage. {{VOICE_DESCRIPTION}} Banned words/phrasing: {{BANNED_PHRASES}}

Sections in order:
1. Header: "{{TOPIC_SLUG}} Update" plus the local date and time.
2. Stat chips row: new messages scanned this check, new relevant messages (A+B), new stories found.
3. NEW STORIES (Bucket A only): one card per story. Each card: headline, a short factual paragraph of what actually happened, a "Why it matters" callout written for {{PERSONA_DESCRIPTION}}, a line listing which source(s) carried it.
4. Footer: list each Bucket B skipped item, count of Bucket C messages ignored this check, count of any messages excluded due to read errors, count of aggregator/digest emails excluded per STEP 3, accuracy note.

ACCURACY IS NON-NEGOTIABLE. Never invent facts, statistics, quotes, links, or counts. Every claim must trace to an email you actually read. Label vendor claims and unconfirmed press reports as such. If a number is an estimate, say so.

If Bucket A has zero stories (only Bucket B items, or nothing left after excluding aggregator emails): do not produce or deliver a file. This is a normal outcome, not a failure, and it satisfies STEP 6's mark-processed gate exactly like a successful delivery would; proceed straight to STEP 6 and mark every Bucket B message processed. The only exception: if any message was excluded due to a genuine read error in this same check (not an aggregator exclusion), send the STEP 7 PushNotification anyway, noting the excluded count, even though no digest file was produced. Otherwise stop here with no notification.

STEP 5 — DELIVER

Call SendUserFile (status "proactive", display "render") with the HTML file. If mcp__remote-devices__create_artifact is available (a Cowork-internal tool that persists a rendered copy to the user's desktop artifact gallery; harmless to skip if it's not present), call it with the returned file_uuid to persist it. If your connector has a STORAGE capability and {{WORKING_FOLDER}} is not "N/A", also save a copy of the same HTML file into {{WORKING_FOLDER}} using STORAGE's upload/write action — this is what the end-of-day run's cleanup step removes later. Apply the DELIVER error handling above if any of this fails.

STEP 6 — MARK PROCESSED

If APPROVAL MODE is AUTO: mark every Bucket A and Bucket B message processed via MARK_PROCESSED, using your connector's capability. This applies whether STEP 5 ran normally (mark processed after it succeeds) or STEP 4 took the zero-Bucket-A branch and skipped STEP 5 entirely (mark processed immediately, per STEP 4's instructions, that branch is a normal outcome, not a pending delivery). Never touch Bucket C messages. The only time you skip this step is a genuine DELIVER failure per ERROR HANDLING above, where messages must stay unprocessed so the next run retries them.

If APPROVAL MODE is REVIEW: this step still applies exactly as above for the hourly check — hourly runs are read-and-classify only, nothing gets sent to anyone outside your own inbox, so there's no meaningful human checkpoint to insert here. REVIEW mode changes STEP 7 in the end-of-day template instead, where an actual email goes out.

STEP 7 — NOTIFY

Send a PushNotification. Wrap the content in routine_summary tags. Lead with the single most important new story in one sentence. Follow with the story count and a note to check the artifact for details.
