# Transport & Change Management – BINARY

## 1. Principles

- **Nothing changes in PRD directly.** All launchpad content, roles and service registrations are transported.
- Every transport links to an approved change ticket.
- Changes are tested in QAS before PRD import.
- Imports avoid month-end/quarter-end close freeze windows.

---

## 2. Transport Routes

```
S4D (DEV, client 100) ──▶ S4Q (QAS, client 100) ──▶ S4P (PRD, client 100)
```

Managed in `STMS`. Imports to PRD are executed by Basis after CAB approval.

---

## 3. What Goes on Which Request

| Object | Request type | Tool where captured |
|---|---|---|
| Business catalogs, tiles, target mappings (CUST) | Customizing | `/UI2/FLPD_CUST` |
| Technical catalogs (CONF) | Workbench | `/UI2/FLPD_CONF` |
| Spaces and pages | Customizing (per release; check prompt in app) | Manage Launchpad Spaces / Pages |
| PFCG roles | Customizing (role transport in `PFCG`) | `PFCG` |
| SU24 proposals | Workbench | `SU24` |
| OData V2 registrations | Workbench | `/IWFND/MAINT_SERVICE` |
| OData V4 service group publishing | Workbench | `/IWFND/V4_ADMIN` |
| Custom UI5 apps / CDS / services | Workbench | ADT / `SE80` |
| Launchpad settings (CUS_CONF) | Customizing | `/UI2/FLP_CUS_CONF` |
| Theme | Workbench (theme transport) | UI Theme Designer |

### Not transportable — per-system checklist
ICF activation, SAML2 configuration, STRUST, system alias targets, user assignments. See [System Landscape & Setup](system-landscape-and-setup.md#4-per-system-activities-not-transportable).

---

## 4. Import Sequence

Within a release, import in dependency order:

1. Development objects (UI5 apps, CDS, services)
2. Gateway service registrations
3. SU24 proposals
4. Technical catalogs
5. Business catalogs
6. Pages, then spaces
7. PFCG roles (including derived roles)
8. Launchpad settings

Bundling these on a **transport of copies** for QAS testing is acceptable; PRD receives the original requests in order.

---

## 5. Post-Import Activities

| Step | Action | Transaction |
|---|---|---|
| 1 | Invalidate launchpad caches | `/UI2/INVALIDATE_GLOBAL_CACHES` (and `/UI2/INVALIDATE_CLIENT_CACHES` where needed) |
| 2 | Clear Gateway metadata cache | `/IWFND/CACHE_CLEANUP`, `/IWBEP/CACHE_CLEANUP` |
| 3 | Recalculate UI5 app index (if apps imported) | `/UI5/APP_INDEX_CALCULATE` |
| 4 | User comparison for changed roles | `PFCG` → user comparison / `PFUD` |
| 5 | Smoke test with test user per changed role | Browser |
| 6 | Check error logs | `/IWFND/ERROR_LOG`, `ST22` |

---

## 6. Change Types

| Type | Examples | Approval | Lead time |
|---|---|---|---|
| **Standard** (pre-approved) | Add SAP-standard tile to existing catalog for same process; translation fix | Business owner + Fiori lead | Next weekly release |
| **Normal** | New catalog, new role, new space/page, new service | Design Authority + CAB | Bi-weekly release |
| **Emergency** | Production-blocking defect | Emergency CAB (2 approvers) | Same day |

### Emergency changes
1. Incident logged with business impact.
2. Fix built in DEV on a separate request; minimal unit test.
3. Emergency CAB approval (IT change manager + business owner).
4. Import DEV → QAS → PRD in quick succession; quick QAS validation.
5. Retrospective review within **2 business days**; documentation updated.

Direct changes in PRD (e.g. via opening client settings) are prohibited. If unavoidable for a P1 outage, they require firefighter access and the same change must be retrofitted into DEV immediately.

---

## 7. Release Calendar

- Weekly release window: Wednesday 18:00 local time.
- Freeze: last 3 and first 5 working days of each month; extended freeze for quarter- and year-end.
- Emergency changes allowed during freezes with CFO delegate approval.

---

## 8. Version Control of Documentation

- This repository is updated in the same change as the configuration.
- `CHANGELOG.md` records content releases with transport numbers.

---

## 9. Transport Checklist (Developer)

- [ ] Ticket number in transport description: `FLP | CHGxxxxxx | text`
- [ ] All objects in correct request type
- [ ] Dependent objects included or already in PRD
- [ ] `/UI2/FLC` checks clean for affected roles
- [ ] Documentation updated
- [ ] Peer review done
- [ ] Release and request import to QAS

---

## Related Documents

- [Governance & Naming Standards](governance-and-naming-standards.md)
- [Testing Strategy](testing-strategy.md)
- [Operations Runbook](operations-runbook.md)
