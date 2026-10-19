# Security & Authorization – BINARY

## 1. Objectives

- Grant users exactly what their job requires (**least privilege**)
- Prevent incompatible duties in one role or user (**segregation of duties**)
- Produce evidence that satisfies SOX ITGC and internal audit
- Make authorization failures quick to diagnose

---

## 2. Authorization Layers in Fiori

| Layer | Controls | Objects / mechanism | Failure symptom |
|---|---|---|---|
| 1. Authentication | Who you are | SAML 2.0 SSO, `SU01` user | Logon error / redirect loop |
| 2. Launchpad content | Which apps you can see | Catalogs + spaces in PFCG role menu | Tile missing, app not in App Finder |
| 3. App start (front-end) | Which UI5 app/transaction may start | `S_TCODE` (GUI), UI5 app access | "Not authorized for transaction" |
| 4. Service access | Which OData services you may call | `S_SERVICE` (V2), `S_START` (V4 service groups) | HTTP 403 on OData request |
| 5. Business data | Which data/actions | `F_BKPF_BUK`, `F_BKPF_BLA`, `F_LFA1_BUK`, `F_REGU_BUK`, `K_CCA`, etc. | Empty list, error message, "no authorization" |

In the embedded deployment all five layers are covered by the same PFCG role.

---

## 3. Key Authorization Objects

| Object | Purpose | Typical values |
|---|---|---|
| `S_SERVICE` | Start OData V2 service (hash of service name) | Proposed via SU24 from catalog |
| `S_START` | Start OData V4 service group / UI5 app | Proposed via SU24 |
| `S_TCODE` | Start transaction (SAP GUI for HTML tiles) | e.g. `FBL1N`, `FB03` |
| `S_RFC` | RFC calls required by apps | Function groups per SU24 |
| `F_BKPF_BUK` | Accounting documents by company code | `BUKRS` = derived; `ACTVT` 01/02/03 |
| `F_BKPF_BLA` | Accounting documents by document type | `BRGRU` per document type groups |
| `F_BKPF_KOA` | By account type (D/K/S/A/M) | Clerk: K only for AP |
| `F_LFA1_BUK` / `F_LFA1_APP` | Supplier master | AP Clerk: `ACTVT 03` only |
| `F_KNA1_BUK` / `F_KNA1_APP` | Customer master | AR Clerk: `ACTVT 03` only |
| `F_REGU_BUK` | Payment program | AP Manager only |
| `F_BNKA_MAN` | Bank master | Cash Manager only |
| `K_CCA` | Cost center accounting | Controller |
| `S_TABU_DIS` / `S_TABU_NAM` | Table maintenance | Not granted to business roles |

---

## 4. Role Build Procedure

1. Add catalogs and space to the role menu.
2. Load SU24 proposals (Authorizations tab → expert mode → "Read old status and merge with new data").
3. Review every proposed object; remove `*` values except where documented.
4. Maintain org levels in derived roles.
5. Generate, then run SoD analysis against the rule set (see §6).
6. Document the role in the authorization matrix.

### Rules
- No `SAP_ALL`, `SAP_NEW`, or `S_DEVELOP` in any business role.
- No `*` for `ACTVT` on master data objects.
- Display-only roles contain only `ACTVT 03` (and `08` change documents where needed).
- Admin roles exist only in DEV with full access; QAS/PRD admin roles are display + cache invalidation.

---

## 5. Front-End / Back-End Consistency

A frequent source of incidents is a role that shows a tile (catalog assigned) but lacks the back-end authorization.

Controls:
- Roles are always built from catalog → SU24 proposals, so service authorizations come with the catalog.
- Authorization test for every role in QAS with `STAUTHTRACE` running, executing every tile (see [Testing Strategy](testing-strategy.md#4-authorization-testing)).
- `/UI2/FLC` content checks run per role before transport.

---

## 6. Segregation of Duties

### 6.1 Rule set (excerpt)

| Risk ID | Conflict | Function A | Function B | Severity |
|---|---|---|---|---|
| SOD-P2P-01 | Maintain vendor and pay vendor | Supplier master maintenance | Payment run / outgoing payments | Critical |
| SOD-P2P-02 | Enter invoice and approve invoice | Supplier invoice entry | Invoice approval / release | High |
| SOD-P2P-03 | Maintain vendor bank data and process payments | Vendor bank details | Payment execution | Critical |
| SOD-O2C-01 | Maintain customer and post incoming payments / credits | Customer master | Incoming payment / credit memo | High |
| SOD-R2R-01 | Post journal entries and open/close periods | Journal posting | Posting period maintenance | High |
| SOD-TRE-01 | Maintain bank master and approve payments | Bank accounts | Bank payment approval | Critical |
| SOD-SEC-01 | Maintain roles and assign roles | `PFCG` | `SU01` | Critical |

### 6.2 Results
| Role | Conflicts found | Resolution |
|---|---|---|
| `Z_FIN_AP_CLERK_BR` | SOD-P2P-02 (release blocked invoices) | Release app moved to `Z_FIN_AP_APPROVAL_BC` (Manager) |
| `Z_FIN_AP_MANAGER_BR` | None after redesign | – |
| `Z_FIN_GL_ACCOUNTANT_BR` | SOD-R2R-01 | Open/Close Posting Periods set to display only; change held by Close Lead role |
| `Z_FIN_CASH_MANAGER_BR` | SOD-TRE-01 | Bank payment approval assigned to a separate treasury approver; mitigating dual-control documented |
| Others | None | – |

### 6.3 User-level conflicts
Users who must hold two roles (e.g. small subsidiary teams) require a **mitigating control** (e.g. monthly review of payments by finance director) documented and approved by the risk owner.

---

## 7. Emergency Access (Firefighter)

- Emergency access in PRD uses time-bound firefighter IDs with logging.
- Every session requires a ticket, reason and post-review by the controller within 2 business days.

---

## 8. Access Reviews

| Review | Frequency | Evidence |
|---|---|---|
| User-to-role recertification | Quarterly | Signed review from role owner |
| Role content review | Semi-annual | Role change log, SoD report |
| Privileged access review | Monthly | Firefighter logs |
| Terminated users | Daily (automated) | HR feed → user lock |

Useful reports: `SUIM` (users by role, roles by authorization value, change documents), `RSUSR002`, `RSUSR100N` (user change docs), `RSUSR200` (user last logon).

---

## 9. Technical Security Settings

| Area | Setting |
|---|---|
| Transport | HTTPS only; HTTP ports closed externally |
| Clickjacking | Framing protection on; whitelist only trusted origins |
| Session | 30-minute timeout; security session management enabled in `SICF_SESSIONS` |
| ICF | Unused services deactivated; quarterly review |
| OData | Unused services deactivated in `/IWFND/MAINT_SERVICE` |
| CSRF | X-CSRF-Token protection active (default) on modifying OData requests |
| Logging | Security Audit Log (`SM19`/`RSAU_CONFIG`) captures failed logons and critical transactions |

---

## 10. Audit Evidence Pack

For each release, the following is retained:
- Transport list with approvals
- Role change documents
- SoD analysis report
- Authorization test results (STAUTHTRACE extracts)
- UAT sign-off

---

## Related Documents

- [Catalog & Role Design](catalog-and-role-design.md)
- [Sample Authorization Matrix](../samples/sample-authorization-matrix.xlsx)
- [Troubleshooting Guide](troubleshooting-guide.md#4-authorization-errors)
