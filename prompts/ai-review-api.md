# AI Review Prompt: API Level

Use this prompt to review one API.

```text
Review the API identified by {apiId}.

Start from design-index.yaml.
Then read the API root RAML, all Operation RAML fragments, and related sections in the basic and detail design documents.

Check whether:
- The root RAML includes all Operations listed in design-index.yaml.
- Operation IDs in RAML match design-index.yaml.
- Request and response types are defined and referenced correctly.
- Common error responses are consistently applied.
- API Manager policies, consumers, and Autodiscovery flowRef are defined.
- Detail design has Operation-to-Flow mapping for all Operations.

Report findings grouped by Operation ID.
```
