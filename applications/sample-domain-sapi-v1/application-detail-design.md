# アプリケーション詳細設計

このドキュメントは、1つのMuleアプリケーションに対する実装入力です。

## 1. アプリケーション詳細

| 項目 | 値 |
|---|---|
| Mule Application | `sample-domain-sapi-v1` |
| Artifact | `sample-domain-sapi-v1.jar` |
| Runtime | Mule 4.9.0 |
| Java | 17 |
| APIkit | 使用する |

## 2. 共通Flow設計

| Flow / Subflow | 責務 | 備考 |
|---|---|---|
| `common-correlation-id-subflow` | Correlation IDを解決または生成する | すべてのAPI entry flowで使用する |
| `common-error-response-subflow` | 共通エラーレスポンスを生成する | `raml/common/types/ErrorResponse.raml` を使用する |
| `common-logging-subflow` | 標準ログを出力する | 機密情報をログ出力しない |

## 3. 共通Connector設定

| Config Name | 種別 | 目的 | Properties |
|---|---|---|---|
| `sample-http-request-config` | HTTP Request | 接続先システム呼び出し | `downstream.host`, `downstream.port`, `downstream.basePath` |

## 4. API別詳細設計

### 4.1 Sample Customer API v1

| 項目 | 値 |
|---|---|
| API ID | `sample-customer-api-v1` |
| Root RAML | `raml/sample-customer-api/v1/sample-customer-api.raml` |
| Entry Flow | `sample-customer-api-main-flow` |
| APIkit Config | `sample-customer-api-config` |
| Autodiscovery Flow Ref | `sample-customer-api-main-flow` |

#### 4.1.1 Operation - Flow対応表

| Operation ID | Flow | DataWeave | MUnit |
|---|---|---|---|
| `sample-customer-api-v1.customer.getById` | `get-customer-by-id-flow` | `customer-get-by-id-response.dwl` | `customer-get-by-id-success-test`, `customer-get-by-id-validation-error-test`, `customer-get-by-id-not-found-test`, `customer-get-by-id-timeout-test` |
| `sample-customer-api-v1.customer.search` | `search-customers-flow` | `customer-search-request.dwl`, `customer-search-response.dwl` | `customer-search-success-test`, `customer-search-validation-error-test`, `customer-search-system-error-test` |

#### 4.1.2 Operation詳細: `sample-customer-api-v1.customer.getById`

| 項目 | 値 |
|---|---|
| Method / Path | `GET /customers/{customerId}` |
| RAML | `raml/sample-customer-api/v1/resources/get_customer-get-by-id.raml` |
| Flow | `get-customer-by-id-flow` |
| Request Headers | `client_id`, `client_secret` |
| Response Type | `Customer` |

##### Request Header

| Header | Required | 入力規則 | 備考 |
|---|---|---|---|
| `client_id` | true | `^[A-Za-z0-9_-]{16,64}$` | Client ID enforcementで検証する |
| `client_secret` | true | 32文字以上 | Client ID enforcementで検証する |

##### Flow図

```text
APIkit Router
  -> customerIdを検証する
  -> 接続先の顧客サービスを呼び出す
  -> 接続先レスポンスをCustomerへ変換する
  -> 200レスポンスを返却する
```

##### Processor明細

| No | Processor | 目的 | 入力 | 出力 | Error |
|---:|---|---|---|---|---|
| 1 | Logger | 開始ログを出力する | headers, path params | - | - |
| 2 | Validation | 必須request headerを検証する | `attributes.headers.client_id`, `attributes.headers.client_secret` | - | `VALIDATION:*` |
| 3 | Validation | `customerId` を検証する | `attributes.uriParams.customerId` | - | `VALIDATION:*` |
| 4 | HTTP Request | 顧客システムを呼び出す | customerId | 接続先レスポンス | `HTTP:*` |
| 5 | Transform Message | APIレスポンスを生成する | 接続先レスポンス | `Customer` | `EXPRESSION` |
| 6 | Logger | 終了ログを出力する | status, elapsed time | - | - |

##### Error Handling

| Error Type | Handling | HTTP Status | Error Code |
|---|---|---:|---|
| `VALIDATION:*` | On Error Continue | 400 | `BAD_REQUEST` |
| `HTTP:NOT_FOUND` | On Error Continue | 404 | `RESOURCE_NOT_FOUND` |
| `HTTP:TIMEOUT` | On Error Propagate | 504 | `GATEWAY_TIMEOUT` |
| `ANY` | On Error Propagate | 500 | `INTERNAL_ERROR` |

##### MUnitシナリオ

| Test | 目的 | Mock | Assert |
|---|---|---|---|
| `customer-get-by-id-success-test` | 正常応答 | HTTP Request | status 200 と `Customer` payload |
| `customer-get-by-id-validation-error-test` | customer ID不正 | なし | status 400 |
| `customer-get-by-id-not-found-test` | 接続先で対象なし | HTTP Request | status 404 |
| `customer-get-by-id-timeout-test` | 接続先タイムアウト | HTTP Request | status 504 と `GATEWAY_TIMEOUT` エラーレスポンス |

#### 4.1.3 Operation詳細: `sample-customer-api-v1.customer.search`

4.1.2 と同じ構成で記載します。

## 5. Connector・Properties

| Property | 例 | 備考 |
|---|---|---|
| `api.sampleCustomer.instanceId` | `${api.sampleCustomer.instanceId}` | 環境ごとのAPI Manager instance ID |
| `downstream.host` | `${downstream.host}` | 外部システムのhost |

## 6. DataWeave・マッピング詳細

| Operation ID | DWL | 入力 | 出力 | 備考 |
|---|---|---|---|---|
| `sample-customer-api-v1.customer.getById` | `customer-get-by-id-response.dwl` | 接続先の顧客レスポンス | `Customer` | 接続先項目をAPI typeへマッピングする |
| `sample-customer-api-v1.customer.search` | `customer-search-request.dwl` | API検索リクエスト | 接続先検索リクエスト | 検索条件を正規化する |
| `sample-customer-api-v1.customer.search` | `customer-search-response.dwl` | 接続先検索レスポンス | `CustomerSearchResponse` | 一覧結果をマッピングする |

## 7. MUnitテスト設計

Operation単位で、正常系と主要異常系のシナリオを定義します。

| Operation ID | 必須シナリオ |
|---|---|
| `sample-customer-api-v1.customer.getById` | success, validation error, not found, timeout |
| `sample-customer-api-v1.customer.search` | success, validation error, system error |
