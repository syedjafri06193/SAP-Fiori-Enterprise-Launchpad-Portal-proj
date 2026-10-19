# Operations Runbook – BINARY

Day-2 operations for the SAP Fiori Launchpad in production (S4P/100).

---

## 1. Support Model

| Role | Responsibility | Hours |
|---|---|---|
| Service desk (L1) | Ticket intake, first-response checklist, known fixes | 24×5 |
| Fiori support (L2) | Content, authorization and Gateway diagnosis | Business hours; on call during close |
| Basis | System health, certificates, jobs, transports | 24×7 on call |
| Security | Roles, SoD, access reviews | Business hours |
| Fiori lead (L3) | Design changes, escalations | Business hours |

### Service levels
| Priority | Definition | Response | Resolution target |
|---|---|---|---|
| P1 | Launchpad down or finance process blocked for many users | 15 min | 4 h |
| P2 | One role/app unavailable, workaround exists | 1 h | 1 business day |
| P3 | Single user issue | 4 h | 3 business days |
| P4 | Request / cosmetic | 1 business day | Next release |

---

## 2. Daily Tasks

| Task | Tool | Action if abnormal |
|---|---|---|
| Review Gateway errors | `/IWFND/ERROR_LOG`, `/IWBEP/ERROR_LOG` | Group by service; open ticket for recurring errors |
| Check dumps | `ST22` | Investigate Fiori/Gateway-related dumps |
| Check background jobs | `SM37` (app index, cache cleanup) | Restart, investigate failures |
| Check system log | `SM21` | Escalate to Basis |
| Review open Fiori tickets | Ticketing system | Re-prioritize, escalate |

---

## 3. Weekly Tasks

| Task | Tool |
|---|---|
| Top services by response time | `/IWFND/STATS` |
| Error trend review | Error log statistics |
| Release window: import approved transports + post-import steps | `STMS`, see [Transport & Change Management](transport-and-change-management.md#5-post-import-activities) |
| Clean up old error log entries | Gateway log cleanup reports |

---

## 4. Monthly / Periodic Tasks

| Task | Frequency |
|---|---|
| Pre-close health check (load times, error volume, capacity) | 3 days before month-end |
| Firefighter log review | Monthly |
| SSL / SAML certificate expiry check (`STRUST`) | Monthly; alert at 60 days |
| Access recertification support | Quarterly |
| Catalog usage review | Semi-annual |
| Inactive OData service / ICF node review | Quarterly |
| SAP Notes review for `CA-FLP`, `OPU-GW` components | Monthly |

---

## 5. Standard Procedures

### 5.1 Invalidate launchpad caches
1. Run `/UI2/INVALIDATE_GLOBAL_CACHES` in the affected system (execute, confirm).
2. Optionally `/UI2/INVALIDATE_CLIENT_CACHES` for a specific user.
3. Ask users to reload.
Use after imports or when content looks stale. Expect a brief slower first load afterwards.

### 5.2 Clear Gateway metadata cache
1. `/IWFND/CACHE_CLEANUP` → for a specific model or all.
2. `/IWBEP/CACHE_CLEANUP` → same.
Use after service changes or "metadata could not be loaded" errors.

### 5.3 Recalculate app index
1. Run report `/UI5/APP_INDEX_CALCULATE` for a single app or full.
Use after UI5 app deployments or when apps fail to load components.

### 5.4 Grant access (standard request)
1. Approved access request with role and company codes.
2. SoD check at user level.
3. Assign derived role in `SU01`/via provisioning; validity dates set.
4. Notify user with quick reference guide.

### 5.5 Certificate renewal
1. Basis requests new certificate 60 days before expiry.
2. Import in `STRUST` (SSL server PSE) and, for IdP signing certificates, update `SAML2` trusted provider.
3. Test logon with pilot users in QAS, then PRD in a maintenance window.

---

## 6. Incident Handling – Major Incident (P1)

1. Declare major incident; open bridge with Fiori, Basis, Security.
2. Determine scope: all users / one role / one app? One system or all servers?
3. Check recent changes: transports imported in last 24 h (`STMS`), certificates, SAP Notes.
4. Check infrastructure: `SMICM`, `SM50`/`SM66`, Web Dispatcher, IdP status.
5. Mitigate: roll forward fix via emergency change; never edit PRD directly without firefighter process.
6. Communicate status every 30 minutes to finance leads.
7. Post-incident review within 5 business days.

---

## 7. Monitoring & Alerts

| Signal | Source | Threshold |
|---|---|---|
| Launchpad availability | Synthetic check of `/sap/bc/ui2/flp` | 2 consecutive failures |
| Gateway error rate | Error log counts | > 2× daily baseline |
| Dynamic tile service time | `/IWFND/STATS` | > 1 s average |
| Certificate expiry | `STRUST` / monitoring | < 60 days |
| Failed jobs | `SM37` | Any failure |

---

## 8. Close Period Readiness

Before each month-end:
- [ ] No pending non-emergency transports during freeze
- [ ] Load spot-check per finance persona
- [ ] Error log review clean
- [ ] On-call roster confirmed
- [ ] Capacity reviewed with Basis

---

## Related Documents

- [Troubleshooting Guide](troubleshooting-guide.md)
- [Transport & Change Management](transport-and-change-management.md)
- [Performance Optimization](performance-optimization.md)
