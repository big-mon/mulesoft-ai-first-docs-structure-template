# AI Review Prompt: Application Level

Use this prompt to review the whole Mule application design.

```text
Read README.md, AGENTS.md, and design-index.yaml first.
Then review all APIs and Operations defined under apis[].

Check whether:
- Every API has a root RAML file.
- Every Operation has an Operation RAML fragment.
- Every Operation has a unique operationId.
- RAML, basic design, detail design, Flow, DataWeave, and MUnit references are traceable.
- API Manager and Autodiscovery settings are defined per API where required.
- No Operation exists only in one artifact without corresponding index entry.

Report inconsistencies by Operation ID.
```
