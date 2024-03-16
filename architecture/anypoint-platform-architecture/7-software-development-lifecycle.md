# Software Development Lifecycle

- [7. Software Development Lifecycle (SDLC)](#7-software-development-lifecycle.md)
    - [7.1. Mule Application Development Considerations](#71-mule-application-development-considerations)
    - [7.2. Version Control Strategy](#72-version-control-strategy)
    - [7.3. Deployment and Sizing Strategy](#73-deployment-and-sizing-strategy)
    - [7.4. Development Standards and Naming Conventions](#74development-standards-and-naming-conventions)
    - [7.5. Build and Deployment Automation (CI/CD)](#75-build-and-deployment-automation-cicd)
    - [7.6. SDLC Tooling Catalog](#76-sdlc-tooling-catalog)

## 7 Software Development Lifecycle

The following section describes the process and phases of the Software Development Life Cycle of a Mule application/API, the tooling and mechanisms used as part of it.

![alt text](./anypoint-platform-architecture/lifecycle.png)

| Phase | Description |
| --- | --- |
| Discovery | Gather information related to specific needs of a defined story (JIRA). Research the available APIs in Exchange. Identify the domain. |
| Design | Design the RAML in API Designer, publish the API Spec to Exchange. |
| Prototype and Validate | Validate the API specification is aligned to API consumer expectations through publishing documentation in Exchange, and through use of the API mocking tools to enable API consumers to interact with the API specification. |
| Development / Implementation | Using Anypoint Studio and reusing any available common service (e.g. auth-connector) <br> - Unit Testing (local): <br> - Push source code to Version Control System <br> - Run build and deployment automation tasks based on agreed and configured CICD processes. |
| Operate/Monitor | Using out of the box features included as part of Anypoint Monitoring to support the operational management and maintenance of the platform. |

## 7.1 Mule Application Development Considerations

## 7.2 Version Control Strategy

## 7.3 Deployment and Sizing

## 7.4 Development Standards and Naming Conventions

## 7.5 Build and Deployment Automation (CI/CD)

## 7.6 SDLC Tooling Catalog


## 9. SDLC
<Provide details of the Software development lifecycle and how the platform capabilities will support the agreed development approach and lifecycle, including details around CICD, project processes, governance and quality controls>



### 9.1. Mule Application Development considerations
#### 9.1.1. Version Control strategy
- Version Control tooling: All MuleSoft API and application source code will be version controlled and stored in a central GIT repository – Azure Repos. 
- Repository Strategy: 1 repository per Mule application. A Mule application can contain one or more Mule APIs or flows, balancing out the architecture principles of high cohesion, low coupling with the need to maximize vCore consumption. 
- Branching strategy: All application source code will leverage a common branching strategy to enable effective and consistent code development and maintenance practices. The branching strategy will use three branches: 
    - main - contains code that has been through the relevant integration testing, and is ready to deploy into acceptance environments (e.g. PREPROD), and eventually into production environments.
    - develop - used as the CI branch and contains code to be deployed into the Development and Functional test environment
        - feature-* - used for development of specific features as part of the delivery of new Mule API or application functionality, changes to existing Mule API capabilities, and defect fixes. 
        - hotfix-* - used for development activities to provide fixes to code already in production. 
        - All merges into the main and develop branch should be via Pull Request with manual reviews taking place.
        - Additional details on the branching strategy is available [here](link)

    ### 9.1.2. Deployment and sizing strategy
    As part of the CloudHub deployment model, every application/API will be running in an isolated container; Each application will require to be properly sized based on non-functional requirements, and results from performance testing. 

    The initial sizing of APIs and applications deployed to MuleSoft will be based on agreed guidelines with the C4E team. 

    ### 9.1.3. Development Standards and Naming Conventions
    All APIs and applications implemented in MuleSoft and deployed to CloudHub will need to adhere to development standards and guidelines, including naming conventions. Consistent standards and naming conventions support maintainability, discoverability and contribute to better quality code. This in turn helps drive reuse and pace of change, and help to reduce technical debt on the platform

    In addition, from a CloudHub application deployment perspective, the application name has the following constraints that need to be addressed as part of the naming convention: 
    - Application names cannot exceed 42 characters in length
    - Application names need to be unique across the MuleSoft deployment region (e.g. EU Frankfurt). This is because the application name is used to generate DNS records (e.g. <app-name>.de-c1.CloudHub.io)  

    The application development standards and guidelines are defined in detail [HERE](link)

    ### 9.2.	Build and Deployment Automation (CICD)

    ![alt text](./anypoint-platform-architecture/ci_cd.png)

    The solution will leverage Github actions to support the end to end development and deployment lifecycle of MuleSoft APIs and applications.  From a MuleSoft perspective, maven is used to support build and deployment activities, via the Mule Maven plugin. Note: The Mule Maven Plugin leverages CloudHub platform APIs under the hood). 

    The detailed configuration of Github Actions is documented [HERE](link)

    ### 9.3. SDLC Tooling Catalog

    | Phase | Tooling | Details |
    | --- | --- | --- |
    | Planning | Github Project | [Link](url) / Licensing and access considerations |
    | Documentation | Github Markdown | [Link](url) / Licensing and access considerations |
    | API Design | Anypoint Design Center | [Link](url) / Licensing and access considerations |
    | API Development and Unit Testing | Anypoint Studio | [Link](url) / Licensing and access considerations |
    | Version Control | Github | [Link](url) / Licensing and access considerations |
    | Build and Deployment Automation | Github Actions | [Link](url) / Licensing and access considerations |
    | Artifact Repository | Github | [Link](url) / Licensing and access considerations |
    | Static Code Analysis | SonarQube | TBC – not required for initial foundation configuration. |
    | Test Automation | TBC | TBC – not required for initial foundation configuration. |
    | Release Management | Github Releases | Aligned to CAB processes. |