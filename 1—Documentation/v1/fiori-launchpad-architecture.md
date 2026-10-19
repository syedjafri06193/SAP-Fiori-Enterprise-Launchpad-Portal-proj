# SAP Fiori Launchpad Architecture – BINARY

## 1. Overview

At BINARY, the SAP Fiori Launchpad (FLP) is the single entry point to SAP S/4HANA for finance users. It delivers role-based access to SAPUI5 apps, classic SAP GUI for HTML transactions and analytical apps, all governed by PFCG roles and secured by SSO over HTTPS.

Architectural goals:

- **Secure**: role-based access, no direct database access from the UI layer, encrypted transport
- **Performant**: cached content, lean catalogs, efficient OData calls
- **Auditable**: every content and configuration change travels DEV → QAS → PRD
- **Maintainable**: SAP-standard content model, clear naming and ownership

---

## 2. Deployment Option

BINARY uses the **embedded deployment**: the front-end server components (SAP_UI, Gateway foundation) run in the same S/4HANA system as the business logic.

| Option | Description | Decision |
|---|---|---|
| Embedded | FES and Gateway inside the S/4HANA system | **Selected** – SAP's recommended default for S/4HANA; no extra system, local OData calls, simpler RFC setup |
| Hub (standalone FES) | Separate Gateway/FES system in front of one or more backends | Rejected – adds a system to patch and secure; only justified for multi-backend aggregation |
| SAP BTP (Build Work Zone) | Cloud launchpad federating on-prem content | Future option for a cross-system portal |

Consequences of the embedded choice:

- OData system alias points to the local system (`LOCAL` / no RFC hop for most services).
- Front-end and back-end authorizations live in the same system, so a single PFCG role carries both.
- UI component upgrades are coupled with S/4HANA upgrades.

---

## 3. Components

### 3.1 Fiori Front-End (SAP_UI / UI services)
- Serves the launchpad shell (`/sap/bc/ui2/flp`) and SAPUI5 libraries (`/sap/public/bc/ui5_ui5`)
- Hosts app repositories (BSP/UI5 ABAP repository) and the app index
- Stores launchpad content: catalogs, target mappings, tiles, spaces, pages
- Provides personalization, themes and launchpad settings

### 3.2 SAP Gateway (Foundation)
- Exposes OData V2 services (registered in `/IWFND/MAINT_SERVICE`) and OData V4 service groups (published in `/IWFND/V4_ADMIN`)
- Routes requests through system aliases to the backend implementation
- Provides logs (`/IWFND/ERROR_LOG`, `/IWBEP/ERROR_LOG`), traces (`/IWFND/TRACES`) and statistics (`/IWFND/STATS`)

### 3.3 S/4HANA Backend
- Business logic in ABAP, CDS views and RAP business objects
- FI, CO and Treasury master and transactional data on SAP HANA
- Back-end authorization checks (e.g. `F_BKPF_BUK`, `F_BKPF_BLA`, `K_CCA`)

### 3.4 Identity & Access
- Corporate IdP issues SAML 2.0 assertions; the ABAP system is the service provider (`SAML2` transaction)
- PFCG business roles carry launchpad content and authorizations
- User provisioning via the access request process (see [User Onboarding](user-onboarding-and-training.md))

### 3.5 Web Dispatcher / Load Balancer
- SAP Web Dispatcher terminates external HTTPS, routes to application servers, and enforces URL filtering
- Only `/sap/bc/ui2/flp`, `/sap/bc/ui5_ui5`, `/sap/opu/odata`, `/sap/opu/odata4`, `/sap/bc/gui/sap/its/webgui` and required public paths are exposed

---

## 4. Request Flow

```
1. User opens https://fiori.binary.example/sap/bc/ui2/flp
2. Web Dispatcher routes to an ABAP application server (ICM)
3. No session → redirect to IdP → SAML assertion → session cookie (MYSAPSSO2/SAP_SESSIONID)
4. FLP shell loads; start-up service returns the user's spaces, pages and visualizations
   (computed from the user's PFCG roles → catalogs → tiles/target mappings)
5. User clicks a tile → intent  #SupplierInvoice-manage
6. Shell resolves intent via target mapping → SAPUI5 component / GUI transaction
7. App loads UI5 resources (cached) and calls OData:
      /sap/opu/odata/sap/<SERVICE>/...   (V2)
      /sap/opu/odata4/sap/<group>/...    (V4)
8. Gateway checks S_SERVICE (V2) / S_START (V4) → dispatches to backend
9. Backend checks business authorizations (e.g. F_BKPF_BUK) → returns data
```

Authorization is enforced at three points:

| Layer | Check | Failure symptom |
|---|---|---|
| Launchpad content | Catalog/space assigned via role | Tile missing |
| Gateway | `S_SERVICE` / `S_START` for the service | HTTP 403 on OData call |
| Business logic | Business authorization objects | Empty results or "No authorization" message |

---

## 5. Launchpad Content Model

| Entity | Purpose | Maintained in |
|---|---|---|
| **Business catalog** | Set of apps (tiles + target mappings) for a business process; the unit of authorization | `/UI2/FLPD_CUST` (Launchpad Designer) or Launchpad App Manager |
| **Technical catalog** | Repository of tiles/target mappings from which business catalogs reference | SAP-delivered (`SAP_TC_*`) and `Z_TEC_*` |
| **Target mapping** | Maps an intent (semantic object + action) to an app | Catalog |
| **Tile** | Visual launch point (static, dynamic, KPI) | Catalog |
| **Space** | Container of pages assigned to a role | Manage Launchpad Spaces app |
| **Page** | Layout of sections with visualizations | Manage Launchpad Pages app |
| **PFCG business role** | Assigns catalogs and spaces + authorizations to users | `PFCG` |

Principles:

- Business catalogs **reference** tiles from technical catalogs rather than copying them, so SAP updates flow through.
- A tile can appear on a page only if the role also grants the catalog containing it.
- Roles are composed from catalogs; users get roles, never catalogs directly.

---

## 6. Caching Architecture

| Cache | Location | Invalidated by |
|---|---|---|
| Browser cache (UI5 resources) | User browser | Cache-buster tokens on UI5 URLs |
| FLP start-up/content cache | Front-end server | `/UI2/INVALIDATE_GLOBAL_CACHES`, transport import, role changes |
| Client-specific caches | Front-end server | `/UI2/INVALIDATE_CLIENT_CACHES` |
| Gateway metadata cache | Gateway | `/IWFND/CACHE_CLEANUP`, `/IWBEP/CACHE_CLEANUP` |
| App index | Front-end server | `/UI5/APP_INDEX_CALCULATE` |

Cache invalidation is part of every post-import procedure (see [Transport & Change Management](transport-and-change-management.md)).

---

## 7. System Landscape

| Tier | SID | Client | Purpose |
|---|---|---|---|
| Development | S4D | 100 | Configuration, content creation, unit tests |
| Quality | S4Q | 100 | Integration, authorization, regression and UAT |
| Production | S4P | 100 | Live use; locked for changes |

All Fiori content, roles and Gateway service registrations are transported DEV → QAS → PRD. Details: [System Landscape & Setup](system-landscape-and-setup.md).

---

## 8. Security Architecture

- **Transport encryption**: HTTPS only (TLS 1.2+); HTTP ports disabled for external access
- **Authentication**: SAML 2.0 SSO; password logon disabled for dialog users on HTTP
- **Authorization**: least-privilege PFCG roles; SoD validated (see [Security & Authorization](security-and-authorization.md))
- **Surface reduction**: only required ICF nodes and OData services are active
- **Clickjacking protection** enabled; whitelist only trusted framing origins
- **No direct DB access** from the UI layer; all data via OData and authorization-checked logic
- **Secure RFC** with trusted relationships where remote calls are needed

---

## 9. High Availability & Scalability

- Multiple application servers behind the Web Dispatcher with logon groups for HTTP
- Stateless OData calls allow load distribution
- UI5 resources served with long cache lifetimes to reduce server load
- Capacity reviewed before quarter-end close

---

## 10. Architecture Decisions Log

| ID | Decision | Rationale |
|---|---|---|
| AD-01 | Embedded deployment | SAP recommendation for S/4HANA; simpler landscape |
| AD-02 | Spaces and pages instead of groups | Admin-controlled layout, better performance, SAP's strategic direction |
| AD-03 | Business catalogs reference SAP technical catalogs | Keeps upgrades simple and content consistent |
| AD-04 | SAML 2.0 SSO via corporate IdP | Central identity, MFA at IdP |
| AD-05 | One primary space per persona | Clear navigation; avoids tile duplication across spaces |
| AD-06 | PRD locked for content changes | SOX ITGC requirement |

---

## Related Documents

- [System Landscape & Setup](system-landscape-and-setup.md)
- [Backend Integration](integration-with-backend.md)
- [Performance Optimization](performance-optimization.md)
