# Project Overview & Scope – BINARY

## 1. Purpose

This document defines the purpose, scope, stakeholders, timeline and success criteria for the SAP Fiori Launchpad (FLP) Configuration & Optimization project at BINARY. It is the reference point for every other document in this repository.

---

## 2. Background

BINARY runs SAP S/4HANA for its core finance processes. Before this project:

- The launchpad used **classic groups** with ~400 tiles assigned through overlapping catalogs.
- Users saw tiles they could not open (front-end authorization without matching back-end authorization), producing frequent 403 errors.
- Catalogs had been created ad hoc by project teams and no longer matched business processes.
- Some content changes had been made directly in production, which internal audit flagged as an ITGC finding.
- Launchpad load times exceeded 9 seconds at P90 during month-end close.

---

## 3. Objectives

| # | Objective | Measure of success |
|---|---|---|
| O1 | Role-based, persona-driven launchpad | Every finance user has exactly one primary business role and space |
| O2 | Migrate groups → spaces and pages | 100% of finance roles on spaces/pages; groups deactivated |
| O3 | Least privilege and SoD compliance | 0 unmitigated SoD conflicts in the role design |
| O4 | Stable Gateway/OData integration | No recurring OData incident categories after hypercare |
| O5 | Audit-ready change control | 0 direct changes in PRD; all content transported |
| O6 | Performance | Launchpad initial load P90 ≤ 4 s |
| O7 | Supportability | Fiori ticket volume reduced by ≥ 50% |

---

## 4. Scope

### In scope

- SAP Fiori Launchpad configuration on the S/4HANA embedded front-end
- Business catalogs, technical catalogs, target mappings and tiles for finance
- PFCG business roles for 8 personas (see [Catalog & Role Design](catalog-and-role-design.md))
- Spaces and pages for every in-scope role
- Activation and transport of required OData V2 services and V4 service groups
- Front-end and back-end authorization design, SU24 maintenance, SoD analysis
- Launchpad settings, theming (corporate theme), and SSO configuration
- Testing: unit, integration, authorization, UAT, regression
- Documentation, training materials and hypercare

### Out of scope

- Custom SAPUI5 application development (only SAP-standard apps plus existing custom apps)
- SAP BTP / cloud launchpad (SAP Build Work Zone) – noted as a future option
- Non-finance modules (MM, SD, HR) except display apps needed by finance
- GRC Access Control implementation (the existing rule set is consumed, not rebuilt)
- Mobile device management

### Assumptions

- S/4HANA and front-end components are on a supported release with current UI add-on/SAP_UI levels.
- Basis provides system access, ICF activation and SSO certificates on schedule.
- Process owners are available for design workshops and UAT sign-off.

### Constraints

- No changes in PRD outside the standard transport process.
- Month-end and quarter-end close freezes (last 3 and first 5 working days of each month).

---

## 5. Stakeholders & RACI

| Activity | Fiori Lead | Security Analyst | Basis | Finance Process Owners | Internal Audit | IT Change Board |
|---|---|---|---|---|---|---|
| Launchpad architecture | **A/R** | C | C | I | I | I |
| Catalog & tile design | **A/R** | C | I | C | I | I |
| Role design (PFCG) | R | **A** | I | C | C | I |
| SoD analysis | C | **A/R** | I | C | C | I |
| Gateway/OData activation | R | I | **A** | I | I | I |
| Spaces & pages layout | **A/R** | I | I | C | I | I |
| UAT | C | C | I | **A/R** | I | I |
| Transport to PRD | R | R | R | I | I | **A** |
| Access review sign-off | I | R | I | **A** | C | I |

R = Responsible, A = Accountable, C = Consulted, I = Informed

---

## 6. Approach & Timeline

| Phase | Duration | Key deliverables |
|---|---|---|
| 1. Discover | 2 weeks | Current-state inventory (tiles, catalogs, roles, usage), persona interviews, pain-point log |
| 2. Design | 3 weeks | Role/catalog model, space/page wireframes, SoD analysis, naming standards |
| 3. Build | 4 weeks | Catalogs, target mappings, roles, spaces/pages, service activation (DEV) |
| 4. Test | 3 weeks | Integration, authorization and performance tests (QAS); UAT with finance |
| 5. Deploy | 1 week | Cutover outside close freeze, user mapping, groups deactivation |
| 6. Hypercare | 4 weeks | Daily triage, fine-tuning, knowledge transfer to support |

### Milestones

1. Design sign-off by finance process owners and security
2. DEV build complete and unit tested
3. QAS integration and authorization tests passed
4. UAT sign-off
5. Change board approval and PRD go-live
6. Hypercare exit and handover to support

---

## 7. Current-State Analysis (Discover Phase)

Activities performed:

- Extracted all catalogs, tiles and target mappings using the launchpad content manager (`/UI2/FLPCM_CONF`) and catalog reports.
- Pulled role-to-catalog assignments with `SUIM` and role menus.
- Captured tile usage where available and interviewed users per persona.
- Logged recurring incidents from the ticketing system by category.

Findings summary:

| Finding | Impact |
|---|---|
| 47 custom catalogs with heavy overlap | Duplicate tiles, unclear ownership |
| Tiles assigned without back-end authorizations | 403 errors, user frustration |
| Groups personalized heavily by users | Hard to roll out changes consistently |
| Unused OData services active | Increased attack surface |
| PRD content changes outside transports | Audit finding |
| Dynamic tiles calling slow services | Slow launchpad at month-end |

---

## 8. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Users lose personalized groups at migration | High | Medium | Communicate early; offer "My Home" and personalization of pages where enabled |
| Missing back-end authorizations after role redesign | Medium | High | Authorization testing with STAUTHTRACE per role in QAS |
| Go-live collides with close | Low | High | Schedule outside freeze windows |
| SoD conflicts discovered late | Medium | High | Run SoD analysis during design, not at build end |
| Gateway service gaps between systems | Medium | Medium | Service activation transported, verified in each system |

---

## 9. Success Metrics (Post Go-Live)

See the outcomes table in the [README](../README.md#key-outcomes). Metrics are reviewed at hypercare exit and then quarterly by the Fiori lead and finance process owners.

---

## Related Documents

- [Fiori Launchpad Architecture](fiori-launchpad-architecture.md)
- [Catalog & Role Design](catalog-and-role-design.md)
- [Transport & Change Management](transport-and-change-management.md)
