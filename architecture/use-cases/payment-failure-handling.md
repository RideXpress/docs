# Payment Failure Handling

> **Diagram:** [Payment Failure Handling Flow](../diagrams/use-cases/payment-failure-handling-use-case.drawio.svg)

## Overview

| Field              | Details                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Use Case ID**    | UC-PAY-01                                                               |
| **Priority**       | Critical                                                                |
| **Actors**         | Client (Rider), System                                                  |
| **Trigger**        | A payment operation (hold, commit, or refund) fails during the ride lifecycle |
| **Preconditions**  | A ride has been requested and a payment link was generated via Square    |
| **Postconditions** | Payment failure is handled gracefully; client is notified; ride proceeds or is canceled based on the failure type |

## Description

As the system, I need to handle payment failures at every stage of the ride lifecycle — payment hold creation, payment commit (charge), and refund processing. The current documentation covers the happy path (payment link created, payment committed at ride end) but does not address what happens when Square returns errors, cards are declined, or the payment link expires.

## Payment Stages and Failure Scenarios

### Stage 1: Payment Hold at Ride Request

During the "Request Ride" flow, a payment link is created via Square (`POST /payment-link`) and the estimated ride amount is placed on hold.

#### Failure Scenarios

| # | Failure                                | System Response                                                        |
|---|----------------------------------------|------------------------------------------------------------------------|
| 1 | Card declined (insufficient funds)     | Client is notified: "Your payment method was declined. Please update your payment method and try again." Ride request is not created |
| 2 | Payment link creation fails (Square error) | System retries up to 3 times with exponential backoff (1s, 2s, 4s). If all retries fail, client is notified: "We could not process your payment. Please try again later." |
| 3 | Payment link expires (not completed by client) | Square payment links expire after a configurable timeout. If the client does not complete payment within **10 minutes**, the ride request is auto-canceled. Client is notified: "Your ride was canceled because payment was not completed in time." |
| 4 | Square API is completely unavailable   | After 3 retries, system returns: "Payment services are temporarily unavailable. Please try again shortly." Ride is not created. An alert is triggered for operations monitoring |

### Stage 2: Payment Commit at Ride Completion

During the "Finish Ride" flow, the payment hold is committed (charged) via Square (`POST /payments/{id}/commit`).

#### Failure Scenarios

| # | Failure                                | System Response                                                        |
|---|----------------------------------------|------------------------------------------------------------------------|
| 1 | Commit fails (Square error)            | System retries up to 3 times with exponential backoff. Ride is still marked as "Finished" regardless of payment status |
| 2 | All commit retries fail                | Payment is placed in a **"pending_collection"** queue in `db-system-api`. A background job retries the commit every **15 minutes** for up to **24 hours** |
| 3 | Payment hold has expired               | Square payment holds typically expire after 7 days. If the hold has expired, the system must create a new charge instead of committing the hold. If the new charge also fails, the amount is flagged as **"uncollected"** for manual resolution |
| 4 | Amount adjustment needed (tip added)   | If the final amount (with tip) exceeds the original hold, a supplemental charge is created. If the supplemental charge fails, the original hold is committed for the base amount and the tip is added to the pending_collection queue |

### Stage 3: Refund at Ride Cancellation

During the "Ride Cancellation" flow, the payment hold is released via Square (`PUT /orders/{id}` with state = CANCELED).

#### Failure Scenarios

| # | Failure                                | System Response                                                        |
|---|----------------------------------------|------------------------------------------------------------------------|
| 1 | Refund/release fails (Square error)    | System retries up to 3 times. Client is notified: "Your refund is being processed and will appear in 3-5 business days." |
| 2 | All refund retries fail                | Refund is placed in a **"pending_refund"** queue. A background job retries every **15 minutes** for up to **48 hours**. If still failing, flagged for manual processing |
| 3 | Partial refund needed                  | If a cancellation fee applies, only the fee portion is committed and the remainder is released. If the partial operation fails, the full amount is queued for manual resolution |

## Retry Strategy

All payment operations follow a consistent retry strategy:

| Parameter           | Value                                   |
|---------------------|-----------------------------------------|
| Max retries         | 3 (immediate)                           |
| Backoff strategy    | Exponential (1s, 2s, 4s)               |
| Idempotency         | All Square requests include an `idempotencyKey` to prevent duplicate charges |
| Background retry    | Every 15 minutes for up to 24-48 hours  |
| Dead letter queue   | After max background retries, flagged for manual resolution |
| Monitoring          | Alert triggered after 3 consecutive failures for any operation type |

## Client-Facing Error Messages

| Scenario                        | Message                                                              |
|---------------------------------|----------------------------------------------------------------------|
| Card declined                   | "Your payment method was declined. Please update your payment method and try again." |
| Payment service unavailable     | "Payment services are temporarily unavailable. Please try again in a few minutes." |
| Payment link expired            | "Your ride was canceled because payment was not completed in time."   |
| Refund pending                  | "Your refund is being processed and will appear within 3-5 business days." |
| Charge pending collection       | "Your ride is complete. Payment will be processed shortly."          |

## Edge Conditions

| # | Condition                               | Behavior                                                           |
|---|-----------------------------------------|--------------------------------------------------------------------|
| 1 | Client's card is removed from Square during ride | Payment commit fails; system sends notification to add a valid payment method; amount goes to pending_collection |
| 2 | Currency mismatch                       | Square validates currency at link creation; mismatch is caught early and surfaced as a configuration error |
| 3 | Double charge due to retry              | Prevented by `idempotencyKey` on all Square API calls              |
| 4 | Ride completed but app crashes before payment commit | Backend timer triggers payment commit independently of app state; ride status and payment status are decoupled |
| 5 | Network timeout during payment call     | Treated as a failure; retry logic applies; idempotency key prevents duplicate charges |

## API Interactions

| Step | API                        | Method | Endpoint                   | Description                              |
|------|----------------------------|--------|----------------------------|------------------------------------------|
| 1    | square-system-api          | POST   | `/payment-link`            | Creates payment link with hold           |
| 2    | square-system-api          | POST   | `/payments/{id}/commit`    | Commits payment hold at ride end         |
| 3    | square-system-api          | PUT    | `/orders/{id}`             | Releases payment hold (refund/cancel)    |
| 4    | db-system-api              | POST   | `/payment-failures`        | Logs payment failure for retry           |
| 5    | db-system-api              | GET    | `/payment-failures/pending`| Background job reads pending retries     |
| 6    | push-notifications-system-api | POST | `/notification`            | Notifies client of payment issues        |

## Monitoring and Alerting

| Metric                              | Alert Threshold                         |
|--------------------------------------|-----------------------------------------|
| Payment hold failure rate            | > 5% of requests in a 15-minute window  |
| Payment commit failure rate          | > 2% of requests in a 15-minute window  |
| Pending collection queue size        | > 100 items                             |
| Pending refund queue size            | > 50 items                              |
| Square API response time             | > 5 seconds (P95)                       |
| Square API availability              | < 99.9% uptime in rolling 1-hour window |
