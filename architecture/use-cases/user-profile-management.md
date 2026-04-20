# User Profile Management

## Overview

| Field              | Details                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Use Case ID**    | UC-PROFILE-01                                                           |
| **Priority**       | Medium                                                                  |
| **Actors**         | Client (Rider), Driver                                                  |
| **Trigger**        | User wants to view or update their personal information                 |
| **Preconditions**  | User is authenticated and logged in                                     |
| **Postconditions** | User profile is updated in the system                                   |

## Description

As a user (client or driver), I want the ability to view and update my personal profile information so that my account stays current. Drivers additionally need to manage vehicle information and banking details.

## Client Profile Fields

| Field              | Editable | Validation                                          |
|--------------------|----------|-----------------------------------------------------|
| First name         | Yes      | Required, 1-100 characters                          |
| Last name          | Yes      | Required, 1-100 characters                          |
| Email              | Yes      | Valid email format; triggers re-verification         |
| Phone              | Yes      | Valid phone with country code; triggers SMS verification |
| Profile photo      | Yes      | JPEG/PNG, max 10 MB                                 |

## Driver Profile Fields (in addition to client fields)

| Field              | Editable | Validation                                          |
|--------------------|----------|-----------------------------------------------------|
| Vehicle make       | Yes      | Required                                            |
| Vehicle model      | Yes      | Required                                            |
| Vehicle year       | Yes      | Must meet minimum year requirement                  |
| Vehicle color      | Yes      | Required                                            |
| License plate      | Yes      | Required; change triggers document re-verification  |
| Vehicle photo      | Yes      | JPEG/PNG, max 10 MB                                 |
| Driver's license   | Yes      | Upload triggers re-verification                     |
| Insurance document | Yes      | Upload triggers re-verification                     |
| Banking info       | Yes      | Takes effect in the next payout cycle               |

## Main Flow

1. User navigates to "Profile" in the app.
2. App retrieves current profile data from `mobile-experience-api` (`GET /users/{id}/profile`).
3. User edits desired fields and taps "Save."
4. App sends updated data to `mobile-experience-api` (`PATCH /users/{id}/profile`).
5. Process API updates the data in `db-system-api` and syncs to relevant external systems (Okta for email, Square for payment profile).
6. User sees confirmation: "Profile updated successfully."

## Edge Conditions

| # | Condition                                | Behavior                                                |
|---|------------------------------------------|---------------------------------------------------------|
| 1 | Email change                             | New email requires verification (verification code sent); old email remains active until new email is verified |
| 2 | Phone change                             | New phone requires SMS verification                      |
| 3 | Driver license plate change              | Triggers document re-verification; driver can continue operating during review |
| 4 | Banking info change                      | New info takes effect in the next payout cycle           |
| 5 | Profile photo upload fails               | Retry up to 3 times; user is notified of the failure    |

## API Interactions

| Step | API                      | Method | Endpoint                | Description                     |
|------|--------------------------|--------|-------------------------|---------------------------------|
| 1    | mobile-experience-api    | GET    | `/users/{id}/profile`   | Retrieve current profile        |
| 2    | mobile-experience-api    | PATCH  | `/users/{id}/profile`   | Update profile fields           |
| 3    | okta-system-api          | PATCH  | `/users/{id}`           | Sync email/name changes to Okta |
| 4    | square-system-api        | PATCH  | `/customers/{id}`       | Sync profile changes to Square  |
| 5    | db-system-api            | PATCH  | `/users/{id}`           | Persist profile changes         |
