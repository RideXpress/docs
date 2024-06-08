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
- [Business and Solution Context](anypoint-platform-architecture/1-business-and-solution-context.md#1-business-and-solution-context)
    - [Goals and Objectives](anypoint-platform-architecture/1-business-and-solution-context.md#goals-and-objectives)
    - [Solution Context](anypoint-platform-architecture/1-business-and-solution-context.md#solution-context)
        - [Key Systems](anypoint-platform-architecture/1-business-and-solution-context.md#key-systems)
    - [Current State Challenges](anypoint-platform-architecture/1-business-and-solution-context.md#current-state-challenges)
    - [Integration Architecture Principles](anypoint-platform-architecture/1-business-and-solution-context.md#1integration-architecture-principles)
- [Anypoint Platform](anypoint-platform-architecture/2-anypoint-platform.md#2-anypoint-platform)
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
- [Security Architecture](#3-security-architecture)
    - [User Identity Management](#31-user-identity-management)
    - [Anypoint User Access Management - Authentication](#32-anypoint-user-access-management-authentication)
    - [Anypoint User Access Management – Authorization](#33-anypoint-user-access-management-authorization)
    - [Anypoint Teams – Structure and Considerations](#34-anypoint-teams-structure-and-considerations)
    - [Platform User Access Management](#35-platform-user-access-management)
    - [User Audit](#36-user-audit)
    - [API and Application Security](#37-api-and-application-security)
    - [API Client Authentication and Authorization](#38-api-client-authentication-and-authorization)
    - [Secure Connectivity](#39-secure-connectivity)
    - [API Policy Management](#310-api-policy-management)
    - [Secure Application Configuration Management](#311-secure-application-configuration-management)
    - [Infrastructure Security](#312-infrastructure-security)
    - [Firewall Rules](#313-firewall-rules)
- [Application Architecture](#4-application-architecture)
    - [API Led Connectivity](#41-api-led-connectivity)
    - [Integration Patterns](#42-integration-patterns)
- [Operations Architecture](#5-operations-architecture)
    - [Log Management](#51-log-management)
    - [Monitoring and Alerting](#52-monitoring-and-alerting)
    - [Platform Alerts](#53-platform-alerts)
- [Common Services](#6-common-services)
    - [API Implementation Template](#61-api-implementation-template)
    - [API Design Template](#62-api-design-template)
    - [Error Handling](#63-error-handling)
    - [Configuration Management](#64-configuration-management)
    - [Secure Properties](#65-secure-properties)
- [Software Development Lifecycle (SDLC)](anypoint-platform-architecture/7-software-development-lifecycle.md)
    - [Mule Application Development Considerations](anypoint-platform-architecture/7-software-development-lifecycle.md#71-mule-application-development-considerations)
    - [Version Control Strategy](anypoint-platform-architecture/7-software-development-lifecycle.md#72-version-control-strategy)
    - [Deployment and Sizing Strategy](anypoint-platform-architecture/7-software-development-lifecycle.md#73-deployment-and-sizing-strategy)
    - [Development Standards and Naming Conventions](anypoint-platform-architecture/7-software-development-lifecycle.md#74-development-standards-and-naming-conventions)
        - [Maven Artifact Id](anypoint-platform-architecture/7-software-development-lifecycle.md#741-maven-artifact-id)
        - [Cloudhub domain](anypoint-platform-architecture/7-software-development-lifecycle.md#742-cloudhub-domain)
        - [API version](anypoint-platform-architecture/7-software-development-lifecycle.md#743-api-version)
        - [API Resources organization](anypoint-platform-architecture/7-software-development-lifecycle.md#744-api-resources-organization)
        - [API Methods](anypoint-platform-architecture/7-software-development-lifecycle.md#745-api-methods)
        - [API body format](anypoint-platform-architecture/7-software-development-lifecycle.md#746-api-body-format)
        - [API pagination and filtering](anypoint-platform-architecture/7-software-development-lifecycle.md#747-api-pagination-and-filtering)
        - [Files names](anypoint-platform-architecture/7-software-development-lifecycle.md#748-files-names)
        - [Flow names](anypoint-platform-architecture/7-software-development-lifecycle.md#749-flow-names)
        - [Dataweave files](anypoint-platform-architecture/7-software-development-lifecycle.md#7410-dataweave-files)
        - [Connector configuration names](anypoint-platform-architecture/7-software-development-lifecycle.md#7411-connector-configuration-names)
        - [Log files](anypoint-platform-architecture/7-software-development-lifecycle.md#7412-log-files)
        - [API, Maven and Cloudhub matching](anypoint-platform-architecture/7-software-development-lifecycle.md#7413-api-maven-and-cloudhub-matching) 
    - [Build and Deployment Automation (CICD)](anypoint-platform-architecture/7-software-development-lifecycle.md#75-build-and-deployment-automation-cicd)
    - [SDLC Tooling Catalog](anypoint-platform-architecture/7-software-development-lifecycle.md#76-sdlc-tooling-catalog)
- [Appendix](#appendix)
    - [Backend Systems Catalog](#backend-systems-catalog)
    - [CloudHub Log Data Externalization Options](#cloudhub-log-data-externalization-options)
    - [Anypoint Team Profiles and Permissions Model](#anypoint-team-profiles-and-permissions-model)
- [About MuleSoft, a Salesforce Company](#about-mulesoft-a-salesforce-company)