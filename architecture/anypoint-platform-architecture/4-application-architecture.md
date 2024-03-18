# Application Architecture

## 4. Platform Architecture and Setup

<Describe the key details of the platform setup with the customer landscape, including details of platform infrastructure considerations like VPCs, DLBs, VPNs > 

The following section describes how the Anypoint platform foundations are architected to support the long-term requirements of the enterprise.
 
_Update diagram based on agreed platform architecture_

### 4.1. Environments Model

Environments are a logical construct within Anypoint. Environments will be created at the business group level. Given the current Route to Live considerations, the minimum set of environments required would be as follows:

| Environment | Description | Classification (vCore) | High Availability? | Deployed By |
| --- | --- | --- | --- | --- |
| TEST | Used for functional and end to end testing | Sandbox | No | DevOps (Manual Approvals) |
| PROD | Production environment | Production | Yes | DevOps (Manual Approvals) |

### 4.2. Infrastructure Setup (CloudHub)

TODO: Infrastructure diagram

 Access from [CUSTOMER] Data Center subnets to Internal CloudHub DLB internal endpoint, or alternatively through use of the mule-worker-internal-* endpoints. |

#### 4.2.1. Load Balancing

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

#### 4.2.1.1. SSL endpoint and DNS configuration

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

##### 4.3.1. TCP Load Balancer
Not applicable at this stage.

##### 4.3.2. Workers 
Not applicable at this stage.

##### 4.3.3. Controllers 
Not applicable at this stage.

### 4.4. High Availability (HA)

CloudHub provides HA and disaster recovery for hardware and application failures by using Amazon AWS as its cloud infrastructure. More details can be found in MuleSoft documentation [here](https://docs.mulesoft.com/runtime-manager/cloudhub-fabric).

From an applications perspective, HA is supported as follows: 
- Both vertical and horizontal scalability is supported. There is an option to either increase the worker size or add more workers for specific application. At a minimum, to ensure HA, CloudHub applications must be deployed to two or more workers – this in turn guarantees deployment of workers across different AWS Availability Zones. 
- Autoscaling is potentially a future state option, however this has commercial implications and will require an upgrade to Enterprise License Agreement (ELA). 
- CloudHub provides zero-downtime deployments to further address availability concerns (Note, zero-downtime deployments do not impact available vCore capacity) 

### 4.5. Disaster Recovery

Several options exist within CloudHub for Disaster Recovery. Defined as the process by which a system is restored to a previous acceptable state, after a natural (flooding, tornadoes, earthquakes, fires, etc.) or human-made (power failures, server failures, misconfigurations, etc.) disaster. MuleSoft uses the Amazon AWS infrastructure to host CloudHub, which in turn provides capabilities to manage Disaster Recovery.

Disaster Recovery will be based on existing CloudHub Availability zone-based DR to support the current state enterprise needs. At this stage, cross-region Disaster Recovery is not considered as a priority given the acceptable risk profile of the default DR capability via Availability Zones. For future reference, the options available for DR are documented in section 4.7.2 

##### 4.5.1. CloudHub capability – key considerations
- CloudHub provides HA and disaster recovery for hardware and application failures by using Amazon AWS as its cloud infrastructure
- From a DR perspective, it is recommended to deploy the CloudHub control plane to a different region to runtime plane - this will enable management and redeployment of runtimes in the case of runtime region failure. For example, customers can configure the control plane in the US-East region, and the runtime plane currently in EU-Central (Frankfurt) region depending on regional considerations (e.g proximity to corporate applications and data).
- Mule applications deployed to multiple CloudHub workers, are deployed across AWS Availability Zones (AZ), which enable better isolation and protection from issues such as power outages, lightning strikes, tornadoes, earthquakes, etc. AZ’s are physically separated by a meaningful distance, many kilometers, from any other AZ, although all are within 100 km (60 miles) of each other

##### 4.5.2. Disaster Recovery – Options considered

- Disaster Recovery using Availability Zones: When MuleSoft applications are deployed to multiple workers, they are automatically distributed across 2 or more availability zones. This provides for availability as well as redundancy in the event of a disaster occurring in one Availability Zone (AZ). Where a Mule application is deployed across AZ’s, the customer is better isolated and protected from issues such as power outages, lightning strikes, tornadoes, earthquakes, etc. AZ’s are physically separated by a meaningful distance, many kilometers, from any other AZ, although all are within 100 km (60 miles) of each other.
- Disaster Recovery across Regions: It is also possible to achieve a greater level of Disaster Recovery protection by deploying Mule applications across multiple regions. This, in turn, will require additional infrastructure, processes, and tooling in place to enable deployments across multiple regions. There are different options that can be considered for Cross-region DR:
    - Cold Standby - The Mule environment is installed and configured. However, the Mule Runtimes are not running and are started after an outage is detected. This will require additional CloudHub components to be provisioned - specifically VPCs, DLBs, VPNs. As vCores will be in an undeployed state, this should not require additional vCores.
    - Warm Standby - The Mule environment is installed, configured, and fully running. However, it is not processing requests until an outage is detected. This will require additional CloudHub components to be provisioned - specifically VPCs, DLBs, VPNs, and vCores ‘in use’.
    - Active-Active - Mule applications are deployed across multiple regions in an active-active setting, with requests being processed by Mule applications across two or more regions. This will require additional CloudHub components to be provisioned - specifically VPCs, DLBs, VPNs, vCores - and will also require an external load balancer to manage traffic routing across the different regions.

Additional details on Amazon AWS infrastructure available [here](https://aws.amazon.com/infrastructure/)

## 6. Application Architecture
This section is a drill down of the Platform Architecture, covering the application related components and their interactions.

### 6.1. API Led Connectivity

The key goal of the approach recommended by this document is to introduce reusable building blocks that can be reused both during the initial implementation and by future projects, resulting in reduced development effort.

API-led connectivity is a methodical way to connect data to applications through a series of reusable and purposeful modern APIs that are each developed to play a specific role – unlock data from systems, compose data into processes, or deliver an experience.

The API building block is a product that consists of functionality and simplicity required for the full lifecycle of APIs. This lifecycle consists of the ability to compose the data and connect to any other source of data. And it must provide full visibility, security, governance right from design.

The diagram below illustrates the API-led connectivity approach composed of three main layers:
- System APIs to unlock backend systems through a consistent contract, making use of our extensive connectivity, 
- Process APIs providing orchestration and transformation into business domain objects for greater agility and value creation and 
- Experience APIs focused on rendering information specific to devices or consuming client applications.

![alt text](./anypoint-platform-architecture/api_led.png)

The value of this approach is enabling a flexible, agile architecture built for reuse and consumption, to speed up project delivery with built in governance and security.

### 6.2. Integration Patterns
Based on the discussions and discovery sessions, the following integration patterns were identified:

| Pattern | Description | Examples |
| --- | --- | --- |
| Real time, Synchronous | Real time or near real time requests to retrieve or process data, or trigger business processes, where the requester requires immediate feedback to support the scenario. | ● Address Verification <br> ● Policy Number Generation |
| Async - Fire and Forget | Async, near real time, Fire and Forget type scenarios where API clients send messages but do not need to wait for a response as part of their processes. | ● Policy Creation (async UW validation, core setup etc.) <br> ● Claim creation (rules processing, etc.) |