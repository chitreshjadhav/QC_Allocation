# QC Allocation

A single-page tool for sampling production reviews and allocating them across a QC team.

Add `.xlsx` / `.xlsm` workbooks, set a QC percentage, and generate a random allocation
that is split as evenly as possible across your reviewers. Results export to Excel with
reviewer dropdown validation, a frozen header and filters.

## How it works

`index.html` is entirely self-contained — JSZip is inlined, there are **no external
requests of any kind**, and no build step. Open it directly from disk or serve it as a
static file; both work identically.

Workbooks are parsed in the browser. Nothing is uploaded, and there is no server
component. Reviewer names and the chosen percentage are kept in `localStorage`, so they
are per-browser and never leave the device.

## Two tabs

**QC Allocation** samples production reviews and splits them across a QC team. Everything
below about eligibility, sampling and the two downloads applies to this tab.

**Content Review** is separate and much simpler: add any number of AI Content
Review workbooks and it stacks their **SERP** sheet into one file. Only the sheet named
SERP is read — Summary, Description and Validation are ignored — and every column is
kept. Columns are matched across workbooks by header name, so a column missing from one
file comes through blank for its rows rather than shifting the data. A workbook with no
SERP sheet is reported and skipped rather than failing the batch. The download is named
`Collated_<date>.xlsx`.

That tab also does its own QC allocation. Set a percentage, generate, and the sampled
rows download with your columns intact plus five appended at the end:

| Column | Contents |
|---|---|
| QC Name | the assigned reviewer, from the random allocation |
| QC Date | the date the file was created |
| Pass/Fail | dropdown: Pass or Fail |
| Criteria | dropdown: Knowledge Gap, Missed to Update or Oversight Error |
| QC Comment | blank free text |

The export keeps **every** collated row, not just the sampled ones. QC Name and QC Date
are filled in on the sampled rows and left blank elsewhere, so one file carries the whole
batch and shows what was picked for review. The Pass/Fail and Criteria dropdowns cover
every row, so an extra row can still be marked up by hand.

Sampling reuses the QC Allocation tab exactly: the same rounding up, the same draw
without replacement, and the same even split across reviewers. Reviewer names come from
Settings, shared by both tabs. The file is named Content_Review_QC_Allocation_<date>.xlsx.

Blank rows are skipped: these workbooks carry formatted-but-empty rows well past the last
real record, so a 381-row sheet with 100 filled rows contributes 100.

## Two downloads

Both keep the columns of your source workbooks rather than a fixed subset.

- **Download Excel** — the QC allocation. Every source column in its original order, up
  to and including **QC Comments**, with the **QC Name** column filled in from the
  allocation and carrying dropdown validation against your reviewer list. Columns that
  sit after QC Comments in the source are left out. If a workbook has no QC Name or QC
  Comments column, they are added at the end.
- **Download collated** — the added workbooks stacked into one sheet, columns and all.
  Every scanned row is included, whether or not it was eligible or selected. Nothing is
  added and nothing is dropped: it is the source files concatenated.

Where workbooks have different columns, the collated sheet is the union of them, matched
by header name, with blanks where a file does not have a column.

Dates and times are written as real Excel values in both, in whichever column they
appear — recognised by each cell's number format, not by column name or position:

| Kind | Written as | Shown as |
|---|---|---|
| Date | Excel date | `dd-mmm-yyyy` |
| Date **and** time — start and end times | Excel date-time | `mm-dd-yyyy hh:mm:ss` |
| Elapsed time — total time | Excel duration | `hh:mm:ss` |

Because they are real values rather than text, they sort and subtract correctly in Excel.
Totals use an elapsed format, so a duration beyond 24 hours reads `30:00:00` rather than
wrapping to `06:00:00`.

## Header formatting

The header row in both downloads keeps the formatting of your source workbooks — font,
fill, border, alignment and row height are copied across, so the output looks like the
files you put in rather than being restyled.

Where workbooks disagree, the first one to define a column sets its look. Columns the tool
adds — QC Name or QC Comments, when a workbook has neither — inherit the neighbouring
header style so the row stays consistent. If a workbook carries no formatting at all, the
built-in style is used. The Settings sheet is the tool's own, so it always uses the
built-in style.

Collating needs only that workbooks have been added — no allocation required. The
on-screen preview still shows the key columns so the table stays readable; the full
column set is in the downloads.

## Allocation rules

- A row is eligible only if it has a non-blank **Identifier** and **Review Type =
  Production**. Matching ignores case and surrounding whitespace; Identifier `0` counts
  as a value.
- The percentage applies to the **combined** eligible production across all imported
  files, not per person or workbook.
- Selected rows = eligible × percentage ÷ 100, **rounded up**. 150 eligible at 10%
  selects 15; at 25%, 38.
- Sampling is without replacement, using `crypto.getRandomValues`. **Shuffle again**
  draws a fresh sample and redistributes it.
- Percentages from 0.01 to 100 with up to two decimal places.

## Deploying

No build, so any static host works: GitHub Pages (Settings → Pages → deploy from branch,
root), Vercel, Cloudflare Pages, Netlify, or an internal web server.

Note the tool has **no authentication** — anyone who can reach the URL can use it. That
is fine for the tool itself (it holds no data), but consider where you host it.

## Making changes

`index.html` is the source of truth — edit it directly. There is no build step and no
generator; what is in the repo is exactly what is served.

Before committing, a local pre-commit hook verifies the file: that it carries no
production data, that it still makes no external requests, that every feature is intact,
and that the inline scripts parse. It fails closed, so a commit is blocked rather than
waved through if the check cannot run. Bypass deliberately with `git commit --no-verify`.

Push to `main` and GitHub Pages redeploys within a minute or two.

## ⚠️ This repository is public

The tool itself holds no data — workbooks are parsed in the browser and never uploaded —
but anything committed here is world-readable and permanent in git history.

The earlier offline build of this tool shipped with a real batch of records baked into
it, so it must never be added to this or any repository. `.gitignore` blocks it by name
along with every `.xlsx`, `.xlsm`, `.xls` and `.xlsb`, and the pre-commit check refuses
any commit that reintroduces real records.
