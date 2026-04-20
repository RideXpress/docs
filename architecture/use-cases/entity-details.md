# Entity Diagram – Field-Level Details

This document provides field-level detail, data types, and relationship cardinalities for the entities shown in the [Entity Diagram](../diagrams/entity-diagrams/entity-diagram.drawio.svg).

## Entity Relationship Summary

```
Users (1) ──── (0..*) Rides
Users (1) ──── (0..*) Payments
Users (1) ──── (0..1) Banking Information    [Drivers only]
Rides (1) ──── (2..*) Locations
Rides (1) ──── (1)    Payments
Rides (1) ──── (0..1) Receipts
```

## Entities

### Users

Represents both clients (riders) and drivers on the platform.

| Field              | Type          | Required | Description                                     |
|--------------------|---------------|----------|-------------------------------------------------|
| id                 | UUID          | Yes      | Unique identifier                               |
| okta_id            | String(50)    | Yes      | Reference to Okta user identity                 |
| square_customer_id | String(50)    | Yes      | Reference to Square customer profile            |
| email              | String(255)   | Yes      | User's email address (unique)                   |
| phone              | String(20)    | Yes      | User's phone number                             |
| first_name         | String(100)   | Yes      | First name                                      |
| last_name          | String(100)   | Yes      | Last name                                       |
| profile_photo_url  | String(500)   | No       | URL to profile photo                            |
| role               | Enum          | Yes      | `client`, `driver`, `both`                      |
| status             | Enum          | Yes      | `active`, `pending_verification`, `suspended`, `deactivated` |
| created_at         | Timestamp     | Yes      | Account creation timestamp                      |
| updated_at         | Timestamp     | Yes      | Last update timestamp                           |

### Driver Details (Extension of Users where role = driver)

| Field                  | Type          | Required | Description                                  |
|------------------------|---------------|----------|----------------------------------------------|
| user_id                | UUID (FK)     | Yes      | Reference to Users.id                        |
| license_number         | String(50)    | Yes      | Driver's license number                      |
| license_expiry         | Date          | Yes      | License expiration date                      |
| license_photo_url      | String(500)   | Yes      | URL to license photo (front)                 |
| insurance_doc_url      | String(500)   | Yes      | URL to insurance document                    |
| vehicle_make           | String(50)    | Yes      | Vehicle manufacturer                         |
| vehicle_model          | String(50)    | Yes      | Vehicle model                                |
| vehicle_year           | Integer       | Yes      | Vehicle year                                 |
| vehicle_color          | String(30)    | Yes      | Vehicle color                                |
| license_plate          | String(20)    | Yes      | Vehicle license plate number                 |
| vehicle_photo_url      | String(500)   | No       | URL to vehicle photo                         |
| background_check_status| Enum          | Yes      | `pending`, `approved`, `rejected`            |
| onboarding_status      | Enum          | Yes      | `incomplete`, `pending_verification`, `active`, `rejected` |
| availability_status    | Enum          | Yes      | `online`, `offline`                          |
| average_rating         | Decimal(3,2)  | No       | Average rating from riders (1.00 - 5.00)     |
| total_rides            | Integer       | Yes      | Total number of completed rides              |

### Rides

Represents a single ride from request to completion.

| Field              | Type          | Required | Description                                     |
|--------------------|---------------|----------|-------------------------------------------------|
| id                 | UUID          | Yes      | Unique ride identifier                          |
| client_id          | UUID (FK)     | Yes      | Reference to Users.id (rider)                   |
| driver_id          | UUID (FK)     | No       | Reference to Users.id (driver); null until assigned |
| status             | Enum          | Yes      | `new`, `waiting_for_driver`, `driving_to_passenger`, `waiting_for_passenger`, `in_route`, `finished`, `canceled_with_charge`, `canceled_without_charge` |
| otp                | String(4)     | Yes      | One-time token for passenger verification       |
| estimated_fare     | Decimal(10,2) | Yes      | Fare estimate at time of request                |
| actual_fare        | Decimal(10,2) | No       | Final fare after ride completion                |
| surge_multiplier   | Decimal(3,2)  | Yes      | Surge pricing multiplier (1.00 = no surge)      |
| estimated_distance | Decimal(10,2) | Yes      | Estimated distance in miles                     |
| actual_distance    | Decimal(10,2) | No       | Actual GPS-tracked distance in miles            |
| estimated_duration | Integer       | Yes      | Estimated duration in minutes                   |
| actual_duration    | Integer       | No       | Actual duration in minutes                      |
| cancellation_reason| String(255)   | No       | Reason for cancellation (if applicable)         |
| canceled_by        | Enum          | No       | `client`, `driver`, `system`                    |
| created_at         | Timestamp     | Yes      | Ride request timestamp                          |
| started_at         | Timestamp     | No       | Ride start timestamp (passenger boarded)        |
| completed_at       | Timestamp     | No       | Ride completion timestamp                       |

### Locations

Represents geographic coordinates associated with a ride or driver.

| Field              | Type          | Required | Description                                     |
|--------------------|---------------|----------|-------------------------------------------------|
| id                 | UUID          | Yes      | Unique identifier                               |
| ride_id            | UUID (FK)     | No       | Reference to Rides.id (null for driver locations)|
| driver_id          | UUID (FK)     | No       | Reference to Users.id (for driver live tracking) |
| type               | Enum          | Yes      | `pickup`, `dropoff`, `driver_live`, `passenger_live` |
| latitude           | Decimal(10,7) | Yes      | GPS latitude                                    |
| longitude          | Decimal(10,7) | Yes      | GPS longitude                                   |
| heading            | Decimal(5,2)  | No       | Direction of travel in degrees                  |
| speed              | Decimal(5,2)  | No       | Speed in km/h                                   |
| accuracy           | Decimal(5,2)  | No       | GPS accuracy in meters                          |
| address            | String(500)   | No       | Human-readable address (geocoded)               |
| recorded_at        | Timestamp     | Yes      | When this location was recorded                 |

### Payments

Represents payment transactions for rides.

| Field              | Type          | Required | Description                                     |
|--------------------|---------------|----------|-------------------------------------------------|
| id                 | UUID          | Yes      | Unique identifier                               |
| ride_id            | UUID (FK)     | Yes      | Reference to Rides.id                           |
| client_id          | UUID (FK)     | Yes      | Reference to Users.id (payer)                   |
| square_order_id    | String(50)    | Yes      | Square order ID                                 |
| square_payment_link| String(500)   | Yes      | Square payment link URL                         |
| amount_held        | Decimal(10,2) | Yes      | Amount placed on hold                           |
| amount_charged     | Decimal(10,2) | No       | Final amount charged (after ride)               |
| tip_amount         | Decimal(10,2) | No       | Tip amount (added post-ride)                    |
| currency           | String(3)     | Yes      | Currency code (e.g., `USD`)                     |
| status             | Enum          | Yes      | `hold_created`, `hold_failed`, `committed`, `commit_failed`, `refunded`, `refund_failed`, `pending_collection`, `pending_refund` |
| idempotency_key    | String(50)    | Yes      | Unique key to prevent duplicate charges         |
| created_at         | Timestamp     | Yes      | Payment creation timestamp                      |
| committed_at       | Timestamp     | No       | When payment was committed                      |

### Receipts

Represents ride receipts sent to clients.

| Field              | Type          | Required | Description                                     |
|--------------------|---------------|----------|-------------------------------------------------|
| id                 | UUID          | Yes      | Unique identifier                               |
| ride_id            | UUID (FK)     | Yes      | Reference to Rides.id                           |
| client_id          | UUID (FK)     | Yes      | Reference to Users.id                           |
| base_fare          | Decimal(10,2) | Yes      | Base fare component                             |
| distance_fare      | Decimal(10,2) | Yes      | Distance-based fare component                   |
| time_fare          | Decimal(10,2) | Yes      | Time-based fare component                       |
| surge_fare         | Decimal(10,2) | Yes      | Surge pricing amount (0 if no surge)            |
| booking_fee        | Decimal(10,2) | Yes      | Platform booking fee                            |
| tip_amount         | Decimal(10,2) | No       | Tip amount                                      |
| total_amount       | Decimal(10,2) | Yes      | Total charged amount                            |
| issued_at          | Timestamp     | Yes      | Receipt generation timestamp                    |

### Banking Information (Drivers only)

Stores driver banking details for payouts. Sensitive data is tokenized.

| Field              | Type          | Required | Description                                     |
|--------------------|---------------|----------|-------------------------------------------------|
| id                 | UUID          | Yes      | Unique identifier                               |
| driver_id          | UUID (FK)     | Yes      | Reference to Users.id                           |
| bank_name          | String(100)   | Yes      | Bank name                                       |
| account_holder     | String(200)   | Yes      | Account holder name                             |
| routing_number_token | String(100) | Yes      | Tokenized routing number (not stored in plain text) |
| account_number_token | String(100) | Yes      | Tokenized account number (not stored in plain text) |
| status             | Enum          | Yes      | `active`, `invalid`, `pending_verification`     |
| created_at         | Timestamp     | Yes      | Record creation timestamp                       |
| updated_at         | Timestamp     | Yes      | Last update timestamp                           |

### Driver Earnings

Tracks per-ride earnings for drivers.

| Field              | Type          | Required | Description                                     |
|--------------------|---------------|----------|-------------------------------------------------|
| id                 | UUID          | Yes      | Unique identifier                               |
| driver_id          | UUID (FK)     | Yes      | Reference to Users.id                           |
| ride_id            | UUID (FK)     | Yes      | Reference to Rides.id                           |
| gross_fare         | Decimal(10,2) | Yes      | Total ride fare                                 |
| platform_commission| Decimal(10,2) | Yes      | RideXpress platform commission                  |
| tip_amount         | Decimal(10,2) | No       | Tip received (100% to driver)                   |
| net_earnings       | Decimal(10,2) | Yes      | Driver's net earnings for this ride             |
| payout_id          | UUID (FK)     | No       | Reference to Driver Payouts.id (null until paid)|
| created_at         | Timestamp     | Yes      | Earnings record creation timestamp              |

### Driver Payouts

Tracks weekly payout transactions to drivers.

| Field              | Type          | Required | Description                                     |
|--------------------|---------------|----------|-------------------------------------------------|
| id                 | UUID          | Yes      | Unique identifier                               |
| driver_id          | UUID (FK)     | Yes      | Reference to Users.id                           |
| amount             | Decimal(10,2) | Yes      | Payout amount                                   |
| currency           | String(3)     | Yes      | Currency code (e.g., `USD`)                     |
| status             | Enum          | Yes      | `processing`, `completed`, `failed`             |
| payout_period_start| Date          | Yes      | Start of the payout period                      |
| payout_period_end  | Date          | Yes      | End of the payout period                        |
| square_payout_id   | String(50)    | No       | Square payout transaction ID                    |
| idempotency_key    | String(50)    | Yes      | Unique key to prevent duplicate payouts         |
| created_at         | Timestamp     | Yes      | Payout initiation timestamp                     |
| completed_at       | Timestamp     | No       | Payout completion timestamp                     |

### Feedback / Ratings

Tracks post-ride ratings and feedback.

| Field              | Type          | Required | Description                                     |
|--------------------|---------------|----------|-------------------------------------------------|
| id                 | UUID          | Yes      | Unique identifier                               |
| ride_id            | UUID (FK)     | Yes      | Reference to Rides.id                           |
| from_user_id       | UUID (FK)     | Yes      | Reference to Users.id (person giving the rating)|
| to_user_id         | UUID (FK)     | Yes      | Reference to Users.id (person being rated)      |
| rating             | Integer       | Yes      | Rating value (1-5)                              |
| comment            | String(1000)  | No       | Optional text feedback                          |
| created_at         | Timestamp     | Yes      | Feedback submission timestamp                   |
