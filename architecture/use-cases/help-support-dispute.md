# Help / Support / Dispute Resolution

## Overview

| Field              | Details                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Use Case ID**    | UC-SUPPORT-01                                                           |
| **Priority**       | Medium                                                                  |
| **Actors**         | Client (Rider), Driver                                                  |
| **Trigger**        | User needs help, wants to report an issue, or disputes a charge        |
| **Preconditions**  | User is authenticated and logged in                                     |
| **Postconditions** | Support ticket is created and routed to the appropriate team            |

## Description

As a user (client or driver), I want the ability to access in-app support, report safety issues, and dispute charges so that my concerns are addressed promptly.

## Support Categories

| Category              | Description                                               | Priority |
|-----------------------|-----------------------------------------------------------|----------|
| Safety issue          | Report a safety concern during or after a ride            | Urgent   |
| Fare dispute          | Dispute a charge or request a fare adjustment             | High     |
| Lost item             | Report a lost item in a vehicle                           | Medium   |
| Account issue         | Login problems, account suspension, profile issues        | Medium   |
| Payment issue         | Payment failures, refund requests, payout problems        | High     |
| Driver/Rider behavior | Report inappropriate behavior                             | High     |
| App bug               | Report a technical issue with the app                     | Low      |
| General inquiry       | General questions about the service                       | Low      |

## Main Flow: Submit a Support Request

1. User navigates to "Help" in the app menu.
2. App displays a list of support categories (see table above).
3. User selects a category and optionally a sub-category.
4. User fills in the details:
   - Description of the issue (free text, required)
   - Related ride (optional, auto-populated if accessed from ride history)
   - Attachments (optional, photos or screenshots)
5. App sends the request to `mobile-experience-api` (`POST /support/tickets`).
6. A support ticket is created in `salesforce-system-api` as a Salesforce Case.
7. User receives confirmation: "Your request has been submitted. Ticket #XXXX. We'll respond within [SLA time]."

## Main Flow: Fare Dispute

1. User navigates to a completed ride in Ride History.
2. User taps "Dispute Fare."
3. App shows the fare breakdown and asks: "What seems wrong?"
   - "I was charged too much"
   - "I was charged for a ride I didn't take"
   - "The route taken was incorrect"
   - "Other"
4. User selects a reason and provides additional details.
5. A dispute ticket is created with priority = High.
6. Support reviews the GPS track, route, and fare calculation.
7. If the dispute is valid:
   - A partial or full refund is issued via `square-system-api`.
   - User is notified: "Your dispute has been resolved. A refund of $X.XX has been issued."
8. If the dispute is not valid:
   - User is notified: "After review, the fare of $X.XX is correct. [Explanation]."

## Main Flow: Safety Report

1. During or after a ride, user taps the "Safety" shield icon.
2. App presents emergency options:
   - **Emergency (911):** Direct dial to local emergency services.
   - **Report safety concern:** Opens a support ticket with urgency = Urgent.
3. Safety reports are escalated immediately to the support team.
4. The reported user's account may be temporarily suspended pending review.

## SLA (Service Level Agreement)

| Priority | First Response | Resolution Target |
|----------|----------------|-------------------|
| Urgent   | 1 hour         | 4 hours           |
| High     | 4 hours        | 24 hours          |
| Medium   | 24 hours       | 72 hours          |
| Low      | 48 hours       | 5 business days   |

## Edge Conditions

| # | Condition                              | Behavior                                                    |
|---|----------------------------------------|-------------------------------------------------------------|
| 1 | User submits duplicate ticket          | System detects similar open ticket and shows: "You have an open ticket for this issue. View it?" |
| 2 | Dispute for a ride older than 30 days  | Dispute is accepted but marked as "late dispute"; refund approval requires manual review |
| 3 | Safety report during active ride       | Ride is flagged; if the client reports, the driver is notified that the ride is being monitored |
| 4 | Lost item contact                      | System facilitates anonymous contact between client and driver for 24 hours after the ride |

## API Interactions

| Step | API                      | Method | Endpoint                    | Description                         |
|------|--------------------------|--------|-----------------------------|-------------------------------------|
| 1    | mobile-experience-api    | POST   | `/support/tickets`          | Create support ticket               |
| 2    | mobile-experience-api    | GET    | `/support/tickets`          | List user's open tickets            |
| 3    | mobile-experience-api    | GET    | `/support/tickets/{id}`     | Get ticket details and status       |
| 4    | salesforce-system-api    | POST   | `/cases`                    | Create Salesforce Case              |
| 5    | square-system-api        | POST   | `/refunds`                  | Issue refund for fare dispute       |
| 6    | push-notifications-system-api | POST | `/notification`           | Notify user of ticket updates       |
