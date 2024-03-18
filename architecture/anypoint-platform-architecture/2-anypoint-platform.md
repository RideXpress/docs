# Anypoint Platform

- [2 Anypoint Platform](#2-anypoint-platform)
    - [2.1 Anypoint Platform Licensed Capabilities](#21-anypoint-platform-licensed-capabilities)
    - [2.2 Platform Architecture and Setup](#22-platform-architecture-and-setup)
    - [2.3 Environments Model](#23-environments-model)
    - [2.4 Infrastructure Setup (CloudHub)](#24-infrastructure-setup-cloudhub)
    - [2.5 Load Balancing](#25-load-balancing)
    - [2.6 SSL Endpoint and DNS Configuration](#26-ssl-endpoint-and-dns-configuration)
    - [2.7 TCP Load Balancer](#27-tcp-load-balancer)
    - [2.8 Workers](#28-workers)
    - [2.9 Controllers](#29-controllers)
    - [2.10 High Availability (HA)](#210-high-availability-ha)
    - [2.11 Disaster Recovery](#211-disaster-recovery)
    - [2.12 CloudHub Capability – Key Considerations](#212-cloudhub-capability-key-considerations)
    - [2.13 Disaster Recovery – Options Considered](#213-disaster-recovery-options-considered)

## Architecture Decisions

<Document the key architecture decisions that have been made as part of the platform architecture definition.Please update the below decisions to reflect the platform architecture decisions agreed with the customer. > 

| Id | Decision | Rationale | Implication |
| --- | --- | --- | --- |
| Dxx | <Description of the decision> | <Describe the rationale of the decision – why was this decision made> | <what are the implications of this decision. What should the stakeholders be aware of because of this decision that they are signing up to> |
| D01 | MuleSoft CloudHub control plane will be hosted in the US. | ● Provide the ability to support multiple runtime plane regions as the platform scales – this is only possible with the US control plane, as the EU control plane only supports Frankfurt, and Ireland regions as of Mar/2022. <br> ● Control plane data is stored in the US. Note that this is limited to API and application meta-data, and CloudHub platform user authentication and authorization data. | |

## 2 Anypoint Platform
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

### 2.1 Anypoint Platform Licensed Capabilities

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

### 2.2 Platform Architecture and Setup

<Describe the key details of the platform setup with the customer landscape, including details of platform infrastructure considerations like VPCs, DLBs, VPNs > 

The following section describes how the Anypoint platform foundations are architected to support the long-term requirements of the enterprise.
 
_Update diagram based on agreed platform architecture_

### 2.3 Environments Model

Environments are a logical construct within Anypoint. Environments will be created at the business group level. Given the current Route to Live considerations, the minimum set of environments required would be as follows:

| Environment | Description | Classification (vCore) | High Availability? | Deployed By |
| --- | --- | --- | --- | --- |
| TEST | Used for functional and end to end testing | Sandbox | No | DevOps (Manual Approvals) |
| PROD | Production environment | Production | Yes | DevOps (Manual Approvals) |

### 2.4 Infrastructure Setup (CloudHub)

TODO: Infrastructure diagram

 Access from [CUSTOMER] Data Center subnets to Internal CloudHub DLB internal endpoint, or alternatively through use of the mule-worker-internal-* endpoints. |

#### 2.4.1 Load Balancing

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

#### 2.4.1.1 SSL endpoint and DNS configuration

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

##### 2.7 TCP Load Balancer
Not applicable at this stage.

##### 2.8 Workers 
Not applicable at this stage.

##### 2.9 Controllers 
Not applicable at this stage.

### 2.10 High Availability (HA)

CloudHub provides HA and disaster recovery for hardware and application failures by using Amazon AWS as its cloud infrastructure. More details can be found in MuleSoft documentation [here](https://docs.mulesoft.com/runtime-manager/cloudhub-fabric).

From an applications perspective, HA is supported as follows: 
- Both vertical and horizontal scalability is supported. There is an option to either increase the worker size or add more workers for specific application. At a minimum, to ensure HA, CloudHub applications must be deployed to two or more workers – this in turn guarantees deployment of workers across different AWS Availability Zones. 
- Autoscaling is potentially a future state option, however this has commercial implications and will require an upgrade to Enterprise License Agreement (ELA). 
- CloudHub provides zero-downtime deployments to further address availability concerns (Note, zero-downtime deployments do not impact available vCore capacity) 

### 2.11 Disaster Recovery

Several options exist within CloudHub for Disaster Recovery. Defined as the process by which a system is restored to a previous acceptable state, after a natural (flooding, tornadoes, earthquakes, fires, etc.) or human-made (power failures, server failures, misconfigurations, etc.) disaster. MuleSoft uses the Amazon AWS infrastructure to host CloudHub, which in turn provides capabilities to manage Disaster Recovery.

Disaster Recovery will be based on existing CloudHub Availability zone-based DR to support the current state enterprise needs. At this stage, cross-region Disaster Recovery is not considered as a priority given the acceptable risk profile of the default DR capability via Availability Zones. For future reference, the options available for DR are documented in section 4.7.2 

### 2.12 CloudHub capability – key considerations
- CloudHub provides HA and disaster recovery for hardware and application failures by using Amazon AWS as its cloud infrastructure
- From a DR perspective, it is recommended to deploy the CloudHub control plane to a different region to runtime plane - this will enable management and redeployment of runtimes in the case of runtime region failure. For example, customers can configure the control plane in the US-East region, and the runtime plane currently in EU-Central (Frankfurt) region depending on regional considerations (e.g proximity to corporate applications and data).
- Mule applications deployed to multiple CloudHub workers, are deployed across AWS Availability Zones (AZ), which enable better isolation and protection from issues such as power outages, lightning strikes, tornadoes, earthquakes, etc. AZ’s are physically separated by a meaningful distance, many kilometers, from any other AZ, although all are within 100 km (60 miles) of each other

### 2.13 Disaster Recovery – Options considered

- Disaster Recovery using Availability Zones: When MuleSoft applications are deployed to multiple workers, they are automatically distributed across 2 or more availability zones. This provides for availability as well as redundancy in the event of a disaster occurring in one Availability Zone (AZ). Where a Mule application is deployed across AZ’s, the customer is better isolated and protected from issues such as power outages, lightning strikes, tornadoes, earthquakes, etc. AZ’s are physically separated by a meaningful distance, many kilometers, from any other AZ, although all are within 100 km (60 miles) of each other.
- Disaster Recovery across Regions: It is also possible to achieve a greater level of Disaster Recovery protection by deploying Mule applications across multiple regions. This, in turn, will require additional infrastructure, processes, and tooling in place to enable deployments across multiple regions. There are different options that can be considered for Cross-region DR:
    - Cold Standby - The Mule environment is installed and configured. However, the Mule Runtimes are not running and are started after an outage is detected. This will require additional CloudHub components to be provisioned - specifically VPCs, DLBs, VPNs. As vCores will be in an undeployed state, this should not require additional vCores.
    - Warm Standby - The Mule environment is installed, configured, and fully running. However, it is not processing requests until an outage is detected. This will require additional CloudHub components to be provisioned - specifically VPCs, DLBs, VPNs, and vCores ‘in use’.
    - Active-Active - Mule applications are deployed across multiple regions in an active-active setting, with requests being processed by Mule applications across two or more regions. This will require additional CloudHub components to be provisioned - specifically VPCs, DLBs, VPNs, vCores - and will also require an external load balancer to manage traffic routing across the different regions.

Additional details on Amazon AWS infrastructure available [here](https://aws.amazon.com/infrastructure/)