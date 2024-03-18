# Operations Architecture

## 7. Operations Architecture
<Document details of the platform setup to support the operational management and maintenance of the platform>

The Anypoint platform provides different capabilities to support the operational aspects of managing and maintaining API and applications deployed to the MuleSoft platform.

### 7.1. Log Management
<Document details specific to log management, specifically in terms of log integration with third party tools, considerations around log retention within the platform etc. >

Mule applications will use a standard logging framework to ensure log data is captured consistently across Mule applications, with the right structure and data points to support effective troubleshooting. 

All application logs within Mule applications will be integrated with the corporate logging solution. This will enable log aggregation between multiple Mule apps (e.g. Experience, Process and System APIs) and other systems where applicable. Data will be pushed from MuleSoft to ELK via Log4J Socket appender configurations. For more information on options to integrate MuleSoft application log data to external applications, see Appendix A2. CloudHub Log data externalization options

Note that logs will not exclusively be pushed to ELK, and that these will also be available within the Anypoint platform – this is to facilitate MuleSoft support processes which require access to the logs as part of troubleshooting and supporting the delivery teams where required. 

Details on Custom Log4J configurations, including CloudHub settings to enable custom Log4J configurations are available here. 

### 7.2. Monitoring and Alerting
<Document details specific to monitoring and alerting capabilities and how this will be setup - both in terms of utilizing Anypoint platform capabilities, as well as external enterprise tooling where appropriate and how the platform data will be made available to these tools >

The Anypoint platform supports the following monitoring capabilities as part of the TITANIUM subscription. Depending on the relevant use cases and operational needs of the use case, one or more of the capabilities listed below can be configured as part of the use case implementation. 
- API / Application alerts - 
    - Alerts on API events (e.g. API errors)
    - Custom application alerts using CloudHub Notifications
    - If log aggregation to external tools, capability within the tooling can be leveraged to support advanced alerting use cases
- Anypoint Monitoring dashboards
    - Out of the box dashboards available for APIs and applications
    - Custom dashboards can be configured to consolidate information for operational / governance purposes 
- Anypoint Visualizer 
    - Troubleshooting and governance of application network
- Functional monitors 
    - Proactive health checking and alerting - can be used with implementation health check configurations

Additional Details: [Anypoint Monitoring Overview](link)

### 7.3. Platform Alerts
At a platform level, alerts should be configured when ARM encounters a problem with a Server or Server Cluster. These settings can be set at a global level, meaning for all APIs deployed to the Business Groups. At minimum, the following conditions should be monitored:

| Source/Server type | Condition | Severity Level | Config details |
| --- | --- | --- | --- |
| Application | CPU Usage | Critical | >=80% for at least 10 mins |
| Application | Deployment failed | Warning | NA |
| Application | Memory Usage (depends on worker size) | Critical | >=80% for at least 10 mins |
| Application | Worker not responding | Critical | NA |

Guidance for alert creation:
- Alert Naming Convention: 
    - `<source-type>-<application type>-<condition>`
    - Sample: `App-CloudHub-Deployment-Failed`
Notes:
- Naming convention is aligned to the key three input parameters that define the alert (i.e. source-type, application-type, condition).
- Name is used only for providing a reference of the to the alert configuration and used for listing here. 

Guidance for Alert Notification Subject creation: 
- Subject (naming convention):  
    - `${severity}: [CUSTOMER] Anypoint Alert (<env>) - default subject header>` 
where `<env>` is DEV, TEST, PREPROD or PROD. 
    - Sample: `HIGH: [CUSTOMER] Anypoint Alert (PROD) - Deployment Failed`
    - Note:  The use of this prefix is IMPORTANT to ensure email subject header is readily identifiable in a recipient’s inbox since there are no means to use properties or parameters to describe the environment where the issue originates from. This can also support future email-based integrations to Service Management tools (E.g. Service Now) 

For more information on ARM Alerts, refer to the following [Link](link).
Additional References: [Configure Alerts](link)