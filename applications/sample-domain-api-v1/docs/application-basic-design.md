# アプリケーション基本設計

このドキュメントは、1つのMuleアプリケーションに対する基本設計のベースラインです。

## 1. アプリケーション概要

| 項目 | 値 |
|---|---|
| Application ID | `sample-domain-api-v1` |
| Mule Application | `sample-domain-api-v1` |
| Artifact | `sample-domain-api-v1.jar` |
| Deployment Unit | jar |
| Repository | `sample-domain-api-v1` |
| Runtime | Mule 4.x |

## 2. 含まれるAPI

| API ID | API名 | Version | Layer | Root RAML | Base Path | API Manager |
|---|---|---|---|---|---|---|
| `sample-customer-api-v1` | Sample Customer API | v1 | Experience | `apis/sample-customer-api/raml/sample-customer-api.raml` | `/api/v1` | Managed |

`Base Path` はRAMLの `baseUri` のパス部分を表します。Operationのパスは別に定義し、RAML上のリソースパスを重複して含めないでください。

## 3. API別基本設計

### 3.1 Sample Customer API v1

| 項目 | 値 |
|---|---|
| 責務 | 顧客情報の参照・検索Operationを提供する。 |
| 利用者 | `sample-web-frontend` |
| 認証 | Client ID enforcement |
| Policy | Client ID enforcement, rate limiting |
| エラーモデル | `ErrorResponse` |

## 4. Operation一覧

| Operation ID | Method | Path | Request Headers | Request Type | Response Type | RAML Fragment |
|---|---|---|---|---|---|---|
| `sample-customer-api-v1.customer.getById` | GET | `/customers/{customerId}` | `client_id`, `client_secret` | - | `Customer` | `apis/sample-customer-api/operations/get-customer-by-id/get-customer-by-id.raml` |
| `sample-customer-api-v1.customer.search` | POST | `/customers/search` | - | `CustomerSearchRequest` | `CustomerSearchResponse` | `apis/sample-customer-api/operations/post-customer-search/post-customer-search.raml` |

### 4.1 Request Header定義

| Operation ID | Header | Required | 入力規則 |
|---|---|---|---|
| `sample-customer-api-v1.customer.getById` | `client_id` | true | `^[A-Za-z0-9_-]{16,64}$` |
| `sample-customer-api-v1.customer.getById` | `client_secret` | true | 32文字以上 |

## 5. シーケンス概要

Operation単位で、正常系と主要な異常系の処理順序を記載します。

推奨する粒度:

```text
入力チェック -> 接続先システム呼び出し -> レスポンス変換 -> APIレスポンス返却
```

## 6. データモデル・マッピング概要

RAML typeをAPI契約の正本とします。詳細な項目マッピングは詳細設計で定義します。

| API Type | Source / Target | 備考 |
|---|---|---|
| `Customer` | CRM customer response | 顧客の基本情報 |
| `CustomerSearchRequest` | API request | 検索条件 |
| `CustomerSearchResponse` | CRM search response | 検索結果 |

## 7. エラー定義

| HTTP Status | Error Code | 区分 | 備考 |
|---:|---|---|---|
| 400 | `BAD_REQUEST` | Client error | 入力値不正 |
| 404 | `RESOURCE_NOT_FOUND` | Business error | 指定されたリソースが存在しない |
| 500 | `INTERNAL_ERROR` | System error | 想定外の内部エラー |
| 503 | `SERVICE_UNAVAILABLE` | System error | 接続先システムが利用不可 |
| 504 | `GATEWAY_TIMEOUT` | System error | 接続先システムのタイムアウト |

## 8. API管理設計

| API ID | Autodiscovery Property | Flow Ref | Policies |
|---|---|---|---|
| `sample-customer-api-v1` | `api.sampleCustomer.instanceId` | `sample-customer-api-main-flow` | Client ID enforcement, rate limiting |

## 9. アプリケーション固有制約

ここにはアプリケーション固有の制約のみを記載します。プラットフォーム全体の運用標準は重複記載せず、必要に応じて参照してください。

| トピック | 方針 |
|---|---|
| Timeout | Connector単位で詳細設計時に定義する |
| Retry | Operation単位で詳細設計時に定義する |
| Logging | チーム標準に従う。Operation固有のログ出力ポイントは詳細設計で定義する |
| Data masking | 明示的に許可されていない限り、PII項目をログ出力しない |
