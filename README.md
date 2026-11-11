# Data Quality Profiler

A single-page tool for checking a client data file before it is loaded. Drop in a CSV, TSV, pipe-delimited or Excel file and get:

- the share of rows that pass every check, shown as Pass, Review or Fail
- four cell-level measures: Completeness, Validity, Uniqueness and Consistency
- a list of what to fix first, with examples and a suggested action for each
- every exception with its row, column, value and reason, filterable and downloadable as CSV
- a plain-text summary you can paste into a ticket or an email

The file is read in your browser. Nothing is uploaded and the page makes no network calls with your data. There is no AI in this tool.

## Run it

Open `index.html` in a browser. There is no build step. Fonts load from Google Fonts and fall back to system fonts when offline. Excel files load a reader from cdnjs the first time one is chosen, so Excel needs an internet connection. CSV does not.

To host it, turn on GitHub Pages for this repository (Settings, Pages, deploy from the `main` branch, root folder).

The page opens on a synthetic sample with problems added on purpose, so you can see how it works before loading a real file.

## What it checks

Rules are suggested from the data when a file loads. You can switch them off, remove them or add your own on the Rules tab.

| Check | What it does |
| --- | --- |
| Required | Flags blank values and placeholders such as `N/A`, `null`, `none`, `-` |
| Unique | Flags a value that repeats an earlier one in the column |
| Is a number | Accepts plain numbers and thousands separators such as `1,200` |
| Is a valid date | Accepts `YYYY-MM-DD`, `YYYY/MM/DD`, `DD/MM/YYYY`, `MM/DD/YYYY`, `DD-Mon-YYYY` and `DD.MM.YYYY`, and rejects impossible dates |
| Within range | Minimum and maximum |
| Allowed values | A comma-separated list |
| Matches pattern | A regular expression that must match the whole value |
| Valid ISIN | Checks the shape and the check digit |
| Valid currency code | Checks against 40 common ISO 4217 codes |
| Valid email | A simple address check |

Always checked, without a rule:

- rows with the wrong number of fields, which usually means a stray delimiter
- fully duplicated rows
- extra spaces at the start or end of a value
- the same value in different cases, such as `Active` and `ACTIVE`
- dates in a minority format within one column
- numeric outliers, which are listed for review and do not change the score

## How the score works

The headline number is the share of rows that pass every check, so one bad cell makes its whole row fail. Pass is 95% or above, Review is 85% to 94.9% and Fail is below 85%.

The four bars measure cells and show where the problems are:

- **Completeness:** cells filled in columns that have a Required rule, or in every column when none is set.
- **Validity:** failed checks across all rules except Required and Unique, plus one row-shape check per row.
- **Uniqueness:** rows that repeat an earlier row or repeat a value in a Unique column.
- **Consistency:** cells with extra spaces, mixed case or a minority date format.

A cell can count in more than one measure.

## Tested and not tested

Tested in Chromium on desktop and phone width, in light and dark themes:

- parsing, delimiter detection, quoted fields, ragged rows, empty and header-only files
- the ISIN check digit against a known valid ISIN, and the date rules
- the synthetic sample, where every planted problem is found with the exact count planted
- adding, switching off and removing rules, filtering, search and CSV export, including protection against values that start with `=`, `+`, `-` or `@`

Not tested:

- Excel files (`.xlsx`, `.xls`). The reader could not be loaded in the test environment. Save as CSV if it fails.
- very large files. Up to 250,000 rows are read, but speed on a file that size has not been measured.
- Safari and Firefox.

## Limits

- Text files are read as UTF-8.
- Only the first sheet of a workbook is read, and values are read as Excel displays them.
- The currency list is 40 codes, so an unusual valid code is reported as unknown.
- Dates such as `03/04/2026` are ambiguous between day-first and month-first. The tool treats them as the same format as the unambiguous dates in the column.
- Two-digit years are not recognised as dates.

## Where to change things

All logic sits in the script in `index.html`. The rule list is `RULE_LABEL`, the suggested rules are in `suggestAll`, the scoring is at the end of `evaluate`, and the sample data is `sampleTable`.

## Data

The sample is synthetic. It contains no real investors, accounts or securities. Do not commit real client files to this repository.
