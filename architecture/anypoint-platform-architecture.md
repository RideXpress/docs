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
- [Security Architecture](#security-architecture)
    - [User Identity Management](#user-identity-management)
    - [Anypoint User Access Management - Authentication](#anypoint-user-access-management-authentication)
    - [Anypoint User Access Management – Authorization](#anypoint-user-access-management-authorization)
    - [Anypoint Teams – Structure and Considerations](#anypoint-teams-structure-and-considerations)
    - [Platform User Access Management](#platform-user-access-management)
    - [User Audit](#user-audit)
    - [API and Application Security](#api-and-application-security)
    - [API Client Authentication and Authorization](#api-client-authentication-and-authorization)
    - [Secure Connectivity](#secure-connectivity)
    - [API Policy Management](#api-policy-management)
    - [Secure Application Configuration Management](#secure-application-configuration-management)
    - [Infrastructure Security](#infrastructure-security)
    - [Firewall Rules](#firewall-rules)
- [Application Architecture](#application-architecture)
    - [API Led Connectivity](#api-led-connectivity)
    - [Integration Patterns](#integration-patterns)
- [Operations Architecture](#operations-architecture)
    - [Log Management](#log-management)
    - [Monitoring and Alerting](#monitoring-and-alerting)
    - [Platform Alerts](#platform-alerts)
- [Common Services](#common-services)
    - [API Implementation Template](#api-implementation-template)
    - [API Design Template](#api-design-template)
    - [Error Handling](#error-handling)
    - [Configuration Management](#configuration-management)
    - [Secure Properties](#secure-properties)
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
- [Appendix](#appendix)
    - [Backend Systems Catalog](#backend-systems-catalog)
    - [CloudHub Log Data Externalization Options](#cloudhub-log-data-externalization-options)
    - [Anypoint Team Profiles and Permissions Model](#anypoint-team-profiles-and-permissions-model)
- [About MuleSoft, a Salesforce Company](#about-mulesoft-a-salesforce-company)