# CLAUDE.md — Personal Site Build Spec

## Who This Is For

James Traina. Assistant Professor of Finance at NYU Stern Abu Dhabi, empirical economics and finance researcher, teaches "AI in Finance" MBA course. GitHub: James-Traina. Email: james.traina@nyu.edu.

## What This Site Is

Personal academic homepage at `https://james-traina.github.io`. Serves as the front door: who James is, what he researches, and where he teaches. Not a blog platform. Not a portfolio. A clean, fast reference page that makes it easy to find papers, courses, and contact info.

## Technical Stack

Jekyll with a bespoke Crisp Editorial Sans theme: two layouts (`default.html`, `home.html`), five includes (`head.html`, `header.html`, `footer.html`, `author-list.html`, `paper-entry.html`), and one stylesheet (`assets/css/main.scss`). No theme dependency, no plugins, no JavaScript, zero webfonts, light-only.

- **Build**: native GitHub Pages. Push markdown to `master`; GitHub builds and deploys. Nothing to install to edit content.
- **Local preview** (optional):
  ```bash
  export PATH="/opt/homebrew/opt/ruby@3.3/bin:$PATH"
  bundle install
  bundle exec jekyll build
  python3 -m http.server -d _site 4123
  ```
- **Domain**: `https://james-traina.github.io`
- **Content source history**: papers were migrated from the old Google Site; the intermediate migration content is pinned at commit `fd98409` (tip of the retained `hugo-migration` branch — keep that branch).

### Site Conventions

- `_config.yml` holds single-source-of-truth `email` and `cv_url` values, and excludes local scratch directories (`notes`, `.agents`, `.claude`, `.playwright-mcp`, `.serena`).
- Papers are markdown files in `_papers/` with `output: false` (no `/papers/<slug>/` stub pages). Front matter: `title`, `authors` (full list; `"James Traina"` is stripped in `"with ..."` coauthor lists), `date`, `category` (`working` | `published` | `other`), `venue` (published/other), `links` (`label` + `url` pairs; the first link is the paper title's direct target), optional `honors`, `summary` (rendered on Home `Selected Work` only), `selected` (homepage feature flag). Body is the abstract markdown — papers with a non-empty body get a zero-JS inline `<details>` `Abstract` toggle.
- `category` must be exactly one of `working`, `published`, or `other` — anything else silently drops the paper from the Research page groups.
- Working papers sort by `date` descending, using dates as display ranks; give a new working paper a later date than the current top entry to list it first. Publications carry real dates.
- `_includes/author-list.html` must remain Liquid 4.0-compatible (GitHub Pages runs Jekyll 3.10) — avoid Liquid 5 filters like `reject` or `where_exp`.
- Courses are markdown files in `_courses/`, rendered inline on the Teaching page — no individual course pages. `link` (`label` + `url`) is optional.
- Adding a paper = adding one file to `_papers/` and pushing. That's the whole workflow.
- Zero `<script>` tags in output — absolute, no exceptions. Zero plugins. Hand-written meta tags in `_includes/head.html`.

## Voice and Style Rules

**Writing voice**: Sharp, first-person, academically conversational. Sounds like a person, not a university web committee. Strong claims, no hedging.

**Hard rules**:
- No ampersands in prose or titles — always "and"
- Oxford commas (serial comma before "and" in lists of three or more)
- No AI slop: no "delve", "crucial", "landscape", "leverage", "at the forefront", "in today's rapidly evolving"
- No emoji unless explicitly requested

## Site Structure

```
Homepage
  ├── Editorial Hero: name, bio, inline contact/profile text links, 4:5 portrait photo
  └── Below hero: Selected Work (papers with selected: true + concise summary)

Nav menu:
  Research → Working Papers / Publications / Other Work (inline <details> abstracts)
  Teaching → courses rendered from _courses/ (optional links)
  Fun      → skit show, podcast, Twitter bots
  CV       → external link to PDF viewer (site.cv_url)
```

Keep nav items to 4 max. Keep pages concise, scannable, and balanced.

## Design System — Crisp Editorial Sans

All tokens are site-local custom properties defined at the top of `assets/css/main.scss`:

- Palette: `--bg: #ffffff`, `--ink: #111315`, `--secondary: #525860`, `--muted: #767d87`, `--border: #e6e8eb`, `--surface: #f6f7f9`, `--accent: #1d3f6e`, `--accent-hover: #102645`.
- Typography: system sans-serif stack, `16.5px` base (`16px` mobile), `1.65` line-height, `12.5px` uppercase tracked `.section-heading` labels (`letter-spacing: 0.08em`), italicized `.paper-venue`.
- Links: inline text links (`.entry-links`, `.hero-links`, muted underlined `.coauthor-link`) instead of gray pill buttons.
- Transitions and accessibility: `0.15s ease` transitions with `@media (prefers-reduced-motion: reduce)` guard, `:focus-visible` accent outlines, and `aria-current="page"` on active nav items.
- Border-radius scale: `4px` (`--radius-small`), `8px` (`--radius`, portrait photo).
- External links and PDFs open in new tabs (`target="_blank" rel="noopener noreferrer"`).

## Deployment

```bash
git add <changed files>
git commit -m "describe the change"
git push
```

GitHub Pages builds automatically from `master` root. Check build status with:
`gh api repos/James-Traina/James-Traina.github.io/pages/builds/latest`

## What Not To Do

- Don't link unpublished/private course repos or add unrequested papers
- Don't add a blog unless there's actual writing to put in it
- Don't add Jekyll plugins — the site runs on zero and the no-script invariant depends on it
- Don't add JavaScript, webfonts, or dark mode
- Don't commit `_site/`, `vendor/`, `notes/`, `.agents/`, `.env`, credentials, or student data
