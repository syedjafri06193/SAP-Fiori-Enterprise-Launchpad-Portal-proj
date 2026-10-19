# Catalog and Role Design – BINARY

## 1. Design Principles

| Principle | How it is applied |
|---|---|
| **Least privilege** | Each role grants only the catalogs and authorizations the persona needs |
| **Segregation of duties** | Conflicting process steps (e.g. maintain vendor ↔ pay vendor) never sit in the same role |
| **Process alignment** | Catalogs mirror business process steps, not org charts |
| **Reuse** | Catalogs are building blocks shared across roles (e.g. reporting) |
| **Reference, don't copy** | Catalogs reference SAP technical catalog content |
| **Scalability** | Master roles + derived roles per company code |
| **Compliance** | Every role has a business owner, SoD check and quarterly review |

---

## 2. Layered Model

```
User ──▶ PFCG Business Role (Z_FIN_*_BR)
           ├── Business Catalogs (Z_FIN_*_BC)      ── front-end: what apps exist
           │      └── references to SAP technical catalog tiles + target mappings
           ├── Space (Z_FIN_*_SP) ─▶ Page(s) (Z_FIN_*_PG) ── front-end: how apps are arranged
           └── Authorizations (S_SERVICE, S_START, S_TCODE, F_*, K_*) ── back-end: what data/actions
```

Each business role consists of:

- One PFCG single role (master) with derived roles per company code where org-level restriction is needed
- One or more business catalogs
- Exactly one primary space (plus optionally a shared space)
- Back-end authorization objects derived from SU24 proposals and reviewed manually

---

## 3. Personas and Roles

| Role | Persona | Core tasks | Frequency drivers |
|---|---|---|---|
| `Z_FIN_AP_CLERK_BR` | AP Clerk | Enter/park supplier invoices, match, resolve blocks, display vendor items | Daily, high volume |
| `Z_FIN_AP_MANAGER_BR` | AP Manager | Approve invoices, release blocked invoices, run/approve payment proposals | Daily/weekly |
| `Z_FIN_AR_CLERK_BR` | AR Clerk | Post incoming payments, clear open items, dunning, display customers | Daily |
| `Z_FIN_GL_ACCOUNTANT_BR` | GL Accountant | Post/park journal entries, accruals, reconciliations, period close tasks | Daily + close |
| `Z_FIN_CASH_MANAGER_BR` | Cash Manager | Cash position, liquidity forecast, bank accounts, bank statements | Daily |
| `Z_FIN_CONTROLLER_BR` | Controller | Cost center reporting, allocations, variance analysis | Weekly + close |
| `Z_FIN_AUDITOR_DISPLAY_BR` | Auditor | Display documents, line items, change logs, reports | Periodic |
| `Z_TEC_FLP_ADMIN_BR` | Launchpad Admin | Maintain catalogs, spaces, pages; cache, diagnostics | As needed (DEV) |

---

## 4. Catalog Inventory

| Catalog | Process | Example apps | Used by roles |
|---|---|---|---|
| `Z_FIN_AP_PROCESSING_BC` | Invoice processing | Create Supplier Invoice, Manage Supplier Invoices, Supplier Line Items (FBL1N) | AP Clerk |
| `Z_FIN_AP_APPROVAL_BC` | Invoice approval | My Inbox, Release Blocked Invoices | AP Manager |
| `Z_FIN_PAYMENT_RUN_BC` | Payments | Manage Automatic Payments, Approve Bank Payments | AP Manager |
| `Z_FIN_VENDOR_DISPLAY_BC` | Supplier master (display) | Display Supplier, Supplier Factsheet | AP Clerk, AP Manager, Auditor |
| `Z_FIN_AP_REPORTING_BC` | AP analytics | Overdue Payables, Aging Analysis | AP Clerk, AP Manager |
| `Z_FIN_AR_PROCESSING_BC` | Receivables | Post Incoming Payments, Clear Open Items, Manage Dunning | AR Clerk |
| `Z_FIN_CUSTOMER_DISPLAY_BC` | Customer master (display) | Display Customer, Customer Line Items | AR Clerk, Auditor |
| `Z_FIN_GL_JOURNAL_BC` | Journal entries | Post General Journal Entries, Manage Journal Entries, Upload Journal Entries | GL Accountant |
| `Z_FIN_PERIOD_CLOSE_BC` | Close | Manage Accruals, Open/Close Posting Periods (display), Closing Cockpit tasks | GL Accountant |
| `Z_FIN_REPORTING_BC` | Financial reporting | Trial Balance, Balance Sheet/Income Statement, Display G/L Account Balances | GL Accountant, Controller |
| `Z_FIN_CASH_POSITION_BC` | Cash | Cash Position, Liquidity Forecast | Cash Manager |
| `Z_FIN_BANK_ACCOUNT_BC` | Banking | Manage Bank Accounts, Manage Bank Statements | Cash Manager |
| `Z_FIN_COST_CENTER_BC` | Controlling | Cost Centers – Actuals, Plan/Actual, Manage Allocations | Controller |
| `Z_FIN_AUDIT_DISPLAY_BC` | Audit | Display Journal Entries, Change Documents, Audit Journal | Auditor |
| `Z_TEC_FIN_CUSTOM_APPS_TC` | Technical repository | Custom finance UI5 apps | Referenced by business catalogs |
| `Z_TEC_FLP_ADMIN_TC` | Administration | Manage Launchpad Spaces/Pages, App Finder admin tools | FLP Admin |

> App names are SAP-standard app titles; confirm availability and IDs in the SAP Fiori Apps Reference Library for the installed release.

---

## 5. Role-to-Catalog Matrix

| Catalog \ Role | AP Clerk | AP Mgr | AR Clerk | GL Acct | Cash Mgr | Controller | Auditor |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| AP_PROCESSING | ✅ | | | | | | |
| AP_APPROVAL | | ✅ | | | | | |
| PAYMENT_RUN | | ✅ | | | | | |
| VENDOR_DISPLAY | ✅ | ✅ | | | | | ✅ |
| AP_REPORTING | ✅ | ✅ | | | | | |
| AR_PROCESSING | | | ✅ | | | | |
| CUSTOMER_DISPLAY | | | ✅ | | | | ✅ |
| GL_JOURNAL | | | | ✅ | | | |
| PERIOD_CLOSE | | | | ✅ | | | |
| REPORTING | | | | ✅ | | ✅ | |
| CASH_POSITION | | | | | ✅ | | |
| BANK_ACCOUNT | | | | | ✅ | | |
| COST_CENTER | | | | | | ✅ | |
| AUDIT_DISPLAY | | | | | | | ✅ |

The full matrix with authorization objects and SoD analysis is in [sample-authorization-matrix.xlsx](../samples/sample-authorization-matrix.xlsx).

---

## 6. Naming Conventions

| Object | Pattern | Example |
|---|---|---|
| Roles | `Z_FIN_<FUNCTION>_BR` | `Z_FIN_AP_CLERK_BR` |
| Derived roles | `Z_FIN_<FUNCTION>_<CC>_DR` | `Z_FIN_AP_CLERK_1000_DR` |
| Business catalogs | `Z_FIN_<PROCESS>_BC` | `Z_FIN_AP_PROCESSING_BC` |
| Technical catalogs | `Z_TEC_<AREA>_TC` | `Z_TEC_FIN_CUSTOM_APPS_TC` |
| Spaces | `Z_FIN_<ROLE>_SP` | `Z_FIN_AP_SP` |
| Pages | `Z_FIN_<ROLE>_PG` | `Z_FIN_AP_PG` |

Full standard: [Governance & Naming Standards](governance-and-naming-standards.md).

---

## 7. Example Role: `Z_FIN_AP_CLERK_BR`

| Attribute | Value |
|---|---|
| Business owner | Head of Accounts Payable |
| Catalogs | `Z_FIN_AP_PROCESSING_BC`, `Z_FIN_VENDOR_DISPLAY_BC`, `Z_FIN_AP_REPORTING_BC` |
| Space / page | `Z_FIN_AP_SP` / `Z_FIN_AP_PG` |
| Org levels | Company code via derived roles (`1000`, `2000`) |
| Excluded by design | Vendor master maintenance, payment run execution, bank data change |
| SoD rationale | Clerk can enter invoices but cannot create vendors or release payments |

Includes:
- Invoice processing applications
- Supplier display apps (no maintenance)
- AP reporting tiles
- An AP space and page optimized for the invoice-to-pay workflow

---

## 8. Derived Roles & Org Levels

- Master role holds catalogs, space and authorization values except org levels.
- Derived roles inherit everything and set company code (`BUKRS`), and where relevant controlling area / cost center groups.
- Launchpad content (catalogs/spaces) is inherited by derived roles from the master's menu.
- Changing the master → regenerate derived roles (`PFCG` → "Generate derived roles") in the same transport.

---

## 9. SU24 Maintenance

- SAP delivers authorization defaults for standard services and transactions; SU25 steps are run after upgrades to compare and adopt changes.
- For each custom OData service and custom transaction, SU24 entries are maintained so PFCG proposes `S_SERVICE`/`S_START` and business objects automatically.
- SU24 changes travel in workbench transports with the related development.

---

## 10. Transport Strategy

- All catalog, page, space and role changes are recorded on transports (customizing for client-specific content; workbench for SU24 and development objects).
- Catalogs and pages are transported **before or with** the roles that reference them.
- Emergency changes follow the documented emergency process with retrospective approval within 2 business days.
- Details: [Transport & Change Management](transport-and-change-management.md).

---

## 11. Review Cycle

| Review | Frequency | Owner |
|---|---|---|
| User access recertification | Quarterly | Business role owners |
| SoD rule set run against roles | Each role change + quarterly | Security |
| Catalog usage review | Semi-annual | Fiori lead |
| SU25 / SU24 after upgrade | Each upgrade | Security |

---

## Related Documents

- [Security & Authorization](security-and-authorization.md)
- [Spaces & Pages Strategy](spaces-and-pages-strategy.md)
- [Sample PFCG Role Design](../samples/sample-pfcg-role-design.md)
- [Sample Business Catalog](../samples/sample-business-catalog.md)
