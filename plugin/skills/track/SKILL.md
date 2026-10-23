---
name: track
description: Record dictated progress on trace items — done, started, blocked, unblocked, cancelled, new dates or estimates. Use when the user states facts like "CPR-12 is done", "started on the webhook", "CPR-14 blocked on client keys", "move CPR-9 to Friday", "CPR-12 is 3 days now".
argument-hint: <what happened>
---

# Track

Input: $ARGUMENTS (or the user's message). Dictated facts are saved **directly**, then echoed. Ask only about what you'd have to infer.

1. Find each item:
   - Refs (`CPR-12`) match exactly → `item_get` (gives the `version`).
   - Bare numbers ("12 and 13") only within the project in focus earlier in this conversation; otherwise ask which project.
   - Titles → `items_search` with `q`. A fuzzy match is shown back ("You mean CPR-12 Stripe webhook?") and written only after the user confirms.
2. Map each fact to one write (`skill: "track"`):
   - done / started / back to todo / backlog → `item_action` `action: "status"` (`done`, `in_progress`, `todo`, `backlog`)
   - blocked → `item_action` `action: "block"`, `reason` (required), `unblocker` if said, `blocked_by` if another item is the blocker (single item only)
   - unblocked → `item_action` `action: "unblock"`
   - cancelled / dropped → `item_action` `action: "cancel"`, `reason` if said
   - dates, estimate, assignee, priority → `item_save` (`ref`, `version`, fields). Estimates in minutes (man-days × `minutes_per_man_day` from `me`).
   - Same change on several items → one call with all of them (`items` / `refs`).
   - tag / untag ("tag CPR-12 mobile") → `item_tag` `list` for the project's tag ids, then `item_save` with the full new `tag_ids`. A tag that doesn't exist yet → `item_tag` `create` first, and say so in the summary.
3. Several facts in one message → several writes, **one** summary.
4. Warnings:
   - "Done" on a parent with open sub-items → `PARENT_HAS_OPEN_SUBITEMS`: show which sub-items are open and ask. Resend with `confirm_open_subitems: true` only on yes.
   - `VERSION_CONFLICT` → re-read; if the change still makes sense on the new state, retry once; otherwise tell the user what changed.
5. Estimate dictated ("CPR-12 is 3 days now") → save it and echo the drift against the original: "CPR-12: 2 md → 3 md (original 2 md, +50%)". An estimate **you** suggest is never saved without OK.
6. After any date or estimate change, call `risks` (`project`) and add a one-line warning for each new `target_at_risk` or `over_capacity` touching those items.

Summary format: one line per item, e.g. `CPR-12 Stripe webhook → done`, `CPR-14 → blocked: client keys (waiting on Acme)`.
