# Bringing in a project that already has a `.qmd` or `.Rmd` file

If a project was already written in Quarto or R Markdown (most of your
Imperial coursework probably was), don't retype it from the PDF — reuse the
source directly. You get live, re-executed code and real output instead of
a static transcription.

## Steps

1. Copy the original file into `/projects`, renamed to match your other
   project slugs, e.g. `projects/hotel-booking-analysis.qmd`.

2. Add (or fix) the front matter at the top so it matches the shape every
   other project page uses — this is what makes it show up correctly on
   the `/projects` grid:

   ```yaml
   ---
   title: "PROJECT TITLE"
   description: "One sentence."
   date: 2026-01-01
   categories: [Report, Code, Academic]
   image: images/PROJECT-SLUG-thumb.png
   ---
   ```

   Coursework files often have their own front matter (author, a different
   title format, etc.) — just replace it with the block above rather than
   stacking both.

3. If it's an `.Rmd` file: Quarto reads `.Rmd` natively, but renaming the
   extension to `.qmd` is worth doing for consistency with everything else
   in this repo — the syntax is close enough that little to nothing breaks.

4. Check any relative paths to data files (`read_csv("data/...")` etc.) —
   either bring the data file into the repo alongside the `.qmd`, or point
   the code at wherever it actually lives. If the dataset is large or not
   yours to redistribute, consider loading a small sample or a public
   mirror instead, and note that in the text.

5. Run `quarto preview` and check the page actually re-executes cleanly —
   coursework `.Rmd` files sometimes depend on packages or setup steps from
   class that aren't obvious until it fails to render.

## When NOT to do this

If the original analysis used a heavy/slow dataset, needs credentials, or
depends on infrastructure you don't want to reproduce for a portfolio page,
it's completely fine to fall back to `template-report-with-code.qmd` instead
— static charts + real code snippets pasted in, rather than a live rebuild.
