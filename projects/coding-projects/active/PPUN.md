# PPUN
*Coding Project - Created 2026-03-24*

## Description
MyInfra PPUN (Penyediaan Pelan Pengurusan Negara) — Government infrastructure asset management system. Backend (Laravel) + Frontend (Nuxt/Vue) with multi-module architecture (Penyediaan & Pelaksanaan flows).

## Project Details
- **Type**: Coding Project
- **Status**: Active
- **Created**: 2026-03-24
- **Last Accessed**: 2026-03-24
- **Position**: #1

## Technical Stack
- **Languages**: PHP, TypeScript, JavaScript
- **Frameworks**: Laravel (Backend), Nuxt 3 / Vue 3 (Frontend), PrimeVue (UI)
- **Database**: PostgreSQL (3 DBs: myinfradb_ppun, myinfradb_ppa, myinfradb — PGSN)
- **Tools**: Docker, Git, MCP DB connections

## Project Structure
```
Backend (Laravel): C:\coding\myinfra-ppun-opa\myinfra-ppun
Frontend (Nuxt):   C:\coding\myinfra-frontend
PPA (Assets):      C:\coding\myinfra-ppun-opa\myinfra-ppa
PGSN (Users/Orgs): C:\coding\myinfra-pgsn
Shared Core:       C:\coding\myinfra-ppun-opa\myinfra-core
```

## Development Goals
1. Maintain and enhance Penyediaan Pelan flow (5-tab registration + verification)
2. Maintain and enhance Pelaksanaan Pelan flow (parallel structure)
3. Implement "Tiada Aktiviti" feature for skipping tabs when no aktiviti needed
4. PDF report generation for Analisis Keperluan Sumber
5. Jumlah kos totals per jenis_projek group + grand total

## Progress Log
### 2026-03-24 (Session 3)
**Asset Dropdown Org Filtering (getDpasByOrg)**
- Stakeholder requirement: "Bagi pilihan Aset Pelan, senarai aset hendaklah ditapis mengikut jabatan/agensi"
- Created new PPA endpoint `getDpasByOrg()` in `AsetController.php` — filters `asets.jabatan_id` by user's org hierarchy
- Kept original `getDpas()` untouched to avoid breaking other modules
- Added PGSN lookup fields: `location_id`, `path`, `depth` to `InternalLookupController` organizations response
- Frontend: new `$services.aset.getDpasByOrg(type)` method in `services/api/aset.ts`
- Updated `PelanBangunan.vue` (penyediaan) to use `getDpasByOrg` instead of `getDpas`
- Remaining pages still need updating: PelanJalan (penyediaan/pelaksanaan), PelanBangunan (pelaksanaan), PelanJalan (pelaporan)

**Pegawai Cross-Org Visibility Fix (Maklumat Struktur)**
- Issue: User A saves pegawai officers → User B from different org can't see them (dropdown filtered by viewer's org)
- Root cause: `assignSelectedOfficer()` searches saved officer IDs against org-filtered `getPegawaiWithRole` results
- Fix (Option A): Backend `getPegawaiWithRole` now accepts `include_ids[]` param — fetches missing saved officers individually via `PgsnApiService::getPegawai()` and injects into correct role list
- Frontend: `pendaftaran/index.vue` + `pengesahan/index.vue` now pass saved officer IDs (`ptf/pif/pdf/pof/pdf_staff_officer_id`) when calling `getPegawaiWithRole`
- `ppun.ts` service updated: `getPegawaiWithRole(jenisAsetId?, includeIds?)` accepts optional include_ids array

**Pegawai Active Status — Already Handled**
- Confirmed PGSN `getPegawaiByRole` (InternalUserController:148) already filters `users.status = 1`
- Only 2 inactive pegawai in system (status=0), 529 active — no changes needed

**Jalan GPS Map — Data Investigation**
- `FormBangunanMaklumatPremis.vue` line 185: GoogleMapRoute uses dummy hardcoded coordinates
- Investigated: all 78 Jalan aset records have NULL `koordinat_seksyen_mula/akhir` fields
- Some DAK-level records have `koordinat_mula_x/y` populated but PPUN selects at DPA (aset) level, not DAK
- Pending: need PPA to populate aset-level coordinates, or alternative approach

### 2026-03-24 (Session 2)
**"Tiada Aktiviti" Feature (Penyediaan only)**
- Created migration `2026_03_16_000000_add_tiada_aktiviti_to_ppuns_table.php` — boolean column on ppuns
- Added `tiada_aktiviti` to Ppun model `$fillable` + `$casts`
- Added validation rule `ppun.tiada_aktiviti` to both Jalan and Bangunan save forms
- Added `tiada_aktiviti` to `PpunResource` API response (was missing — root cause of persistence bug)
- Frontend: checkbox on Tab 3 (AnalisisKeperluanSumber) with confirmation dialog
- Frontend: disables stepper steps 4 & 5 (Kawalan Rekod, Rujukan) + Tab disabled for non-Draf
- Frontend: shows "Hantar" button on Tab 3 when checked, hides "Simpan & Seterusnya"
- Frontend: saves `tiada_aktiviti` in all payloads (updateOpa, saveAnalisisKeperluan, saveAllTabs)
- Frontend: handleSubmit saves analisis tab instead of rujukan when tiada aktiviti

**Jumlah Kos Totals (Bangunan — both Penyediaan & Pelaksanaan)**
- Added `#groupfooter` template to DataTable — subtotal per jenis_projek group
- Added `groupTotals` computed (keyed by jenis_projek)
- Penyediaan: added grand total bar below DataTable
- Pelaksanaan: group subtotals added (grand total already existed in "Keseluruhan" section)

**Keperluan Sumber Dropdown — "Tiada Keperluan" option**
- Added "Tiada Keperluan" to `KeperluanSumberSeeder.php` in PPA repo (uses firstOrCreate)
- Seeder command: `cd myinfra-ppa && php artisan db:seed --class=KeperluanSumberSeeder`

**PDF Report Fixes (ppun-report.blade.php)**
- Fixed signature section: was using `['user']['supervisor']['name']` but PGSN API returns flat `['nama']`
- Pegawai penyedia/pengesah now use correct `['nama']` and `['jawatan']` fields
- "(Cap Nama & Jawatan)" now only shows when pegawai is not assigned (conditional @if)
- Added "Dijana pada: dd/mm/yyyy hh:mm:ss" date stamp above border (position: fixed, top: -0.15in)
- Same date stamp added to pelaksanaan-report.blade.php

**Checklist Save Bug Fix (both Penyediaan & Pelaksanaan)**
- handleSaveChecklist was replacing entire skopRecords array from DB, wiping unsaved records
- Fix: preserve unsaved records (filter `!r.uuid`) and append back after refresh

### 2026-03-24 (Session 1)
- Fixed checklist data persistence on reopen (AddSkopModal state reset + direct API refresh)
- Fixed catatan not saving to DB (added validation rule in UpdateSkopRequest.php)
- Fixed duplicate jenisSkop sections in pelaksanaan ViewSkopModal (watch vs onMounted)
- Fixed pengesahan saveSkopRecord outdated payload (missing sub_item_details, dak_ids)
- Fixed Koordinat (X/Y) not showing in pengesahan (missing lokasi assignment in data loading)
- Applied lokasi fix to pelaksanaan pengesahan as well
- Added saveRujukanTab() before submit in both pendaftaran and pengesahan handleSubmit/confirmVerify
- Planned "Tiada Aktiviti" feature — checkbox on Tab 3 to disable tabs 4&5 and allow submit from Tab 3

## Code Standards
- **Style Guide**: Vue 3 Composition API with `<script setup>`, PrimeVue components
- **Testing**: Feature tests (Laravel)
- **Documentation**: CLAUDE.md memory system
- **Git Workflow**: Feature branches off master

## Current Tasks
- [x] Implement "Tiada Aktiviti" feature (backend migration + frontend logic)
- [x] Jumlah kos group subtotals + grand total (Penyediaan & Pelaksanaan)
- [x] "Tiada Keperluan" dropdown option in Keperluan Sumber
- [x] PDF signature section fix (pegawai penyedia/pengesah names)
- [x] PDF generated date stamp
- [x] Checklist save preserves unsaved skop records
- [x] Asset dropdown org filtering — `getDpasByOrg` (PelanBangunan penyediaan done)
- [x] Pegawai cross-org visibility — `include_ids` param (pendaftaran + pengesahan done)
- [ ] Update remaining pages to use `getDpasByOrg`: PelanJalan (penyediaan/pelaksanaan), PelanBangunan (pelaksanaan), PelanJalan (pelaporan)
- [ ] Clean up debug logs from PPA AsetController + PPUN AssetProxyController
- [ ] Jalan GPS map — replace dummy coordinates with real aset data (blocked: aset coordinates all NULL)
- [ ] AktivitiPickerModal for Jalan assets (planned, not started)
- [ ] Pegawai pengesah assignment flow (pegawai_pengesah_id always null — needs mapping from reviewer on "Selesai")

## Known Issues
- `pegawai_pengesah_id` is never populated — needs flow to set it when status changes to "Selesai"
- Checklist button shows on unsaved records (no uuid) — will fail on save since no backend record exists
- All 78 Jalan asets have NULL GPS coordinates at aset level — GoogleMapRoute shows dummy data
- Two PPUN pegawai endpoints (`/ppun/pegawai` and `/ppun/pegawai-struktur`) — both call PGSN but with different filters

## Resources & References
- Org hierarchy doc: `c:/coding/myinfra-pgsn/docs/organization-hierarchy.md`
- MCP DB connections: postgres-pgsn, postgres-ppa, postgres-ppun
- Frontend PPUN pages: `pages/(modules)/ppun/penyediaan-pelan/` and `pelaksanaan-pelan/`
- PGSN internal API returns flat pegawai: `{ id, nama, jawatan, no_kp, emel, ... }` — NOT nested user/supervisor

## Session Notes
### Key Patterns
- `props.skopRecords.splice(0, len, ...newData)` — direct prop mutation pattern used throughout
- `sub_item_details` — per-butiran detail fields, needs JSON.parse(JSON.stringify()) to clone from reactive proxy
- Bangunan vs Jalan: different butiran flows (free-text vs picker, section-level vs per-entry detail fields)
- StepperComponent supports `step.disabled` property for greying out steps
- Both penyediaan and pelaksanaan share same blade template for PDF: `analisis-keperluan.blade.php`
- `defineModel('tiadaAktiviti')` — Vue 3.4+ two-way binding for component state
- Checklist refresh: filter `!r.uuid` to preserve unsaved records before splice

## Memory Patterns
- Laravel strips unvalidated fields from `$request->validated()` — always add new fields to validation rules
- `onMounted` fires before props are ready — use `watch(..., { immediate: true })` for prop-dependent fetches
- `defineProps` must come before any `watch` that references `props` in `<script setup>`
- `emit('event')` is fire-and-forget for async parent handlers — use direct API calls for guaranteed sequencing
- PpunResource explicitly lists returned fields — new columns MUST be added there or frontend won't receive them
- PGSN getPegawai API returns flat `['nama']` not nested `['user']['name']` — blade templates must match
- Asset scoping uses `asets.jabatan_id` (PGSN org ID), NOT `location_daerah_id`/`location_negeri_id`
- PGSN internal lookups cache 3600s — must clear cache on BOTH PGSN and consuming services after changes
- New endpoints preferred over modifying existing ones when other modules depend on them (e.g. `getDpasByOrg` vs `getDpas`)
- Cross-org data visibility: when saving references (officer IDs), ensure viewing users can resolve them even if outside their org filter
- PGSN `getPegawaiByRole` already filters `users.status = 1` — no need to filter active users at PPUN level

---
*Coding Project Template v1.0*
