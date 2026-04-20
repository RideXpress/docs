# Driver Sign Up / Onboarding

> **Diagram:** [Driver Sign Up / Onboarding Flow](../diagrams/use-cases/driver-sign-up-onboarding-use-case.drawio.svg)

## Overview

| Field              | Details                                                                    |
|--------------------|----------------------------------------------------------------------------|
| **Use Case ID**    | UC-DRIVER-01                                                               |
| **Priority**       | Critical                                                                   |
| **Actors**         | Driver (applicant)                                                         |
| **Trigger**        | A new driver wants to register on the RideXpress platform                  |
| **Preconditions**  | Driver has a valid driver's license, vehicle, insurance, and a smartphone  |
| **Postconditions** | Driver account is created, vehicle is registered, background check is initiated, and banking information is stored for payouts |

## Description

As a driver, I want the ability to sign up on the RideXpress platform so that I can start accepting ride requests. The driver onboarding flow is distinct from the client sign-up flow because it requires additional verification steps: vehicle registration, driver's license validation, insurance verification, background check initiation, and banking/payout setup.

## Main Flow

1. Driver opens the RideXpress app and selects "Sign Up as Driver."
2. Driver completes the registration form:
   - Full name, email, phone number, profile photo.
3. App sends registration data to `mobile-experience-api` → `sign-up-user-process-api` (`POST /users`).
4. Process API creates the user in `okta-system-api` with role = "driver."
5. Process API creates a customer profile in `square-system-api` (`POST /customers`) for future payouts.
6. Process API stores driver data in `db-system-api` (`POST /drivers`).
7. Email verification code is sent via `email-system-api` (`POST /emails`).
8. Driver enters verification code in the app (`POST /users/{id}/code`).
9. Upon email verification, driver is redirected to the **Vehicle Registration** step.

### Vehicle Registration (Step 2 of Onboarding)

10. Driver enters vehicle details:
    - Make, model, year, color, license plate number, vehicle photo.
11. App sends vehicle data to `mobile-experience-api` → `sign-up-user-process-api` (`POST /drivers/{id}/vehicle`).
12. Data is stored in `db-system-api`.

### License & Insurance Verification (Step 3 of Onboarding)

13. Driver uploads:
    - Driver's license photo (front and back).
    - Vehicle insurance document.
14. App sends documents to `mobile-experience-api` → `sign-up-user-process-api` (`POST /drivers/{id}/documents`).
15. Documents are stored and a verification request is created in `db-system-api` with status = "pending_review."
16. A background check is initiated asynchronously (integration with a third-party verification service via `background-check-system-api` or manual review queue in Salesforce).

### Banking Setup (Step 4 of Onboarding)

17. Driver enters banking information:
    - Bank name, account holder name, routing number, account number.
18. App sends banking data to `mobile-experience-api` → `sign-up-user-process-api` (`POST /drivers/{id}/banking`).
19. Banking information is securely stored (sensitive data handling via tokenization; RideXpress does not store raw bank account numbers).

### Onboarding Completion

20. Driver's onboarding status is set to "pending_verification" in `db-system-api`.
21. Driver is notified: "Your application is under review. You will be notified once approved."
22. Once background check and document review are complete:
    - **Approved:** Driver status is set to "active"; push notification is sent: "Welcome to RideXpress! You can now go online and start accepting rides."
    - **Rejected:** Driver status is set to "rejected"; email is sent with the reason for rejection and instructions for resubmission if applicable.

## Alternative Flows

### AF-1: Document Re-upload (Rejection)

1. Uploaded documents fail verification (blurry image, expired license, etc.).
2. Driver is notified via push notification and email with the specific issue.
3. Driver re-uploads the corrected documents.
4. Verification process restarts for the re-uploaded documents only.

### AF-2: Incomplete Onboarding (Abandoned)

1. Driver completes registration (steps 1-9) but does not complete vehicle or document steps.
2. After 7 days, a reminder email is sent: "Complete your driver registration to start earning."
3. After 30 days of inactivity, the incomplete application is archived.
4. Driver can resume onboarding from where they left off by logging back in.

## Edge Conditions

| # | Condition                                | Behavior                                                                |
|---|------------------------------------------|-------------------------------------------------------------------------|
| 1 | Email already registered as a client     | System prompts: "This email is already registered as a rider. Would you like to add a driver profile?" If yes, the existing user gets a driver role added |
| 2 | Driver's license is expired              | Document verification fails; driver is asked to upload a valid license  |
| 3 | Background check returns a flag          | Application is routed to manual review queue; driver is notified of a delay |
| 4 | Banking info validation fails            | Driver is notified with specific error (e.g., invalid routing number); prompted to re-enter |
| 5 | Vehicle does not meet requirements       | System enforces minimum year (e.g., vehicles older than 15 years are rejected); driver is informed |
| 6 | Upload fails due to file size/format     | App validates file size (max 10 MB) and format (JPEG, PNG, PDF) before upload; error shown if invalid |

## API Interactions

| Step | API                          | Method | Endpoint                        | Description                              |
|------|------------------------------|--------|---------------------------------|------------------------------------------|
| 1    | mobile-experience-api        | POST   | `/drivers/signup`               | Entry point for driver registration      |
| 2    | sign-up-user-process-api     | POST   | `/users`                        | Creates user account                     |
| 3    | okta-system-api              | POST   | `/users`                        | Creates user in Okta with driver role    |
| 4    | square-system-api            | POST   | `/customers`                    | Creates customer for payout              |
| 5    | db-system-api                | POST   | `/drivers`                      | Stores driver profile                    |
| 6    | email-system-api             | POST   | `/emails`                       | Sends verification email                 |
| 7    | mobile-experience-api        | POST   | `/drivers/{id}/vehicle`         | Submits vehicle information              |
| 8    | mobile-experience-api        | POST   | `/drivers/{id}/documents`       | Uploads license and insurance docs       |
| 9    | mobile-experience-api        | POST   | `/drivers/{id}/banking`         | Submits banking information              |
| 10   | db-system-api                | PATCH  | `/drivers/{id}/status`          | Updates onboarding status                |
| 11   | push-notifications-system-api| POST   | `/notification`                 | Sends approval/rejection notification    |

## Differences from Client Sign Up

| Aspect                  | Client Sign Up                  | Driver Sign Up                               |
|-------------------------|----------------------------------|----------------------------------------------|
| Role assigned           | "client" / "rider"              | "driver"                                     |
| Vehicle registration    | Not required                    | Required (make, model, year, plate, photo)   |
| Document upload         | Not required                    | Required (license, insurance)                |
| Background check        | Not required                    | Required                                     |
| Banking setup           | Not required                    | Required (for payouts)                       |
| Approval process        | Immediate (after email verify)  | Delayed (pending document & background check)|
| Square integration      | Payment method (charge)         | Payout recipient (receive)                   |
