---
name: close-project
description: This skill should be used when the user wants to close out or archive a tracked project — via phrases like "close project X", "/close-project", "archive X", "I'm done with X", or "mark X complete/abandoned". Requires an existing project created by a prior /new-project run. Archives the Obsidian folder, moves the local support folder, files a debrief note, closes the Todoist project via API, updates the master list, and reminds the user to manually archive the claude.ai Project.
argument-hint: <project name or id>
---

Close out a project tracked in the virtual PMO: archive the Obsidian folder, file a debrief note,
close the Todoist project, move the local support folder, and update the master list. Usable from
Claude Code only, same as `/new-project`.

## Configuration

```
VPMO_REPO         = ~/dev_projects/virtual-pmo
CLOSE_PROJECT_CMD = python3 ~/dev_projects/virtual-pmo/scripts/close_project.py
```

## Workflow

1. **Identify the project.** If `$ARGUMENTS` names it, use that. Otherwise ask which project.
   Never guess between multiple plausible matches.

2. **Resolve it.** Run `${CLOSE_PROJECT_CMD} --dry-run "<query>" --status closed_complete` first —
   it looks up the query against the master list by id or name substring:
   - **No match** — tell the user, and suggest `/new-project` if this was actually never tracked.
   - **Multiple matches** — the script lists them; ask the user which one and re-run with the
     more specific id.
   - **One match** — the dry run also prints the planned Obsidian/local/Todoist archive paths and
     confirms the Todoist/1Password connection works. Relay any failure here directly (it names
     the fix) before going further.

3. **Ask the closure details with `AskUserQuestion`:**
   - **End date** — default today.
   - **Final status** — `closed_complete` or `closed_abandoned`.
   - **Debrief** — a short free-text account: what happened, what got done, what didn't, any
     lessons. This becomes the debrief note filed in the archived Obsidian folder.

4. **Run the orchestrator for real:**
   ```
   ${CLOSE_PROJECT_CMD} "<id>" --status <closed_complete|closed_abandoned> \
     --end-date <yyyy-mm-dd> --debrief "<debrief text>"
   ```

5. **Deliver the manual claude.ai reminder — every time, without exception.** Tell the user to
   archive or delete the claude.ai Project for this work by hand; there's no API for that step.

6. **Report back** with the Obsidian archive path and the local archive path the script printed.
