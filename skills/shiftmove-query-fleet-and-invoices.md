---
name: Query fleet vehicles, drivers and invoices
description: Page and filter vehicles, drivers and invoices from the Avrios Fleet-API (Shiftmove) for reporting and reconciliation.
api: openapi/shiftmove-fleet-openapi.json
operations: [getAllVehicles, queryVehicles, getAllDrivers, queryDrivers, getAllInvoices, queryInvoices, getAllInvoicesItems, getVehicleCurrentFinancing]
---

# Query fleet vehicles, drivers and invoices

Read-side flow over the Avrios Fleet-API (base URL `https://api.avrios.com`, paths under `/v1`) for pulling fleet state and cost data.

## Authentication
Send `Authorization: Basic {base64(username:password)}` on every request. See `authentication/shiftmove-authentication.yml`.

## Steps
1. **List vehicles** — `GET /v1/vehicles` (`getAllVehicles`) with `pageNumber`, `limit`, `sortBy`, `reverse`. The `ApiPage` envelope returns `items`, `page`, `totalItems`. For filtered pulls use `POST /v1/vehicles/query` (`queryVehicles`).
2. **List drivers** — `GET /v1/drivers` (`getAllDrivers`) or `POST /v1/drivers/query` (`queryDrivers`), same paging shape.
3. **Pull invoices** — `GET /v1/invoices` (`getAllInvoices`) or `POST /v1/invoices/query` (`queryInvoices`). For line items, `GET /v1/invoices/{invoiceUuid}/items` (`getAllInvoicesItems`).
4. **Financing** — `GET /v1/vehicles/{uuid}/financing/current` (`getVehicleCurrentFinancing`) for the current financing contract per vehicle.

## Rules
- Only populated fields are returned; `null` fields are stripped from responses.
- Page through until `page * limit >= totalItems`; keep total request volume under 300/minute.
- Some filter/date parameters accept `yyyy-MM-dd`, interpreted as the first second of the day in the company timezone.
- See `conventions/shiftmove-conventions.yml` for the full pagination and null-handling contract.
