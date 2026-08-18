# Connector capability map

The templates don't hardcode a connector. They call five capabilities by name, and this file is where you map each one to an actual tool call for whatever you're connecting to. Fill in your row, then paste the five values into the `{{...}}` placeholders in the templates.

Every connector this automation works with needs to support the first four. The fifth (scratch storage) is optional.

| # | Capability | What it needs to do |
|---|---|---|
| 1 | SCAN | List new/unread messages in the inbox, ideally filterable by "since last check" or "unread" |
| 2 | READ | Fetch the full plain-text body of one specific message, not just a snippet |
| 3 | MARK_PROCESSED | Tag or flag a message so the next SCAN doesn't return it again |
| 4 | SEND / DRAFT | Send an email, and separately, create a draft without sending (needed for REVIEW approval mode) |
| 5 | STORAGE (optional) | Upload a file to a scratch folder (each hourly artifact gets a copy saved here), then list and delete files in that folder (the end-of-day run sweeps it clean) |

## Reference: Gmail

| # | Tool call |
|---|---|
| 1 | `mcp__Gmail__search_threads` — query `in:inbox is:unread newer_than:1d` (hourly) or `in:inbox newer_than:1d` (end of day), pageSize 50 |
| 2 | `mcp__Gmail__get_thread` — messageFormat `PLAIN_TEXT` |
| 3 | `mcp__Gmail__unlabel_message` — labelIds `["UNREAD"]` |
| 4 | `mcp__Gmail__send_message` (send) / `mcp__Gmail__create_draft` (draft) |
| 5 | `mcp__Google_Drive__create_file` (upload, with `parents: ['<folder id>']`) / `mcp__Google_Drive__search_files` (list, query `parentId = '<folder id>'`) / `mcp__Google_Drive__trash_file` (delete) |

## Reference: Microsoft 365

| # | Tool call |
|---|---|
| 1 | Mail resource browsing (`ListMcpResourcesTool` / `ReadMcpResourceDirTool` against server `Microsoft_365`) — **verify the exact resource URI in your own environment before relying on this.** As of this writing there is no single dedicated "search mail" tool exposed the way Gmail has one; this may have changed by the time you read this. |
| 2 | `ReadMcpResourceTool` on the message resource |
| 3 | `mcp__Microsoft_365__outlook_modify_thread_labels` — add/remove a category you designate as your "processed" marker |
| 4 | `mcp__Microsoft_365__outlook_send_mail` (send, `bodyType: "html"`) / a draft-creation tool if your environment exposes one — check before assuming; if it doesn't, REVIEW mode isn't available on M365 until it does, and AUTO is your only option |
| 5 | `mcp__Microsoft_365__sharepoint_upload_file` (upload) / `mcp__Microsoft_365__sharepoint_*` tools (list/delete/move an item) |

**Known gap, not yet resolved, read before running AUTO mode:** the SCAN row above (mail resource browsing) has no confirmed way to filter *by category*, only by folder/date in whatever the resource URI exposes. That means there's no verified way to make SCAN skip messages already marked processed via the category in row 3, the mechanic the rest of this repo assumes works the same way Gmail's `is:unread` filter does. Until you've confirmed otherwise in your own environment, assume M365 SCAN may re-return already-processed messages, and MARK_PROCESSED just relabels them without actually excluding them from the next scan, meaning the same messages could get reclassified (though not re-delivered, since bucket assignment is idempotent) every run. Test this specifically before trusting AUTO mode unattended: run SCAN, mark one result processed, run SCAN again, confirm it's actually gone. If it isn't, either find the right filter parameter for your environment, fall back to a date-range filter (only scan messages received since the last run) instead of relying on the category, or stay in REVIEW mode where a human is checking the output anyway.

Note on M365 email sending: `outlook_send_mail`'s HTML sanitizer strips `<style>` blocks and a handful of other tags, as of August 2026 — check whether this is still accurate in your own environment before assuming it. If your digest's visual styling depends on a `<style>` block, either inline every style as a per-element `style="..."` attribute, or accept the emailed copy will look plainer than the in-app artifact and treat the artifact as the real deliverable.

## Other connectors

There's no reference row here because it depends entirely on what you're connecting to. Before writing your row:

1. In an ordinary (non-scheduled) Cowork conversation, ask what mail-related tools are available, or use `ToolSearch` with a query like "email search read send" to see what's actually exposed.
2. Manually test each of the four required capabilities once, by hand, in that conversation. Confirm SCAN actually returns new messages, READ actually returns a full body (not a snippet), MARK_PROCESSED actually prevents a message from reappearing in the next SCAN, and SEND actually delivers. Don't skip this: putting an untested tool call into an unattended cron job is how you get a scheduled task that fails silently for weeks.
3. If you can't find a MARK_PROCESSED equivalent, this automation will re-read and re-classify the same messages every run. That's wasteful but not broken; just be aware of it, and consider a workaround like filtering SCAN by date/time of last run instead of a processed flag.
4. Write your findings into a third row in the tables above, and keep this file updated if the tool names change later; connector tool surfaces are not stable indefinitely.

| # | Tool call (fill in) |
|---|---|
| 1 | |
| 2 | |
| 3 | |
| 4 | |
| 5 | |
