

# Integration Catalog

The Integration Catalog provides a high-level overview of the integration flows for the system.

<a href="../../architecture/diagrams/entity-diagrams/integration-context-diagram.drawio.svg">
  <img src="../../architecture/diagrams/entity-diagrams/integration-context-diagram.drawio.svg" alt="alt text" width="800"/>
</a>

The catalog is structured to be driven by the inbound triggers, where a single integration flow will lead to outbound interactions with one or more target systems. It's important to note that multiple source systems can trigger the same integration flow.

## Main Integration Use Cases

- [User sign up](#user-sign-up)
- [Request ride](#request-a-ride)
- [Accepting ride](#accepting-ride)
- [Wait for ride](#wait-for-ride)
- [Finish ride](#finish-ride)

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
- Timeliness: 
- Business Trigger: A new Ride is requestes by a customer.
- Source System(s): N/A

<a href="../../architecture/diagrams/use-cases/request-ride-use-case.drawio.svg">
  <img src="../../architecture/diagrams/use-cases/request-ride-use-case.drawio.svg" alt="alt text" width="300"/>
</a>



### Accepting ride

- Description: As a driver, I want the ability to accept or deny any inbound ride.
- Timeliness: 
- Business Trigger: A ride is accepted by a driver to be made.
- Source System(s): N/A

<a href="../../architecture/diagrams/use-cases/accepting-ride-use-case.drawio.svg">
  <img src="../../architecture/diagrams/use-cases/accepting-ride-use-case.drawio.svg" alt="alt text" width="300"/>
</a>


### Wait for ride

- Description: As a customer I want the ability to see the location of the driver to know when it will be near my pick up location.
- Timeliness: 
- Business Trigger: N/A
- Source System(s): N/A

<a href="../../architecture/diagrams/use-cases/wait-for-ride-use-case.drawio.svg">
  <img src="../../architecture/diagrams/use-cases/wait-for-ride-use-case.drawio.svg" alt="alt text" width="300"/>
</a>



### Finish ride

- Description: As a driver I want the ability to finish the drive and as a user, I want the ability to confirm the end of the trip by scanning a QR code, using an NFC, or some other method.
- Timeliness: 
- Business Trigger: A ride is finished, and the payment must be confirmed.
- Source System(s): N/A

<a href="../../architecture/diagrams/use-cases/finishing-ride-use-case.drawio.svg">
  <img src="../../architecture/diagrams/use-cases/finishing-ride-use-case.drawio.svg" alt="alt text" width="300"/>
</a>
