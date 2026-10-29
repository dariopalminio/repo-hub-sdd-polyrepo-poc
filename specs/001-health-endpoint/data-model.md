# Data Model: Health Endpoint

## Health Response

Represents the service's reported state when it can handle a health request. It is an HTTP response representation only; it is not persisted.

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `status` | string | Yes | Must equal `ok` |

The response contains no other fields. There are no relationships or state transitions.