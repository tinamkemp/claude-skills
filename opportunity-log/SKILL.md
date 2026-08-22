---
name: opportunity-log
description: This skill should be used when the user wants to record how a specific opportunity is progressing after the initial SWOT — recruiter or interviewer feedback, an action taken (and its outcome), or a status change — via phrases like "log this update", "/opportunity-log", "record this feedback", "note that I did X for this opportunity", or "update the log for <company>". Requires an existing opportunity folder created by a prior `/swot` run — this skill never creates a new opportunity from scratch. Never writes to about-me.md directly; it only flags candidate gaps and tells the user to run `/swot-refresh`.
argument-hint: <company-or-role-or-slug> [what happened]
---

Capture an ongoing update for a specific opportunity — the middle layer between the frozen point-in-time
SWOT brief and the durable, cross-opportunity `about-me.md` profile. Usable from either Claude Code or
claude.ai chat.

## Configuration

```
CAREER_DRIVE_FOLDER_ID    = 15frOJ6UugnUgKtZquxw1FjpXiD9Ly1n0   # "Career" folder in Google Drive
OPPORTUNITIES_FOLDER_ID   = 1AbFjNEaqk1ktKQOmXfd_WK1Ni4HwmTgk   # Career/Opportunities
```
Every write is a fresh `create_file` call — one immutable, dated file per log entry. There is no
in-place content-edit capability on these Drive files, so this skill never rewrites an existing entry.

## Workflow

1. **Identify the opportunity.** If `$ARGUMENTS` names a company, role, or slug, use it. Otherwise ask
   the user which opportunity this update is about (`AskUserQuestion`). Never guess silently between
   multiple plausible matches.

2. **Resolve the opportunity folder.** `search_files` under `${OPPORTUNITIES_FOLDER_ID}`, matching on
   slug substring or company/role text in the folder title. If more than one folder matches (e.g. two
   rounds with the same company), list them and ask which one. If none matches, tell the user no
   opportunity folder was found and suggest running `/swot` first — this skill never creates an
   opportunity folder from scratch, only appends to an existing one.

3. **Pull context.** `search_files` the resolved folder's `brief/` subfolder for `swot-*.md`, take the
   highest-dated one, and `read_file_content` it so the update can be understood against the existing
   SWOT. Also `search_files` the `log/` subfolder to see existing entries and avoid duplicating an
   already-logged event. On claude.ai (not the CLI), if the brief's `**Published brief:**` link points to
   an Artifact, also check that Artifact for new comment threads since the last log entry's date, using
   the Artifact tool's own comment-reading capability — this is not something the Drive tools expose.
   Skip this sub-step silently if unavailable on the current surface.

4. **Ask what happened.** `AskUserQuestion` with three category choices — feedback received, an action
   taken, or a status change — plus a plain-language description of what happened. If step 3 surfaced
   unread Artifact comments, summarize them back to the user as candidate content rather than making them
   retype what's already visible in the thread.

5. **Assess for an about-me gap.** Ask whether this update surfaces a genuine, evidence-based gap not
   already captured in about-me.md's "Not yet demonstrated / gaps" section (e.g. an interviewer named a
   specific missing skill) — as opposed to routine scheduling/logistics with no skill-fit signal. If yes,
   set `about_me_gap_flagged: true` and write a "Flagged for about-me refresh" subsection naming the gap
   and its evidence, worded so it's ready to hand to `/swot-refresh` almost verbatim. **This skill never
   writes to about-me.md itself** — full stop. Tell the user plainly, once, to run `/swot-refresh`.

6. **Slug and write the entry.** Build a short slug from the category plus a couple of words of the
   summary (e.g. `20260825-recruiter-call.md`, `20260827-status-onsite-scheduled.md`). `create_file` it
   into that opportunity's `log/` subfolder with this structure:

   ```yaml
   ---
   date: 2026-08-25
   opportunity_slug: anthropic-applied-ai-architect-20260822
   category: feedback   # one of: feedback | action | status-change
   summary: "Recruiter call — hiring manager pushed timeline to next Monday"
   about_me_gap_flagged: false
   ---
   ```
   followed by a freeform markdown body describing what happened, and (if flagged) the "Flagged for
   about-me refresh" subsection.

7. **Report back.** Confirm the entry was logged, restate whether a gap was flagged (repeat the
   `/swot-refresh` nudge once, plainly, if so — don't bury it), and give the Drive file link.
