---
name: note
description: Keep a note, an idea or a decision in trace, or upload a file with a summary note, and share documents with a project's guests. Use when the user says "note that…", "we decided…", "keep this", "save this doc/file", "share this with the client", or hands over meeting notes, a decision or a file path to store against a project, initiative or item.
argument-hint: <text or file path>
---

# Note

Input: $ARGUMENTS (text, a decision, or a local file path).

1. Pick the kind:
   - `decision`: a choice that was made ("we'll use Stripe"). Decisions are frozen once published.
   - `idea`: a maybe.
   - `note`: anything else worth keeping (meeting notes, findings).
   - A **file path** → upload the file and add a short summary `note` linked to it (step 4).
   - A **screenshot or image** that belongs inside the text → inline it in the body (step 4).
2. Find what it attaches to: a project (`projects_search`), an initiative (`project_get` with `initiative`), or an item (`item_get` / `items_search`). Use the project in focus if the user didn't name one. Ideas may float without a project. Optional folder `path` (lowercase segments joined by `/`) if the user keeps folders.
3. For a decision, check existing decisions on the topic: `documents_search` (`kind: ["decision"]`, `project`, `q`). If one covers the same topic, propose **superseding** it instead of creating a second one.
4. Show kind, title and attachment in **one line**; save on OK with `skill: "note"`:
   - text → `document_save` (`kind`, `title`, `body` in Markdown, `project_id`, `initiative_id` or `item_id`, `path`; `idempotency_key`)
   - supersede → `document_save` (`id` of the old decision, its `version`, `supersede: true`, new `title`/`body`)
   - file → compute size and sha256 locally (`wc -c`, `shasum -a 256`), `file` `start_upload` (with `local_path`), run the returned `curl` line, then `file` `save` (`upload_id`, `project_id`, …). Then create the summary `note` and link it: `link` `add`, `type: "derived_from"`, `from` the note, `to` the file.
   - image inside a body → upload it as above (`file` `save` attached where the document is; unattached only for a floating idea or report), then write `![alt](file:<id>)` in a Markdown `body` or `<img src="file:<id>" alt="…">` in HTML. Never paste a download URL: it expires in 5 minutes, the `file:` ref resolves on every read. PNG, JPEG, GIF and WebP render.
5. Reply with title and `url`.

**Sharing with the client** ("share the API decision with the client", "unshare that note"): find the document (`documents_search`), then `project_get` for its project's guests. Show one line — title, and who'll see it (guest names), or "no guests yet" — and on OK `document_save` with `id`, `version`, `share: true` (or `false`). Any kind in a project, reports included; guests read the current version only, never drafts, history or what it's attached to. Uploaded files can't be shared; images inside the body show.

Rules: published unless the user says "draft". Keep the user's wording for decisions; summarise only notes you were asked to summarise.
