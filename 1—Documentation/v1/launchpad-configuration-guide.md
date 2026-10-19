# Launchpad Configuration Guide – BINARY

Step-by-step procedures used to configure the SAP Fiori Launchpad at BINARY. All steps are performed in **DEV (S4D/100)** on a transport request unless stated otherwise.

---

## 1. Tools Overview

| Tool | Transaction / App | Scope | Used for |
|---|---|---|---|
| Launchpad Designer | `/UI2/FLPD_CUST` | Client-specific (customizing) | Business catalogs, tiles, target mappings |
| Launchpad Designer (config) | `/UI2/FLPD_CONF` | Cross-client (configuration) | Technical catalogs (custom) |
| Launchpad Content Manager | `/UI2/FLPCM_CUST`, `/UI2/FLPCM_CONF` | Client / cross-client | Catalog and tile inventory, mass analysis, role assignment view |
| Launchpad App Manager | `/UI2/FLPAM` | Client / cross-client | Maintain apps (tile + target mapping) in technical catalogs |
| Manage Launchpad Spaces | Fiori app | Customizing | Spaces |
| Manage Launchpad Pages | Fiori app | Customizing | Pages, sections, visualizations |
| Launchpad settings | `/UI2/FLP_CUS_CONF`, `/UI2/FLP_SYS_CONF` | Client / system | Global behavior |
| Intent analysis | `/UI2/FLIA` | – | Resolve which target mapping wins for a user/intent |
| Content checks | `/UI2/FLC` | – | Consistency checks for catalogs and roles |
| Role maintenance | `PFCG` | – | Assign catalogs and spaces to roles |

> **Rule:** Business catalogs are maintained in **customizing** scope (`/UI2/FLPD_CUST`) so they are client-specific and recorded on customizing requests.

---

## 2. Global Launchpad Settings

Maintained in `/UI2/FLP_CUS_CONF` (client) with system defaults in `/UI2/FLP_SYS_CONF`. Representative settings at BINARY:

| Setting | Value | Reason |
|---|---|---|
| `SPACES` | `true` | Enable spaces and pages |
| `SPACES_ENABLE_USER_ACTIVATION` | `false` | Users cannot switch back to classic home page |
| `HOME_APP`/`MY_HOME` | My Home enabled | Personal entry page with favorite apps |
| `ENABLE_PERSONALIZATION` | `true` (pages: limited) | Allow users to personalize where sensible |
| `SESSION_TIMEOUT_MINUTES` | `30` | Aligns with corporate security policy |
| `SESSION_TIMEOUT_REMINDER_MINUTES` | `5` | Warn before logout |
| `APPFINDER_SEARCH` | `true` | Users can find apps they are authorized for |
| `ENABLE_HELP` / `ENABLE_ACCESSIBILITY` | `true` | Web Assistant, accessibility |
| Theme | Corporate theme on SAP Horizon | Branding |
| `ENABLE_NOTIFICATIONS` | `true` | Workflow notifications for approvers |
| `CONTACT_SUPPORT` | `true` | Users raise tickets with screenshot + context |

> Exact parameter names vary by `SAP_UI` release. Use the value help in the transaction and confirm against SAP Help for your release.

---

## 3. Create a Business Catalog

1. Open `/UI2/FLPD_CUST` → **Catalogs** → **+** (Create).
2. Choose **Standard** catalog. Title: `Accounts Payable – Invoice Processing`, ID: `Z_FIN_AP_PROCESSING_BC`.
3. Assign a **customizing transport** when prompted.
4. Add content by **reference** from the SAP technical catalog:
   - Open the SAP technical catalog for the app area (e.g. `SAP_TC_FIN_ACC_BE_APPS` / `SAP_TC_FIN_ACC_COMMON` — identify via the SAP Fiori Apps Reference Library entry for the app).
   - Select the tile → **Create Reference** → target `Z_FIN_AP_PROCESSING_BC`.
   - Repeat for the target mapping of the same app. Tile **and** target mapping are both required.
5. Save. Verify the catalog lists each app with a tile and matching target mapping.

> Use references, not copies. A reference inherits SAP updates; a copy freezes the definition and drifts after upgrades.

---

## 4. Create a Custom Tile and Target Mapping

For custom UI5 apps (e.g. `ZFIN_APDASH`):

### 4.1 Target mapping
| Field | Value |
|---|---|
| Semantic object | `SupplierInvoice` (reuse SAP's where meaningful) or `ZVendorRiskReview` |
| Action | `analyzeRisk` |
| Application type | SAPUI5 Fiori App |
| Title | Vendor Risk Review |
| URL | `/sap/bc/ui5_ui5/sap/zfin_apdash` |
| ID (component) | `binary.fin.apdash` |
| Device types | Desktop, Tablet |
| Parameters | `CompanyCode` (optional, default from user parameter `BUK`) |
| Allow additional parameters | Ignored / Allowed per app needs |

### 4.2 Tile
| Field | Value |
|---|---|
| Type | Dynamic (count of high-risk vendors) |
| Title / Subtitle | Vendor Risk Review / High-risk vendors |
| Icon | `sap-icon://warning` |
| Service URL (dynamic) | `/sap/opu/odata/sap/ZFIN_AP_DASHBOARD_SRV/HighRiskVendors/$count` |
| Refresh interval | 300 s (never below 60 s) |
| Navigation | Semantic object `ZVendorRiskReview`, action `analyzeRisk` |

### 4.3 Classic GUI transaction (SAP GUI for HTML)
| Field | Value |
|---|---|
| Application type | Transaction |
| Transaction | e.g. `FBL1N` |
| System alias | `LOCAL` (or the mapped alias) |
| UI technology | SAP GUI for HTML |

---

## 5. Create a PFCG Business Role

1. `PFCG` → Role `Z_FIN_AP_CLERK_BR` → **Single Role** → description "AP Clerk – Business Role".
2. **Menu** tab → **Insert node** → **SAP Fiori Launchpad** →
   - **Launchpad Catalog**: add `Z_FIN_AP_PROCESSING_BC`, `Z_FIN_VENDOR_DISPLAY_BC`, `Z_FIN_AP_REPORTING_BC`
   - **Launchpad Space**: add `Z_FIN_AP_SP`
3. **Authorizations** tab → **Change authorization data**. Proposals from SU24 load for the services and transactions in the catalogs (`S_SERVICE`, `S_START`, `S_TCODE`, plus business objects).
4. Maintain org levels (company code, etc.) or leave open for derivation in derived roles.
5. Generate profile. Save to the transport.
6. **User** tab: assign test users in DEV/QAS only. Perform user comparison.

Detailed design: [sample-pfcg-role-design.md](../samples/sample-pfcg-role-design.md).

---

## 6. Create Spaces and Pages

### 6.1 Page
1. Open **Manage Launchpad Pages** → **Create**.
2. Page ID `Z_FIN_AP_PG`, title "Accounts Payable", transport package `ZFIN_FLP`, customizing request.
3. In the page editor, add sections: "Process Invoices", "Payments", "Supplier Information", "Analytics". (Layout spec: [sample-space-page-layout.md](../samples/sample-space-page-layout.md).)
4. Add visualizations from the **role context** of `Z_FIN_AP_CLERK_BR` so only apps from the role's catalogs are offered.
5. Choose tile vs. link display per visualization. Save.

### 6.2 Space
1. Open **Manage Launchpad Spaces** → **Create**.
2. Space ID `Z_FIN_AP_SP`, title "Accounts Payable".
3. Assign page `Z_FIN_AP_PG`. Save on the same transport.

### 6.3 Assign
Add the space to the role menu (§5 step 2). A space is visible only when its role is assigned.

---

## 7. Configure Notifications & Workflow Inbox

- Activate notification channel services (`/IWNGW/NOTIFICATION_SRV` / V4 equivalents per release) and the **My Inbox** app for approvers.
- Configure workflow scenarios for supplier invoice approval to raise launchpad notifications.
- Include **My Inbox** in `Z_FIN_AP_APPROVAL_BC` only.

---

## 8. Configure Search

- Enable Enterprise Search / app search on the launchpad.
- App search returns only apps authorized via roles; business object search requires the search models to be activated and the respective authorizations.

---

## 9. Post-Configuration Checks

| Check | Tool | Expected |
|---|---|---|
| Catalog consistency | `/UI2/FLC` | No errors for the new catalog/role |
| Intent resolution | `/UI2/FLIA` | Intent resolves to intended target mapping for test user |
| Role authorizations | `PFCG` → status | Profile generated, no open fields |
| Launchpad render | Browser | Space and page visible, all tiles launch |
| OData | `/IWFND/ERROR_LOG` | No errors from smoke test |

---

## 10. Common Configuration Mistakes

| Mistake | Effect | Prevention |
|---|---|---|
| Tile added without target mapping | Tile visible but "could not open app" | Always reference both |
| Catalog maintained in CONF instead of CUST | Wrong scope, unexpected transport type | Use `/UI2/FLPD_CUST` for business catalogs |
| Page contains a tile from a catalog not in the role | Tile hidden on page | Build pages in role context |
| Copying SAP tiles | Upgrades don't propagate | Use references |
| Forgetting SU24 proposals for custom services | 403 on OData | Maintain SU24 for every custom service |
| Dynamic tile refresh too frequent | Gateway load | Minimum 60 s, prefer static where possible |

---

## Related Documents

- [Catalog & Role Design](catalog-and-role-design.md)
- [Spaces & Pages Strategy](spaces-and-pages-strategy.md)
- [Security & Authorization](security-and-authorization.md)
