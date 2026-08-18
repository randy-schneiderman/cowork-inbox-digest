# Example: competitive intelligence tracking

A product or strategy lead tracking what a specific, named set of competitors is actually doing: launches, pricing, hires, funding, positioning. The key difference from the other examples is that Bucket A here is defined by a specific list of company names, not a topic area, and the digest is only as good as how tightly that name list is scoped.

- `{{PERSON_NAME}}`: your name
- `{{TOPIC_SLUG}}`: Competitive Intel Digest
- `{{PERSONA_DESCRIPTION}}`: A product/strategy lead tracking a specific, named set of competitors.
- `{{BUCKET_A_DEFINITION}}`: Product launches and feature announcements from [list your named competitors here]; pricing or packaging changes from those competitors; executive hires or departures at those competitors; funding rounds or M&A involving those competitors; analyst or press coverage that directly compares your company to one of them; patent filings or public roadmap signals from those competitors. Replace the bracketed list with the actual companies you're tracking, this bucket doesn't work as a vague "competitor news" catch-all.
- `{{BUCKET_B_DEFINITION}}`: General industry trend pieces that mention a tracked competitor only in passing; competitor content marketing (blog posts, webinars) that isn't announcing anything concrete.
- `{{BUCKET_C_LIST}}`: Job alerts, recruiter outreach, unrelated vendor pitches, generic industry news not tied to a named competitor, personal mail.
- `{{CONNECTOR}}`: Gmail (use the Gmail reference row in `connector-capability-map.md` as-is)
- `{{APPROVAL_MODE}}`: AUTO
- `{{MAX_RETRIES}}`: 2
- `{{TIMEZONE}}`: America/Los_Angeles
- `{{OPERATING_WINDOW_START_HOUR}}` / `{{OPERATING_WINDOW_END_HOUR}}`: 6 / 21
- Hourly cron: `33 * * * *`
- End-of-day cron: 6 PM Pacific is `0 1 * * *` UTC the next day during daylight time (UTC-7), or `0 2 * * *` during standard time (UTC-8). Check which is in effect.
- `{{DELIVERY_EMAIL}}`: your own email address, or a shared team alias if this is going to more than one person
- `{{WORKING_FOLDER}}`: your own Google Drive folder ID
- `{{VOICE_DESCRIPTION}}`: Names the specific competitor and the specific claim, distinguishes a competitor's own announcement from press speculation about them, flags when a "launch" is actually a re-announcement of something already shipped.
- `{{BANNED_PHRASES}}`: No vague "competitors are moving fast" framing without naming who. No treating a single social media post as a confirmed strategic shift.
- `{{ANGLE_SECTION_TITLE}}`: COMPETITIVE IMPLICATIONS
- `{{SKILL_SECTION_TITLE}}`: WORTH TESTING (one competitor feature or positioning claim worth trying out or benchmarking against this week)

Everything else follows the templates as written, using the Gmail row of `connector-capability-map.md`. If this digest is going to a team alias rather than one person, revisit `{{APPROVAL_MODE}}`, AUTO sending to a shared inbox with no review step raises the cost of a bad classification call.
