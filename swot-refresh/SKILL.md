---
name: swot-refresh
description: This skill should be used when the user explicitly asks to "refresh my profile", "update my about-me", "resync my career profile", "refresh my career profile from Drive/LinkedIn", or otherwise wants to regenerate the durable about-me career profile from source documents. Only runs when the user explicitly requests a refresh — the separate swot skill never triggers this automatically, and reads the profile this skill produces without modifying it.
---

Re-gather career evidence and update the durable `about-me.md` profile that the `swot` skill reads from.
This is always an explicit, user-initiated action — never run it automatically, and never run it just
because a `/swot` run noticed the profile is old.

## Configuration

```
CAREER_REPO_PATH = /Users/tina/dev_projects/professional_swot
ABOUT_ME         = ${CAREER_REPO_PATH}/about-me.md
```
If `${CAREER_REPO_PATH}` is ever renamed or moved, this is the only line that needs to change in this
skill (the `swot` skill has its own copy of this constant — they don't share state).

## Workflow

1. **Scope the refresh.** If the user's request is ambiguous about scope (e.g. "update my profile" with
   no detail), ask briefly whether this is a full refresh or something specific (a new role, a closed
   skill gap, a corrected figure). Otherwise proceed with a full refresh.

2. **Re-gather source material.**
   - Search Google Drive (via the `mcp__claude_ai_Google_Drive__*` tools) for the resume and any cover
     letters or career documents — look for anything newer than the "Last verified" dates already in
     `${ABOUT_ME}`'s "Canonical source documents" section, plus a general resume/CV search in case
     something new was added.
   - Fetch the personal website's current content (its URL and/or source repo are listed in `${ABOUT_ME}`).
   - LinkedIn cannot be fetched automatically — ask the user to paste an updated LinkedIn export (or the
     specific sections that changed). This is a standing manual step on every refresh, not a one-time gap.

3. **Diff before overwriting.** Compare newly gathered material against the current `${ABOUT_ME}`. Before
   changing anything, summarize for the user: what's new, what changed, what appears resolved (e.g. a
   flagged data inconsistency that a new document clarifies), and what's simply unchanged. Get
   confirmation before overwriting any existing fact — don't silently replace a number or claim.

4. **Rewrite the profile.** Update `${ABOUT_ME}` following the schema in `references/about-me-schema.md`
   exactly — same nine sections, same order. Update the header's `Last refreshed` date and append a dated
   line to the Change log describing what changed this time.

5. **Commit.** In `${CAREER_REPO_PATH}`: `git add about-me.md`, then
   `git commit -m "Refresh about-me profile — <date>"`, then `git push`.

6. **Report back** with a short summary of what changed — new roles/achievements added, gaps closed (with
   the evidence for closing them), conflicts resolved, anything still open.
