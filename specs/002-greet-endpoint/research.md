# Research: Greet Endpoint

## Routing and Validation

- **Decision**: Extend the existing built-in `node:http` request handler and URL-based path parsing. Match exactly one name segment under `/greet/`, decode that segment once, and accept it only when it is non-empty and matches ASCII letters or digits. Treat an absent or empty name and invalid encoding as HTTP 400; leave unrelated paths and methods on the existing 404 path.
- **Rationale**: The service already uses this small, dependency-free structure for `/health`. The validation rule is specified as ASCII alphanumeric, so `[A-Za-z0-9]+` is deterministic and testable.
- **Alternatives considered**: Adding a routing framework or a separate controller layer was rejected because one route does not justify a dependency or structural expansion. Validating the entire pathname as the name was rejected because only the single route segment is the parameter.

## HTTP Contract

- **Decision**: Define the new operation in an OpenAPI 3.1 document under the feature's `contracts/` directory, retaining the existing `/health` definition. Declare `name` as a required path parameter with `minLength: 1` and pattern `^[A-Za-z0-9]+$`; describe the exact 200 greeting schema and a 400 response without a body schema.
- **Rationale**: The health feature already uses OpenAPI 3.1 and stores its contract with the feature artifacts. The request requires the 400 status but does not define an error body, so the contract should not invent one.
- **Alternatives considered**: Adding the new contract to the service repository was not selected because the existing feature contract is maintained under the workspace's `specs/` directory. Defining an error-body schema was rejected as unspecified scope.

## Validation Approach

- **Decision**: Use manual HTTP smoke checks plus inspection of the OpenAPI operation. Cover a valid name, an empty name, an invalid name, `/health`, and an unrelated path.
- **Rationale**: The service package currently exposes only `npm start`; it does not define a test script or include a test framework. These checks exercise the externally observable requirements without adding dependencies in this feature.
- **Alternatives considered**: Introducing a test framework during this endpoint change was rejected pending a separate project-level testing decision; it would expand scope beyond the requested route and contract.