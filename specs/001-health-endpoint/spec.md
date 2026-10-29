# Feature Specification: Health Endpoint

**Feature Branch**: `001-health-endpoint`

**Created**: 2026-10-28

**Status**: Draft

**Input**: User description: "Añadir un endpoint `GET /health` en el repo servicio que devuelva `{"status":"ok"}`. El contrato OpenAPI debe reflejarlo."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Check service availability (Priority: P1)

A monitoring client or service operator requests the health endpoint and receives a clear, predictable indication that the service is available.

**Why this priority**: A reliable health response lets operators and monitoring clients identify an available service without interpreting application-specific data.

**Independent Test**: Send a `GET /health` request and verify its HTTP status, content type, and exact JSON response against the published API contract.

**Acceptance Scenarios**:

1. **Given** the service can handle requests, **When** a client sends `GET /health`, **Then** the service responds with HTTP 200, JSON content, and exactly `{"status":"ok"}`.
2. **Given** the health endpoint is included in the service API contract, **When** a client inspects the OpenAPI definition, **Then** it finds `GET /health` with a successful response matching the runtime status and response body.

### Edge Cases

- Requests using a method other than `GET` are outside this feature's contract.
- If the service cannot handle the health request, it must not report a successful health response.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The service MUST expose a health endpoint at `GET /health`.
- **FR-002**: When the service can handle the request, the endpoint MUST return HTTP 200 with JSON content whose complete body is exactly `{"status":"ok"}`.
- **FR-003**: The OpenAPI contract MUST declare `GET /health`, its HTTP 200 response, and a response schema that requires the `status` property to have the value `ok` and permits no additional properties.
- **FR-004**: The runtime response and the OpenAPI response definition MUST describe the same status code and JSON body.

### Key Entities

- **Health response**: The endpoint's response indicating service availability; its only property is `status`, with the value `ok` when the service can handle the request.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of successful health requests return HTTP 200 and the exact JSON object `{"status":"ok"}`.
- **SC-002**: The OpenAPI definition includes `GET /health` and a successful response definition that matches the runtime response in every contract check.
- **SC-003**: A monitoring client can determine the service's reported healthy state from one request and the `status` value, without interpreting additional response fields.

## Assumptions

- HTTP 200 and `application/json` are the successful response status and content type for this endpoint.
- This endpoint reports whether the service can handle the health request; checking downstream dependencies is outside this feature's scope.
- The health endpoint does not require application credentials so monitoring can probe it; infrastructure may still restrict network access.