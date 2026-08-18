# Example: cloud infrastructure advisories and alerts

An SRE or infrastructure lead who needs to know about provider incidents, security advisories, and deprecations across AWS, Azure, and GCP, fast, without digging through vendor marketing mail to find them. Uses the Microsoft 365 connector against a work inbox, unlike the other examples here, to show that path filled in for real.

- `{{PERSON_NAME}}`: your name
- `{{TOPIC_SLUG}}`: Cloud Advisory Digest
- `{{PERSONA_DESCRIPTION}}`: A cloud infrastructure lead responsible for uptime, security, and cost across AWS, Azure, and GCP.
- `{{BUCKET_A_DEFINITION}}`: Cloud provider status incidents and post-incident reports for AWS, Azure, or GCP; security advisories and CVEs affecting cloud services actually in use; deprecation and end-of-life notices for services actually in use; pricing or billing model changes; new service or region launches relevant to current architecture; compliance and certification updates (SOC2, FedRAMP, etc.) for vendors actually in use; capacity or quota changes.
- `{{BUCKET_B_DEFINITION}}`: General cloud "best practices" marketing content, vendor newsletters promoting unrelated products, generic cloud cost-optimization listicles not tied to specific services in use.
- `{{BUCKET_C_LIST}}`: Job alerts, recruiter outreach, unrelated SaaS vendor pitches, generic tech news, personal mail, internal company all-hands content unrelated to infrastructure.
- `{{CONNECTOR}}`: Microsoft 365 (use the M365 row of `connector-capability-map.md`; work inbox)
- `{{APPROVAL_MODE}}`: AUTO (these are factual alerts, not judgment calls, low value in a human review step)
- `{{MAX_RETRIES}}`: 3 (higher than the default here; for an alerting use case, a missed advisory because of a transient API hiccup is worse than a slightly slower run)
- `{{TIMEZONE}}`: America/Denver
- `{{OPERATING_WINDOW_START_HOUR}}` / `{{OPERATING_WINDOW_END_HOUR}}`: 5 / 22 (wider than usual since infrastructure incidents don't wait for business hours)
- Hourly cron: `15 * * * *`
- End-of-day cron: 6 PM Mountain is `0 0 * * *` UTC during daylight time (UTC-6), or `0 1 * * *` during standard time (UTC-7). Check which is in effect.
- `{{DELIVERY_EMAIL}}`: your work email address
- `{{WORKING_FOLDER}}`: your own SharePoint or OneDrive folder path
- `{{VOICE_DESCRIPTION}}`: Terse and operational, leads with impact and whether action is required, treats every included item as something that might need a ticket opened.
- `{{BANNED_PHRASES}}`: No vague reassurance language ("nothing to worry about") that buries the actual severity. No marketing tone on what is functionally an incident feed.
- `{{ANGLE_SECTION_TITLE}}`: ACTION REQUIRED
- `{{SKILL_SECTION_TITLE}}`: RUNBOOK UPDATE (one thing to add or update in the team's incident runbook this week, based on what the day's advisories actually revealed)

Everything else follows the templates as written, using the Microsoft 365 row of `connector-capability-map.md`. Since this example uses M365, read the note in that file about the sanitized email body stripping `<style>` blocks before assuming the emailed copy will look like the artifact.
