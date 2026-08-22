---
name: swot
description: This skill should be used when the user asks to "generate a SWOT", "/swot", "run a SWOT for this job", "analyze this posting for fit", "do a SWOT analysis for <company>", "build a candidate readiness brief", or wants a Strengths/Weaknesses/Opportunities/Threats assessment of their fit for a specific job opportunity against their career background. Requires an existing about-me career profile — this skill never gathers or refreshes career data itself; use the separate swot-refresh skill for that.
argument-hint: <job-posting-url-or-text>
---

Generate a SWOT (Strengths, Weaknesses, Opportunities, Threats) candidate-readiness brief for a specific
job opportunity, reusing a persisted career profile so the user doesn't have to re-gather their resume,
LinkedIn, and website every time they apply somewhere new.

## Configuration

```
CAREER_REPO_PATH = /Users/tina/dev_projects/professional_swot
ABOUT_ME         = ${CAREER_REPO_PATH}/about-me.md
SWOT_DIR         = ${CAREER_REPO_PATH}/swot/
TEMPLATE         = ~/.claude/skills/swot/assets/swot-template.html
```
If `${CAREER_REPO_PATH}` is ever renamed or moved, this is the only line that needs to change.

## Workflow

1. **Load the profile.** Read `${ABOUT_ME}`. If it doesn't exist, stop and tell the user to run
   `/swot-refresh` first — do not attempt to gather career data yourself in this skill.

2. **Freshness check.** Compare the profile's `Last refreshed` date to today. If it's more than 90 days
   old, mention this once, briefly, and suggest `/swot-refresh` — but proceed with the existing profile
   regardless. Never auto-refresh; that's an explicit, separate action the user takes on request.

3. **Resolve the opportunity.** `$ARGUMENTS` is either a job posting URL or pasted job posting text.
   - If it's a URL, `WebFetch` it and extract: company, role title/level, location, required
     qualifications, responsibilities, and any named application/interview process details.
   - If it's pasted text, parse the same fields directly from it.
   - If neither a URL nor recognizable job posting content was given, ask the user for it.

4. **Research the company.** Run `WebSearch` on this specific company's interview process, hiring
   culture, and any relevant recent news. This is always done fresh — it's the one part of the process
   that genuinely differs company to company and can't live in the profile.

5. **Ask only what the profile can't answer.** Use `AskUserQuestion` sparingly, and only for
   opportunity-specific unknowns: recruiter or interviewer feedback already received on *this*
   opportunity, current pipeline stage, application-specific framing decisions, or anything the posting
   raises that the profile doesn't cover. Do **not** re-ask about career narrative, known skill gaps, or
   personal constraints (location, etc.) — those live in `about-me.md` and should be reused directly.

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

8. **Build the artifact.** Read `${TEMPLATE}` and `references/design-system.md`. Fill in the template's
   placeholders with this run's content. Do not redesign, restructure, or reskin it — the visual identity
   is intentionally consistent across every opportunity. Publish with the Artifact tool. If the current
   surface doesn't support Artifacts, write the markdown output only (step 9) and tell the user why no
   artifact link is included.

9. **Save and track.**
   - Slug the opportunity as `<company-slug>-<role-slug>-<yyyymmdd>` (lowercase, hyphenated).
   - Write `${SWOT_DIR}<slug>.md` — same content as the artifact, in markdown, structured like existing
     files in that directory (position summary, four SWOT sections, action list, sources), plus a line
     near the top: `**Published brief:** <artifact URL>`.
   - In `${CAREER_REPO_PATH}`: `git add`, then `git commit -m "Add SWOT for <Company> <Role>"`, then
     `git push`.
   - `gh issue create --repo <owner>/<repo> --title "SWOT: <Company> <Role> (<date>)" --body "..."` (read
     the remote URL from `git remote -v` in `${CAREER_REPO_PATH}` to get `<owner>/<repo>` — don't
     hardcode it, the repo may be renamed again later), then immediately `gh issue close` it with a
     comment summarizing the result and linking the file + artifact. One issue per run — this workflow is
     lighter-weight than a first-time full gather, so it doesn't need a multi-issue breakdown.

10. **Report back** to the user with the artifact link and the repo file path. Keep the summary tight —
    the artifact itself is the deliverable, not a restated wall of text in chat.
