# RAML Structure

RAML is the source of truth for API request and response contracts.

Each API has a root RAML file. Each Operation is defined as a RAML method fragment and included from the root RAML.

```text
raml/
  common/
    types/
    traits/
    examples/
  {api-id}/
    v1/
      {api-id}.raml
      resources/
      types/
      traits/
      examples/
```

## Rules

- Root RAML represents the API contract entry point.
- Operation RAML fragments represent method-level contracts.
- Common error and header definitions live under `raml/common/`.
- API-specific domain types should usually remain under each API directory.
- Do not duplicate contract details in design documents when RAML is the source of truth.
