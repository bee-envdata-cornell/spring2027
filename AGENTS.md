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
- **Julia 1.12.5.** `_quarto.yml` and `syllabus.qmd` both set `exeflags: ["+1.12.5"]`. The rule is
  **one version across the repo** — 4750 has a split between its project default (`1.12`) and its
  individual files (`1.11.5`, which wins); do not reproduce that here. A pinned patch is used rather
  than the `1.12` channel because 1.12.5 is what is installed; `julia +1.12` is not a resolvable
  channel on this machine without `juliaup add 1.12`.
- **`Manifest.toml` is committed**, pinned to `julia_version = "1.12.5"`. Regenerate it rather than
  editing it, and commit the result — without it renders are not reproducible across machines.
- **One Julia environment.** A single root `Project.toml`. 4750's multi-environment layout
  (`slides/`, `tutorials/` with their own) does not apply here *yet* — if decks grow heavy
  dependencies, add `slides/Project.toml` deliberately and record it here.
- **No** Makefile, CI workflows, linters, or test framework.

## Julia version — moved to 1.12.5

Spring 2026 and every sibling environment ran **1.11.5**; this offering runs **1.12.5**, decided
before any deck or assignment content was written, which is the cheapest moment to move.

**Verified before committing to it.** Both heavy environments resolve cleanly under 1.12.5, tested in
scratch copies so the real ones were untouched:

| Environment | Packages | `Pkg.resolve()` under 1.12.5 |
|---|---|---|
| `spring2026/slides/Project.toml` | 39 — Turing, Distributions, DifferentialEquations, Plots, StatsPlots, MCMCChains, Extremes, GLM, HiddenMarkovModels, Surrogates, … | exit 0 |
| `spring2026/tutorials/Project.toml` | 16 | exit 0 |
| This repo's root | 9 | exit 0, full `quarto render` clean |

**Where the environments actually are** — this differs from 4750 and an earlier draft of this file got
it wrong:

- The sibling `~/Teaching/BEE4850/slides/` directory is **an empty, stale environment** — `[deps]` is
  empty and its Manifest is from Julia 1.10.4. It is not the deck environment. Do not use it.
- The real heavy environments live **inside the site repo**, at `slides/Project.toml` and
  `tutorials/Project.toml`. This repo has neither yet; create `slides/Project.toml` when decks are
  written, seeded from `spring2026/slides/Project.toml`.
- Each `hw/hw*`, `solutions/hw*`, and `quiz/quiz*` repo has its own environment, all still on 1.11.5.
  **They are not migrated.** Move each when its `Spring27` branch is created, not before.

## Do not run `julia --project` without `--startup-file=no`

`~/.julia/config/startup.jl` runs `Pkg.add` for Revise, OhMyREPL, and BenchmarkTools whenever they are
not already resolvable — **which writes them into whichever project is active.** A plain
`julia --project=. -e 'using Pkg; Pkg.instantiate()'` in this repo added all three to `Project.toml`
on the first 1.12 resolve.

Always pass `--startup-file=no` for project operations:

```bash
julia +1.12.5 --startup-file=no --project=. -e 'using Pkg; Pkg.instantiate()'
```

Spring 2026's `Project.toml` is clean, so this has not bitten before; check `git diff Project.toml`
after any Pkg operation.

## Deployment — Cloudflare, from this repository

[instructor] **The site is served via Cloudflare directly from the git repository.**

**The consequence, and it is not optional: `_site/` must be committed.** Cloudflare serves the built
output as it stands in the repo; it does not run Quarto. A push that updates sources without
rebuilding publishes stale pages, and a repo that ignores `_site/` publishes nothing.

So the working sequence for any content change is:

```bash
quarto render          # exits 0, and check the log rather than a pipeline's status
git add -A             # includes _site/ and _freeze/
git commit && git push # this is the deploy
```

- **`_site/` is tracked because the deploy needs it.** Spring 2025 and Spring 2026 both commit it.
- **`_freeze/` is tracked for a different reason** — [instructor] it carries computed output between
  **laptop and desktop**, so the second machine does not re-run the Julia in every deck. It is a
  convenience, not a deploy requirement: a stale or missing `_freeze/` costs render time, not a broken
  site. Do not treat a `_freeze/` conflict as urgent the way a stale `_site/` is.
- `.quarto/` is the only true local artifact and stays ignored.

> **Both machines need the same Julia.** `_freeze/` only gets reused when the version matches, so the
> move to **1.12.5** has to happen on the laptop *and* the desktop — `juliaup add 1.12.5` on whichever
> has not got it. A machine still on 1.11.5 will silently re-run everything and rewrite the caches.
- There is **no** GitHub Actions workflow, no `gh-pages` branch, and no `CNAME`. 4750 uses a `CNAME`
  and `gh-pages`; that pattern does not apply here.

> **`envdata.viveks.me` returns HTTP 200 for paths that do not exist** — `/spring2099/` answers 200
> with the site-wide index. A 200 is therefore **not** evidence that an offering is deployed. Check
> for content unique to the offering, not for a status code.

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
