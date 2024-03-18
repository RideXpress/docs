# Anypoint Platform Architecture

Reference: [Knowledge Hub](https://knowledgehub.mulesoft.com/s/article/Anypoint-Platform-Architecture-Template)

# About this Document

This document presents the reference architecture for Anypoint Platform. The platform components will become the foundational building blocks for current and future API solutions, where MuleSoft will be the primary platform for connectivity.

The document is not intended to provide the solution design and architecture for a particular project or API initiative, this document describes a strategic architecture blueprint of the underlying MuleSoft platform and is intended to provide the foundation for delivering future API solutions.

# Intended Audience

The intended audience for this document comprises the API Leads / Product Owners, Delivery Leads, Enterprise Architects, and Solutions Architects. 

In addition to these roles, it is essential that the Business Teams (Business Analysts, Consultants, and Line of Business IT) are aware of this document and its content, as it describes many activities and processes involved in the delivery of APIs.

# Table of Contents

## Table of Contents

- [About this Document](#about-this-document)
- [Intended Audience](#intended-audience)
- [1 Business and Solution Context](anypoint-platform-architecture/1-business-and-solution-context.md#1-business-and-solution-context)
    - [1.1 Goals and Objectives](anypoint-platform-architecture/1-business-and-solution-context.md#11-goals-and-objectives)
    - [1.2 Solution Context](anypoint-platform-architecture/1-business-and-solution-context.md#12-solution-context)
        - [1.2.1 Key Systems](anypoint-platform-architecture/1-business-and-solution-context.md#121-key-systems)
    - [1.3 Current State Challenges](anypoint-platform-architecture/1-business-and-solution-context.md#13-current-state-challenges)
    - [1.4 Integration Architecture Principles](anypoint-platform-architecture/1-business-and-solution-context.md#14-integration-architecture-principles)
- [2 Anypoint Platform](anypoint-platform-architecture/2-anypoint-platform.md#2-anypoint-platform)
    - [2.1 Anypoint Platform Licensed Capabilities](anypoint-platform-architecture/2-anypoint-platform.md#21-anypoint-platform-licensed-capabilities)
    - [2.2 Platform Architecture and Setup](anypoint-platform-architecture/2-anypoint-platform.md#22-platform-architecture-and-setup)
    - [2.3 Environments Model](anypoint-platform-architecture/2-anypoint-platform.md#23-environments-model)
    - [2.4 Infrastructure Setup (CloudHub)](anypoint-platform-architecture/2-anypoint-platform.md#24-infrastructure-setup-cloudhub)
    - [2.5 Load Balancing](anypoint-platform-architecture/2-anypoint-platform.md#25-load-balancing)
    - [2.6 SSL Endpoint and DNS Configuration](anypoint-platform-architecture/2-anypoint-platform.md#26-ssl-endpoint-and-dns-configuration)
    - [2.7 TCP Load Balancer](anypoint-platform-architecture/2-anypoint-platform.md#27-tcp-load-balancer)
    - [2.8 Workers](anypoint-platform-architecture/2-anypoint-platform.md#28-workers)
    - [2.9 Controllers](anypoint-platform-architecture/2-anypoint-platform.md#29-controllers)
    - [2.10 High Availability (HA)](anypoint-platform-architecture/2-anypoint-platform.md#210-high-availability-ha)
    - [2.11 Disaster Recovery](anypoint-platform-architecture/2-anypoint-platform.md#211-disaster-recovery)
    - [2.12 CloudHub Capability – Key Considerations](anypoint-platform-architecture/2-anypoint-platform.md#212-cloudhub-capability-key-considerations)
    - [2.13 Disaster Recovery – Options Considered](anypoint-platform-architecture/2-anypoint-platform.md#213-disaster-recovery-options-considered)
- [3. Security Architecture](#security-architecture)
    - [3.1. User Identity Management](#user-identity-management)
    - [3.2. Anypoint User Access Management - Authentication](#anypoint-user-access-management-authentication)
    - [3.3. Anypoint User Access Management – Authorization](#anypoint-user-access-management-authorization)
    - [3.4. Anypoint Teams – Structure and Considerations](#anypoint-teams-structure-and-considerations)
    - [3.5. Platform User Access Management](#platform-user-access-management)
    - [3.6. User Audit](#user-audit)
    - [3.7. API and Application Security](#api-and-application-security)
    - [3.8. API Client Authentication and Authorization](#api-client-authentication-and-authorization)
    - [3.9. Secure Connectivity](#secure-connectivity)
    - [3.10. API Policy Management](#api-policy-management)
    - [3.11. Secure Application Configuration Management](#secure-application-configuration-management)
    - [3.12. Infrastructure Security](#infrastructure-security)
    - [3.13. Firewall Rules](#firewall-rules)
- [4. Application Architecture](#application-architecture)
    - [4.1. API Led Connectivity](#api-led-connectivity)
    - [4.2. Integration Patterns](#integration-patterns)
- [5. Operations Architecture](#operations-architecture)
    - [5.1. Log Management](#log-management)
    - [5.2. Monitoring and Alerting](#monitoring-and-alerting)
    - [5.3. Platform Alerts](#platform-alerts)
- [6. Common Services](#common-services)
    - [6.1. API Implementation Template](#api-implementation-template)
    - [6.2. API Design Template](#api-design-template)
    - [6.3. Error Handling](#error-handling)
    - [6.4. Configuration Management](#configuration-management)
    - [6.5. Secure Properties](#secure-properties)
- [7. Software Development Lifecycle (SDLC)](anypoint-platform-architecture/7-software-development-lifecycle.md)
    - [7.1. Mule Application Development Considerations](anypoint-platform-architecture/7-software-development-lifecycle.md#71-mule-application-development-considerations)
    - [7.2. Version Control Strategy](anypoint-platform-architecture/7-software-development-lifecycle.md#72-version-control-strategy)
    - [7.3. Deployment and Sizing Strategy](anypoint-platform-architecture/7-software-development-lifecycle.md#73-deployment-and-sizing-strategy)
    - [7.4. Development Standards and Naming Conventions](anypoint-platform-architecture/7-software-development-lifecycle.md#74-development-standards-and-naming-conventions)
    - [7.5. Build and Deployment Automation (CICD)](anypoint-platform-architecture/7-software-development-lifecycle.md#75-build-and-deployment-automation-cicd)
    - [7.6. SDLC Tooling Catalog](anypoint-platform-architecture/7-software-development-lifecycle.md#76-sdlc-tooling-catalog)
- [Appendix](#appendix)
    - [Backend Systems Catalog](#backend-systems-catalog)
    - [CloudHub Log Data Externalization Options](#cloudhub-log-data-externalization-options)
    - [Anypoint Team Profiles and Permissions Model](#anypoint-team-profiles-and-permissions-model)
- [About MuleSoft, a Salesforce Company](#about-mulesoft-a-salesforce-company)