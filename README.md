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

## ⚠️ Never commit the offline build

`QC_Allocation_Tool.html` from the original project hardcodes real production data — the
07 Sep batch, 150 ticket identifiers and 7 named colleagues. It is in `.gitignore`, and it
must not be added to this or any other repository. `build-web.js` regenerates the safe
version from it and fails rather than emitting a file that still contains real records.
