# Anypoint Platform Architecture

Note: document based of https://knowledgehub.mulesoft.com/s/article/Integration-Architecture-Specification

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
- [1. Business and Solution Context](#business-and-solution-context)
    - [1.1. Goals and Objectives](#goals-and-objectives)
    - [1.2. Solution Context](#solution-context)
        - [1.2.1. Key Systems](#key-systems)
    - [1.3. Current State Challenges](#current-state-challenges)
    - [1.4. Integration Architecture Principles](#integration-architecture-principles)
- [2. Anypoint Platform](#anypoint-platform)
    - [2.1. Anypoint Platform Licensed Capabilities](#anypoint-platform-licensed-capabilities)
    - [2.2. Platform Architecture and Setup](#platform-architecture-and-setup)
    - [2.3. Business Groups](#business-groups)
    - [2.4. Environments Model](#environments-model)
    - [2.5. Infrastructure Setup (CloudHub)](#infrastructure-setup-cloudhub)
    - [2.6. Virtual Private Cloud (VPC)](#virtual-private-cloud-vpc)
    - [2.7. Secure Connectivity Configurations](#secure-connectivity-configurations)
    - [2.8. Load Balancing](#load-balancing)
    - [2.9. SSL Endpoint and DNS Configuration](#ssl-endpoint-and-dns-configuration)
    - [2.10. Routing Rules](#routing-rules)
    - [2.11. Infrastructure Setup (On-premise Bare Metal)](#infrastructure-setup-on-premise-bare-metal)
    - [2.12. Mule Clusters](#mule-clusters)
    - [2.13. Mule Server Groups](#mule-server-groups)
    - [2.14. Infrastructure Setup (Runtime Fabric)](#infrastructure-setup-runtime-fabric)
    - [2.15. TCP Load Balancer](#tcp-load-balancer)
    - [2.16. Workers](#workers)
    - [2.17. Controllers](#controllers)
    - [2.18. High Availability (HA)](#high-availability-ha)
    - [2.19. Disaster Recovery](#disaster-recovery)
    - [2.20. CloudHub Capability – Key Considerations](#cloudhub-capability-key-considerations)
    - [2.21. Disaster Recovery – Options Considered](#disaster-recovery-options-considered)
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
    - [3.13. DLB Security](#dlb-security)
    - [3.14. Firewall Rules](#firewall-rules)
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
- [7. SDLC](#sdlc)
    - [7.1. Mule Application Development Considerations](#mule-application-development-considerations)
    - [7.2. Version Control Strategy](#version-control-strategy)
    - [7.3. Deployment and Sizing Strategy](#deployment-and-sizing-strategy)
    - [7.4. Development Standards and Naming Conventions](#development-standards-and-naming-conventions)
    - [7.5. Build and Deployment Automation (CICD)](#build-and-deployment-automation-cicd)
    - [7.6. SDLC Tooling Catalog](#sdlc-tooling-catalog)
- [Appendix](#appendix)
    - [Backend Systems Catalog](#backend-systems-catalog)
    - [CloudHub Log Data Externalization Options](#cloudhub-log-data-externalization-options)
    - [CloudHub DLB Detailed Connectivity Flow](#cloudhub-dlb-detailed-connectivity-flow)
    - [Anypoint Team Profiles and Permissions Model](#anypoint-team-profiles-and-permissions-model)
- [About MuleSoft, a Salesforce Company](#about-mulesoft-a-salesforce-company)


## 1. Business and Solution Context

RideXpress is a fictional ride sharing company founded with the solely purpose of demonstrating the power of the API-First approach. It was created with the idea of powering every single API with Anypoint Platform, from the main API to the integration to third party APIs or Applications, every single piece of code runs on a flavor of the Mule Runtime.

RideXpress is completely Open Source and Community driven so we can learn from each other based on our diverse backgrounds and experiences.

### 1.1. Goals and Objectives

- User-Centric Subscription Model: Predictable pricing for drivers with monthly subscription plans.
- Stability for Drivers: Consistent earnings through a steady flow of riders.
- Seamless Experience: Effortless rides without surge pricing worries.
- Tech-Driven Community: Optimized matching and efficient service.
- Disruptive Innovation: A proposal that challenges traditional pay-per-ride models like Uber or Lyft.

### 1.2. Solution Context

<TODO: Describe the current state solution landscape within the customer, the target state architecture, and how MuleSoft will complement, or replace existing integration capabilities as the customer transitions from as-is to to-be state>

#### 1.2.1. Key Systems 

| System | Description | Deployment Model | Key integration concerns |
| --- | --- | --- | --- |
| Salesforce | Customer Management platform to support sales and service activities. | Cloud | MuleSoft connectors available |
| Google Maps | Maps and routes system to support geo location and building the best routes | Cloud | MuleSoft connectors available |
| Okta | Authentication platform to manage users credentials and platform authentication | Cloud | Mulesoft connectors available |
| Square | Payment platform to manage payments of rides | Cloud | REST API Available |
| Gmail | Email plaform to send notifications to users | Cloud | Mulesoft connectors available |
| Apple Push Notification | Push notifications plaform to send notifications to users | Cloud | REST API Available |
| DB System | Database engine to store support information related to rides, users and drivers | Cloud | REST API Available |

### 1.3. Current State Challenges

There are several challenges within the existing IT landscape that create friction and hinder the ability to realize their broader business objectives, including:

1.  Find low/non cost alternatives for the MVP
1.  Understand the regulations of payments
1.  Understand the regulations of personal information 
1.  Being an Open Source company can affect the main interests

### 1.4. Integration Architecture Principles

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

### 1.5. Assumptions and Dependencies

<Document key assumptions and dependencies that have been considered as part of this platform architecture definition. > 

- <Assumption / Dependency>

## 2. Architecture Decisions

<Document the key architecture decisions that have been made as part of the platform architecture definition.Please update the below decisions to reflect the platform architecture decisions agreed with the customer. > 

| Id | Decision | Rationale | Implication |
| --- | --- | --- | --- |
| Dxx | <Description of the decision> | <Describe the rationale of the decision – why was this decision made> | <what are the implications of this decision. What should the stakeholders be aware of because of this decision that they are signing up to> |
| D01 | MuleSoft CloudHub control plane will be hosted in the US. | ● Provide the ability to support multiple runtime plane regions as the platform scales – this is only possible with the US control plane, as the EU control plane only supports Frankfurt, and Ireland regions as of Mar/2022. <br> ● Control plane data is stored in the US. Note that this is limited to API and application meta-data, and CloudHub platform user authentication and authorization data. | |


## 3. Anypoint Platform
![alt text](anypoint-platform-architecture/plaform_diagram.png)

The Anypoint Platform has five major components and will be the core to future integration capabilities:
- **Anypoint Design center**: Comprehensive tools to develop APIs and integrations faster and easier
- **Exchange**: Marketplace of API and integration assets, promotes reuse of pre-built connectors, templates, examples, and APIs 
- **Anypoint Management center**: Manage and monitor your applications across the platform in a single place
- **Mule runtime engine & services**: Single runtime for your Mule applications, easily configurable, performant, and deployable anywhere
- **Anypoint MQ**: Anypoint Cloud based messaging platform, to support asynchronous messaging use cases such as queueing and publish-subscribe with fully hosted and managed cloud-based Message Queues and Exchanges
- **Object Store**: Mechanism for storing arbitrary objects in MuleSoft using key-value pairs
- **Security**: Safeguard sensitive information with layers of protection
- **Anypoint Connectors**: Provide a means to connect to endpoints from your integration applications. Connectors fall into different categories.
    - **Community**: MuleSoft or members of the MuleSoft community write and maintain the Community connectors. Connectors built by the community or MuleSoft are generally open-source, although each package may vary. Partner-built connectors may not be open-source. Contact the partner directly for more information. You do not need any special account or license to use a Community connector
    - **MuleSoft Certified**: MuleSoft Certified connectors are developed by MuleSoft’s partners and developer community and are reviewed and certified by MuleSoft. For support, customers should contact the MuleSoft partner that created the MuleSoft Certified connector.
    - **Select**: MuleSoft maintains Select connectors. Connectors included in the open source Mule distribution can be used by everyone, however support is only included in an Anypoint Platform subscription.
    - **Premium**: MuleSoft maintains Premium connectors; you must have an active CloudHub Premium plan or an Enterprise subscription with an entitlement for the specific connector you wish to use.

### 3.1. Anypoint Platform Licensed Capabilities

Current provision of the platform  [Titanium / Platinum / Gold] MuleSoft subscription, with the following license entitlements as of <Jan 1, 2022> 

- **Production Cores/vCores**: <4> 
- **Sandbox Cores/vCores**: <8>
- **Anypoint VPCs**: <4>
- **Anypoint VPNs**: <4>
- **Anypoint DLBs (Dedicated Load Balancer)**: <4>
- **Static IPs**: <8>
- **API Manager and Analytics**
- **Premium Connectors**
- **SAP** [<1> Production, and <2> non-production environments]

**Reference**: [Product Subscription Plans](https://www.mulesoft.com/prod-subscription-plans)

## 4. Platform Architecture and Setup

<Describe the key details of the platform setup with the customer landscape, including details of platform infrastructure considerations like VPCs, DLBs, VPNs > 

The following section describes how the Anypoint platform foundations are architected to support the long-term requirements of the enterprise.
 
_Update diagram based on agreed platform architecture_

### 4.1. Business Groups

Business Groups provide a mechanism for delegating management and administration of Anypoint Platform to users within different business units or functions. Additionally, Business Groups provide isolation of resources allowing for multi-tenant use cases within the Anypoint Platform. 

Note that the Anypoint Platform imposes a limit of 100 Business Groups per master organization. This is not expected to be an issue today, but worth considering for future expansion and Business Group strategy. 

Additional details on Business Groups are available [here](#).

| Business Group | Description |
| --- | --- |
| [CUSTOMER] | Master Business Group |
| [CUSTOMER] Central IT | Central IT organization for central MuleSoft assets |

### 4.2. Environments Model

Environments are a logical construct within Anypoint. Environments will be created at the business group level. Given the current Route to Live considerations, the minimum set of environments required would be as follows:

| Environment | Description | Classification (vCore) | High Availability? | Deployed By |
| --- | --- | --- | --- | --- |
| DEV | To support development and CI activities | Sandbox | No | DevOps + Developer |
| TEST | Used for functional and end to end testing | Sandbox | No | DevOps (Manual Approvals) |
| PREPROD | Like for like production environment to support acceptance and non-functional testing | Sandbox | Yes | DevOps (Manual Approvals) |
| PROD | Production environment | Production | Yes | DevOps (Manual Approvals) |

### 4.3. Infrastructure Setup (CloudHub)

TODO: Infrastructure diagram

#### 4.3.1. Virtual Private Cloud (VPC)

CloudHub VPCs provide a virtual, private, and isolated network segment where CloudHub workers (and in turn deployed MuleSoft applications) are hosted. 

A key consideration of the VPC setup is the size of the VPC, i.e. the range of IP addresses available for this network, specified in the form of a Classless Inter-Domain Routing (CIDR) block. The VPC size needs to consider the number of Mule APIs and applications that are planned to be deployed to the VPC, including allowances for high availability, zero downtime deployments and scaling considerations. Additional details on VPC sizing are available [here](#). 

As part of the CloudHub Setup, the following VPCs will be configured.

TODO: update the table

| VPC name | Region | CIDR | Environments | Business Group |
| --- | --- | --- | --- | --- |
| vpc01-nonprod-de-c1 | EU - Frankfurt | 10.0.0.0/16 | DEV, TEST, PREPROD | [CUSTOMER] |
| vpc02-prod-de-c1 | EU - Frankfurt | 10.1.0.0/16 | PROD | [CUSTOMER] |

Additional details on VPC sizing are available [here](https://docs.mulesoft.com/runtime-manager/virtual-private-cloud).

##### 4.3.1.1. Virtual Private Cloud (VPC)

VPC firewall rules will be configured to restrict traffic to internally originating traffic only, i.e. traffic originating within the VPC itself, or traffic originating from the corporate data center which is securely connected to the CloudHub VPC via VPN. 

In addition, all applications will be deployed to CloudHub private port (8092) which restricts access to the Dedicated Load Balancer and mule-worker-internal-* endpoints only.

Additional details on VPC firewall rules are available [here](https://docs.mulesoft.com/runtime-manager/vpc-firewall-rules-concept).

TODO: Complete the table

| Source CIDR Range | Destination Port Range | Scope |
| --- | --- | --- |
| Local VPC CIDR | 8091-8092 | Access to traffic originating within the VPC. Either DLB to Mule applications, or Mule to Mule applications. |
| [CUSTOMER] VPN remote subnets | 8091-8092 | Access from [CUSTOMER] Data Center subnets to Internal CloudHub DLB internal endpoint, or alternatively through use of the mule-worker-internal-* endpoints. |

#### 4.3.2. Secure Connectivity configurations 

Connectivity between the Anypoint CloudHub platform and corporate internal / partner networks will be established via IPsec Tunnel (or VPC Peering / Direct Connect if appropriate)  

The following table captures the key details:

| VPN name | VPC association | Routing | ASNs | Remote IP | Static Routes |
| --- | --- | --- | --- | --- | --- |
| nonprod-vpn-vpc01-to-dc1 | vpc01-nonprod-eu-frankfurt | Static | N/A | x.x.x.x | ●	x.x.x.x/24, ●	x.x.x.x/20 |
| prod-vpn-vpc02-to-dc1 | vpc02-prod-eu-frankfurt | Static | N/A | x.x.x.x | ●	x.x.x.x/24, ●	x.x.x.x/20 |

TODO: Complete the table ^

Additional details on VPN configurations are available [here](https://docs.mulesoft.com/runtime-manager/vpn-about).

#### 4.3.3. Load Balancing

Dedicated Load balancers (DLBs) will be used to support load balancing of all MuleSoft hosted applications – both external facing and internal facing APIs. Access to shared load balancer endpoints will be restricted via VPC firewall rules to restrict access exclusively through the DLBs. DLBs also provide High Availability through deployment on two or more workers. This does NOT reduce the number of vCores available on CloudHub. Note that at this stage, the default DLB entitlement of 2 workers will be used, but this can be extended if additional throughput is required (additional DLB entitlements have commercial considerations and will need to be reviewed with the MuleSoft Account Team). 

Two DLBs instances will be configured, 
- External facing DLB to host SSL endpoints to external partners and customers. 
- Internal facing DLB to host internal only SSL endpoints – these will be used for Mule to Mule API connectivity, and internal traffic originating from the corporate Data Center or other trusted network securely connected to CloudHub VPC (e.g. via VPN) 

| DLB Name | VPC Name | Role (e.g. Internal) | Allowlisted CIDRs |
| --- | --- | --- | --- |
| [CUSTOMER]-nonprod-ext-de-c1 | vpc01-nonprod-de-c1 | External | 0.0.0.0/0 |
| [CUSTOMER]-nonprod-int-de-c1 | vpc01-nonprod-de-c1 | Internal | <<Internal corporate CIDRs>> |
| [CUSTOMER]-prod-ext-de-c1 | vpc02-prod-de-c1 | External | 0.0.0.0/0 |
| [CUSTOMER]-prod-int-de-c1 | vpc02-prod-de-c1 | Internal | <<Internal corporate CIDRs>> |

TODO: Complete the table ^

Additional details on CloudHub networking and Dedicated Load Balancers are available [here](https://docs.mulesoft.com/runtime-manager/cloudhub-networking-guide) and [here](https://docs.mulesoft.com/runtime-manager/cloudhub-dedicated-load-balancer) respectively.

#### 4.3.3.1. SSL endpoint and DNS configuration

External Facing Dedicated Load Balancer endpoints

| Environment | Public Endpoint DNS | Certificate Config | CNAME configuration |
| --- | --- | --- | --- |
| DEV | dev.api.[CUSTOMER].com | CA signed certificate | [CUSTOMER]-nonprod-ext-de-c1.lb.anypointdns.net |
| TEST | test.api.[CUSTOMER].com | CA signed certificate | [CUSTOMER]-nonprod-ext-de-c1.lb.anypointdns.net |
| PREPROD | pp.api.[CUSTOMER].com | CA signed certificate | [CUSTOMER]-nonprod-ext-de-c1.lb.anypointdns.net |
| PROD | api.[CUSTOMER].com | CA signed certificate | [CUSTOMER]-prod-ext-de-c1.lb.anypointdns.net |

TODO: Complete the table ^

Internal Facing Dedicated Load Balancer endpoints

| Environment | Public Endpoint DNS | Certificate Config | CNAME configuration |
| --- | --- | --- | --- |
| DEV | dev.api.[CUSTOMER].internal | Internal PKI signed Non-Prod SAN certificate with config for all three non-prod endpoints | [CUSTOMER]-nonprod-int-eu.lb.anypointdns.net |
| TEST | test.api.[CUSTOMER].internal | | [CUSTOMER]-nonprod-ext-eu.lb.anypointdns.net |
| PREPROD | pp.api.[CUSTOMER].internal | | [CUSTOMER]-nonprod-int-eu.lb.anypointdns.net |
| PROD | api.[CUSTOMER].internal | CA signed certificate | [CUSTOMER]-prod-int-eu.lb.anypointdns.net |

TODO: Complete the table ^

Additional details on resolving private domains in a VPC are available [here](https://docs.mulesoft.com/runtime-manager/resolve-private-domains-vpc-task).

#### 4.3.3.2. Routing Rules

The routing rules are aligned to the agreed application naming conventions to enable the effective routing of API traffic to the appropriate MuleSoft API endpoint. 

External DLB Mapping Rules

Update Mapping rules as required

| Detail | Input path | Target App | Output path | Protocol |
| --- | --- | --- | --- | --- |
| Routes for shared (global), generic external endpoints (to ext prefix apps) | /global/{consumer}/{function}/api/{version}/ | ext-[CUSTOMER]-exp-{consumer}-{function}-api-{version}-{subdomain}* | /api/ | https |
| Routes for shared (global), function specific (e.g. web / mobile) external endpoints (to ext prefix apps) | /global/{consumer}/api/{version}/ | ext-[CUSTOMER]-exp-{consumer}-api-{version}-{subdomain}* | /api/ | https |
| Routes for specific scope external endpoints (to ext prefix apps) | /{scope}/{consumer}/{function}/api/{version}/ | ext-{scope}-exp-{consumer}-{function}-api-{version}-{subdomain}* | /api/ | https |
| Routes for specific scope , specific function (e.g. web / mobile) external endpoints (to ext prefix apps) | /{scope}/{consumer}/api/{version}/ | ext-{scope}-exp-{consumer}-api-{version}-{subdomain}* | /api/ | https |
| Testing only in non-production | /internal/{app}/ | {app}-{subdomain}* | /api/ | https |

*Note: -{subdomain} suffix on the target app rules is only required for non production environments.*

Internal DLB Mapping Rules

| Detail | Input path | Target App | Output path | Protocol |
| --- | --- | --- | --- | --- |
| Routes for System API | /{scope}/sys/{system}/api/{version}/ | {scope}-sys-{system}-api-{version}-{subdomain} | /api/ | https |
| Routes for System API | /{scope}/sys/{system}/{domain}/api/{version}/ | {scope}-sys-{system}-{domain}-api-{version}-{subdomain} | /api/ | https |
| Routes for Process API | /{scope}/pro/{process}/api/{version}/ | {scope}-pro-{process}-api-{version}-{subdomain} | /api/ | https |
| Routes for Process API | /{scope}/pro/{process}/{subprocess}/api/{version}/ | {scope}-pro-{process}-{subprocess}-api-{version}-{subdomain} | /api/ | https |
| Routes for Experience API – Generic (global) endpoints | /global/{consumer}/api/{version}/ | ext-<customer>-exp-{consumer}-api-{version}-{subdomain} | /api/ | https |
| Routes for Experience API – Generic (global) endpoints, function specific (web, mobile) | /global/{consumer}/{function}/api/{version}/ | ext-<customer>-exp-{consumer}-{function}-api-{version}-{subdomain} | /api/ | https |
| Routes for Experience API | /{scope}/{consumer}/api/{version}/ | ext-{scope}-exp-{consumer}-api-{version}-{subdomain} | /api/ | https |
| Routes for Experience API - function specific (web, mobile) | /{scope}/{consumer}/{function}/api/{version}/ | ext-{scope}-exp-{consumer}-{function}-api-{version}-{subdomain} | /api/ | https |

*Note: -{subdomain} suffix on the target app rules is only required for non-production environments.*

#### 4.4. Infrastructure Setup (On-premise Bare Metal)

##### 4.4.1. Mule Clusters
Not applicable at this stage.

##### 4.4.2. Mule Server Groups 
Not applicable at this stage.

#### 4.5. Infrastructure Setup (Runtime Fabric)

##### 4.5.1. TCP Load Balancer
Not applicable at this stage.

##### 4.5.2. Workers 
Not applicable at this stage.

##### 4.5.3. Controllers 
Not applicable at this stage.

#### 4.6. High Availability (HA)

CloudHub provides HA and disaster recovery for hardware and application failures by using Amazon AWS as its cloud infrastructure. More details can be found in MuleSoft documentation [here](https://docs.mulesoft.com/runtime-manager/cloudhub-fabric).

From an applications perspective, HA is supported as follows: 
- Both vertical and horizontal scalability is supported. There is an option to either increase the worker size or add more workers for specific application. At a minimum, to ensure HA, CloudHub applications must be deployed to two or more workers – this in turn guarantees deployment of workers across different AWS Availability Zones. 
- Autoscaling is potentially a future state option, however this has commercial implications and will require an upgrade to Enterprise License Agreement (ELA). 
- CloudHub provides zero-downtime deployments to further address availability concerns (Note, zero-downtime deployments do not impact available vCore capacity) 

#### 4.7. Disaster Recovery

Several options exist within CloudHub for Disaster Recovery. Defined as the process by which a system is restored to a previous acceptable state, after a natural (flooding, tornadoes, earthquakes, fires, etc.) or human-made (power failures, server failures, misconfigurations, etc.) disaster. MuleSoft uses the Amazon AWS infrastructure to host CloudHub, which in turn provides capabilities to manage Disaster Recovery.

Disaster Recovery will be based on existing CloudHub Availability zone-based DR to support the current state enterprise needs. At this stage, cross-region Disaster Recovery is not considered as a priority given the acceptable risk profile of the default DR capability via Availability Zones. For future reference, the options available for DR are documented in section 4.7.2 

##### 4.7.1. CloudHub capability – key considerations
- CloudHub provides HA and disaster recovery for hardware and application failures by using Amazon AWS as its cloud infrastructure
- From a DR perspective, it is recommended to deploy the CloudHub control plane to a different region to runtime plane - this will enable management and redeployment of runtimes in the case of runtime region failure. For example, customers can configure the control plane in the US-East region, and the runtime plane currently in EU-Central (Frankfurt) region depending on regional considerations (e.g proximity to corporate applications and data).
- Mule applications deployed to multiple CloudHub workers, are deployed across AWS Availability Zones (AZ), which enable better isolation and protection from issues such as power outages, lightning strikes, tornadoes, earthquakes, etc. AZ’s are physically separated by a meaningful distance, many kilometers, from any other AZ, although all are within 100 km (60 miles) of each other

##### 4.7.2. Disaster Recovery – Options considered

- Disaster Recovery using Availability Zones: When MuleSoft applications are deployed to multiple workers, they are automatically distributed across 2 or more availability zones. This provides for availability as well as redundancy in the event of a disaster occurring in one Availability Zone (AZ). Where a Mule application is deployed across AZ’s, the customer is better isolated and protected from issues such as power outages, lightning strikes, tornadoes, earthquakes, etc. AZ’s are physically separated by a meaningful distance, many kilometers, from any other AZ, although all are within 100 km (60 miles) of each other.
- Disaster Recovery across Regions: It is also possible to achieve a greater level of Disaster Recovery protection by deploying Mule applications across multiple regions. This, in turn, will require additional infrastructure, processes, and tooling in place to enable deployments across multiple regions. There are different options that can be considered for Cross-region DR:
    - Cold Standby - The Mule environment is installed and configured. However, the Mule Runtimes are not running and are started after an outage is detected. This will require additional CloudHub components to be provisioned - specifically VPCs, DLBs, VPNs. As vCores will be in an undeployed state, this should not require additional vCores.
    - Warm Standby - The Mule environment is installed, configured, and fully running. However, it is not processing requests until an outage is detected. This will require additional CloudHub components to be provisioned - specifically VPCs, DLBs, VPNs, and vCores ‘in use’.
    - Active-Active - Mule applications are deployed across multiple regions in an active-active setting, with requests being processed by Mule applications across two or more regions. This will require additional CloudHub components to be provisioned - specifically VPCs, DLBs, VPNs, vCores - and will also require an external load balancer to manage traffic routing across the different regions.

Additional details on Amazon AWS infrastructure available [here](https://aws.amazon.com/infrastructure/)

