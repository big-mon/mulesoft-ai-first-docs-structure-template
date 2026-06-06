# Application Detail Design

This document is the implementation input for one Mule application.

## 1. Application Detail

| Item | Value |
|---|---|
| Mule Application | `sample-domain-api` |
| Artifact | `sample-domain-api.jar` |
| Runtime | Mule 4.x |
| Java | To be defined |
| APIkit | Used |

## 2. Common Flow Design

| Flow / Subflow | Responsibility | Notes |
|---|---|---|
| `common-correlation-id-subflow` | Resolve or generate correlation ID | Used by all API entry flows |
| `common-error-response-subflow` | Build common error response | Uses `ErrorResponse` model |
| `common-logging-subflow` | Standard logging | Do not log sensitive values |

## 3. Common Connector Settings

| Config Name | Type | Purpose | Properties |
|---|---|---|---|
| `sample-http-request-config` | HTTP Request | Downstream system call | `downstream.host`, `downstream.port`, `downstream.basePath` |

## 4. API Detail Design

### 4.1 Sample Customer API v1

| Item | Value |
|---|---|
| API ID | `sample-customer-api-v1` |
| Root RAML | `raml/sample-customer-api/v1/sample-customer-api.raml` |
| Entry Flow | `sample-customer-api-main-flow` |
| APIkit Config | `sample-customer-api-config` |
| Autodiscovery Flow Ref | `sample-customer-api-main-flow` |

#### 4.1.1 Operation - Flow Mapping

| Operation ID | Flow | DataWeave | MUnit |
|---|---|---|---|
| `sample-customer-api-v1.customer.getById` | `get-customer-by-id-flow` | `customer-get-by-id-response.dwl` | `customer-get-by-id-success-test` |
| `sample-customer-api-v1.customer.search` | `search-customers-flow` | `customer-search-request.dwl`, `customer-search-response.dwl` | `customer-search-success-test` |

#### 4.1.2 Operation Detail: `sample-customer-api-v1.customer.getById`

| Item | Value |
|---|---|
| Method / Path | `GET /customers/{customerId}` |
| RAML | `raml/sample-customer-api/v1/resources/customer-get-by-id.raml` |
| Flow | `get-customer-by-id-flow` |
| Response Type | `Customer` |

##### Flow Diagram

```text
APIkit Router
  -> validate customerId
  -> call downstream customer service
  -> transform downstream response to Customer
  -> return 200 response
```

##### Processor Details

| No | Processor | Purpose | Input | Output | Error |
|---:|---|---|---|---|---|
| 1 | Logger | Start log | headers, path params | - | - |
| 2 | Validation | Validate `customerId` | `attributes.uriParams.customerId` | - | `VALIDATION:*` |
| 3 | HTTP Request | Call customer system | customerId | downstream response | `HTTP:*` |
| 4 | Transform Message | Build API response | downstream response | `Customer` | `EXPRESSION` |
| 5 | Logger | End log | status, elapsed time | - | - |

##### Error Handling

| Error Type | Handling | HTTP Status | Error Code |
|---|---|---:|---|
| `VALIDATION:*` | On Error Continue | 400 | `BAD_REQUEST` |
| `HTTP:NOT_FOUND` | On Error Continue | 404 | `RESOURCE_NOT_FOUND` |
| `HTTP:TIMEOUT` | On Error Propagate | 504 | `GATEWAY_TIMEOUT` |
| `ANY` | On Error Propagate | 500 | `INTERNAL_ERROR` |

##### MUnit Scenarios

| Test | Purpose | Mock | Assert |
|---|---|---|---|
| `customer-get-by-id-success-test` | Normal response | HTTP Request | status 200 and `Customer` payload |
| `customer-get-by-id-validation-error-test` | Invalid customer ID | none | status 400 |
| `customer-get-by-id-not-found-test` | Downstream not found | HTTP Request | status 404 |

#### 4.1.3 Operation Detail: `sample-customer-api-v1.customer.search`

Use the same structure as 4.1.2.

### 4.2 Sample Address API v1

| Item | Value |
|---|---|
| API ID | `sample-address-api-v1` |
| Root RAML | `raml/sample-address-api/v1/sample-address-api.raml` |
| Entry Flow | `sample-address-api-main-flow` |
| APIkit Config | `sample-address-api-config` |
| Autodiscovery Flow Ref | `sample-address-api-main-flow` |

#### 4.2.1 Operation - Flow Mapping

| Operation ID | Flow | DataWeave | MUnit |
|---|---|---|---|
| `sample-address-api-v1.address.getByCustomerId` | `get-address-by-customer-id-flow` | `address-get-by-customer-id-response.dwl` | `address-get-by-customer-id-success-test` |

## 5. Connector and Properties

| Property | Example | Notes |
|---|---|---|
| `api.sampleCustomer.instanceId` | `${api.sampleCustomer.instanceId}` | API Manager instance ID per environment |
| `api.sampleAddress.instanceId` | `${api.sampleAddress.instanceId}` | API Manager instance ID per environment |
| `downstream.host` | `${downstream.host}` | External system host |

## 6. DataWeave and Mapping Detail

| Operation ID | DWL | Input | Output | Notes |
|---|---|---|---|---|
| `sample-customer-api-v1.customer.getById` | `customer-get-by-id-response.dwl` | Downstream customer response | `Customer` | Map downstream fields to API type |
| `sample-customer-api-v1.customer.search` | `customer-search-request.dwl` | API search request | Downstream search request | Normalize search conditions |
| `sample-customer-api-v1.customer.search` | `customer-search-response.dwl` | Downstream search response | `CustomerSearchResponse` | Map list results |
| `sample-address-api-v1.address.getByCustomerId` | `address-get-by-customer-id-response.dwl` | Downstream address response | `AddressList` | Map list results |

## 7. MUnit Test Design

Define normal and major error scenarios per Operation.

| Operation ID | Required Scenarios |
|---|---|
| `sample-customer-api-v1.customer.getById` | success, validation error, not found, timeout |
| `sample-customer-api-v1.customer.search` | success, validation error, system error |
| `sample-address-api-v1.address.getByCustomerId` | success, not found |
