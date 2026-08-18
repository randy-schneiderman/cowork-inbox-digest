# Example: platform engineering leader tracking AI

This is the interview filled in for the original use case this repo was built from: a DevOps/platform engineering leader who wants a daily read on AI industry news relevant to the job, without wading through newsletter noise. Folder IDs and the email address below are placeholders, swap in your own.

- `{{PERSON_NAME}}`: your name
- `{{TOPIC_SLUG}}`: AI Digest
- `{{PERSONA_DESCRIPTION}}`: A DevOps/platform engineering leader growing into AI engineering.
- `{{BUCKET_A_DEFINITION}}`: AI content substantively relevant to a platform/DevOps engineering leader. This means AI/ML models and releases, agents, AI infrastructure, AI tooling, AI engineering practice, AI industry news, AI policy affecting engineering, inference cost, observability, eval tooling, MCP, agent orchestration, GPU infrastructure, model serving, CI/CD with AI, supply-chain security for AI systems.
- `{{BUCKET_B_DEFINITION}}`: AI content whose primary audience is solo founders, entrepreneurs, small business owners, or consumers rather than platform engineers. Example: email drip courses on using AI for business growth, AI productivity tips for non-technical audiences, marketing automation with AI.
- `{{BUCKET_C_LIST}}`: Job alerts (even AI-titled roles), recruiter and vendor sales outreach that merely name-drops AI, financial/billing notices, personal mail, school, medical, travel, parking, neighborhood apps, general non-AI DevOps content.
- `{{CONNECTOR}}`: Gmail (use the Gmail reference row in `connector-capability-map.md` as-is)
- `{{APPROVAL_MODE}}`: AUTO (ran in REVIEW for the first two weeks while tuning Bucket A/B/C, then switched over)
- `{{MAX_RETRIES}}`: 2
- `{{TIMEZONE}}`: America/Chicago
- `{{OPERATING_WINDOW_START_HOUR}}` / `{{OPERATING_WINDOW_END_HOUR}}`: 6 / 21 (6 AM to 9 PM Central — these are plain local hours as printed by `TZ='America/Chicago' date '+%H'`, not a UTC offset; adjust to match whatever your own timezone prints for your desired local start/end hour)
- Hourly cron: `41 * * * *`
- End-of-day cron: `0 3 * * *` UTC (10 PM Central during daylight time, UTC-5), or `0 4 * * *` during standard time (UTC-6). Check which is in effect.
- `{{DELIVERY_EMAIL}}`: your own email address
- `{{WORKING_FOLDER}}`: your own Google Drive folder ID
- `{{VOICE_DESCRIPTION}}`: Write in a voice that's thoughtful, pragmatic, analytically rigorous, challenges assumptions, evaluates tradeoffs, no fluff. Bullet points over paragraphs where it fits.
- `{{BANNED_PHRASES}}`: No em dashes anywhere in the text, use commas, colons, or parentheses instead. No corporate jargon: never write "leverage," "synergy," "deep dive," "at the end of the day," "game-changer," "unlock," or "landscape."
- `{{ANGLE_SECTION_TITLE}}`: PLATFORM / DEVOPS ANGLE
- `{{SKILL_SECTION_TITLE}}`: SKILL-BUILDING

Everything else follows the templates as written, using the Gmail row of `connector-capability-map.md`.
