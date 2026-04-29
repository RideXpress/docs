# Archetype Design

- [Archetype Design](#archetype-design)
  - [Overview](#overview)
  - [Getting Started](#getting-started)
    - [Option 1 – Maven Archetype](#option-1--maven-archetype)
    - [Option 2 – Download Template from Exchange](#option-2--download-template-from-exchange)
  - [Project Structure](#project-structure)
  - [Components](#components)
    - [HTTP Listener](#http-listener)
    - [Listener Path](#listener-path)
    - [JSON Logger](#json-logger)
    - [Error Handling](#error-handling)
    - [Configuration Files](#configuration-files)
    - [Autodiscovery](#autodiscovery)
    - [Metrics and Monitoring](#metrics-and-monitoring)
  - [Parent POM](#parent-pom)
  - [References](#references)

---

## Overview

The RideXpress Maven Archetype provides a standardised starting point for every MuleSoft API project. It encapsulates the conventions defined in the [Software Development Lifecycle](anypoint-platform-architecture/software-development-lifecycle.md) and the [Common MuleSoft Best Practices](common-mulesoft-best-practices.md) documents, so that every new API is born already compliant and ready to run on CloudHub.

Developers have two ways to bootstrap a new API project:

1. **Maven Archetype** – generate the scaffolding from the command line.
2. **Anypoint Exchange template** – download and import the template directly into Anypoint Studio or Code Builder.

After bootstrapping the project the developer downloads the RAML specification from Design Center to generate the implementation scaffolding code using the MuleSoft Maven plugin.

---

## Getting Started

### Option 1 – Maven Archetype

Run the following command in your terminal, replacing the placeholder values with the actual API details:

```bash
mvn archetype:generate \
  -DarchetypeGroupId=com.ridexpress \
  -DarchetypeArtifactId=ridexpress-api-archetype \
  -DarchetypeVersion=1.0.0 \
  -DgroupId=com.ridexpress \
  -DartifactId=<api-name> \
  -Dversion=1.0.0-SNAPSHOT \
  -DapiName=<api-name> \
  -DapiVersion=v1
```

**Parameter reference**

| Parameter | Description | Example |
|-----------|-------------|---------|
| `artifactId` | Maven artifact identifier (see [naming conventions](anypoint-platform-architecture/software-development-lifecycle.md#maven-artifact-id)) | `okta-system-api` |
| `apiName` | Human-readable API name | `Okta System API` |
| `apiVersion` | Semantic API version prefix | `v1` |

### Option 2 – Download Template from Exchange

1. Open **Anypoint Exchange** and search for `RideXpress API Template`.
2. Click **Download** → **As Template**.
3. Import the downloaded project into **Anypoint Studio** or **Anypoint Code Builder**.
4. Rename the project and update `pom.xml` with the correct `artifactId` and `version`.

---

## Project Structure

```
<api-name>/
├── pom.xml                          # Maven build descriptor (inherits parent POM)
├── mule-artifact.json               # Mule runtime metadata
└── src/
    ├── main/
    │   ├── mule/
    │   │   ├── <api-name>.xml       # Main Mule configuration (HTTP Listener, autodiscovery)
    │   │   ├── error-handling.xml   # Global error handler
    │   │   └── global-config.xml    # Connector and property placeholder configurations
    │   └── resources/
    │       ├── api/                 # RAML specification (downloaded from Design Center)
    │       ├── dwl/                 # DataWeave transformation files
    │       ├── properties/
    │       │   ├── sandbox.yaml     # Sandbox environment properties
    │       │   ├── dev.yaml         # Development environment properties
    │       │   └── prod.yaml        # Production environment properties
    │       └── log4j2.xml           # Logging configuration
    └── test/
        └── munit/                   # MUnit test suites
```

---

## Components

### HTTP Listener

Every API exposes an HTTPS endpoint through a single HTTP Listener configuration. The listener uses the `${https.port}` property so that it is automatically assigned by CloudHub at runtime.

**`global-config.xml` – Listener configuration**

```xml
<http:listener-config name="http-listener-config"
                      doc:name="HTTP Listener Config">
    <http:listener-connection
        host="0.0.0.0"
        port="${https.port}"
        protocol="HTTPS" />
</http:listener-config>
```

**`<api-name>.xml` – Listener usage**

```xml
<http:listener config-ref="http-listener-config"
               path="${api.basePath}/*"
               doc:name="HTTP Listener" />
```

> **Note:** `${https.port}` resolves to `8082` on CloudHub. Local development can override this value in the environment properties file.

---

### Listener Path

The HTTP Listener base path follows the URL convention defined in the [Software Development Lifecycle](anypoint-platform-architecture/software-development-lifecycle.md#cloudhub-domain) document:

```
/{api_version}/{resources}/{resource_id}
```

The full CloudHub URL follows the pattern:

```
https://ridexpress-{environment}-{api_short_name}.shard.{region}.cloudhub.io/{api_version}/{resources}
```

**Examples**

```
https://ridexpress-prd-mobile.shard.usa-w2.cloudhub.io/v1/rides
https://ridexpress-prd-okta.shard.usa-w2.cloudhub.io/v1/users
https://ridexpress-prd-google-maps.shard.usa-w2.cloudhub.io/v1/routes
```

The `api.basePath` property in the environment properties file should be set to `/{api_version}`, e.g. `/v1`.

---

### JSON Logger

All APIs use the JSON Logger library for structured, machine-readable log output. Log entries are emitted in JSON format so they can be ingested by any log aggregation tool.

**Dependency in `pom.xml`**

```xml
<dependency>
    <groupId>org.mule.modules</groupId>
    <artifactId>mule-json-logger-module</artifactId>
    <version>${json.logger.version}</version>
    <classifier>mule-plugin</classifier>
</dependency>
```

**`log4j2.xml`** – root appender configuration outputs JSON to stdout so CloudHub captures it automatically.

**Logger usage inside flows**

```xml
<json-logger:logger
    doc:name="Log request"
    config-ref="json-logger-config"
    message="API request received"
    level="INFO" />
```

The log output follows the standard defined in the [SDLC document](anypoint-platform-architecture/software-development-lifecycle.md#log-files):

```json
{
    "priority": "INFO",
    "correlationId": "e58ed763-928c-4155-bee9-fdbaaadc15f3",
    "timestamp": "2024-01-15T10:30:00.000Z",
    "message": "API request received",
    "applicationName": "okta-system-api",
    "applicationVersion": "1.0.0-SNAPSHOT",
    "environment": "sandbox",
    "payload": {}
}
```

---

### Error Handling

A global error handler (`error-handling.xml`) is included in the archetype to provide consistent HTTP error responses across all APIs. It maps standard Mule error types to the appropriate HTTP status codes and returns a uniform JSON error payload.

**`error-handling.xml`**

```xml
<error-handler name="global-error-handler" doc:name="Global Error Handler">

    <!-- Client errors -->
    <on-error-propagate type="HTTP:BAD_REQUEST, MULE:EXPRESSION, MULE:TRANSFORMATION"
                        enableNotifications="true" logException="true">
        <http:listener-error-response statusCode="400">
            <http:body>#[output application/json --- { "status": 400, "message": "Bad Request", "detail": error.description }]</http:body>
        </http:listener-error-response>
    </on-error-propagate>

    <on-error-propagate type="HTTP:UNAUTHORIZED"
                        enableNotifications="true" logException="false">
        <http:listener-error-response statusCode="401">
            <http:body>#[output application/json --- { "status": 401, "message": "Unauthorized", "detail": error.description }]</http:body>
        </http:listener-error-response>
    </on-error-propagate>

    <on-error-propagate type="HTTP:FORBIDDEN"
                        enableNotifications="true" logException="false">
        <http:listener-error-response statusCode="403">
            <http:body>#[output application/json --- { "status": 403, "message": "Forbidden", "detail": error.description }]</http:body>
        </http:listener-error-response>
    </on-error-propagate>

    <on-error-propagate type="HTTP:NOT_FOUND"
                        enableNotifications="true" logException="false">
        <http:listener-error-response statusCode="404">
            <http:body>#[output application/json --- { "status": 404, "message": "Not Found", "detail": error.description }]</http:body>
        </http:listener-error-response>
    </on-error-propagate>

    <!-- Server / connectivity errors -->
    <on-error-propagate type="HTTP:CONNECTIVITY, HTTP:TIMEOUT"
                        enableNotifications="true" logException="true">
        <http:listener-error-response statusCode="503">
            <http:body>#[output application/json --- { "status": 503, "message": "Service Unavailable", "detail": error.description }]</http:body>
        </http:listener-error-response>
    </on-error-propagate>

    <!-- Catch-all -->
    <on-error-propagate type="ANY"
                        enableNotifications="true" logException="true">
        <http:listener-error-response statusCode="500">
            <http:body>#[output application/json --- { "status": 500, "message": "Internal Server Error", "detail": error.description }]</http:body>
        </http:listener-error-response>
    </on-error-propagate>

</error-handler>
```

The global error handler is referenced in the main configuration file:

```xml
<flow name="main-flow">
    <http:listener config-ref="http-listener-config" path="${api.basePath}/*" />
    <!-- flow logic -->
    <error-handler ref="global-error-handler" />
</flow>
```

---

### Configuration Files

Environment-specific properties are stored in YAML files under `src/main/resources/properties/`. The active file is selected at startup via the `env` system property (e.g. `-Denv=sandbox`).

**`global-config.xml` – property placeholder**

```xml
<configuration-properties
    file="properties/${env}.yaml"
    doc:name="Configuration Properties" />
```

**`sandbox.yaml` template**

```yaml
# HTTP / CloudHub
https:
  port: "8082"

api:
  basePath: "/v1"
  id: "<api-manager-api-id>"           # Populated by CI/CD pipeline after API Manager registration

# JSON Logger
json:
  logger:
    applicationName: "${app.name}"
    applicationVersion: "${app.version}"
    environment: "sandbox"
    level: "DEBUG"

# Anypoint Monitoring (optional)
monitoring:
  enabled: false
```

**`prod.yaml` template**

```yaml
https:
  port: "8082"

api:
  basePath: "/v1"
  id: "<api-manager-api-id>"

json:
  logger:
    applicationName: "${app.name}"
    applicationVersion: "${app.version}"
    environment: "prod"
    level: "INFO"

monitoring:
  enabled: true
```

> Sensitive values (credentials, secrets) must **never** be stored in property files. Use **CloudHub Properties** (encrypted at rest) or **Anypoint Secrets Manager** instead.

---

### Autodiscovery

The Autodiscovery component registers the running application with **Anypoint API Manager**, enabling policy enforcement, analytics, and client management.

**`<api-name>.xml` – Autodiscovery configuration**

```xml
<api-gateway:autodiscovery
    apiId="${api.id}"
    flowRef="main-flow"
    doc:name="Autodiscovery" />
```

The `api.id` property is populated automatically by the CI/CD pipeline after the API is registered in API Manager (see the [CI/CD design document](ci-cd-design.md#3-build-and-deploy)).

---

### Metrics and Monitoring

When `monitoring.enabled` is `true`, the API emits custom metrics to **Anypoint Monitoring** using the Anypoint Monitoring connector. This allows tracking business KPIs in addition to the default infrastructure metrics.

**Dependency in `pom.xml`** (include only when metrics are required)

```xml
<dependency>
    <groupId>com.mulesoft.connectors</groupId>
    <artifactId>mule-custom-metrics-extension</artifactId>
    <version>${custom.metrics.version}</version>
    <classifier>mule-plugin</classifier>
</dependency>
```

**Usage example**

```xml
<custom-metrics:increment-counter
    doc:name="Increment ride request counter"
    counterName="ride_requests_total"
    incrementValue="1" />
```

> The metrics component is **optional** and should only be added when there is a specific need to track custom business metrics beyond the built-in Anypoint Monitoring capabilities.

---

## Parent POM

All RideXpress API projects inherit from the shared `ridexpress-parent-pom`. This parent POM manages:

- Mule Maven Plugin version and configuration
- Common dependency versions (JSON Logger, MUnit, etc.)
- CloudHub deployment settings (region, worker size, Mule Runtime version)
- MUnit coverage thresholds

**`pom.xml` – parent reference**

```xml
<parent>
    <groupId>com.ridexpress</groupId>
    <artifactId>ridexpress-parent-pom</artifactId>
    <version>1.0.0</version>
    <relativePath />    <!-- resolved from Anypoint Exchange -->
</parent>

<groupId>com.ridexpress</groupId>
<artifactId>okta-system-api</artifactId>
<version>1.0.0-SNAPSHOT</version>
<packaging>mule-application</packaging>
```

Worker size and Mule Runtime version defaults are defined in the parent POM aligned with the [Deployment and Sizing Strategy](anypoint-platform-architecture/software-development-lifecycle.md#deployment-and-sizing-strategy):

| Setting | Value |
|---------|-------|
| Worker size | `0.1 vCores` |
| Worker memory | `1 GB` |
| Mule Runtime | `4.6.1` |
| CloudHub region | `us-east-2` (USA West 2) |

---

## References

- [Software Development Lifecycle](anypoint-platform-architecture/software-development-lifecycle.md)
- [Common MuleSoft Best Practices](common-mulesoft-best-practices.md)
- [CI/CD Design](ci-cd-design.md)
- [API Development Process](api-development-process.md)
- [MuleSoft JSON Logger](https://github.com/mulesoft-catalyst/json-logger)
- [Anypoint Autodiscovery](https://docs.mulesoft.com/api-manager/2.x/api-gateway-configure-autodiscovery)
- [CloudHub Workers](https://docs.mulesoft.com/cloudhub/cloudhub-architecture#cloudhub-workers)
- [Anypoint Custom Metrics](https://docs.mulesoft.com/monitoring/custom-metrics-connector)
