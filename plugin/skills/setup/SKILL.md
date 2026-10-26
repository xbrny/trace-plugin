---
name: setup
description: Check the trace connection, show my timezone and availability, offer to fix them, suggest permission rules, and invite, deactivate or reactivate trace members. Use after installing the trace plugin, when trace tools fail to connect, when the user wants to change their timezone, work days or focus, or says "invite Sam to trace", "deactivate Mei", "cancel Ana's invite".
---

# trace setup

1. Call `me`.
   - Fails with 401 / UNAUTHENTICATED → the token is wrong, expired or revoked. Tell the user to create a token on trace's Account page and re-enter it with `/plugin` → trace → Configure. Stop.
   - Fails to connect → `trace_url` is wrong or trace is down. Show the URL the plugin uses and stop.
2. Show in one short table: name, email, timezone, work days per week, focus %, minutes per man-day (global, set by the operator: read-only here), other users.
3. Ask whether anything is wrong. Typical fixes: timezone (IANA name, e.g. `Asia/Kuala_Lumpur`), `work_days_per_week` (0–5, steps of 0.5), `focus_percent` (1–100). A wrong man-day length is an operator fix, not a setup change.
4. On OK, call `settings_save` with `skill: "setup"` and my `version` from `me`. Echo what changed.
5. Member changes, when asked (not project guests or project members: those are the triage skill). Use `user` with `skill: "setup"`; `member` is the exact name, email or id. Show one line and save on OK:
   - `invite` (email + name): `Invite Sam <sam@example.com> to trace`. Inviting someone who hasn't signed in yet resends the email. If `invite_email` is `failed`, say so; they can still sign in from the sign-in page.
   - `cancel`: only for someone who never signed in. `USER_ACTIVE` → offer deactivate instead.
   - `deactivate`: say they're signed out, their tokens are revoked and they can't sign in; their records and project access stay. Not yourself.
   - `reactivate`: they can sign in again; old tokens stay revoked, so they make a new one.
     New members see no project until someone adds them (triage skill, `member add`).
6. Suggest permission rules (the plugin can't set them). Read-only tools, dictated status updates and dictated triage may run without asking; everything else should ask. Offer to add this to the user's `~/.claude/settings.json` under `permissions.allow`, and only edit it on OK:

```json
[
  "mcp__plugin_trace_trace__me",
  "mcp__plugin_trace_trace__overview",
  "mcp__plugin_trace_trace__projects_search",
  "mcp__plugin_trace_trace__project_get",
  "mcp__plugin_trace_trace__items_search",
  "mcp__plugin_trace_trace__item_get",
  "mcp__plugin_trace_trace__documents_search",
  "mcp__plugin_trace_trace__document_get",
  "mcp__plugin_trace_trace__history",
  "mcp__plugin_trace_trace__trash",
  "mcp__plugin_trace_trace__capacity",
  "mcp__plugin_trace_trace__risks",
  "mcp__plugin_trace_trace__report_data",
  "mcp__plugin_trace_trace__client_issues",
  "mcp__plugin_trace_trace__item_action",
  "mcp__plugin_trace_trace__client_issue_triage"
]
```

Leave `plan_apply`, `record_action`, `report`, `client_issue_save`, `guest`, `member`, `user` and the other write tools on "ask".
