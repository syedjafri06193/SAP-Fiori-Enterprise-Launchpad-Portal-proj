# Testing Strategy – BINARY

## 1. Test Levels

| Level | System | Who | Goal |
|---|---|---|---|
| Unit test | DEV | Fiori consultant | Each tile/target mapping launches; page renders |
| Integration test | QAS | Fiori + functional consultants | Apps work end-to-end with backend data and services |
| Authorization test | QAS | Security analyst | Each role can do exactly what it should — and nothing more |
| Performance test | QAS | Fiori + Basis | Load time and service response targets met |
| UAT | QAS | Finance key users | Business processes work on the new launchpad |
| Regression | QAS | Fiori team (+ automation) | Existing roles unaffected by changes/upgrades |
| Smoke test | PRD | Support | Post-import sanity check |

---

## 2. Entry & Exit Criteria

| Level | Entry | Exit |
|---|---|---|
| Integration | Transports imported to QAS, caches cleared | All P1/P2 defects fixed; ≥ 95% test cases passed |
| Authorization | Roles generated in QAS, test users assigned | 100% positive and negative cases passed |
| UAT | Integration + authorization passed | Signed by process owner for each persona |
| Go-live | UAT signed, CAB approved | Smoke tests passed in PRD |

---

## 3. Integration Testing

For every catalog:
1. Log on as a test user holding the role.
2. Launch each tile from the page **and** via App Finder.
3. Execute the core function (display, create, change) with QAS test data.
4. Verify cross-app navigation (e.g. from Supplier Line Items to Supplier Invoice) resolves to the intended target.
5. Record OData errors from `/IWFND/ERROR_LOG`.

---

## 4. Authorization Testing

### 4.1 Positive tests
- Run `STAUTHTRACE` for the test user.
- Execute each tile and its key actions.
- Confirm no authorization failures (return code ≠ 0) in the trace for intended actions.

### 4.2 Negative tests
Verify that a role **cannot**:

| Role | Must not be able to |
|---|---|
| AP Clerk | Create/change supplier master, run payment program, release blocked invoices |
| AP Manager | Change supplier bank details |
| AR Clerk | Change customer master credit data |
| GL Accountant | Open/close posting periods |
| Cash Manager | Approve payments they created |
| Auditor | Post or change any document |

Test also for company codes outside the derived role's org level.

### 4.3 Evidence
STAUTHTRACE extract, screenshots of denied actions, and the signed test log go into the audit evidence pack.

---

## 5. Performance Testing

| Measure | Target | Method |
|---|---|---|
| Launchpad first load (cold cache) | P90 ≤ 4 s | Browser dev tools, 10 runs per persona |
| Launchpad warm load | P90 ≤ 2 s | Same |
| Dynamic tile `$count` | ≤ 500 ms | `/IWFND/STATS`, network tab |
| App first data load | ≤ 3 s | Network tab |

Details: [Performance Optimization](performance-optimization.md).

---

## 6. User Acceptance Testing

- Test scripts per persona derived from real daily tasks (see [sample-test-scripts.md](../samples/sample-test-scripts.md)).
- Key users execute in QAS with realistic data.
- Defects logged with severity, persona, app, steps and screenshot.
- Sign-off form per persona by the business owner.

### Defect severity
| Severity | Definition | Fix required before go-live? |
|---|---|---|
| P1 – Critical | Process blocked, no workaround | Yes |
| P2 – High | Process impaired, workaround painful | Yes |
| P3 – Medium | Workaround acceptable | Agreed case by case |
| P4 – Low | Cosmetic | No |

---

## 7. Regression Testing

Triggered by:
- SAP_UI / S/4HANA upgrades and support packages
- SAP Note implementation affecting launchpad or Gateway
- Changes to shared catalogs (e.g. `Z_FIN_REPORTING_BC`)

Regression suite:
- Launch every tile per role (scripted where possible with OPA5/wdi5 or a test automation tool)
- `/UI2/FLC` checks for all `Z_FIN_*` roles
- Service inventory comparison DEV/QAS/PRD

---

## 8. Smoke Test (PRD)

After every import:
1. Log on with a smoke-test user per changed role (PRD read-only test IDs).
2. Open the space/page; verify changed tiles render and launch.
3. Check `/IWFND/ERROR_LOG` and `ST22` for new entries.
4. Record result in the change ticket.

---

## Related Documents

- [Sample Test Scripts](../samples/sample-test-scripts.md)
- [Security & Authorization](security-and-authorization.md)
- [Transport & Change Management](transport-and-change-management.md)
