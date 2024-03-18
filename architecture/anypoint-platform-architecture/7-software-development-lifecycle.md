# Software Development Lifecycle

- [7 Software Development Lifecycle (SDLC)](#7-software-development-lifecycle.md)
    - [7.1 Mule Application Development Considerations](#71-mule-application-development-considerations)
    - [7.2 Version Control Strategy](#72-version-control-strategy)
    - [7.3 Deployment and Sizing Strategy](#73-deployment-and-sizing-strategy)
    - [7.4 Development Standards and Naming Conventions](#74-development-standards-and-naming-conventions)
    - [7.5 Build and Deployment Automation (CI/CD)](#75-build-and-deployment-automation-cicd)
    - [7.6 SDLC Tooling Catalog](#76-sdlc-tooling-catalog)

## 7 Software Development Lifecycle (SDLC)

The following section describes the process and phases of the Software Development Life Cycle of a Mule application/API, the tooling and mechanisms used as part of it.

![alt text](./images/lifecycle.png)
[TODO] Update lifecycle diagram, it looks out-to-date

| Phase | Description |
| --- | --- |
| Discovery | Gather information related to specific needs of a defined story (GitHub issues). Research the available APIs in Exchange. Identify the domain. |
| Design | Design the RAML in API Designer |
| Prototype and Validate | Validate the API specification is aligned to API consumer expectations through publishing documentation in Exchange, and through use of the API mocking tools to enable API consumers to interact with the API specification. |
| Development / Implementation | Using Anypoint Studio and reusing any available common service (e.g. json-logger) <br> - Unit Testing (local): <br> - Push source code to Version Control System <br> - Run build and deployment automation tasks based on agreed and configured CI/CD processes. |
| Operate/Monitor | Using out of the box features included as part of Anypoint Monitoring to support the operational management and maintenance of the platform. |

## 7.1 Mule Application Development Considerations

- **Version Control tooling**: All Mulesoft code will be stored and managed in GitHub.
- **Repository Strategy**: We'll pick a monorepo to store all the MuleSoft-related code. This approach will bring some advantages that can be found [here](https://semaphoreci.com/blog/what-is-monorepo). APIs or Integrations will be related to one system of records per System API i.e. okta-system-api, salesforce-sytem-api, google-maps-system-api. One Process API per use case. i.e. request-ride-process-api, accept-ride-process-api, finish-ride-process-api, etc. And one Experience API per consumer platform. i.e. mobile-experience-api.
- **Branching strategy**: We'll follow the widely known [Git flow](https://www.gitkraken.com/learn/git/git-flow) which consists in keeping the latest stable code in the main branch and adding new features from feature branches that are mapped to user stories (GitHub issues). Merging changes into the main branch will require a core review and approval from at least 1 team member.

## 7.2 Version Control Strategy

The development team will follow the [Semantic Versioning](https://semver.org) and the Apache Maven conventions.

From SemVer we'll take the following convention:

Given a version number MAJOR.MINOR.PATCH, increment the:

- **MAJOR** version when you make incompatible API changes
- **MINOR** version when you add functionality in a backward compatible manner
- **PATCH** version when you make backward compatible bug fixes

From Maven we'll make use of **SNAPSHOT** versions that are extremely handy for ongoing development.

- **okta-system-api-1.0.0-SNAPSHOT** is a version that is under development
- **okta-system-api-1.0.0** is a release version

## 7.3 Deployment and Sizing Strategy

For the MVP all the applications will use the minimum capacity

| Worker Size | Worker Memory | Heap Memory | Disk Storage Size |
| --- | --- | --- | --- |
| 0.1 vCores | 1 GB | 500 MB | 8 GB |

Workers with 0.1 vCores and 0.2 vCores:
- Provide limited CPU and I/O for apps with smaller workloads
- Can burst to higher CPU speeds for a short time
- This ability helps to improve application startup times and to process infrequent, large workloads. If you need consistent performance, use workers with more vCores.

| Mule Runtime Version |
| --- |
| 4.6.1 |

[Reference](https://docs.mulesoft.com/cloudhub/cloudhub-architecture#cloudhub-workers)

## 7.4 Development Standards and Naming Conventions

All APIs and applications implemented in MuleSoft and deployed to CloudHub will need to adhere to development standards and guidelines, including naming conventions. Consistent standards and naming conventions support maintainability, discoverability and contribute to better quality code. This in turn helps drive reuse and pace of change, and help to reduce technical debt in the future.

| **Domain** | **Convention** |
| --- | --- |
| **Maven Artifact Id** | Full names are favored to improve readibility |
| **Cloudhub domain** | Cloudhub follows the `https://{application_name}.cloudhub.io` convention with a maximum length of 42 characters and names should be unique for the whole cloudhub.io domain. For RideXpress we'll adopt the following convention: `https://ridexpress-{api_name}.cloudhub.io/{api_version}/{api_resources}`. Please refer to the table below for more details. |
| **API version** | We'll follow the `v1, v2, v{n}` standard |
| **API Resources organization** | API resources should be aligned with business resources and should follow the most common REST API standard `/{resources}/{resource_id}` i.e. `users/123456` |
| **API Methods** | [TODO] |
| **API body format** | [TODO] |
| **API pagination and filtering** | [TODO] |
| **Files names** | [TODO] |
| **Flow names** | [TODO] |
| **Dataweave files** | [TODO] |
| **Connector configuration names** | [TODO] |
| **Log files (when required)** | [TODO] |



| **API** | **Maven Artifact Id** | **Cloudhub domain** |
| --- | --- | --- |
| Mobile | mobile-experience-api | https://ridexpress-mobile.cloudhub.io/v1 |
| User Sign up | user-sign-up-process-api | https://ridexpress-user-sign-up.cloudhub.io/v1 |
| Request Ride | request-ride-process-api | https://ridexpress-request-ride.cloudhub.io/v1 |
| Accept Ride | accept-ride-process-api | https://ridexpress-accept-ride.cloudhub.io/v1 |
| Wait for Ride | wait-for-ride-process-api | https://ridexpress-wait-for-ride.cloudhub.io/v1 |
| Finish Ride | finish-ride-process-api | https://ridexpress-finish-ride.cloudhub.io/v1 |
| Okta | okta-system-api | https://ridexpress-okta.cloudhub.io/v1 |
| Google Maps | google-maps-system-api | https://ridexpress-google-maps.cloudhub.io/v1 |
| Salesfore | salesforce-system-api | https://ridexpress-salesforce.cloudhub.io/v1 |
| Square | square-system-api | https://ridexpress-square.cloudhub.io/v1 |
| Push Notifications | push-notifications-system-api | https://ridexpress-push-notifications.cloudhub.io/v1 |
| Database | database-system-api | https://ridexpress-db.cloudhub.io/v1 |
| Email | email-system-api | https://ridexpress-email.cloudhub.io/v1 |


For additional details such as error handling or logging, please refer to the Archetype design document.

[TODO] Link to the Archetype design document
[TODO] For further discussion:
Should Names in Design Center be Human friendly? The Audience for Design Center is Architects and since we plan to automate the sync between Design Center, Studio and Exchange, probably matching with Maven would be more functional even though I think Human Friendly should be the norm.
What about Exchange? The audience here may not be very technical so they are more focused on the functionality side and also on the process side so my suggestion is to make it Human friendly.
My last question is around the table above, is this the right document? Initially I wanted to describe the naming convention only but somehow I ended up creating that table that may include the asset in Exchange and the API in Design Center.
Initially I thought about using the artifact id in the cloudhub domain but we have a 42 characters restriction and RideXpress already used 10 so I kept it short and I liked it. Also thought about using the api layer as part of the url but didn't find any value in doing so, keeping the API url simple seems more adequate to me. The layers are more relevant at the Maven and the Exchange level.

## 7.5 Build and Deployment Automation (CI/CD)

 The solution will leverage Github actions to support the end-to-end development and deployment lifecycle of MuleSoft APIs and applications. From a MuleSoft perspective, Maven is used to support test, build and deployment activities through the Mule Maven plugin.
 
 The CI/CD Pipeline will also use Anypoint CLI to orchestrate some tasks to automate repetitive tasks such as pulling the latest changes from Design Center, publishing snapshot versions to Exchange, Applying predefined Policies and SLAs to API Manager and deploying to Runtime Manager.

 For more information about the CI/CD configuration, please refer to the CI/CD design document.
 [TODO] Link to the CI/CD design document

## 7.6 SDLC Tooling Catalog

| Phase | Tooling | Details |
| --- | --- | --- |
| Planning | [Github Projects](https://docs.github.com/en/issues/planning-and-tracking-with-projects/learning-about-projects/about-projects) | Projects is an adaptable, flexible tool for planning and tracking work on GitHub. |
| Documentation | [Github Markdown](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax) | Markdown is a human-readable format that is perfect for documentation and integrates perfectly with Github |
| API Design | [Anypoint Design Center](https://www.mulesoft.com/platform/anypoint-design-center) | Design and build APIs and integrations at lightning speed, all in one product |
| API Development and Unit Testing | [Anypoint Studio](https://www.mulesoft.com/platform/studio) | IDE for integration and API development that has prebuilt modules for common integration requirements, including querying backend systems, routing events, business transformation logic, and error handling. <br>- Jumpstart your integrations with prebuilt connectors, templates, and examples<br>- Debug with design time error handling<br>- Normalize, join, filter, or map any data format<br>- Integrate automated testing into your existing CI/CD pipeline with MUnit<br>- Deploy APIs and integrations to the cloud or on-premises<br> |
| Version Control | [Github](https://github.com) | The complete developer platform to build, scale, and deliver secure software. Github is a collaborative platform for Version Control |
| Build and Deployment Automation | [Github Actions](https://github.com/features/actions)<br> [Anypoint CLI](https://docs.mulesoft.com/anypoint-cli/latest/) | GitHub Actions makes it easy to automate all your software workflows, now with world-class CI/CD. Build, test, and deploy your code right from GitHub. Make code reviews, branch management, and issue triaging work the way you want. Anypoint CLI will be used to automate and ochestrate the snapshots deployment and release process |
| Artifact Repository | [Anypoint Exchange](https://www.mulesoft.com/exchange) | Anypoint Exchange is the marketplace for connectors, templates, examples, and APIs. Discover and use prebuilt assets from the MuleSoft ecosystem, or use Exchange to save, share, and reuse internal best practices. |
| Release Management | [Github Releases](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository) | You can create new releases with release notes, @mentions of contributors, and links to binary files, as well as edit or delete existing releases. You can also create, modify, and delete releases by using the Releases API. For more information, see "REST API endpoints for releases" in the REST API documentation. |

[TODO] Add more tools and utilities like a rest client, MUnit, draw.io, etc.