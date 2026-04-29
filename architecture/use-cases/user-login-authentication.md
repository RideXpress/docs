# User Login / Authentication

> **Diagram:** [User Login / Authentication Flow](../diagrams/use-cases/user-login-authentication-use-case.drawio.svg)

## Overview

| Field              | Details                                                       |
|--------------------|---------------------------------------------------------------|
| **Use Case ID**    | UC-AUTH-01                                                    |
| **Priority**       | Critical                                                      |
| **Actors**         | Client (Rider), Driver                                        |
| **Trigger**        | A returning user opens the mobile app and needs to authenticate |
| **Preconditions**  | User has a registered account (completed sign-up flow)        |
| **Postconditions** | User is authenticated and redirected to the app main menu     |

## Description

As a returning user (client or driver), I want the ability to log in to the RideXpress mobile app so that I can access ride services without re-registering. The app should securely authenticate me via Okta, manage my session, and allow me to stay logged in across app restarts.

## Main Flow

1. User opens the RideXpress mobile app.
2. App checks for a valid cached session token (stored securely on device).
3. **If valid token exists:** App validates the token with Okta (`GET /users/{id}/session`).
   - If token is still valid → Skip to step 8.
   - If token is expired → Proceed to step 4 (refresh flow).
4. **If no valid token or token expired:** App displays the login screen.
5. User enters their registered email and password.
6. App sends credentials to `mobile-experience-api` → `sign-up-user-process-api` → `okta-system-api` (`POST /authn`).
7. Okta validates credentials and returns an authentication token (OAuth 2.0 access token + refresh token).
8. App stores tokens securely on the device (Keychain on iOS, Keystore on Android).
9. App redirects the user to the main menu (client or driver view, based on the user role).

## Alternative Flows

### AF-1: Token Refresh

1. App detects that the access token has expired but a refresh token is available.
2. App sends refresh token to `okta-system-api` (`POST /token/refresh`).
3. Okta validates the refresh token and issues a new access token.
4. App updates the stored token and proceeds to the main menu.

### AF-2: Invalid Credentials

1. User enters incorrect email or password.
2. Okta returns an authentication error.
3. App displays an error message: "Invalid email or password. Please try again."
4. After 5 consecutive failed attempts, the account is temporarily locked for 15 minutes.
5. App displays: "Account temporarily locked. Please try again in 15 minutes or reset your password."

### AF-3: Password Reset

1. User selects "Forgot Password" on the login screen.
2. App sends a password reset request to `okta-system-api` (`POST /users/{email}/reset-password`).
3. Okta sends a password reset email to the registered address.
4. User follows the email link and sets a new password.
5. User returns to the login screen and logs in with the new password.

### AF-4: Multi-Device Login

1. User logs in from a new device while already logged in on another device.
2. The system allows concurrent sessions (configurable limit, default: 2 active devices).
3. If the device limit is exceeded, the oldest session is invalidated.
4. A push notification is sent to the invalidated device: "You have been logged out because you signed in on another device."

## Edge Conditions

| # | Condition                          | Behavior                                                                 |
|---|------------------------------------|--------------------------------------------------------------------------|
| 1 | Refresh token expired              | User is redirected to the login screen to re-enter credentials           |
| 2 | Okta service unavailable           | App displays: "Authentication service temporarily unavailable. Please try again shortly." App retries up to 3 times with exponential backoff |
| 3 | Network connectivity loss          | App displays offline message; cached session remains valid for read-only access to ride history; no new ride requests allowed |
| 4 | Account deactivated/suspended      | Okta returns account status; app displays: "Your account has been suspended. Please contact support." |
| 5 | App crash during login             | On restart, app returns to the login screen; no partial state is persisted |

## API Interactions

| Step | API                        | Method | Endpoint                    | Description                       |
|------|----------------------------|--------|-----------------------------|-----------------------------------|
| 1    | mobile-experience-api      | POST   | `/auth/login`               | Entry point for login request     |
| 2    | sign-up-user-process-api   | POST   | `/auth/authenticate`        | Orchestrates authentication flow  |
| 3    | okta-system-api            | POST   | `/authn`                    | Validates credentials with Okta   |
| 4    | okta-system-api            | POST   | `/token/refresh`            | Refreshes expired access token    |
| 5    | okta-system-api            | POST   | `/users/{email}/reset-password` | Initiates password reset      |
| 6    | db-system-api              | GET    | `/users/{id}`               | Retrieves user profile and role   |

## Security Considerations

- All authentication traffic must use HTTPS/TLS 1.2+.
- Access tokens have a short TTL (15 minutes); refresh tokens have a longer TTL (30 days).
- Refresh tokens are rotated on each use (one-time use).
- Passwords are never stored on the device or in the integration layer; authentication is delegated entirely to Okta.
- Failed login attempts are logged for audit and fraud detection.
