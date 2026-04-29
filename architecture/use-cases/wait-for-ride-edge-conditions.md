# Wait for Ride – Edge Conditions & Alternative Flows

This document supplements the [Wait for Ride use case diagram](../diagrams/use-cases/wait-for-ride-use-case.drawio.svg) with edge conditions and alternative flows that are not covered in the diagram.

## Edge Conditions

### EC-1: Driver Does Not Arrive in Expected Time

**Trigger:** The driver's actual arrival time exceeds the estimated ETA by a significant margin.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | System continuously compares the driver's live location with the estimated arrival time   |
| 2    | If the driver has not arrived within the ETA + **5-minute grace period**, the client is notified: "Your driver is running late. Would you like to continue waiting or cancel?" |
| 3    | If the client chooses to cancel → Ride is canceled **without charge** (see [Ride Cancellation](ride-cancellation.md)) |
| 4    | If the client continues waiting, the ETA is recalculated based on the driver's current location |
| 5    | If the driver still has not arrived after ETA + **15 minutes**, the system auto-cancels and triggers ride reassignment |

### EC-2: Client Not at Pickup Location

**Trigger:** The driver arrives at the pickup location but the client is not there.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | Driver arrives at the pickup GPS coordinates                                             |
| 2    | System detects arrival (driver location matches pickup within **50 meters**)             |
| 3    | A **5-minute countdown** begins                                                          |
| 4    | Client is notified: "Your driver has arrived. Please proceed to the pickup point."       |
| 5    | At the 2-minute mark, a second notification is sent: "Your driver is waiting. 3 minutes remaining." |
| 6    | If the client does not board within 5 minutes, the driver can mark the ride as "No-show" |
| 7    | Ride is canceled **with charge** (cancellation fee applied to the client)                |
| 8    | Driver is freed to accept new rides                                                      |

### EC-3: Driver Goes to Wrong Location

**Trigger:** The driver's GPS shows them at a different location than the pickup point.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | Client observes on the map that the driver is not heading toward or at the correct pickup location |
| 2    | Client can contact the driver via in-app communication (call or message)                 |
| 3    | If the driver cannot reach the correct location within a reasonable time, the client can cancel without charge |
| 4    | The ride is reassigned to another driver                                                 |

### EC-4: Network Connectivity Loss (Client)

**Trigger:** Client loses network during the "waiting for ride" phase.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | The client's app loses the ability to poll for driver location updates                   |
| 2    | App displays: "Connection lost. Reconnecting..." with the last known driver position     |
| 3    | The ride continues on the server side regardless of the client's connectivity            |
| 4    | When connectivity is restored, the app fetches the latest ride state (`GET /rides/{id}`) |
| 5    | If the driver has already arrived, the app shows the "Driver arrived" screen             |

### EC-5: Network Connectivity Loss (Driver)

**Trigger:** Driver loses network during transit to the pickup location.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | Driver's location updates stop being received by the server                              |
| 2    | After **2 minutes** without updates, the driver's location is marked as "stale"          |
| 3    | Client is shown the last known driver position with a note: "Driver location may be outdated" |
| 4    | After **10 minutes** without updates, the system considers the driver unreachable        |
| 5    | Client is prompted: "We've lost contact with your driver. Would you like to cancel and find a new driver?" |
| 6    | If canceled, no charge is applied to the client                                          |

### EC-6: Driver's App Crashes

**Trigger:** The driver's app crashes or is killed by the OS during transit.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | Location updates stop → stale location detection activates (same as EC-5)                |
| 2    | When the driver restarts the app, the app queries the server for active rides (`GET /drivers/{id}/active-ride`) |
| 3    | If an active ride exists, the app resumes the ride flow from the current state           |
| 4    | Location broadcasting restarts automatically                                             |

### EC-7: Ride State Recovery After App Crash (Client)

**Trigger:** The client's app crashes during the wait phase.

| Step | Action                                                                                   |
|------|------------------------------------------------------------------------------------------|
| 1    | Client reopens the app                                                                   |
| 2    | App checks for an active ride (`GET /rides?client_id={id}&status=active`)                |
| 3    | If an active ride is found, the app resumes the wait-for-ride screen with current driver location |
| 4    | No ride state is lost; the server is the source of truth                                 |

## Driver Location Polling Specification

| Parameter                    | Value                                                |
|------------------------------|------------------------------------------------------|
| Client polling frequency     | Every **5 seconds** (to display driver on map)       |
| Driver location push frequency | Every **10 seconds** (from driver app to server)   |
| Stale threshold              | **2 minutes** without updates                        |
| Arrival detection radius     | **50 meters** from pickup coordinates                |
