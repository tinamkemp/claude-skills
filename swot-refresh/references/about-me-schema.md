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
   across applications (career transitions, degree-vs-role mismatches, "why this specialization," etc.).
   Write these as ready-to-use paragraphs, not bullet fragments — the goal is that `/swot` can lift them
   directly instead of re-deriving the framing through another round of clarifying questions.

6. **Personal constraints** — location, relocation/travel flexibility, and any other standing constraint
   relevant to fit assessment (compensation floor, start-date constraints, etc., if the user shares them).

7. **Known data inconsistencies** — an explicit, dated list of any conflicts found between source
   documents (e.g. two different figures for the same achievement). When a refresh resolves one, don't
   delete it — strike it through and note the resolution and date, so the audit trail survives.

8. **Canonical source documents** — every document `/swot-refresh` has pulled from, with enough detail to
   re-find it (Drive file name/ID, website URL/repo, LinkedIn export method) and a `Last verified` date
   per document, so a future refresh knows exactly what's stale.

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
