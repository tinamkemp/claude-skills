---
name: new-project
description: This skill should be used when the user wants to start tracking a new project — a software side-project, career move, faith/seminary application, home/life-admin task, or any personal initiative — via phrases like "new project", "/new-project", "start tracking X", or "add X to my project list". Creates the master-list entry, a local support folder, an Obsidian folder + master note, and a Todoist project; also reminds the user to manually create a claude.ai Project for this work's chats (no API exists for that step).
argument-hint: <project name (optional, will be asked if omitted)>
---

Register a new project in the virtual PMO: one master-list record, fanned out to a local support
folder, an Obsidian folder + note, and a Todoist project. Usable from Claude Code only — it shells
out to a local Python engine and touches the local filesystem, so it does not work from claude.ai
chat.

## Configuration

```
VPMO_REPO       = ~/dev_projects/virtual-pmo
NEW_PROJECT_CMD = python3 ~/dev_projects/virtual-pmo/scripts/new_project.py
```
The actual logic (master-list schema, Obsidian/Todoist calls, 1Password token fetch) lives in
`${VPMO_REPO}` — see that repo's README for what each script does. This file only owns the
conversational collection of fields and the confirm-before-acting step.

## Workflow

1. **Get the name.** Use `$ARGUMENTS` if given. Otherwise ask.

2. **Collect the remaining fields with `AskUserQuestion`:**
   - **Priority** — high / medium / low.
   - **Life domain** — first run `python3 ${VPMO_REPO}/scripts/new_project.py --list-domains` to
     get her existing domain values and offer them as options (plus "something new"), so domains
     don't drift into near-duplicates over time.
   - **Acceptance criteria** — free text: what "done" looks like for this project.

   Start date defaults to today and is not asked unless the project actually started earlier.
   Status always starts `active`. End date is never asked here — it's only set at closure.

3. **Confirm before creating anything.** Echo back the name, priority, domain, and acceptance
   criteria in one short summary and get an explicit go-ahead. This step is about to create things
   in three separate external systems (filesystem, Obsidian vault, Todoist) — a typo here means
   cleanup in three places, so don't skip the confirmation.

4. **Run the orchestrator:**
   ```
   ${NEW_PROJECT_CMD} --name "<name>" --priority <priority> --domain "<domain>" \
     --acceptance-criteria "<acceptance criteria>" [--start-date <yyyy-mm-dd>]
   ```
   The script checks the Todoist/1Password connection *before* creating anything, so a broken
   token fails loudly with nothing left half-created. If it fails, relay its error message
   directly — it already names the specific fix (e.g. "check the Todoist token in 1Password").

5. **Deliver the manual claude.ai reminder — every time, without exception.** There's no API for
   claude.ai Projects, so this never gets automated: tell the user to create a claude.ai Project
   named for this work and use it for related chats. Don't silently skip this because it feels
   repetitive — it's the one step that has to stay manual.

6. **Report back** with the local folder path, the Obsidian note path, and the Todoist project
   link that the script printed on success.
