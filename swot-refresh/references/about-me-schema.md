# about-me.md schema

The durable career profile has nine sections, in this order. Preserve this structure on every refresh —
add/update content within sections rather than restructuring the file.

1. **Header** — name, location, target function (kept general — e.g. "Applied AI / Solutions Architecture
   / presales technical leadership" — not scoped to whatever single company prompted the last refresh),
   `Last refreshed` date.

2. **Career timeline & quantified achievements** — reverse-chronological role list (company, title,
   dates). Every bullet cites its source document (e.g. `Resume 2024`, `LinkedIn`). Carry forward every
   hard number already on file (deal values, deal counts, YoY growth, team size, etc.) — don't drop
   figures on refresh just because a newer document doesn't repeat them; instead flag any conflict (see
   section 7).

3. **Awards / external validation** — named awards, year, source. Third-party recognition only — not
   self-assessed accomplishments (those belong in section 2).

4. **Skills & certifications** — a demonstrated list, plus an explicit **"Not yet demonstrated / gaps"**
   subsection. This gaps list is load-bearing: every `/swot` run inherits it as-is unless a refresh
   explicitly updates it to say a gap has closed (with evidence — a course completed, a project shipped,
   not just an intention).

5. **Recurring narrative / talking points** — reusable prose for framing questions that tend to recur
   across opportunities (career transitions, degree-vs-role mismatches, "why this specialization," etc.).
   Write these as ready-to-use paragraphs, not bullet fragments — the goal is that `/swot` can lift them
   directly instead of re-deriving the framing through another round of clarifying questions.

6. **Personal constraints** — location, relocation/travel flexibility, and any other standing constraint
   relevant to fit assessment (compensation floor, start-date constraints, etc., if the user shares them).
   Anything that's still being worked out on a per-opportunity basis (e.g. relocation flexibility not yet
   settled into a standard answer) should point at that opportunity's `log/` entries (via
   `/opportunity-log`), not at its SWOT brief — the brief is a frozen point-in-time snapshot, while the log
   is where an evolving answer actually gets gathered over time. Update this section here only once a
   consistent position solidifies across opportunities.

7. **Known data inconsistencies** — an explicit, dated list of any conflicts found between source
   documents (e.g. two different figures for the same achievement). When a refresh resolves one, don't
   delete it — strike it through and note the resolution and date, so the audit trail survives.

8. **Canonical source documents** — every document `/swot-refresh` has pulled from, with enough detail to
   re-find it (Drive file name/ID, website URL/repo, LinkedIn export method) and a `Last verified` date
   per document, so a future refresh knows exactly what's stale. A flagged `opportunity-log` entry
   (`about_me_gap_flagged: true`, surfaced mid-opportunity-prep and swept up on the next refresh) is also
   a recognized source type — cite it inline where the fact lands in section 2/4/etc. (opportunity slug +
   entry date) rather than listing it here, since it's a one-time input, not an ongoing document to
   re-check for staleness.

9. **Change log** — one dated one-line entry per refresh, human-readable, on top of whatever git history
   already shows.

## Refresh discipline

- Never silently overwrite a fact — if a newly gathered document contradicts what's already in the file,
  add it to section 7 rather than picking one silently, unless the user has explicitly confirmed which is
  correct in this session.
- Always update the `Last refreshed` header date and append a section 9 entry, even for a small or partial
  refresh.
- Gaps in section 4 should only be marked closed with the user's explicit confirmation and a concrete
  reason — "started using X" is progress worth noting in section 5, not the same as "gap closed."
- A flagged `opportunity-log` entry is a candidate, not an automatic edit — it goes through the same
  diff-and-confirm step as any other newly gathered material before it lands anywhere in this file.
