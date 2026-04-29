# Notification Preferences

> **Diagram:** [Notification Preferences Flow](../diagrams/use-cases/notification-preferences-use-case.drawio.svg)

## Overview

| Field              | Details                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Use Case ID**    | UC-NOTIF-01                                                             |
| **Priority**       | Low                                                                     |
| **Actors**         | Client (Rider), Driver                                                  |
| **Trigger**        | User wants to manage their push notification and email preferences      |
| **Preconditions**  | User is authenticated and logged in                                     |
| **Postconditions** | Notification preferences are updated; future notifications respect the settings |

## Description

As a user, I want the ability to manage my notification preferences so I can control which notifications I receive via push and email.

## Notification Types

| Notification                     | Default (Push) | Default (Email) | Mandatory |
|----------------------------------|---------------|-----------------|-----------|
| Ride status updates              | On            | Off             | Yes       |
| Driver/ride assigned             | On            | Off             | Yes       |
| Payment confirmation             | On            | On              | Yes       |
| Ride receipt                     | Off           | On              | No        |
| Promotions and offers            | On            | On              | No        |
| Weekly earnings summary (driver) | On            | On              | No        |
| Payout confirmation (driver)     | On            | On              | Yes       |
| Safety alerts                    | On            | On              | Yes       |
| Account security (password change, new device login) | On | On     | Yes       |
| Support ticket updates           | On            | On              | No        |

**Mandatory notifications** cannot be disabled as they are critical for ride operations and safety.

## Main Flow

1. User navigates to "Settings" → "Notifications" in the app.
2. App retrieves current preferences from `mobile-experience-api` (`GET /users/{id}/notification-preferences`).
3. User toggles individual notification types on or off for push and email channels.
4. User taps "Save."
5. App sends updated preferences to `mobile-experience-api` (`PATCH /users/{id}/notification-preferences`).
6. Preferences are stored in `db-system-api`.
7. All future notifications check the user's preferences before sending.

## Edge Conditions

| # | Condition                                   | Behavior                                                 |
|---|---------------------------------------------|----------------------------------------------------------|
| 1 | User disables all non-mandatory push notifications | Only mandatory notifications are sent via push       |
| 2 | Push notification delivery fails             | Retry once; if still failing, log the failure but do not fall back to email unless the user has email enabled for that notification type |
| 3 | User uninstalls the app                      | Push token becomes invalid; system detects delivery failure and stops sending push until the user re-registers |
| 4 | OS-level notification permission denied      | App prompts: "Notifications are disabled in your device settings. Enable them to receive ride updates." Links to device settings |
| 5 | Email bounces                                | After 3 bounces, email notifications are auto-disabled for the user; user is prompted to update their email on next login |

## API Interactions

| Step | API                      | Method | Endpoint                                  | Description                        |
|------|--------------------------|--------|-------------------------------------------|------------------------------------|
| 1    | mobile-experience-api    | GET    | `/users/{id}/notification-preferences`    | Retrieve current preferences       |
| 2    | mobile-experience-api    | PATCH  | `/users/{id}/notification-preferences`    | Update preferences                 |
| 3    | db-system-api            | PATCH  | `/user-preferences/{id}`                  | Persist preferences                |
| 4    | push-notifications-system-api | POST | `/notification`                          | Sends notification (checks prefs)  |
| 5    | email-system-api         | POST   | `/emails`                                 | Sends email (checks prefs)         |
