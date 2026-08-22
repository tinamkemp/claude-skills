---
name: swot
description: This skill should be used when the user asks to "generate a SWOT", "/swot", "run a SWOT for this opportunity", "analyze this posting for fit", "do a SWOT analysis for <company>", "build a candidate readiness brief", or wants a Strengths/Weaknesses/Opportunities/Threats assessment of their fit for a specific opportunity (a job posting, a board seat, a client contract) against their career background. Requires an existing about-me career profile — this skill never gathers or refreshes career data itself; use the separate swot-refresh skill for that.
argument-hint: <opportunity-posting-or-description-url-or-text>
---

Generate a SWOT (Strengths, Weaknesses, Opportunities, Threats) candidate-readiness brief for a specific
opportunity, reusing a persisted career profile so the user doesn't have to re-gather their resume,
LinkedIn, and website every time they evaluate something new.

## Configuration

```
CAREER_DRIVE_FOLDER_ID    = 15frOJ6UugnUgKtZquxw1FjpXiD9Ly1n0   # "Career" folder in Google Drive
GENERAL_ABOUT_ME_FOLDER_ID = 1yfNxMx2gPcwVD6SWarfa5w9v3B2VH8Xq   # Career/General/about-me
OPPORTUNITIES_FOLDER_ID   = 1AbFjNEaqk1ktKQOmXfd_WK1Ni4HwmTgk   # Career/Opportunities
TEMPLATE                  = ~/.claude/skills/swot/assets/swot-template.html
```
All data lives in Google Drive, not a local repo — this works the same whether run from Claude Code or
claude.ai chat. If any of these folders is ever recreated or moved, these are the only lines that need to
change. Every write in this skill is a fresh `create_file` call (a new, dated file) — the Drive tools used
here have no in-place content-edit capability, so nothing is ever overwritten.

## Workflow

1. **Load the profile.** In `${GENERAL_ABOUT_ME_FOLDER_ID}`, `search_files` for files titled
   `about-me-*.md` and take the one with the highest (most recent) date in its filename — that's the
   current profile. `read_file_content` it. If the folder is empty or doesn't exist, stop and tell the
   user to run `/swot-refresh` first — do not attempt to gather career data yourself in this skill.

2. **Freshness check.** Compare the profile's `Last refreshed` date to today. If it's more than 90 days
   old, mention this once, briefly, and suggest `/swot-refresh` — but proceed with the existing profile
   regardless. Never auto-refresh; that's an explicit, separate action the user takes on request.

3. **Resolve the opportunity.** `$ARGUMENTS` is either a posting/description URL or pasted text (a job
   posting, a board seat description, a client contract brief).
   - If it's a URL, `WebFetch` it and extract: organization, role title/level, location, required
     qualifications, responsibilities, and any named application/interview process details.
   - If it's pasted text, parse the same fields directly from it.
   - If neither a URL nor recognizable posting content was given, ask the user for it.
   - Either way, keep the raw text/URL verbatim — it gets saved into `intake/` in step 9.
   - Slug it as `<company-slug>-<role-slug>-<yyyymmdd>` (lowercase, hyphenated), then `search_files`
     under `${OPPORTUNITIES_FOLDER_ID}` for an existing folder with this exact slug.
     - **Found → this run is a rebuild.** Read the highest-dated `brief/swot-*.md` and *every* file in
       `log/` (per-opportunity volume is small enough that no date-filtering is needed here, unlike the
       cross-opportunity scan in `swot-refresh`). Carry this context into steps 4, 5, and 7 below.
     - **Not found → this run is new.** Proceed through steps 4–8 as a cold start; folder creation
       happens in step 9 as before.

4. **Research the company.** On a new opportunity, run `WebSearch` on this specific company's interview
   process, hiring culture, and any relevant recent news, same as always. On a **rebuild**, don't blindly
   redo research already reflected in the prior brief — only refresh what could plausibly have changed
   (recent news, anything the `log/` entries suggest shifted, e.g. a hiring-manager change). State
   explicitly in the new brief what's carried forward from the prior research vs. re-verified this run.

5. **Ask only what the profile and the log can't answer.** Use `AskUserQuestion` sparingly, and only for
   opportunity-specific unknowns: recruiter or interviewer feedback already received on *this*
   opportunity, current pipeline stage, application-specific framing decisions, or anything the posting
   raises that the profile doesn't cover. Do **not** re-ask about career narrative, known skill gaps, or
   personal constraints (location, etc.) — those live in `about-me.md` and should be reused directly. On a
   **rebuild**, also do not re-ask anything already answered by a `log/` entry pulled in step 3 — only ask
   about genuinely new unknowns since that entry's date.

6. **Check for open data conflicts.** Before citing any quantified figure (deal size, dates, team size),
   check the "Known data inconsistencies" section of `about-me.md`. If something relevant is still
   unresolved there, flag it in Weaknesses exactly as the profile does — don't silently pick a number.

7. **Synthesize the SWOT.**
   - **Strengths / Weaknesses** — internal, evidence-backed. Draw from `about-me.md`'s career timeline,
     achievements, skills/gaps, and narrative points, cross-referenced against what *this specific*
     posting asks for. A strength or weakness should be traceable to a specific source, not asserted.
   - **Opportunities / Threats** — external, situational. Draw from this run's fresh company/role research
     plus anything opportunity-specific the user shared in step 5.
   - Every SWOT item needs a source citation (see `references/design-system.md` for the exact markup).
   - Include a short, concrete action list tied to the nearest real milestone (an interview date, an
     application deadline) if one is known.
   - **On a rebuild**, don't synthesize from scratch — reconcile against the prior brief pulled in step 3:
     carry forward SWOT items that are still valid, update any the `log/` entries show have changed (a
     weakness resolved by logged feedback, a threat shifted by a status change), and add a short "what's
     changed since the last brief" note near the top of the position summary so this reads as an update,
     not an unrelated fresh take.

8. **Build the artifact.** Read `${TEMPLATE}` and `references/design-system.md`. Fill in the template's
   placeholders with this run's content. Do not redesign, restructure, or reskin it — the visual identity
   is intentionally consistent across every opportunity. Publish with the Artifact tool. If the current
   surface doesn't support Artifacts, write the markdown output only (step 9) and tell the user why no
   artifact link is included.

9. **Save and track.**
   - The opportunity folder was already resolved in step 3. On a new opportunity, `create_file` it now
     (the slug from step 3), plus its four subfolders: `intake`, `brief`, `materials`, `log`. On a
     rebuild, it already exists — just write into it.
   - `create_file` the step-3 posting capture (raw text/URL + parsed fields) into `intake/` as
     `posting-<yyyymmdd>.md`.
   - `create_file` the SWOT markdown into `brief/` as `swot-<yyyymmdd>.md` — same content as the artifact
     (position summary, four SWOT sections, action list, sources), plus a line near the top:
     `**Published brief:** <artifact URL>`.
   - If this run also produced a tailored resume or cover letter, `create_file` each into `materials/`.
   - Nothing to commit and no issue to open/close — the Drive files created above are the durable record.
     Anything that happens after this initial brief (feedback, actions taken, status changes) belongs in
     that opportunity's `log/` subfolder via the separate `/opportunity-log` skill, not here.

10. **Report back** to the user with the artifact link and the Drive opportunity folder (or the
    `brief/swot-<yyyymmdd>.md` file link). Keep the summary tight — the artifact itself is the deliverable,
    not a restated wall of text in chat.
