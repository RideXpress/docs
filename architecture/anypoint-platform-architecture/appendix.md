# Appendix

## A. Appendix
    ### A.1. Backend Systems Catalog

    | System name | Location | Access Protocol | Host | Access |
    | --- | --- | --- | --- | --- |
    | Salesforce - sandbox - partial | Cloud | HTTPS | https://my.salesforce.com | Integration user |
    | Salesforce - sandbox - full | Cloud | HTTPS | TBD | Integration user |
    | Salesforce - production | Cloud | HTTPS | TBD | Integration user |
    | Workday - sandbox | Cloud | HTTPS | wd5-impl-services1.workday.com | Integration user |
    | Workday - production | Cloud | HTTPS | TBD | Integration user |

    ### A.2. CloudHub Log data externalization options

    ![alt text](./anypoint-platform-architecture/logs_external.png)

    ### A.3. CloudHub DLB Detailed Connectivity Flow

    ![alt text](./anypoint-platform-architecture/dlb_sequence.png)

    ### A.4. Anypoint Team profiles and permissions model

    | #.# | Team | Description |
    | --- | --- | --- |
    | | Everyone at [CUSTOMER] | Root level Team which all subsequent teams will inherit. Assign view only to users who would like to explore the Anypoint platform and discover assets within Anypoint Exchange. Note: This can be limited in case confidential assets exist within a specific Business Group |
    | 1 | C4E | Core C4E team. Assign C4E users who want read-only permissions across the Anypoint platform (All Business Groups) and the ability to contribute to the documentation within Exchange. This includes access to Design Center, and View permissions to API manager, Anypoint MQ, Runtime Manager, and Anypoint Monitoring. |
    | 1.1 | C4E Platform Administrators | C4E platform administrators. Assign users who have global administrator access to the platform, from management of infrastructure and administration of wider aspects of the platform. This includes administration and management access to API manager, Runtime manager, Anypoint MQ, Anypoint Monitoring, including Org Administrator permissions. This Team should be limited in access – e.g., assigned to core C4E Platform Lead |
    | 1.2 | C4E Platform Engineers | C4E platform engineers. Assign users who are responsible for the technical management and evolution of the Platform. This includes the ability to manage and support the configuration of the platform infrastructure through Runtime Manager (e.g. creation and configuration of VPC, VPN, DLBs) and administrator level access to Anypoint Monitoring, Visualizer and Exchange |
    | 2 | Business Group Teams (E.g. [CUSTOMER] Central IT) | Business group team. Assign business and project users who need to view Exchange assets, and review API and application metrics via Anypoint Monitoring within the specific business group. |
    | 2.1 | Development Team | Development team for a specific business group. Assign to business unit specific development teams. This team will have permissions to support the following: Contribute assets and documents to Exchange. Design APIs in Design Center. Manage APIs in API Manager, Anypoint MQ and Runtime Manager in DEV. Support troubleshooting activities in TEST, PREPROD activities via read only permissions to API Manager, Anypoint MQ and Runtime Manager. |
    | 2.2 | Business Analysis Team | Business Analysis Team for a specific business group. Assign to members who need to review and collaborate on API specifications and documentation in Exchange. |
    | 2.3 | Test Team | Test Team for a specific business group. Assign to members who support testing activities of Mule APIs and applications. This team will need the ability to view and manage aspects of applications in non production environments – E.g. ability to view and manage queues, manage schedules in Runtime Manager (for batch applications) and view application logs. |
    | 2.4 | Business Support and Operations | Operations Team for a specific business group. Assign to business unit specific operational users who need to analyse and troubleshoot issues aligned to the business group, including support analyst who are responsible for L1 and L2 support of the platform. This role will need to permissions for the following: Ability to view API configurations in API Manager in preprod and production environments. Ability to view Runtime Manager configurations in preprod and production environments. Ability to view messaging assets in Anypoint MQ. |
    | 2.5 | Group Administrators | Administrators for a specific business group. Assign to business unit specific administrators to manage configuration of business group, including managing permissions and users within the business group. |
    | 3 | Global Support and Operations | Central Support and Operations Team. Assign to Central support and operations teams |
    | 3.1 | Support Analysts | Shared Services and Support Analysts across the organization. Assign to support analysts who are responsible for L1 and L2 support of the platform, in a shared services model. This role will need to permissions for the following: Ability to view API configurations in API Manager in preprod and production environments. Ability to view Runtime Manager configurations in preprod and production environments. Ability to view messaging assets in Anypoint MQ. |
    | 3.2 | Release Managers | Central Release Management Team. Assign to the release management team who will be responsible for managing Mule application related releases. Ability to manage API configurations in API Manager in preprod and production environments. Ability to manage Runtime Manager configurations in preprod and production environments. Ability to manage messaging assets in Anypoint MQ. |