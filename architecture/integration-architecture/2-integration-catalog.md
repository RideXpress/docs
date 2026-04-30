

# Integration Catalog

The Integration Catalog provides a high-level overview of the integration flows for the system.

<a href="../../architecture/diagrams/entity-diagrams/integration-context-diagram.drawio.svg">
  <img src="../../architecture/diagrams/entity-diagrams/integration-context-diagram.drawio.svg" alt="alt text" width="800"/>
</a>

The catalog is structured to be driven by the inbound triggers, where a single integration flow will lead to outbound interactions with one or more target systems. It's important to note that multiple source systems can trigger the same integration flow.

## Main Integration Use Cases

- [User sign up](#user-sign-up)
- [User login / Authentication](#user-login--authentication)
- [Request ride](#request-a-ride)
- [Accepting ride](#accepting-ride)
- [Wait for ride](#wait-for-ride)
- [Finish ride](#finish-ride)
- [Ride cancellation](#ride-cancellation)
- [Driver sign up / Onboarding](#driver-sign-up--onboarding)
- [Driver availability](#driver-availability)
- [Ride pricing](#ride-pricing)
- [Payment failure handling](#payment-failure-handling)
- [Driver payout / Earnings](#driver-payout--earnings)

### User sign up 

- Description: As a user I want the ability to sign up to the application using a Mobile App.
The app will send a verification code to the registered email and/or phone.
- Timeliness: Near real-time the first response, the email verification will depend on the user.
- Business Trigger: A new user trying to register in the platform.
- Source System(s): N/A

<a href="../../architecture/diagrams/use-cases/sign-up-user-use-case.drawio.svg">
  <img src="../../architecture/diagrams/use-cases/sign-up-user-use-case.drawio.svg" alt="alt text" width="300"/>
</a>

### Request a ride

- Description: As a customer I want the ability to select a destination in the Mobile App and the App will show the price based on X criteria and also the estimated arrival time.
- Timeliness: Near real-time; ride creation, driver matching, and payment hold are processed within seconds.
- Business Trigger: A new Ride is requested by a customer.
- Source System(s): N/A

<a href="../../architecture/diagrams/use-cases/request-ride-use-case.drawio.svg">
  <img src="../../architecture/diagrams/use-cases/request-ride-use-case.drawio.svg" alt="alt text" width="300"/>
</a>



### Accepting ride

- Description: As a driver, I want the ability to accept or deny any inbound ride.
- Timeliness: Near real-time; driver must respond within the broadcast timeout window.
- Business Trigger: A ride is accepted by a driver to be made.
- Source System(s): N/A

<a href="../../architecture/diagrams/use-cases/accepting-ride-use-case.drawio.svg">
  <img src="../../architecture/diagrams/use-cases/accepting-ride-use-case.drawio.svg" alt="alt text" width="300"/>
</a>


### Wait for ride

- Description: As a customer I want the ability to see the location of the driver to know when it will be near my pick up location.
- Timeliness: Real-time; driver location updates every 10 seconds, client polls every 5 seconds.
- Business Trigger: A driver has accepted a ride and is en route to the pickup location.
- Source System(s): N/A

<a href="../../architecture/diagrams/use-cases/wait-for-ride-use-case.drawio.svg">
  <img src="../../architecture/diagrams/use-cases/wait-for-ride-use-case.drawio.svg" alt="alt text" width="300"/>
</a>



### Finish ride

- Description: As a driver I want the ability to finish the drive and as a user, I want the ability to confirm the end of the trip by scanning a QR code, using an NFC, or some other method.
- Timeliness: Near real-time; payment commit and ride status update occur within seconds of the driver marking the ride complete.
- Business Trigger: A ride is finished, and the payment must be confirmed.
- Source System(s): N/A

<a href="../../architecture/diagrams/use-cases/finishing-ride-use-case.drawio.svg">
  <img src="../../architecture/diagrams/use-cases/finishing-ride-use-case.drawio.svg" alt="alt text" width="300"/>
</a>


### User login / Authentication

- Description: As a returning user (client or driver), I want the ability to log in to the RideXpress mobile app so that I can access ride services without re-registering. The app should securely authenticate me via Okta, manage my session, and allow me to stay logged in across app restarts.
- Timeliness: Near real-time; authentication should complete within seconds.
- Business Trigger: A returning user opens the app and needs to authenticate.
- Source System(s): N/A

📄 [View full use case documentation](../../architecture/use-cases/user-login-authentication.md)


### Ride cancellation

- Description: As a client or driver, I want the ability to cancel a ride. The system must enforce cancellation policies that determine whether the client is charged or refunded, and handle ride reassignment when a driver cancels.
- Timeliness: Near real-time; cancellation processing, payment refund/charge, and notifications occur within seconds.
- Business Trigger: A client or driver initiates a cancellation of an active ride.
- Source System(s): N/A

📄 [View full use case documentation](../../architecture/use-cases/ride-cancellation.md)


### Driver sign up / Onboarding

- Description: As a driver, I want the ability to sign up on the RideXpress platform with vehicle registration, license verification, background check, and banking setup so I can start accepting ride requests.
- Timeliness: Registration is near real-time; background check and document verification may take 1-3 business days.
- Business Trigger: A new driver wants to register on the platform.
- Source System(s): N/A

📄 [View full use case documentation](../../architecture/use-cases/driver-sign-up-onboarding.md)


### Driver availability

- Description: As a driver, I want the ability to toggle my status to "Online" or "Offline" so the system knows when I am available to accept ride requests and can track my real-time location.
- Timeliness: Real-time; status change is immediate, location updates every 10 seconds.
- Business Trigger: Driver wants to start or stop accepting ride requests.
- Source System(s): N/A

📄 [View full use case documentation](../../architecture/use-cases/driver-availability.md)


### Ride pricing

- Description: As a client, I want to see the estimated price of a ride before confirming the request. The system calculates fares based on distance, time, base fare, and surge pricing.
- Timeliness: Near real-time; fare estimate is calculated and returned within seconds.
- Business Trigger: Client selects a destination and requests a fare estimate.
- Source System(s): N/A

📄 [View full use case documentation](../../architecture/use-cases/ride-pricing.md)


### Payment failure handling

- Description: As the system, I need to handle payment failures (hold, commit, refund) gracefully with retry logic, client notifications, and background recovery queues.
- Timeliness: Immediate retry (up to 3 times); background retry every 15 minutes for up to 24-48 hours.
- Business Trigger: A payment operation fails at any stage of the ride lifecycle.
- Source System(s): N/A

📄 [View full use case documentation](../../architecture/use-cases/payment-failure-handling.md)


### Driver payout / Earnings

- Description: As a driver, I want the ability to view my earnings and receive payouts to my bank account on a regular schedule. Earnings are calculated per ride and disbursed weekly.
- Timeliness: Earnings recorded in real-time per ride; payouts processed weekly (Monday).
- Business Trigger: A ride is completed and payment is committed; or the weekly payout cycle runs.
- Source System(s): N/A

📄 [View full use case documentation](../../architecture/use-cases/driver-payout-earnings.md)
