# Jekyll Port — Design Spec

**Date:** 2026-06-06
**Status:** Approved by James (design walkthrough, this session)
**Branch:** `jekyll-port` (off `master`)

## Decision Log

These were settled explicitly during brainstorming and are not open questions:

1. **Jekyll, not Hugo.** James chose to rebuild in Jekyll despite the completed
   Hugo migration on `hugo-migration`. That branch becomes a content source,
   then gets deleted after ship. CLAUDE.md's Hugo stack section is outdated
   and will be rewritten (see §6).
2. **Native GitHub Pages build.** Push markdown, GitHub builds. Only
   whitelisted plugins (we use `jekyll-seo-tag` only). No CI workflow, no
   committed build output, no submodules, no local toolchain required to edit.
3. **Bespoke minimal theme.** ~8 layout/include files plus one stylesheet.
   No theme dependency (minima, academicpages, and al-folio all rejected:
   al-folio needs non-whitelisted plugins; academicpages is the 29k-line
   codebase the Hugo migration deleted; minima would be overridden entirely).
4. **Bio:** "Assistant Professor of Finance at NYU Stern Abu Dhabi" (the
   Hugo-branch version). The Google Site's "senior research scientist, on
   leave" bio is outdated.
5. **Design tiebreaker:** when in doubt, simple, minimalist, and clean.
   James delegated visual decisions.
6. **No URL redirects** from old academicpages paths (`/publications/` etc.).
   The Google Site was the canonical link target, not the old GitHub site.

## 1. Architecture

- Branch `jekyll-port` off `master`. First implementation commit removes all
  academicpages files. Merge to `master` ships the site.
- Deploy: GitHub Pages, branch `master`, root directory, no custom workflow.
- `Gemfile` with `github-pages` gem exists for local preview only.
- File tree (complete — anything not listed should not exist):

```
_config.yml
Gemfile
CLAUDE.md
404.html
index.md            # home: profile hero + selected work
research.md         # papers grouped: Working Papers / Publications / Other Writing
teaching.md         # courses list
fun.md              # ported from hugo-migration
_layouts/
  default.html      # skeleton: head, header, footer
  home.html         # profile hero + selected papers
  list.html         # grouped collection index
  single.html       # one paper page
_includes/
  head.html         # meta, seo-tag, css link
  header.html       # site name + nav
  footer.html       # minimal footer
_papers/            # 12 files, see §3
_courses/           # 2 files, see §3
assets/
  css/main.scss
  picture.jpg       # headshot, see §4
  favicon.svg       # minimal authored monogram (no binary asset exists in repo)
docs/superpowers/specs/   # this spec and successors
```

## 2. Content Model

### Papers collection (`_papers/`)

Front matter schema:

```yaml
title: ""
authors: []          # full list including James
date: YYYY-MM-DD
category: working | published | other
venue: ""            # journal citation for published; omit for working
links:               # ordered list of {label, url} pairs
  - label: Paper
    url: ""
summary: ""          # one-to-two sentence plain-language summary
selected: false      # homepage feature flag
```

Body = abstract (plus award notes where they exist). Content source:
`git show hugo-migration:content/papers/<slug>/index.md`.

### Paper inventory (complete, verified against Google Site)

Working papers (6): profit-puzzles, markups-markdowns,
measuring-markups-revenue, demand-elasticities,
measuring-markups-production, aggregate-market-power.

Publications (5): beginning-of-trend (Economics Bulletin 2024),
production-approach (Economics Letters 2022), resolving-tbtf (J. Financial
Services Research 2021), bank-complexity (Economic Policy Review 2014),
tbtf-risk (Economic Policy Review 2014).

Other writing (1): biden-labor-competition (ProMarket 2022).

`selected: true` on profit-puzzles and markups-markdowns (the two the Google
Site features).

### Courses collection (`_courses/`)

Rendered inline on `teaching.md` (`output: false` — no individual course
pages; each entry is a title, two-line description, and an external link).
Papers, by contrast, get individual pages (`output: true`) because the
abstract justifies a page.

- **AI in Finance** — live MBA course, NYU Stern Abu Dhabi. Links to
  `https://james-traina.github.io/ai-finance/`. Do not duplicate course
  content.
- **Competitive Strategy** — UChicago undergraduate, Spring 2021 and 2022,
  with GitHub materials link.
- TA history is intentionally dropped (lives in the CV PDF).

### Pages and nav

- Nav: Research, Teaching, Fun — three items.
- Homepage hero: name; bio paragraph (Assistant Professor version, ported
  from hugo-migration config `profileMode.subtitle`); headshot; social icons
  (CV → Google Drive PDF, email, Google Scholar, GitHub, X); buttons
  (Research, Teaching, Fun). Below hero: "Selected work" listing the two
  selected papers.
- Fun page: ported from `hugo-migration:content/fun/index.md`.
- 404.html: one line, link home.

## 3. Visual Design

- **From pmichaillat/PaperMod:** compact profile hero (no full-viewport
  min-height), light-only theme, system font stack, header nav, minimal
  footer.
- **From al-folio (idea only):** paper entry layout — title, author line with
  James bolded, venue + year, row of small link buttons opening in new tabs.
- **From academicpages (idea only):** structured publication front matter,
  collection-driven architecture.
- One stylesheet `assets/css/main.scss` (~300 lines). CSS custom properties
  use PaperMod's names (`--gap`, `--radius`, `--border`, `--primary`,
  `--secondary`, `--content`) so CLAUDE.md's design system reads unchanged.
- CLAUDE.md design-system rules apply verbatim: 0.15s ease transitions on all
  interactive elements; 10px/16px button padding; 4px/8px border-radius
  scale; `#2563eb` content links (hover `#1d4ed8`); letter-spacing 0.05em on
  uppercase text.
- **Zero JavaScript.** No webfonts. No dark mode.

## 4. Profile Photo (content acquisition)

No usable headshot exists in the repo: `hugo-migration` references
`picture.jpeg` but contains no image files (pre-existing broken image), and
`master`'s images are academicpages placeholders. Acquire the headshot from
the Google Site homepage (googleusercontent image URL) and save as
`assets/picture.jpg`. Fallback if extraction fails: ask James for the file.

## 5. Verification

1. `bundle exec jekyll build` exits 0, no warnings, output contains no
   `<script` tags.
2. `jekyll serve` + Playwright screenshots of every page at 1280px and 390px,
   reviewed against the design rules above.
3. curl pass over every external link (paper PDFs, SSRN, DOI, course site,
   social URLs): no 404s.
4. After merge and push: GitHub Pages build green, live site spot-checked
   (homepage, research, one paper page).

## 6. CLAUDE.md Update

Rewrite Technical Stack, Config Decisions, and Deployment sections for
Jekyll + native Pages (push-to-build, no `public/`, no Hugo). Update the
"What Not To Do" Hugo references. Voice rules, design system, and site
structure sections survive unchanged. The nav spec line changes from
"Research / Teaching / About" to "Research / Teaching / Fun" to match what
was actually built and approved.

## Out of Scope (YAGNI)

Blog, dark mode, JavaScript, search, tags/taxonomies, BibTeX rendering,
talk maps, RSS beyond Pages defaults, redirects from old URLs, analytics.
