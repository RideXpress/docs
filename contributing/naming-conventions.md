## API Conventions in MuleSoft - RideXpress

# Naming Conventions
APIs will be named {layer}-{entity}-{env}
QueryParams will be written camel case

# URL Structure1
/v1/{entity}/{id}/{subentity}
/v1/{entity}?{queryParams}={} -> query params in all the filter criteria

# Versioning
All APIs will use as root `/v1` or the corresponding API version as required

# HTTP Verbs
All API will use HTTP verbs (GET, POST, PUT, DELETE) accurately to perform the respective operations on resources.

# Request/Response Formats
All APIs will use JSON as the default format for request and response payloads.
All the resources will use a consistent structure for JSON objects, using camelCase for field names.
Include appropriate headers to specify the content type and encoding (e.g., `Content-Type: application/json`).

# Error Handling
All APIs will use the common error handling dependency “TBD”  

# Pagination and Filtering
Use the predefined traits for pagination and filtering.

# Authentication and Authorization
All APIs will use the common trait in the secure configuration
The archetype should define the basic Auth config

# Testing and Validation
CI/CD should enforce the tests execution, minimum of 80% of code coverage
