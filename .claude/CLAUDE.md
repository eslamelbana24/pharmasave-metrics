# Pharmasave Escarpment Metrics: notes for Claude

The owner's metrics app for an independent pharmacy. Read this before changing anything.

## Ground rules
- **This repo is public** (GitHub Pages needs it). Never commit bank or Kroll data, PDFs, CSV exports, screenshots of the app's data, dollar figures from the books, account numbers, or staff names and emails. Real financial data stays on the owner's machine.
- Commit only what the task changed, normally `index.html`. End commit messages with the Co-Authored-By trailer the harness gives you.
- Pushing to `main` deploys. GitHub Pages serves `index.html` at metrics.escarpmentpharmasave.ca (see `CNAME`) about 60 seconds later. Confirm a deploy by fetching the site and grepping for a string you added.
- Never enter passwords or sign in for the owner. Never use another user's credentials.

## How the owner likes to work
- The owner is a pharmacist and business owner, not an accountant. Use plain language and explain what each number means.
- For anything bigger than a fix, the owner likes multiple-choice questions plus your suggestions before you build.
- Do exactly the scope asked. For example, "retire the Forecast page" meant only Forecast, not the Weekly or Monthly pages.
- Check math independently and render the result before saying something works. Report problems plainly.

## Architecture
- One file: `index.html`, vanilla JS with no build step. It uses Firebase compat 10.8.0 (Auth + Firestore), Chart.js 4.4.0 and Google Fonts. pdf.js 3.11.174 loads lazily from cdnjs on the Bank page.
- Pages come from `PAGE_MAP`: Dashboard, Projections (the landing page, `LANDING_PAGE`), EHC, Insights, Weekly Metrics, Monthly Data, Expenses, Health, Bank and Settings. Each page renders into `#page-<id>` through `renderPage()`.
- Access: each user has a `users/{email}` doc with a `pages` array. `pageAllowed(p)` checks `p.needs || p.name`, so Health opens for whoever has Bank. `GRANTABLE_PAGES()` leaves out `needs` pages. Bank is owner-only by default and isn't in `ALL_PAGES`.
- Firestore:
  - `pharmasave` (`FS_COL`) holds months, settings (including the shared `settings.growthModel`), `projectionPlans`, weekly data and targets, and `revokedUsers`.
  - Users are in `users` and logins in `loginActivity`.
  - `bank` holds `td-<from>` statements, `kroll-<from>_<to>` reports, and `config.overrides` (manual categories keyed `<statementId>#<index>`). Its security rule allows only users whose `pages` include `'Bank'`; the rule text is in `BK_RULE`.
- `calcMonthData(m)` is the profit definition used everywhere: total GP (Rx GP + front GP + rebate) minus expenses.
- The growth engine (`growthRows`, `growthTrend`, `growthReach`, `growthDescribe`) is shared by Dashboard, Projections and Insights. It has three methods ($ net, scripts, %) and a 3/6/12/All window, default 3. It's one shared setting in `settings.growthModel`.

## Bank & Collections page
The module sits between `// PAGE: BANK & COLLECTIONS` and `// PAGE: HEALTH`.
- **Uploads:** TD business-account statement PDFs and Kroll "Plan Billing Summary" PDFs are parsed in the browser. Only the extracted numbers are saved.
- **TD parser:** columns by x position (description <197, debit 197–299, credit 299–401, date 401–433, balance ≥433). "OD" means negative. Each page's Credits/Debits counts and totals must match, or the statement isn't saved.
- **Categories:** set in `bkCategorize`, then two passes in `bkBuildModel`:
  - An "Express Scripts" credit on a Tuesday, or the smaller of two that day, is NIHB.
  - An un-coded e-transfer equal to a `SALA` payment within 3 days is payroll.
- **Payer schedules (`BK_PAYERS`):**
  - ODB: claims from the 23rd to the 8th are paid at month end; the 9th to the 22nd are paid mid next month (ODP Reference Manual §11.4).
  - TELUS: twice a month, keeping $0.10 per claim. It covers Assure and PSHCP.
  - Express Scripts (ESI): weekly on Wednesdays. NIHB: every second Tuesday.
  - Green Shield and ClaimSecure: every second Tuesday.
  - Medavie and IFHP: every second Wednesday.
  - MDM and NexgenRx: about monthly.
  - Other lag days were fitted to this account's deposits.
- **Model:** `bkModel()` is cached by a signature. `bkBuildModel()` returns per-payer figures (billed, received, owed, `expectedOwed`, fees, unexplained, `lagPart`, status, `paidThrough`, `next`) and per-month figures (`otherIn`, `partial`, `settledTo`, `operating`, and so on).
- **Day counts use calendar dates, never `Date` subtraction.** A DST bug came from exactly that.
- **Overdraft days** are calendar days that ended below zero, carrying the balance over days with no transactions.
- `bkExpenseSummary()` feeds the Dashboard's Monthly Expenses tile: TD-paid vs entered.

## Health page (`renderHealth`)
- Profit comes from Monthly Data and cash from `bkModel()`, over the last 12 months that have both.
- It has a scorecard, SVG charts (`hlCols`, `hlLines`) and a profit-to-cash waterfall (`hlWaterfall`). The waterfall always closes: what's unexplained shows as "Other differences".
- Chart colours are a validated palette (`#2a78d6`, `#eb6834`, `#1baf7a`; blue/red diverging). Light-mode aqua needs visible labels.

## Owner decisions to respect
- Monthly Data expenses are the full cost, including costs paid from accounts other than TD. Monthly Data is the profit view; TD is the cash view.
- Kroll "Cash" plus front-store sales minus what's banked is shown as "patient-pay not banked". Don't speculate about its causes in anything committed here.
- Transfers in from the partner account, e-transfers in of $5,000 or more, and Pharmasave credits of $10,000 or more are partner/owner capital.
- Transfers out to the partner account (`TFR-TO <digits>`) are owners' pay and are already counted in Monthly Data's expenses. They're the `owner-pay` category (opex), not partner money. A real repayment can be re-filed as `capital-out` by hand.

## Testing (nothing here touches real data)
- **Syntax:** extract the inline `<script>` from `index.html` and run `node --check` on it.
- **Headless Chrome via CDP** (Node's built-in `fetch`/`WebSocket`, a fresh `--user-data-dir`):
  - Serve the repo with `python3 -m http.server 8731 --bind 127.0.0.1`.
  - In the page, set `userRecord`, `months` and `settings`.
  - Replace `db.collection` for `FS_BANK` with fixture docs, call `navigate('bank' | 'health' | 'dashboard')`, then read the DOM and take screenshots at 1300px and 390px.
- **Node:** extract the module text between the `// PAGE:` markers and `eval` it with stubs for `document` and `db`.
- The owner's real test data lives only on their machine. In the cloud, make **synthetic** fixtures in a scratch folder. Never commit fixtures or outputs.
