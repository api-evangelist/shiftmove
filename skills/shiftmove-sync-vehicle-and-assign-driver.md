---
name: Sync a vehicle and assign a driver
description: Create or update a vehicle in the Avrios Fleet-API (Shiftmove), then assign a driver to it over a duration.
api: openapi/shiftmove-fleet-openapi.json
operations: [createVehicle, findVehicleByUuid, updateVehicleByUuid, getAllDrivers, createVehicleAssignment, findVehicleAssignments]
---

# Sync a vehicle and assign a driver

Use the Avrios Fleet-API (base URL `https://api.avrios.com`, paths under `/v1`) to add a vehicle and put a driver in it.

## Authentication
Send `Authorization: Basic {base64(username:password)}` on every request. See `authentication/shiftmove-authentication.yml`.

## Steps
1. **Create the vehicle** — `POST /v1/vehicles` (`createVehicle`). Send the vehicle body (VIN, license plate, model, `organizationUuid`). The response returns the vehicle `uuid`.
2. **(Optional) Verify** — `GET /v1/vehicles/{uuid}` (`findVehicleByUuid`) to confirm the persisted record.
3. **(Optional) Update** — `POST /v1/vehicles/{uuid}` (`updateVehicleByUuid`). Only properties present in the body are changed; send `null` to clear a property. Note `internalId` cannot be changed for import-managed vehicles.
4. **Find the driver** — `GET /v1/drivers` (`getAllDrivers`) paginated (`pageNumber`, `limit`), or `POST /v1/drivers/query` to filter. Capture the driver `uuid`.
5. **Assign the driver** — `POST /v1/vehicles/{uuid}/assignments` (`createVehicleAssignment`) with the `driverUuid` and duration. A **409** is returned if the assignment window conflicts with an existing one; the conflicting assignments come back in the body — adjust the timing and retry.
6. **Confirm** — `GET /v1/vehicles/{uuid}/assignments` (`findVehicleAssignments`).

## Rules
- Rate limit: 300 requests/minute (`rate-limits/shiftmove-rate-limits.yml`).
- No idempotency-key support — do not blind-retry a POST; on network failure re-query before recreating.
- Dates may be `yyyy-MM-dd`, interpreted as the first second of the day in the company timezone.
- Errors: 400 = ValidationFailed (invalid fields in body), 404 = not found, 409 = conflict. See `errors/shiftmove-problem-types.yml`.
