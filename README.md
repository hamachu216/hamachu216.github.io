# hamachu216.github.io

Personal academic website of Satoki Hamanaka — <https://hamachu216.github.io>

Built with [Academic Pages](https://github.com/academicpages/academicpages.github.io), a Jekyll
template hosted on GitHub Pages. Pushing to `master` rebuilds and redeploys the site.

## Where the content lives

| What | Where |
| --- | --- |
| Site-wide settings, sidebar profile, social links | `_config.yml` |
| Header menu | `_data/navigation.yml` |
| Front page | `_pages/about.md` |
| CV page | `_pages/cv.md` |
| Publications (one file per paper) | `_publications/` |
| Teaching (one file per course) | `_teaching/` |
| Downloadable files (CV PDF, papers, slides) | `files/` → `https://hamachu216.github.io/files/...` |

## Adding a publication

Create `_publications/YYYY-MM-DD-slug.md`:

```yaml
---
title: "Paper Title"
collection: publications
category: manuscripts   # manuscripts | conferences | workshops
permalink: /publication/YYYY-MM-DD-slug
date: YYYY-MM-DD
venue: 'Venue Name'
paperurl: 'https://hamachu216.github.io/files/paper.pdf'
citation: 'Authors. &quot;Title&quot;. <i>Venue</i>, year, pages.'
---
```

Category headings are defined under `publication_category` in `_config.yml`.

## Local preview

```sh
bundle install
bundle exec jekyll serve -l -H localhost
```
