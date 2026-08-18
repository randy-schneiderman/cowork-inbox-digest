# Example: AI engineer tracking implementation details

A hands-on AI engineer who wants technical depth on building and shipping LLM features, not industry news or strategy takes. Different audience than the platform engineering example: this one wants "does this technique actually work," not "is AI a big deal."

- `{{PERSON_NAME}}`: your name
- `{{TOPIC_SLUG}}`: AI Engineering Digest
- `{{PERSONA_DESCRIPTION}}`: A hands-on AI engineer building and shipping LLM-based features, focused on practical implementation over industry news.
- `{{BUCKET_A_DEFINITION}}`: Model releases and benchmarks with real capability deltas (not just a new version number), prompting and context engineering techniques, RAG and retrieval architecture, agent frameworks and orchestration patterns, eval and testing methodology for LLM applications, fine-tuning and distillation, token cost and latency optimization, open-source model releases and their licensing terms, vector database and embedding updates, structured output and tool-use patterns.
- `{{BUCKET_B_DEFINITION}}`: AI content aimed at business leaders or non-technical audiences (strategy think-pieces, "AI transformation" content, high-level AI news without technical depth), plus general software engineering content that only tangentially mentions AI.
- `{{BUCKET_C_LIST}}`: Job alerts, recruiter outreach, vendor sales pitches with no technical substance, AI-generated spam newsletters, general tech news unrelated to AI/ML, personal mail.
- `{{CONNECTOR}}`: Gmail (use the Gmail reference row in `connector-capability-map.md` as-is)
- `{{APPROVAL_MODE}}`: REVIEW (kept in review mode longer than usual here, since a wrong call on "does this benchmark actually mean anything" is easy to make early on)
- `{{MAX_RETRIES}}`: 2
- `{{TIMEZONE}}`: America/New_York
- `{{OPERATING_WINDOW_START_HOUR}}` / `{{OPERATING_WINDOW_END_HOUR}}`: 7 / 22 (roughly 7 AM to 10 PM Eastern, adjust to match what `date` prints for your timezone)
- Hourly cron: `22 * * * *`
- End-of-day cron: 9 PM Eastern is `0 1 * * *` UTC during daylight time (UTC-4), or `0 2 * * *` during standard time (UTC-5). Check which is in effect.
- `{{DELIVERY_EMAIL}}`: your own email address
- `{{WORKING_FOLDER}}`: your own Google Drive folder ID
- `{{VOICE_DESCRIPTION}}`: Technical and precise, skips the hype, evaluates whether a technique actually works before recommending it, prefers code-level or architectural detail over marketing claims.
- `{{BANNED_PHRASES}}`: No "game-changer," "revolutionize," "10x," or similar hype language. No stating an unverified benchmark claim as settled fact, flag it as a claim instead.
- `{{ANGLE_SECTION_TITLE}}`: IMPLEMENTATION NOTES
- `{{SKILL_SECTION_TITLE}}`: THIS WEEK'S BUILD

Everything else follows the templates as written, using the Gmail row of `connector-capability-map.md`.
