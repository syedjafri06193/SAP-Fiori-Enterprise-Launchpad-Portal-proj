# Backend Integration – BINARY

## 1. Overview

Fiori apps reach S/4HANA business logic through SAP Gateway using OData. This document covers the OData service lifecycle, Gateway configuration, connectivity, security controls and validation.

```
UI5 app ──HTTP(S)──▶ /sap/opu/odata(4)/... ──▶ Gateway hub layer (/IWFND)
                                                 │ system alias (LOCAL)
                                                 ▼
                                   Backend layer (/IWBEP) ──▶ CDS / RAP / ABAP logic ──▶ HANA
```

---

## 2. OData V2 vs. V4

| Aspect | OData V2 | OData V4 |
|---|---|---|
| Registration | Per service in `/IWFND/MAINT_SERVICE` | Per service group in `/IWFND/V4_ADMIN` |
| Start authorization | `S_SERVICE` | `S_START` |
| Typical source | SEGW projects, CDS `@OData.publish`, older SAP apps | RAP service bindings, newer SAP apps |
| URL | `/sap/opu/odata/sap/<SERVICE>` | `/sap/opu/odata4/sap/<group>/srvd/sap/<service>/<version>` |
| Logs | `/IWFND/ERROR_LOG`, `/IWBEP/ERROR_LOG` | Same plus V4 admin traces |

---

## 3. OData Service Lifecycle

| Step | Activity | Tool | Transported? |
|---|---|---|---|
| 1 | Identify required services per app (Fiori Apps Reference Library → implementation info) | – | – |
| 2 | Register/activate V2 service with system alias | `/IWFND/MAINT_SERVICE` | Yes (workbench request, package `ZFIN_FLP_GW`) |
| 3 | Publish V4 service group | `/IWFND/V4_ADMIN` | Yes |
| 4 | Maintain SU24 for custom services | `SU24` | Yes |
| 5 | Add service authorization to roles | `PFCG` | Yes |
| 6 | Test in Gateway client | `/IWFND/GW_CLIENT` | – |
| 7 | Import QAS → test → PRD | `STMS` | – |
| 8 | Post-import: clear metadata caches | `/IWFND/CACHE_CLEANUP`, `/IWBEP/CACHE_CLEANUP` | – |
| 9 | Decommission: deactivate unused services | `/IWFND/MAINT_SERVICE` | Yes |

### Versioning
- Service versions are kept explicit (`;v=0002`) when an incompatible change is introduced.
- Old versions remain active until all consuming apps are migrated, then are deactivated.

---

## 4. Gateway Configuration

### 4.1 System alias
| Alias | Target | Use |
|---|---|---|
| `LOCAL` | Same system (embedded) | All S/4HANA standard and custom services |
| `Z_BW_RFC` (example) | Remote analytics system via trusted RFC | Only if remote analytical services are consumed |

Rules:
- Same alias names in DEV, QAS and PRD so service registrations transport unchanged; the alias **target** is maintained per system.
- In embedded deployment, prefer `LOCAL` / local processing to avoid unnecessary RFC hops.

### 4.2 Trusted RFC (only for remote aliases)
- `SMT1` trust between systems; `SM59` destination with "Current user" logon and trust enabled.
- `S_RFCACL` granted to users who use trusted RFC.
- Destination tested with **Authorization test** in `SM59`.

### 4.3 HTTPS enforcement
- ICM HTTPS port configured; HTTP disabled externally.
- `STRUST` SSL server PSE valid; certificate expiry monitored (see [Operations Runbook](operations-runbook.md)).

### 4.4 Performance settings
- Gateway metadata cache active in PRD (default).
- Soft-state only for services designed for it.
- `$batch` used by UI5 apps by default; no custom changes.

---

## 5. Custom Service Standards

For custom services (e.g. `ZFIN_AP_DASHBOARD_SRV`):

- Built from CDS views (`@OData.publish` or RAP service binding) wherever possible.
- Authorization checks implemented through **CDS access controls (DCL)** referencing the same business objects as standard apps (e.g. `F_BKPF_BUK`).
- No direct table access from the UI; no generic `SELECT *` services.
- Paging (`$top`/`$skip`) and server-side filtering mandatory for lists.
- `$count` support for dynamic tiles.
- SU24 entries maintained.

---

## 6. Security Controls

| Control | Implementation |
|---|---|
| Role-based access only | `S_SERVICE` / `S_START` + business objects in PFCG roles |
| No direct table access | All data via services with DCL/authority checks |
| Encrypted transport | HTTPS / TLS 1.2+ |
| CSRF protection | X-CSRF-Token on modifying requests |
| Minimal surface | Only services used by in-scope apps are active |
| Audit | Service activations transported with approvals; change logs retained |

---

## 7. Single Sign-On to Backend

- In the embedded deployment, the SAML session established at the launchpad is valid for the OData calls (same system).
- For remote aliases, trusted RFC passes the user identity; no shared technical users for dialog requests.

---

## 8. Validation

### 8.1 Per service
1. `/IWFND/MAINT_SERVICE` → service → **SAP Gateway Client** → `GET $metadata` returns 200.
2. Execute a representative entity set read with filter, as a test user with the business role.
3. Confirm `/IWFND/ERROR_LOG` shows no entries.

### 8.2 End-to-end
- Each tile executed end-to-end by role in QAS (see [Testing Strategy](testing-strategy.md)).
- Response times recorded via browser dev tools and `/IWFND/STATS`.
- Functional sign-off by the process owner before PRD deployment.

### 8.3 Across systems
- After each import, a smoke test confirms service availability in the target system.
- A service inventory report (services active in DEV vs QAS vs PRD) is compared to detect drift.

---

## 9. Common Integration Issues

| Symptom | Likely cause | Resolution |
|---|---|---|
| 403 Forbidden on OData | Missing `S_SERVICE`/`S_START` | SU53 / STAUTHTRACE → add via SU24 + role |
| 404 / "No service found for namespace" | Service not activated in this system | Activate/transport in `/IWFND/MAINT_SERVICE` |
| 500 + "No system alias found" | Alias missing or wrong mapping | Maintain system alias for the service |
| Stale metadata after transport | Metadata cache | `/IWFND/CACHE_CLEANUP` and `/IWBEP/CACHE_CLEANUP` |
| Slow responses | Unfiltered reads, expensive CDS | `/IWFND/STATS`, `ST05`, optimize filters/CDS |
| RFC error for remote alias | Trust or destination issue | `SM59` test, `SMT1` trust, `S_RFCACL` |

More: [Troubleshooting Guide](troubleshooting-guide.md#3-odata-service-errors).

---

## Related Documents

- [Fiori Launchpad Architecture](fiori-launchpad-architecture.md)
- [Security & Authorization](security-and-authorization.md)
- [Performance Optimization](performance-optimization.md)
