# CLAUDE.md — Personal Site Build Spec

## Who This Is For

James Traina. NYU Stern Abu Dhabi professor, empirical economics researcher, teaches "AI in Finance" MBA course. GitHub: James-Traina. Email: james.traina@nyu.edu.

## What This Site Is

Personal academic homepage at `https://james-traina.github.io`. Serves as the front door: who James is, what he researches, where he teaches. Not a blog platform. Not a portfolio. A clean, fast reference page that makes it easy to find papers, courses, and contact info.

The course site already lives at `james-traina.github.io/ai-finance/` (separate repo). This site links to it, never duplicates it.

## Technical Stack

Jekyll with a bespoke minimal theme: three layouts, five includes, one stylesheet. No theme dependency, no plugins, no JavaScript.

- **Build**: native GitHub Pages. Push markdown to `master`; GitHub builds and deploys. Nothing to install to edit content.
- **Local preview** (optional): `bundle install && bundle exec jekyll serve` (requires Ruby 3.1: `export PATH="$(brew --prefix ruby@3.1)/bin:$PATH"`)
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
