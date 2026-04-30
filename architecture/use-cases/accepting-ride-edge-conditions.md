# Accepting Ride – Edge Conditions & Alternative Flows

This document supplements the [Accepting Ride use case diagram](../diagrams/use-cases/accepting-ride-use-case.drawio.svg) with edge conditions and alternative flows that are not covered in the diagram.

## Edge Conditions

### EC-1: Passenger Verification (OTP Mismatch)

**Trigger:** The passenger provides an incorrect one-time token when boarding the vehicle.

The ride-states diagram references that "the passenger is not giving the correct one-time token when getting into the car to the driver (means the driver never picked up the correct passenger)." This verification flow should work as follows:

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | When the ride is created, a one-time token (OTP) is generated and shared with both the client and the driver (displayed in their respective apps) |
| 2    | When the client arrives at the vehicle, the driver requests the OTP                       |
| 3    | **Match:** Driver enters/confirms the OTP in the app → Ride proceeds to "In route" status |
| 4    | **Mismatch (attempt 1):** Driver enters wrong OTP → App displays: "Token does not match. Please verify the passenger." Driver can retry |
| 5    | **Mismatch (attempt 2):** Second wrong OTP → Same message. Driver can retry once more    |
| 6    | **Mismatch (attempt 3):** Third wrong OTP → Ride is flagged. System prompts: "Unable to verify passenger. Would you like to cancel this ride?" |
| 7    | If the driver cancels due to OTP mismatch → Ride is canceled **with charge** to the unverified client (wrong passenger attempted boarding) |
| 8    | If the actual client contacts support, a dispute resolution process is initiated          |

### EC-2: Client Not at Pickup Location (5-Minute Wait Rule)

**Trigger:** Driver arrives at pickup but the client has not boarded after the waiting period.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | System detects driver arrival (GPS within 50 meters of pickup)                           |
| 2    | Ride status changes to "Waiting for passenger"                                           |
| 3    | A **5-minute countdown** starts                                                          |
| 4    | At 0:00 → Client notification: "Your driver has arrived at the pickup location."         |
| 5    | At 2:00 → Client notification: "Your driver is waiting. 3 minutes remaining."            |
| 6    | At 4:00 → Client notification: "Last minute warning. Your driver will leave in 1 minute."|
| 7    | At 5:00 → Driver can tap "Rider No-Show" in the app                                     |
| 8    | Ride is canceled **with charge** (cancellation fee applied to client)                    |
| 9    | Driver is freed to accept new rides                                                      |

### EC-3: Driver Cannot Find the Client

**Trigger:** Driver is near the pickup location but cannot visually locate the client.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | Driver taps "Contact Rider" in the app                                                   |
| 2    | An in-app call or message is initiated to the client (phone number masked for privacy)   |
| 3    | If the client responds and provides updated location → Driver proceeds to the new spot   |
| 4    | If the client does not respond within 5 minutes → Driver can mark as "No-show" (see EC-2)|

### EC-4: Multiple Passengers (Ride Sharing Scenario)

**Trigger:** Multiple people attempt to board the vehicle.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | OTP verification ensures only the correct client (and their party) boards                |
| 2    | The driver's app shows the number of expected passengers (if provided during ride request)|
| 3    | If more passengers than expected attempt to board, the driver can decline and contact support |

### EC-5: Client Wants to Change Pickup Location After Driver Accepted

**Trigger:** After a driver has accepted the ride and is en route, the client wants to change the pickup point.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | Client taps "Change Pickup" in the app                                                   |
| 2    | The new pickup location is sent to `mobile-experience-api`                               |
| 3    | The system recalculates the driver's ETA to the new location                             |
| 4    | If the new location is within a reasonable range (< 2 km from original), the pickup is updated and the driver is notified |
| 5    | If the new location is significantly different, the client is prompted: "This is far from the original pickup. Your driver may cancel. Continue?" |
| 6    | Driver receives: "Rider has changed the pickup location. New ETA: X minutes. Accept or cancel?" |

### EC-6: Driver Accepts but Vehicle Breaks Down

**Trigger:** Driver experiences a vehicle issue after accepting but before picking up the client.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | Driver cancels the ride with reason "Vehicle issue"                                      |
| 2    | Client is not charged (driver-initiated cancellation)                                    |
| 3    | Ride is reassigned to another driver automatically (see [Ride Cancellation](ride-cancellation.md)) |
| 4    | Driver is prompted to go offline; repeated vehicle-issue cancellations may trigger a review |

## OTP Verification Specification

| Parameter           | Value                                                          |
|---------------------|----------------------------------------------------------------|
| OTP format          | 4-digit numeric code                                           |
| OTP generation      | Generated at ride creation; stored in `db-system-api`          |
| OTP display         | Shown to client in the "Waiting for ride" screen; shown to driver in the "Accepting ride" screen |
| OTP validity        | Valid for the duration of the ride (from creation to boarding)  |
| Max OTP attempts    | 3 attempts before ride is flagged                              |
| OTP verification    | Driver enters OTP via `PATCH /rides/{id}` with `otp` field    |
