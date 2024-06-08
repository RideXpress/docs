# Common Services

- [Common Services](#common-services)
    - [API Implementation Template](#api-implementation-template)
    - [API Design Template](#api-design-template)
    - [Error Handling](#error-handling)
    - [Configuration Management](#configuration-management)
    - [Secure Properties](#secure-properties)

## Common Services

Provide details of common services that will be configured to standardize and streamline Mule application development and delivery>

### API Implementation Template

To support and streamline application development, API templates will be published for consistent and streamlined development activity. Templates will de created using maven archetypes and they will be published to our code repository and Anypoint Exchange.

These templates encapsulate the key standards, and structures to be used as part of development. Key elements included in the API template:
- Standard structure of Mule Application files
- Global configurations,
- API interface and implementation files

Projects will follow this structure:

| Name | Description |
| --- | --- |
| artifact-name.xml | This file will contain the API interface and all main flows auto-generated |
| implementation.xml| If multiple files are needed for implementation, a folder will be used with the same name and files will be named as per use case|
| common.xml | It will contain project configurations as database connections, jms, API autodisovery conf, HTTP request conf, etc. |
| errors.xml | It will contain error strategies for the project|


- Structure and placeholders for common, and environment specific properties, including secure properties.

Configurations properties will be yaml files and they will contain all common properties under config.yaml. 

| Name | Location | Description |
| --- | --- | --- |
| config.yaml | src/main/resources | Configurations |
| example.dwl | src/main/resources/dw | Dataweave files |
| example.dwl | src/main/resources/dw | Dataweave files |

- Standard maven POM structure, including Parent POM and related configurations

A parent POM will be used to include all common dependencies, deployment configurations and properties. All projects will declare a parent POM that will reside in the code repository.

- CICD structure.

There are 2 main branches in code repository: sandbox, main.
- main will be used as base to build applications into our sandbox and prod environment

[Github actions](https://docs.github.com/es/actions)  will be used to build CICD pipeline. It allows to automate, customize, and execute your software development workflows right in your repository with GitHub Actions. You can discover, create, and share actions to perform any job you'd like, including CI/CD, and combine actions in a completely customized workflow.

High level CI/CD pipeline:

1. Code is created in each developer branch
2. After completion, commit will be done and and pull request will be raised
3. After review, code will be merged into main branch
4. A github action will be triggered automatically to deploy code into sandbox environment and Artifactory
5. A different github action will be used to promote code to production


- Error handling configurations 

  Every project contains an errors.xml containing common error handling for all projects. Most projects will log exceptions and will return http status error code in the request, they will return #[error.description], #[error.errorType] and #[correlationId], depending of the nature of the API or Batch job/Sync, more detailed information will be needed including additional fields. #[error.errorType] can be used to define retries strategy, example: HTTP:CONNECTIVITY can be used to define if an Until Successful component needs to be used.

- Health Check framework (which can be integrated with Anypoint Functional Monitoring)

Every project will expose a GET endpoint for Health checks.
GET:/health-check
Health checks framework will be implemeted using Anypoint Functional Monitoring and alerts will be configured. Using the same convention for all projects will allow seamless integration with the framework.

- Logging standards and configurations

Logs should be formatted in JSON so they can be easily extracted by any Log Aggregation Tool. Logs will have the following format:
For the MVP, logs will in cloudhub only.
```
{
    "priority": "INFO",
    "correlationId": "e58ed763-928c-4155-bee9-fdbaaadc15f3",
    "timestamp": "2012-04-23T18:25:43.511Z",
    "message": "This is a log entry",
    "applicationName": "test-api",
    "applicationVersion": "1.0.0-SNAPSHOT",
    "environment": "sandbox",
    "payload": {
        "customFields": "This section will be reserved for custom fields"
    }
}
```


The following API templates have been published to Anypoint Exchange, and aligned to the Repository creation process: 
| Name | Link | When to use? |
| --- | --- | --- |
| mule4-rest-api-template | [<<LINK>>](link) | Use this template to support the development of REST APIs. |
| mule4-batch-template | [<<LINK>>](link) | Use this template to support the development of Batch applications. This template contains common aspects of Batch application development, including: <br> - Batch framework and common configuration placeholders (e.g. schedulers) <br> - Framework to support custom alerts, in conjunction with Alert Utility application. See below for details. |

### API Design Template

API Design Templates provide a starting point for the consistent structure and design of MuleSoft API specifications based on RAML. The API design template contains the following to support a consistent design approach: 
- Standard structure for API Design Artifacts
- Incorporation of common traits. Traits will be pusblished as single components, example: one RAML artifact for orderBy and a different trait for searheable. Unless multiple traits are used across different projects, a library will be created.  These provide for common handling of the following API aspects
    - API Authentication related headers
    - API Error handling structures and example content
    - API health checks
    - Standard design of common API functions – e.g. Pagination
The API design template is published the Anypoint Exchange and can be imported as part of new API creation in Anypoint Design Center: [<<LINK TO API DESIGN TEMPLATE>>](link)

### Error Handling

A common Error handler library has been configured to support consistent error handling approaches as part of MuleSoft projects. The library leverages existing Mule error handling capabilities and builds on this to provide a framework for the management for errors in Mule APIs and applications. 

The library includes default error handling logic for common error scenarios (e.g., standard HTTP errors) and the flexibility to support more bespoke error handling requirements. Most projects will log exceptions and will return http status error code in the request, they will return #[error.description], #[error.errorType] and #[correlationId], depending of the nature of the API or Batch job/Sync, more detailed information will be needed including additional fields.

A framework will be configured for different use cases:

    - APIs will use APIKit configurations and logging containing date, correlationId, error message and error type
    
    Every project contains an errors.xml containing common error handling for all projects. Most projects will log exceptions and will return http status error code in the request, they will return #[error.description], #[error.errorType] and #[correlationId], depending of the nature of the API or Batch job/Sync, more detailed information will be needed including additional fields. #[error.errorType] can be used to define retries strategy, example: HTTP:CONNECTIVITY can be used to define if an Until Successful component needs to be used.
    
    - Async processes will have a common error handler that will log the error and will have standardized reprocessing mechanism, every use will have the option to modify the reprocessing strategy as per requirements
    - If batch process are needed, errors will need a correlation Id as well and logging to batch results will be mandatory, special error handling will be performed as per requirements.


### Configuration Management

Mule applications typically need specific configurations properties to support implementations. These can be split into the following categories:
- Standard configuration properties – e.g. API endpoints. 
- Secure configuration properties – e.g. API/DB credentials

Properties can further be typically classified as common or environment agnostic properties, or environment specific properties. Common properties should be managed in a separate properties file vs. environment specific properties, to avoid repetition.
Configurations properties will be yaml files and they will contain all common properties under config.yaml. If properties values are changed accross environments, there will be a different property per environment: 

Examples:
square-staging-host: "connect.sandbox.squareup.com"
square-prod.yaml: "connect.squareup.com"

The API template will be defined the structure and placeholders for Mule API and application configuration management in line with best practice to manage common properties, environment specific properties and secure properties. 

### Secure properties

Secrets will be saved in github secrets and are accesible via github actions.

    - The secure property key needs to be updated to the CI/CD variable at the project level - and this needs to be done for each environment that the application is deployed to.
    - Only admins will be able to manage secure properties and any change needs to be approved by the team.
    
- Every API deployed on CloudHub must use safely hidden properties for sensitivity attributes like credentials. CloudHub supports safely hidden application properties, where the name of a property is visible in the console, but the value is not displayed or retrievable by any user. CloudHub resolves the property at runtime without exposing the sensitive information. More details available [here](link).