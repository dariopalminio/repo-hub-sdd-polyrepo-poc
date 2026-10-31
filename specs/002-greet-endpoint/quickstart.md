# Quickstart: Greet Endpoint

## Prerequisites

- Node.js 22 or later.
- Run commands from the workspace root unless noted.

## Start the Service

In a terminal:

```powershell
Push-Location repos/repo-service-sdd-polyrepo-poc
npm start
```

The service listens at `http://localhost:3000` by default. Keep this terminal running and use another terminal for the requests below.

## Validate Runtime Behavior

Valid name:

```powershell
curl.exe -i http://localhost:3000/greet/Ada
```

Expected: HTTP 200, JSON content type, and body `{"greeting":"Hello, Ada!"}`.

Empty name:

```powershell
curl.exe -i http://localhost:3000/greet
curl.exe -i http://localhost:3000/greet/
```

Expected: HTTP 400 for both requests.

Non-alphanumeric name:

```powershell
curl.exe -i http://localhost:3000/greet/Ada-Smith
```

Expected: HTTP 400.

Malformed percent-encoding:

```powershell
node -e "require('node:http').get({host:'localhost', port:3000, path:'/greet/%ZZ'}, r => { console.log(r.statusCode); r.resume(); })"
```

Expected: HTTP 400. This sends the path literally so the client does not normalize or reject the malformed escape first.

Regression checks:

```powershell
curl.exe -i http://localhost:3000/health
curl.exe -i http://localhost:3000/not-a-route
```

Expected: `/health` remains HTTP 200 with `{"status":"ok"}`; the unrelated path remains HTTP 404 with `{"error":"Not Found"}`.

## Validate the Contract

Inspect [contracts/openapi.yaml](contracts/openapi.yaml) and verify that:

- `/greet/{name}` declares a required path parameter with the same non-empty ASCII alphanumeric rule.
- The 200 response describes the exact greeting object.
- The 400 response is documented without inventing an unspecified response body.
- The existing `/health` operation remains unchanged.