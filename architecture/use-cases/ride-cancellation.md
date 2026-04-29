# Ride Cancellation (Client & Driver)

> **Diagram:** [Ride Cancellation Flow](../diagrams/use-cases/ride-cancellation-use-case.drawio.svg)

## Overview

| Field              | Details                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Use Case ID**    | UC-RIDE-CANCEL-01                                                       |
| **Priority**       | Critical                                                                |
| **Actors**         | Client (Rider), Driver                                                  |
| **Trigger**        | A client or driver initiates a cancellation of an active ride           |
| **Preconditions**  | A ride exists in a cancellable state (New, Waiting for driver, Driving to passenger location, or Waiting for passenger) |
| **Postconditions** | Ride status is updated to "Canceled"; payment is refunded or charged based on cancellation rules; both parties are notified |

## Description

As a client, I want the ability to cancel a requested ride before or during the pickup phase. As a driver, I want the ability to cancel an accepted ride if I am unable to complete the pickup. The system must enforce cancellation policies that determine whether the client is charged or refunded, and must handle ride reassignment when applicable.

## Cancellation Rules (from Ride States)

The ride-states diagram defines two types of cancellations:

### Canceled Without Charge (Refund)

The payment hold is released and no charge is applied when:

| # | Condition                                                                                      |
|---|-----------------------------------------------------------------------------------------------|
| 1 | The driver has not been confirmed after **2 minutes** and the passenger does not want to wait longer |
| 2 | There are no drivers available and no one confirms after **3 minutes**                         |
| 3 | The driver has confirmed but will take longer to get to the passenger's location than expected  |

### Canceled With Charge

The payment hold is committed (ride is charged) when:

| # | Condition                                                                                      |
|---|-----------------------------------------------------------------------------------------------|
| 1 | The driver arrives at the passenger's location but the passenger does not get into the car within a **5-minute** period |
| 2 | The passenger cancels the ride and the driver is on their way and will arrive at the passenger's location in under **3 minutes** |
| 3 | The passenger does not provide the correct one-time token when getting into the car (driver never picked up the correct passenger) |

## Main Flow: Client Cancellation

1. Client selects "Cancel Ride" in the mobile app.
2. App sends cancellation request to `mobile-experience-api` → `cancel-ride-process-api` (`PATCH /rides/{id}/status`).
3. Process API retrieves the current ride state from `db-system-api` (`GET /rides/{id}`).
4. Process API evaluates the cancellation rules based on ride state and timestamps:
   - **Ride is in "New" or "Waiting for driver" state (no driver confirmed yet):**
     - Cancel without charge → release payment hold via `square-system-api` (`PUT /orders/{id}` with state = CANCELED).
   - **Ride is in "Driving to passenger location" state:**
     - Calculate driver ETA. If driver arrives in under 3 minutes → cancel with charge (commit payment via `square-system-api` `POST /payments/{id}/commit`).
     - If driver ETA is over 3 minutes → cancel without charge (release hold).
   - **Ride is in "Waiting for passenger" state:**
     - Cancel with charge (passenger failed to board).
5. Process API updates ride status to "Canceled" in `db-system-api` (`PATCH /rides/{id}`).
6. Push notification sent to the driver via `push-notifications-system-api` (`POST /notification`) informing them of the cancellation.
7. Client app displays cancellation confirmation with charge/refund details.

## Main Flow: Driver Cancellation

1. Driver selects "Cancel Ride" in the mobile app.
2. App sends cancellation request to `mobile-experience-api` → `cancel-ride-process-api` (`PATCH /rides/{id}/status`).
3. Process API retrieves the current ride state and evaluates:
   - **Regardless of state:** Client is never charged when the driver cancels.
   - Payment hold is released via `square-system-api`.
4. Process API updates ride status to "Canceled (by driver)" in `db-system-api`.
5. **Ride reassignment flow:**
   - Process API triggers a new driver search by calling `request-ride-process-api` with the existing ride details.
   - Available drivers near the client's pickup location are queried from `db-system-api` (`GET /drivers`).
   - A new driver is selected and notified via `push-notifications-system-api`.
   - The original driver is excluded from the candidate pool.
6. Client is notified: "Your driver canceled. We are finding you a new driver."
7. If no new driver is found within 3 minutes, the ride is fully canceled and the client is notified.

## Alternative Flows

### AF-1: System-Initiated Cancellation (No Drivers Available)

1. After a ride is requested, the system waits for a driver to accept.
2. If no driver accepts within **3 minutes**, the system auto-cancels the ride.
3. Payment hold is released (cancel without charge).
4. Client is notified: "No drivers are available at this time. Please try again later."

### AF-2: System-Initiated Cancellation (Driver No-Show)

1. A driver has been confirmed and is en route to the pickup location.
2. If the driver has not arrived within the expected ETA + **5-minute grace period**, the system flags the ride.
3. Client is given the option to continue waiting or cancel without charge.
4. If the client cancels, the driver may receive a penalty flag on their account.

### AF-3: Client Cancellation During "Waiting for Passenger"

1. Driver arrives at the pickup location and waits.
2. After **5 minutes** of waiting, the driver can mark the client as a no-show.
3. Ride is canceled with charge; the cancellation fee is applied to the client.
4. Driver is freed to accept new rides.

## Edge Conditions

| # | Condition                              | Behavior                                                              |
|---|----------------------------------------|-----------------------------------------------------------------------|
| 1 | Client cancels while ride is "In route" | Cancellation is not allowed once the trip has started; client must wait for ride to finish |
| 2 | Both client and driver cancel simultaneously | First cancellation request processed wins; second request returns conflict error |
| 3 | Square API unavailable during refund   | Refund is queued for retry; client is notified that refund is pending; system retries with exponential backoff |
| 4 | Network loss during cancellation       | App retries cancellation request when connectivity is restored; ride remains in current state until confirmed |
| 5 | Driver cancels multiple rides in a row | After 3 cancellations within 1 hour, the driver's status is automatically set to offline; a warning is issued |
| 6 | Client repeatedly cancels rides        | After 3 cancellations in 24 hours, a cancellation fee is applied to subsequent cancellations regardless of timing |

## API Interactions

| Step | API                             | Method | Endpoint                    | Description                             |
|------|---------------------------------|--------|-----------------------------|-----------------------------------------|
| 1    | mobile-experience-api           | PATCH  | `/rides/{id}/cancel`        | Entry point for cancellation            |
| 2    | cancel-ride-process-api         | PATCH  | `/rides/{id}/status`        | Orchestrates cancellation logic         |
| 3    | db-system-api                   | GET    | `/rides/{id}`               | Retrieves current ride state            |
| 4    | square-system-api               | PUT    | `/orders/{id}`              | Releases payment hold (refund)          |
| 5    | square-system-api               | POST   | `/payments/{id}/commit`     | Commits payment (charge)                |
| 6    | db-system-api                   | PATCH  | `/rides/{id}`               | Updates ride status to Canceled         |
| 7    | push-notifications-system-api   | POST   | `/notification`             | Notifies the other party                |
| 8    | request-ride-process-api        | POST   | `/rides/{id}/reassign`      | Triggers driver reassignment (driver cancel only) |

## New Process API Required

A new **`cancel-ride-process-api`** (Process layer) should be added to the API Application Catalogue to handle the orchestration logic for ride cancellations. This API encapsulates:

- Cancellation rule evaluation
- Payment hold release or commit decision
- Ride status update
- Notification dispatch
- Driver reassignment trigger (when applicable)
