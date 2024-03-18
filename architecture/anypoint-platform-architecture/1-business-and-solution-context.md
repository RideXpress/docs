# Business and Solution Context

- [1 Business and Solution Context](#1-business-and-solution-context)
    - [1.1 Goals and Objectives](#11-goals-and-objectives)
    - [1.2 Solution Context](#12-solution-context)
        - [1.2.1 Key Systems](#121-key-systems)
    - [1.3 Current State Challenges](#13-current-state-challenges)
    - [1.4 Integration Architecture Principles](#14-integration-architecture-principles)

## 1 Business and Solution Context

RideXpress is a fictional ride sharing company founded with the solely purpose of demonstrating the power of the API-First approach. It was created with the idea of powering every single API with Anypoint Platform, from the main API to the integration to third party APIs or Applications, every single piece of code runs on a flavor of the Mule Runtime.

RideXpress is completely Open Source and Community driven so we can learn from each other based on our diverse backgrounds and experiences.

### 1.1 Goals and Objectives

- User-Centric Subscription Model: Predictable pricing for drivers with monthly subscription plans.
- Stability for Drivers: Consistent earnings through a steady flow of riders.
- Seamless Experience: Effortless rides without surge pricing worries.
- Tech-Driven Community: Optimized matching and efficient service.
- Disruptive Innovation: A proposal that challenges traditional pay-per-ride models like Uber or Lyft.

### 1.2 Solution Context

<TODO: Describe the current state solution landscape within the customer, the target state architecture, and how MuleSoft will complement, or replace existing integration capabilities as the customer transitions from as-is to to-be state>

#### 1.2.1 Key Systems 

| System | Description | Deployment Model | Key integration concerns |
| --- | --- | --- | --- |
| Salesforce | Customer Management platform to support sales and service activities. | Cloud | MuleSoft connectors available |
| Google Maps | Maps and routes system to support geo location and building the best routes | Cloud | MuleSoft connectors available |
| Okta | Authentication platform to manage users credentials and platform authentication | Cloud | Mulesoft connectors available |
| Square | Payment platform to manage payments of rides | Cloud | REST API Available |
| Gmail | Email plaform to send notifications to users | Cloud | Mulesoft connectors available |
| Apple Push Notification | Push notifications plaform to send notifications to users | Cloud | REST API Available |
| DB System | Database engine to store support information related to rides, users and drivers | Cloud | REST API Available |

### 1.3 Current State Challenges

There are several challenges within the existing IT landscape that create friction and hinder the ability to realize their broader business objectives, including:

1.  Find low/non cost alternatives for the MVP
1.  Understand the regulations of payments
1.  Understand the regulations of personal information 
1.  Being an Open Source company can affect the main interests

### 1.4 Integration Architecture Principles

- **APIs as a product**: Treat every API as a product - effectively every API should be an independent product, with its own lifecycle and ownership. This is one of the core principles which underpins the application network, and effectively drives the vision around the ‘composable’ enterprise, and the goal to increase time to market for new products and propositions.
- **Secure**: Given the critical nature of services that need to be supported, it is essential that security is built from the ground up. This must all elements of the integration, for example 
    - API Security 
    - Data security (in transit and at rest)
    - Operations Security (e.g. proactive monitoring and alerts, audit)
- **Scalable**: Integration services are highly utilized and need to scale to meet growing demand, as well as changing customer behaviors. Solutions built must be scalable, and application and infrastructure design must consider this as a core capability. 
- **Resilient**: Services should be built with resilience and reliability in mind. Where appropriate, asynchronous processing patterns should be considered, with appropriate error management and alerting to support resilient operations. 
- **Measurable**: Services built should have metrics configured to analyze and improve on service delivery. This will help to more proactively identify and manage issues, and capacity going forward. 
- **Discoverable**: All assets should be published to a central repository so that teams can discover and reuse assets to drive new business initiatives and outcomes. 
- **Canonical Data Model**: There must be a canonical data model for each business domain (this does NOT mean having a canonical data model for the whole organization, which would NOT be recommended). Canonical data model must be maintained as a separate reusable RAML fragment documented on Exchange.
- **Loosely coupled, highly cohesive**: APIs and applications should have a single responsibility when it comes to supporting wider business outcomes. This aligns with the API led architecture approach and enables the creation of assets that can be reused and ‘composed’ to drive future capabilities.

### 1.5 Assumptions and Dependencies

<Document key assumptions and dependencies that have been considered as part of this platform architecture definition. > 

- <Assumption / Dependency>