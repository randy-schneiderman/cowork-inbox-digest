# Cowork Inbox Digest

Two scheduled Claude Cowork tasks that turn a firehose of inbox newsletters and alerts into a short, honest digest of what's actually relevant to you: a quiet hourly check plus a deduplicated end-of-day summary emailed to yourself.

Originally built on Gmail and Google Drive. This repo generalizes the setup so you can run it on either Gmail/Google Drive or Microsoft 365 (Outlook/SharePoint), and customize the classification logic for any focus area, not just AI or engineering.

No servers, no code to deploy. Two prompts, a mail connector, and Claude Cowork's scheduled tasks.

## What it does

- **Hourly check**: scans new unread mail since the last run, classifies each thread, and only produces an update artifact if something new and relevant showed up. Silent otherwise. Marks processed mail read so the next run doesn't re-scan it.
- **End of day digest**: rescans the full day (read or unread), deduplicates the same story across multiple newsletters, ranks by how many independent sources covered it, and emails you a formatted summary. Cleans up the scratch copies the hourly runs left behind in your working folder, if you set one up.

## Why two tiers

The hourly check is for real-time signal without noise: it says nothing when there's nothing new, so it never trains you to ignore it. The end of day digest is the record: one email, deduplicated, ranked, with a footer that tells you exactly what got skipped and why. Running both gives you low-latency awareness during the day and a single source of truth at night.

## How it's built

- Two Claude Cowork scheduled tasks (`create_trigger`), each a cron-based job that runs the prompt in a fresh session.
- A 3-bucket classification model in the prompt itself: Bucket A (include in digest), Bucket B (adjacent audience, skip but log why), Bucket C (ignore entirely, never mention).
- Style rules baked into the prompt: no em dashes, no corporate jargon, never invents a fact it didn't read, says "estimate" when it's an estimate, labels vendor claims as vendor claims.
- Connector-agnostic: a capability map (`templates/connector-capability-map.md`) separates "what the automation needs to do" from "which tool call does it on your connector." Gmail and Microsoft 365 are filled in as reference rows; anything else, you fill in your own.
- Explicit error handling: transient failures retry a configurable number of times, permanent failures don't, a broken scan gets you a notification instead of false silence, one bad message doesn't kill the whole run, and a failed delivery never marks messages processed.
- A choice between AUTO (fully unattended) and REVIEW approval modes, so you can run this in review mode until you trust the classification. REVIEW's "nothing marked processed until you approve" guarantee applies to the end-of-day send, where a real email would otherwise go out: it drafts instead of sending, and holds messages unprocessed until you send the draft yourself. The hourly tier marks messages processed in REVIEW mode too, same as AUTO, since it never sends anything outside your own inbox and there's no external action to gate on a human check.

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
  - `sample-output.html` — a real rendered digest, not a mockup, open it directly to see the format before setting anything up.

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

- Classification quality is entirely a function of how specific your Bucket A/B/C definitions are. A vague "AI stuff" definition produces a mediocre digest; a specific one doesn't. A topic-based definition also won't separate by audience level on its own, a technically on-topic beginner tutorial and an advanced one both pass the same Bucket A test unless you add an audience rule.
- On Microsoft 365, mail search and reading in this environment goes through generic MCP resource browsing tools (list/read resource) rather than one dedicated "search mail" tool, as of August 2026. Confirm the exact tools and resource URIs available in your own environment before relying on this — connector tool surfaces change, and this repo cannot promise the names will still be accurate when you read this.
- On Microsoft 365, there's currently no confirmed way to filter the mail scan by "not yet processed," only by folder or date. This means AUTO mode on M365 is running with a real, unverified assumption baked in. Read the note in `connector-capability-map.md` and test it yourself before trusting it unattended; `examples/cloud-advisory-example.md` runs REVIEW mode for exactly this reason.
- REVIEW approval mode depends on your connector having a draft-creation capability. If it doesn't, REVIEW mode isn't available for the send step; AUTO is your only option until one exists. Also note REVIEW's "nothing marked processed until you approve" only applies at the end-of-day tier, see "How it's built" above for the hourly-tier asymmetry.
- Aggregator-style "daily digest" emails from other services (a newsletter platform's own roundup of unrelated headlines with one-line teasers) have no full article body to read. The templates exclude these from classification rather than force a decision from a snippet, and count them separately in the footer.
- No built-in guardrail against an overly broad "include" definition burning time scanning mail that isn't actually relevant. Start narrow, widen later.
- Retries help with transient failures but won't save you from a genuinely broken connector or a revoked authorization. The failure notification is there so you find out, not so it self-heals.
- Everything here depends on Claude Cowork's scheduled tasks product continuing to work the way it does today. If that changes, the templates will need updating.

## Privacy and security

This grants a scheduled, unattended task recurring read access to your inbox, and (in AUTO mode) send access too. Worth being deliberate about before you set it up:

- Start in REVIEW mode, especially on a work inbox, until you've watched it run for a while and trust the classification. Nothing gets sent or permanently marked read without you seeing it first in that mode.
- The scheduled task's prompt is exactly what you pasted into `create_trigger`. Anyone with access to edit your scheduled tasks can edit that prompt, including what it's allowed to read, send, or where it delivers to. Treat the trigger the same way you'd treat any standing credential.
- Bucket C messages are explicitly never read in full, only classified from subject/sender metadata during SCAN, then ignored. If that's not tight enough for something in your inbox (legal, medical, financial), consider a narrower `{{BUCKET_A_DEFINITION}}` or a dedicated mailbox instead of running this against a general-purpose inbox.
- The digest itself, and any email it sends, is only as private as `{{DELIVERY_EMAIL}}` and whoever else has access to your Cowork scheduled task history and run outputs.

## What real output looks like

`examples/sample-output.html` is a real digest built by actually running the hourly template against a live inbox during testing, not a mockup. Open it to see the format before you invest the setup time.

## License

MIT. Use it, fork it, adapt it for your own focus area or connector. Pull requests that add a connector path or improve the interview are welcome.
