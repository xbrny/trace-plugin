# trace plugin

Claude Code plugin for trace's project-management workflow (`docs/03-spec/10-plugin-pm.md`). Skills hold the workflow; trace's `/mcp` endpoint stores the records.

## Install

1. In trace, open **Account** → create an API token (`trc_…`). It's shown once.
2. In Claude Code:
   ```
   /plugin marketplace add xbrny/trace-plugin
   /plugin install trace@trace
   ```
3. Claude Code asks for `trace_url` (e.g. `http://localhost:3000` for local dev) and `trace_token`. The token goes to the OS keychain. Change either later with `/plugin` → trace → Configure.
4. Run `/trace:setup`: checks the connection, shows your timezone and availability, and offers the permission rules below.

## Skills

| Command                                       | For                                                                  |
| --------------------------------------------- | -------------------------------------------------------------------- |
| `/trace:capture <text>`                       | New item, project or idea from a loose sentence                      |
| `/trace:note <text or file>`                  | Note, idea, decision, or a file upload                               |
| `/trace:plan [project or text]`               | Break work into initiatives and items, check capacity, write on `ok` |
| `/trace:track <text>`                         | Dictated progress: done, started, blocked, dates, estimates          |
| `/trace:capacity [question]`                  | Free time and "does this fit?"                                       |
| `/trace:risks [project]`                      | Risk review                                                          |
| `/trace:report [project] [period] [audience]` | Status report or email: draft, approve, mark sent                    |
| `/trace:triage [project or text]`             | Triage client issues; log issues and invite guests for a client      |
| `/trace:setup`                                | Connection check and settings                                        |

They also trigger from plain chat ("CPR-12 is done").

## Permissions

Suggested `permissions.allow` in `~/.claude/settings.json`: the read tools plus `item_action` and `client_issue_triage` (dictated status changes and triage save directly). Everything else keeps asking, incl. `client_issue_save` and `guest`.

```json
"mcp__plugin_trace_trace__me", "mcp__plugin_trace_trace__overview",
"mcp__plugin_trace_trace__projects_search", "mcp__plugin_trace_trace__project_get",
"mcp__plugin_trace_trace__items_search", "mcp__plugin_trace_trace__item_get",
"mcp__plugin_trace_trace__documents_search", "mcp__plugin_trace_trace__document_get",
"mcp__plugin_trace_trace__history", "mcp__plugin_trace_trace__trash",
"mcp__plugin_trace_trace__capacity", "mcp__plugin_trace_trace__risks",
"mcp__plugin_trace_trace__report_data", "mcp__plugin_trace_trace__client_issues",
"mcp__plugin_trace_trace__item_action", "mcp__plugin_trace_trace__client_issue_triage"
```

## Develop

- Validate: `claude plugin validate ./plugin` and `claude plugin validate .` (marketplace).
- Try local edits without reinstalling: `claude --plugin-dir ./plugin`.
- Tools live in `apps/api/src/mcp/`; this folder holds no code and imports nothing from `apps/`.
