# AI Review Prompt: Operation Level

Use this prompt to review one Operation.

```text
Review operationId: {operationId}.

Read design-index.yaml first, then read the Operation RAML fragment, related RAML types/examples, and the corresponding Operation detail section.

Check whether:
- Method and path are consistent between design-index.yaml and RAML.
- Request body, response body, headers, and status codes are clear.
- Error responses in RAML and Error Handler design are consistent.
- Flow diagram and Processor details are sufficient for implementation.
- DataWeave mapping is defined where transformation is required.
- MUnit scenarios cover normal and major error cases.

Do not modify files. Report issues and suggested fixes.
```
