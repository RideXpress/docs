# Finish Ride – Edge Conditions & Alternative Flows

This document supplements the [Finish Ride use case diagram](../diagrams/use-cases/finishing-ride-use-case.drawio.svg) with edge conditions and alternative flows that are not covered in the diagram.

## Edge Conditions

### EC-1: Payment Commit Failure at Ride Completion

**Trigger:** The payment hold cannot be committed (charged) when the ride finishes.

See the [Payment Failure Handling](payment-failure-handling.md) use case for detailed scenarios. Summary:

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | Driver marks ride as complete; `finish-ride-process-api` attempts to commit payment      |
| 2    | `square-system-api` (`POST /payments/{id}/commit`) returns an error                     |
| 3    | System retries up to 3 times with exponential backoff                                    |
| 4    | If all retries fail, ride is still marked as "Finished" but payment status is "pending_collection" |
| 5    | A background job retries the commit every 15 minutes for up to 24 hours                  |
| 6    | Client sees: "Your ride is complete. Payment will be processed shortly."                 |
| 7    | Driver's earnings are still calculated and recorded (payout proceeds regardless)         |

### EC-2: Square API Downtime During Ride Completion

**Trigger:** Square is completely unavailable when the ride finishes.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | All payment operations (commit, tip processing) fail                                     |
| 2    | Ride is still completed from a ride-status perspective                                   |
| 3    | All payment operations are queued for background retry                                   |
| 4    | An operations alert is triggered for "Square API unavailable"                            |
| 5    | Client and driver can still complete the satisfaction survey                              |

### EC-3: App Crash During Ride Completion

**Trigger:** The driver's or client's app crashes during the ride finishing process.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | **Driver app crash:** The ride remains in "In route" status until the driver restarts the app |
| 2    | On restart, the app detects the active ride and displays the ride completion screen       |
| 3    | If the driver is at or near the destination (GPS within 100 meters), the app prompts: "It looks like you've arrived. Complete this ride?" |
| 4    | **Client app crash:** The ride completion is primarily driver-initiated; client app crash does not block completion |
| 5    | When the client restarts the app, they see the ride summary and can complete the satisfaction survey |
| 6    | **Backend safety net:** If a ride has been "In route" for longer than 2× the estimated duration and the driver is at the destination, the system sends a push notification: "Have you completed your ride? Tap to finish." |

### EC-4: Client Disputes the Final Fare

**Trigger:** The actual fare differs significantly from the estimate and the client is unhappy.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | Client reviews the fare on the ride summary screen                                       |
| 2    | If the fare exceeds the estimate by more than **20%**, a "Fare seems high?" link is shown |
| 3    | Client taps the link and sees a fare breakdown: base fare, distance, time, surge, tip    |
| 4    | If the client still disagrees, they can tap "Dispute Fare" which creates a support ticket |
| 5    | Until resolved, the original fare stands; the client's payment is committed              |
| 6    | Support reviews the GPS track, route, and pricing and may issue a partial refund         |

### EC-5: Tip Processing

**Trigger:** Client adds a tip after the ride is completed.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | After the ride is complete, client is shown tip options (e.g., $1, $2, $5, custom)       |
| 2    | Client selects a tip amount within the **24-hour tip window**                            |
| 3    | Tip is processed via `square-system-api` as a supplemental charge                        |
| 4    | If the tip charge fails, it is queued for retry (same retry logic as payment failures)   |
| 5    | The tip is recorded separately in the driver's earnings (`POST /drivers/{id}/earnings`)  |
| 6    | 100% of the tip goes to the driver (no platform commission on tips)                      |

### EC-6: Driver Ends Ride at Wrong Location

**Trigger:** The driver marks the ride as complete but they are not at the destination.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | `finish-ride-process-api` checks the driver's GPS against the ride destination           |
| 2    | If the driver is more than **500 meters** from the destination, the app warns: "You appear to be far from the destination. Are you sure you want to end this ride?" |
| 3    | Driver can confirm (legitimate early drop-off at client's request) or cancel the end     |
| 4    | If confirmed, the fare is calculated based on the actual distance traveled               |
| 5    | The ride is flagged for review if the actual distance is < 50% of the estimated distance |

### EC-7: Satisfaction Survey Not Completed

**Trigger:** Client or driver does not complete the post-ride satisfaction survey.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | After ride completion, both client and driver are prompted to rate each other             |
| 2    | The survey is optional; the ride completion is not blocked by survey submission           |
| 3    | If the survey is not completed within **24 hours**, the opportunity expires               |
| 4    | A reminder push notification is sent after **1 hour**: "How was your ride? Rate your experience." |
| 5    | Unrated rides do not affect the driver's or client's average rating                      |

## Post-Ride Data Flow

| Step | API                           | Method | Endpoint                      | Description                              |
|------|-------------------------------|--------|-------------------------------|------------------------------------------|
| 1    | mobile-experience-api         | PATCH  | `/rides/{id}/status`          | Driver marks ride as complete            |
| 2    | finish-ride-process-api       | PATCH  | `/rides/{id}`                 | Orchestrates completion flow             |
| 3    | square-system-api             | POST   | `/payments/{id}/commit`       | Commits payment hold                     |
| 4    | db-system-api                 | PATCH  | `/rides/{id}`                 | Updates ride status to "Finished"        |
| 5    | db-system-api                 | POST   | `/driver-earnings`            | Records driver's earnings for this ride  |
| 6    | push-notifications-system-api | POST   | `/notification`               | Sends ride summary to client             |
| 7    | mobile-experience-api         | POST   | `/rides/{id}/feedback`        | Client submits driver rating             |
| 8    | mobile-experience-api         | POST   | `/drivers/{id}/feedback`      | Driver submits client rating             |
