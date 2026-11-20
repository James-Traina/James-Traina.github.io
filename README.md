# james-traina.github.io

Personal academic homepage for James Traina, Assistant Professor of Finance at NYU Stern Abu Dhabi. It is the front door: who James is, what he researches, and where he teaches.

Live at https://james-traina.github.io.

## Stack

Jekyll with a bespoke Crisp Editorial Sans theme: two layouts, five includes, and one stylesheet. No theme dependency, no plugins, no JavaScript, zero webfonts, and light-only. GitHub Pages builds and deploys automatically on push to `master`, so nothing needs to be installed to edit content.

Invariants, in order of importance:

- Zero Jekyll plugins.
- Zero `<script>` tags in the output.
- No JavaScript, no webfonts, and no dark mode.
- Native GitHub Pages build (the `github-pages` gem pins the Jekyll version).

## Adding a paper

Add one Markdown file to `_papers/` (`output: false`, so no standalone paper pages are generated) and push. Front matter: `title`, `authors`, `date`, `category` (`working`, `published`, or `other`), `venue` (published and other), `links` (`label` and `url` pairs; the first link is the paper title's direct destination), optional `honors`, `summary` (rendered on Home `Selected Work` only), and `selected` (homepage feature flag). The body is the abstract; papers with a body automatically get a zero-JS inline `<details>` `Abstract` toggle. `category` must be exactly one of the three values, or the paper drops out of the Research page groups.

## Local preview (optional)

Requires Ruby 3.3:

    export PATH="/opt/homebrew/opt/ruby@3.3/bin:$PATH"
    bundle install
    bundle exec jekyll build
    python3 -m http.server -d _site 4123

## Layout

- `index.md` — home: left-aligned editorial hero (bio, inline contact/profile links, `4:5` portrait) and Selected Work.
- `research.md` — papers grouped into Working Papers, Publications, and Other Work.
- `teaching.md` — courses rendered inline from `_courses/` (with optional links).
- `fun.md` — skit show, podcast, and Twitter bots.
- Top nav also links directly to `CV` (`cv_url` in `_config.yml`).

See [CLAUDE.md](CLAUDE.md) for the full build spec and [docs/](docs/) for historical design rationale.
