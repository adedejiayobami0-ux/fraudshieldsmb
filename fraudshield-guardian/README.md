# FraudShield Guardian — local-only MVP

This is an additive prototype for the static HTML/JS `fraudshieldsmb` frontend.

## Install
1. Add `guardian.html` to the root of the GitHub repository.
2. Deploy via the existing Vercel setup. The page should be available at `/guardian` (cleanUrls enabled) or `/guardian.html`.
3. Add a navigation link to the main site and workspace: `<a href="/guardian">Open Guardian (beta)</a>`.
4. Test all functionality in a staging environment before advertising or shipping it.

## Working behavior
- Manual payable/receivable records (vendor, invoice reference, amount, due date, payment-detail status).
- Deterministic possible-duplicate detection based on same entry type, normalized party, invoice reference, and amount.
- Overdue customer receivable totals based on due dates and open review status.
- Payment-detail-change warnings based on the user's own selection, **not independent verification**.
- Local persistence, item resolution, queue filters, CSV export, clear-data control, and example records labeled fictional.
- Brief regenerated from locally entered records.

## Limits and privacy
- No Supabase synchronization, authentication, automated account analysis, bank verification, AI scoring, alerts, document upload, or actual fraud determination.
- Data is saved in browser localStorage; do **not** enter sensitive banking, personal or confidential details. Browser storage is not appropriate for production financial records.
- CSV exports can contain sensitive content and need handling accordingly.
- Before a public launch, implement authenticated per-tenant server persistence with Supabase RLS, audit trail, data retention/deletion, security review, server-side validation, and separate real integrations.

## Suggested next technical milestones
1. Supabase tables `guardian_cases`, `guardian_case_events`, `vendor_baselines` with strict user/business RLS and server-side validation.
2. Connect user-authorized invoice reads from existing QuickBooks integration; tag data provenance and last-sync timestamps.
3. Implement deterministic duplicate detection with independently tested behavior and false-positive explanations.
4. Add a read-only weekly brief and opt-in notifications only when useful new findings occur.
5. Measure first-review completion, 7-day and 30-day return rates, resolved items, and pilot conversion.
