# Driver Payout / Earnings

## Overview

| Field              | Details                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Use Case ID**    | UC-PAY-02                                                               |
| **Priority**       | High                                                                    |
| **Actors**         | Driver, System                                                          |
| **Trigger**        | A ride is completed and payment is committed; or the weekly payout cycle runs |
| **Preconditions**  | Driver has an active account with verified banking information          |
| **Postconditions** | Driver's earnings are calculated, recorded, and transferred to their bank account |

## Description

As a driver, I want the ability to view my earnings from completed rides and receive payouts to my bank account on a regular schedule. The entity diagram references "Banking Information" and "Routing Number to pay" for drivers, but there is no documented flow for how earnings are calculated, accumulated, or disbursed.

## Earnings Calculation

After each completed ride, the driver's earnings are calculated as follows:

| Component            | Description                                                        |
|----------------------|--------------------------------------------------------------------|
| Base fare            | Percentage of the total ride fare (e.g., 75% to driver, 25% to RideXpress) |
| Tip                  | 100% of any tip goes to the driver                                 |
| Cancellation fee     | When a rider is charged a cancellation fee, a portion may go to the driver (if they were en route) |
| Deductions           | Any applicable deductions (platform fees, insurance contributions)  |
| **Net earnings**     | Base fare + Tip + Cancellation portion - Deductions                |

## Main Flow: Earnings Tracking

1. A ride is completed and payment is committed via `square-system-api`.
2. `finish-ride-process-api` calculates the driver's earnings for that ride.
3. Earnings record is created in `db-system-api` (`POST /drivers/{id}/earnings`) with:
   - Ride ID, gross fare, platform commission, tip amount, net earnings, timestamp.
4. Driver's cumulative balance is updated in `db-system-api`.
5. Driver can view earnings in the app:
   - **Today's earnings** (real-time running total).
   - **Weekly earnings** (current payout period).
   - **Earnings history** (past payouts with itemized ride breakdown).

## Main Flow: Weekly Payout

1. A scheduled job runs every **Monday at 00:00 UTC** (configurable).
2. The job queries `db-system-api` for all drivers with a positive balance (`GET /drivers/payouts/pending`).
3. For each driver with earnings >= minimum payout threshold (**$10.00**):
   - A payout request is created via `square-system-api` (`POST /payouts`) with the driver's banking information.
   - The payout record is stored in `db-system-api` (`POST /drivers/{id}/payouts`) with status = "processing."
4. Square processes the bank transfer (typically 1-3 business days).
5. Upon confirmation from Square (webhook or polling), the payout status is updated to "completed."
6. Driver is notified via `push-notifications-system-api`: "Your weekly payout of $XX.XX has been deposited."

## Alternative Flows

### AF-1: Instant Payout (Future Enhancement)

1. Driver requests an instant payout from the app (subject to a small processing fee, e.g., $0.50).
2. System validates the driver has sufficient balance.
3. An instant transfer is initiated via Square.
4. Funds are available within minutes (depending on the bank).

### AF-2: Payout Below Minimum Threshold

1. Driver's accumulated earnings are below the minimum payout threshold ($10.00).
2. Earnings roll over to the next payout cycle.
3. Driver is informed: "Your current balance of $X.XX will be included in your next payout once it reaches $10.00."

### AF-3: Banking Information Invalid or Missing

1. Payout job attempts to process a driver's payout but banking info is missing or invalid.
2. Payout is skipped for this cycle; status is set to "failed."
3. Driver is notified: "We could not process your payout. Please update your banking information."
4. Earnings remain in the driver's balance until banking info is corrected and the next payout cycle runs.

### AF-4: Earnings Dispute

1. Driver believes an earnings calculation is incorrect.
2. Driver contacts support via the Help section (see Help / Support use case).
3. Support reviews the ride details, fare, and commission breakdown.
4. If an adjustment is warranted, a manual earnings correction is applied to the driver's balance.

## Edge Conditions

| # | Condition                                | Behavior                                                           |
|---|------------------------------------------|--------------------------------------------------------------------|
| 1 | Square payout API fails                  | Payout is queued for retry in the next 6 hours; retries up to 3 times. If all fail, flagged for manual processing |
| 2 | Driver changes bank info mid-payout cycle | New banking info takes effect in the next payout cycle; current cycle uses the info on file at cycle start |
| 3 | Negative balance (refund after payout)   | If a refund is issued after the driver has been paid, the refund amount is deducted from the next payout |
| 4 | Driver account deactivated before payout | Pending earnings are held for 30 days; if no reactivation, a final payout is processed to the last known banking info |
| 5 | Duplicate payout prevention              | Each payout cycle generates a unique idempotency key per driver to prevent double payments |
| 6 | Tax reporting                            | Annual earnings summaries are generated for tax purposes; stored in `db-system-api` and accessible via driver earnings history |

## API Interactions

| Step | API                           | Method | Endpoint                          | Description                               |
|------|-------------------------------|--------|-----------------------------------|-------------------------------------------|
| 1    | finish-ride-process-api       | POST   | `/drivers/{id}/earnings`          | Calculates and records ride earnings      |
| 2    | db-system-api                 | POST   | `/driver-earnings`                | Stores earnings record                    |
| 3    | db-system-api                 | GET    | `/drivers/{id}/earnings`          | Retrieves earnings history                |
| 4    | db-system-api                 | GET    | `/drivers/payouts/pending`        | Lists drivers eligible for payout         |
| 5    | square-system-api             | POST   | `/payouts`                        | Initiates bank transfer                   |
| 6    | db-system-api                 | POST   | `/driver-payouts`                 | Records payout transaction                |
| 7    | push-notifications-system-api | POST   | `/notification`                   | Notifies driver of payout                 |
| 8    | mobile-experience-api         | GET    | `/drivers/{id}/earnings/summary`  | Returns earnings dashboard data           |

## Earnings Dashboard (Mobile App)

The driver's earnings dashboard should display:

| Section             | Data                                                    |
|---------------------|---------------------------------------------------------|
| Today's Earnings    | Running total of rides completed today with ride count  |
| This Week           | Cumulative earnings for the current payout period       |
| Pending Payout      | Amount scheduled for next payout                        |
| Payout History      | List of past payouts with date, amount, and status      |
| Ride Breakdown      | Per-ride detail: fare, commission, tip, net earnings    |
