# james-traina.github.io

Personal academic homepage for James Traina, Assistant Professor of Finance at NYU Stern Abu Dhabi. It is the front door: who James is, what he researches, and where he teaches. Papers, courses, and contact on one clean, fast page.

Live at https://james-traina.github.io.

## Stack

Jekyll with a bespoke minimal theme: three layouts, five includes, and one stylesheet. No theme dependency, no plugins, and no JavaScript. GitHub Pages builds and deploys automatically on push to `master`, so nothing needs to be installed to edit content.

Invariants, in order of importance:

- Zero Jekyll plugins.
- Zero `<script>` tags in the output.
- No JavaScript, no webfonts, and no dark mode.
- Native GitHub Pages build (the `github-pages` gem pins the Jekyll version).

## Adding a paper

Add one Markdown file to `_papers/` and push. Front matter: `title`, `authors`, `date`, `category` (working, published, or other), `venue` (published and other only), `links` (label and url pairs), `summary`, and `selected` (homepage feature flag). The body is the abstract or citation. `category` must be exactly one of the three values, or the paper drops out of the Research page groups.

## Local preview (optional)

Requires Ruby 3.1:

    export PATH="$(brew --prefix ruby@3.1)/bin:$PATH"
    bundle install
    bundle exec jekyll serve

## Layout

- `index.md` — home: hero, bio, social icons, and selected work.
- `research.md` — papers grouped into Working Papers, Publications, and Other Writing.
- `teaching.md` — courses, rendered inline from `_courses/`.
- `fun.md` — skit show, podcast, and Twitter bots.

See `CLAUDE.md` for the full build spec and `docs/` for the design rationale.
