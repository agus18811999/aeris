# AERIS Corporate Finance — Setup & Architecture

AERIS is an original **corporate finance management starter application** following Finora's interaction pattern, not a reuse of Finora data. Responsive pale-blue interface, Firebase Auth + Google Sign-In, verified email, Apps Script backend, Google Sheets separate by organization.

## Project files

- `index.html`: login / registration / forgot password / Google Sign-In
- `main.html`: premium multi-module corporate finance dashboard
- `verify.html`: email verification and reset password handling
- `Code.gs`: **single Google Apps Script backend**
- `config.js`: public Firebase web settings and `/exec` URL; edit once

## Setup (in order)

1. Create a **new Firebase project**, enable **Authentication → Email/Password** and **Google** sign-in methods. In Authentication → Settings → Authorized domains, add your Vercel domain and `localhost` if you intend to test there. Obtain Firebase Web configuration (`apiKey`, `authDomain`, `projectId`, `appId`) and paste into `config.js`.
2. Create a **new standalone Google Apps Script project**, paste the entire `Code.gs` contents. Under Project Settings → Script Properties set:
   - `FIREBASE_WEB_API_KEY` = the same Firebase project **Web API key** (public identifier, but keep config in Script Properties for GAS)
   - `ALLOWED_ORIGINS` = e.g. `https://your-domain.vercel.app,http://127.0.0.1:5500,http://localhost:5500` (comma separated; exact scheme+host+port; without trailing slash)
   - `MASTER_SHEET_ID` = **leave empty before first setup**, or set existing master spreadsheet ID if migrating.
3. From the Apps Script editor, run `aerisSetup()` **once** to create the master registry and grant SpreadsheetApp access. Note the master spreadsheet URL in logs; it should contain `Organizations`, `Memberships`, and `Invites` sheets. It creates a master spreadsheet under the Apps Script owner's Google Drive.
4. **Deploy → New deployment → Web app**, Execute as **Me**, Who has access **Anyone**. Copy `/exec` URL into `config.js` → `appsScriptUrl`. This public endpoint does **not** grant anonymous finance access: each action verifies a Firebase ID token and checks organization membership on the server.
5. Upload `index.html`, `main.html`, `verify.html`, and `config.js` to a **new GitHub repo** (do not mix with Finora). Connect it to Vercel for static deployment. Check Google Sign-In authorized domains.
6. Register a user, verify their email, sign in, **create company workspace**, create a bank account, then create transactions. Invite other members using **Team & Access**; the emailed invite recipient manually receives the one-time code from the owner. Only the email matching the invitation may join. No automated invitation email in this version.
7. Optional recurring automation: run `aerisInstallDailyTrigger()` **once** from the Apps Script editor. Google account owner authorizes triggers. Daily sweep checks recurring rules in **every organization spreadsheet**, with reference-based idempotency. It may run within the service's daily quotas.
8. For backend changes after first deployment: **Manage deployments → Edit → New version → Deploy**. `main.html` and `config.js` deploy via Vercel normally.

## Features (functional initial version)

- Organization onboarding, switching, verified account, membership by invite codes, Owner / Editor / Viewer permissions.
- Financial dashboard: bank balances and actual cash ledger, six-month inflow/outflow, open accounts receivable/payable.
- CRUD: accounts, business cash transactions, invoices, bills, budgets, departments, customers/vendors, recurring rules, approvals, payroll planning, tax obligations.
- Invoice & bill **Record Payment**: creates a linked cash transaction and marks document paid in one backend action. Linked cash transactions cannot be manually edited or deleted. This is operational cash bookkeeping, not full double-entry accounting.
- Activity audit log, 150 newest actions, cashflow, CSV export, responsive modals and mobile navigation.
- Lazy-load secondary tables; readonly viewer access. No seeded or fabricated business metrics.

## Important limitations / next iteration

- **Not a production-ready ERP or certified accounting system**. IDR-only calculations at this stage (multi-currency consolidation requires FX-rate handling). No double-entry general ledger, journal posting engine, chart of accounts, accrual accounting, bank feed, automated payment gateway, statutory tax calculation, invoice PDF generation, close periods, or accrual statements.
- Payroll, approval status, and tax tracker are informational and don't automatically debit cash or enforce approval gating.
- Company membership is checked server-side on each request; Apps Script runs as its owner. This means the Google Apps Script owner can access company spreadsheets (not client-side end-to-end encrypted).
- For **real companies**, implement additional audit/approval policy, validation rules, rate limiting, backups, accounting review, retention, and penetration testing before handling sensitive production records.
- The Apps Script master includes emails, membership roles and organization spreadsheet IDs. Never publish it. Each company sheet is owned by the Apps Script account, not shared automatically with finance users.
- Google Sheets/Apps Script quotas and 45-second browser RPC timeout may affect scale; migrate to a managed database + secure server-side auth for high-volume enterprise usage.
- The Firebase Web API key in public `config.js` is intentionally a web SDK identifier, **not** a secret service-account credential. Do not embed service account secrets or Script Properties in client files.

## Troubleshooting

- **Origin not allowed / timeout**: inspect `ALLOWED_ORIGINS` exact Vercel domain (or localhost port) and new `/exec` deployment; Apps Script must be accessible to Anyone.
- **Firebase unauthorized domain**: add your site origin's hostname to Firebase Auth authorized domains.
- **Email link not working**: ensure Firebase Action URL and authorized domains are configured. `verify.html` handles direct `mode=verifyEmail`/`resetPassword` links when available and otherwise rechecks authenticated account.
- **Not authorized**: backend requires a verified Firebase ID token and membership in a company. Never use a spreadsheet ID in the browser to determine tenant access.
- **No data yet**: create a business account first. A new company's sheets start empty.

## Proposed next milestone

Add immutable accounting journals, double-entry GL and chart of accounts, period close/reversal workflow, finance approval gates, document attachments, audited balance sheet / P&L, multi-currency FX treatment and fiscal tax configuration. These require accounting-specific design choices, not just Finora field renaming.
