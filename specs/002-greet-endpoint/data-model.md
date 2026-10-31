# Data Model: Greet Endpoint

## Name

The required value captured from the single path segment in `/greet/{name}`.

| Field | Type | Constraints |
|-------|------|-------------|
| `name` | string | Required; after URL path decoding, one or more ASCII letters (`A-Z`, `a-z`) or digits (`0-9`) only |

Invalid states include a missing or empty value, malformed percent encoding, or any decoded value containing characters outside the accepted set. These requests receive HTTP 400.

## Greeting Response

The JSON response returned for a valid name.

| Field | Type | Constraints |
|-------|------|-------------|
| `greeting` | string | Required; exactly `Hello, <name>!`, preserving the name's case |

The response contains no additional properties. It is returned with HTTP 200 and `application/json`.

## Existing Responses Preserved

- The health response remains the existing JSON object `{"status":"ok"}` with HTTP 200.
- Unhandled paths and unsupported methods continue to receive HTTP 404 with `{"error":"Not Found"}`.

There is no persistence, shared state, or relationship between request entities.