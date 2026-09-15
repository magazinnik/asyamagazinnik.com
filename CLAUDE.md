# Project: personal academic website rebuild

## Context

I'm an academic (political science / causal inference / computational social
science) based in Berlin. I'm rebuilding my personal website.

- Current site: https://www.asyamagazinnik.com/, built on Squarespace.
- Goal: get off Squarespace, keep the domain, own the source, edit in text
  files rather than a drag-and-drop editor.
- I'm fluent in R (tidyverse, spatial, simulation) and LaTeX/Beamer. I am not
  a web developer. Explain front-end choices rather than assuming I know them.

## Design references

Two sites I want to look and feel like — minimal, typographic, fast, no
framework bloat, essentially one long readable column:

- https://www.hannohilbig.com/
- https://saschariaz.com/

## Structure

Organize research by **theme**, not as a flat CV-style list of papers. The
three clusters are roughly:

1. Electoral systems, voting rights, and local/subnational representation
2. Conjoint experimental methodology and the measurement of preferences
3. Housing policy and land use

Each theme gets a short framing paragraph explaining the question, with the
relevant papers listed underneath.

The site is five nav items: About, Research, Teaching, CV (a direct link to
`pdf/cv.pdf`), and Contact. There were once Group and Data & Software pages;
both were dropped in September 2026. Every replication link from the old
Data & Software page already appears on the relevant paper in `research.qmd`,
so nothing was lost.

## Stack

**Decided: Quarto.** Pages are `.qmd`, styling is one commented `styles.scss`.
Render with `quarto render`, preview with `quarto preview`.

Quarto is currently only available as the copy bundled inside RStudio:
`/Applications/RStudio.app/Contents/Resources/app/quarto/bin/quarto`. To get it
on `PATH`, run `brew install --cask quarto`.

Hosting is GitHub Pages, Cloudflare Pages, or Netlify — all free. Not yet set up.

### Design decisions

- Layout is a **single centered column**, not the sidebar that saschariaz.com
  actually uses. Colors and link styling are taken from his stylesheet
  (crimson `#b22222`, text `#2d2d2d`).
- **No JavaScript**, including the interactive parts. The research-page topic
  filter is a native `<select>` read by CSS via `:has(option:checked)`; the
  show/hide abstracts are the native `<details>` element.
- **No Google Fonts hotlink** — it sends visitor IPs to Google, which is a
  GDPR problem in Germany. Using a system font stack. To match Riaz's Lato
  exactly, self-host the `.woff2` in `fonts/` and add an `@font-face` block.

### Research page: structure and the topic filter

Papers are grouped under two headings: **Peer-reviewed publications** (with
forthcoming pieces at the top, then reverse-chronological) and **Working
papers**. A dropdown at the top filters by topic *within both groups at once*.

The dropdown is a plain `<select>`. CSS reads it directly — `option:checked`
tracks the current selection and `:has()` lets the wrapping `.filter` div
react, so `.filter:has(option[value="housing"]:checked) ~ .pubs` styles the
list. No JavaScript. A browser without `:has()` support just shows every
paper, which is a safe failure.

Each paper `<li>` carries a CSS class for **every** topic it belongs to, so
assignment is not mutually exclusive and nothing is duplicated. Topics so far:
`electoral`, `conjoint`, `housing`, `federalism`. Adding a topic means: a new
`<option>` in `research.qmd`, the name added to the `@each` list in
`styles.scss`, and the class on the relevant papers.

Paper numbering is a CSS counter, so it renumbers itself when filtered. Each
section numbers independently, starting at 1. Sections with no matching paper
hide themselves via `:not(:has(...))`.

## Domain

`asyamagazinnik.com` is registered through Squarespace. Plan is either to
transfer the registration to a cheaper registrar (Cloudflare Registrar,
Porkbun, Namecheap) or leave it registered there and just repoint DNS.

Sequence matters: build and verify the new site at a temporary URL **first**,
then flip DNS, then cancel the Squarespace subscription. Before any transfer,
record the existing DNS records (especially MX, if there's email on the
domain) and disable DNSSEC.

## Working preferences

- Show me the plan before making sweeping changes.
- Keep the CSS legible and commented — I want to be able to tweak it myself.
- Don't add analytics, cookie banners, or JavaScript unless I ask.
