# Setup interview

Answer these before touching the templates. Write your answers down somewhere (a scratch doc is fine); you'll paste them into `templates/hourly-check.template.md` and `templates/end-of-day-digest.template.md` in `SETUP.md`.

Don't skip the vague-sounding ones. "AI stuff relevant to me" is not a classification rule. "Model releases, agent tooling, inference cost, and infrastructure security, written for a platform engineering audience, not for solo founders" is.

## 1. Who is this for, and what's the focus

- Your name, or whatever you want the digest to call you in its greeting/headers. This one isn't a design decision, just write your own name. `{{PERSON_NAME}}` in the templates.
- What's your role or the lens you want the digest filtered through? (Example: "DevOps/platform engineering leader growing into AI engineering.") `{{PERSONA_DESCRIPTION}}` in the templates.
- What's the one-line focus area? This becomes the digest's name (shown in headers and the email subject) and is also converted into a filename-safe slug automatically by the templates, so pick something readable in both a title and lowercase-with-hyphens. (Example: "AI Digest".) `{{TOPIC_SLUG}}` in the templates.

## 2. Bucket A: what counts as in scope

List the specific subtopics that should show up in the digest. Be concrete, not aspirational. If you can't tell someone else how to sort an email into this bucket, the definition isn't done yet.

Example (platform engineering / AI focus): "AI/ML models and releases, agents, AI infrastructure, AI tooling, AI engineering practice, AI industry news, AI policy affecting engineering, inference cost, observability, eval tooling, MCP, agent orchestration, GPU infrastructure, model serving, CI/CD with AI, supply-chain security for AI systems."

If audience level matters to you as much as topic (for example, you want advanced technical content but not the same topic aimed at beginners), say so explicitly here. The bucket definitions below are topic-based by default; a topic-based definition will happily include something genuinely on-topic but written for the wrong skill level, since nothing in the classification step reasons about audience unless you tell it to.

`{{BUCKET_A_DEFINITION}}` in the templates.

## 3. Bucket B: adjacent but wrong audience

What's related to your focus area but aimed at the wrong audience, and should be logged (with a one-line reason) but not included as a full story? This keeps the digest from silently dropping things you might want to double check later.

Example: "AI content whose primary audience is solo founders, entrepreneurs, or consumers rather than platform engineers: drip courses on AI for business growth, AI productivity tips for non-technical audiences, marketing automation with AI."

Also decide: should a Bucket B item ever get promoted to Bucket A? (Recommended: yes, if it contains a concrete technique or pattern genuinely useful to your Bucket A audience.)

`{{BUCKET_B_DEFINITION}}` in the templates.

## 4. Bucket C: ignore entirely

What should never be mentioned, never marked read, never touched? Get this list explicit so the automation doesn't waste a read call on it.

Common defaults, adjust as needed: job alerts (even if the job title mentions your focus area), recruiter and vendor sales outreach that only name-drops your focus area, financial/billing notices, personal mail, school, medical, travel, parking, neighborhood apps, and anything off-topic entirely.

`{{BUCKET_C_LIST}}` in the templates.

## 5. Mail connector and account

Don't assume everyone's on Gmail or Microsoft 365. Pick one:

- **Gmail**
- **Microsoft 365** (Outlook)
- **Other** (a different connector, or one reached through Zapier or a similar bridge)

Which mailbox? (Work or personal; the templates assume the account is already authorized in Cowork before the scheduled task runs.)

Then go fill out `templates/connector-capability-map.md`. Gmail and Microsoft 365 have reference rows already filled in there. If you picked Other, that file walks you through finding your connector's equivalent tools before you touch the templates — the templates call the capability map, not a hardcoded connector, so this is the one file that actually changes per connector.

`{{CONNECTOR}}` in the templates refers to whichever row of the capability map you filled in.

## 6. Operating hours and timezone

- What timezone should the automation reason in? (Example: `America/Chicago`.)
- What hours should the hourly check actually run? Outside those hours it should do nothing, even if the schedule fires. (Example: 6 AM to 9 PM Central, so it stays out of the middle of the night.)

`{{TIMEZONE}}`, `{{OPERATING_WINDOW_START_HOUR}}`, `{{OPERATING_WINDOW_END_HOUR}}` in the templates.

## 7. Schedule

- Hourly check: pick a cron minute other than `0` so you're not colliding with everyone else's on-the-hour jobs. (Example: `41 * * * *`.)
- End of day digest: what local time should it fire? Remember cron in `create_trigger` runs in UTC, so convert. (Example: 10 PM Central in August, UTC-5, is `0 3 * * *` the next day UTC.)

## 8. Delivery

- What email address should the end-of-day digest be sent to?
- Do you want a push notification on the hourly check too, or only for the end-of-day digest?

`{{DELIVERY_EMAIL}}` in the templates.

## 9. Storage

- Gmail path: a Google Drive folder ID for scratch files during a run.
- M365 path: a SharePoint or OneDrive folder path/ID for scratch files.
- Should the end-of-day run clean up files the hourly runs left behind? (Recommended: yes.)

`{{WORKING_FOLDER}}` in the templates.

## 10. Voice and style

- What tone should the digest be written in? (Example: "thoughtful, pragmatic, analytically rigorous, challenges assumptions, evaluates tradeoffs, no fluff.")
- Any banned words or phrases? (Example: no em dashes, no "leverage," "synergy," "deep dive," "at the end of the day," "game-changer," "unlock," "landscape.")
- Bullet points or prose by default?
- The end-of-day digest has two titled sections beyond the story cards: one for implications specific to your role/focus, and one for a concrete weekly action item. What should those two section titles say? (Examples: "PLATFORM / DEVOPS ANGLE" and "SKILL-BUILDING"; or "ACTION REQUIRED" and "RUNBOOK UPDATE.")

`{{VOICE_DESCRIPTION}}`, `{{BANNED_PHRASES}}` in the templates. The two section titles are `{{ANGLE_SECTION_TITLE}}` and `{{SKILL_SECTION_TITLE}}` in `templates/end-of-day-digest.template.md`.

## 11. File naming

Generated filenames are built automatically by the templates: `{DATE}_{slug}-update-{hour}h.html` for the hourly artifact and `{DATE}_{slug}-digest.html` for the end-of-day one, where `{slug}` is your `{{TOPIC_SLUG}}` from section 1, lowercased with hyphens instead of spaces. Nothing to fill in here beyond picking a good `{{TOPIC_SLUG}}` in section 1.

## 12. Approval mode

Scheduled tasks run unattended, so there's no one there to click approve on each tool call mid-run. Cowork handles this by putting scheduled runs in auto-approval mode by default; that's what actually makes an hourly job practical. But "fully auto" and "human checks before anything leaves your control" are both legitimate defaults depending on how much you trust the classification yet.

Pick one:

- **AUTO** (recommended once you trust your Bucket A/B/C definitions): the run sends the email and marks messages processed with no human step in between.
- **REVIEW**: the run still scans, classifies, and builds the digest unattended, but instead of sending the email it creates a draft, and it does not mark messages processed. You review the draft and send it yourself; because messages stay unprocessed, the next scheduled run picks the same ones up again until you do. Good for the first week or two while you're still tuning bucket definitions, or for anything going to a work inbox where you want a last look before it's out of your hands.

`{{APPROVAL_MODE}}` in the templates. You can start on REVIEW and flip to AUTO later with `update_trigger` once you trust it, without losing the run history.

## 13. Error handling and retries

Decide what "broken" should look like versus "quiet."

- How many retries for a transient failure (rate limit, timeout, a connector hiccup) before giving up on that step? Two or three is reasonable; more than that just delays a failure notification you'd want sooner.
- If the whole scan step fails outright (not "zero new messages," an actual error), should you get notified? Recommended: yes, always. Silence should only ever mean "nothing new," never "it broke and nobody told me." Otherwise a broken connector looks identical to a quiet week.
- If one message out of many fails to read, should the run abort, or skip that message and keep going? Recommended: skip it, note the count in the digest footer, keep going. One bad message shouldn't kill the whole digest.
- If delivery (the artifact, or the email) fails after retries, messages should not be marked processed — otherwise you lose them permanently on a run that never actually delivered anything. Recommended: always leave them unprocessed on a failed delivery, no exceptions.

`{{MAX_RETRIES}}` in the templates. Everything else in this section is already baked into the template logic as the recommended defaults above; call it out in your own copy if you want different behavior.

Once you've got answers to all thirteen, move to `SETUP.md`.
