# BEE 4850/5850 Course Website — Agent Guide

Spring 2027 offering. Adapted from the BEE 4750 guide at `~/Teaching/BEE4750/fall2026/AGENTS.md`;
everything below was verified against *this* course rather than carried over. Where 4750's guidance
does not apply, that is noted rather than silently dropped.

## Most course material lives outside this repository

This repo is the **website**. Almost everything else is a separate repository checked out as a
sibling under `~/Teaching/BEE4850/`:

| Path | Contents |
|------|----------|
| `spring2027/` | This repo — site, syllabus, policies, schedule |
| `hw/hw01`–`hw06` | Homework repos, one each |
| `solutions/hw01`–`hw05` | Solution repos |
| `quiz/quiz01`–`quiz04` | Quiz repos |
| `labs/lab01` | A single lab repo (Spring 2024-era; **the course has no labs in 2027**) |
| `slides/` | Shared Julia environment for decks — `Project.toml`/`Manifest.toml` only |
| `spring2026/` | Previous offering, full site — the scaffolding here was copied from it |
| `spring2025/`, `sp24/`, `new-web/`, `website/` | Earlier offerings |

Before editing an assignment or quiz, confirm you are in the right repository. A directory named
`hw/` inside this repo is not where homework lives.

## Branch per year — `SpringNN`

Assignment, solution, and quiz repos store each offering as a **branch**: `Spring24`, `Spring25`,
`Spring26`, `Spring27`. Create the new year's branch from the previous one rather than editing in
place.

```bash
git -C ~/Teaching/BEE4850/hw/hw01 show Spring26:hw01.qmd   # read last year's version
```

**Verified state as of this writing:**

- On GitHub, `hw01` and `hw02` already have a `Spring27` branch; `quiz01` has only `Spring26` and `main`.
- Every **local checkout** is still on `Spring26` — except `quiz/quiz03`, which is on `main`.

So the local working copies are behind the remote for at least `hw01` and `hw02`. **Run
`git branch -a` rather than assuming**, and fetch before concluding a branch does not exist.

## One GitHub organization

Everything current lives under **`bee-envdata-cornell/`** — this site is
`bee-envdata-cornell/spring2027`. Unlike 4750, there is no second organization to disambiguate.

One historical wrinkle: `vsrikrish/simulation-data-analysis` holds the **Spring 2024 and Spring 2025**
offerings as *branches* (`Spring24`, `Spring25`), which is the older pattern. It has had no commits
since June 2025. Do not add to it.

## Toolchain

- **Quarto** builds the site; Julia is the execution engine.
  - `quarto preview` — dev server on port 4200
  - `quarto render` — build to `_site/`
- **Julia 1.11.5 currently, and this is an open decision.** `_quarto.yml` and `syllabus.qmd` both set
  `exeflags: ["+1.11.5"]`. **A move to 1.12 for the Spring 2027 offering is under consideration** — see
  [Julia version](#julia-version-open) below. Whatever the answer, the rule is: **one version across
  the repo.** 4750 has a split between its project default (`1.12`) and its individual files
  (`1.11.5`, which wins); do not reproduce that here.
- **No `Manifest.toml`.** Spring 2026 has one pinned to `julia_version = "1.11.5"`; it was not copied,
  so package resolution here is unpinned. Add one once the Julia version is settled, or renders will
  not be reproducible across machines.
- **One Julia environment.** A single root `Project.toml`. 4750's multi-environment layout
  (`slides/`, `tutorials/` with their own) does not apply here *yet* — if decks grow heavy
  dependencies, add `slides/Project.toml` deliberately and record it here.
- **No** Makefile, CI workflows, linters, or test framework.

## Julia version (open) {#julia-version-open}

**Local state:** juliaup has 1.9.4, 1.10.4, 1.11.4, 1.11.5 (default), 1.11.6, and **1.12.5**
installed. The `release` channel currently resolves to 1.11.6 locally and reports **1.13.0** as
available — so 1.13 is the current release as of September 2026, and will be roughly sixteen months
old by the time Spring 2027 starts.

**The site environment is the cheap part.** The root `Project.toml` carries nine light packages (CSV,
DataFrames, DataFramesMeta, Dates, LaTeXStrings, Latexify, Preferences, PrettyTables,
QuartoNotebookRunner). Bumping it is near-zero risk.

**The expensive part is elsewhere** and must be tested before the decision is made:

- `~/Teaching/BEE4850/slides/Project.toml` — the shared deck environment, which carries the plotting
  and statistics packages
- the six `hw/hw*` repos and four `quiz/quiz*` repos, each with its own environment, all on `Spring26`

A version move is only real once those resolve and render. **Do not bump `_quarto.yml` alone** — that
produces 4750's current half-done state, where the project declares one version and every file uses
another.

## Deployment — not in this repo, and not fully known

`spring2026` has **no `.github/workflows`, no `gh-pages` branch, no `_publish.yml`, and no `CNAME`**,
yet `envdata.viveks.me/spring2026` serves behind Cloudflare. The build is therefore wired outside the
repository — most likely a Cloudflare Pages project pointed at it, but **this has not been confirmed.**

Consequence: **pushing to `main` may or may not deploy.** Do not assume a push is inert, and do not
assume it publishes. Confirm before relying on either. `site-url` here is already set to
`https://envdata.viveks.me/spring2027`.

(4750 *does* use a `CNAME` and a `gh-pages` branch. That pattern does not apply to this course.)

## Structure

- **`_quarto.yml`** — site config, nav, sidebar, formats (HTML, Typst, RevealJS, Beamer)
- **`_variables.yml`** — course number, instructor, TA, meeting pattern, room; used via `{{< var … >}}`
- **`_assets/`** — logos, Lua filters, Typst helpers, CSL
- **`_schedule-table.qmd`** — the class-by-class table, included by `schedule.qmd`. Kept separate so the
  same table can also render into the syllabus PDF
- **`schedule.qmd`** — class-by-class only; deadlines live on `homework.qmd`, `quizzes.qmd`,
  `projects.qmd`, which are the single source of truth for their dates
- **`policies/`** — the detail behind the syllabus
- **`slides/`** — empty. The schedule links `slides/lecture01.qmd`–`lecture42.qmd`; those 42 links
  currently 404

**Slide naming differs from 4750.** Here it is flat and sequential (`lecture07.qmd` = session 7).
4750 encodes week and session in the filename (`lecture06-2-…`) *and* carries a separate
`subtitle: "Lecture NN"` counter that skips labs. **This repo has one numbering system; keep it that
way.**

## Typst rendering

The syllabus renders to PDF and Word as well as HTML, through the project filter
`_assets/typst-pdf/processing.lua`.

**Which front-matter fields are actually live** — verified by reading the filter and `note.typ`:

| Field | Status |
|-------|--------|
| `pdf-title`, `pdf-subtitle` | **Live** — substituted by `processing.lua` |
| `pdf-note` | **Live** — injected by `_assets/typst-pdf/note.typ` via `{{< meta pdf-note >}}` |
| `pdf-header-left`, `pdf-header-right`, `pdf-footer-left` | **Inert.** Nothing reads them |

The three inert fields are present in `syllabus.qmd`, inherited from Spring 2026. They do nothing —
do not build on them, and do not add more.

**Rendering a file standalone, outside this project, silently loses the filter**, and the title block
falls back to the plain `title`.

## These two courses cross-contaminate — check before copying

The 4750 and 4850 sites are repeatedly built from each other, and material leaks in both directions.
Two confirmed instances:

- **Spring 2026's `syllabus.qmd` has `pdf-note` pointing at `envsys.viveks.me/fall2026`** — BEE 4750's
  Fall 2026 URL, inside BEE 4850's syllabus. Corrected here; check it survives.
- **4750's `data/schedule.csv` contains this course's content** from row 14 onward (Bootstrap, Missing
  Data, GLMs, March–May dates), per its own AGENTS.md.

When copying anything from 4750 — or from a previous 4850 offering — **grep the result for the other
course's number, URL, and term** before committing.

## Known traps

- **Callout syntax.** Use `::: {.callout-note}` — hyphenated. The space-separated
  `::: {.callout .note}` is invalid, and renders a callout titled literally "None" rather than failing.
- **`freeze: auto`** caches computed output in `_freeze/`, keyed by filename. Renaming a deck orphans
  its cache; `git mv` the matching `_freeze/` directory alongside it.
- **`date-format: long` strips leading zeros**, so grepping rendered output for a literal source date
  fails.
- **Rendering is not verification.** Invalid markup frequently renders with no error. Render, then
  look at the output.
- **Do not read `$?` from a pipeline.** `quarto render … | grep …` reports *grep's* status, not
  quarto's. Redirect to a file, check the exit code, then read the log.
- **Plots.jl SVG casing** (applies once decks exist): Quarto's HTML pass lowercases SVG attribute
  names (`viewBox` → `viewbox`, `clipPath` → `clippath`). Browsers correct for this, so the deck looks
  fine, but an extracted figure handed to a strict renderer crops to the top-left. Restore the casing
  before converting, and use `rsvg-convert` rather than `qlmanage -t`.

## Known stale or incomplete

- **`slides/` is empty** — 42 links in the schedule resolve to nothing.
- **`data/` is empty.** Spring 2026 carries a `data/schedule.csv` that disagrees with its own
  `schedule.qmd`; it was deliberately not copied. If a machine-readable schedule is wanted here,
  generate it from `_schedule-table.qmd` rather than hand-maintaining a second copy.
- **`setup/index.qmd` is a placeholder** pointing at the Spring 2026 setup pages.
- **TA and office hours are `TBD`** in `_variables.yml`; the meeting time and room are carried over
  from Spring 2026 and unconfirmed.
- **Every date is provisional** — no syllabus has been distributed.

## Course facts worth knowing before editing content

- The spine is a four-step loop: **why that distribution and link → simulate → fit → test**. Step 4 is
  the recurring verification beat and is the point of the course.
- Assessment: homework 20% (nine assignments, 20 points each), quizzes 25% (six, lowest dropped),
  mini projects 20%, term project 25%, readings 10%.
- **Homework here means short, pen-and-paper rehearsal for the quizzes**, plus one small computational
  item — *not* the two-week computational assignments the word meant in previous offerings.
- **4850 and 5850 differ in required work, not in marking severity** — see the syllabus. Do not
  introduce differentiation by length or by stricter grading.
- **Learning objectives are deliberately kept separate from ABET outcomes.** Do not map them or write
  them in accreditation vocabulary.
