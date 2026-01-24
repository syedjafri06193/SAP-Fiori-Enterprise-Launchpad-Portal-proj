# SAP Fiori Launchpad Architecture – BINARY

## Overview

At BINARY, SAP Fiori Launchpad (FLP) serves as the centralized access point for SAP S/4HANA business applications. The architecture is designed to support secure, role-based access while ensuring high availability, performance, and compliance with financial industry standards.

---

## Architecture Components

### Fiori Front-End Server (FES)
- Hosts SAP Fiori Launchpad
- Runs SAPUI5 applications
- Manages UI services and themes

### SAP Gateway
- Handles OData communication
- Routes requests between front end and backend systems
- Manages system aliases and service activation

### SAP S/4HANA Backend
- Core business logic
- CDS views and transactional processing
- Financial master and transactional data

---

## Launchpad Content Model

- **Business Catalogs:** Contain role-specific Fiori applications
- **Technical Catalogs:** Support administrative and technical apps
- **Spaces & Pages:** Define the visual layout and navigation
- **PFCG Roles:** Control authorization and access

---

## System Landscape

- Development (DEV)
- Quality Assurance (QAS)
- Production (PRD)

All Fiori content and Gateway services are transported through the DEV → QAS → PRD landscape to meet audit and compliance requirements.

---

## Security Considerations

- Role-based access control
- No direct database access from UI layer
- Secure RFC and HTTPS communication enforced
