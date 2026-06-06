# CLAUDE.md — Personal Site Build Spec

## Who This Is For

James Traina. NYU Stern Abu Dhabi professor, empirical economics researcher, teaches "AI in Finance" MBA course. GitHub: James-Traina. Email: james.traina@nyu.edu.

## What This Site Is

Personal academic homepage at `https://james-traina.github.io`. Serves as the front door: who James is, what he researches, where he teaches. Not a blog platform. Not a portfolio. A clean, fast reference page that makes it easy to find papers, courses, and contact info.

The course site already lives at `james-traina.github.io/ai-finance/` (separate repo). This site should link to it, not duplicate it.

## Technical Stack

Hugo + PaperMod theme (specifically the [pmichaillat/hugo-website](https://github.com/pmichaillat/hugo-website) academic variant). This is proven — the course site uses it successfully. Reuse the same foundation.

- **Hugo**: `brew install hugo` (v0.147+)
- **Theme**: PaperMod via pmichaillat/hugo-website template
- **Deploy**: GitHub Pages from `public/` directory (committed to repo)
- **Domain**: `https://james-traina.github.io`
- **No build pipeline**: just `hugo` locally, commit `public/`, push

### Config Decisions (carry from course site)

```yaml
defaultTheme: light
disableThemeToggle: true      # No dark mode toggle
ShowShareButtons: false
ShowReadingTime: false
ShowBreadCrumbs: false
ShowPostNavLinks: false
disableAnchoredHeadings: true
disableScrollToTop: false
ShowCodeCopyButtons: false    # No code on personal site
profileMode:
  enabled: true               # Hero-style homepage
```

## Voice and Style Rules

**Writing voice**: Sharp, first-person, academically conversational. Sounds like a person, not a university web committee. Exclamation marks and parenthetical asides are intentional. Strong claims, no hedging.

**Hard rules**:
- No ampersands in prose or titles — always "and"
- Oxford commas (serial comma before "and" in lists of three or more)
- No AI slop: no "delve", "crucial", "landscape", "leverage", "at the forefront", "in today's rapidly evolving"
- No emoji unless explicitly requested

## Site Structure

```
Homepage (profileMode)
  ├── Hero: name, one-line tagline, social icons, CTA buttons
  └── Below hero: brief content (selected papers, news, or nothing)

Nav menu:
  Research → papers/publications list
  Teaching → courses taught (link to ai-finance site)
  About    → bio, CV link, contact
```

Keep it to 3-4 nav items max. Every page should be one screen of content or less. If a page needs scrolling, it's too long for a personal site.

## Design System (distilled from course site polish)

### CSS Patterns That Work

These were validated through iterative visual testing on the course site:

```css
/* Hero: do NOT use PaperMod's full-viewport min-height */
.main .profile {
    min-height: auto;
    padding-top: calc(var(--gap) * 3);    /* 72px */
    padding-bottom: calc(var(--gap) * 3);
}

/* Buttons: 1.5:1 horizontal-to-vertical padding ratio */
.button {
    padding: 10px 16px;
}

/* All hover states: smooth 0.15s ease transitions, never snap */
/* Apply to: nav links, buttons, footer links, cards, tag pills */

/* Uppercase text always gets letter-spacing */
/* text-transform: uppercase → add letter-spacing: 0.05em */

/* Border-radius scale: 4px (small: code, images), 8px (large: cards, blocks) */
/* Never use 2px — it looks accidental next to 8px */

/* Links: distinct blue, not inherited color */
.post-content a { color: #2563eb; }
.post-content a:hover { color: #1d4ed8; }

/* PDFs open in new tabs (via render-link.html) */
/* External links open in new tabs (already in PaperMod) */
```

### Design Principles

1. **Minimalism means restraint, not emptiness** — every element should earn its space, but don't strip away things that help the reader
2. **Only "clearly correct" changes** — if a reasonable designer might disagree, don't do it
3. **Transitions everywhere** — interactive elements that snap between states feel broken; 0.15s ease on color, background, border-color
4. **Respect PaperMod's variable system** — use `var(--gap)`, `var(--radius)`, `var(--border)`, etc. Don't introduce magic numbers
5. **Content below the hero matters** — PaperMod's default profile mode wastes a full viewport on the hero; always override `min-height`

## Deployment

```bash
cd site/  # or wherever hugo source lives
hugo
git add public/
git commit -m "build"
git push
```

GitHub Pages serves from the `public/` directory on `main` branch. Configure in repo Settings → Pages.

Note: GitHub Pages requires **public repos** on the free plan, or a Pro/Team plan for private repos.

## What Not To Do

- Don't duplicate course content here — link to `james-traina.github.io/ai-finance/`
- Don't add a blog unless there's actual writing to put in it
- Don't over-engineer the CSS — the pmichaillat template is already good
- Don't add JavaScript beyond what PaperMod provides
- Don't use dark mode (disabled by design)
- Don't commit `.env`, credentials, or student data
