# Portfolio site

A [Quarto](https://quarto.org) website. Every project page is a `.qmd` file
(markdown + optional live R/Python code) — this is the same format your
coursework reports are already written in, so filling in a project page is
mostly copy-and-clean-up, not new writing.

## What's here

```
_quarto.yml         site config — nav, theme, output settings
index.qmd           homepage
machine-learning.qmd  grid of ML/DS projects (explicit file list, see inside)
business-analytics.qmd  grid of data/BA projects (same mechanism)
about.qmd            bio page
styles.css            light custom styling on top of the theme
projects/
  retail-dashboard.qmd          -> business-analytics.qmd
  hotel-booking-analysis.qmd    -> business-analytics.qmd
  citation-prediction-nlp.qmd   -> machine-learning.qmd (marked draft)
  images/             put project screenshots/charts here
files/                put PDFs (CV, full reports) here
```

Projects live in one flat `/projects` folder regardless of track — it's
`machine-learning.qmd` and `business-analytics.qmd`'s `contents:` lists that
decide which page a project shows up on. Adding a new project means editing
one of those two files as well as dropping the `.qmd` into `/projects`.

Every project `.qmd` has `<!-- TODO -->` comments marking what still needs
your input — findings, screenshots, real links. Search the repo for `TODO`
to find everything at once.

## 1. Install Quarto

Download from **https://quarto.org/docs/get-started/** (there's a
straightforward installer for Mac/Windows/Linux — no config needed).

Verify it worked:

```bash
quarto --version
```

## 2. Preview locally

From the project root:

```bash
quarto preview
```

This opens a live-reloading local preview in your browser — edit any `.qmd`
file and it updates automatically. Use this while you fill in the `TODO`s.

## 3. Add your content

For each project page:

1. Add a thumbnail image to `projects/images/` (update the `image:` field
   in the front matter to match).
2. Fill in the `<!-- TODO -->` sections with real findings and links.
3. Add full-size charts to `projects/images/` and reference them with
   `![](images/your-chart.png)`.
4. If you have the original `.Rmd`/`.ipynb` source for a project (rather
   than just a PDF export), it's worth re-basing the page on that — you get
   live, re-runnable code instead of a static snippet.
5. Drop full report PDFs into `files/` and update the "Full report (PDF)"
   links.
6. Update every `YOUR-GITHUB-USERNAME` placeholder (site-wide: `_quarto.yml`,
   `about.qmd`, and each project page) once your code repos are pushed.

## 4. Publish to GitHub Pages

This site is configured to render into `/docs` (see `output-dir: docs` in
`_quarto.yml`), which is the simplest GitHub Pages setup — no CI required.

```bash
quarto render
git add .
git commit -m "Update portfolio"
git push
```

Then, one-time setup in your GitHub repo:
**Settings → Pages → Source → Deploy from a branch → `main` / `/docs`**

Your site will be live at `https://YOUR-GITHUB-USERNAME.github.io/REPO-NAME/`
(or your repo's configured custom domain).

If you'd rather not commit rendered HTML at all, Quarto also supports a
GitHub Actions-based publish flow (`quarto publish gh-pages`) — more setup,
cleaner repo. Not necessary to start; switch to it later if it bothers you.

## Adding a new project later

Your projects won't all look the same — some are a report with code, some
are just a write-up, some are a slide deck, some already exist as a
`.qmd`/`.Rmd` file you don't want to retype. `/_templates` has a starting
point for each:

```
_templates/
  README.md                       — which template to use, and the category
                                     tag taxonomy (keep tags consistent!)
  template-report-with-code.qmd   — write-up + real code (default choice)
  template-report-only.qmd        — write-up, no code section
  template-presentation.qmd       — embeds a PDF slide deck
  bringing-in-existing-qmd.md     — guide for reusing an original .qmd/.Rmd
                                     file directly instead of retyping it
```

Copy the relevant template into `/projects/your-project-slug.qmd`, fill in
the front matter and `<!-- TODO -->` sections, add a thumbnail to
`/projects/images/`. It appears on the Projects page automatically —
nothing else to wire up. Read `_templates/README.md` first if this is your
first new project since the category tags are what drive the filter chips
on `/projects`, and they only stay useful if you reuse the same ones.
