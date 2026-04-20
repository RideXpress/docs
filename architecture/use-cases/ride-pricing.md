# Ride Pricing / Fare Estimation

> **Diagram:** [Ride Pricing / Fare Estimation Flow](../diagrams/use-cases/ride-pricing-use-case.drawio.svg)

## Overview

| Field              | Details                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Use Case ID**    | UC-RIDE-PRICE-01                                                        |
| **Priority**       | High                                                                    |
| **Actors**         | Client (Rider), System                                                  |
| **Trigger**        | Client selects a destination and requests a fare estimate               |
| **Preconditions**  | Client is logged in and has selected a pickup and drop-off location     |
| **Postconditions** | A fare estimate is displayed to the client before they confirm the ride |

## Description

As a client, I want to see the estimated price of a ride before confirming the request, so that I can make an informed decision. The "Request Ride" use case states "the App will show the price based on X criteria" but does not document the pricing logic, fare components, or surge pricing rules.

## Fare Components

The ride fare is calculated based on the following components:

| Component          | Description                                                      | Example         |
|--------------------|------------------------------------------------------------------|-----------------|
| Base fare          | Fixed charge applied to every ride                               | $2.50           |
| Per-mile rate      | Charge per mile of estimated route distance                      | $1.25 / mile    |
| Per-minute rate    | Charge per minute of estimated ride duration                     | $0.20 / minute  |
| Minimum fare       | Minimum charge regardless of distance or time                    | $5.00           |
| Booking fee        | Platform fee applied to every ride                               | $1.50           |
| Surge multiplier   | Dynamic multiplier applied during high-demand periods            | 1.0x - 3.0x    |
| **Estimated total** | max(Base + Distance + Time, Minimum) × Surge + Booking fee     |                 |

## Main Flow: Fare Estimation

1. Client enters pickup location and destination in the mobile app.
2. App sends a fare estimate request to `mobile-experience-api` (`POST /rides/estimate`).
3. `request-ride-process-api` calls `google-maps-system-api` (`POST /geolocation`) to:
   - Calculate the route distance (in miles/km).
   - Calculate the estimated travel time (in minutes).
4. `request-ride-process-api` retrieves current pricing parameters from `db-system-api` (`GET /pricing`):
   - Base fare, per-mile rate, per-minute rate, minimum fare, booking fee.
5. `request-ride-process-api` checks for surge pricing:
   - Queries `db-system-api` for current demand in the pickup area (`GET /pricing/surge?zone={zone_id}`).
   - If demand exceeds supply thresholds → surge multiplier is applied.
6. Fare is calculated:
   ```
   ride_fare = max(base_fare + (distance × per_mile_rate) + (duration × per_minute_rate), minimum_fare)
   estimated_total = (ride_fare × surge_multiplier) + booking_fee
   ```
7. Fare estimate is returned to the client with a breakdown:
   - Estimated fare range (± 15% to account for traffic variability).
   - Surge indicator (if applicable).
   - Estimated arrival time.
8. Client confirms the ride at the estimated price (fare is locked for **5 minutes**).

## Surge Pricing Logic

Surge pricing is activated when demand outpaces driver supply in a geographic zone.

| Condition                                  | Surge Multiplier |
|--------------------------------------------|------------------|
| Demand/supply ratio < 1.5                  | 1.0x (no surge)  |
| Demand/supply ratio 1.5 - 2.0             | 1.5x             |
| Demand/supply ratio 2.0 - 3.0             | 2.0x             |
| Demand/supply ratio > 3.0                  | 2.5x - 3.0x (cap)|

### Surge Zone Calculation

- The service area is divided into geographic zones (configurable grid or geofence-based).
- Demand = number of ride requests in the zone in the last **10 minutes**.
- Supply = number of online drivers in the zone in the last **5 minutes**.
- Surge multiplier is recalculated every **2 minutes**.
- Surge pricing is displayed to the client before confirmation: "Prices are higher than usual due to increased demand."

## Alternative Flows

### AF-1: Fare Estimate with No Surge

1. Demand/supply ratio is below the surge threshold.
2. Fare is calculated at the standard rate (1.0x multiplier).
3. No surge indicator is shown to the client.

### AF-2: Fare Locked Price Expires

1. Client receives a fare estimate but does not confirm within **5 minutes**.
2. Fare estimate expires; surge conditions may have changed.
3. Client must request a new estimate.
4. App displays: "Your fare estimate has expired. Please request a new estimate."

### AF-3: Route Unavailable

1. Google Maps cannot calculate a route (e.g., destination is inaccessible, over water, etc.).
2. System returns: "We cannot calculate a route to this destination. Please select a different location."
3. Ride request is not created.

## Edge Conditions

| # | Condition                              | Behavior                                                           |
|---|----------------------------------------|--------------------------------------------------------------------|
| 1 | Google Maps API unavailable            | System retries 3 times; if still failing, uses a simplified distance estimate (straight-line distance × 1.4 factor); fare is marked as "approximate" |
| 2 | Actual ride distance differs from estimate | Final fare is based on actual GPS-tracked distance/time; client is notified if the final fare exceeds the estimate by more than 20% |
| 3 | Surge changes between estimate and ride start | The locked fare at confirmation time is honored for the ride |
| 4 | Very short ride (below minimum)        | Minimum fare is applied                                            |
| 5 | Very long ride (> 50 miles)            | Fare is calculated normally; no cap. A confirmation prompt is shown: "This is a long ride. Estimated fare: $XX. Confirm?" |
| 6 | Pricing configuration missing          | System uses hardcoded default pricing; alerts operations team of missing config |

## API Interactions

| Step | API                          | Method | Endpoint                              | Description                            |
|------|------------------------------|--------|---------------------------------------|----------------------------------------|
| 1    | mobile-experience-api        | POST   | `/rides/estimate`                     | Requests fare estimate                 |
| 2    | request-ride-process-api     | POST   | `/pricing/calculate`                  | Orchestrates pricing logic             |
| 3    | google-maps-system-api       | POST   | `/geolocation`                        | Gets route distance and duration       |
| 4    | db-system-api                | GET    | `/pricing`                            | Retrieves pricing parameters           |
| 5    | db-system-api                | GET    | `/pricing/surge?zone={zone_id}`       | Gets current surge multiplier          |
| 6    | db-system-api                | POST   | `/fare-estimates`                     | Stores fare estimate with lock timeout |

## Final Fare vs. Estimated Fare

| Scenario                           | Final Fare Calculation                                   |
|------------------------------------|----------------------------------------------------------|
| Route matches estimate             | Final = Estimated (no adjustment)                        |
| Route shorter than estimated       | Final = Actual (client pays less)                        |
| Route longer due to traffic/detour | Final = Actual (up to 20% over estimate without approval)|
| Route > 20% over estimate          | Fare is capped at estimate + 20%; difference absorbed by platform unless route change was client-requested |
| Client-requested route change      | Fare is recalculated based on new route; client is notified |
