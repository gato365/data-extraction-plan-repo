# README for QMD Extraction Scripts

This repository contains three Quarto Markdown (`.qmd`) extraction scripts
that accompany the Data Extraction Plan (DEP) case studies reported in the
paper. The scripts are written as annotated, executable supplements: each one
walks through the source, access decision, extraction logic, validation checks,
scope rules, and saved reproducibility artifacts for one case study.

The three scripts are:

- `dep_case1_wikipedia.qmd`: Wikipedia U.S. historical population tables.
- `dep_case2_bballref.qmd`: Basketball Reference NBA player statistics.
- `dep_case3_spotify.qmd`: Spotify audio features via the Spotify Web API,
  preserved as a historical 2021 workflow.

Each QMD follows the matching DEP case-study plan. The section numbers and
chunk comments map the code back to the DEP sections, so reviewers can see how
the source choice, access rules, scope decisions, validation checks, and
contingency plans are implemented in code. The comments inside the QMD files
explain what each code block does and how that block connects to the plan.

## Summary of Reported Case-Study Results

The paper uses these scripts to demonstrate how a written extraction plan can
be translated into a reproducible extraction workflow.

| Case | DEP decision | Main reproducibility result |
| --- | --- | --- |
| Case 1: Wikipedia | Proceed | A single public Wikipedia table is extracted, spot-checked against pre-specified Census values, flagged by state/territory status, and saved in a dated run folder with a CSV and raw HTML snapshot. |
| Case 2: Basketball Reference | Proceed with conditions | NBA per-game and advanced tables are collected for seasons 2001-2025 with rate limiting, dated run folders, raw HTML snapshots, validation checks, and exclusion flags for later regression use. |
| Case 3: Spotify | Proceed in the original 2021 context | The QMD documents the historical Spotify workflow and shows that the DEP's contingency scenarios later materialized; new Spotify developer accounts should not expect this extraction to run today. |

The stored completed run in this repository includes:

- `data/raw/wikipedia/2026-06-24/wikipedia_population.csv`
- `data/raw/wikipedia/2026-06-24/wikipedia_population_raw.html`
- `data/raw/bball_ref/2026-06-24/csv/per_game_all.csv`
- `data/raw/bball_ref/2026-06-24/csv/advanced_all.csv`
- `data/raw/bball_ref/2026-06-24/html/`, containing 50 Basketball
  Reference HTML snapshots, one per season and table type.

For the stored `2026-06-24` run, the Wikipedia output has 57 rows and 9
columns, with 50 rows flagged as states and 7 rows flagged as territories or
other non-state rows. The Basketball Reference outputs contain 15,397 rows in
each CSV; the per-game file has 34 columns and the advanced file has 30
columns. The Basketball Reference validation spot-check finds Stephen Curry's
2015-16 `3PA` value as `11.2`, and 3,498 per-game rows are flagged for
exclusion from later regression because the player appeared in fewer than 20
games.

## Required Software

Install the following before reproducing the scripts.

| Software or package | Used by | Purpose |
| --- | --- | --- |
| R | All cases | Executes the extraction code. |
| Quarto CLI | All cases | Renders `.qmd` files to browser-viewable HTML. |
| `knitr` | All cases | Extracts or executes R code chunks from QMD files. |
| `rvest` | Cases 1 and 2 | Reads HTML pages and parses HTML tables. |
| `xml2` | Cases 1 and 2 | Saves raw HTML snapshots as reproducibility anchors. |
| `dplyr` | All cases | Filters, mutates, counts, validates, and prepares extracted data. |
| `stringr` | Case 1 | Cleans population values before spot-check validation. |
| `purrr` | Cases 2 and 3 | Iterates over seasons, table types, artists, and genres. |
| `spotifyr` | Case 3 only | Historical wrapper for Spotify API access; no longer available from CRAN in the current API context. |
| `remotes` | Case 3 only | Installs the archived `spotifyr` version if reproducing the historical workflow. |

For Cases 1 and 2, install the R dependencies with:

```sh
Rscript -e 'install.packages(c("knitr", "rvest", "xml2", "dplyr", "stringr", "purrr"))'
```

For Case 3, the QMD documents a historical workflow. If reproducing the
original 2021-era workflow from an archived environment, install the pinned
package version using the method noted in the QMD, for example:

```sh
Rscript -e 'install.packages("remotes"); remotes::install_github("charlie86/spotifyr@v2.2.4")'
```

## Viewing the QMD Scripts as HTML

The QMD files are configured with:

```yaml
execute:
  eval: false
```

This means Quarto renders the annotated code and explanation without executing
the web-scraping or API calls. This is intentional for the supplementary HTML
files because reviewers can inspect the workflow safely without triggering
network requests.

Render the browser-viewable HTML files from the repository root:

```sh
quarto render dep_case1_wikipedia.qmd
quarto render dep_case2_bballref.qmd
quarto render dep_case3_spotify.qmd
```

The corresponding HTML files can then be opened in any web browser:

- `dep_case1_wikipedia.html`
- `dep_case2_bballref.html`
- `dep_case3_spotify.html`

## Reproducing the Extraction Results

Run all commands from the repository root. The scripts do not require
command-line arguments. Configuration settings are defined near the top of each
QMD as named constants, such as `WIKI_URL`, `TARGET_INDEX`, `SEASONS`,
`DELAY_SEC`, `RUN_DATE`, `RUN_DIR`, `RAW_DIR`, `REPLACE_EXISTING_DATA`, and the
Spotify credential environment variables.

Each extraction script now writes into a date-stamped run directory:

- `data/raw/wikipedia/YYYY-MM-DD/`
- `data/raw/bball_ref/YYYY-MM-DD/`
- `data/raw/spotify/YYYY-MM-DD/`

By default, `REPLACE_EXISTING_DATA <- TRUE`. If a reviewer reruns the same case
on the same date, the script removes that date's existing run folder and writes
a fresh copy. This prevents the same source data from being stored multiple
times in different files inside the same date folder. If a reviewer wants to
keep an earlier same-date run, change `REPLACE_EXISTING_DATA` to `FALSE` before
running the script.

Because the QMDs use `eval: false` for safe HTML rendering, executing the
workflow requires either running the chunks interactively in RStudio or
extracting the R code with `knitr::purl()` and sourcing it.

### Case 1: Wikipedia

Purpose: extract the Wikipedia table for U.S. states and territories by
historical population, save the raw HTML page, validate pre-specified values,
flag state rows, and write a dated CSV.

Run:

```sh
Rscript -e 'src <- tempfile(fileext = ".R"); knitr::purl("dep_case1_wikipedia.qmd", output = src, quiet = TRUE); source(src, echo = TRUE)'
```

Expected behavior:

- Creates `data/raw/wikipedia/YYYY-MM-DD/` for the run date.
- Replaces that date folder first when `REPLACE_EXISTING_DATA <- TRUE`.
- Saves a raw HTML snapshot named `wikipedia_population_raw.html` inside the
  date folder.
- Saves extracted data named `wikipedia_population.csv` inside the date folder.
- Prints the first three rows and selected column names for visual inspection.
- Checks the pre-specified 2020 values for California and Wyoming.
- Flags the 50 U.S. states with `is_state == TRUE`.

The date is attached to the Wikipedia directory name so reviewers can identify
which exact extraction run produced each artifact without mixing files from
multiple runs in one shared folder.

### Case 2: Basketball Reference

Purpose: extract per-game and advanced NBA player statistics from Basketball
Reference for seasons 2001-2025, respecting the DEP rate-limit plan and saving
raw HTML snapshots for each page.

Run:

```sh
Rscript -e 'src <- tempfile(fileext = ".R"); knitr::purl("dep_case2_bballref.qmd", output = src, quiet = TRUE); source(src, echo = TRUE)'
```

Expected behavior:

- Creates `data/raw/bball_ref/YYYY-MM-DD/html/` and
  `data/raw/bball_ref/YYYY-MM-DD/csv/`.
- Replaces that date folder first when `REPLACE_EXISTING_DATA <- TRUE`.
- Requests 50 pages total: 25 seasons times 2 table types.
- Waits `DELAY_SEC <- 5` seconds before each request.
- Saves raw HTML snapshots with season and table type in the file name, such as
  `per_game_2025.html` and `advanced_2025.html`.
- Saves CSV files named `per_game_all.csv` and `advanced_all.csv` inside the
  date folder.
- Validates row counts, Stephen Curry's 2015-16 `3PA` value, and column counts.
- Adds `exclude_regression` and `exclusion_reason` fields for players with
  fewer than 20 games played.

The Basketball Reference data for a run is stored together under
`data/raw/bball_ref/YYYY-MM-DD/`. The directory name records the extraction
date, and the HTML snapshot file names record the season and table type.
Keeping the dated directory intact makes it clear which raw HTML files support
the extracted Basketball Reference results.

If the site returns `HTTP 403` or becomes unavailable, follow the contingency
notes in the QMD: stop retrying, wait 24 hours, increase the delay on the next
attempt, or switch to one of the documented fallback sources.

### Case 3: Spotify

Purpose: document the historical 2021 Spotify audio-features extraction plan,
including OAuth authentication, pre-specified artist IDs, feature extraction,
exclusion rules, validation checks, and CSV/RDS outputs.

The QMD can be rendered to HTML for review:

```sh
quarto render dep_case3_spotify.qmd
```

Running the extraction itself is not expected to work with a new Spotify
developer account in the current API environment. The QMD explains that Spotify
deprecated or restricted the relevant audio-feature access after the original
2021 context. If using a historical account and archived package environment,
set credentials in `.Renviron` before execution:

```text
SPOTIFY_CLIENT_ID=your_client_id_here
SPOTIFY_CLIENT_SECRET=your_client_secret_here
```

Then run:

```sh
Rscript -e 'src <- tempfile(fileext = ".R"); knitr::purl("dep_case3_spotify.qmd", output = src, quiet = TRUE); source(src, echo = TRUE)'
```

Expected historical behavior:

- Creates `data/raw/spotify/YYYY-MM-DD/`.
- Replaces that date folder first when `REPLACE_EXISTING_DATA <- TRUE`.
- Authenticates through environment variables, never hardcoded credentials.
- Extracts artist audio features by genre.
- Applies pre-specified exclusions for popularity, missing audio features, and
  compilation-album scope.
- Validates minimum tracks per artist, bounded feature ranges, tempo range,
  popularity range, and duplicate track IDs.
- Saves `spotify_all.rds` and `spotify_all.csv` inside the date folder.

For current reproducibility review, treat this QMD as documentation of the
historical extraction procedure and of the DEP contingency plan. The fact that
the API endpoint and package availability changed is part of the case-study
result.

## Reproducibility Checklist

Use this checklist to reproduce or audit the reported outputs:

1. Confirm that R and Quarto are installed.
2. Install the listed R packages.
3. Open or render each HTML file to inspect the annotated QMD workflow.
4. Run Case 1 from the repository root and compare the new dated Wikipedia
   folder to the stored `data/raw/wikipedia/2026-06-24/` artifact.
5. Run Case 2 only when network access and site-use conditions allow; compare
   the new dated Basketball Reference folder to the stored
   `data/raw/bball_ref/2026-06-24/` artifact.
6. Review Case 3 as a historical API workflow unless an archived Spotify API
   environment is available.
7. Preserve dated run folders and raw HTML snapshots with any reproduced run so
   future readers can identify the exact source data used.
