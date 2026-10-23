---
name: plan
description: Break a goal, document or idea into initiatives and estimated items in trace, check them against capacity, and write the approved plan in one call. Use when the user says "plan this", "break this down", "turn this into tasks", or gives a goal, spec or idea to schedule.
argument-hint: [project or text]
---

# Plan

Input: $ARGUMENTS — a project, a goal in text, a document id, or an idea.

1. Gather context:
   - Project: `projects_search` / `project_get` (existing initiatives, totals, baselines). Ask if unclear.
   - A document or idea: `document_get`. Existing decisions: `documents_search` `kind: ["decision"]`, `project`.
   - Existing open items so you don't duplicate: `items_search` (`project`, `status: ["backlog","todo","in_progress"]`).
   - `me` for `minutes_per_man_day` and users.
2. Draft the plan. Group items under initiatives (new or existing). Each item: title, estimate in man-days, start and due dates if known, assignee (default me).
   - Items over ~3 man-days: suggest a split — **siblings** when the parts are independent, **sub-items** when they finish one deliverable together. A parent row carries no estimate of its own; show its sub-items' sum in brackets.
3. Capacity: send **every** item in one `capacity` call with `check` (`key` = row number, `assignee`, `estimate_minutes`, `due_date`, `start_date`). Add one line per assignee: "Fits" or "Short by 1.5 md before 14 Nov".
4. Show the proposal table, nothing written yet:

| #   | Initiative | Parent | Item                   | Est.   | Start | Due    | Assignee | Note           |
| --- | ---------- | ------ | ---------------------- | ------ | ----- | ------ | -------- | -------------- |
| 1   | Payments   |        | Stripe webhook handler | 2 md   |       | 10 Nov | me       |                |
| 2   | Payments   |        | Refunds                | (4 md) |       | 14 Nov | me       | Sum of 2a + 2b |
| 2a  | Payments   | 2      | Refund API             | 2 md   |       | 12 Nov | me       |                |
| 2b  | Payments   | 2      | Refund UI              | 2 md   |       | 14 Nov | me       |                |

5. The user answers `ok`, `ok without 2`, `change 1 to 3 md`, etc. Apply edits, re-run the capacity check if estimates, dates or assignees changed, and show the table again until `ok`.
6. On `ok`: **one** `plan_apply` call, `skill: "plan"`, a fresh `idempotency_key`:
   - `initiatives`: new ones with `tmp_id` and `baseline: true`; existing initiatives are referenced by their id in item rows.
   - `items`: `tmp_id`, `title`, `initiative_id` (tmp_id or real id), `parent_item_id` (a parent's tmp_id for sub-items), `estimate_minutes` (not on parents), `start_date`, `due_date`, `assignee`, `status: "todo"`, `tag_ids` only for tags the user asked for (`item_tag` `list` / `create` before the call).
   - `BULK_INVALID` → nothing was written; fix the listed rows and resend the whole plan.
   - Existing items joining a new initiative: after `plan_apply`, move them with `item_save` (`refs`, `initiative_id`), then `initiative_save` `baseline: true` so the baseline includes them.
7. Reply with the created refs and the project `url`.

Rules: published unless the user says "draft". A baseline counts only work everyone can see, so in a draft project (or for draft items) it reads 0 — say so, and re-baseline after publishing. Assigning to someone else in a draft plan: warn — they won't see it, and it counts in nobody's capacity until published.
