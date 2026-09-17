# BEE 4850/5850: Environmental Data Analysis and Simulation (Spring 2027)

Course website for the Spring 2027 offering of BEE 4850/5850 at Cornell University.

Built with [Quarto](https://quarto.org/). Based on the Spring 2026 site.

## Building

```sh
quarto preview   # local preview on port 4200
quarto render    # full build to _site/
```

## Structure

- `syllabus.qmd` — full syllabus (renders to HTML, PDF via Typst, and Word)
- `schedule.qmd` — class-by-class schedule; the table itself is in `_schedule-table.qmd`
  so it can be included in both the schedule page and the syllabus PDF
- `policies/` — detailed course policies
- `_variables.yml` — course metadata (instructor, TA, meeting times, URLs)

## Status

Syllabus, schedule, and policies are drafted. Still to come: slides, practice sets,
mini projects, setup pages, and resources.
