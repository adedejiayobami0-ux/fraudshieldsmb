# FraudShield — product-first fintech site

## What to upload
This package is an **overlay** for the existing FraudShield GitHub repository. Copy its contents into the repository **without deleting other existing files**. This preserves the current `chat.html`, `dashboard.html`, `auth.html`, `sw.js`, icons and privacy/terms pages, plus existing integrations.

- `index.html`: redesigned product-first homepage
- `verify.html`: functional, completely client-side manual payment review workflow
- `assets/site.css`, `assets/site.js`, `assets/verify.js`: shared styles and behavior
- `fraudshield-guardian/guardian.html`, `guardian.css`, `guardian.js`, `sample-invoices-template.csv`: authenticated Guardian app, including CSV invoice import
- `vercel.json`: existing host redirect and clean URL config

Use the existing Vercel project and deploy. Intended routes: `/`, `/verify`, `/fraudshield-guardian/guardian`, `/dashboard` (existing app). The `/dashboard` route already routes into existing workspace (keep its original file). The homepage's mobile menu and Verify form work without a backend.

## Working features
- **Verify:** browser-only checklist takes manually entered details and returns a grounded review summary with actions and copy-to-clipboard. No data transmission or bank verification.
- **Guardian:** uses project `nivqhnqquwszwtsgupgx` with a **publishable client key** and Supabase session already used by `chat.html`. Saves, queries, reopens and deletes `guardian_records`; CSV import and export; duplicate rule matches `(kind, vendor/customer, invoice reference, amount)`; overdue receivable review; read-only view of available `transactions`; weekly recap and search.
- **Website:** fully responsive landing, direct product navigation, no fabricated account metrics or false claims of successful fraud interception. Existing chat/workspace can continue separately.

## Important limitations
- The connected Supabase project already has the `guardian_records` table and 4 owner-scoped RLS policies. Verify it in production with two separate ordinary user accounts before collecting sensitive business data.
- There is **no automated QuickBooks, Stripe, Gmail ingestion** in Guardian; no AI fraud identification, bank ownership verification or automatic payment blocking. Those need additional properly authorized backend integrations and controls.
- The Verify form does not save to Guardian automatically. After checking, customers can manually add a review in Guardian.
- Do not replace the complete repository with only this overlay: existing `chat.html` and login/integrations are preserved by **retaining the current files**.
- The Guardian UI depends on the Supabase UMD SDK from jsDelivr. Network, CORS, OAuth redirect and device testing are still necessary before production rollout.
- No claims about actual prevented fraud losses, customer counts, security certification, or banking partnerships.

## Test checklist
1. Load homepage on desktop and mobile; inspect menu and CTA routes.
2. Use `/verify` with changed-details checked, then with a clean checklist; verify clear guidance and clipboard.
3. Sign in through `/dashboard` with a genuine test account.
4. Open `/fraudshield-guardian/guardian`; create an invoice, refresh, reopen, export.
5. Import a sample CSV of non-sensitive data; check duplicate and due-date signals.
6. Sign out; repeat with another account and ensure no other user's records are visible.
7. Verify all existing chat agents and original integration routes still work.
