# Common Services

- [6. Common Services](#common-services)
    - [6.1. API Implementation Template](#api-implementation-template)
    - [6.2. API Design Template](#api-design-template)
    - [6.3. Error Handling](#error-handling)
    - [6.4. Configuration Management](#configuration-management)
    - [6.5. Secure Properties](#secure-properties)

## 8. Common Services
<Provide details of common services that will be configured to standardize and streamline Mule application development and delivery>

### 8.1. API Implementation Template
To support and streamline application development, API templates will be published for consistent and streamlined development activity. 

These templates encapsulate the key standards, and structures to be used as part of development. Key elements included in the API template:
- Standard structure of Mule Application files – including global configurations, API interface and implementation files
- Structure and placeholders for common, and environment specific properties, including secure properties. 
- Standard maven POM structure, including Parent POM and related configurations
- CICD structure.
- Error handling configurations (utilizing common error handler framework) 
- Health Check framework (which can be integrated with Anypoint Functional Monitoring) 
- Logging standards and configurations

The following API templates have been published to Anypoint Exchange, and aligned to the Repository creation process: 
| Name | Link | When to use? |
| --- | --- | --- |
| mule4-rest-api-template | [<<LINK>>](link) | Use this template to support the development of REST APIs. |
| mule4-anypoint-mq-consumer-template | [<<LINK>>](link) | Use this template to support the development of Anypoint MQ Consumer Application. |
| mule4-batch-template | [<<LINK>>](link) | Use this template to support the development of Batch applications. This template contains common aspects of Batch application development, including: <br> - Batch framework and common configuration placeholders (e.g. schedulers) <br> - Framework to support custom alerts, in conjunction with Alert Utility application. See below for details. |

### 8.2. API Design Template
API Design Templates provide a starting point for the consistent structure and design of MuleSoft API specifications based on RAML. The API design template contains the following to support a consistent design approach: 
- Standard structure for API Design Artifacts
- Incorporation of common traits (published separately as common-traits-library). These provide for common handling of the following API aspects
    - API Authentication related headers
    - API Error handling structures and example content
    - API health checks
    - Standard design of common API functions – e.g. Pagination
The API design template is published the Anypoint Exchange and can be imported as part of new API creation in Anypoint Design Center: [<<LINK TO API DESIGN TEMPLATE>>](link)

### 8.3. Error Handling
A common Error handler library has been configured to support consistent error handling approaches as part of MuleSoft projects. The library leverages existing Mule error handling capabilities and builds on this to provide a framework for the management for errors in Mule APIs and applications. 

The library includes default error handling logic for common error scenarios (e.g., standard HTTP errors) and the flexibility to support more bespoke error handling requirements. 

A detailed view of the error handler library, including guidelines on how to implement this as part of Mule API implementations is available [<<HERE>>](link)

### 8.4. Configuration Management
Mule applications typically need specific configurations properties to support implementations. These can be split into the following categories:
- Standard configuration properties – e.g. API endpoints. 
- Secure configuration properties – e.g. API/DB credentials

Properties can further be typically classified as common or environment agnostic properties, or environment specific properties. Common properties should be managed in a separate properties file vs. environment specific properties, to avoid repetition.

The API template will be defined the structure and placeholders for Mule API and application configuration management in line with best practice to manage common properties, environment specific properties and secure properties. 

#### 8.4.1. Secure properties
- MuleSoft provides a secure property placeholder component to enable the encryption of secure properties using a private key. These can then be stored in an encrypted form in the codebase, which in turn is used to build the application and leveraged at runtime. Additional details available [here](link)
    - The secure property key needs to be updated to the CI/CD variable at the project level - and this needs to be done for each environment that the application is deployed to.
    - Note the underlying process and ownership of managing and encrypting secure properties will need to be agreed as part of the broader development and operations process. 
- Every API deployed on CloudHub must use safely hidden properties for sensitivity attributes like credentials. CloudHub supports safely hidden application properties, where the name of a property is visible in the console, but the value is not displayed or retrievable by any user. CloudHub resolves the property at runtime without exposing the sensitive information. More details available [here](link).