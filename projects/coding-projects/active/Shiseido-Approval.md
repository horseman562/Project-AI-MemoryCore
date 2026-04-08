# Shiseido Approval
*Coding Project - Created 2026-03-31*

## Description
Shiseido Approval System — Purchase request approval workflow with inventory management, multi-tier approval matrix, and Excel-based inventory uploads. Production: approval.shiseido.com.my, Staging: approval.beaconx.com.my.

## Project Details
- **Type**: Coding Project
- **Status**: Active
- **Created**: 2026-03-31
- **Last Accessed**: 2026-04-07
- **Position**: #1

## Technical Stack
- **Languages**: PHP, JavaScript
- **Frameworks**: Laravel (Backend), Vue/Inertia (Frontend)
- **Database**: PostgreSQL
- **Auth**: Laravel Fortify + Jetstream, Spatie Permissions, 2FA
- **Excel**: PhpOffice/PhpSpreadsheet, Maatwebsite/Excel
- **Server**: AWS EC2 (Amazon Linux), Nginx, PHP-FPM, Cloudflare
- **Queue**: Laravel Queue (database driver), systemd services

## Project Structure
```
Codebase:    C:\coding\approval
Production:  /var/www/html/approval (EC2: ip-172-31-37-152)
Staging:     /var/www/html/approval (EC2: ip-172-31-40-98)
```

## Server Architecture
- **Prod EC2**: ip-172-31-37-152 (7.7GB RAM) — shared with einvoice app
- **Staging EC2**: ip-172-31-40-98
- **Queue workers**: `laravel-queue-approval.service` (systemd)
- **Other services on prod**: `laravel-queue-prod.service` (einvoice), `laravel-queue.service` (einvoice UAT)
- **Cloudflare**: in front of both prod and staging (~100s timeout)

## Key Files
- `app/Http/Controllers/InventoryController.php` — upload/process Excel inventory files
- `app/Http/Controllers/DashboardController.php` — dashboard with approval counts
- `app/Services/ApprovalService.php` — pending/returned approval queries (N+1 pattern)
- `app/Services/InventoryUpdateService.php` — reserved stock recalculation
- `app/Jobs/ProcessInventoryChunk.php` — chunked inventory import (500 per batch)
- `app/Jobs/FinalizeInventoryDeltaLogJob.php` — delta log after import
- `app/Http/Middleware/HandleInertiaRequests.php` — 40+ permission checks per request

## Data Scale
- 3,471 submissions | 48 pending | 68 returned
- 67 users | 10,705 approvers | 7,884 inventory items
- 367 BP customers | 0 suppliers
- Upload files: ~4.5-10MB Excel (xlsm), 14k+ records per file

## Progress Log
### 2026-04-06/07
**Server OOM & PHP-FPM Fixes**
- Root cause: PHP-FPM workers never recycled (`pm.max_requests` disabled), PhpSpreadsheet memory leaks accumulated over days → 1.6GB → OOM kill → 502 errors
- Enabled `pm.max_requests = 500` on both prod and staging
- Created `php-fpm-restart.timer` on both servers (daily restart at 4am UTC / 12pm MYT)
- Enabled `laravel-queue-approval.service` on prod (was disabled, 152 jobs stuck)
- Added `fastcgi_read_timeout 300` to staging nginx config
- Fixed `InventoryUpdateService.php` — PostgreSQL `reserved_stock` type mismatch (`::numeric` cast)

### 2026-04-01
**Inventory Upload Memory Fix**
- Added `$spreadsheet->disconnectWorksheets()` + `unset()` after reading Excel in both `upload()` and `process()` methods
- Fixed `getAllInventory()` called 4x → 1x in upload response
- Memory per upload reduced from ~300MB to ~30MB

**Dashboard Query Optimization**
- Eliminated duplicate `ApprovalService` calls (getAllPendingApprovals called 2x, getPendingApprovals called 2x)
- Replaced 15 individual `COUNT()` queries with 3 `GROUP BY` queries
- Reduced dashboard queries from ~278 to ~30 per load

### 2026-03-03 (prior)
**Inventory Import Optimization** (commit 9a86814)
- Changed pre-import snapshots from `Inventory::get()` (full Eloquent models) to `DB::table()->pluck()` + `chunk(2000)`

## Known Issues
- **ApprovalService N+1**: `getPendingApprovals()` does `$query->get()` then loops with 2 DB queries per submission — should use database-level filtering
- **HandleInertiaRequests**: 40+ permission checks on every request — could cache per session
- **PhpSpreadsheet**: Still loads entire file into memory — long-term fix is switch to streaming reader (OpenSpout)
- **Prod nginx**: `fastcgi_read_timeout` not yet added for approval.shiseido.com.my
- **Daily restart timer**: Set to 4am UTC (12pm MYT) — may want to change to 8pm UTC (4am MYT) for less user impact
- **storage/app/uploads/**: 812MB of old import files never cleaned up on prod
- **Bot scanning**: einvoice site getting scanned for wp-content/plugins, .env etc (low risk, just bots)

## Server Config Reference
- PHP-FPM config: `/etc/php-fpm.d/www.conf`
- Nginx approval (staging): `/etc/nginx/conf.d/approval.conf`
- Queue service (approval): `/etc/systemd/system/laravel-queue-approval.service`
- PHP-FPM restart timer: `/etc/systemd/system/php-fpm-restart.timer`
- PHP memory_limit: 1000M | upload_max_filesize: 1000M

### 2026-04-07
**Reviewed BeaconX iApprove v6.0 Proposal (Popular Approval Flow Chart)**
- Boss confirmed PDF is for new features to build on this system
- It's a **Vendor Portal module** — practically a second module, not a small feature
- 3 vendor account opening flows: Standard, Taiwan Agency/Publisher, Guangzhou Buying Office
- 2 vendor account maintenance flows: External (supplier self-service) and Internal (MM-driven)
- Conditional approval routing: credit terms/trade discount changes → ED approval, otherwise → Finance only
- Business rule: MM must confirm meeting with Ms Lim if credit terms shortened or trade discount lowered
- Trade discounts scoped per department — MMs only edit their own
- Supplier portal needed (external users) — current system is internal-only
- SAP vendor code creation + update flow
- 16 vendor data fields defined (name, addresses, bank details, SST, TIN, etc.)
- Roles: Super Admin, Merchandiser (MM), Supplier (external), Finance, Executive Director (ED)
- Assessment: reuse existing ApprovalFlow engine, but forms/portal/conditional routing are all new work

### 2026-04-07 (continued)
**Vendor Portal Module — Full Analysis & Planning Session**

Critical decisions made:
1. **v2 approach** — new tables, don't modify existing v1 tables
2. **2-stage opening = 2 separate submissions** (Option A) — linked by `vendor_account_id`, keeps approval engine untouched
3. **New tables**: `vendor_accounts`, `vendor_account_documents`, `vendor_trade_discounts`, `vendor_account_submissions`
4. **Portal access by vendor type**: Standard = portal login, Taiwan/GZ = no login (MM proxy)
5. **Maintenance**: 2 forms only (External + Internal), not 3. External only for Standard vendors with portal login
6. **Admin/Superadmin**: system setup only, NOT part of vendor workflows. Just needs new supplier user creation + dept assignment on existing user CRUD

Docs created in project:
- `docs/vendor-portal-plan.md` — phases, decisions, architecture
- `docs/vendor-maintenance-explained.md` — maintenance breakdown + how vendor types handle maintenance
- `docs/vendor-roles-and-flows.md` — all 5 roles, ASCII flow diagrams for all 5 workflows, role access summary

**Database Schema Designed (7 new tables, zero existing modified):**
1. `vendor_accounts` — central vendor entity (16 structured columns, not JSON)
2. `vendor_users` — bridge table solving user_type without modifying `users` table
3. `vendor_account_submissions` — pivot linking vendor to submissions (Stage 1/2/maintenance)
4. `vendor_account_documents` — document checklist with Uploaded/N/A/pending status
5. `vendor_trade_discounts` — department-scoped via brand_code + brand_user pivot
6. `vendor_change_requests` — maintenance diffs as JSON {field: {old, new, triggers_ed}}
7. `vendor_routing_rules` — config table for conditional ED routing (data-driven, not hardcoded)

Schema docs: `docs/vendor-database-schema.md`
**Branches completed (2026-04-07):**
1. `feat/foundation-migrations` — 7 tables, 7 models, 1 seeder ✅ merged to dev-v2
2. `feat/permissions-roles` — 20 permissions, 4 new roles, 11 User helpers ✅ merged to dev-v2
3. `feat/vendor-workflow-service` — VendorAccountOpeningService + 6 PrTypes seeder ✅ merged to dev-v2
4. `feat/conditional-tier-routing` — VendorRoutingService (evaluateChanges, applyChanges, Ms Lim check) ✅ merged to dev-v2
5. `feat/vendor-notifications` — 5 mail classes + 5 Blade templates ✅ merged to dev-v2

**Branch structure:** All vendor branches merge to `dev-v2` (not `dev`). `dev` stays clean for v1 hotfixes. When vendor portal fully tested, merge `dev-v2 → dev → main`.

**ALL 14 BRANCHES COMPLETED (2026-04-07):**
6. `feat/vendor-opening-backend` — controller, routes, 4 Vue pages (Index/Create/Show/Edit) ✅
7. `feat/vendor-opening-taiwan-gz` — SKIPPED (covered in branch 6, variant dropdown already built) ✅
8. `feat/post-approval-actions` — VendorSubmissionObserver, SAP code entry, suspend/reactivate ✅
9. `feat/supplier-portal` — LoginResponse redirect, EnsureVendorUser middleware, portal layout, dashboard/profile/documents ✅
10. `feat/vendor-maintenance-service` — VendorMaintenanceService (createChangeRequest, submitForApproval, applyChanges) ✅
11. `feat/vendor-maintenance-backend` — VendorMaintenanceController, maintenance form with diff + Ms Lim popup + history ✅
12. `feat/vendor-maintenance-external` — portal maintenance page, limited fields for suppliers ✅
13. `feat/trade-discount-controls` — VendorTradeDiscountController, department-scoped CRUD via brand_user ✅
14. `feat/admin-vendor-management` — VendorUserController (admin), create/deactivate/activate/archive vendor portal users ✅

**All merged to `dev-v2`. Full vendor portal module scaffolded.**

**Standard flow gaps ALL CLOSED (2026-04-07):**
- feat/vendor-portal-proposal-form ✅ — supplier fills E-VAOPF + E-VAOF via portal, added proposal_flow_id/opening_flow_id columns
- feat/vendor-document-uploads ✅ — VendorDocumentController, upload/download/N/A on portal
- feat/vendor-email-wiring ✅ — observer sends invite on proposal approved, service sends SAP code email
- feat/vendor-mm-recommendations ✅ — VendorSubmissionDetails component on Show.vue, existing remarks = recommendations

**CORRECTION (2026-04-08): Re-read PDF page 5 in detail. Standard flow has gaps:**
1. MM only creates User ID + password, NOT vendor data — simplify create form
2. Email sends immediately on account creation, not after approval
3. Supplier must change password on first login
4. MM is a MANUAL GATE — supplier submits, MM reviews, MM routes to ED (not auto-approval)
5. NEW ACTOR: CA (PIC) uploads Vendor Agreement after ED approves Stage 1, before supplier fills E-VAOF
6. Stage 2 approval is MM → Finance → ED (3 tiers, not 2)
7. Finance needs export/download vendor data from portal
8. Finance does VA stamping online + uploads stamped VA
9. Every routing step must trigger email notification

Corrected flow doc: `docs/vendor-opening-standard-corrected.md`

**STANDARD OPENING FLOW COMPLETE (2026-04-08) — tested and working.**
- Role-based approval (VendorApprovalService) — no submission/approval matrix dependency
- Status machine: proposal_pending → proposal_submitted → proposal_mm_approved → proposal_approved → opening_pending → opening_submitted → opening_mm_approved → opening_finance_approved → opening_approved → active
- VA upload step by Finance between Stage 1 and Stage 2
- Buttons restricted by role (Finance sees VA upload + SAP code, ED sees approve only)
- Forms are placeholder — waiting for actual form specs from product owner
- Testing guide: docs/testing-vendor-opening-standard.md

**ARCHITECTURE DECISION (2026-04-08): Vendor opening does NOT use ApprovalFlow/ApprovalTier/Approver tables.**
- Flow is FIXED per the PDF, not configurable per brand
- Route by ROLE (MM, Finance, ED), not by configured approval tiers
- Still reuse Submission engine (create, status tracking, approve/reject/return)
- Stage 1: Supplier submits → MM reviews → ED approves
- Stage 2: Supplier submits → MM reviews → Finance reviews → ED approves
- No approval matrix needed for vendor module
- This is a big change from initial build — need to refactor service layer

**Opening: COMPLETE.** All 3 variants, portal, approval, SAP code entry, observer.

**Maintenance: SCAFFOLDED but has 3 known gaps to fix:**
1. External maintenance portal controller creates change request but doesn't call `submitForApproval()` — MM review step missing in the flow
2. Trade discount changes are in separate table (`vendor_trade_discounts`), not a `vendor_accounts` column — routing service only diffs vendor_accounts columns, so trade discount changes won't trigger ED routing yet
3. Observer logs maintenance approval but doesn't auto-apply changes — currently manual Finance action via "Apply Changes" button. Need to confirm if this is correct or should auto-apply

## Current Tasks
- [x] Diagnose 502 Bad Gateway on production
- [x] Enable pm.max_requests = 500 (prod + staging)
- [x] Create php-fpm-restart.timer (prod + staging)
- [x] Enable laravel-queue-approval.service on prod
- [x] Fix InventoryUpdateService reserved_stock type mismatch
- [x] Add disconnectWorksheets() memory fix (local, needs deploy)
- [x] Dashboard query optimization (local, needs deploy)
- [ ] Add fastcgi_read_timeout to prod nginx config
- [ ] Deploy code changes to prod after QA
- [ ] Clean up old upload files on prod (812MB)
- [ ] Consider changing timer to 4am MYT (8pm UTC)

## Memory Patterns
- PhpSpreadsheet loads files at 10-40x file size in memory — always call `disconnectWorksheets()` + `unset()` after reading
- PHP garbage collector can't fully clean up PhpSpreadsheet's internal object references — causes ~10-20MB leak per upload
- `pm.max_requests` is essential for any PHP app that processes large files
- PostgreSQL `VALUES` clause treats all values as text — need explicit `::numeric` or `::integer` casts
- Laravel queue workers must be restarted after code deploy: `php artisan queue:restart` or `sudo systemctl restart laravel-queue-approval.service`
- Cloudflare has ~100s timeout — long requests need background job pattern or increased nginx timeout

---
*Coding Project Template v1.0*
