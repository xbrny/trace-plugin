---
name: triage
description: Triage client issues in trace — new items from issues, link to existing items, mark duplicates, decline, reopen — and log issues for a client or add project people (guests, members). Use for "triage CPR", "CPR-C4 is a dup of C2", "C5 goes into CPR-12", "decline C6, out of contract", "log this for Acme", "invite ali@client.com to CPR", "add Mei to CPR".
argument-hint: [project or dictated triage]
---

# Triage

Input: $ARGUMENTS (or the user's message). Every write carries `skill: "triage"`.

## Find the issues

- Refs (`CPR-C4`) match exactly → `client_issues` with `ref` (gives the `version`). Bare refs ("C5", "4 and 6") only within the project in focus earlier in this conversation; otherwise ask which project.
- Titles → `client_issues` with `project` and `q`. A fuzzy match is shown back and written only after the user confirms.

## Dictated triage: save directly, then echo

The user states the decision; map each one to one `client_issue_triage` call:

| Said                          | Call                                                                              |
| ----------------------------- | --------------------------------------------------------------------------------- |
| "C4 is a dup of C2"           | `action: "duplicate"`, `duplicate_of: "CPR-C2"`, `reason` if said                 |
| "C5 goes into CPR-12"         | `action: "link"`, `item_id: "CPR-12"`                                             |
| "make an item for C7"         | `action: "new_item"`, `item` with any fields said (title defaults to the issue's) |
| "decline C6, out of contract" | `action: "decline"`, `reason` (required; ask if none was given)                   |
| "reopen C6"                   | `action: "reopen"`                                                                |

Tags ("tag C4 checkout", "untag C5"): `client_issue_tag` `list` for the project's tag ids, then `client_issue_save` with the full new `tag_ids`. A tag that doesn't exist yet → `client_issue_tag` `create` first, and say so in the echo.

Several decisions in one message → one call each, **one** summary. Anything you'd have to infer (which item, which original, a reason) → ask instead.

## Proposed triage: table first

`/trace:triage <project>` with no decisions:

1. Read `client_issues` (`project`, `status: ["new"]`): the untriaged ones. Open each with `ref` for its description.
2. Read the project's open items to suggest links: `items_search` (`project`, `status: ["backlog","todo","in_progress"]`).
3. Propose one decision per issue. Nothing written yet:

| #   | Issue                         | Action    | Item or target                    | Reason                     |
| --- | ----------------------------- | --------- | --------------------------------- | -------------------------- |
| 1   | CPR-C4 Safari checkout broken | duplicate | CPR-C2                            | Same report                |
| 2   | CPR-C5 Export to CSV          | link      | CPR-12 Export page                | Covered by the export work |
| 3   | CPR-C7 Login timeout          | new_item  | "Fix login session timeout", high |                            |
| 4   | CPR-C6 Native app             | decline   |                                   | Out of contract            |

- `new_item` rows: title (default the issue's), priority from the issue, status `backlog` unless the user says otherwise. Estimates and dates come later (`/trace:plan` or `/trace:track`).
- A duplicate's target must be the original, never an issue that is itself a duplicate.

4. The user answers `ok`, `ok without 2`, `change 1 to link CPR-12`, etc. Apply edits and show the table again until `ok`.
5. On `ok`: one `client_issue_triage` call per row. Rows are independent: a failed row is reported with its error and hint, the rest stand. Don't retry failed rows unasked.

## Errors

Follow the error's `hint`. In particular:

- `ISSUE_HAS_ITEMS` (duplicate / decline with linked items): show `details.items` and ask whether to unlink them first (`link` op `remove`, link id from `item_get`).
- `DUPLICATE_CHAIN`: point at the original the hint names; ask before re-pointing other issues.
- `ISSUE_CLOSED`: the issue is declined or a duplicate; ask before reopening it.
- `VERSION_CONFLICT`: re-read; if the decision still makes sense on the new state, retry once; otherwise tell the user what changed.

## Logging for a client, project people

One line each, saved on OK:

- Log an issue on a client's behalf (from an email or a call): `client_issue_save` with `project`, `title`, `type`, `priority`, `description`. You become the reporter; start the description with "Reported by <name>". Show: `Log in CPR: "Checkout fails on Safari" (bug, high), reported by Ali`.
- Invite or remove a guest: `guest` `invite` (`email`, `name` for someone new) or `remove` (`guest`). Show: `Invite Ali <ali@client.com> to CPR as a guest`. Guests see every client issue in the project, never items.
- Add or remove a member: `member` `add` or `remove` (`member`: exact name, email or id of an existing trace member). Show: `Add Mei <mei@studio.test> to CPR as a member`. Members see everything in the project. Removing someone moves their open items to the project owner; say so in the line. The owner can't be removed. New member accounts aren't made here: the operator adds them with `user:add`.

## Summary

One line per issue, e.g. `CPR-C4 → duplicate of CPR-C2`, `CPR-C5 → linked to CPR-12 (planned)`, `CPR-C7 → new item CPR-31 (planned)`, `CPR-C6 → declined: out of contract`. Include the status the issue now shows the client, and each issue's `url`.
