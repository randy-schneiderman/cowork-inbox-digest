# Setup prompt (automated path)

This is not documentation to read. It's a prompt to paste directly into a Claude Cowork conversation. It runs the interview live, tests your connector for real, generates both filled scheduled-task prompts from the templates in this repo, shows them to you, and only creates the two scheduled tasks after you say go.

If you'd rather do this by hand and read every step yourself before anything touches your inbox, use `SETUP.md` instead. Both paths end at the same two scheduled tasks; this one is faster, `SETUP.md` is more auditable. Nothing stops you from reading the generated prompts closely in Step 5 below even on this path, that review step is not optional in what you're about to paste.

Before pasting this in: make sure this repo (or at minimum `templates/hourly-check.template.md`, `templates/end-of-day-digest.template.md`, and `templates/connector-capability-map.md`) is available in the Cowork session's workspace. The prompt below reads those files directly rather than carrying its own copy, so the two stay in sync and you're never running an out-of-date version of one or the other.

If you're starting from nothing: either clone this GitHub repo into the workspace (ask Cowork to clone `https://github.com/randy-schneiderman/cowork-inbox-digest`, or paste that URL and ask it to fetch the repo), or download the repo as a zip from GitHub and upload the three files above directly into the conversation. Either way, confirm Cowork can actually `Read` those three files before moving on, don't just assume the upload worked.

---

Paste everything below this line into a Cowork conversation.

---

You are setting up the Cowork Inbox Digest toolkit for me, end to end, in this conversation. Follow these steps in order and do not skip the confirmation step before creating anything.

STEP 1 — LOCATE THE TEMPLATES

Read `templates/hourly-check.template.md`, `templates/end-of-day-digest.template.md`, and `templates/connector-capability-map.md` from the current workspace. If you can't find them, stop and tell me to upload or clone the repo first. Don't improvise a version from memory.

STEP 2 — RUN THE INTERVIEW

Ask me the following in groups, waiting for my answers before moving to the next group. Use a multiple-choice question where the answer is a small fixed set (connector, approval mode); ask in plain text where it's open-ended (bucket definitions, persona, schedule times, voice). If an answer to a bucket definition is vague ("AI stuff relevant to me"), push back and ask me to be specific before accepting it, the same way `INTERVIEW.md` tells a human reader to.

- Group 1, identity and focus: persona/role description, one-line topic slug.
- Group 2, classification: Bucket A (in scope), Bucket B (adjacent, log but skip), Bucket C (ignore entirely).
- Group 3, connector: Gmail, Microsoft 365, or other. If other, don't guess. Use ToolSearch and a couple of manual test calls in this conversation to find and verify the four required capabilities (scan, read, mark processed, send/draft) before continuing, per the "Other connectors" instructions in `connector-capability-map.md`. Report what you found and confirm it with me before treating it as settled.
- Group 4, schedule: timezone, operating window hours, hourly cron minute, end-of-day local send time.
- Group 5, delivery: destination email, notification preferences.
- Group 6, storage: working folder ID/path, or none if the connector has no storage capability.
- Group 7, voice: tone description, banned words/phrases, and the two end-of-day section titles (one for role-specific implications, one for a weekly action item, see `INTERVIEW.md` §10 for examples).
- Group 8, safety: approval mode (AUTO or REVIEW) and max retries. If I seem unsure, recommend REVIEW and a low retry count as the safer starting point, matching the recommendation in `INTERVIEW.md`.

STEP 3 — CONVERT SCHEDULE TO CRON

Using the timezone and times from Group 4, compute the actual UTC cron expressions `create_trigger` needs. Show me the conversion, including which UTC offset you used and why, so I can catch a daylight saving time mistake before it ships.

STEP 4 — FILL THE TEMPLATES

Using my answers and the connector capability map, produce the fully filled contents of both templates, with every `{{PLACEHOLDER}}` replaced with a real value. Do not create anything yet.

STEP 5 — SHOW ME BEFORE CREATING ANYTHING

Show me both filled prompts in full, plus the two cron expressions and the trigger names you're about to use. Ask explicitly: "Create both scheduled tasks with this configuration?" Wait for an explicit yes. If I ask for changes, go back to Step 2 or Step 4 as needed and show me the result again before asking again. Do not call `create_trigger` until I've said yes to the exact configuration you just showed me, not an earlier draft of it.

STEP 6 — CREATE THE SCHEDULED TASKS

Once I confirm, call `create_trigger` twice: once for the hourly check, once for the end-of-day digest, using the names, cron expressions, prompts, and notification preferences from Step 5.

STEP 7 — HAND BACK CONTROL

Report both trigger IDs and their next run times. Tell me to test with `fire_trigger` before trusting the schedule, per `SETUP.md` step 8, including the "deliberately break something" check that confirms failures actually notify me instead of failing silently. Don't run that test yourself unless I ask you to; creating the tasks is as far as this prompt goes.
