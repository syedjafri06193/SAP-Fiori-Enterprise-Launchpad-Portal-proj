# Governance & Naming Standards – BINARY

## 1. Purpose

Defines who owns launchpad content, how decisions are made, and the naming conventions every object must follow. Consistent naming makes roles auditable, searchable and easy to support.

---

## 2. Naming Conventions

### 2.1 General rules
- Customer namespace prefix `Z_` for all objects.
- Domain segment: `FIN` (finance business content), `TEC` (technical/admin content).
- Uppercase, underscores only, no spaces.
- Suffix identifies the object type.
- Keep IDs ≤ 35 characters where the object type allows (spaces/pages IDs are limited).

### 2.2 Object patterns

| Object | Pattern | Example |
|---|---|---|
| PFCG business role | `Z_FIN_<FUNCTION>_BR` | `Z_FIN_AP_CLERK_BR` |
| Composite role (if used) | `Z_FIN_<FUNCTION>_CR` | `Z_FIN_AP_LEAD_CR` |
| Derived role (org-level) | `Z_FIN_<FUNCTION>_<CC>_DR` | `Z_FIN_AP_CLERK_1000_DR` (company code 1000) |
| Business catalog | `Z_FIN_<PROCESS>_BC` | `Z_FIN_AP_PROCESSING_BC` |
| Technical catalog | `Z_TEC_<AREA>_TC` | `Z_TEC_FIN_CUSTOM_APPS_TC` |
| Space | `Z_FIN_<ROLE>_SP` | `Z_FIN_AP_SP` |
| Page | `Z_FIN_<ROLE>_PG` (or `_PG<NN>` if multiple) | `Z_FIN_AP_PG`, `Z_FIN_GL_PG02` |
| Custom semantic object | `Z<Object>` (CamelCase) | `ZVendorRiskReview` |
| Custom OData service | `Z<AREA>_<PURPOSE>_SRV` | `ZFIN_AP_DASHBOARD_SRV` |
| Custom UI5 app (BSP) | `ZFIN_<APP>` | `ZFIN_APDASH` |
| Transport description | `FLP | <ticket> | <short text>` | `FLP | CHG004512 | AP catalog tile update` |

### 2.3 Titles shown to users
- Space/page titles in business language: "Accounts Payable", not "Z_FIN_AP_SP".
- Section titles describe tasks: "Process Invoices", "Approve Payments".
- Tile titles: keep SAP standard titles unless finance requests a change, documented in the catalog spec.
- Provide translations (EN plus any additional logon languages) via the translation tools (`SE63`) or the apps' text fields.

---

## 3. Ownership

| Content | Business owner | Technical owner |
|---|---|---|
| AP / AR catalogs and spaces | Head of AP / AR | Fiori team |
| GL / period close | Group Financial Controller | Fiori team |
| Treasury | Treasurer | Fiori team |
| Controlling | Head of Controlling | Fiori team |
| Audit display role | Head of Internal Audit | Security team |
| Admin/technical catalogs | Fiori lead | Fiori team |
| All PFCG roles | Process owner (approves) | Security team (builds) |

Business owners approve content changes and quarterly access reviews for their roles.

---

## 4. Design Authority

A **Fiori Design Authority** (Fiori lead, security lead, Basis lead, one finance representative) meets bi-weekly and approves:

- New business catalogs or roles
- New spaces/pages or major layout changes
- Activation of new OData services
- Exceptions to naming standards or SoD rules (with documented mitigation)

Minor changes (adding an SAP-standard tile to an existing catalog for the same process, title/translation fixes) are approved by the content's business owner and the Fiori lead.

---

## 5. Content Lifecycle

```
Request → Assess (DA) → Design → Build (DEV) → Test (QAS) → Approve (CAB) → Deploy (PRD) → Review
```

| Stage | Exit criteria |
|---|---|
| Request | Ticket with business justification and requesting persona |
| Assess | Fit with existing catalogs checked; SoD impact assessed |
| Design | Catalog/role/page spec updated in this repository's format |
| Build | Objects created in DEV on a transport; unit tested |
| Test | Integration + authorization tests passed in QAS |
| Approve | Change Advisory Board approval, business owner sign-off |
| Deploy | Imported to PRD; caches invalidated; smoke tested |
| Review | Usage reviewed after 90 days; unused tiles removed |

### Decommissioning
- Tiles with zero usage for two quarters are proposed for removal.
- Retired catalogs are removed from roles first, then deleted in a later transport.
- Inactive OData services are deactivated to reduce the attack surface.

---

## 6. Documentation Standards

- Every catalog, role and space has a spec following the templates in [`/samples`](../samples).
- Specs are updated in the same change as the configuration.
- `CHANGELOG.md` records every release of launchpad content.

---

## Related Documents

- [Catalog & Role Design](catalog-and-role-design.md)
- [Transport & Change Management](transport-and-change-management.md)
