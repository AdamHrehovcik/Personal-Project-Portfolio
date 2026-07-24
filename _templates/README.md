# Templates & category taxonomy

Four ways a project can show up, four templates:

| Template | Use when |
|---|---|
| `template-report-with-code.qmd` | You have a write-up **and** code worth showing. Default choice — use this whenever you can. |
| `template-report-only.qmd` | You have a write-up but no code worth showing separately (e.g. it was done in Excel/a BI tool). |
| `template-presentation.qmd` | The deliverable is a slide deck (PDF), not a written report. |
| `bringing-in-existing-qmd.md` | Not a template — a guide for reusing an original `.qmd`/`.Rmd` file directly instead of retyping from a PDF. Use this first if the source file still exists. |

## Category tags

The `categories:` field in each project's front matter drives the clickable
filter chips on `/projects`. Keep it to a **small, consistent** set or the
filters turn into noise. Suggested taxonomy — pick one from each row:

**Format** (what the deliverable is):
`Report` · `Presentation` · `Dashboard`

**Has code:**
`Code` (add this tag if there's real code attached, regardless of format)

**Context:**
`Academic` · `Personal`

**Domain/tools** (add freely, these are the ones worth searching by):
`Python` · `R` · `SQL` · `Forecasting` · `Regression` · `NLP` ·
`Data Visualisation` · `Streamlit` · `tidyverse` · etc.

Example: a personal side-project dashboard built in Python would be
`categories: [Dashboard, Code, Personal, Python, Streamlit]`.

Don't invent a new "Format" or "Context" tag per project — reuse the ones
above so the filter chips stay meaningful as the number of projects grows.

## New project checklist

1. Copy the right template into `/projects/your-project-slug.qmd`
2. Fill in front matter — title, description, date, categories, image
3. Add a thumbnail to `/projects/images/`
4. Fill in the `<!-- TODO -->` sections
5. If code exists: paste real snippets, or better, follow
   `bringing-in-existing-qmd.md` if the original source file is available
6. Add any PDFs (report/slides) to `/files`
7. `quarto preview` to check it renders and shows up on `/projects`
