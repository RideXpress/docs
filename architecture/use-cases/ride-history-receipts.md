# Ride History & Receipts

> **Diagram:** [Ride History & Receipts Flow](../diagrams/use-cases/ride-history-receipts-use-case.drawio.svg)

## Overview

| Field              | Details                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Use Case ID**    | UC-HISTORY-01                                                           |
| **Priority**       | Medium                                                                  |
| **Actors**         | Client (Rider), Driver                                                  |
| **Trigger**        | User wants to view past rides or download a receipt                     |
| **Preconditions**  | User is authenticated and has at least one completed ride               |
| **Postconditions** | User can view ride details and receipt information                      |

## Description

As a user (client or driver), I want the ability to view my past rides, see trip details, and download receipts so I can track my ride history and expenses.

## Main Flow: View Ride History

1. User navigates to "Ride History" in the app.
2. App requests ride history from `mobile-experience-api` (`GET /users/{id}/rides?page=1&size=20`).
3. API returns a paginated list of rides sorted by date (most recent first).
4. Each ride in the list shows:
   - Date and time
   - Pickup and drop-off addresses
   - Fare amount
   - Driver/rider name and photo
   - Ride status (completed, canceled)
   - Rating given

## Main Flow: View Ride Details

1. User taps on a specific ride in the history list.
2. App requests ride details from `mobile-experience-api` (`GET /rides/{id}/details`).
3. Details screen shows:
   - Route map (pickup to drop-off with actual path)
   - Fare breakdown (base fare, distance, time, surge, booking fee, tip)
   - Payment method used
   - Driver/vehicle information (for clients) or client information (for drivers)
   - Rating and feedback given/received
   - Ride duration and distance

## Main Flow: Download Receipt

1. User taps "Download Receipt" on the ride details screen.
2. App requests receipt from `mobile-experience-api` (`GET /rides/{id}/receipt`).
3. Receipt includes:
   - RideXpress logo and company info
   - Ride date, time, and route
   - Itemized fare breakdown
   - Payment method (last 4 digits)
   - Total charged
   - Ride ID for reference
4. Receipt is available as a downloadable PDF or can be emailed to the user.

## Edge Conditions

| # | Condition                              | Behavior                                                    |
|---|----------------------------------------|-------------------------------------------------------------|
| 1 | No ride history                        | App shows: "No rides yet. Request your first ride!"         |
| 2 | Canceled ride in history               | Shown with "Canceled" badge; shows any cancellation fee     |
| 3 | Ride with pending payment              | Shown with "Payment pending" indicator                      |
| 4 | Large ride history (pagination)        | Infinite scroll with 20 rides per page                      |
| 5 | Receipt for canceled ride              | Receipt shows cancellation fee (if charged) or $0.00        |
| 6 | Offline access                         | Recently viewed rides are cached locally for offline viewing |

## API Interactions

| Step | API                      | Method | Endpoint                    | Description                        |
|------|--------------------------|--------|-----------------------------|------------------------------------|
| 1    | mobile-experience-api    | GET    | `/users/{id}/rides`         | Paginated ride history             |
| 2    | mobile-experience-api    | GET    | `/rides/{id}/details`       | Detailed ride information          |
| 3    | mobile-experience-api    | GET    | `/rides/{id}/receipt`       | Receipt data (PDF or JSON)         |
| 4    | db-system-api            | GET    | `/rides?user_id={id}`       | Query rides from database          |
| 5    | google-maps-system-api   | GET    | `/routes/{ride_id}`         | Retrieve route for map display     |
