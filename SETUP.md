# Setup

This is the manual, step-by-step path. If you'd rather have a Cowork conversation run the interview and set everything up for you, with a confirmation step before anything is created, use `SETUP-PROMPT.md` instead. Both end at the same two scheduled tasks.

Assumes you've already answered `INTERVIEW.md`. If not, do that first.

## 1. Connect your mail connector

In Claude Cowork, connect whichever mail connector you picked in `INTERVIEW.md` (Gmail, Microsoft 365, or another) under your connector settings. This repo assumes it's already authorized before the scheduled tasks run; the scheduled task itself does not handle authorization.

## 2. Fill in the connector capability map

Open `templates/connector-capability-map.md`. Gmail and Microsoft 365 already have reference rows. If you're on something else, follow the "Other connectors" instructions in that file first, including the manual test of each capability, before you write anything into a schedule.

## 3. Create a scratch folder (optional)

Only if your connector has a STORAGE capability in the map above.

- Gmail path: create a folder in Google Drive, open it, and copy the folder ID out of the URL (the string after `/folders/`).
- M365 path: create a folder in SharePoint or OneDrive and note its path or ID, depending on what the SharePoint tools in your environment expect.
- No STORAGE capability: skip this, leave `{{WORKING_FOLDER}}` as "N/A" in the templates, and the end-of-day template's cleanup step will skip itself.

This is your `{{WORKING_FOLDER}}` value.

## 4. Decide approval mode and retry count

From `INTERVIEW.md` sections 12 and 13: pick `{{APPROVAL_MODE}}` (AUTO or REVIEW) and `{{MAX_RETRIES}}` (2 or 3 is reasonable). If you're not sure yet, start with REVIEW and a low retry count; you can loosen both later with `update_trigger` once you trust the classification.

## 5. Fill in the templates

Open `templates/hourly-check.template.md` and `templates/end-of-day-digest.template.md`. Replace every `{{PLACEHOLDER}}` with your `INTERVIEW.md` answers and your filled-in capability map. There are no connector branches to delete anymore — the templates reference your capability map by name, so the same two files work for Gmail, M365, or anything else you mapped.

Check `examples/platform-engineering-example.md` if you want to see one fully filled in.

## 6. Create the hourly scheduled task

In a Claude Cowork conversation, ask it to create a scheduled task (the underlying tool is `create_trigger`, but you can just describe what you want):

- Name: something like "{{TOPIC_SLUG}} — Hourly Update"
- Cron: run on a minute other than `:00`, every hour. Example: `41 * * * *`. Cron in `create_trigger` is UTC — if you want it to actually run every hour in your local timezone, an ordinary hourly cron already does that (it's every hour regardless of timezone); what you need to convert is the *operating window* hours inside the prompt (`{{OPERATING_WINDOW_START_HOUR}}` / `{{OPERATING_WINDOW_END_HOUR}}`), which are evaluated against local time inside the prompt itself, not the cron schedule.
- Prompt: the filled-in contents of `templates/hourly-check.template.md`.
- Notifications: your choice; push-only is a reasonable default so you're not getting emails every hour.

## 7. Create the end-of-day scheduled task

- Name: something like "{{TOPIC_SLUG}} — End of Day Email"
- Cron: pick your local send time and convert to UTC. Example: 10 PM Central during daylight time (UTC-5) is `0 3 * * *` (3 AM UTC, the next calendar day). Central Standard Time (UTC-6) would be `0 4 * * *`. Check which offset is in effect on the day you're setting this up.
- Prompt: the filled-in contents of `templates/end-of-day-digest.template.md`.
- Notifications: push and/or email, your choice — this task also sends its own email as part of the prompt, so a notification here is just the phone alert, not a duplicate digest.

## 8. Test before trusting the schedule

Don't wait for the cron to fire. Use `fire_trigger` (or ask Claude Cowork to "run that scheduled task now") on the hourly one first, since it's the cheaper one to iterate on. Check:

- Did it correctly stay silent if there was nothing to report, or did it produce a real output?
- Are Bucket A stories actually the things you'd want to see? If not, your `{{BUCKET_A_DEFINITION}}` from the interview is too broad or too narrow. Go tighten it.
- Are Bucket C items actually being ignored, and not silently costing you a read call?
- Deliberately break something (temporarily point SCAN at a folder ID that doesn't exist, or similar) to confirm you actually get a failure notification instead of silence. Do this once before you trust the "silence means nothing new" behavior with real stakes.

Then fire the end-of-day one and check: the email arrives (or, if you set REVIEW mode, a draft shows up and no email actually sends), the HTML renders, and on M365 the sanitized email body still looks reasonable given the stripped `<style>` block.

If you set REVIEW mode, confirm the messages stayed unprocessed after that test run, and that firing the hourly check again re-surfaces the same messages rather than silently dropping them.

## 9. Tune

Your first classification pass is very unlikely to be right, and that's expected, not a bug. Go back to `INTERVIEW.md` sections 2 through 4, rewrite the bucket definitions to be more specific about what belongs where, and use `update_trigger` to replace the prompt on the existing scheduled task (this keeps its run history, unlike deleting and recreating it). Once you trust it, flip `{{APPROVAL_MODE}}` from REVIEW to AUTO the same way.

## Troubleshooting

- **Nothing ever fires**: check the scheduled task is enabled (`list_triggers`), and that the operating window hours in the prompt aren't accidentally excluding every hour cron actually fires at.
- **Digest includes obviously wrong content**: your Bucket A definition is too broad, or the model is promoting Bucket B items too eagerly. Tighten the promotion rule in the template.
- **M365 mail scan finds nothing**: the connector's mail resource browsing tools and URIs may not match what's documented in `connector-capability-map.md`. Run a one-off check in a normal (non-scheduled) Cowork conversation with `ListMcpResourcesTool` against the Microsoft_365 server to see what's actually exposed before troubleshooting further.
- **A different connector's SCAN/READ/MARK_PROCESSED silently does nothing**: you likely skipped the manual per-capability test in step 3 of "Other connectors" in `connector-capability-map.md`. Go do that now, in a plain conversation, before debugging the scheduled version.
- **Emailed digest looks worse than the artifact (M365 only)**: expected. `outlook_send_mail`'s HTML sanitizer strips `<style>` blocks and a few other tags. Either inline styles per-element in the template, or treat the in-app artifact as the real deliverable and the email as a lighter-weight notification.
- **Digest keeps re-including things you already saw**: either the "mark processed" step isn't running or is failing silently, your SCAN query in Step 1 isn't actually filtering on the processed marker, or you're in REVIEW mode and haven't sent the draft yet (which is working as intended, not a bug).
- **You never got a failure notification when something obviously broke**: check the ERROR HANDLING block at the top of the template actually made it into what you pasted into `create_trigger` — it's easy to trim it out by accident while filling in placeholders. Re-run the deliberate-break test from step 8.
- **REVIEW mode has no draft tool for your connector**: check `connector-capability-map.md` row 4. If your connector has no DRAFT capability, REVIEW mode isn't available for the send step yet; AUTO is your only option until one exists, or you build the human checkpoint some other way (e.g. reviewing the artifact and manually triggering send).
