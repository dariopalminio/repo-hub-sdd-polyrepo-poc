---
description: "Task list for the greet endpoint feature"
---

# Tasks: Greet Endpoint

**Input**: Design documents from `/specs/002-greet-endpoint/`

**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/openapi.yaml`, and `quickstart.md`

**Tests**: No automated test tasks are included because the service has no test runner and the specification does not request TDD. Manual validation is covered in the final phase.

**Organization**: Tasks are grouped by user story. Both stories are P1 and together deliver the feature MVP.

## Phase 1: Setup

**Purpose**: The service project already exists and requires no initialization or new dependencies.

No setup tasks are required.

## Phase 2: Foundational

**Purpose**: No shared infrastructure is needed; the route uses the existing HTTP server.

No foundational tasks are required.

## Phase 3: User Story 1 - Greet a named caller (Priority: P1)

**Goal**: Return the exact JSON greeting for a valid alphanumeric name and describe the operation in OpenAPI.

**Independent Test**: Request `GET /greet/Ada`; verify HTTP 200, JSON content type, exact body `{"greeting":"Hello, Ada!"}`, and the matching OpenAPI path parameter and response.

- [ ] T001 [P] [US1] Add the `GET /greet/{name}` OpenAPI operation, required path parameter, 200 greeting schema, and documented 400 response while preserving `/health` in `specs/002-greet-endpoint/contracts/openapi.yaml`
- [ ] T002 [P] [US1] Add the valid `GET /greet/{name}` response to the existing request handler while preserving `/health` and the 404 fallback in `repos/repo-service-sdd-polyrepo-poc/src/server.js`

**Checkpoint**: A valid name produces the contract-defined greeting without changing existing routes.

## Phase 4: User Story 2 - Reject an invalid name (Priority: P1)

**Goal**: Return HTTP 400 for an empty name or a name containing non-alphanumeric characters.

**Independent Test**: Request `/greet`, `/greet/`, and `/greet/Ada-Smith`; verify each returns HTTP 400 while `/health` and an unrelated path retain their existing responses.

- [ ] T003 [US2] Validate the decoded name as one or more ASCII letters or digits and return HTTP 400 for missing, empty, malformed, or invalid names in `repos/repo-service-sdd-polyrepo-poc/src/server.js`

**Checkpoint**: Valid and invalid name cases have distinct documented responses; health and 404 behavior remain unchanged.

## Phase 5: Polish & Cross-Cutting Concerns

**Purpose**: Verify the full contract and preserve existing service behavior.

- [ ] T004 Run the valid, invalid, health, and 404 scenarios in `specs/002-greet-endpoint/quickstart.md` and correct the guide if observed behavior differs

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No tasks; the service is already initialized.
- **Foundational (Phase 2)**: No tasks; no shared infrastructure is required.
- **User Story 1 (Phase 3)**: Can start immediately. T001 and T002 are independent and may run in parallel.
- **User Story 2 (Phase 4)**: T003 depends on T002 because invalid-input handling extends the new route.
- **Polish (Phase 5)**: T004 depends on T001, T002, and T003.

### User Story Dependencies

- **US1 (P1)**: No dependencies on other stories; independently verifies a valid greeting and its contract.
- **US2 (P1)**: Depends on the route from US1; independently verifies invalid-name handling and regression behavior.

### Parallel Opportunities

- T001 and T002 can be implemented in parallel because they modify different files.
- T003 follows T002; T004 follows all implementation tasks.

## Parallel Example: User Story 1

```text
Task: T001 - Update specs/002-greet-endpoint/contracts/openapi.yaml
Task: T002 - Update repos/repo-service-sdd-polyrepo-poc/src/server.js
```

## Implementation Strategy

### MVP Scope

Both P1 stories are required for the MVP: implement the valid greeting and invalid-name response, then verify the OpenAPI contract and existing route behavior.

### Incremental Delivery

1. Complete T001 and T002 in parallel; verify a valid name and contract.
2. Complete T003; verify empty and non-alphanumeric names return HTTP 400.
3. Complete T004; verify `/health` and the existing 404 response remain unchanged.

## Notes

- [P] tasks touch different files and have no dependencies on incomplete tasks.
- User story labels map tasks to acceptance criteria in `specs/002-greet-endpoint/spec.md`.
- Manual HTTP scenarios are documented in `specs/002-greet-endpoint/quickstart.md`.