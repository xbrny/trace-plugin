---
name: report
description: Write a status report or update email for a project and period from trace data, revise it with the user, approve it, and record when it was sent. Use for "weekly report for CPR", "status update for the client", "draft the email to Acme about last week".
argument-hint: [project] [period] [audience]
---

# Report

Input: $ARGUMENTS — project (or all), period (default: last full week, Mon–Fri, in the user's timezone), audience, and `status_report` (default) or `email`.

1. `report_data` (`project`, `kind`, `from`, `to`) is the **only** source. Don't add facts it doesn't contain. It returns only published work: if the project or its items are drafts, it comes back empty by design — say so instead of reporting "no activity".
2. Write for the audience: client → outcomes and dates, no internal names or refs unless asked; internal → refs, owners, blockers. Usual sections: Summary, Done, In progress, Blocked (with what unblocks it), Risks and decisions, Next. Note what changed since the previous report if `report_data` returns one. An email also needs a subject.
3. Save the draft as soon as you first show it: `report` `op: "save"`, `skill: "report"`, `kind`, `title`, `body` (Markdown), `project_id`, `audience`, `period_from`, `period_to`, `email_subject` (email), `source_refs` (refs you used), `idempotency_key`. Reports always start as drafts.
4. Each edit the user accepts in chat → `report` `op: "save"` with `id`, `version` and the changed fields.
5. Approve only when the user says so: `report` `op: "approve"`. If it warns `REPORT_REFERENCES_DRAFTS`, show the listed records and ask; resend with `confirm_draft_refs: true` only on yes. Never override a warning silently. After approving a project report, if `project_get` lists guests, offer once to share it with them (`document_save` `id`, `version`, `share: true`, on OK).
6. When the user says they sent it: `report` `op: "mark_sent"`, `sent_to`, optional `sent_at`, `remark`.
7. Reply with the `url` after each save.

Screenshots the user hands over: compute size and sha256 locally (`wc -c`, `shasum -a 256`), `file` `start_upload` (with `local_path`), run the `curl` line, `file` `save` with the report's `project_id` (none for an all-projects report), then put `![alt](file:<id>)` in the body. Never a download URL: it expires.

Approved reports are frozen. To change one, a superseding report is needed; the plugin can't do that yet — tell the user.
