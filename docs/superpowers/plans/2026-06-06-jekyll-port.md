# Jekyll Port Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers-extended-cc:subagent-driven-development (recommended) or superpowers-extended-cc:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the academicpages Jekyll fork on `master` with a bespoke minimal Jekyll site (pmichaillat/PaperMod look) built natively by GitHub Pages.

**Architecture:** Zero-dependency theme: 3 layouts, 5 includes, 1 stylesheet, 2 collections (`_papers` with pages, `_courses` inline-only). Content ported from pinned commit `fd98409`. No plugins, no JavaScript, no build pipeline — push markdown and GitHub Pages builds.

**Tech Stack:** Jekyll (github-pages gem locally, native build remotely), kramdown, Sass (single file), Liquid.

**Spec:** `docs/superpowers/specs/2026-06-06-jekyll-port-design.md`

**Branch:** `jekyll-port` (already exists, contains spec + CLAUDE.md). Work happens here; ship = fast-forward merge to `master`.

**Native tasks:** Plan Task 1→#8, 2→#9, 3→#10, 4→#11, 5→#12, 6→#13.

**One documented spec deviation:** the spec's `_layouts/list.html` is replaced by `_includes/paper-entry.html` + `_includes/author-list.html`. Reason: the research page renders three groups of papers and the homepage renders selected papers — a layout can't be reused for that, an include can. Same file count, removes markup triplication (DRY).

---

## Context an Engineer Needs

- **Content source of truth:** commit `fd98409` (tip of retained `hugo-migration` branch). Read files with `git show fd98409:<path>`. Never re-scrape the Google Site for paper data — it's already captured, EXCEPT three link URLs that were `"#"` placeholders in fd98409; the correct URLs are embedded in Task 2 below (recovered from the live Google Site this session).
- **Voice rules (CLAUDE.md):** no ampersands in prose, Oxford commas, no emoji, sharp first-person voice. The author-list include implements Oxford commas mechanically.
- **Design rules (CLAUDE.md):** 0.15s ease transitions on everything interactive; buttons 10px/16px padding; radius scale 4px/8px only; content links `#2563eb` (hover `#1d4ed8`); uppercase text gets `letter-spacing: 0.05em`; hero compact, never full-viewport.
- **Hard invariant:** zero `<script` in generated output. We run zero plugins, so nothing can inject one.
- **GitHub Pages is already configured** for this repo (master branch, root, native Jekyll — that's how the old academicpages site serves). Shipping requires no settings change.

---

### Task 1: Clean slate + Jekyll skeleton (native task #8)

**Goal:** Remove all academicpages files and stand up an empty-but-building bespoke Jekyll skeleton.

**Files:**
- Delete: every tracked file except `CLAUDE.md`, `docs/`, `.gitignore`
- Create: `.gitignore` (replace), `Gemfile`, `_config.yml`, `_layouts/default.html`, `_layouts/home.html`, `_layouts/single.html`, `_includes/head.html`, `_includes/header.html`, `_includes/footer.html`, `_includes/paper-entry.html`, `_includes/author-list.html`, `404.html`

**Acceptance Criteria:**
- [ ] `bundle exec jekyll build` exits 0 with zero warnings
- [ ] Zero plugins; hand-written title + meta description in head.html
- [ ] No `<script` anywhere in `_site/` output — absolute
- [ ] `git status` clean after commit (Hugo leftovers ignored or removed)

**Verify:** `bundle exec jekyll build && grep -ri '<script' _site/ ; echo "exit=$?"` → build succeeds, grep finds nothing (exit=1)

**Steps:**

- [ ] **Step 1: Confirm branch and Ruby toolchain**

```bash
git branch --show-current   # must print: jekyll-port
ruby -v
```

If Ruby < 3.0 (macOS system Ruby is 2.6), install via Homebrew and use it for this session:

```bash
brew install ruby
export PATH="$(brew --prefix ruby)/bin:$PATH"
gem install bundler --no-document
```

- [ ] **Step 2: Delete academicpages files and Hugo leftovers**

```bash
# Tracked academicpages files (keeps CLAUDE.md, docs/, .gitignore via exclude pathspecs)
git rm -r --quiet -- . ':(exclude)CLAUDE.md' ':(exclude)docs' ':(exclude).gitignore'
# Untracked Hugo build leftovers (regenerable: public/ is hugo output, themes/ is a submodule clone)
rm -rf public themes .hugo_build.lock .gitmodules
```

Note: `.gitmodules` is tracked on master — the first `git rm` removes it from the index; the `rm -rf` line just cleans any working-tree remnant.

- [ ] **Step 3: Write `.gitignore`** (full replacement)

```
_site/
.jekyll-cache/
.jekyll-metadata
vendor/
.bundle/
Gemfile.lock
.DS_Store
.playwright-mcp/
.serena/
```

- [ ] **Step 4: Write `Gemfile`**

```ruby
source "https://rubygems.org"
gem "github-pages", group: :jekyll_plugins
```

- [ ] **Step 5: Write `_config.yml`**

```yaml
title: James Traina
description: "James Traina — Assistant Professor of Finance, NYU Stern Abu Dhabi. Research on markups, market power, and firm behavior."
url: "https://james-traina.github.io"

permalink: pretty
markdown: kramdown

collections:
  papers:
    output: true
    permalink: /papers/:name/
  courses:
    output: false

defaults:
  - scope:
      type: papers
    values:
      layout: single

exclude:
  - Gemfile
  - Gemfile.lock
  - vendor
  - docs
  - CLAUDE.md
```

- [ ] **Step 6: Write `_layouts/default.html`**

```html
<!DOCTYPE html>
<html lang="en">
{% include head.html %}
<body>
  {% include header.html %}
  <main class="main">
    {{ content }}
  </main>
  {% include footer.html %}
</body>
</html>
```

- [ ] **Step 7: Write `_includes/head.html`**

```html
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{% if page.title and page.url != "/" %}{{ page.title }} · {{ site.title }}{% else %}{{ site.title }}{% endif %}</title>
  <meta name="description" content="{{ page.summary | default: page.description | default: site.description }}">
  <link rel="icon" href="{{ '/assets/favicon.svg' | relative_url }}" type="image/svg+xml">
  <link rel="stylesheet" href="{{ '/assets/css/main.css' | relative_url }}">
</head>
```

- [ ] **Step 8: Write `_includes/header.html`**

```html
<header class="header">
  <div class="header-inner">
    <a class="site-name" href="{{ '/' | relative_url }}">{{ site.title }}</a>
    <nav class="nav">
      <a href="{{ '/research/' | relative_url }}">Research</a>
      <a href="{{ '/teaching/' | relative_url }}">Teaching</a>
      <a href="{{ '/fun/' | relative_url }}">Fun</a>
    </nav>
  </div>
</header>
```

- [ ] **Step 9: Write `_includes/footer.html`**

```html
<footer class="footer">
  <span>© {{ 'now' | date: "%Y" }} James Traina</span>
</footer>
```

- [ ] **Step 10: Write `_includes/author-list.html`**

Renders an author array with Oxford commas and James bolded. Two authors: "A and B". Three or more: "A, B, and C".

```html
{%- assign authors = include.authors -%}
{%- for a in authors -%}
  {%- if forloop.last and forloop.length > 1 %}and {% endif -%}
  {%- if a == "James Traina" -%}<strong>{{ a }}</strong>{%- else -%}{{ a }}{%- endif -%}
  {%- unless forloop.last -%}{%- if forloop.length == 2 %} {% else %}, {% endif -%}{%- endunless -%}
{%- endfor -%}
```

- [ ] **Step 11: Write `_includes/paper-entry.html`**

One paper entry used by the research page groups and the homepage selected list. Link buttons open in new tabs (PDF/external rule). `paper-links` is a `div`, not `p`, so prose-link styling never hits the buttons.

```html
{%- assign paper = include.paper -%}
<article class="paper-entry">
  <h3 class="paper-title"><a href="{{ paper.url | relative_url }}">{{ paper.title }}</a></h3>
  <p class="paper-meta">
    <span class="paper-authors">{% include author-list.html authors=paper.authors %}</span>
    {%- if paper.venue %} <span class="paper-venue">{{ paper.venue }}</span>{% endif %}
  </p>
  {% if paper.summary %}<p class="paper-summary">{{ paper.summary }}</p>{% endif %}
  <div class="paper-links">
    {% for link in paper.links %}<a class="button button-small" href="{{ link.url }}" target="_blank" rel="noopener noreferrer">{{ link.label }}</a>{% endfor %}
  </div>
</article>
```

- [ ] **Step 12: Write `_layouts/home.html`**

Compact hero: photo, name, bio (page content), social icons (inline SVG — the only place icons appear, so no separate include), buttons, then selected work.

```html
---
layout: default
---
<section class="profile">
  <img class="profile-photo" src="{{ '/assets/picture.jpg' | relative_url }}" alt="James Traina" width="160" height="160">
  <h1 class="profile-name">{{ site.title }}</h1>
  <div class="profile-bio">{{ content }}</div>
  <div class="profile-icons">
    <a href="https://drive.google.com/uc?export=download&id=1WMr_ua2RZnOMaluS_mRwDkQT8z8FeTrv" target="_blank" rel="noopener noreferrer" aria-label="CV" title="CV">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></svg>
    </a>
    <a href="mailto:james.traina@nyu.edu" aria-label="Email" title="Email">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/></svg>
    </a>
    <a href="https://scholar.google.com/citations?user=Pjrmo2cAAAAJ" target="_blank" rel="noopener noreferrer" aria-label="Google Scholar" title="Google Scholar">
      <svg viewBox="0 0 24 24" fill="currentColor"><path d="M12 3L1 9l11 6 9-4.91V17h2V9L12 3z"/><path d="M5 13.18v4L12 21l7-3.82v-4L12 17l-7-3.82z"/></svg>
    </a>
    <a href="https://github.com/James-Traina" target="_blank" rel="noopener noreferrer" aria-label="GitHub" title="GitHub">
      <svg viewBox="0 0 24 24" fill="currentColor"><path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/></svg>
    </a>
    <a href="https://x.com/EconTraina" target="_blank" rel="noopener noreferrer" aria-label="X" title="X">
      <svg viewBox="0 0 24 24" fill="currentColor"><path d="M18.901 1.153h3.68l-8.04 9.19L24 22.846h-7.406l-5.8-7.584-6.638 7.584H.474l8.6-9.83L0 1.154h7.594l5.243 6.932ZM17.61 20.644h2.039L6.486 3.24H4.298Z"/></svg>
    </a>
  </div>
  <div class="profile-buttons">
    <a class="button" href="{{ '/research/' | relative_url }}">Research</a>
    <a class="button" href="{{ '/teaching/' | relative_url }}">Teaching</a>
    <a class="button" href="{{ '/fun/' | relative_url }}">Fun</a>
  </div>
</section>
<section class="selected">
  <h2>Selected work</h2>
  {% assign sel = site.papers | where_exp: "p", "p.selected" | sort: "date" | reverse %}
  {% for paper in sel %}{% include paper-entry.html paper=paper %}{% endfor %}
</section>
```

- [ ] **Step 13: Write `_layouts/single.html`** (individual paper page)

```html
---
layout: default
---
<article class="paper">
  <h1 class="page-title">{{ page.title }}</h1>
  <p class="paper-meta">
    <span class="paper-authors">{% include author-list.html authors=page.authors %}</span>
    {%- if page.venue %} <span class="paper-venue">{{ page.venue }}</span>{% endif %}
  </p>
  <div class="paper-links">
    {% for link in page.links %}<a class="button button-small" href="{{ link.url }}" target="_blank" rel="noopener noreferrer">{{ link.label }}</a>{% endfor %}
  </div>
  <div class="content">{{ content }}</div>
</article>
```

- [ ] **Step 14: Write `404.html`**

```html
---
layout: default
title: Page not found
permalink: /404.html
---
<h1>Page not found</h1>
<p><a href="{{ '/' | relative_url }}">Back to the homepage</a></p>
```

- [ ] **Step 15: Build and verify the invariants**

```bash
bundle config set --local path vendor/bundle
bundle install
bundle exec jekyll build
grep -ri '<script' _site/ ; echo "grep exit: $?"
```

Expected: build exits 0 (no index page yet — that's fine, content comes in Task 2); grep exit 1 (nothing found).

- [ ] **Step 16: Commit**

```bash
git add -A
git commit -m "Replace academicpages with bespoke minimal Jekyll skeleton"
```

---

### Task 2: Port content from fd98409 (native task #9)

**Goal:** All 12 papers, 2 courses, 4 pages, headshot, and favicon in place with corrected links.

**Files:**
- Create: `_papers/*.md` (12 files, complete contents below), `_courses/ai-finance.md`, `_courses/competitive-strategy.md`, `index.md`, `research.md`, `teaching.md`, `fun.md`, `assets/picture.jpg`, `assets/favicon.svg`

**Acceptance Criteria:**
- [ ] 12 papers, category counts: working=6, published=5, other=1
- [ ] Bio is the approved Assistant Professor version
- [ ] Teaching links to live ai-finance course site; TA history dropped; courses have no individual pages
- [ ] `selected: true` on exactly profit-puzzles and markups-markdowns
- [ ] Three formerly-`"#"` links carry real URLs (beginning-of-trend → SSRN 4362776, production-approach → DOI, biden-labor-competition → ProMarket)
- [ ] Headshot is a real photo (not a placeholder silhouette), `file` reports JPEG

**Verify:** `bundle exec jekyll build && ls _site/papers | wc -l` → 12; `grep -c 'paper-entry' _site/research/index.html` → 12

**Steps:**

- [ ] **Step 1: Acquire headshot from the Google Site**

```bash
curl -sL -o /tmp/headshot 'https://lh3.googleusercontent.com/sitesv/AA5AbUBK_b2DKT-YBGrESG6EOmLLMjcbHTUGEJkstcNHS1OmuaQm_xgSHeIwxt7R9X7vttjpmyWKL4_6Vk76j5KA8JAzhemoSUZpEFgY2Pg01x2XrmLYw3nAxCR9OVJyMvNxbRlQgbWzMn1I3QQkt78yzJjWBgOhiQ40D4V8_5SNERjGh2jM3wxbvRELgdh31MaZCyCfSUNY-kkagX7m9bvJiFz8UCeQWVkgXzneRiOTQXI=w1280'
file /tmp/headshot   # expect an image type, any format
mkdir -p assets
sips -s format jpeg -Z 480 /tmp/headshot --out assets/picture.jpg
```

Then **visually verify** it is a headshot of a person (Read the file). If the URL has expired or returns HTML, STOP and ask James for a photo file — do not ship a placeholder.

- [ ] **Step 2: Write `assets/favicon.svg`**

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64"><rect width="64" height="64" rx="12" fill="#2563eb"/><text x="32" y="43" font-family="-apple-system, Helvetica, Arial, sans-serif" font-size="28" font-weight="700" fill="#ffffff" text-anchor="middle">JT</text></svg>
```

- [ ] **Step 3: Write the 12 paper files** (complete contents — do not improvise)

`_papers/profit-puzzles.md`

```markdown
---
title: "Profit Puzzles and the Fall of Public-Firm Profit Rates"
authors: ["Carter Davis", "Alex Sollaci", "James Traina"]
date: 2024-06-01
category: working
links:
  - label: Paper
    url: "https://raw.githubusercontent.com/James-Traina/Working-Papers/main/Profit-Puzzles.pdf"
summary: "Why have public-firm profit rates fallen while aggregate profits rose? Private-firm profitability has increased, explaining the gap."
selected: true
---
Why have public-firm profit rates fallen while aggregate profits rose? National accounts include all firms; financial markets track only public ones. This paper shows that private-firm profitability has increased since 1980, explaining the divergent trends in profit rates across these two measurement approaches.
```

`_papers/markups-markdowns.md`

```markdown
---
title: "Labor Market Power and Technological Change in US Manufacturing"
authors: ["Ivan Kirov", "James Traina"]
date: 2024-05-01
category: working
links:
  - label: Paper
    url: "https://raw.githubusercontent.com/James-Traina/Working-Papers/main/Markups-Markdowns.pdf"
summary: "Wage markdowns rose substantially while price markups stayed competitively flat. Technological change, not concentration, drove the patterns. Winner of the 2018-19 Stigler Center PhD Dissertation Award."
selected: true
---
Using Census data, we estimate plant-level wage markdowns and price markups in US manufacturing. Wage markdowns increased significantly while price markups remained competitively flat. Technological change rather than market concentration drove these patterns.

Winner of the 2018-19 Stigler Center PhD Dissertation Award.
```

`_papers/measuring-markups-revenue.md`

```markdown
---
title: "Measuring Markups with Revenue Data"
authors: ["Ivan Kirov", "Paolo Mengano", "James Traina"]
date: 2024-04-01
category: working
links:
  - label: SSRN
    url: "https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3912966"
summary: "A new method for estimating markups when output prices are unobserved, using revenue data alone."
---
Standard markup estimation methods require observing output prices. This paper proposes a novel approach using revenue data alone to estimate markups when output prices are unobserved.
```

`_papers/demand-elasticities.md`

```markdown
---
title: "Seven Million Demand Elasticities"
authors: ["Jordan Rosenthal-Kay", "Uyen Tran", "James Traina"]
date: 2024-03-01
category: working
links:
  - label: Paper
    url: "https://jrosenthalkay.github.io/pdfs/Demand_Elasticities.pdf"
summary: "7.5 million product-level price elasticities estimated across regions and years using retail scanner data."
---
We estimate 7.5 million product-level price elasticities across regions and years using retail scanner data.
```

`_papers/measuring-markups-production.md`

```markdown
---
title: "Measuring Markups with Production Data"
authors: ["Zach Flynn", "Amit Gandhi", "James Traina"]
date: 2024-02-01
category: working
links:
  - label: SSRN
    url: "https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3358472"
summary: "Standard production function approaches to markup estimation fail to identify markups. We propose a solution using constant returns to scale."
---
Standard production function methods fail to identify markups. We demonstrate this failure and propose a solution based on constant returns to scale.
```

`_papers/aggregate-market-power.md`

```markdown
---
title: "Is Aggregate Market Power Increasing? Production Trends Using Financial Statements"
authors: ["James Traina"]
date: 2024-01-01
category: working
links:
  - label: SSRN
    url: "https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3120849"
summary: "Public firm markups increased only modestly and are within historical variation, once marketing and management expenses are properly accounted for."
---
Public firm markups increased only modestly and are within historical variation, once marketing and management expenses are properly accounted for in the production approach to markup estimation.
```

`_papers/beginning-of-trend.md` — **link was `"#"` in fd98409; real URL below**

```markdown
---
title: "The Beginning of the Trend: Interest Rates, Profits, and Markups"
authors: ["Anton Bobrov", "James Traina"]
date: 2024-07-01
category: published
venue: "Economics Bulletin, 2024"
links:
  - label: SSRN
    url: "https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4362776"
summary: "Interest rates, profits, and markups -- tracing the beginning of the trend."
---
Bobrov, Anton, and James Traina. 2024. "The Beginning of the Trend: Interest Rates, Profits, and Markups." *Economics Bulletin*.
```

`_papers/production-approach.md` — **link was `"#"` in fd98409; real URL below**

```markdown
---
title: "The Production Approach to Markup Estimation Often Measures Input Distortions"
authors: ["Arshia Hashemi", "Ivan Kirov", "James Traina"]
date: 2022-08-01
category: published
venue: "Economics Letters 217, 2022"
links:
  - label: DOI
    url: "https://doi.org/10.1016/j.econlet.2022.110673"
summary: "The standard production approach to markup estimation often captures input distortions rather than true markups."
---
Hashemi, Arshia, Ivan Kirov, and James Traina. 2022. "The Production Approach to Markup Estimation Often Measures Input Distortions." *Economics Letters* 217.
```

`_papers/resolving-tbtf.md`

```markdown
---
title: "Resolving 'Too Big to Fail'"
authors: ["Nicola Cetorelli", "James Traina"]
date: 2021-01-01
category: published
venue: "Journal of Financial Services Research 60 (1), 2021"
links:
  - label: Paper
    url: "https://www.newyorkfed.org/medialibrary/media/research/staff_reports/sr859.pdf"
summary: "How do we resolve the too-big-to-fail problem in banking?"
---
Cetorelli, Nicola, and James Traina. 2021. "Resolving 'Too Big to Fail.'" *Journal of Financial Services Research* 60 (1).
```

`_papers/bank-complexity.md`

```markdown
---
title: "Evolution in Bank Complexity"
authors: ["Nicola Cetorelli", "Jamie McAndrews", "James Traina"]
date: 2014-12-01
category: published
venue: "Economic Policy Review 20 (2), 2014"
links:
  - label: Paper
    url: "https://www.newyorkfed.org/medialibrary/media/research/epr/2014/1412cet2.pdf"
summary: "How has the organizational complexity of banks evolved over time?"
---
Cetorelli, Nicola, Jamie McAndrews, and James Traina. 2014. "Evolution in Bank Complexity." *Economic Policy Review* 20 (2).
```

`_papers/tbtf-risk.md`

```markdown
---
title: "Do 'Too-Big-to-Fail' Banks Take On More Risk?"
authors: ["Gara Afonso", "João Santos", "James Traina"]
date: 2014-11-01
category: published
venue: "Economic Policy Review 20 (2), 2014"
links:
  - label: Paper
    url: "https://www.newyorkfed.org/medialibrary/media/research/epr/2014/1412afon.pdf"
summary: "Do banks that are perceived as too big to fail take on more risk as a result?"
---
Afonso, Gara, João Santos, and James Traina. 2014. "Do 'Too-Big-to-Fail' Banks Take On More Risk?" *Economic Policy Review* 20 (2).
```

`_papers/biden-labor-competition.md` — **link was `"#"` in fd98409; real URL below**

```markdown
---
title: "How Practical Are Biden's Proposals to Promote Labor Market Competition?"
authors: ["Jordan Rosenthal-Kay", "James Traina"]
date: 2022-01-01
category: other
venue: "ProMarket, 2022"
links:
  - label: Article
    url: "https://promarket.org/2022/03/25/biden-treasury-report-labor-markets"
summary: "An assessment of the Biden administration's proposals to promote labor market competition."
---
Rosenthal-Kay, Jordan, and James Traina. 2022. "How Practical Are Biden's Proposals to Promote Labor Market Competition?" *ProMarket*.
```

- [ ] **Step 4: Write the 2 course files**

`_courses/ai-finance.md`

```markdown
---
title: "AI in Finance"
date: 2024-01-01
meta: "MBA, NYU Stern Abu Dhabi"
summary: "MBA course on artificial intelligence in finance. Everything lives on the course site."
link:
  label: Course site
  url: "https://james-traina.github.io/ai-finance/"
---
```

`_courses/competitive-strategy.md`

```markdown
---
title: "Competitive Strategy"
date: 2022-03-01
meta: "Undergraduate, University of Chicago, Spring 2021 and Spring 2022"
summary: "Applies microeconomics and game theory to firm decision-making: competitive advantage, entry, firm scope, and network effects. Cases include Moneyball, the Cola Wars, Walmart vs. Amazon, and Nintendo and Sega."
link:
  label: Course materials on GitHub
  url: "https://github.com/James-Traina"
---
```

- [ ] **Step 5: Write `index.md`** (bio is the layout's `{{ content }}`)

```markdown
---
layout: home
---
I am an Assistant Professor of Finance at NYU Stern Abu Dhabi. I study how firms set prices and wages -- and what happens when they have the power to distort both. My research uses tools from corporate finance and industrial organization to examine markups, markdowns, and the structural underpinnings of investment, pricing, and financing.
```

- [ ] **Step 6: Write `research.md`**

```markdown
---
layout: default
title: Research
permalink: /research/
---
# Research

## Working Papers

{% assign working = site.papers | where: "category", "working" | sort: "date" | reverse %}
{% for paper in working %}{% include paper-entry.html paper=paper %}{% endfor %}

## Publications

{% assign published = site.papers | where: "category", "published" | sort: "date" | reverse %}
{% for paper in published %}{% include paper-entry.html paper=paper %}{% endfor %}

## Other Writing

{% assign other = site.papers | where: "category", "other" | sort: "date" | reverse %}
{% for paper in other %}{% include paper-entry.html paper=paper %}{% endfor %}
```

- [ ] **Step 7: Write `teaching.md`**

```markdown
---
layout: default
title: Teaching
permalink: /teaching/
---
# Teaching

{% assign courses = site.courses | sort: "date" | reverse %}
{% for course in courses %}
<article class="course-entry">
  <h2>{{ course.title }}</h2>
  <p class="course-meta">{{ course.meta }}</p>
  <p>{{ course.summary }}</p>
  <div class="paper-links"><a class="button button-small" href="{{ course.link.url }}" target="_blank" rel="noopener noreferrer">{{ course.link.label }}</a></div>
</article>
{% endfor %}
```

- [ ] **Step 8: Write `fun.md`** (ported from fd98409, body verbatim)

```markdown
---
layout: default
title: Fun
permalink: /fun/
---
# Fun

## UChicago 2017 Skit Show

I organized comedy videos for the Economics Department. Some highlights:

+ **le bonhomme** -- French expressions gone wrong
+ **Math Camp** -- the PhD student arrival experience
+ **eXperiMintZ** -- natural experiments, naturally
+ **STRUCTURAL BREAK** -- women in economics
+ **eXtenZe** -- the tenure clock, ticking
+ **The Basement Project** -- if you know the Chicago basement, you know

## Capitalisn't

My advisor Luigi Zingales co-hosts [Capitalisn't](https://www.capitalisnt.com/), a podcast about what's working in capitalism and what isn't. Recommended episodes on labor markets, antitrust, the gender wage gap, and economic research methodology.

## Economics Twitter Bots

Two bots worth following: one generates fake econ abstracts, another posts random FRED graphs. Both are funnier than they should be.
```

- [ ] **Step 9: Build and verify counts**

```bash
bundle exec jekyll build
ls _site/papers | wc -l                                  # expect 12
grep -c '<article class="paper-entry"' _site/research/index.html   # expect 12
grep -c '<article class="paper-entry"' _site/index.html            # expect 2
grep -c '<article class="course-entry"' _site/teaching/index.html  # expect 2
grep -ri '<script' _site/ ; echo "grep exit: $?"                   # expect exit 1
```

- [ ] **Step 10: Commit**

```bash
git add -A
git commit -m "Port papers, courses, and pages from fd98409 with corrected links"
```

---

### Task 3: Stylesheet (native task #10)

**Goal:** Single `assets/css/main.scss` implementing the design system: compact hero, PaperMod-derived look, site-local tokens.

**Files:**
- Create: `assets/css/main.scss`

**Acceptance Criteria:**
- [ ] Site-local CSS custom properties (`--gap`, `--radius`, `--border`, etc.)
- [ ] Compact hero (no full-viewport min-height), 0.15s ease transitions, 10px/16px buttons, 4px/8px radii, `#2563eb` prose links, letter-spacing on uppercase
- [ ] Light theme only, system font stack, no webfonts, no imports

**Verify:** `bundle exec jekyll build && test -f _site/assets/css/main.css && grep -c 'transition' _site/assets/css/main.css` → file exists, several transitions

**Steps:**

- [ ] **Step 1: Write `assets/css/main.scss`** (complete file; the leading `---` pair is Jekyll front matter that triggers Sass compilation)

```scss
---
---
/* ===== Tokens (site-local) ===== */
:root {
  --gap: 24px;
  --radius: 8px;
  --radius-small: 4px;
  --main-width: 720px;
  --theme: #ffffff;
  --primary: #1e1e1e;
  --secondary: #6c6c6c;
  --border: #eeeeee;
  --surface: #f8f8f8;
  --link: #2563eb;
  --link-hover: #1d4ed8;
}

/* ===== Base ===== */
*,
*::before,
*::after { box-sizing: border-box; }

body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif;
  font-size: 17px;
  line-height: 1.6;
  color: var(--primary);
  background: var(--theme);
  -webkit-font-smoothing: antialiased;
}

h1, h2, h3 { line-height: 1.25; }

img {
  max-width: 100%;
  border-radius: var(--radius-small);
}

a {
  color: inherit;
  text-decoration: none;
  transition: color 0.15s ease, background 0.15s ease, border-color 0.15s ease;
}

.main {
  max-width: var(--main-width);
  margin: 0 auto;
  padding: 0 var(--gap) calc(var(--gap) * 2);
}

.main h1 { font-size: 26px; margin: calc(var(--gap) * 1.5) 0 0; }
.main h2 { font-size: 21px; margin: calc(var(--gap) * 1.5) 0 0; }

/* Prose links: distinct blue. Buttons opt out via :not(). */
.main p a:not(.button),
.main li a:not(.button) {
  color: var(--link);
}
.main p a:not(.button):hover,
.main li a:not(.button):hover {
  color: var(--link-hover);
}

/* ===== Header ===== */
.header { border-bottom: 1px solid var(--border); }

.header-inner {
  max-width: var(--main-width);
  margin: 0 auto;
  padding: 0 var(--gap);
  height: 56px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.site-name { font-weight: 700; }
.site-name:hover { color: var(--link); }

.nav { display: flex; gap: 18px; }
.nav a { color: var(--secondary); }
.nav a:hover { color: var(--primary); }

/* ===== Profile hero (compact — never full-viewport) ===== */
.profile {
  text-align: center;
  padding: calc(var(--gap) * 3) 0 calc(var(--gap) * 2);
}

.profile-photo {
  width: 160px;
  height: 160px;
  border-radius: 50%;
  object-fit: cover;
}

.profile-name {
  margin: var(--gap) 0 0;
  font-size: 28px;
}

.profile-bio {
  max-width: 560px;
  margin: calc(var(--gap) * 0.5) auto 0;
  color: var(--secondary);
}
.profile-bio p { margin: 0; }

.profile-icons {
  margin-top: var(--gap);
  display: flex;
  justify-content: center;
  gap: 12px;
}
.profile-icons a {
  display: inline-flex;
  padding: 6px;
  border-radius: var(--radius-small);
  color: var(--primary);
}
.profile-icons a:hover { color: var(--link); }
.profile-icons svg { width: 24px; height: 24px; }

.profile-buttons {
  margin-top: var(--gap);
  display: flex;
  justify-content: center;
  gap: 12px;
  flex-wrap: wrap;
}

/* ===== Buttons (10px/16px = 1.5:1 ratio) ===== */
.button {
  display: inline-block;
  padding: 10px 16px;
  border: 1px solid var(--border);
  border-radius: var(--radius);
  background: var(--surface);
  font-size: 15px;
  font-weight: 500;
}
.button:hover {
  border-color: var(--link);
  color: var(--link);
}

.button-small {
  padding: 4px 10px;
  font-size: 13px;
  border-radius: var(--radius-small);
}

/* ===== Paper entries ===== */
.paper-entry {
  padding: var(--gap) 0;
  border-bottom: 1px solid var(--border);
}
.paper-entry:last-of-type { border-bottom: none; }

.paper-title { margin: 0; font-size: 19px; }
.paper-title a:hover { color: var(--link); }

.paper-meta {
  margin: 6px 0 0;
  color: var(--secondary);
  font-size: 15px;
}
.paper-venue::before { content: "·"; margin: 0 6px; }

.paper-summary { margin: 8px 0 0; font-size: 15px; }

.paper-links {
  margin-top: 10px;
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

/* ===== Single paper page ===== */
.paper .page-title { margin-bottom: 0; }
.paper .paper-meta { margin-top: 8px; }
.paper .paper-links { margin-top: 14px; }
.paper .content { margin-top: calc(var(--gap) * 0.75); }

/* ===== Course entries ===== */
.course-entry {
  padding: var(--gap) 0;
  border-bottom: 1px solid var(--border);
}
.course-entry:last-of-type { border-bottom: none; }

.course-entry h2 { margin: 0; font-size: 20px; }

.course-meta {
  margin: 4px 0 0;
  color: var(--secondary);
  font-size: 13px;
  text-transform: uppercase;
  letter-spacing: 0.05em; /* uppercase always gets letter-spacing */
}

/* ===== Footer ===== */
.footer {
  border-top: 1px solid var(--border);
  margin-top: calc(var(--gap) * 2);
  padding: var(--gap);
  text-align: center;
  color: var(--secondary);
  font-size: 14px;
}

/* ===== Responsive ===== */
@media (max-width: 600px) {
  body { font-size: 16px; }
  .header-inner { padding: 0 16px; }
  .main { padding: 0 16px calc(var(--gap) * 2); }
  .profile { padding: calc(var(--gap) * 2) 0 var(--gap); }
  .profile-photo { width: 128px; height: 128px; }
}
```

- [ ] **Step 2: Build and verify**

```bash
bundle exec jekyll build
test -f _site/assets/css/main.css && echo "css ok"
grep -c '0.15s ease' _site/assets/css/main.css   # expect >= 1
```

- [ ] **Step 3: Commit**

```bash
git add assets/css/main.scss
git commit -m "Add site stylesheet implementing the design system"
```

---

### Task 4: Local verification (native task #11)

**Goal:** Captured evidence the site is correct: visual review, link check, invariants.

**Files:** none created (screenshots go to the session, not the repo)

**Acceptance Criteria:**
- [ ] `jekyll serve` renders all pages without errors
- [ ] Screenshots reviewed at 1280px and 390px for: home, research, teaching, fun, one paper page (markups-markdowns)
- [ ] Every external link in `_site/` returns a non-error HTTP status (SSRN may 403 curl as bot-blocking — verify those two URLs in the Playwright browser instead, then treat as pass)
- [ ] All external/PDF links carry `target="_blank"`

**Verify:** link-check loop output shows no 404/410/5xx; screenshots visually match design rules (compact hero, blue prose links, smooth spacing)

**Steps:**

- [ ] **Step 1: Serve**

```bash
bundle exec jekyll serve --port 4123 --detach
```

- [ ] **Step 2: Screenshot every page at both widths** (Playwright MCP)

Navigate to each of `http://127.0.0.1:4123/`, `/research/`, `/teaching/`, `/fun/`, `/papers/markups-markdowns/` — at browser size 1280x900, screenshot; resize to 390x844, screenshot. Review each against the design rules: compact hero, no horizontal scroll on mobile, button padding, letter-spaced course meta, blue prose links.

- [ ] **Step 3: Link check**

```bash
grep -rhoE 'href="https?://[^"]+"' _site | cut -d'"' -f2 | sort -u | while read -r u; do
  code=$(curl -s -o /dev/null -w '%{http_code}' -L --max-time 20 "$u")
  echo "$code $u"
done | sort
```

Expected: all 200 (or 30x→200). Known exception: `papers.ssrn.com` URLs may return 403 to curl — open those in the Playwright browser and confirm the abstract page loads, then treat as pass.

- [ ] **Step 4: New-tab and zero-script invariants**

```bash
grep -L 'target="_blank"' _site/papers/*/index.html   # expect empty (every paper page has new-tab links)
grep -ri '<script' _site/ ; echo "grep exit: $?"      # expect exit 1
```

- [ ] **Step 5: Stop server, record results in task, commit nothing** (no repo changes in this task)

```bash
pkill -f 'jekyll serve --port 4123'
```

---

### Task 5: Rewrite CLAUDE.md for the Jekyll stack (native task #12)

**Goal:** CLAUDE.md describes the site as it now exists — Jekyll, native Pages, zero plugins — with no Hugo or PaperMod references.

**Files:**
- Modify: `CLAUDE.md` (full replacement, complete content below)

**Acceptance Criteria:**
- [ ] `grep -in 'hugo\|papermod' CLAUDE.md` returns ONLY the one `hugo-migration` branch-name line in Technical Stack (content provenance) — no other Hugo/PaperMod references
- [ ] Deployment section describes push-to-build
- [ ] Voice rules unchanged; design values unchanged (reworded as site tokens)

**Verify:** `grep -in 'hugo\|papermod' CLAUDE.md` → exactly 1 line, containing "hugo-migration"

**Steps:**

- [ ] **Step 1: Replace `CLAUDE.md` entirely with:**

```markdown
# CLAUDE.md — Personal Site Build Spec

## Who This Is For

James Traina. NYU Stern Abu Dhabi professor, empirical economics researcher, teaches "AI in Finance" MBA course. GitHub: James-Traina. Email: james.traina@nyu.edu.

## What This Site Is

Personal academic homepage at `https://james-traina.github.io`. Serves as the front door: who James is, what he researches, where he teaches. Not a blog platform. Not a portfolio. A clean, fast reference page that makes it easy to find papers, courses, and contact info.

The course site already lives at `james-traina.github.io/ai-finance/` (separate repo). This site links to it, never duplicates it.

## Technical Stack

Jekyll with a bespoke minimal theme: three layouts, five includes, one stylesheet. No theme dependency, no plugins, no JavaScript.

- **Build**: native GitHub Pages. Push markdown to `master`; GitHub builds and deploys. Nothing to install to edit content.
- **Local preview** (optional): `bundle install && bundle exec jekyll serve`
- **Domain**: `https://james-traina.github.io`
- **Content source history**: papers were migrated from the old Google Site; the intermediate migration content is pinned at commit `fd98409` (tip of the retained `hugo-migration` branch — keep that branch).

### Site Conventions

- Papers are single markdown files in `_papers/`. Front matter: `title`, `authors` (full list), `date`, `category` (working | published | other), `venue` (published/other only), `links` (label + url pairs), `summary`, `selected` (homepage feature flag). Body is the abstract or citation.
- Courses are markdown files in `_courses/`, rendered inline on the Teaching page — no individual course pages.
- Adding a paper = adding one file to `_papers/` and pushing. That's the whole workflow.
- Zero `<script>` tags in output — absolute, no exceptions. Zero plugins. Hand-written meta tags in `_includes/head.html`.

## Voice and Style Rules

**Writing voice**: Sharp, first-person, academically conversational. Sounds like a person, not a university web committee. Exclamation marks and parenthetical asides are intentional. Strong claims, no hedging.

**Hard rules**:
- No ampersands in prose or titles — always "and"
- Oxford commas (serial comma before "and" in lists of three or more)
- No AI slop: no "delve", "crucial", "landscape", "leverage", "at the forefront", "in today's rapidly evolving"
- No emoji unless explicitly requested

## Site Structure

```
Homepage
  ├── Hero: photo, name, bio paragraph, social icons (CV, email, Scholar, GitHub, X), three buttons
  └── Below hero: Selected work (papers with selected: true)

Nav menu:
  Research → papers grouped: Working Papers / Publications / Other Writing
  Teaching → courses (AI in Finance links out to the course site)
  Fun      → skit show, podcast, twitter bots
```

Keep it to 3-4 nav items max. Every page should be one screen of content or less. If a page needs scrolling, it's too long for a personal site.

## Design System

### CSS Patterns That Work

These were validated through iterative visual testing. All tokens are site-local custom properties defined at the top of `assets/css/main.scss`.

```css
/* Hero: compact, never full-viewport */
.profile {
    padding: calc(var(--gap) * 3) 0 calc(var(--gap) * 2);
}

/* Buttons: 1.5:1 horizontal-to-vertical padding ratio */
.button {
    padding: 10px 16px;
}

/* All hover states: smooth 0.15s ease transitions, never snap */
/* Apply to: nav links, buttons, footer links, entries */

/* Uppercase text always gets letter-spacing */
/* text-transform: uppercase → add letter-spacing: 0.05em */

/* Border-radius scale: 4px (small: images, small buttons), 8px (large: buttons, cards) */
/* Never use 2px — it looks accidental next to 8px */

/* Prose links: distinct blue, not inherited color */
/* color: #2563eb; hover: #1d4ed8 */

/* PDFs and external links open in new tabs (target="_blank" in includes) */
```

### Design Principles

1. **Minimalism means restraint, not emptiness** — every element should earn its space, but don't strip away things that help the reader
2. **Only "clearly correct" changes** — if a reasonable designer might disagree, don't do it
3. **Transitions everywhere** — interactive elements that snap between states feel broken; 0.15s ease on color, background, border-color
4. **Use the site's CSS variables** — `var(--gap)`, `var(--radius)`, `var(--border)`, etc. Don't introduce magic numbers
5. **Content below the hero matters** — keep the hero compact; the selected papers should be visible without scrolling far

## Deployment

```bash
git add <changed files>
git commit -m "describe the change"
git push
```

GitHub Pages builds automatically from `master` root. Check build status with:
`gh api repos/James-Traina/James-Traina.github.io/pages/builds/latest`

GitHub Pages requires **public repos** on the free plan, or a Pro/Team plan for private repos.

## What Not To Do

- Don't duplicate course content here — link to `james-traina.github.io/ai-finance/`
- Don't add a blog unless there's actual writing to put in it
- Don't add Jekyll plugins — the site runs on zero and the no-script invariant depends on it
- Don't add JavaScript, webfonts, or dark mode
- Don't commit `_site/`, `vendor/`, `.env`, credentials, or student data
```

- [ ] **Step 2: Verify and commit**

```bash
grep -in 'hugo\|papermod' CLAUDE.md
```

Expected: exactly one line — the `hugo-migration` branch-name mention in Technical Stack (content provenance). Anything else is a failure.

```bash
git add CLAUDE.md
git commit -m "Rewrite CLAUDE.md for the Jekyll stack"
```

---

### Task 6: Ship (native task #13)

**Goal:** New site live at `https://james-traina.github.io`.

**Files:** none (git operations only)

**Acceptance Criteria:**
- [ ] **CONFIRM WITH JAMES before pushing** — this replaces the live site
- [ ] `jekyll-port` fast-forward merged to `master` and pushed
- [ ] `hugo-migration` pushed to origin (durable remote ref for fd98409) and RETAINED — do not delete
- [ ] GitHub Pages build status `built`
- [ ] Live site spot-checked: homepage shows hero + 2 selected papers; `/research/` shows 12 papers; one paper page loads

**Verify:** `curl -s https://james-traina.github.io/ | grep -c 'profile-photo'` → 1 (after build completes)

**Steps:**

- [ ] **Step 1: Ask James to confirm the push** (AskUserQuestion — this is the point of no return for the live site)

- [ ] **Step 2: Merge and push**

```bash
git checkout master
git merge --ff-only jekyll-port
git push origin master
git push -u origin hugo-migration   # durable remote ref for content-source commit fd98409
```

- [ ] **Step 3: Watch the Pages build**

```bash
gh api repos/James-Traina/James-Traina.github.io/pages/builds/latest --jq '.status'
```

Repeat until `built` (typically under 2 minutes). If `errored`, fetch the error with `--jq '.error.message'`.

- [ ] **Step 4: Spot-check the live site**

```bash
curl -s https://james-traina.github.io/ | grep -c 'profile-photo'          # expect 1
curl -s https://james-traina.github.io/research/ | grep -c 'paper-entry'   # expect 12
curl -s -o /dev/null -w '%{http_code}\n' https://james-traina.github.io/papers/markups-markdowns/   # expect 200
```

May need a few minutes for CDN propagation; retry before concluding failure.

- [ ] **Step 5: Tidy local state**

```bash
git branch -d jekyll-port    # fully merged into master
git log --oneline -3         # confirm master tip is the port
```

`hugo-migration` stays. Done.
