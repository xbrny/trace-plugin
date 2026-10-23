---
name: capture
description: Turn a loose sentence into a trace record — a new item in a project's backlog, a new project, or a floating idea. Use when the user says "add a task…", "remind me to…", "new project for…", "idea: …", or jots something down that should be tracked.
argument-hint: <text>
---

# Capture

Input: $ARGUMENTS (or the user's last message).

1. Decide what it is:
   - **Item**: something to do inside a known project. Status `backlog` unless the user says it's ready (`todo`).
   - **Project**: a new body of work with its own name. Needs a new 2–6 letter uppercase key; propose one.
   - **Idea**: a maybe, with no commitment. An `idea` document, no project needed ("floating").
2. Find the project: call `projects_search` with the name or key the user used (or the project in focus earlier in this conversation). One clear match → use it. Several or none → ask which, listing candidates. For an idea, also check `documents_search` (`kind: ["idea"]`, `q`) so you don't duplicate a recent one; mention a near-duplicate if found.
3. Show what you will create in **one line**, e.g. `Item in CPR (backlog): "Stripe webhook retries", assignee me` and ask for OK. Include assignee, due date or estimate only if the user said them.
4. On OK, write with `skill: "capture"` and a fresh `idempotency_key`:
   - item → `item_save` (`project`, `title`, optional `description`, `assignee`, `due_date`, `estimate_minutes`, `status`, `tag_ids`). Tags the user named: `item_tag` `list` for their ids; a missing one → `item_tag` `create` first, shown in the one-line preview
   - project → `project_save` (`new_key`, `name`, optional `description`, `target_date`)
   - idea → `document_save` (`kind: "idea"`, `title`, optional `body`, optional `project_id`)
5. Reply with the ref / key / title and its `url`.

Rules:

- Published by default. Only create a draft (`is_draft: true`) if the user says "draft".
- Assigning to someone else while saving as a draft: warn first — they won't see it and it counts in nobody's capacity until published.
- Estimates are minutes: man-days × `minutes_per_man_day` from `me` (call it once if you don't know the value).
- Don't invent fields the user didn't give.
- A screenshot for the item or idea: compute size and sha256 locally (`wc -c`, `shasum -a 256`), `file` `start_upload` (with `local_path`), run the `curl` line, `file` `save` attached to the same project (unattached only for a floating idea), then put `![alt](file:<id>)` in the `description` / `body`. Never a download URL: it expires.
