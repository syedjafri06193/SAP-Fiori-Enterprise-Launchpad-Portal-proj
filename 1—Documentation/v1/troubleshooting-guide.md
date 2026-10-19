# SAP Fiori Troubleshooting Guide – BINARY

Symptom-driven playbooks for the most common launchpad incidents. Work top-down: confirm the layer (content, authorization, service, backend, browser) before changing anything.

---

## 0. First Response Checklist

Collect before diagnosing:

- [ ] User ID, system, time of error (with timezone)
- [ ] Space/page and tile name, or the app's intent (URL hash after `#`, e.g. `#SupplierInvoice-manage`)
- [ ] Screenshot of the error and the full message text
- [ ] Browser and version; reproducible in incognito?
- [ ] Does another user with the same role see the issue?

### Diagnostic tool map

| Layer | Tools |
|---|---|
| Content | `/UI2/FLPD_CUST`, `/UI2/FLPCM_CUST`, `/UI2/FLIA`, `/UI2/FLC`, Manage Launchpad Pages |
| Authorization | `SU53`, `STAUTHTRACE`, `SUIM`, `PFCG`, `SU01` |
| Gateway | `/IWFND/ERROR_LOG`, `/IWBEP/ERROR_LOG`, `/IWFND/GW_CLIENT`, `/IWFND/TRACES`, `/IWFND/MAINT_SERVICE`, `/IWFND/V4_ADMIN` |
| Backend | `ST22`, `SM21`, `ST05`, `SAT`, `SLG1` |
| Browser | DevTools Console + Network, `sap-ui-debug=true`, UI5 diagnostics (`Ctrl+Alt+Shift+S`) |
| Caches | `/UI2/INVALIDATE_GLOBAL_CACHES`, `/IWFND/CACHE_CLEANUP`, `/UI5/APP_INDEX_CALCULATE` |

---

## 1. Application Does Not Launch

**Symptoms:** "Could not open app", "Navigation to this application is not supported", blank screen after clicking a tile.

| Step | Action | What to look for |
|---|---|---|
| 1 | Verify PFCG role assignment in `SU01` (Roles tab, validity dates) | Role present and valid; user comparison done |
| 2 | Run `/UI2/FLIA` for the user and intent | Does the intent resolve? Which catalog/target mapping wins? |
| 3 | Check target mapping in the catalog | Correct semantic object/action, app ID/URL, device types include the user's device |
| 4 | Browser console | `Failed to load component`, 404 on `Component-preload.js` → app not deployed/app index stale |
| 5 | Execute `SU53` immediately after the error (or `STAUTHTRACE`) | Missing `S_TCODE`, `S_START`, `S_SERVICE` |
| 6 | For GUI transactions: check `/sap/bc/gui/sap/its/webgui` active in `SICF` | 403/404 on webgui |

**Common fixes:**
- Add missing target mapping reference to the business catalog.
- Run `/UI5/APP_INDEX_CALCULATE` for the app after deployment.
- Add device type "Desktop" to target mapping.
- Add authorization via SU24 → role regeneration.

---

## 2. Missing Tiles

**Symptoms:** Tile not on page, or not found in App Finder.

```
Is the app in App Finder?
├── No  → catalog not in user's roles / target mapping missing / role not assigned → check PFCG + catalog
└── Yes → catalog OK; page doesn't contain the visualization, or page not in user's space
```

| Step | Action |
|---|---|
| 1 | Confirm the business catalog is in the role menu (`PFCG` → Menu) |
| 2 | Confirm user has the role and comparison is current (`SU01`, `PFUD`) |
| 3 | Check the catalog has both **tile and target mapping** for the app |
| 4 | Check the page contains the visualization and the space is in the role |
| 5 | Confirm the transport is imported in this system (`STMS` import history) |
| 6 | Invalidate caches: `/UI2/INVALIDATE_GLOBAL_CACHES`; ask user to reload (Ctrl+F5) |
| 7 | Run `/UI2/FLC` for the role to detect broken references |

**Note:** A visualization on a page is hidden when the user's roles don't contain a catalog with that app — layout and authorization are independent by design.

---

## 3. OData Service Errors

| HTTP status / message | Meaning | Action |
|---|---|---|
| **403 Forbidden** | Missing `S_SERVICE` (V2) or `S_START` (V4) | `SU53` / `STAUTHTRACE` → SU24 + role |
| **404 / "No service found for namespace…"** | Service not activated in this system | `/IWFND/MAINT_SERVICE` → activate (transport from DEV) |
| **500 "No system alias found"** | Alias missing for service | Add system alias in `/IWFND/MAINT_SERVICE` |
| **500 / 501 runtime error** | Backend exception | `/IWBEP/ERROR_LOG`, `ST22` dump |
| **"Metadata could not be loaded"** | Stale cache or broken service | `/IWFND/CACHE_CLEANUP` + `/IWBEP/CACHE_CLEANUP`; test `$metadata` in GW client |
| **CSRF token validation failed** | Session/token issue, proxy stripping headers | Reload; check Web Dispatcher/proxy header handling |

Procedure:
1. Open `/IWFND/ERROR_LOG`, filter by user and time.
2. Open the entry → **Replay in Gateway client** to reproduce.
3. Use the "Backend error log" link to jump to `/IWBEP/ERROR_LOG`.
4. Verify activation in `/IWFND/MAINT_SERVICE` (V2) or `/IWFND/V4_ADMIN` (V4).
5. Validate system alias and (for remote aliases) RFC destination in `SM59`.

---

## 4. Authorization Errors

| Step | Tool | Notes |
|---|---|---|
| 1 | `SU53` | Run directly after the failure; in Fiori, open `SU53` for the user from GUI (`SU53` → "Display for other user") |
| 2 | `STAUTHTRACE` | Preferred for Fiori: system-wide trace filtered by user; reproduce, then evaluate failed checks |
| 3 | `SUIM` | Which roles contain the needed object/value? Is it SoD-relevant? |
| 4 | `ST22` | Dumps caused by missing authorization in custom code |
| 5 | `SM21` | System log entries around the time of failure |
| 6 | `SLG1` | Application logs for apps that log authorization messages |

**Fix process:** never add authorizations directly in PRD. Update SU24/role in DEV → SoD check → transport.

**Business-data symptoms:** a list showing *no* results (rather than an error) often means business-object authorization (e.g. company code) filters out data. Check `F_BKPF_BUK` values in the derived role.

---

## 5. Launchpad Does Not Load / Logon Issues

| Symptom | Check |
|---|---|
| Redirect loop to IdP | `SAML2` configuration, NameID mapping, clock skew between IdP and SAP |
| "Logon failed" | User locked/expired in `SU01`, NameID not mapped |
| Blank shell, console errors | `/sap/bc/ui2/flp` or `/sap/public/bc/ui5_ui5` inactive in `SICF` |
| "Error while loading the launchpad" | Start-up service error → `/IWFND/ERROR_LOG`, `ST22`; `/UI2/FLC` for user's roles |
| Works in incognito only | Browser cache/extensions; clear cache |

---

## 6. Performance Issues

| Step | Action |
|---|---|
| 1 | Browser DevTools → Network: identify slowest requests (start-up vs. `$count` vs. app data) |
| 2 | Count visualizations and dynamic tiles on the user's page |
| 3 | `/IWFND/STATS`: response time per service, hub vs. backend split |
| 4 | `ST05` SQL trace / `SAT` for slow backend services |
| 5 | Check app index job and UI5 cache buster |
| 6 | Coordinate with Basis on work process load (`SM50`/`SM66`), ICM (`SMICM`) and network |

See [Performance Optimization](performance-optimization.md).

---

## 7. Tile Shows Wrong Number / "Cannot Load Tile"

| Cause | Fix |
|---|---|
| `$count` service returns error (often 403) | Add service authorization; test URL in GW client |
| Service URL wrong in tile config | Correct in catalog |
| Timeout | Optimize service; increase refresh interval |
| Data correct but cached | Wait for refresh interval or reload page |

---

## 8. Changes Not Visible After Transport

1. Confirm transport imported (`STMS` → import history) with return code ≤ 4.
2. `/UI2/INVALIDATE_GLOBAL_CACHES`.
3. `/IWFND/CACHE_CLEANUP` + `/IWBEP/CACHE_CLEANUP` if services changed.
4. `/UI5/APP_INDEX_CALCULATE` if UI5 app changed.
5. User comparison for roles (`PFUD`).
6. User hard-reloads (Ctrl+F5) or clears cache.

---

## 9. Translation / Text Issues

- Tile shows technical ID: missing translation for logon language → maintain texts in catalog/page in that language.
- Mixed languages: user's logon language vs. browser language; use `sap-language` URL parameter to test.

---

## 10. Escalation

| Level | Team | When |
|---|---|---|
| L1 | Service desk | Collect info (§0), known-error fixes (cache, reload) |
| L2 | Fiori support team | Content, authorization, Gateway diagnosis |
| L3 | Fiori lead / Basis / Security / Functional | Design changes, system issues, backend defects |
| Vendor | SAP support (incident on component `CA-FLP-*`, `OPU-GW-*`, etc.) | Product defects; check SAP Notes first |

---

## Related Documents

- [Operations Runbook](operations-runbook.md)
- [Security & Authorization](security-and-authorization.md)
- [Backend Integration](integration-with-backend.md)
