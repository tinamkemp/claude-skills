# SWOT artifact design system

`assets/swot-template.html` is the visual design for every `/swot` artifact. **Reuse it as-is** —
substitute the bracketed placeholders below with this run's content; never restructure the CSS, change
the color tokens, or redesign the layout. If a quadrant color genuinely needs adjusting for a specific
run (it shouldn't, normally), change only the hex *values* of the existing `--sw-*`/`--wk-*`/`--op-*`/
`--th-*` tokens, in both the light `:root` block and both dark blocks, keeping all three in sync.

## Concept

A "candidate readiness brief" — a polished, memo-style document, not a marketing page. Warm-neutral
"stone" paper ground, deep slate-green ink, one warm ochre/rust accent used sparingly. A classic 2×2 SWOT
matrix (Internal/External × Helpful/Harmful) executed with real typographic care: a serif display face
for headings (`.serif` class — system serif stack, no webfont), a system sans for body text, tabular
numerals where digits line up. Each quadrant gets a distinct, desaturated color wash (sage/clay/slate-blue/
dusty-rose) — categorical, not semantic-severity — so the four are visually distinct without any one
shouting over the others. Light and dark themes are both fully defined; do not add a third look.

## Placeholder reference

| Placeholder | Fill with |
|---|---|
| `{{PAGE_TITLE}}` | `<title>` text, e.g. `"<Company> Candidacy Brief"` — a name, not a category label |
| `{{H1}}` | `"<Candidate Name> — <Role Title>"` |
| `{{SUBTITLE}}` | One sentence: what this brief is prepared against (posting + career record + market research) |
| `{{STATUS_STRIP_HTML}}` | 2–4 `<span>` items, pattern below |
| `{{SUMMARY_HTML}}` | 2–3 sentence position summary; wrap the single most important clause in `<strong>` |
| `{{STRENGTHS_ITEMS_HTML}}` / `{{WEAKNESSES_ITEMS_HTML}}` / `{{OPPORTUNITIES_ITEMS_HTML}}` / `{{THREATS_ITEMS_HTML}}` | 4–6 `<li>` items each, pattern below |
| `{{ACTIONS_HEADING}}` | e.g. `"Before Monday"` or `"Next Steps"` — tie to the actual next milestone/date if known |
| `{{ACTIONS_ITEMS_HTML}}` | 3–5 `<li>` items, pattern below — no manual numbering, CSS counters handle it |
| `{{LOOP_STRIP_HTML}}` | Optional. Either the full `.loop-strip` block (pattern below) if this run's research surfaced a specific interview-loop structure worth citing, or an **empty string** to omit the section entirely |
| `{{SOURCES_HTML}}` | One paragraph, semicolon-separated list of every source used this run |

## Markup patterns

**Status strip item:**
```html
<span><strong>Label:</strong> value</span>
```
Use `<span class="mono">` around dates/figures that should line up (tabular numerals).

**SWOT quadrant item** (same pattern in all four lists):
```html
<li>
  <p>Claim, stated plainly, with the specific evidence inline (numbers, names, dates).</p>
  <span class="src">Source A · Source B</span>
</li>
```
Every item must cite at least one source tag. Never include an item without one — an uncited claim in
this document is a defect, not a stylistic choice.

**Action item:**
```html
<li>Imperative, concrete action — something that visibly gets done, not a vague intention.</li>
```

**Loop strip block** (omit entirely — set `{{LOOP_STRIP_HTML}}` to `""` — if not applicable):
```html
<div class="loop-strip">
  <h3>Reported interview loop, for reference</h3>
  <div class="loop-steps">
    <span>Stage one</span>
    <span>Stage two</span>
    <span>Stage three</span>
  </div>
</div>
```

## What never changes

- The CSS token block (all three theme states: light `:root`, `@media (prefers-color-scheme: dark)`
  guarded block, `:root[data-theme="dark"]` block) — copy verbatim.
- The overall page structure: header → summary → axis row → 2×2 grid (strengths/opportunities on top,
  weaknesses/threats on bottom, matching the template's existing div order) → actions panel → optional
  loop strip → sources footer.
- Font stacks — system fonts only, no external webfont links (the Artifact CSP blocks font CDNs).
