---
name: risks
description: Review delivery risks in trace — overdue items, missed or at-risk targets, long blocks, over-capacity weeks, estimate drift — for one project or all. Use for "what's at risk?", "anything slipping?", "risk review for CPR", "what should I worry about this week?".
argument-hint: [project]
---

# Risks

Read only, unless the user asks to save commentary. Input: $ARGUMENTS (a project, or all).

1. `risks` (`project` if given). Without a project it returns my share only — my items, projects and initiatives I own, my over capacity — so say "your risks" and offer a project for the full picture. For context on specific entries: `item_get`, `project_get` (initiative targets and baselines).
2. Group by type, most urgent first: overdue, target missed, blocked long, over capacity, target at risk, estimate drift. One line each with ref, title and the fact (days overdue, days blocked, shortfall, drift %).
3. Then your own read: what is likely to slip and why, and one concrete suggestion each (re-plan, unblock, cut scope). This commentary is a risk call — show it in chat only.
4. Save commentary only if the user says so: `document_save` `kind: "note"`, attached to the project / initiative / item, `skill: "risks"`, after OK.

No risks → say so in one line.
