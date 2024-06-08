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
- [Business and Solution Context](anypoint-platform-architecture/1-business-and-solution-context.md#business-and-solution-context)
    - [Goals and Objectives](anypoint-platform-architecture/1-business-and-solution-context.md#goals-and-objectives)
    - [Solution Context](anypoint-platform-architecture/1-business-and-solution-context.md#solution-context)
        - [Key Systems](anypoint-platform-architecture/1-business-and-solution-context.md#key-systems)
    - [Current State Challenges](anypoint-platform-architecture/1-business-and-solution-context.md#current-state-challenges)
    - [Integration Architecture Principles](anypoint-platform-architecture/1-business-and-solution-context.md#1integration-architecture-principles)
- [Anypoint Platform](anypoint-platform-architecture/2-anypoint-platform.md#anypoint-platform)
    - [Anypoint Platform Licensed Capabilities](anypoint-platform-architecture/2-anypoint-platform.md#anypoint-platform-licensed-capabilities)
    - [Platform Architecture and Setup](anypoint-platform-architecture/2-anypoint-platform.md#platform-architecture-and-setup)
    - [Environments Model](anypoint-platform-architecture/2-anypoint-platform.md#environments-model)
    - [Infrastructure Setup (CloudHub)](anypoint-platform-architecture/2-anypoint-platform.md#infrastructure-setup-cloudhub)
    - [Load Balancing](anypoint-platform-architecture/2-anypoint-platform.md#load-balancing)
    - [SSL Endpoint and DNS Configuration](anypoint-platform-architecture/2-anypoint-platform.md#ssl-endpoint-and-dns-configuration)
    - [TCP Load Balancer](anypoint-platform-architecture/2-anypoint-platform.md#tcp-load-balancer)
    - [Workers](anypoint-platform-architecture/2-anypoint-platform.md#workers)
    - [Controllers](anypoint-platform-architecture/2-anypoint-platform.md#controllers)
    - [High Availability (HA)](anypoint-platform-architecture/2-anypoint-platform.md#high-availability-ha)
    - [Disaster Recovery](anypoint-platform-architecture/2-anypoint-platform.md#disaster-recovery)
    - [CloudHub Capability – Key Considerations](anypoint-platform-architecture/2-anypoint-platform.md#cloudhub-capability-key-considerations)
    - [Disaster Recovery – Options Considered](anypoint-platform-architecture/2-anypoint-platform.md#disaster-recovery-options-considered)
- [Security Architecture](anypoint-platform-architecture/3-security-architecture.md#security-architecture)
    - [User Identity Management](anypoint-platform-architecture/3-security-architecture.md#user-identity-management)
    - [Anypoint User Access Management - Authentication](anypoint-platform-architecture/3-security-architecture.md#anypoint-user-access-management-authentication)
    - [Anypoint User Access Management – Authorization](anypoint-platform-architecture/3-security-architecture.md#anypoint-user-access-management-authorization)
    - [Anypoint Teams – Structure and Considerations](anypoint-platform-architecture/3-security-architecture.md#anypoint-teams-structure-and-considerations)
    - [Platform User Access Management](anypoint-platform-architecture/3-security-architecture.md#platform-user-access-management)
    - [User Audit](anypoint-platform-architecture/3-security-architecture.md#user-audit)
    - [API and Application Security](anypoint-platform-architecture/3-security-architecture.md#api-and-application-security)
    - [API Client Authentication and Authorization](anypoint-platform-architecture/3-security-architecture.md#api-client-authentication-and-authorization)
    - [Secure Connectivity](anypoint-platform-architecture/3-security-architecture.md#secure-connectivity)
    - [API Policy Management](anypoint-platform-architecture/3-security-architecture.md#api-policy-management)
    - [Secure Application Configuration Management](anypoint-platform-architecture/3-security-architecture.md#secure-application-configuration-management)
    - [Infrastructure Security](anypoint-platform-architecture/3-security-architecture.md#infrastructure-security)
    - [Firewall Rules](anypoint-platform-architecture/3-security-architecture.md#firewall-rules)
- [Application Architecture](anypoint-platform-architecture/4-application-architecture.md#application-architecture)
    - [API Led Connectivity](anypoint-platform-architecture/4-application-architecture.md#api-led-connectivity)
    - [Integration Patterns](anypoint-platform-architecture/4-application-architecture.md#integration-patterns)
- [Operations Architecture](anypoint-platform-architecture/5-operations-architecture.md#operations-architecture)
    - [Log Management](anypoint-platform-architecture/5-operations-architecture.md#log-management)
    - [Monitoring and Alerting](anypoint-platform-architecture/5-operations-architecture.md#monitoring-and-alerting)
    - [Platform Alerts](anypoint-platform-architecture/5-operations-architecture.md#platform-alerts)
- [Common Services](anypoint-platform-architecture/6-common-services.md#common-services)
    - [API Implementation Template](anypoint-platform-architecture/6-common-services.md#api-implementation-template)
    - [API Design Template](anypoint-platform-architecture/6-common-services.md#api-design-template)
    - [Error Handling](anypoint-platform-architecture/6-common-services.md#error-handling)
    - [Configuration Management](anypoint-platform-architecture/6-common-services.md#configuration-management)
    - [Secure Properties](anypoint-platform-architecture/6-common-services.md#secure-properties)
- [Software Development Lifecycle (SDLC)](anypoint-platform-architecture/7-software-development-lifecycle.md)
    - [Mule Application Development Considerations](anypoint-platform-architecture/7-software-development-lifecycle.md#mule-application-development-considerations)
    - [Version Control Strategy](anypoint-platform-architecture/7-software-development-lifecycle.md#version-control-strategy)
    - [Deployment and Sizing Strategy](anypoint-platform-architecture/7-software-development-lifecycle.md#deployment-and-sizing-strategy)
    - [Development Standards and Naming Conventions](anypoint-platform-architecture/7-software-development-lifecycle.md#development-standards-and-naming-conventions)
        - [Maven Artifact Id](anypoint-platform-architecture/7-software-development-lifecycle.md#maven-artifact-id)
        - [Cloudhub domain](anypoint-platform-architecture/7-software-development-lifecycle.md#cloudhub-domain)
        - [API version](anypoint-platform-architecture/7-software-development-lifecycle.md#api-version)
        - [API Resources organization](anypoint-platform-architecture/7-software-development-lifecycle.md#api-resources-organization)
        - [API Methods](anypoint-platform-architecture/7-software-development-lifecycle.md#api-methods)
        - [API body format](anypoint-platform-architecture/7-software-development-lifecycle.md#api-body-format)
        - [API pagination and filtering](anypoint-platform-architecture/7-software-development-lifecycle.md#api-pagination-and-filtering)
        - [Files names](anypoint-platform-architecture/7-software-development-lifecycle.md#files-names)
        - [Flow names](anypoint-platform-architecture/7-software-development-lifecycle.md#flow-names)
        - [Dataweave files](anypoint-platform-architecture/7-software-development-lifecycle.md#dataweave-files)
        - [Connector configuration names](anypoint-platform-architecture/7-software-development-lifecycle.md#connector-configuration-names)
        - [Log files](anypoint-platform-architecture/7-software-development-lifecycle.md#log-files)
        - [API, Maven and Cloudhub matching](anypoint-platform-architecture/7-software-development-lifecycle.md#api-maven-and-cloudhub-matching) 
    - [Build and Deployment Automation (CICD)](anypoint-platform-architecture/7-software-development-lifecycle.md#build-and-deployment-automation-cicd)
    - [SDLC Tooling Catalog](anypoint-platform-architecture/7-software-development-lifecycle.md#sdlc-tooling-catalog)
- [Appendix](anypoint-platform-architecture/appendix.md#appendix)
    - [Backend Systems Catalog](anypoint-platform-architecture/appendix.md#backend-systems-catalog)
    - [CloudHub Log Data Externalization Options](#cloudhub-log-data-externalization-options)
    - [Anypoint Team Profiles and Permissions Model](#anypoint-team-profiles-and-permissions-model)
- [About MuleSoft, a Salesforce Company](anypoint-platform-architecture/about.md#about-mulesoft-a-salesforce-company)