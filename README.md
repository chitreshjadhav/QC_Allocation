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

## Two downloads

- **Download Excel** — the QC allocation: the sampled rows with a QC Name column,
  reviewer dropdown validation, a frozen header and filters.
- **Download collated** — every workbook you added, merged into one sheet. This includes
  rows that were *not* selected for QC and rows that were filtered out entirely, so it
  doubles as an audit trail of what the tool saw. Columns are the six source fields plus
  **Review Type**, **Status** (`Eligible`, `Excluded: no Identifier`, or
  `Excluded: Review Type is not Production`) and the originating **Source File**,
  **Sheet** and **Row**, so any line can be traced back to the workbook it came from.

Collating needs only that workbooks have been added — no allocation required.

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
