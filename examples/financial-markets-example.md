# Example: personal markets digest

An individual investor tracking macro conditions and specific holdings, not a professional trader, who wants signal without hype or a brokerage trying to sell something. This is a template for organizing publicly available newsletter content, not investment advice, and the digest itself should say so.

- `{{PERSON_NAME}}`: your name
- `{{TOPIC_SLUG}}`: Markets Digest
- `{{PERSONA_DESCRIPTION}}`: An individual investor tracking macro market conditions and a specific set of holdings, not a professional trader.
- `{{BUCKET_A_DEFINITION}}`: Federal Reserve policy and interest rate decisions; macro indicators (inflation, employment, GDP) and how markets actually reacted to them; earnings from specifically tracked companies or sectors; material developments for specifically tracked holdings; credible analysis of market-moving events; changes in composition or fees for funds or ETFs actually held.
- `{{BUCKET_B_DEFINITION}}`: Generic "stocks to watch" content, promotional newsletters from brokerages pushing a product, general personal finance content (budgeting, credit card rewards) not tied to markets or investing.
- `{{BUCKET_C_LIST}}`: Job alerts, unrelated retail promotions, crypto hype content with no substantive analysis, clickbait ("this one stock could 10x"), personal mail.
- `{{CONNECTOR}}`: Gmail (use the Gmail reference row in `connector-capability-map.md` as-is)
- `{{APPROVAL_MODE}}`: AUTO
- `{{MAX_RETRIES}}`: 2
- `{{TIMEZONE}}`: America/Chicago
- `{{OPERATING_WINDOW_START_HOUR}}` / `{{OPERATING_WINDOW_END_HOUR}}`: 3 / 20 (covers premarket through after-hours coverage arriving as newsletters, not live trading)
- Hourly cron: `50 * * * *` (mainly catches breaking macro news; this persona doesn't day-trade, so the hourly tier matters less here than the end-of-day one, consider narrowing the operating window further or skipping the hourly tier entirely if that's true for you)
- End-of-day cron: 5 PM Central (after US market close) is `0 22 * * *` UTC during daylight time (UTC-5), or `0 23 * * *` during standard time (UTC-6). Check which is in effect.
- `{{DELIVERY_EMAIL}}`: your own email address
- `{{WORKING_FOLDER}}`: your own Google Drive folder ID
- `{{VOICE_DESCRIPTION}}`: Skeptical of hype, careful to distinguish "the market moved because of X" (a causal claim, often overstated) from "X happened and the market also moved" (correlation), presents facts and leaves the decision to you rather than recommending trades. Every digest footer should include a one-line reminder that this is informational only, not financial advice.
- `{{BANNED_PHRASES}}`: No "to the moon," no unqualified predictions stated as certainties, no cherry-picked hindsight comparisons.
- `{{ANGLE_SECTION_TITLE}}`: WHAT ACTUALLY MOVED THE MARKET
- `{{SKILL_SECTION_TITLE}}`: WORTH DIGGING INTO

Everything else follows the templates as written, using the Gmail row of `connector-capability-map.md`. This example is a starting point for organizing newsletter content you already subscribe to; it is not a substitute for professional financial advice, and the digest it produces shouldn't be treated as one either.
