# CLAUDE.md

**Read [AGENTS.md](AGENTS.md) before working in this repository.** It is the full guide: what lives
where, branch conventions, the toolchain, Typst rendering, known traps, and what is stale or
unbuilt. This file exists so that guide gets loaded.

Four things worth knowing before the first edit:

1. **This repo is the website, not the course.** Homework, solutions, and quizzes are separate
   repositories checked out as siblings under `~/Teaching/BEE4850/`. A directory named `hw/` here
   does not contain the homework. Confirm the repository before editing an assignment.

2. **Those repos store a year per branch** (`Spring25`, `Spring26`, `Spring27`). Create the new year's
   branch from the previous one rather than editing in place. `hw01` and `hw02` already have
   `Spring27` on GitHub while every local checkout is still on `Spring26` — run `git branch -a` and
   fetch rather than assuming.

3. **Rendering is not verification.** Invalid markup routinely renders without erroring. Render, then
   look at the output.

4. **This course and BEE 4750 cross-contaminate.** Each site gets built from the other, and material
   leaks both ways — Spring 2026's syllabus has 4750's URL in its `pdf-note`, and 4750's
   `data/schedule.csv` contains this course's schedule. After copying anything from either course,
   grep the result for the other's course number, URL, and term.

## Writing code for slides and assignments

**Students read this code.** Assume no prior Julia and a modest programming background. Clever is
worse than clear, every time.

- **Plain, descriptive names.** `annual_maxima`, not `am`. `n_samples`, not `N`. Use standard symbols
  in prose and mathematics where they are the convention (ξ for a GEV shape parameter); use words in
  code. No single letters beyond loop indices, no Unicode subscripts, no abbreviations a reader has
  to decode.
- **Explicit loops over clever constructs.** A plain `for` loop with an `if` inside is the target, not
  a comprehension trick or a returned closure.
- **Structure the code like the model.** If the prose says "simulate 200 datasets from the fit and
  refit each," the code should loop over datasets and refit — not vectorise the whole thing into an
  opaque one-liner. The four-step loop should be visible in the code.
- **Never shadow Base names** (`last`, `min`, `step`, `filter`).
- **Course vocabulary.** *Specify* a model, *calibrate* it, *simulate* from it, *test* whether it
  recovers what was put in. Not "tune", not "train", not "run".

## Writing slides

**Slide titles.** Three forms — a **question** the slide answers ("Why That Distribution?", "Has It
Converged?", "What Does the Shape Parameter Mean?"), a **claim** it establishes ("OLS Maximizes the
Gaussian Likelihood", "Two Defensible Nulls Give Two p-Values", "The Bootstrap Assumes the Sample Is
the Population"), or a **plain name** for the thing shown ("The Tide Gauge Record", "A Trace That Has
Not Converged"). Two to six words, concrete and specific to this material.

Avoid the generic-explainer register: no "A Closer Look at…", "Deep Dive", "Unpacking…",
"Understanding…", "Exploring…", "The Power of…", "Key Considerations". A title should name *this*
slide's content, not a category it belongs to.

**No status symbols in tables.** No ✓, ❌, emoji, or coloured marks. Put the verdict in words or let
bolding carry it.

**No course policy in the body of a lecture.** Grading, weighting, what will be assessed, and late
policy belong on the opening or closing slide — never as an aside inside content ("this will be on
the quiz"). It interrupts the argument and dates the deck.

## Verifying that work

- **Every number in prose, a table, a caption, or a solution key must come from running the code.**
  This is the course's own standard, stated in the homework scopes: *no number goes in a solution key
  that was not computed.* A value from a scratch script is not the same value as one from the deck's
  own function.
- **Rendering is not verification** (see above).
- **Do not read `$?` from a pipeline.** `quarto render … | grep …` reports *grep's* exit status, not
  quarto's, and discards the error. Redirect to a file, check the code, then read the log.
- **Set and record a seed for anything stochastic.** Monte Carlo results, bootstrap intervals, and
  simulated datasets must reproduce exactly, or a student cannot check their answer against the key
  and a reader cannot tell a real difference from sampling noise.
- **Justify the sample size.** A Monte Carlo estimate or a bootstrap interval needs enough draws that
  the reported digits are stable. If a number moves when the seed changes, it is reported to too many
  digits.

Course logistics — assessment rules, weights, the schedule — belong in the syllabus and the policy
pages, not here.
