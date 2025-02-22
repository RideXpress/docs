# Square API Design Document
**RideXpress**  
**Version:** 1.0  
**Date:** 2024-05-11  

## Purpose
This document describes the technical design for the Square API.

## Intended Audience
This document is primarily intended for:
- Architects
- Consultants
- Developers and testers engaged in architecting, designing, developing, and testing API-based solutions

## Taxonomy
Refer to the glossary or taxonomy document for general terms and definitions.

---

## API Overview

| **Field**              | **Details**                              |
|-------------------------|------------------------------------------|
| **API Name (Technical)**| `square-system-api`                     |
| **API Name (Business)** | Square                                  |
| **API Type**           | System                                  |
| **Business Domain**    | Domain                                  |
| **Purpose of API**     | Manage all communication with Square for Payments and Orders |
| **API Owner**          | Juan Castelan                           |
| **Version(s)**         | 1.0.0                                   |

---

## API Architecture
The Square API provides operations to handle payments across RideXpress. It operates at a system level and abstracts the extense API to the ednpoints needed for RideXpress.  
This API can be found in the overall architecture diagram.

<a href="../../../architecture/diagrams/entity-diagrams/solution-diagram.drawio.svg">
  <img src="../../../architecture/diagrams/entity-diagrams/solution-diagram.drawio.svg" alt="alt text" width="800"/>
</a>

RideXpress leverages the following Square APIs:
- **Quick Payments API**: For payment link generation
- **Orders API**: For order management  

**Key Design Notes:**
- RideXpress will not store sensitive user financial information (e.g., credit cards) or private addresses.
- Payment confirmation and processing are delegated to Square.

### Sequence Diagram
<a href="../../../architecture/diagrams/api-led/square-sequence.drawio.svg">
  <img src="../../../architecture/diagrams/api-led/square-sequence.drawio.svg" alt="alt text" width="800"/>
</a>
1. User requests a ride; RideXpress orders a payment link from Square.
2. User enters payment information; Square validates it.
3. RideXpress requests the order using the order ID returned by Square.
4. After the ride, RideXpress uses the Update Order endpoint to confirm or cancel payment.

---

## API Resources
The following resources handle payments:

| **Resource**     | **HTTP Methods** | **Description**                              |
|-------------------|------------------|----------------------------------------------|
| `/payment-link`  | POST             | Create a payment link when a ride is requested |
| `/orders/{id}`   | GET, PUT         | Get order details or update it when a ride is completed/canceled |

---

## API Nonfunctional Requirements

### Hours of Operation
- **Availability:** 24x7 (required for every ride to process payments)

### Availability
- **Target:** 99.99% uptime
- **Reason:** Handles critical financial information; the app cannot function without this API.

### Reliability
- **Calculation:**
  - Total operating time: 8 hours/day × 5 days/week × 52 weeks = **2,080 hours**
  - Number of failures: **4**
  - Mean-Time-Between-Failures (MTBF): 2,080 hours / 4 = **520 hours**
- **Note:** May vary based on other system requirements or capabilities. A "break" occurs when the API fails with specific data or underlying systems are down.

### Support
- Issues handled per SLAs in the Architecture document.
- Enhancements or new features are not considered support.

### Throughput
- **Baseline:** 3 requests per ride
- **Scalability:** Increases with the number of rides

### Transaction Response Times
- **Target:** 95% of transactions within **2 seconds**

### Change Management
- Targets for approving, handling, and implementing changes depend on category or urgency/priority.
- Refer to the organization’s Change Management procedures for compliance.
- Defines buffer time for API changes and client adaptation (to be specified).

*Insert Change Management Requirements here.*

### Continuity
- **Recovery Time Objective (RTO):** Specify recovery time in hours post-failure.
- **Recovery Point Objective (RPO):** Define acceptable data loss.

*Insert Continuity Requirements here.*

### Contingency
- Planning for recovery solutions (e.g., physical disasters, reversing production implementations).
- Must comply with existing disaster recovery plans.

*Insert Contingency Requirements here.*

---

## Security
Security requirements align with corporate business policies and must be reviewed/approved by the Information Security team.

| **Security Requirement**         | **Scope**       | **Requirement Decision** |
|----------------------------------|-----------------|--------------------------|
| Entity Authentication            |                 |                          |
| Data Authentication              |                 |                          |
| Authorization                    |                 |                          |
| Data Protection in Motion        | Yes             | Yes                      |
| Data Protection at Rest          | Yes             |                          |
| Message Validation               |                 |                          |
| Content Scanning                 |                 |                          |
| Auditing                         | Yes             |                          |
| Monitoring                       | Yes             |                          |
| Management & Administration      |                 |                          |
| Client Provisioning              |                 |                          |
| Trust Management                 |                 |                          |

### Scope
- Security scope may be point-to-point or end-to-end, as recommended by the security decision maker.

### API Access
- Define roles, individual partners, or entities allowed to execute this API.
- Specify invocation mechanisms and security procedures for consumption.

*Insert additional API consumption security details here.*

---
