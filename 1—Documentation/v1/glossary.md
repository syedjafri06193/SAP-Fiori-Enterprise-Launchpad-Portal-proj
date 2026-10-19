# Glossary – BINARY Fiori Launchpad

## Terms

| Term | Definition |
|---|---|
| **App Finder** | Launchpad feature listing all apps a user is authorized for via their catalogs |
| **App index** | Server-side index of SAPUI5 apps and libraries used to resolve app resources |
| **Business catalog** | Set of apps (tiles + target mappings) for a business process; assigned to PFCG roles |
| **Business role** | PFCG role combining catalogs, spaces and authorizations for a persona |
| **CDS view** | Core Data Services view – semantic data model on SAP HANA, often exposed via OData |
| **DCL** | Data Control Language – CDS access controls that enforce authorizations on CDS views |
| **Derived role** | PFCG role inheriting from a master role, differing only in org levels |
| **Dynamic tile** | Tile showing a number retrieved periodically from a service |
| **Embedded deployment** | Fiori front-end server running inside the S/4HANA system |
| **FES** | Front-End Server – hosts the launchpad and UI5 apps |
| **FLP** | SAP Fiori Launchpad |
| **Intent** | Semantic object + action (+ parameters), e.g. `#SupplierInvoice-manage` |
| **ITGC** | IT General Controls (SOX) |
| **KPI tile** | Tile displaying a key performance indicator with thresholds |
| **My Home** | Personal page in spaces mode for favorite apps |
| **OData** | Open Data Protocol – REST-based protocol used by Fiori apps |
| **Page** | Layout of sections and visualizations within a space |
| **PFCG** | Profile Generator – role maintenance transaction |
| **RAP** | ABAP RESTful Application Programming model |
| **SAML 2.0** | Standard for SSO assertions between IdP and service provider |
| **Section** | Titled group of visualizations on a page |
| **Semantic object** | Business entity used in intents (e.g. `SupplierInvoice`) |
| **SoD** | Segregation of duties |
| **Space** | Container of pages, assigned to users via roles; appears as a tab |
| **SU24** | Maintenance of authorization default values proposed to PFCG |
| **System alias** | Gateway mapping to the backend system processing a service |
| **Target mapping** | Configuration resolving an intent to an app |
| **Technical catalog** | Repository catalog of tiles/target mappings, referenced by business catalogs |
| **Tile** | Visual launch element for an app |
| **Visualization** | A tile or link representing an app on a page |

## Transactions & Reports

| Code | Purpose |
|---|---|
| `/UI2/FLP` | Launch the launchpad from SAP GUI |
| `/UI2/FLPD_CUST` | Launchpad Designer – client-specific (customizing) |
| `/UI2/FLPD_CONF` | Launchpad Designer – cross-client (configuration) |
| `/UI2/FLPCM_CUST`, `/UI2/FLPCM_CONF` | Launchpad Content Manager |
| `/UI2/FLPAM` | Launchpad App Manager |
| `/UI2/FLIA` | Launchpad intent analysis |
| `/UI2/FLC` | Launchpad content checks |
| `/UI2/FLP_CUS_CONF`, `/UI2/FLP_SYS_CONF` | Launchpad settings (client / system) |
| `/UI2/INVALIDATE_GLOBAL_CACHES` | Invalidate global launchpad caches |
| `/UI2/INVALIDATE_CLIENT_CACHES` | Invalidate client caches (all or per user) |
| `/UI5/APP_INDEX_CALCULATE` | Recalculate UI5 app index |
| `/UI5/THEME_DESIGNER` | UI Theme Designer |
| `/IWFND/MAINT_SERVICE` | Activate and maintain OData V2 services |
| `/IWFND/V4_ADMIN` | Publish/maintain OData V4 service groups |
| `/IWFND/ERROR_LOG` | Gateway hub error log |
| `/IWBEP/ERROR_LOG` | Gateway backend error log |
| `/IWFND/GW_CLIENT` | Gateway client (test OData calls) |
| `/IWFND/TRACES` | Gateway traces (payload/performance) |
| `/IWFND/STATS` | Gateway statistics |
| `/IWFND/CACHE_CLEANUP`, `/IWBEP/CACHE_CLEANUP` | Clear Gateway metadata caches |
| `PFCG` | Role maintenance |
| `PFUD` | User master comparison |
| `SU01` | User maintenance |
| `SU24` | Authorization defaults |
| `SU25` | Upgrade tool for authorization defaults |
| `SU53` | Last failed authorization check |
| `STAUTHTRACE` | System-wide authorization trace |
| `SUIM` | User information system |
| `SICF` | ICF service maintenance |
| `SAML2` | SAML 2.0 configuration |
| `STRUST` | Trust manager (certificates) |
| `SM59` | RFC destinations |
| `SMT1` | Trusted–trusting relationships |
| `STMS` | Transport management |
| `STC01` | Task manager (task lists) |
| `ST22` | ABAP dumps |
| `SM21` | System log |
| `ST05` | SQL / performance trace |
| `SAT` | ABAP runtime analysis |
| `SLG1` | Application log |
| `SMICM` | ICM monitor |

> Transaction availability and names depend on the `SAP_UI` / S/4HANA release.
