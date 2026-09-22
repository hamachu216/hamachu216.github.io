# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Satoki Hamanaka's personal academic site (<https://hamachu216.github.io>), built from the
[Academic Pages](https://github.com/academicpages/academicpages.github.io) Jekyll template and served by
GitHub Pages. **Pushing to `master` builds and deploys.**

**A green local build does not mean a green deploy.** `docker compose up` builds with this repo's
`Gemfile` (plain `jekyll` plus the plugins listed there); GitHub Pages builds with the `github-pages` gem,
whose plugin set is larger. The gap has already broken a deploy once: `jekyll-optional-front-matter` is in
the Pages set and not in ours, so Pages renders every root `*.md` without front matter as a page — it
tried to run this file through Liquid, hit the `{%- if … %}` in an example, and failed the build with
`'if' tag was never closed in CLAUDE.md`. Hence `CLAUDE.md` in `exclude:`. **Any new root-level Markdown
that quotes Liquid must be added to `exclude:` in `_config.yml`, or it will pass locally and break the
deploy.** (`AGENTS.md` is not excluded and is published as a page; harmless, but that is why.)

There is no CI workflow of the repo's own (the template's were removed in `218e7a3`), so after a push
check the deploy itself: `gh run list --limit 3`, and `gh run view <id> --log-failed` when it is red.

Almost every task here is a content edit, not a code change. `AGENTS.md` is the upstream template's file;
it has nothing to say about this fork.

## Commands

```sh
bundle install
bundle exec jekyll serve -l -H localhost   # http://localhost:4000, live reload
docker compose up                          # same thing in Ruby 3.2 (_config.yml + _config_docker.yml)
npm run build:js                           # only after editing assets/js/*.js → assets/js/main.min.js
```

`_config.yml` is not reloaded by `jekyll serve`; restart the server after changing it.
There is no test suite.

## Architecture

Data first: each publication and course is one Markdown file whose YAML front matter *is* the record.
The pages are near-empty loops over those collections — never retype an entry's content into a page.

| Change | Edit |
| --- | --- |
| Site title, sidebar profile, social links, publication category headings | `_config.yml` |
| Header menu | `_data/navigation.yml` |
| Front page (bio, Experience, News) | `_pages/about.md` |
| CV | `_pages/cv.md` (hand-written Markdown) |
| A paper | `_publications/YYYY-MM-DD-slug.md` |
| A course | `_teaching/YYYY-term-slug.md` |
| Downloadables (CV PDF, papers, slides) | `files/` → `https://hamachu216.github.io/files/...` |

`_pages/publications.html` groups entries by the `category` front-matter key, in the order of
`publication_category` in `_config.yml`: `manuscripts` (Journal Articles), `conferences` (Conference
Papers), `workshops` (Workshop Papers, Posters, and Demos). A file whose `category` is not one of those
three silently renders nowhere.

Publication front matter (see `_publications/2026-08-04-sensor-augmented-vap-sigdial.md`); `citation` is
raw HTML with the author's own name in `<b>`, quotes as `&quot;`, venue in `<i>`:

```yaml
title: "Paper Title"
collection: publications
category: conferences
permalink: /publication/YYYY-MM-DD-slug
date: YYYY-MM-DD
venue: 'Venue Name'              # rendered as the .publication__venue badge
award: 'Best Paper Award'        # optional; rendered as the coloured .publication__award badge
doi: '10.1145/3712345.3712346'   # optional; bare DOI, the template prepends https://doi.org/
paperurl: 'https://hamachu216.github.io/files/paper.pdf'   # optional; adds a Download Paper link
citation: '<b>Satoki Hamanaka</b>, Co Author. &quot;Title&quot;. <i>Venue</i>, 2026, City, Country, pp. 1-9.'
```

An award belongs in `award:` only — do not also spell it out at the end of `citation`, or the two copies
drift apart. `_pages/cv.md` keeps its own hand-written "Awards and honors" list, which is the one place
awards for talks with no `_publications/` entry can live.

Unlisted/in-submission work is deliberately absent from `_publications/` (`f16f027`); don't re-add papers
found only in `_pages/cv.md`.

## Conventions

**Diverge from the upstream template only on purpose, and keep the list short.** The rendering of a
publication now differs from Academic Pages in three deliberate ways, all of them in
`_includes/archive-single.html` and `_layouts/single.html`:

- The `Recommended citation:` label is deleted (`5f97085`).
- The venue is a `.publication__venue` badge instead of `Published in <i>…</i>`, followed by an optional
  `.publication__award` badge.
- The branch cascade over `citation`/`paperurl`/`slidesurl`/`bibtexurl` is one `capture` … `split` …
  `join` block. Upstream enumerates every combination; adding `[DOI]` to that shape would have taken it to
  ~30 branches. **Add a new link type by adding one `{%- if … %}` line inside the capture — never by
  branching.**

`f16f027` once restored that cascade to upstream's text to keep merges clean; the DOI work overrode that
trade-off. Before pulling upstream changes into these two files, expect a conflict here and re-apply the
three points above rather than taking either side wholesale.

A Talks page was built over the `talks` collection and taken out again in the same day (`c376f2a`, then
this commit): two workshop visits did not pay for a menu item, a page, a collection and a meta-line branch
in each of those two files. They are four lines of hand-written Markdown — `Academic visits` in
`_pages/cv.md`, and two `## News` bullets — and the collection, `_layouts/talk.html` (a copy of
`single.html` keyed on a `talk_type` nothing set) and `_pages/talks.html` are gone. **Don't rebuild it for
a handful of entries.** A real list of invited talks would be worth the collection; a few trips are not.

Badge CSS lives at the end of `_sass/layout/_archive.scss`. The venue badge uses only pre-existing
`--global-*` custom properties, so it follows all 6 themes × light/dark for free; the award badge needed a
warm hue no theme defines, so `--publication-award-color` / `--publication-award-bg` are set once on
`:root` and once on `html[data-theme="dark"]` (the theme switch is `data-theme`, not a media query).

Content rules that still apply as written in the root `CLAUDE.md` (Donenfeld の精神): prefer deleting an
entry, a branch, or a file over adding one; this site's value is in the data, not in machinery around it.

Dormant template leftovers, referenced by nothing: `scripts/` and `_layouts/cv-layout.html` +
`_includes/cv-template.html` drive a JSON-based CV from `_data/cv.json`, which was deleted in `a0c26ca`;
`markdown_generator/` bulk-generates `_publications/` from CSV/BibTeX; `_portfolio/` and `_posts/`
are empty. Don't wire any of them back in without being asked.
