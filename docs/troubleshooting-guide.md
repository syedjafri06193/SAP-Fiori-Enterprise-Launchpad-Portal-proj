# SAP Fiori Troubleshooting Guide – BINARY

## Common Issues and Resolutions

---

## Application Does Not Launch

Steps:
1. Verify PFCG role assignment
2. Execute SU53 immediately after error
3. Check target mapping and intent configuration

---

## Missing Tiles

- Confirm business catalog assignment
- Validate space and page configuration
- Clear user and FLP cache

---

## OData Service Errors

- Check /IWFND/ERROR_LOG
- Verify service activation in /IWFND/MAINT_SERVICE
- Validate system alias and RFC destination

---

## Authorization Errors

- Review ST22 dumps
- Check SM21 system logs
- Use authorization trace (STAUTHTRACE)

---

## Performance Issues

- Review browser and Gateway logs
- Validate service response times
- Coordinate with Basis team if needed
