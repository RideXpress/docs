# Request Ride – Edge Conditions & Alternative Flows

This document supplements the [Request Ride use case diagram](../diagrams/use-cases/request-ride-use-case.drawio.svg) with edge conditions and alternative flows that are not covered in the diagram.

## Assumptions (from diagram)

1. Drivers will inform their location and availability periodically with the mobile app to the server.
2. Users will have to be logged in to create a ride.
3. If a driver is active and with no current ride, then they can accept a new ride.

## Edge Conditions

### EC-1: No Drivers Available

**Trigger:** No drivers accept the ride request within the timeout period.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | Ride is created with status "New" and broadcast to nearby drivers                        |
| 2    | System waits for a driver to accept (timeout: **3 minutes**)                             |
| 3    | If no driver accepts within 3 minutes, ride status is updated to "Canceled"              |
| 4    | Payment hold is released via `square-system-api` (`PUT /orders/{id}` with state = CANCELED) |
| 5    | Client is notified: "No drivers are available at this time. Please try again later."     |
| 6    | Client can retry immediately; the ride request is treated as a new request               |

### EC-2: Concurrent Ride Requests (Client)

**Trigger:** Client attempts to request a new ride while they already have an active ride.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | Client sends a ride request via `mobile-experience-api`                                  |
| 2    | `request-ride-process-api` checks `db-system-api` for any active rides for this client   |
| 3    | If an active ride exists (status: New, Waiting for driver, In route, etc.), the request is rejected |
| 4    | Client receives: "You already have an active ride. Please complete or cancel your current ride before requesting a new one." |

### EC-3: Concurrent Ride Acceptance (Driver)

**Trigger:** A driver attempts to accept a ride while they already have an assigned ride.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | The driver broadcast filter already excludes drivers with an active ride (`GET /drivers` filters by `status = available`) |
| 2    | In the rare case of a race condition (two rides accepted simultaneously), the second acceptance returns a conflict error |
| 3    | The ride is re-broadcast to other available drivers                                      |

### EC-4: GPS / Location Service Failure

**Trigger:** The client's GPS is unavailable or inaccurate when requesting a ride.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | Client's app cannot determine the current location                                       |
| 2    | App displays: "Unable to determine your location. Please enable GPS or enter your pickup address manually." |
| 3    | Client can manually enter a pickup address (geocoded via `google-maps-system-api`)       |
| 4    | If Google Maps geocoding also fails, the ride request cannot proceed                     |

### EC-5: Google Maps API Failure

**Trigger:** Google Maps is unavailable when calculating the route.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | `request-ride-process-api` calls `google-maps-system-api` but receives an error          |
| 2    | System retries up to 3 times with exponential backoff (1s, 2s, 4s)                       |
| 3    | If all retries fail, the ride request fails with: "Unable to calculate your route. Please try again shortly." |
| 4    | No ride is created; no payment hold is placed                                            |

### EC-6: Payment Hold Failure at Ride Request

**Trigger:** Square cannot place a hold on the client's payment method.

See the [Payment Failure Handling](payment-failure-handling.md) use case for detailed scenarios including card declined, Square API errors, and payment link expiration.

### EC-7: Network Connectivity Loss (Client)

**Trigger:** Client loses network connectivity during ride request.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | If connectivity is lost before the request is sent, the app queues the request locally   |
| 2    | When connectivity is restored, the app resends the request                               |
| 3    | If connectivity is lost after the request is sent but before the response, the app displays: "Checking your ride status..." |
| 4    | On reconnection, the app polls `GET /rides/{id}` to determine the current ride state     |
| 5    | If the ride was created successfully, the app displays the ride details; otherwise, the client can retry |

### EC-8: Driver Broadcast Failure

**Trigger:** Push notification to drivers fails.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | `push-notifications-system-api` reports delivery failure for one or more drivers         |
| 2    | Failed notifications are retried once                                                    |
| 3    | If a driver's notification repeatedly fails, they are excluded from the broadcast pool   |
| 4    | The ride assignment proceeds with the remaining drivers                                  |
