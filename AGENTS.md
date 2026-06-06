# AGENTS.md

This repository is designed for both human developers and AI agents.

AI agents must use this file as the working guide for reading, reviewing, and modifying this repository.

## Required Reading Order

Before making or reviewing changes, read the following files in order:

1. `README.md`
2. `design-index.yaml`
3. `docs/application-basic-design.md`
4. Relevant RAML root file
5. Relevant Operation RAML fragment
6. `docs/application-detail-design.md`
7. Relevant Mule XML, DataWeave, and MUnit files if implementation exists

## Core Hierarchy

```text
Application
  └─ API
       └─ Operation
```

- Application = Mule app / jar / repository / deployment unit
- API = RAML root / API Manager unit / APIkit router unit
- Operation = HTTP Method + Path / RAML fragment / Mule Flow unit

## Source of Truth

| Topic | Source |
|---|---|
| Application, API, Operation list | `design-index.yaml` |
| API request / response contract | RAML |
| Basic design | `docs/application-basic-design.md` |
| Flow and Processor design | `docs/application-detail-design.md` |
| Mule implementation | `src/main/mule/`, `src/main/resources/dwl/` |
| Unit test implementation | `src/test/munit/` |

## Path Composition Rule

Use this rule when validating paths:

```text
full API path = api.basePath + operation.path
```

- `api.basePath` must match the path component of the RAML root `baseUri`.
- `operation.path` must match the RAML resource path for the Operation.
- Do not duplicate the same resource segment in both `api.basePath` and `operation.path`.

Example:

| Field | Value |
|---|---|
| RAML `baseUri` | `https://api.example.com/api/v1` |
| `api.basePath` | `/api/v1` |
| `operation.path` | `/customers/{customerId}` |
| Full path | `/api/v1/customers/{customerId}` |

## Change Rules

When changing an Operation RAML fragment, also check:

- `design-index.yaml`
- RAML root file
- `docs/application-basic-design.md`
- `docs/application-detail-design.md`
- DataWeave mapping or implementation
- MUnit test design or implementation

Do not change `operationId` unless explicitly requested.

Do not infer new API behavior from implementation alone.

If RAML and implementation conflict, report the conflict instead of silently resolving it.

## Naming Rules

| Asset | Naming Rule | Example |
|---|---|---|
| API ID | `{domain}-api-v{version}` | `sample-customer-api-v1` |
| Operation ID | `{apiId}.{resource}.{action}` | `sample-customer-api-v1.customer.getById` |
| Operation RAML | kebab-case | `customer-get-by-id.raml` |
| Flow | kebab-case + `-flow` | `get-customer-by-id-flow` |
| DataWeave | operation-based | `customer-get-by-id-response.dwl` |
| MUnit | operation + scenario | `customer-get-by-id-success-test` |

## AI Review Checklist

For each Operation, verify:

- Operation exists in `design-index.yaml`
- Operation RAML fragment exists
- RAML root references the Operation fragment
- `api.basePath + operation.path` matches the RAML contract route
- Flow is defined in detail design
- Error responses are consistent with RAML
- Mapping/DataWeave is defined where needed
- MUnit scenarios cover normal and major error cases

## Prohibited Actions

- Do not rename Operation IDs without explicit instruction.
- Do not treat RAML changes as minor edits when they change API contract.
- Do not resolve contradictions silently; report them.
- Do not copy sample values into production design without replacing them.
- Do not add new APIs outside `apis[]` in `design-index.yaml`.
