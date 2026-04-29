# Sign Up – Edge Conditions & Alternative Flows

This document supplements the [Sign Up User use case diagram](../diagrams/use-cases/sign-up-user-use-case.drawio.svg) with edge conditions and alternative flows that are not covered in the diagram.

## Edge Conditions

### EC-1: Email Verification Code Expiration

**Trigger:** The email verification code expires before the user enters it.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | User receives verification code via email                                                |
| 2    | Verification code has a TTL of **10 minutes**                                            |
| 3    | If the user enters the code after expiration, the system returns: "This verification code has expired. Please request a new one." |
| 4    | User taps "Resend Code" in the app                                                       |
| 5    | A new code is generated and sent via `email-system-api` (`POST /emails`)                 |
| 6    | The old code is immediately invalidated                                                  |

### EC-2: Verification Code Resend

**Trigger:** User did not receive the verification email or needs a new code.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | User taps "Resend Code" on the verification screen                                       |
| 2    | System enforces a **60-second cooldown** between resend requests to prevent abuse        |
| 3    | A new verification code is generated; the previous code is invalidated                   |
| 4    | Maximum of **5 resend attempts** within a 1-hour window                                  |
| 5    | After 5 attempts, user is shown: "Too many attempts. Please try again in 1 hour or contact support." |

### EC-3: Email Already Registered

**Trigger:** User attempts to sign up with an email that already exists in the system.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | User enters their email in the registration form                                         |
| 2    | `sign-up-user-process-api` checks `okta-system-api` (`GET /users?email={email}`)        |
| 3    | If the email already exists, the system returns: "This email is already registered. Please log in or use a different email." |
| 4    | A "Go to Login" button is provided for convenience                                       |

### EC-4: Invalid Email Format or Phone Number

**Trigger:** User enters malformed data in the registration form.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | App performs client-side validation on email format and phone number                      |
| 2    | If email is invalid → "Please enter a valid email address."                              |
| 3    | If phone number is invalid → "Please enter a valid phone number with country code."      |
| 4    | Server-side validation is also performed by `mobile-experience-api` as a second check    |

### EC-5: Okta Service Unavailable During Registration

**Trigger:** Okta is down when the user tries to register.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | `sign-up-user-process-api` attempts to create user in Okta but receives an error         |
| 2    | System retries up to 3 times with exponential backoff                                    |
| 3    | If all retries fail: "Registration service is temporarily unavailable. Please try again in a few minutes." |
| 4    | No partial user record is created; the registration is fully rolled back                 |

### EC-6: Square Customer Creation Failure

**Trigger:** The Square customer profile cannot be created during sign-up.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | User account is created in Okta successfully, but `square-system-api` (`POST /customers`) fails |
| 2    | System retries up to 3 times                                                             |
| 3    | If all retries fail, the user account is still created but flagged as "payment_setup_pending" |
| 4    | The user can proceed to the main menu but cannot request a ride until the Square customer is created |
| 5    | A background job retries the Square customer creation every 5 minutes                    |
| 6    | If the user attempts to request a ride before payment setup completes: "Please wait while we finish setting up your payment profile." |

### EC-7: Network Loss During Registration

**Trigger:** User loses connectivity during the sign-up process.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | **Before form submission:** App shows: "No internet connection. Please check your connection." Form data is preserved locally |
| 2    | **During form submission:** Request times out. App shows: "Connection lost. Retrying..." Auto-retries when connectivity returns |
| 3    | **After submission but before verification:** User account may have been created. On reconnection, app checks if the email is already registered and resumes verification flow |
| 4    | **During verification code entry:** Code entry fails. On reconnection, user can retry the code (if still within TTL) or request a new code |

### EC-8: User Abandons Registration Mid-Way

**Trigger:** User starts registration but closes the app before completing.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | **Before Okta account creation:** No data is persisted server-side; nothing to clean up  |
| 2    | **After Okta account creation but before email verification:** Account exists in Okta with status "pending_verification" |
| 3    | If the user returns to the app within 7 days, they can resume verification               |
| 4    | After 30 days without verification, the unverified account is deactivated in Okta        |
| 5    | The user can re-register with the same email (deactivated account is cleaned up)         |

## Verification Code Specification

| Parameter            | Value                                          |
|----------------------|------------------------------------------------|
| Code format          | 6-digit numeric code                           |
| Code TTL             | 10 minutes                                     |
| Resend cooldown      | 60 seconds                                     |
| Max resend attempts  | 5 per hour                                     |
| Delivery method      | Email (primary); SMS (future enhancement)      |
| Code storage         | Stored in `db-system-api` with expiration timestamp |
