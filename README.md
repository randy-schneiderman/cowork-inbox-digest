# Cowork Inbox Digest

Two scheduled Claude Cowork tasks that turn a firehose of inbox newsletters and alerts into a short, honest digest of what's actually relevant to you: a quiet hourly check plus a deduplicated end-of-day summary emailed to yourself.

Originally built on Gmail and Google Drive. This repo generalizes the setup so you can run it on either Gmail/Google Drive or Microsoft 365 (Outlook/SharePoint), and customize the classification logic for any focus area, not just AI or engineering.

No servers, no code to deploy. Two prompts, a mail connector, and Claude Cowork's scheduled tasks.

## What it does

- **Hourly check**: scans new unread mail since the last run, classifies each thread, and only produces an update artifact if something new and relevant showed up. Silent otherwise. Marks processed mail read so the next run doesn't re-scan it.
- **End of day digest**: rescans the full day (read or unread), deduplicates the same story across multiple newsletters, ranks by how many independent sources covered it, and emails you a formatted summary. Cleans up its own scratch files when done.

## Why two tiers

The hourly check is for real-time signal without noise: it says nothing when there's nothing new, so it never trains you to ignore it. The end of day digest is the record: one email, deduplicated, ranked, with a footer that tells you exactly what got skipped and why. Running both gives you low-latency awareness during the day and a single source of truth at night.

## How it's built

- Two Claude Cowork scheduled tasks (`create_trigger`), each a cron-based job that runs the prompt in a fresh session.
- A 3-bucket classification model in the prompt itself: Bucket A (include in digest), Bucket B (adjacent audience, skip but log why), Bucket C (ignore entirely, never mention).
- Style rules baked into the prompt: no em dashes, no corporate jargon, never invents a fact it didn't read, says "estimate" when it's an estimate, labels vendor claims as vendor claims.
- Connector-agnostic: a capability map (`templates/connector-capability-map.md`) separates "what the automation needs to do" from "which tool call does it on your connector." Gmail and Microsoft 365 are filled in as reference rows; anything else, you fill in your own.
- Explicit error handling: transient failures retry a configurable number of times, permanent failures don't, a broken scan gets you a notification instead of false silence, one bad message doesn't kill the whole run, and a failed delivery never marks messages processed.
- A choice between AUTO (fully unattended) and REVIEW (drafts instead of sending, nothing marked processed until you approve) approval modes, so you can run this in review mode until you trust the classification.

## Repo structure

- `INTERVIEW.md` — answer these questions first. This is what actually customizes the automation to you, including which connector you're on, approval mode, and error handling defaults.
- `templates/connector-capability-map.md` — maps five required capabilities (scan, read, mark processed, send/draft, optional scratch storage) to actual tool calls. Reference rows for Gmail and Microsoft 365; a blank one for anything else.
- `templates/hourly-check.template.md` and `templates/end-of-day-digest.template.md` — the two prompts, connector-agnostic, with `{{PLACEHOLDER}}` tokens for your interview answers and capability map.
- `SETUP.md` — turn your filled-in templates into working scheduled tasks, by hand, one step at a time.
- `SETUP-PROMPT.md` — the same setup, faster: a prompt you paste into a Cowork conversation that runs the interview live, fills the templates, shows you the result, and creates the scheduled tasks after you confirm.
- `examples/` — five filled-in examples across different focus areas and connectors, for reference:
  - `platform-engineering-example.md` — DevOps/platform engineering leader tracking AI industry news. Gmail, AUTO mode.
  - `ai-engineering-example.md` — hands-on AI engineer tracking implementation techniques, not industry news. Gmail, REVIEW mode.
  - `cloud-advisory-example.md` — infrastructure lead tracking AWS/Azure/GCP incidents and advisories. Microsoft 365, AUTO mode, the filled-in M365 path.
  - `financial-markets-example.md` — individual investor tracking macro conditions and specific holdings. Gmail, AUTO mode.
  - `competitive-intelligence-example.md` — product/strategy lead tracking a named set of competitors. Gmail, AUTO mode.

## Prerequisites

- Claude Cowork access with scheduled tasks enabled.
- A connected mail connector: Gmail, Microsoft 365, or another one you've mapped in `connector-capability-map.md`.
- A scratch folder the connector can write to during a run, if your connector supports one. Optional otherwise.
- For the end-of-day tier: permission to send mail (AUTO mode) or create drafts (REVIEW mode) from that account.

## Quick start

Two paths to the same result. Pick one.

**Fast path**: get this repo into a Cowork session's workspace, then paste `SETUP-PROMPT.md` into the conversation. It interviews you live, fills the templates, shows you exactly what it's about to create, and only creates the two scheduled tasks after you confirm.

**Manual path**, more auditable, slower:
1. Read `INTERVIEW.md` and answer the questions, including which connector, which approval mode, and how many retries. Twenty minutes, most of it is deciding what counts as in scope.
2. Fill in `templates/connector-capability-map.md` for your connector.
3. Copy the two files in `templates/` and fill in your answers and capability map.
4. Follow `SETUP.md` to create the two scheduled tasks.
5. Let the hourly one run for a day, then confirm the end-of-day digest (or draft, in REVIEW mode) shows up.
6. Tune. Your first Bucket A/B/C split will be wrong in some way. Adjust the interview answers and update the trigger prompt.

Either way, review what you're about to create before it starts reading your inbox. `SETUP-PROMPT.md` builds a confirmation step in for exactly this reason.

## Known limitations

- Classification quality is entirely a function of how specific your Bucket A/B/C definitions are. A vague "AI stuff" definition produces a mediocre digest; a specific one doesn't.
- On Microsoft 365, mail search and reading in this environment goes through generic MCP resource browsing tools (list/read resource) rather than one dedicated "search mail" tool, as of August 2026. Confirm the exact tools and resource URIs available in your own environment before relying on this — connector tool surfaces change, and this repo cannot promise the names will still be accurate when you read this.
- Outlook categories and Gmail labels are not a 1:1 mapping. The "mark processed mail so it's not rescanned" mechanic needs a small adjustment on M365. See the capability map.
- REVIEW approval mode depends on your connector having a draft-creation capability. If it doesn't, REVIEW mode isn't available for the send step; AUTO is your only option until one exists.
- No built-in guardrail against an overly broad "include" definition burning time scanning mail that isn't actually relevant. Start narrow, widen later.
- Retries help with transient failures but won't save you from a genuinely broken connector or a revoked authorization. The failure notification is there so you find out, not so it self-heals.
- Everything here depends on Claude Cowork's scheduled tasks product continuing to work the way it does today. If that changes, the templates will need updating.

## License

MIT. Use it, fork it, adapt it for your own focus area or connector. Pull requests that add a connector path or improve the interview are welcome.
