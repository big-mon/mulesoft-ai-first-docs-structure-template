# アプリケーション詳細設計

このドキュメントは、1つのMuleアプリケーションに対する共通詳細設計と、Operation別詳細設計への導線です。

Operation別の詳細シーケンス図、Processor表、Flow詳細、DataWeave詳細、MUnit観点は `operations/{apiId}/{method}_{operationName}/operation-detail-design.md` に記載します。

## 1. アプリケーション詳細

| 項目 | 値 |
|---|---|
| Mule Application | `sample-domain-sapi-v1` |
| Artifact | `sample-domain-sapi-v1.jar` |
| Runtime | Mule 4.9.0 |
| Java | 17 |
| APIkit | 使用する |
| Design Index | `design-index.yaml` |
| Basic Design | `application-basic-design.md` |
| Operation Detail Design | `operations/{apiId}/{method}_{operationName}/operation-detail-design.md` |

## 2. 共通Flow設計

| Flow / Subflow | 責務 | 入力 | 出力 | 備考 |
|---|---|---|---|---|
| `common-correlation-id-subflow` | Correlation IDを解決または生成する | `attributes.headers.X-Correlation-ID` | `vars.correlationId` | すべてのAPI entry flowで使用する |
| `common-error-response-subflow` | 共通エラーレスポンスを生成する | error, `vars.correlationId` | `ErrorResponse` payload | `raml/common/types/ErrorResponse.raml` を使用する |
| `common-logging-subflow` | 標準ログを出力する | operationId, status, elapsed time | log event | 機密情報をログ出力しない |

## 3. 共通Connector設定

| Config Name | 種別 | 目的 | Properties | 使用Operation |
|---|---|---|---|---|
| `sample-http-request-config` | HTTP Request | Customer System呼び出し | `downstream.host`, `downstream.port`, `downstream.basePath` | `sample-customer-api-v1.customer.getById`, `sample-customer-api-v1.customer.search` |

## 4. API別詳細設計

### 4.1 Customer API v1

#### 4.1.1 APIkit Router / entry flow

| 項目 | 値 |
|---|---|
| API ID | `sample-customer-api-v1` |
| Root RAML | `raml/sample-customer-api/v1/sample-customer-api.raml` |
| APIkit Config | `sample-customer-api-config` |
| Entry Flow | `sample-customer-api-main-flow` |
| Autodiscovery Flow Ref | `sample-customer-api-main-flow` |
| Autodiscovery Property | `api.sampleCustomer.instanceId` |

| Flow | Processor | 目的 | 入力 | 出力 |
|---|---|---|---|---|
| `sample-customer-api-main-flow` | HTTP Listener | APIリクエストを受け付ける | HTTP request | Mule event |
| `sample-customer-api-main-flow` | APIkit Router | RAML契約に基づいてOperation flowへルーティングする | method, path, headers, body | target flow invocation |
| `sample-customer-api-main-flow` | Error Handler | 共通エラーハンドリングへ委譲する | error | `ErrorResponse` |

#### 4.1.2 Operation - Flow対応表

| Operation ID | Method / Path | Flow | DataWeave | MUnit | Operation Detail |
|---|---|---|---|---|---|
| `sample-customer-api-v1.customer.getById` | `GET /customers/{customerId}` | `get-customer-by-id-flow` | `customer-get-by-id-response.dwl` | `customer-get-by-id-success-test`, `customer-get-by-id-validation-error-test`, `customer-get-by-id-not-found-test`, `customer-get-by-id-timeout-test` | `operations/sample-customer-api-v1/get_customer-get-by-id/operation-detail-design.md` |
| `sample-customer-api-v1.customer.search` | `POST /customers/search` | `search-customers-flow` | `customer-search-request.dwl`, `customer-search-response.dwl` | `customer-search-success-test`, `customer-search-validation-error-test`, `customer-search-system-error-test` | `operations/sample-customer-api-v1/post_customer-search/operation-detail-design.md` |

#### 4.1.3 Operation別詳細設計

| Operation ID | Detail Design | 主な記載内容 |
|---|---|---|
| `sample-customer-api-v1.customer.getById` | `operations/sample-customer-api-v1/get_customer-get-by-id/operation-detail-design.md` | 詳細シーケンス、Flow詳細、Processor表、DataWeave、Connector呼び出し、Error処理、MUnit観点 |
| `sample-customer-api-v1.customer.search` | `operations/sample-customer-api-v1/post_customer-search/operation-detail-design.md` | 詳細シーケンス、Flow詳細、Processor表、DataWeave、Connector呼び出し、Error処理、MUnit観点 |

## 5. Error Handler詳細

| Error Type | Handling | HTTP Status | Error Code | Response Model | 備考 |
|---|---|---:|---|---|---|
| `VALIDATION:*` | On Error Continue | 400 | `BAD_REQUEST` | `ErrorResponse` | Header、path parameter、request bodyの入力値不正 |
| `HTTP:NOT_FOUND` | On Error Continue | 404 | `RESOURCE_NOT_FOUND` | `ErrorResponse` | 接続先で対象リソースが存在しない |
| `HTTP:TIMEOUT` | On Error Propagate | 504 | `GATEWAY_TIMEOUT` | `ErrorResponse` | 接続先タイムアウト |
| `HTTP:CONNECTIVITY` | On Error Propagate | 503 | `SERVICE_UNAVAILABLE` | `ErrorResponse` | 接続先サービス利用不可 |
| `ANY` | On Error Propagate | 500 | `INTERNAL_ERROR` | `ErrorResponse` | 想定外の内部エラー |

| 共通資産 | パス |
|---|---|
| ErrorResponse type | `raml/common/types/ErrorResponse.raml` |
| Common errors trait | `raml/common/traits/common-errors.raml` |
| Error example | `raml/common/examples/error-response.json` |

## 6. DataWeave・Connector・設定一覧

### 6.1 DataWeave

| Operation ID | DWL | 入力 | 出力 | 備考 |
|---|---|---|---|---|
| `sample-customer-api-v1.customer.getById` | `src/main/resources/dwl/customer-get-by-id-response.dwl` | downstream customer response | `Customer` | 接続先項目をAPI typeへマッピングする |
| `sample-customer-api-v1.customer.search` | `src/main/resources/dwl/customer-search-request.dwl` | `CustomerSearchRequest` | downstream search request | 検索条件を正規化する |
| `sample-customer-api-v1.customer.search` | `src/main/resources/dwl/customer-search-response.dwl` | downstream search response | `CustomerSearchResponse` | 一覧結果をマッピングする |

### 6.2 Connector

| Config Name | Connector | 使用Flow | 設定値 |
|---|---|---|---|
| `sample-http-request-config` | HTTP Request | `get-customer-by-id-flow`, `search-customers-flow` | `downstream.host`, `downstream.port`, `downstream.basePath` |

### 6.3 Properties

| Property | 例 | 用途 |
|---|---|---|
| `api.sampleCustomer.instanceId` | `${api.sampleCustomer.instanceId}` | API Manager instance ID |
| `downstream.host` | `${downstream.host}` | Customer System host |
| `downstream.port` | `${downstream.port}` | Customer System port |
| `downstream.basePath` | `${downstream.basePath}` | Customer System base path |

## 7. MUnitテスト設計

| Operation ID | 主なシナリオ | Operation Detail |
|---|---|---|
| `sample-customer-api-v1.customer.getById` | 正常応答、入力値不正、対象なし、接続先タイムアウト | `operations/sample-customer-api-v1/get_customer-get-by-id/operation-detail-design.md` |
| `sample-customer-api-v1.customer.search` | 正常応答、request body不正、接続先または内部エラー | `operations/sample-customer-api-v1/post_customer-search/operation-detail-design.md` |
