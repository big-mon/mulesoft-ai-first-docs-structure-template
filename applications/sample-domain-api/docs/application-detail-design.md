# アプリケーション詳細設計

このドキュメントは、1つのMuleアプリケーションに対する実装入力です。

## 1. アプリケーション詳細

| 項目 | 値 |
|---|---|
| Mule Application | `sample-domain-api` |
| Artifact | `sample-domain-api.jar` |
| Runtime | Mule 4.x |
| Java | To be defined |
| APIkit | 使用する |

## 2. 共通Flow設計

| Flow / Subflow | 責務 | 備考 |
|---|---|---|
| `common-correlation-id-subflow` | Correlation IDを解決または生成する | すべてのAPI entry flowで使用する |
| `common-error-response-subflow` | 共通エラーレスポンスを生成する | `ErrorResponse` モデルを使用する |
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
| Root RAML | `apis/sample-customer-api-v1/raml/v1/sample-customer-api.raml` |
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
| RAML | `apis/sample-customer-api-v1/operations/customer-get-by-id/customer-get-by-id.raml` |
| Flow | `get-customer-by-id-flow` |
| Response Type | `Customer` |

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
| 2 | Validation | `customerId` を検証する | `attributes.uriParams.customerId` | - | `VALIDATION:*` |
| 3 | HTTP Request | 顧客システムを呼び出す | customerId | 接続先レスポンス | `HTTP:*` |
| 4 | Transform Message | APIレスポンスを生成する | 接続先レスポンス | `Customer` | `EXPRESSION` |
| 5 | Logger | 終了ログを出力する | status, elapsed time | - | - |

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

### 4.2 Sample Address API v1

| 項目 | 値 |
|---|---|
| API ID | `sample-address-api-v1` |
| Root RAML | `apis/sample-address-api-v1/raml/v1/sample-address-api.raml` |
| Entry Flow | `sample-address-api-main-flow` |
| APIkit Config | `sample-address-api-config` |
| Autodiscovery Flow Ref | `sample-address-api-main-flow` |

#### 4.2.1 Operation - Flow対応表

| Operation ID | Flow | DataWeave | MUnit |
|---|---|---|---|
| `sample-address-api-v1.address.getByCustomerId` | `get-address-by-customer-id-flow` | `address-get-by-customer-id-response.dwl` | `address-get-by-customer-id-success-test`, `address-get-by-customer-id-not-found-test` |

## 5. Connector・Properties

| Property | 例 | 備考 |
|---|---|---|
| `api.sampleCustomer.instanceId` | `${api.sampleCustomer.instanceId}` | 環境ごとのAPI Manager instance ID |
| `api.sampleAddress.instanceId` | `${api.sampleAddress.instanceId}` | 環境ごとのAPI Manager instance ID |
| `downstream.host` | `${downstream.host}` | 外部システムのhost |

## 6. DataWeave・マッピング詳細

| Operation ID | DWL | 入力 | 出力 | 備考 |
|---|---|---|---|---|
| `sample-customer-api-v1.customer.getById` | `customer-get-by-id-response.dwl` | 接続先の顧客レスポンス | `Customer` | 接続先項目をAPI typeへマッピングする |
| `sample-customer-api-v1.customer.search` | `customer-search-request.dwl` | API検索リクエスト | 接続先検索リクエスト | 検索条件を正規化する |
| `sample-customer-api-v1.customer.search` | `customer-search-response.dwl` | 接続先検索レスポンス | `CustomerSearchResponse` | 一覧結果をマッピングする |
| `sample-address-api-v1.address.getByCustomerId` | `address-get-by-customer-id-response.dwl` | 接続先住所レスポンス | `AddressList` | 一覧結果をマッピングする |

## 7. MUnitテスト設計

Operation単位で、正常系と主要異常系のシナリオを定義します。

| Operation ID | 必須シナリオ |
|---|---|
| `sample-customer-api-v1.customer.getById` | success, validation error, not found, timeout |
| `sample-customer-api-v1.customer.search` | success, validation error, system error |
| `sample-address-api-v1.address.getByCustomerId` | success, not found |
