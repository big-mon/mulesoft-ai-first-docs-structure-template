# MuleSoft AI-first Docs Structure Template

This repository is a template for MuleSoft application design assets that are readable by both humans and AI agents.

The goal is not to increase the number of documents. The goal is to make the responsibilities and traceability of design assets explicit.

## Core Concept

This template uses the following hierarchy.

```text
Application
  └─ API
       └─ Operation
```

| Layer | Meaning | Main Responsibility |
|---|---|---|
| Application | Mule app / jar / repository / deployment unit | Runtime unit, shared flows, shared settings, external connections |
| API | RAML root / API Manager / APIkit router unit | API contract, API policies, consumers, base path |
| Operation | HTTP method + path / RAML fragment / Mule flow unit | Request/response, flow design, mapping, error handling, MUnit |

## Repository Structure

| Path | Responsibility |
|---|---|
| `README.md` | Entry point for humans and AI agents |
| `AGENTS.md` | AI agent instructions, source-of-truth rules, review checklist |
| `design-index.yaml` | Traceability map across Application, API, Operation, RAML, Flow, DataWeave, and MUnit |
| `docs/application-basic-design.md` | Basic design baseline |
| `docs/application-detail-design.md` | Detail design for Mule implementation |
| `raml/` | API contract specifications |
| `raml/common/` | Shared RAML types, traits, and examples |
| `src/main/mule/` | Mule XML implementation |
| `src/main/resources/dwl/` | DataWeave implementation |
| `src/main/resources/properties/` | Environment properties placeholder |
| `src/test/munit/` | MUnit tests |
| `prompts/` | Reusable AI review prompts |

## Reading Order

1. `README.md`
2. `AGENTS.md`
3. `design-index.yaml`
4. `docs/application-basic-design.md`
5. RAML root files and Operation fragments
6. `docs/application-detail-design.md`
7. Mule implementation under `src/`

## Source of Truth

| Information | Source of Truth |
|---|---|
| Application / API / Operation mapping | `design-index.yaml` |
| API request/response contract | RAML |
| Basic design decisions | `docs/application-basic-design.md` |
| Flow / Processor / Error Handler design | `docs/application-detail-design.md` |
| Mule implementation | `src/main/mule/` |
| DataWeave implementation | `src/main/resources/dwl/` |
| Unit test implementation | `src/test/munit/` |

## Phase-based Update Rules

| Phase | Main Update Targets | Purpose |
|---|---|---|
| Requirements | `design-index.yaml` | Identify Application, API, and Operation candidates |
| Basic Design | `docs/application-basic-design.md`, `raml/**`, `design-index.yaml` | Define API contract, responsibilities, API management, sequence, and error policy |
| Detail Design | `docs/application-detail-design.md`, `design-index.yaml` | Define Mule flows, processors, connectors, DataWeave, error handlers, and MUnit scenarios |
| Implementation | `src/main/mule/**`, `src/main/resources/dwl/**`, `src/test/munit/**` | Implement Mule app and tests |
| Change Management | Affected RAML, docs, index, implementation, tests | Track impact by Operation ID |

## Important Rule

RAML is created during basic design and treated as the API contract baseline. If RAML changes during detail design or implementation, treat it as an API contract change and update related design, mapping, and test assets.

## How to Start

1. Copy this template repository.
2. Replace the sample application and API IDs in `design-index.yaml`.
3. Create or update RAML root and Operation fragments under `raml/{api-id}/v1/`.
4. Fill `docs/application-basic-design.md` during basic design.
5. Fill `docs/application-detail-design.md` during detail design.
6. Use prompts under `prompts/` for AI-assisted review.
