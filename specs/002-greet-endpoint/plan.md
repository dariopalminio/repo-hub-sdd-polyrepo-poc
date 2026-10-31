# Implementation Plan: Greet Endpoint

**Branch**: `002-greet-endpoint` | **Date**: 2026-10-31 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/002-greet-endpoint/spec.md`

## Affected Repositories

| Repo Path | Type | Reason |
|-----------|------|--------|
| repos/repo-service-sdd-polyrepo-poc | independent | Adds the `/greet/{name}` service endpoint |

## Summary

Add `GET /greet/{name}` to the existing Node.js HTTP service. Reuse its built-in HTTP server and URL parsing, validate one decoded path segment against the ASCII alphanumeric rule, and return the specified JSON greeting or HTTP 400. Add the matching OpenAPI 3.1 contract while preserving `/health` and the existing 404 response.

## Technical Context

**Language/Version**: JavaScript, Node.js >=22 (from the service package engine requirement)

**Primary Dependencies**: Node.js built-in `node:http` and `URL`; no external runtime dependencies

**Storage**: N/A

**Testing**: Manual HTTP smoke checks; the service currently has no test script or test framework

**Target Platform**: Node.js 22 or later

**Project Type**: HTTP web service in the independent member repository `repos/repo-service-sdd-polyrepo-poc`

**Performance Goals**: No feature-specific performance target; preserve the current lightweight request handling

**Constraints**: Keep `GET /health` unchanged; preserve the existing JSON 404 behavior for unhandled requests; accept only non-empty ASCII letters and digits for the name; add no runtime dependency

**Scale/Scope**: One new route, one response entity, and one OpenAPI operation; no persistence or downstream integrations

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

The constitution file is still an unratified template. No project principles or gates are inferred from its example placeholders.

- No ratified constraint applies to this small endpoint addition. Reusing the built-in HTTP server and existing contract conventions introduces no justified complexity or dependency.
- **Post-design gate: PASS**; the completed design stays within the stated scope, adds no dependencies, and has no applicable ratified constitution constraints.

## Project Structure

### Documentation (this feature)

```text
specs/002-greet-endpoint/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
└── contracts/
  └── openapi.yaml
```

### Service Repository

```text
repos/repo-service-sdd-polyrepo-poc/
├── package.json
└── src/
    └── server.js
```

**Structure Decision**: Keep runtime changes within `repos/repo-service-sdd-polyrepo-poc/src/server.js`. Keep the feature's OpenAPI contract with its design artifacts under `specs/002-greet-endpoint/contracts/`, matching the existing health-endpoint feature. No test directory currently exists; validation will use the documented HTTP smoke checks.
