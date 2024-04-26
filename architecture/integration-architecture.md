# Integration Architecture Specification

Reference: [Knowledge Hub](https://knowledgehub.mulesoft.com/s/article/Integration-Architecture-Specification)

## Table of Contents
- [Introduction](#introduction)
    - [Key Business Drivers](#key-business-drivers)
    - [References](#references)
    - [Acronyms](#acronyms)
- [Solution Architecture](integration-architecture/1-solution-architecture.md#solution-architecture)
    - [Integration Platform Capabilities](integration-architecture/1-solution-architecture.md#integration-platform-capabilities)
        - [Patterns](integration-architecture/1-solution-architecture.md#patterns)
        - [Connectivity](integration-architecture/1-solution-architecture.md#connectivity)
        - [Technical Capabilities](integration-architecture/1-solution-architecture.md#technical-capabilities)
- [Integration Catalog](integration-architecture/2-integration-catalog.md#integration-catalog)
- [Mulesoft Integration Implementation](integration-architecture/3-mulesoft-integration-implementation.md#mulesoft-integration-implementation)
    - [Mulesoft Anypoint Platform](integration-architecture/3-mulesoft-integration-implementation.md#mulesoft-anypoint-platform)
    - [What is an API?](integration-architecture/3-mulesoft-integration-implementation.md#what-is-an-api)
    - [Mulesoft Connectors](integration-architecture/3-mulesoft-integration-implementation.md#mulesoft-connectors)
    - [API Application Catalogue](integration-architecture/3-mulesoft-integration-implementation.md#api-application-catalogue)
    - [Reusable APIs](integration-architecture/3-mulesoft-integration-implementation.md#reusable-apis)
- [Mulesoft Platform Reference Architecture](integration-architecture/4-mulesoft-platform-reference-architecture.md#mulesoft-platform-reference-architecture)
    - [Platform Capabilities](integration-architecture/4-mulesoft-platform-reference-architecture.md#platform-capabilities)
        - [Runtime Manager](integration-architecture/4-mulesoft-platform-reference-architecture.md#runtime-manager)
        - [API Manager](integration-architecture/4-mulesoft-platform-reference-architecture.md#api-manager)
        - [Design Center](integration-architecture/4-mulesoft-platform-reference-architecture.md#design-center)
        - [Exchange](integration-architecture/4-mulesoft-platform-reference-architecture.md#exchange)
        - [Connectors](integration-architecture/4-mulesoft-platform-reference-architecture.md#connectors)
        - [Platform Services](integration-architecture/4-mulesoft-platform-reference-architecture.md#platform-services)
            - [Anypoint Security](integration-architecture/4-mulesoft-platform-reference-architecture.md#anypoint-security)
            - [Virtual Private Cloud(VPC)](integration-architecture/4-mulesoft-platform-reference-architecture.md#virtual-private-cloud-vpc)
            - [Object Store](integration-architecture/4-mulesoft-platform-reference-architecture.md#object-store)
            - [Dedicated Load Balancers (DLB)](integration-architecture/4-mulesoft-platform-reference-architecture.md#dedicated-load-balancers-dlb)
        - [Extensibility](integration-architecture/4-mulesoft-platform-reference-architecture.md#extensibility)
    - [Platform Setup](integration-architecture/4-mulesoft-platform-reference-architecture.md#platform-setup)
        - [Deployment Regions](integration-architecture/4-mulesoft-platform-reference-architecture.md#deployment-regions)
        - [Business Groups](integration-architecture/4-mulesoft-platform-reference-architecture.md#business-groups)
            - [Organization](integration-architecture/4-mulesoft-platform-reference-architecture.md#organization)
            - [Organization Owner](integration-architecture/4-mulesoft-platform-reference-architecture.md#organization-owner)
            - [Business Groups Explained](integration-architecture/4-mulesoft-platform-reference-architecture.md#business-groups-explained)
            - [Business Group Structure](integration-architecture/4-mulesoft-platform-reference-architecture.md#business-group-structure)
        - [Environments](integration-architecture/4-mulesoft-platform-reference-architecture.md#environments)
    - [Platform Security](integration-architecture/4-mulesoft-platform-reference-architecture.md#platform-security)
        - [Access Management](integration-architecture/4-mulesoft-platform-reference-architecture.md#access-management)
        - [Identity Management](integration-architecture/4-mulesoft-platform-reference-architecture.md#identity-management)
        - [Platform Roles and Permissions](integration-architecture/4-mulesoft-platform-reference-architecture.md#platform-roles-and-permissions)
            - [Platform Role Types](integration-architecture/4-mulesoft-platform-reference-architecture.md#platform-role-types)
            - [Default Platform Roles & Permissions](integration-architecture/4-mulesoft-platform-reference-architecture.md#default-platform-roles-permissions)
    - [Proposed Deployment Architecture](integration-architecture/4-mulesoft-platform-reference-architecture.md#proposed-deployment-architecture)
        - [Environments](integration-architecture/4-mulesoft-platform-reference-architecture.md#environments-2)
        - [Dedicated Load Balancers](integration-architecture/4-mulesoft-platform-reference-architecture.md#dedicated-load-balancers)
        - [Virtual Private Clouds](integration-architecture/4-mulesoft-platform-reference-architecture.md#virtual-private-clouds)
        - [Virtual Private Networks](integration-architecture/4-mulesoft-platform-reference-architecture.md#virtual-private-networks)
    - [About MuleSoft, a Salesforce company](integration-architecture/4-mulesoft-platform-reference-architecture.md#about-mulesoft-a-salesforce-company)


## Introduction

This document describes the proposed integration architecture to support the RideXPress project. The integration solution is split into the following aspects:

- The proposed to-be logical integration processes and associated mechanisms for getting data into and out of the different systems that provide support to the RideXpress solution in a way that is agnostic of the integration layer; it is assumed that the integration platform supports all the required integration capabilities.
- The mechanism for functionally implementing these integration processes using MuleSoft as the integration platform.
- A proposed CloudHub-based platform reference architecture for deploying MuleSoft as the integration platform.

Note that any changes to ways of working as well as other organizational and operating model changes that are required to optimally deliver and maintain an integration platform whilst maximizing business agility are outside the scope of this document.


### Key Business Drivers
The key business drivers for this initiative that are relevant for the integration platform are as follows:

- Create an API driven company

### References

Please note that any version numbers are current as of the time of writing this document. Update this table of references to align with your integration architecture specifics.

- **Salesforce Platform REST API**: [Salesforce REST API Documentation](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/intro_what_is_rest_api.htm)
- **Authentication**: [Okta OAuth]()
- **API-led Connectivity**: [MuleSoft API-led Connectivity Whitepaper](https://www.mulesoft.com/lp/whitepaper/api/api-led-connectivity)
- **MuleSoft CloudHub**: [MuleSoft CloudHub Information](https://www.mulesoft.com/platform/saas/cloudhub-ipaas-cloud-based-integration)
- **Mulesoft Anypoint Platform**: [Mulesoft Enterprise Integration Platform](https://www.mulesoft.com/platform/enterprise-integration)
- **Mule Runtime**: [Mule Runtime Documentation](https://docs.mulesoft.com/mule-runtime/4.3/)
- **MuleSoft Salesforce Connector**: [MuleSoft Salesforce Connector Documentation](https://docs.mulesoft.com/salesforce-connector/10.9/)
- **Maps API**: [Google Maps]()
- **Payments API**: [Square]()
- **Data base**: 
- **Push Notifications API**: 
- **Email API**: 

### Acronyms

- **DB**: Data Base
- **MULE**: