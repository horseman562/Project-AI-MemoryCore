# Vendor Portal Module — Implementation Plan
## Shiseido Approval System × BeaconX iApprove v6.0
*Created: 2026-04-07*

---

## 2 Major Modules

### Module 1: Vendor Account Opening via Portal (PRIORITY — BUILD FIRST)
The process of registering a new supplier/vendor into the system so they can do business with the company. Supplier needs an approved vendor account with a SAP Vendor Code.

**3 Variants:**

#### 1.1 Standard Flow
1. MM creates supplier account in portal
2. System auto-emails supplier (User ID + temp password)
3. Supplier logs in, fills **E-VAOPF** (Vendor Account Proposal Form) — company info + uploads docs (SSM, Section 17/78/58 Form, Form 9/24/49)
4. MM reviews proposal, adds recommendations, submits to ED
5. ED approves proposal
6. System notifies supplier → supplier fills **E-VAOF** (Vendor Account Opening Form), signs **Vendor Agreement (VA)**, uploads bank docs
7. Finance reviews, verifies, submits to ED
8. ED gives final approval
9. Finance downloads supplier info → uploads to SAP → creates Vendor Code
10. Finance updates Vendor Code back into portal — DONE

**2 Forms:**
- **E-VAOPF** (Proposal) = "Should we work with this supplier?" — screening
- **E-VAOF** (Opening) = "Let's officially onboard them" — registration with bank details + signed VA

#### 1.2 Taiwan Agency / Publisher Flow
- Supplier (TPBC) emails info & agreement to MM
- MM fills E-VAOF + uploads docs on behalf of supplier
- Finance verifies → ED approves → Finance creates SAP Vendor Code

#### 1.3 Guangzhou Buying Office Flow
- GZ Office sends docs + signed VA to MM
- MM fills E-VAOF + uploads VA on behalf of supplier
- Finance verifies → ED approves → Finance creates SAP Vendor Code

**Note:** All suppliers (Trade and Non-Trade) use Standard flow, EXCEPT Taiwan Agency/Publisher and GZ Buying Office vendors.

---

### Module 2: Vendor Account Maintenance Flow (BUILD AFTER MODULE 1)
Updating existing vendor information after account is already opened.

**2 Variants:**

#### 2.1 External (Supplier-initiated)
1. Supplier logs into portal
2. Fills E-Vendor Account Maintenance form (External) or updates existing profile
3. MM receives notification, reviews, submits
4. **Conditional routing:**
   - If credit terms/trade discount changed → ED approval needed
   - If NO credit/discount changes → Finance only
5. Finance downloads changed info → updates SAP

#### 2.2 Internal (MM-initiated)
1. MM logs in, fills E-Vendor Account Maintenance form (Internal)
2. **Same conditional routing** as External
3. Finance updates SAP

**Key Business Rules:**
- If credit terms shortened OR trade discount lowered → popup notification, MM must tick checkbox confirming meeting with **Ms Lim**
- Trade discounts maintained **separately by department**
- MMs can only view/edit trade discounts for **their own department**
- Existing (old) info **pre-filled** in forms for reference
- Suppliers can only access departments **enabled/extended by MM**

---

## Implementation Phases

| Phase | What | Complexity | Depends On |
|-------|------|-----------|------------|
| **1** | Supplier Data Model + User Type separation | M | — |
| **2** | Supplier Portal Auth + Shell | M | Phase 1 |
| **3** | Vendor Account Opening (3 flows) | XL | Phase 1+2 |
| **4** | Vendor Account Maintenance (2 flows) | L | Phase 1+2+3 |
| **5** | Reporting + Polish | M | Phase 1-4 |

**Decision:** Focus on Account Opening (Phases 1-3) first. Maintenance (Phase 4) comes after because:
1. Can't maintain a vendor that doesn't exist
2. Opening builds the foundation (portal, forms, uploads, SAP tracking)
3. Maintenance's conditional routing can wait — Opening uses linear tier approval

---

## Vendor Data Fields (16)
1. Vendor Name
2. Business Address (Street, City, Postal code, Country)
3. Warehouse Address (Street, City, Postal code, Country)
4. Telephone no.
5. Mobile Phone no.
6. Fax no.
7. Email
8. SST no.
9. TIN no.
10. Bank details (Bank name, Account number, SWIFT code, CNAP, IBAN)
11. Company ID (OLD)
12. Company ID (NEW)
13. Identification no. (IC)
14. Payment term
15. Order currency
16. Vendor sub-range

## Roles
- **Super Admin** — create backend users, assign dept access, manage approval matrix, audit trails
- **Merchandiser (MM)** — initiates vendor accounts, reviews supplier submissions
- **Supplier (External)** — logs into portal, fills forms, uploads docs
- **Finance** — verifies, creates SAP Vendor Code, updates portal
- **Executive Director (ED)** — final approval authority

## Portal
**Supplier Dashboard includes:**
- Vendor Account Proposal Form
- Vendor Account Opening Form
- Vendor Account Maintenance Form (External)

**Backend User Screen:**
- Can view vendor code + vendor name for all suppliers
- Can only view selected supplier profiles for their own department

---

## What Can Be Reused From Existing System (~40-50% savings)
- ApprovalService (1,642 lines) — approve/reject/return/delegate/escalation
- ApprovalFlow + Tier + Approver + Approval models
- Attachment model — document uploads
- Email infrastructure — 7 existing Mail classes as templates
- Spatie Permissions — add new roles
- PrimeVue components — all form inputs, dialogs, tables
- PrType JSON payload system

## What's New Build
- Supplier portal auth + layout
- Vendor-specific forms (E-VAOPF, E-VAOF, Maintenance)
- Conditional approval routing (Phase 4 only)
- Department-scoped trade discounts
- SAP vendor code lifecycle tracking
- Document checklist component (Uploaded / N/A)

---
*Reference: BeaconX iApprove Proposal v6.0 (Popular Approval Flow Chart)*
*Full technical plan: C:\Users\Datakraf\.claude\plans\resilient-hatching-grove.md*
