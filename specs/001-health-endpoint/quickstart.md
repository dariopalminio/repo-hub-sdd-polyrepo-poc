# Quickstart: Health Endpoint

## Prerequisites

- Node.js 22 or later and npm are installed.
- Node.js and npm are available for the one-time OpenAPI lint command.

## Start the Service

From the service member repository root, run:

```powershell
npm start
```

The service listens at `http://localhost:3000`. To use another port, set `PORT` before starting it:

```powershell
$env:PORT = 8080
npm start
```

## Validate the OpenAPI Contract

From the hub repository root, run:

```powershell
npx --yes @redocly/cli lint specs/001-health-endpoint/contracts/openapi.yaml
```

Expected result: the OpenAPI document passes linting and declares `GET /health` with the response schema described in [the contract](contracts/openapi.yaml).

## Validate the Running Endpoint

With the service running on its default port, set the base URL and run:

```powershell
$BaseUrl = 'http://localhost:3000'
$response = Invoke-WebRequest -Method Get -Uri "$BaseUrl/health"
if ($response.StatusCode -ne 200) { throw "Expected HTTP 200" }
if ($response.Headers['Content-Type'] -notmatch '^application/json(?:;|$)') { throw "Expected application/json" }
if ($response.Content -cne '{"status":"ok"}') { throw "Unexpected response body: $($response.Content)" }
```

Expected result: the command completes without throwing, and the response body is exactly `{"status":"ok"}`. When using a custom port, set `$BaseUrl` to that port before running the check.