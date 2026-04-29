# Archetype Design

## Overview

The RideXpress archetype is a pre-configured project template that standardizes how MuleSoft API applications are created. It provides a consistent foundation for all new APIs, reducing setup time and ensuring adherence to architectural standards.

## Archetype Structure

All RideXpress MuleSoft projects follow this standard directory structure:

```
{api-name}/
├── src/
│   ├── main/
│   │   ├── mule/
│   │   │   ├── {api-name}.xml           # Main flow
│   │   │   ├── global-config.xml         # Global configurations
│   │   │   ├── error-handler.xml         # Centralized error handling
│   │   │   └── implementations/          # Implementation flows
│   │   └── resources/
│   │       ├── api/                      # RAML/OAS specification
│   │       ├── properties/
│   │       │   ├── local.yaml            # Local environment properties
│   │       │   ├── dev.yaml              # Development properties
│   │       │   ├── test.yaml             # Test properties
│   │       │   └── prod.yaml             # Production properties
│   │       └── dwl/                      # DataWeave transformation files
│   └── test/
│       ├── munit/                        # MUnit test suites
│       └── resources/                    # Test resources and fixtures
├── pom.xml                               # Maven project configuration
├── mule-artifact.json                    # Mule artifact descriptor
└── README.md                             # Project-specific documentation
```

## Archetype Components

### Global Configuration (`global-config.xml`)

- HTTP Listener configuration (shared via domain project)
- HTTP Request configurations for downstream APIs
- Error handler reference
- Secure properties configuration
- Autodiscovery configuration for API Manager

### Error Handling (`error-handler.xml`)

All APIs use the standardized `errors-library` to provide consistent error responses:

| HTTP Status | Error Type            | Description                          |
|-------------|-----------------------|--------------------------------------|
| 400         | BAD_REQUEST           | Invalid request payload or parameters |
| 401         | UNAUTHORIZED          | Missing or invalid authentication     |
| 403         | FORBIDDEN             | Insufficient permissions              |
| 404         | NOT_FOUND             | Resource not found                    |
| 405         | METHOD_NOT_ALLOWED    | HTTP method not supported             |
| 409         | CONFLICT              | Resource state conflict               |
| 429         | TOO_MANY_REQUESTS     | Rate limit exceeded                   |
| 500         | INTERNAL_SERVER_ERROR | Unexpected server error               |
| 502         | BAD_GATEWAY           | Downstream service error              |
| 503         | SERVICE_UNAVAILABLE   | Service temporarily unavailable       |

### Properties Management

Properties are externalized per environment using YAML files:

- **`local.yaml`**: For local development and testing
- **`dev.yaml`**: Development environment (CloudHub Sandbox)
- **`test.yaml`**: Test/staging environment (CloudHub Sandbox)
- **`prod.yaml`**: Production environment (CloudHub Production)

Sensitive properties (passwords, tokens, keys) are encrypted using the Mule Secure Properties module.

### Domain Project

A shared domain project (`ridexpress-domain`) provides common configurations reused across all APIs:

- Shared HTTP Listener on a common port
- Common DataWeave utility functions
- Shared error response structures
- Common logging configuration

## Creating a New API from the Archetype

1. Run the Maven archetype generation command:
   ```bash
   mvn archetype:generate \
     -DarchetypeGroupId=com.ridexpress \
     -DarchetypeArtifactId=ridexpress-api-archetype \
     -DarchetypeVersion=1.0.0 \
     -DgroupId=com.ridexpress \
     -DartifactId={api-name} \
     -Dversion=1.0.0-SNAPSHOT
   ```
2. Import the RAML specification from Design Center into `src/main/resources/api/`.
3. Scaffold the API flows using Anypoint Studio's API Kit Router.
4. Implement the business logic in the `implementations/` directory.
5. Write MUnit tests in `src/test/munit/`.
6. Configure environment properties in `src/main/resources/properties/`.

## Naming Conventions

| Component       | Convention                                    | Example                          |
|-----------------|-----------------------------------------------|----------------------------------|
| API artifact    | `{domain}-{layer}-api`                        | `square-system-api`              |
| Main flow       | `{api-name}.xml`                              | `square-system-api.xml`          |
| Implementation  | `{resource}-impl.xml`                         | `payments-impl.xml`              |
| MUnit test      | `{resource}-test-suite.xml`                   | `payments-test-suite.xml`        |
| DataWeave file  | `{operation}-{direction}.dwl`                 | `create-payment-request.dwl`     |
| Properties      | `{environment}.yaml`                          | `dev.yaml`                       |
