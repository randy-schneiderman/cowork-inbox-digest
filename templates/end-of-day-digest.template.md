# End of day digest template

Copy this, fill in every `{{PLACEHOLDER}}` from your `INTERVIEW.md` answers and your filled-in `connector-capability-map.md`, then paste the result as the `prompt` when you create the scheduled task in `SETUP.md`.

---

Build {{PERSON_NAME}}'s end-of-day {{TOPIC_SLUG}} digest, deliver it as an HTML artifact, and send it as an email. {{PERSONA_DESCRIPTION}}

CONNECTOR: {{CONNECTOR}}. Use the tool calls from your connector capability map for every SCAN, READ, MARK_PROCESSED, SEND/DRAFT, and STORAGE step below.

APPROVAL MODE: {{APPROVAL_MODE}}

WORKING FOLDER: {{WORKING_FOLDER}} (clean up at the end; if your connector has no STORAGE capability, skip Step 9 entirely)

MAX RETRIES: {{MAX_RETRIES}}

FIRST: Run in bash:
  date
  TZ='{{TIMEZONE}}' date '+%Y-%m-%d'

Use the local date (LOCAL_DATE) everywhere: filenames, headers, and the email subject. (This template only needs the date, not the hour, so the zero-padded-hour bash arithmetic issue noted in the hourly template doesn't apply here.)

ERROR HANDLING (applies to every step below that calls an external tool):
- Transient failure (rate limit, timeout, 5xx, a connector hiccup): retry up to {{MAX_RETRIES}} times with a short pause between attempts.
- Permanent failure (auth error, permission denied, malformed request): do not retry, treat as a hard failure immediately.
- A hard failure on SCAN itself: stop the run and send a PushNotification stating what failed. Don't produce a "quiet day" digest when the real story is a broken connector.
- A hard failure reading one specific message: skip it, note the count in the footer as "N messages excluded due to read errors," continue with the rest.
- A hard failure on DELIVER (artifact), SEND, or DRAFT: after exhausting retries, do NOT proceed to mark-processed or cleanup. Leave every message exactly as found so the next hourly run picks it up again. Send a failure PushNotification instead of Step 10's normal notification.

STEP 1 — SCAN

Use your connector's SCAN capability to list all inbox messages from the last 24 hours, regardless of processed/unprocessed state (hourly runs mark relevant mail processed throughout the day; this step needs everything, not just what's still unprocessed). Paginate if there are more results.

STEP 2 — CLASSIFY

For each message decide which bucket it belongs to:

BUCKET A — INCLUDE IN DIGEST: {{BUCKET_A_DEFINITION}}

BUCKET B — SKIPPED (read to confirm, note in footer, mark processed): {{BUCKET_B_DEFINITION}} Include in the footer as: "Skipped: [subject] — [one-line reason]." Mark processed after delivery. Promote to Bucket A if it genuinely belongs there.

BUCKET C — EXCLUDE ENTIRELY (no mention, no mark-processed): {{BUCKET_C_LIST}}

STEP 3 — READ

For every Bucket A and Bucket B message, use your connector's READ capability to get the full plain-text body. Do not write the digest from subject lines or snippets. Keep a note of each message ID you read and its bucket. Apply the per-message error handling above for anything that fails to read.

Aggregator or "daily digest" emails (a platform's own roundup linking to several unrelated articles, each with only a one-line teaser and no full body) aren't a read error and shouldn't be retried. Exclude them from classification and count them in the footer as "N aggregator/digest emails excluded (snippet-only, no full body available)."

STEP 4 — DEDUPLICATE

Group Bucket A coverage into distinct stories. Count how many separate sources carried each story. Rank by a combination of source count and relevance to {{PERSONA_DESCRIPTION}}. If most coverage came from a single source, say so plainly — repetition within one outlet is not consensus.

STEP 5 — BUILD THE ARTIFACT

Write a single self-contained HTML file. For the filename, convert {{TOPIC_SLUG}} to a filename-safe slug first: lowercase, spaces and punctuation to hyphens (e.g. "AI Digest" becomes "ai-digest"). Name the file `{LOCAL_DATE}_{slug}-digest.html`. Use {{TOPIC_SLUG}} in its original casing everywhere else (headers, subject line).

Style: light and dark mode via prefers-color-scheme, max-width 820px, system font stack, all CSS inline, no external assets, no localStorage. {{VOICE_DESCRIPTION}} Banned words/phrasing: {{BANNED_PHRASES}}

Sections in order:
1. Header: "{{TOPIC_SLUG}} Digest", LOCAL_DATE formatted as "Month DD, YYYY", "Past 24 hours".
2. Stat chips: messages scanned, relevant count (A+B), distinct stories after dedup, how many sources were read in full.
3. TOP STORIES, DEDUPLICATED. One card per story with a source-count badge: headline, a short factual paragraph of what happened, a "Why it matters" callout for {{PERSONA_DESCRIPTION}}, which sources carried it.
4. {{ANGLE_SECTION_TITLE}}. A bulleted list pulling out the implications specifically relevant to {{PERSONA_DESCRIPTION}}.
5. MUST-READ. 3 to 5 links worth full attention, each with one line on why. Use the real URLs from the source messages. Never substitute a homepage placeholder link.
6. {{SKILL_SECTION_TITLE}}. One concrete thing to do this week, drawn from the day's mail, tied to {{PERSONA_DESCRIPTION}}'s goals. Explain why it's the right one.
7. Footer: count of Bucket C excluded messages, list of Bucket B skipped items with reasons, count of read errors if any, count of aggregator/digest emails excluded per STEP 3, which sources were read in full, any source-concentration caveat.

ACCURACY IS NON-NEGOTIABLE. Never invent facts, statistics, quotes, links, or counts. Every claim must trace to a message you actually read. Label unverified vendor or press claims as such. If a number is an estimate, say estimate. If the day is quiet, produce a short honest digest that says so — do not pad it.

STEP 6 — DELIVER HTML ARTIFACT

Call SendUserFile (status "proactive", display "render") with the HTML file. If mcp__remote-devices__create_artifact is available (a Cowork-internal tool that persists a rendered copy to the user's desktop artifact gallery; harmless to skip if it's not present), call it with the returned file_uuid. Apply the DELIVER error handling above if this fails.

STEP 7 — SEND OR DRAFT EMAIL

Read the HTML file content from disk first.

If APPROVAL MODE is AUTO: use your connector's SEND capability. To: {{DELIVERY_EMAIL}}, subject "{{TOPIC_SLUG}} Digest — {LOCAL_DATE}", body the full HTML content (mark it as HTML body where the connector distinguishes plain text from HTML).

If APPROVAL MODE is REVIEW: use your connector's DRAFT capability instead of SEND, with the same To/subject/body. Do not send it. Note in the notification (Step 10) that a draft is waiting for review and has not gone out yet. If your connector has no DRAFT capability (check your connector-capability-map.md), REVIEW mode isn't available to you yet — either add one, or accept AUTO is your only option for this connector.

STEP 8 — MARK PROCESSED

If APPROVAL MODE is AUTO and Step 7 succeeded: use your connector's MARK_PROCESSED capability on every Bucket A and Bucket B message. Never touch Bucket C. Skip this step entirely if Step 6 or 7 failed — see ERROR HANDLING above.

If APPROVAL MODE is REVIEW: do not mark anything processed yet, regardless of whether the draft was created successfully. Messages stay marked unprocessed until you actually send the draft yourself; the next hourly run will otherwise just re-surface them, which is the intended behavior — it means nothing gets silently dropped while you're still deciding whether to trust the digest.

STEP 9 — CLEANUP WORKING FOLDER

If your connector has a STORAGE capability: use it to find and remove files left in {{WORKING_FOLDER}} by the hourly runs (each hourly run that produced an artifact saved a copy there per its STEP 5). If it doesn't, or if WORKING_FOLDER is "N/A", skip this step, there's nothing to clean up.

STEP 10 — NOTIFY

Send a PushNotification. Wrap the content in routine_summary tags. Lead with the top story of the day in one sentence. Follow with: total relevant messages processed today, whether the email was sent to {{DELIVERY_EMAIL}} or left as a draft awaiting review (per APPROVAL MODE), and a quiet/busy signal (e.g. "light day, 2 stories" or "busy day, 7 stories").

Finish with a plain-text summary: messages scanned, relevant count, top story, messages marked processed (or "none, awaiting review" if REVIEW mode), whether delivery succeeded.
