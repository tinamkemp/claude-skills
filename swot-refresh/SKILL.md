---
name: swot-refresh
description: This skill should be used when the user explicitly asks to "refresh my profile", "update my about-me", "resync my career profile", "refresh my career profile from Drive/LinkedIn", or otherwise wants to regenerate the durable about-me career profile from source documents. Only runs when the user explicitly requests a refresh — the separate swot skill never triggers this automatically, and reads the profile this skill produces without modifying it.
---

Re-gather career evidence and update the durable `about-me.md` profile that the `swot` skill reads from.
This is always an explicit, user-initiated action — never run it automatically, and never run it just
because a `/swot` run noticed the profile is old.

## Configuration

```
CAREER_DRIVE_FOLDER_ID    = 15frOJ6UugnUgKtZquxw1FjpXiD9Ly1n0   # "Career" folder in Google Drive
ABOUT_ME_FOLDER_ID        = 1yfNxMx2gPcwVD6SWarfa5w9v3B2VH8Xq   # Career/General/about-me
SOURCE_DOCS_FOLDER_ID     = 1vE75R7GWYW7mjhyaIzcsBxGp4bq41ObI   # Career/General/source-documents
OPPORTUNITIES_FOLDER_ID   = 1AbFjNEaqk1ktKQOmXfd_WK1Ni4HwmTgk   # Career/Opportunities
```
All data lives in Google Drive, not a local repo — this works the same whether run from Claude Code or
claude.ai chat. If any of these folders is ever recreated or moved, these are the only lines that need to
change in this skill (the `swot` skill has its own copy of these constants — they don't share state).
There is no in-place content-edit capability on these Drive files, so every refresh writes a fresh, dated
`about-me-<yyyymmdd>.md` rather than overwriting the previous one — the prior file is left in place, which
doubles as free version history.

## Workflow

1. **Scope the refresh.** If the user's request is ambiguous about scope (e.g. "update my profile" with
   no detail), ask briefly whether this is a full refresh or something specific (a new role, a closed
   skill gap, a corrected figure). Otherwise proceed with a full refresh.

2. **Re-gather source material.**
   - Load the current profile first: `search_files` in `${ABOUT_ME_FOLDER_ID}` for `about-me-*.md`, take
     the highest-dated one, `read_file_content` it. Note its `Last refreshed` date — the next sub-step
     needs it.
   - Search `${SOURCE_DOCS_FOLDER_ID}` for the resume, cover letters, and any other career documents —
     look for anything newer than the "Last verified" dates already in the current profile's "Canonical
     source documents" section. Only widen to an unscoped Drive search (via the
     `mcp__claude_ai_Google_Drive__*` tools) if nothing relevant turns up there, or the user mentions a
     document that isn't in that folder yet.
   - **Scan for flagged opportunity-log entries** (this is how a gap surfaced mid-opportunity-prep — e.g.
     recruiter feedback naming a skill, or an unrelated fact like prior volunteer IT work that turns out to
     be relevant — makes it into the durable profile without the user having to remember and re-type it
     later): `search_files(parentId = OPPORTUNITIES_FOLDER_ID)` to enumerate every opportunity folder; for
     each, resolve its `log/` subfolder id and `search_files` inside it with
     `createdTime > <the current profile's Last refreshed date>`. This date filter is the only "already
     consumed" tracking needed — a flagged entry stops turning up once a refresh happens after its date, no
     separate state to maintain. `read_file_content` each match and check its YAML frontmatter for
     `about_me_gap_flagged: true`; collect the ones that are. Run by default, same as the document scan
     above — not something the user has to ask for separately.
   - Fetch the personal website's current content (its URL and/or source repo are listed in the current
     profile).
   - LinkedIn cannot be fetched automatically — ask the user to paste an updated LinkedIn export (or the
     specific sections that changed), or to upload the export file directly into
     `${SOURCE_DOCS_FOLDER_ID}`. This is a standing manual step on every refresh, not a one-time gap.

3. **Diff before overwriting.** Compare everything gathered in step 2 — new/updated documents, flagged
   log entries, website, LinkedIn — against the current profile already loaded above. Before changing
   anything, summarize for the user: what's new, what changed, what appears resolved (e.g. a flagged data
   inconsistency that a new document clarifies), and what's simply unchanged. For each flagged log entry,
   name which opportunity and date it came from and what it says, same as any other candidate source. Get
   confirmation before overwriting any existing fact or incorporating a flagged item — don't silently
   replace a number or claim, and don't silently fold in a gap just because it was flagged.

4. **Rewrite the profile.** Compose the full new profile in memory, following the schema in
   `references/about-me-schema.md` exactly — same nine sections, same order — carrying forward every
   section unchanged except what this refresh updates (this is a full-content copy each time, not a patch,
   since there's no partial-write capability). When incorporating a fact sourced from a flagged log entry,
   cite it the same way other bullets cite `(Resume 2024)` — e.g.
   `(logged via anthropic-applied-ai-architect-20260822, 2026-08-25)` — so the audit trail survives. Update
   the header's `Last refreshed` date to today and append a dated line to the Change log describing what
   changed this time. `create_file` the result into `${ABOUT_ME_FOLDER_ID}` as `about-me-<yyyymmdd>.md`
   (today's date) — do not touch or trash the previous file.

5. **Report back** with a short summary of what changed — new roles/achievements added, gaps closed (with
   the evidence for closing them), conflicts resolved, anything still open — plus the new file's Drive
   link.
