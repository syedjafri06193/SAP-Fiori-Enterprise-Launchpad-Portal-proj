# Spaces and Pages Strategy – BINARY

## 1. Overview

BINARY adopts SAP's **spaces and pages** layout model to give every finance persona a consistent, admin-controlled entry point. Spaces and pages replace the classic home page with groups.

| Concept | Description |
|---|---|
| **Space** | A tab in the launchpad shell bar, assigned to users through a PFCG role |
| **Page** | Content of a space, made of sections |
| **Section** | Titled area on a page grouping related apps |
| **Visualization** | A tile or link representing an app from a catalog in the user's roles |
| **My Home** | Personal page where users pin favorite apps (where enabled) |

---

## 2. Why Spaces and Pages

| Classic groups | Spaces and pages |
|---|---|
| Layout derived from catalogs and personalization; inconsistent across users | Layout designed by admins per role; consistent |
| Heavy personalization makes rollouts difficult | Changes transported and appear for all role holders |
| All tiles of all groups render at start-up | Only the active page renders; faster load |
| Groups tied to catalogs | Layout decoupled from authorization (catalogs) |
| Being superseded | SAP's strategic layout model |

---

## 3. Design Guidelines

### 3.1 Structure
- **One primary space per persona**, named in business language ("Accounts Payable").
- **One page per space** by default; add a second page only when a persona has a clearly separate workload (e.g. GL: "Daily Accounting" and "Period Close").
- **3–5 sections per page**, ordered by frequency of use.
- **≤ 25 visualizations per page**. Beyond that, use links rather than tiles or split pages.

### 3.2 Section order
1. **Daily tasks** — the highest-frequency transactional apps
2. **Approvals / worklists** — My Inbox, blocked items
3. **Reporting and analytics** — KPI and analytical tiles
4. **Master data (display)** — factsheets and lookups
5. **Administrative / occasional** — shown as links

### 3.3 Tiles vs. links
| Use a tile when | Use a link when |
|---|---|
| App is used daily | App is used occasionally |
| A dynamic count helps the user prioritize (e.g. invoices to approve) | The app is a lookup or report |
| KPI display is valuable | Space is limited |

### 3.4 Dynamic tiles
- Only where the number drives action (e.g. "Blocked Invoices: 14").
- Backed by a fast `$count` call; refresh ≥ 300 s unless justified.
- Maximum ~4 dynamic/KPI tiles per page to protect performance.

### 3.5 Titles & language
- Section titles are verbs + objects: "Process Invoices", "Approve Payments".
- Translated into all logon languages in scope.

---

## 4. Space Inventory

| Space | Page(s) | Role(s) |
|---|---|---|
| `Z_FIN_AP_SP` – Accounts Payable | `Z_FIN_AP_PG` | `Z_FIN_AP_CLERK_BR` |
| `Z_FIN_AP_MGR_SP` – AP Management | `Z_FIN_AP_MGR_PG` | `Z_FIN_AP_MANAGER_BR` |
| `Z_FIN_AR_SP` – Accounts Receivable | `Z_FIN_AR_PG` | `Z_FIN_AR_CLERK_BR` |
| `Z_FIN_GL_SP` – General Ledger | `Z_FIN_GL_PG` (Daily), `Z_FIN_GL_PG02` (Period Close) | `Z_FIN_GL_ACCOUNTANT_BR` |
| `Z_FIN_CASH_SP` – Cash Management | `Z_FIN_CASH_PG` | `Z_FIN_CASH_MANAGER_BR` |
| `Z_FIN_CTRL_SP` – Controlling | `Z_FIN_CTRL_PG` | `Z_FIN_CONTROLLER_BR` |
| `Z_FIN_AUDIT_SP` – Audit | `Z_FIN_AUDIT_PG` | `Z_FIN_AUDITOR_DISPLAY_BR` |
| `Z_TEC_ADMIN_SP` – Launchpad Administration | `Z_TEC_ADMIN_PG` | `Z_TEC_FLP_ADMIN_BR` |

---

## 5. Sample Page Layout – General Ledger (`Z_FIN_GL_PG`)

| Section | Visualizations | Display |
|---|---|---|
| 1. Daily Accounting | Post General Journal Entries, Manage Journal Entries, Upload Journal Entries | Tiles |
| 2. Reconciliation | Display G/L Account Line Items, Clear G/L Accounts | Tiles |
| 3. Reporting & Analytics | Trial Balance, Display G/L Account Balances, Balance Sheet/Income Statement | Tiles (1 KPI) |
| 4. Reference | Display Chart of Accounts, Display G/L Account Master Data | Links |

Period-close tasks live on the second page `Z_FIN_GL_PG02`, so day-to-day users are not distracted by them for most of the month.

The AP layout is specified in [sample-space-page-layout.md](../samples/sample-space-page-layout.md).

---

## 6. Migration from Groups

### 6.1 Approach
1. **Inventory** existing groups per role, and tile usage.
2. **Design** target pages per persona (workshops with process owners).
3. **Generate** draft pages from roles with SAP's role-based page creation where available (creates pages from catalog content), then curate manually.
4. **Build** spaces/pages in DEV; assign to roles.
5. **Test** in QAS with representative users.
6. **Enable** spaces mode in launchpad settings and set user activation off.
7. **Communicate** the change and train users (see [User Onboarding](user-onboarding-and-training.md)).

### 6.2 Personalization
- Users' personalized groups are not migrated 1:1. Users are told in advance and can recreate favorites on **My Home**.
- Page personalization (if enabled) is limited so admin-controlled updates remain visible.

### 6.3 Cutover checklist
- [ ] All target pages imported to PRD
- [ ] Roles updated with spaces
- [ ] Launchpad settings switched to spaces mode
- [ ] Global caches invalidated
- [ ] Pilot users verified
- [ ] Communication sent

---

## 7. Maintenance Rules

- Add a tile to a page only after its catalog is in the role.
- Removing an app: remove from the page first, then from the catalog in a later transport (avoids broken visualizations).
- Page changes are transported and tested like any other content.
- Quarterly usage review removes tiles nobody opens.

---

## 8. Benefits Realized

- Reduced cognitive load: median 18 visualizations per user versus 62 before
- Faster task execution: daily apps one click from the landing page
- Consistent experience across departments and users
- Faster launchpad start-up because only the current page renders

---

## Related Documents

- [Catalog & Role Design](catalog-and-role-design.md)
- [Launchpad Configuration Guide](launchpad-configuration-guide.md#6-create-spaces-and-pages)
- [Performance Optimization](performance-optimization.md)
