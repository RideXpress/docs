# Driver Availability (Go Online / Offline)

## Overview

| Field              | Details                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Use Case ID**    | UC-DRIVER-02                                                            |
| **Priority**       | Critical                                                                |
| **Actors**         | Driver                                                                  |
| **Trigger**        | Driver wants to start or stop accepting ride requests                   |
| **Preconditions**  | Driver has an active, approved account and is logged in                 |
| **Postconditions** | Driver availability status is updated; location broadcasting starts or stops |

## Description

As a driver, I want the ability to toggle my availability status to "Online" or "Offline" so that the system knows when I am available to accept ride requests. When online, the driver's real-time location is periodically broadcast to the server so that the system can match them with nearby ride requests.

## Main Flow: Going Online

1. Driver opens the app and taps the "Go Online" button.
2. App requests GPS/location permissions (if not already granted).
3. App sends an availability update to `mobile-experience-api` → `driver-availability-process-api` (`PATCH /drivers/{id}/status` with `status = "online"`).
4. Process API updates the driver's status in `db-system-api` (`PATCH /drivers/{id}`).
5. App begins periodic location broadcasting:
   - Every **10 seconds**, the app sends the driver's GPS coordinates to `mobile-experience-api` (`POST /drivers/{id}/location`).
   - Location data is stored in `db-system-api` with a timestamp.
6. Driver's status is displayed as "Online" in the app; the driver is now eligible to receive ride requests.

## Main Flow: Going Offline

1. Driver taps the "Go Offline" button.
2. App sends an availability update to `mobile-experience-api` → `driver-availability-process-api` (`PATCH /drivers/{id}/status` with `status = "offline"`).
3. Process API updates the driver's status in `db-system-api`.
4. App stops periodic location broadcasting.
5. Driver is no longer eligible to receive ride requests.

## Alternative Flows

### AF-1: Automatic Offline (Idle Timeout)

1. Driver is online but has not moved or interacted with the app for **30 minutes**.
2. System sends a push notification: "You have been idle for a while. Are you still available?"
3. If the driver does not respond within **5 minutes**, the system automatically sets their status to "offline."
4. Location broadcasting stops.

### AF-2: Automatic Offline (Stale Location)

1. The system has not received a location update from a driver for **2 minutes** (e.g., due to GPS failure or app being killed by the OS).
2. System marks the driver's location as "stale" in `db-system-api`.
3. Stale drivers are excluded from ride matching queries.
4. If no location update is received for **10 minutes**, the driver is automatically set to "offline."
5. When the driver reopens the app, they must explicitly go online again.

### AF-3: Cannot Go Online (Active Ride in Progress)

1. Driver attempts to go offline while they have an active ride (status: "In route").
2. System prevents the status change: "You have an active ride. Complete the ride before going offline."
3. After the ride is completed, the driver can go offline.

### AF-4: Cannot Go Online (Account Issue)

1. Driver attempts to go online but their account status is "suspended" or "pending_verification."
2. System displays: "Your account is not currently active. Please contact support."
3. Status change is blocked.

## Stale Location Detection Logic

| Condition                          | Action                                                         |
|------------------------------------|----------------------------------------------------------------|
| No update for 2 minutes            | Mark driver location as "stale"; exclude from ride matching    |
| No update for 10 minutes           | Auto-set driver to "offline"; stop including in any queries    |
| Location jump > 200 km/h velocity  | Flag as GPS anomaly; use last known valid location             |
| GPS accuracy > 100 meters          | Mark location as "low accuracy"; deprioritize in ride matching |

## Location Broadcasting Specification

| Parameter          | Value                                             |
|--------------------|---------------------------------------------------|
| Frequency          | Every 10 seconds while online                     |
| Data transmitted   | Latitude, longitude, heading, speed, accuracy, timestamp |
| Protocol           | HTTPS POST to REST API                            |
| Battery optimization | Reduced frequency (every 30 seconds) when stationary for > 5 minutes |
| Background mode    | App must support background location updates (iOS/Android) |

## Edge Conditions

| # | Condition                              | Behavior                                                         |
|---|----------------------------------------|------------------------------------------------------------------|
| 1 | GPS permission denied                  | App prompts: "Location services are required to go online. Please enable GPS in settings." Status remains offline |
| 2 | App killed by OS                       | Location updates stop; stale detection kicks in after 2 minutes  |
| 3 | Phone battery critically low (< 10%)   | App sends warning: "Low battery. Consider going offline to save power." No forced action |
| 4 | Driver goes online in unsupported area | System allows online status but warns: "You are outside the service area. You may not receive ride requests here." |
| 5 | Network connectivity loss              | App queues location updates locally; sends batch update when connectivity resumes; if offline for > 10 minutes, driver is auto-offlined |
| 6 | Multiple devices logged in             | Only the most recently active device can toggle online/offline; other devices show read-only status |

## API Interactions

| Step | API                           | Method | Endpoint                     | Description                            |
|------|-------------------------------|--------|------------------------------|----------------------------------------|
| 1    | mobile-experience-api         | PATCH  | `/drivers/{id}/status`       | Toggle online/offline                  |
| 2    | driver-availability-process-api | PATCH | `/drivers/{id}/availability` | Orchestrates availability logic        |
| 3    | db-system-api                 | PATCH  | `/drivers/{id}`              | Updates driver status in database      |
| 4    | mobile-experience-api         | POST   | `/drivers/{id}/location`     | Periodic location update               |
| 5    | db-system-api                 | POST   | `/driver-locations`          | Stores location with timestamp         |
| 6    | push-notifications-system-api | POST   | `/notification`              | Sends idle timeout warning             |

## New Process API Required

A new **`driver-availability-process-api`** (Process layer) should be added to the API Application Catalogue to handle:

- Online/offline status transitions with validation
- Stale location detection and automatic offline logic
- Idle timeout management
- Integration with the ride matching system to include/exclude drivers
