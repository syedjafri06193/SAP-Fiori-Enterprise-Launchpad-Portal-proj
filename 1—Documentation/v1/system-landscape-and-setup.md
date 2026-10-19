# System Landscape & Setup – BINARY

## 1. Landscape

```
 ┌──────────┐  transport  ┌──────────┐  transport  ┌──────────┐
 │  S4D/100 │ ──────────▶ │  S4Q/100 │ ──────────▶ │  S4P/100 │
 │   DEV    │             │   QAS    │             │   PRD    │
 └──────────┘             └──────────┘             └──────────┘
   build &                  integration,             live,
   unit test                auth, UAT                locked
```

| Property | DEV | QAS | PRD |
|---|---|---|---|
| Client settings (`SCC4`) | Changes with automatic recording | No changes allowed | No changes allowed |
| Customizing / cross-client | Allowed | Locked | Locked |
| Launchpad content changes | Allowed (transported) | Not allowed | Not allowed |
| Who has admin roles | Fiori team | Fiori team (display + cache) | Support (display + cache, firefighter for emergencies) |
| Test data | Minimal | Copy of anonymized PRD | Live |

---

## 2. Prerequisites

### 2.1 Software components
Confirm in `System → Status → Component information`:

- `SAP_UI` at the level delivered with the S/4HANA release (supports spaces and pages)
- `SAP_GWFND` (Gateway foundation)
- `S4CORE` / `S4FND`
- Latest SAP Notes for launchpad and Gateway applied per SAP's recommended note lists

### 2.2 Technical users and RFC
| Item | Purpose |
|---|---|
| `SM59` destination `<SID>CLNT100` (if any remote alias) | Trusted RFC for Gateway routing |
| Technical user for background jobs (`SAP_WORKFLOW` style service user) | Cache cleanups, app index calculation |

### 2.3 Browser support
Chrome and Microsoft Edge (current versions) per SAPUI5 browser support matrix. Internet Explorer is not supported.

---

## 3. Initial Setup Checklist

Performed once per system (DEV first, then reproduced through transports or per-system activities in QAS/PRD).

### 3.1 Activate ICF services (`SICF`)
Activate only what is needed:

| Path | Purpose |
|---|---|
| `/sap/bc/ui2/flp` | Launchpad entry point |
| `/sap/bc/ui2/start_up` | Launchpad start-up service |
| `/sap/bc/ui5_ui5` | UI5 application repository |
| `/sap/public/bc/ui5_ui5` | SAPUI5 libraries |
| `/sap/public/bc/ui2` | Public UI2 resources |
| `/sap/opu/odata` | OData V2 entry |
| `/sap/opu/odata4` | OData V4 entry |
| `/sap/bc/gui/sap/its/webgui` | SAP GUI for HTML (classic transactions) |
| `/sap/bc/ui2/nwbc` | Required for some launchpad features |

> ICF activation is client-independent but **not transportable** by default; record it in the per-system activation checklist.

### 3.2 Activate launchpad OData services
Activate the launchpad infrastructure services in `/IWFND/MAINT_SERVICE` (e.g. `/UI2/PAGE_BUILDER_PERS`, `/UI2/PAGE_BUILDER_CONF`, `/UI2/PAGE_BUILDER_CUST`, `/UI2/INTEROP`, `/UI2/LAUNCHPAD`, `/UI2/TRANSPORT`) and the V4 service groups used by Manage Launchpad Spaces/Pages in `/IWFND/V4_ADMIN`. SAP's task list `SAP_GATEWAY_ACTIVATE_ODATA_SERV` in `STC01` can activate these in bulk.

### 3.3 Run SAP task lists (`STC01`)
| Task list | Purpose |
|---|---|
| `SAP_GATEWAY_BASIC_CONFIG` | Basic Gateway configuration |
| `SAP_FIORI_FOUNDATION_S4` | Fiori foundation for S/4HANA (FLP + services) |
| `SAP_FIORI_CONTENT_ACTIVATION` | Activate SAP Fiori content for selected business roles (used for reference roles only) |

### 3.4 Configure SSO
1. `STRUST`: import the IdP signing certificate and ensure SSL server PSE is valid.
2. `SAML2`: enable SAML 2.0, create the local provider, add the corporate IdP as trusted provider, set NameID mapping (e.g. e-mail → `SU01` e-mail or user alias).
3. `SICF`: set logon procedure on `/sap/bc/ui2/flp` and `/sap/opu` to use SAML.
4. Test with a pilot user in each system.

### 3.5 Launchpad settings
Configure via `/UI2/FLP_CUS_CONF` (client-specific) and `/UI2/FLP_SYS_CONF` (system-wide). Key settings are listed in the [Configuration Guide](launchpad-configuration-guide.md#2-global-launchpad-settings).

### 3.6 Theme
Use the corporate theme built with UI Theme Designer (`/UI5/THEME_DESIGNER`) on top of the SAP Horizon theme and set it as default in the launchpad settings.

### 3.7 Background jobs
| Job | Program/Report | Frequency |
|---|---|---|
| App index calculation | `/UI5/APP_INDEX_CALCULATE` | Daily, and after UI5 app imports |
| Gateway cache cleanup | `/IWFND/R_SM_CLEANUP` | Daily |
| Gateway log cleanup | Cleanup of `/IWFND/ERROR_LOG` and `/IWBEP/ERROR_LOG` | Weekly |

### 3.8 Verify
- Open `https://<host>/sap/bc/ui2/flp` with a test user → launchpad loads with expected space.
- Run `/UI2/FLC` (launchpad content checks) for the test role.
- Check `/IWFND/ERROR_LOG` is empty after smoke tests.

---

## 4. Per-System Activities (Not Transportable)

Maintain this list in the change ticket for each system build:

| Activity | DEV | QAS | PRD |
|---|---|---|---|
| ICF node activation | ✅ | ✅ | ✅ |
| SAML2 provider configuration | ✅ | ✅ | ✅ |
| STRUST certificates | ✅ | ✅ | ✅ |
| System alias for remote backends (if any) | ✅ | ✅ | ✅ |
| User-to-role assignment | Test users | Test users | Via access request process |
| Background job scheduling | ✅ | ✅ | ✅ |

---

## 5. URLs

| System | Launchpad URL |
|---|---|
| DEV | `https://s4d.binary.example/sap/bc/ui2/flp?sap-client=100` |
| QAS | `https://s4q.binary.example/sap/bc/ui2/flp?sap-client=100` |
| PRD | `https://fiori.binary.example/sap/bc/ui2/flp` |

Useful URL parameters for diagnostics:

| Parameter | Effect |
|---|---|
| `sap-ui-debug=true` | Loads non-minified UI5 sources |
| `sap-language=EN` | Forces logon language |
| `sap-theme=sap_horizon` | Forces a theme (isolate theme issues) |
| `sap-ushell-cdm-cache=false` | Bypasses client-side content cache (testing only) |

---

## Related Documents

- [Fiori Launchpad Architecture](fiori-launchpad-architecture.md)
- [Launchpad Configuration Guide](launchpad-configuration-guide.md)
- [Operations Runbook](operations-runbook.md)
