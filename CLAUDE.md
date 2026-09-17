# Project: asyamagazinnik.com

Personal academic website for Asya Magazinnik (political science / causal
inference / computational social science, Hertie School, Berlin).

**Live at <https://asyamagazinnik.com>** since 15 September 2026. Built with
Quarto, hosted free on GitHub Pages. The old Squarespace site is gone.

## Context

- I am fluent in R (tidyverse, spatial, simulation) and LaTeX/Beamer. I am not
  a web developer. Explain front-end choices rather than assuming I know them.
- Design references: <https://www.hannohilbig.com/>,
  <https://saschariaz.com/>, and <https://taraslough.github.io/> (the About
  page layout and the name-in-the-top-bar behaviour came from Slough's site).

## Working preferences

- Show me the plan before making sweeping changes.
- Keep the CSS legible and commented — I want to be able to tweak it myself.
- Don't add analytics, cookie banners, or JavaScript unless I ask.
- Don't open a browser tab after each edit; `quarto preview` live-reloads the
  tab I already have open.

---

# Where things live

    ~/Projects/website/            <- this folder; the site source
      *.qmd                        four pages: index, research, teaching, cv
      styles.scss                  ALL styling, heavily commented
      _quarto.yml                  site config, navbar, resources list
      papers/                      paper PDFs + appendices (descriptive slugs)
      syllabi/                     the 8 course syllabi
      talks/                       talk slides (PolMeth 2026)
      cv/Magazinnik_CV.pdf         the CV the site serves
      photo/asya-magazinnik.jpg    headshot, EXIF-stripped
      fonts/                       6 self-hosted Roboto / Roboto Slab woff2
      favicon.png, apple-touch-icon.png, icon-512.png
      CNAME                        the custom domain — see warning below
      _originals/                  NOT published, NOT in git. Holds the
                                   untouched headshot (still has GPS EXIF)
                                   and the pre-migration DNS record.

    ~/Projects/math-for-data-science/   git repo -> github.com/magazinnik/math-for-data-science
    ~/Projects/stats-ii/               git repo -> github.com/magazinnik/stats-ii

The two slide repos are separate from the website; the Teaching page just
links to them.

---

# Making updates

Work in `~/Projects/website`.

    quarto preview          # live-reloading local server, usually :4321
    ...edit...
    quarto publish gh-pages # builds and deploys to the live site

`quarto publish gh-pages` is the only deploy step. It rebuilds, pushes to the
`gh-pages` branch, and the live site updates in a minute or two. Commit the
source separately (`git add -A && git commit && git push`) — publishing does
not do that for you.

## Common edits

**Add a paper** — copy an existing `<li class="paper ...">` block in
`research.qmd`. The classes after `paper` are its keywords (see below). Put
the PDF in `papers/` with a descriptive name. Link the title to its DOI.

**Add an Updates entry** — copy an `<li>` in the `<ul class="news">` block in
`index.qmd`, newest at the top. Only ~3 are visible; the rest scroll inside
the panel.

**Add a course** — copy a `<div class="course">` block in `teaching.qmd`.
Syllabus goes in `syllabi/`.

**Change the look** — everything is in `styles.scss`, organised by section
with comments explaining *why*, not just what. The colour, font and column
width variables are all at the top.

**Replace the CV** — drop the new PDF at `cv/Magazinnik_CV.pdf` and publish.

## Keywords (the Research page filter)

Nine keywords, defined in three places that must stay in sync:

1. an `<option>` in the `<select>` in `research.qmd`
2. the name in the `@each` list in `styles.scss`
3. the class on each relevant `<li class="paper ...">`

    conjoint · education · federalism · housing · immigration
    measurement · geography · sheriffs · voting

A paper carries a class for *every* keyword it belongs to, so nothing is
duplicated. The filter is pure CSS: `option:checked` plus `:has()`. No
JavaScript. A browser without `:has()` support simply shows every paper.

---

# Traps we actually hit

Worth reading before debugging something that "isn't updating".

**Quarto preview serves stale builds.** If you run `quarto render` while
`quarto preview` is running, the preview's file watcher can overwrite `_site/`
with older output. Symptom: the browser contradicts the source file. Fix: kill
preview, `rm -rf _site .quarto`, render, restart preview. Don't run both.

**Preview does not re-copy changed static files.** Editing a PNG or PDF in
`photo/`, `papers/` etc. does not trigger a re-copy — the old file keeps being
served. Fix: `touch _quarto.yml` to force a rebuild.

**Anything new must be added to `resources:` in `_quarto.yml`,** or it is not
published at all and every link to it 404s. This applies to whole folders.

**`CNAME` must stay in `resources:`.** It tells GitHub Pages which domain
serves the site. If it stops being republished, the custom domain is forgotten
on the next deploy and the site goes down.

**The email address is percent-encoded on purpose.** In `index.qmd` the
mailto link is written as `mailto:%61%2E%6D...` so the page contains no
readable address for a harvester. Browsers decode it before use, so it works
as a normal email link. HTML entities do *not* work for this — pandoc decodes
them back to plain text when it builds the page. If you edit that link, keep
it percent-encoded.

**Linked titles lose their styling.** Quarto's base CSS sets
`a { font-weight: 400 }`, so a link inside a bold title renders unbolded and
crimson. `.pub-title a` and `.course-title a` fix this with
`color: inherit; font-weight: inherit`. Reuse that pattern for any new linked
title.

**CSS source order matters more than you'd think.** A `@media` block adds no
specificity. The `.news` width override has to sit *after* the base
`.news { margin: ... }` shorthand or it silently does nothing.

**Set a custom domain AFTER DNS points at the host, not before.** GitHub
checks DNS at the moment you set the domain. We set it first, so certificate
provisioning never started and sat idle. Clearing and re-setting the domain
fixed it in seconds.

---

# Design decisions

- **Single centered column**, 42rem. On the About page the hero and the
  photo/bio row hang 7rem into the left margin above 1040px, and the bio and
  Updates list run 3.5rem past the right edge.
- **No JavaScript** in anything I wrote. The topic filter is CSS `:has()`,
  the abstracts and course descriptions are native `<details>`, the CV page is
  an `<object>` handing the PDF to the browser's own viewer. Quarto still
  ships ~8 of its own scripts; search is disabled (`search: false`) because it
  was pulling in Fuse.js.
- **No Google Fonts hotlink** — it would send every visitor's IP to Google, a
  GDPR problem in Germany. Roboto and Roboto Slab are self-hosted in `fonts/`
  as 6 variable-font woff2 files (156 KB). The `@font-face` block is at the
  top of `styles.scss`.
- **Typography**: Roboto Slab throughout, headings and body. Nav links stay in
  the sans face. Curly quotes and apostrophes are written as HTML entities
  (`&ldquo;` `&rsquo;`), em dashes always spaced, en dashes never.
- **Page headings are hidden** on every page via `:has()` — the nav already
  says where you are. The YAML `title` is kept so browser tabs and link
  previews still work.
- **Favicon** is a script "A" (Savoye LET) in crimson on white. To regenerate:
  render the letter in Chrome headless, measure its alpha bbox, scale to ~60%
  of the tile, compose on a rounded white square. Two letters were illegible
  at 16px; a single letter is not.

---

# Hosting and domain

- **Source**: <https://github.com/magazinnik/asyamagazinnik.com> (public)
- **Deploy branch**: `gh-pages`, written by `quarto publish`
- **Registrar**: Tucows, resold by Squarespace. $20/yr, renews 14 Dec 2026,
  auto-renew ON, WHOIS privacy ON, transfer lock ON.
- **DNS** is still managed in the Squarespace domain panel:
  - four A records at `@` → `185.199.108-111.153` (GitHub Pages)
  - `www` CNAME → `magazinnik.github.io`
  - a `_domainconnect` CNAME, harmless, left in place
- **TLS**: Let's Encrypt via GitHub, auto-renewing, HTTPS enforced.
- The pre-migration DNS is recorded in `_originals/dns-before-migration.txt`.
  There were never any MX records, so email was never at risk. There *was* an
  HTTPS/SVCB record pinning Squarespace IPs — easy to miss, and it would have
  silently overridden the A records.

Possible later: move the registration to Cloudflare Registrar or Porkbun
(~half the price). Needs the transfer lock off and an auth code.

---

# Still outstanding

- **The CV disagrees with the site** on the MIT course: the CV says *Formal
  Approaches to American Political Institutions*, the syllabus and site say
  *Formal Approaches to Political Institutions (17.S953)*.
- **The CV dropped two preprint links** (OSF for the conjoint paper, SSRN for
  Reform Drift) that the Research page still carries.
- **The Policy Adjacent** is cited as *AJPS* 1–16 because it is still
  early-view. When it gets an issue, update both the site and the CV.
- **No contact route.** The Contact page was deleted and no email address
  appears anywhere on the site.
- **The Math for Data Science syllabus** has "Fall Semester 2024" in its
  header though it is the 2025 version.
- **`figures/`** is empty and still listed in `resources:`; the CSS for
  figures inside abstracts is unused but kept in case it is wanted later.
- **DNS TTL** is at 30 minutes from the migration; could go back to a few hours.
