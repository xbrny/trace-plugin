---
name: capacity
description: Answer workload and availability questions from trace — free time per week, who has room, whether new work fits by a date. Use for "can I take on a 3-day job by Friday?", "how busy am I next week?", "who has capacity?", "when could I finish X?".
argument-hint: [question]
---

# Capacity

Read only. Input: $ARGUMENTS.

- **"Does this fit?"** (estimate + due date): `capacity` with `check` (`assignee`, `estimate_minutes`, `due_date`, optional `start_date`). Answer with fits / short by N md and the earliest finish date. New work is placed after existing work.
- **"How busy am I / is X?"**: `capacity` with `user` (`me` or a name), `bucket` week (or day for this week), `from`/`to` if the user gave a range. Summarise free man-days per week; mention unscheduled and unestimated work, which isn't in the weekly numbers.
- **"Who has room?"**: `capacity` `user: "all"` with a `project` — that project's members, each with their load from every project. No project named: ask which team (project) once, or check named people one by one.

Convert minutes to man-days with `minutes_per_man_day` from `me` (call once). Round to 0.5 md. Drafts show as tentative load: say so when they change the answer.

Keep the answer to a few lines or a small table. Don't write anything.
