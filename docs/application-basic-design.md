# Application Basic Design

This document is the basic design baseline for one Mule application.

## 1. Application Overview

| Item | Value |
|---|---|
| Application ID | `sample-domain-api` |
| Mule Application | `sample-domain-api` |
| Artifact | `sample-domain-api.jar` |
| Deployment Unit | jar |
| Repository | `sample-domain-api` |
| Runtime | Mule 4.x |

## 2. Included APIs

| API ID | API Name | Version | Layer | Root RAML | Base Path | API Manager |
|---|---|---|---|---|---|---|
| `sample-customer-api-v1` | Sample Customer API | v1 | Experience | `raml/sample-customer-api/v1/sample-customer-api.raml` | `/api/v1/customers` | Managed |
| `sample-address-api-v1` | Sample Address API | v1 | Process | `raml/sample-address-api/v1/sample-address-api.raml` | `/api/v1/addresses` | Managed |

## 3. API Basic Design

### 3.1 Sample Customer API v1

| Item | Value |
|---|---|
| Responsibility | Provide customer reference and search operations. |
| Consumer | `sample-web-frontend` |
| Authentication | Client ID enforcement |
| Policies | Client ID enforcement, rate limiting |
| Error Model | `ErrorResponse` |

### 3.2 Sample Address API v1

| Item | Value |
|---|---|
| Responsibility | Provide customer address reference operations. |
| Consumer | `sample-web-frontend` |
| Authentication | Client ID enforcement |
| Policies | Client ID enforcement |
| Error Model | `ErrorResponse` |

## 4. Operation List

| Operation ID | Method | Path | Request Type | Response Type | RAML Fragment |
|---|---|---|---|---|---|
| `sample-customer-api-v1.customer.getById` | GET | `/customers/{customerId}` | - | `Customer` | `raml/sample-customer-api/v1/resources/customer-get-by-id.raml` |
| `sample-customer-api-v1.customer.search` | POST | `/customers/search` | `CustomerSearchRequest` | `CustomerSearchResponse` | `raml/sample-customer-api/v1/resources/customer-search.raml` |
| `sample-address-api-v1.address.getByCustomerId` | GET | `/customers/{customerId}/addresses` | - | `AddressList` | `raml/sample-address-api/v1/resources/address-get-by-customer-id.raml` |

## 5. Sequence Overview

Describe the normal and major error sequences at Operation level.

Recommended level:

```text
Validate request -> call target system -> transform response -> return API response
```

## 6. Data Model and Mapping Overview

RAML types define the API contract. Mapping details are completed during detail design.

| API Type | Source / Target | Notes |
|---|---|---|
| `Customer` | CRM customer response | Basic customer profile |
| `CustomerSearchRequest` | API request | Search conditions |
| `CustomerSearchResponse` | CRM search response | Search results |
| `AddressList` | Address system response | Customer address list |

## 7. Error Definition

| HTTP Status | Error Code | Category | Notes |
|---:|---|---|---|
| 400 | `BAD_REQUEST` | Client error | Invalid input |
| 404 | `RESOURCE_NOT_FOUND` | Business error | Requested resource was not found |
| 500 | `INTERNAL_ERROR` | System error | Unexpected internal error |
| 503 | `SERVICE_UNAVAILABLE` | System error | Downstream system unavailable |
| 504 | `GATEWAY_TIMEOUT` | System error | Downstream timeout |

## 8. API Management Design

| API ID | Autodiscovery Property | Flow Ref | Policies |
|---|---|---|---|
| `sample-customer-api-v1` | `api.sampleCustomer.instanceId` | `sample-customer-api-main-flow` | Client ID enforcement, rate limiting |
| `sample-address-api-v1` | `api.sampleAddress.instanceId` | `sample-address-api-main-flow` | Client ID enforcement |

## 9. Application-specific Constraints

Document only application-specific constraints here. Platform-wide operational standards should be referenced, not duplicated.

| Topic | Decision |
|---|---|
| Timeout | To be defined per connector in detail design |
| Retry | To be defined per Operation in detail design |
| Logging | Use team standard. Operation-specific log points are defined in detail design. |
| Data masking | Do not log PII fields unless explicitly approved. |
