# RideXpress Official Documentation

- [Contributing](contributing/README.md)
- [Integration Architecture](architecture/integration-architecture.md)
- [Anypoint Platform Architecture](architecture/anypoint-platform-architecture.md)
- [Archetype Design](architecture/archetype-design.md)
- [CI/CD Design](architecture/ci-cd-design.md)
- [Technical Design Document Template](architecture/tdd-template.md)
- [Diagrams](architecture/diagrams)
    - [Use cases](architecture/diagrams/use-cases/)
    - [API-led](architecture/diagrams/api-led/)
    - [High Level Flows](architecture/diagrams/high-level-flows/)
    - [Entities](architecture/diagrams/entity-diagrams/)

## Use Cases

### Core Use Cases (with diagrams)
- [User Sign Up](architecture/diagrams/use-cases/sign-up-user-use-case.drawio.svg) | [Edge Conditions](architecture/use-cases/sign-up-edge-conditions.md)
- [Request Ride](architecture/diagrams/use-cases/request-ride-use-case.drawio.svg) | [Edge Conditions](architecture/use-cases/request-ride-edge-conditions.md)
- [Accepting Ride](architecture/diagrams/use-cases/accepting-ride-use-case.drawio.svg) | [Edge Conditions](architecture/use-cases/accepting-ride-edge-conditions.md)
- [Wait for Ride](architecture/diagrams/use-cases/wait-for-ride-use-case.drawio.svg) | [Edge Conditions](architecture/use-cases/wait-for-ride-edge-conditions.md)
- [Finish Ride](architecture/diagrams/use-cases/finishing-ride-use-case.drawio.svg) | [Edge Conditions](architecture/use-cases/finish-ride-edge-conditions.md)
- [Ride States](architecture/diagrams/use-cases/ride-states.drawio.svg)

### Critical Missing Use Cases (new)
- [User Login / Authentication](architecture/use-cases/user-login-authentication.md)
- [Ride Cancellation (Client & Driver)](architecture/use-cases/ride-cancellation.md)
- [Driver Sign Up / Onboarding](architecture/use-cases/driver-sign-up-onboarding.md)
- [Driver Availability (Go Online/Offline)](architecture/use-cases/driver-availability.md)
- [Payment Failure Handling](architecture/use-cases/payment-failure-handling.md)
- [Driver Payout / Earnings](architecture/use-cases/driver-payout-earnings.md)
- [Ride Pricing / Fare Estimation](architecture/use-cases/ride-pricing.md)

### Additional Use Cases (new)
- [User Profile Management](architecture/use-cases/user-profile-management.md)
- [Ride History & Receipts](architecture/use-cases/ride-history-receipts.md)
- [Ride Type Selection](architecture/use-cases/ride-type-selection.md)
- [Help / Support / Dispute Resolution](architecture/use-cases/help-support-dispute.md)
- [Real-time In-Ride Tracking](architecture/use-cases/real-time-ride-tracking.md)
- [Notification Preferences](architecture/use-cases/notification-preferences.md)

### Reference
- [Entity Details (Field-Level)](architecture/use-cases/entity-details.md)