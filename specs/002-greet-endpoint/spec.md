# Feature Specification: Greet Endpoint

**Feature Branch**: `002-greet-endpoint`

**Created**: 2026-10-31

**Status**: Draft

**Input**: User description: "Añadir un endpoint `GET /greet/{name}` en el repo servicio. Debe extraer el nombre de la ruta y devolver HTTP 200 con `{"greeting": "Hello, <name>!"}`. Si el nombre está vacío o contiene caracteres no alfanuméricos, devolver HTTP 400. El contrato OpenAPI debe reflejar el parámetro de ruta y las respuestas. Asegúrate de mantener la lógica existente de `GET /health` y el manejo de 404."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Greet a named caller (Priority: P1)

A client submits a valid alphanumeric name and receives a greeting containing that name.

**Why this priority**: This is the endpoint's primary purpose.

**Independent Test**: Send `GET /greet/Ada` and verify the status, JSON content type, exact response body, and matching API contract.

**Acceptance Scenarios**:

1. **Given** a non-empty alphanumeric name, **When** a client sends `GET /greet/Ada`, **Then** the service responds with HTTP 200 and `{"greeting":"Hello, Ada!"}`.
2. **Given** the endpoint is documented, **When** a client inspects the OpenAPI contract, **Then** it finds the required `name` path parameter and the documented 200 and 400 responses.

### User Story 2 - Reject an invalid name (Priority: P1)

A client receives a client-error response when the name is empty or contains characters outside the accepted alphanumeric set.

**Why this priority**: Predictable validation prevents invalid input from being treated as a valid greeting.

**Independent Test**: Request the endpoint with an empty name, a name containing punctuation or whitespace, and a malformed percent-encoding; verify each returns HTTP 400.

**Acceptance Scenarios**:

1. **Given** an empty name, **When** a client requests `/greet` or `/greet/`, **Then** the service responds with HTTP 400.
2. **Given** a name containing non-alphanumeric characters, **When** a client requests `GET /greet/<name>`, **Then** the service responds with HTTP 400.
2. **Given** a name containing non-alphanumeric characters, **When** a client requests `GET /greet/<name>`, **Then** the service responds with HTTP 400.
3. **Given** a malformed percent-encoding in the name path segment, **When** a client requests `GET /greet/%ZZ`, **Then** the service responds with HTTP 400.

### Edge Cases

- Names preserve their original letter case in the greeting.
- Requests to unrelated paths continue to receive the existing HTTP 404 response.
- Existing `GET /health` behavior and response remain unchanged.
- Requests using methods other than `GET` continue to receive the existing HTTP 404 response.
- Requests using methods other than `GET` continue to receive the existing HTTP 404 response.
- A malformed percent-encoding in the name path segment receives HTTP 400.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The service MUST expose `GET /greet/{name}`.
- **FR-002**: The service MUST accept a non-empty name consisting only of ASCII letters (`A-Z`, `a-z`) and digits (`0-9`).
- **FR-003**: For a valid name, the service MUST return HTTP 200 with JSON body exactly `{"greeting":"Hello, <name>!"}`, preserving the supplied name.
- **FR-004**: The service MUST return HTTP 400 when the name is missing or empty, contains any non-alphanumeric character after path decoding, or has malformed percent-encoding.
- **FR-005**: The existing `GET /health` endpoint MUST continue to return HTTP 200 with JSON body exactly `{"status":"ok"}`.
- **FR-006**: Unhandled paths and methods other than `GET` MUST retain the existing HTTP 404 JSON response `{"error":"Not Found"}`.
- **FR-007**: The OpenAPI contract MUST declare the required `name` path parameter, the HTTP 200 greeting response, and the HTTP 400 invalid-name response, while preserving the existing `/health` definition.
- **FR-008**: The OpenAPI response definition for HTTP 200 MUST match the runtime status and greeting body. The HTTP 400 response MUST be documented; its response body is unspecified by this feature.

### Key Entities

- **Greeting response**: The response for a valid name; it contains the greeting text formed from the supplied name.
- **Name**: The required path value, limited to one or more ASCII letters or digits.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of requests with valid names return HTTP 200 and the exact greeting for the supplied name.
- **SC-002**: 100% of requests with empty, malformedly encoded, or non-alphanumeric names return HTTP 400.
- **SC-003**: `GET /health` continues to return HTTP 200 and the exact JSON object `{"status":"ok"}`.
- **SC-004**: Unhandled paths and unsupported methods continue to return the existing HTTP 404 response.
- **SC-005**: Contract checks confirm the path parameter and documented responses match runtime behavior.

## Assumptions

- “Alphanumeric” means ASCII letters and digits; punctuation, spaces, and non-ASCII letters are rejected.
- An omitted name (`/greet`) and an empty final path segment (`/greet/`) are treated as an empty name and return HTTP 400.
- An omitted name (`/greet`) and an empty final path segment (`/greet/`) are treated as an empty name and return HTTP 400.
- The name path segment is decoded once; malformed percent-encoding is rejected with HTTP 400.
- The HTTP 400 status is required, but its response body is not specified.
- The OpenAPI contract remains in the feature artifacts under the hub, consistent with the existing health-endpoint feature.