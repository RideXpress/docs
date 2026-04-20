# Real-time In-Ride Tracking

> **Diagram:** [Real-time In-Ride Tracking Flow](../diagrams/use-cases/real-time-ride-tracking-use-case.drawio.svg)

## Overview

| Field              | Details                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Use Case ID**    | UC-RIDE-TRACK-01                                                        |
| **Priority**       | Medium                                                                  |
| **Actors**         | Client (Rider), Driver                                                  |
| **Trigger**        | A ride transitions to "In route" status (passenger has boarded)        |
| **Preconditions**  | Ride is active with status "In route"; OTP verification is complete    |
| **Postconditions** | Client can track the ride in real-time on a map until destination      |

## Description

As a client, I want the ability to track the progress of my ride on a map in real-time during the trip (while "In route") so I can see the route being taken, the estimated time of arrival at my destination, and share my trip with trusted contacts.

The existing "Wait for Ride" use case covers tracking before pickup. This use case covers tracking from the moment the passenger boards until the ride is completed.

## Main Flow

1. Passenger boards the vehicle; OTP is verified; ride status changes to "In route."
2. Client's app switches to the "In-Ride" view showing:
   - Live map with the vehicle's current position (updated every 5 seconds).
   - Route to the destination (highlighted on the map).
   - Estimated time of arrival (ETA) at the destination.
   - Distance remaining.
3. Driver's app continues broadcasting location every **10 seconds** via `mobile-experience-api` (`POST /drivers/{id}/location`).
4. Client's app polls for location updates every **5 seconds** via `mobile-experience-api` (`GET /rides/{id}/live-location`).
5. As the vehicle approaches the destination (within 500 meters), the client receives: "Almost there! Arriving at your destination shortly."
6. When the driver marks the ride as complete, the client's app transitions to the ride summary screen.

## Trip Sharing

1. Client taps "Share Trip" during the ride.
2. A shareable link is generated that shows a live map view of the ride (read-only).
3. Client shares the link via SMS, messaging app, or email.
4. The link recipient can view:
   - Driver's name, vehicle info, and license plate.
   - Real-time vehicle location on a map.
   - ETA at the destination.
5. The shared link expires when the ride is completed.

## Edge Conditions

| # | Condition                               | Behavior                                                   |
|---|-----------------------------------------|------------------------------------------------------------|
| 1 | GPS signal lost (driver)                | Last known position is shown; "Location updating..." indicator displayed |
| 2 | Network loss (client)                   | Last known position shown; map freezes with "Reconnecting..." overlay. On reconnection, map snaps to current position |
| 3 | Driver deviates from route              | If the driver deviates significantly (> 1 km from expected route), client receives an alert: "Your driver has taken a different route." Safety button is highlighted |
| 4 | App goes to background (client)         | Location tracking continues server-side; client receives push notification with ETA updates at key milestones |
| 5 | Very long ride (> 2 hours)              | ETA is recalculated periodically based on current traffic; client is notified if the ETA changes by more than 15 minutes |
| 6 | Trip sharing link accessed after ride ends | Shows a static summary of the completed ride route and timeline |

## API Interactions

| Step | API                      | Method | Endpoint                           | Description                          |
|------|--------------------------|--------|------------------------------------|--------------------------------------|
| 1    | mobile-experience-api    | POST   | `/drivers/{id}/location`           | Driver broadcasts location           |
| 2    | mobile-experience-api    | GET    | `/rides/{id}/live-location`        | Client polls for live location       |
| 3    | mobile-experience-api    | POST   | `/rides/{id}/share`                | Generate trip sharing link           |
| 4    | mobile-experience-api    | GET    | `/rides/{id}/shared-view`          | Public endpoint for shared trip view |
| 5    | google-maps-system-api   | GET    | `/routes/{ride_id}/eta`            | Recalculate ETA during ride          |
| 6    | push-notifications-system-api | POST | `/notification`                 | ETA and route deviation alerts       |
