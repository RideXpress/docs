# Ride Type Selection

## Overview

| Field              | Details                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Use Case ID**    | UC-RIDE-TYPE-01                                                         |
| **Priority**       | Medium                                                                  |
| **Actors**         | Client (Rider)                                                          |
| **Trigger**        | Client wants to select a ride type before confirming a ride request     |
| **Preconditions**  | Client is logged in and has entered pickup and destination locations    |
| **Postconditions** | A ride is requested with the selected ride type; pricing reflects the type |

## Description

As a client, I want the ability to choose between different ride types (e.g., Economy, Comfort, Premium) so I can select the option that best fits my needs and budget.

## Ride Types

| Ride Type   | Description                                                | Pricing Multiplier | Vehicle Requirements                    |
|-------------|------------------------------------------------------------|--------------------|------------------------------------------|
| Economy     | Affordable everyday rides                                  | 1.0x (base rate)   | Any approved vehicle                     |
| Comfort     | Newer vehicles with more legroom                           | 1.3x               | Vehicle year ≥ 2020, sedan or larger     |
| Premium     | Luxury vehicles with top-rated drivers                     | 2.0x               | Vehicle year ≥ 2022, luxury brands only  |

## Main Flow

1. Client enters pickup and destination locations.
2. App requests fare estimates for all available ride types from `mobile-experience-api` (`POST /rides/estimate`).
3. `request-ride-process-api` calculates the fare for each ride type by applying the type-specific pricing multiplier.
4. App displays a list of available ride types with:
   - Ride type name and description
   - Estimated fare
   - Estimated arrival time (ETA of nearest eligible driver)
   - Vehicle icon
5. Client selects a ride type and taps "Confirm."
6. Ride request proceeds with the selected type; only drivers with matching vehicle criteria are eligible.

## Alternative Flows

### AF-1: No Drivers for Selected Type

1. Client selects "Premium" but no Premium-eligible drivers are available in the area.
2. App shows: "No Premium drivers available right now. Try Economy or Comfort."
3. The unavailable ride type is grayed out but still visible with the estimated fare.

### AF-2: All Types Unavailable

1. No drivers of any type are available.
2. App shows: "No drivers are available in your area right now. Please try again later."

## Edge Conditions

| # | Condition                               | Behavior                                                     |
|---|-----------------------------------------|--------------------------------------------------------------|
| 1 | Surge pricing applies                   | Surge multiplier stacks with ride type multiplier            |
| 2 | Driver vehicle no longer qualifies      | If a driver's vehicle drops below the type requirement (e.g., year ages out), they are excluded from that type's pool |
| 3 | Type selected but fare estimate expires | Client must re-estimate; the ride type selection is preserved |

## API Interactions

| Step | API                      | Method | Endpoint              | Description                              |
|------|--------------------------|--------|-----------------------|------------------------------------------|
| 1    | mobile-experience-api    | POST   | `/rides/estimate`     | Returns fare estimates for all ride types |
| 2    | request-ride-process-api | POST   | `/pricing/calculate`  | Calculates per-type fare                 |
| 3    | db-system-api            | GET    | `/drivers?type={type}`| Queries eligible drivers by ride type    |
| 4    | mobile-experience-api    | POST   | `/rides`              | Creates ride with selected type          |
