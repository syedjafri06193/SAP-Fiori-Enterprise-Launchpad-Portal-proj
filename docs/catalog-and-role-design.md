# Catalog and Role Design – BINARY

## Design Principles

- Least privilege access
- Clear separation of duties
- Scalable and reusable role structure
- Alignment with financial compliance standards

---

## Role Structure

Each business role at BINARY consists of:
- One PFCG business role
- One or more business catalogs
- Assigned space and page
- Backend authorization objects

---

## Naming Conventions

- Roles: Z_FIN_<FUNCTION>_BR
- Business Catalogs: Z_FIN_<PROCESS>_BC
- Spaces: Z_FIN_<ROLE>_SP
- Pages: Z_FIN_<ROLE>_PG

---

## Example Role

**Role:** Z_FIN_AP_CLERK_BR

Includes:
- Invoice processing applications
- Vendor management apps
- Reporting tiles
- Assigned space and page optimized for AP workflow

---

## Transport Strategy

- All catalog and role changes are transport-controlled
- Emergency changes follow documented approval process
