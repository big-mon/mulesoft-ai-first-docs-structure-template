# アプリケーション基本設計

このドキュメントは、1つのMuleアプリケーションに対する基本設計のベースラインです。

## 1. アプリケーション概要

### 1.1 アプリケーション責務

| 項目 | 値 |
|---|---|
| Application ID | `sample-domain-sapi-v1` |
| Mule Application | `sample-domain-sapi-v1` |
| 責務 | 顧客ドメインに関するAPIを提供し、利用者からのリクエストを接続先システム向けに中継・変換する。 |
| Runtime | Mule 4.9.0 |
| Java | 17 |

### 1.2 jar / repo / deployment単位

| 項目 | 値 |
|---|---|
| Application Root | `applications/sample-domain-sapi-v1` |
| Artifact | `sample-domain-sapi-v1.jar` |
| Deployment Unit | jar |
| POM | `pom.xml` |
| Design Index | `design-index.yaml` |

### 1.3 外部接続先一覧

| 接続先 | 種別 | 用途 | 備考 |
|---|---|---|---|
| Customer System | HTTP | 顧客情報の参照・検索 | 詳細なhost、port、basePathは環境別プロパティで管理する。 |

### 1.4 共通処理方針

| トピック | 方針 | 関連資産 |
|---|---|---|
| Correlation ID | リクエスト単位の追跡IDを解決または生成する。 | `raml/common/traits/correlation-id.raml` |
| Client ID enforcement | API Manager policyでAPI単位に必要な認証ヘッダーをルーティング前に検証する。Mule Flow内では `client_id` / `client_secret` を手動Validationしない。 | `raml/common/traits/client-id-required.raml` |
| Error response | 共通エラーモデルでレスポンスを返却する。 | `raml/common/types/ErrorResponse.raml`, `raml/common/traits/common-errors.raml` |
| Timeout | Connector単位で詳細設計時に定義する。 | `application-detail-design.md` |
| Retry | Operation単位で詳細設計時に定義する。 | `application-detail-design.md` |
| Logging | チーム標準に従う。Operation固有のログ出力ポイントは詳細設計で定義する。 | `application-detail-design.md` |
| Data masking | 明示的に許可されていない限り、PII項目をログ出力しない。 | `application-detail-design.md` |

#### 共通Request Header

| Header | Required | 入力規則 | 適用方針 |
|---|---|---|---|
| `client_id` | true | `^[A-Za-z0-9_-]{16,64}$` | API Managerが `sample-customer-api-v1` の全Operationで検証する |
| `client_secret` | true | 32文字以上 | API Managerが `sample-customer-api-v1` の全Operationで検証する |

## 2. API一覧

### 2.1 Customer API v1

| API ID | API名 | Version | Root RAML | Base Path | API Manager |
|---|---|---|---|---|---|
| `sample-customer-api-v1` | Sample Customer API | v1 | `raml/sample-customer-api/v1/sample-customer-api.raml` | `/api/v1` | Managed |

## 3. API別基本設計

### 3.1 Customer API v1

| 項目 | 値 |
|---|---|
| API責務 | 顧客情報の参照・検索Operationを提供する。 |
| root RAML | `raml/sample-customer-api/v1/sample-customer-api.raml` |
| basePath | `/api/v1` |
| 利用者 | `sample-web-frontend` |
| API Manager方針 | Client ID enforcement、rate limitingを適用する。 |
| 共通エラーモデル | `ErrorResponse` |

#### 3.1.1 Operation一覧

| Operation ID | Method | Path | 概要 | RAML Fragment |
|---|---|---|---|---|
| `sample-customer-api-v1.customer.getById` | GET | `/customers/{customerId}` | 顧客IDを指定して顧客情報を取得する。 | `raml/sample-customer-api/v1/resources/get_customer-get-by-id.raml` |
| `sample-customer-api-v1.customer.search` | POST | `/customers/search` | 条件を指定して顧客情報を検索する。 | `raml/sample-customer-api/v1/resources/post_customer-search.raml` |

## 4. Operation別概要

| Operation ID | 概要 | 入力概念 | 出力概念 | 主な共通方針 |
|---|---|---|---|---|
| `sample-customer-api-v1.customer.getById` | 顧客IDを指定して顧客情報を取得する。 | 顧客ID | 顧客情報 | Correlation ID、Client ID enforcement、共通エラー |
| `sample-customer-api-v1.customer.search` | 条件を指定して顧客情報を検索する。 | 検索条件 | 検索結果 | Correlation ID、Client ID enforcement、共通エラー |

## 5. シーケンス概要

Operation単位で、正常系と主要な異常系の処理順序を記載します。

| Operation ID | 正常系概要 |
|---|---|
| `sample-customer-api-v1.customer.getById` | 入力チェック -> 顧客システム呼び出し -> 顧客情報へレスポンス変換 -> APIレスポンス返却 |
| `sample-customer-api-v1.customer.search` | 入力チェック -> 検索条件の正規化 -> 顧客システム呼び出し -> 検索結果へレスポンス変換 -> APIレスポンス返却 |

## 6. データモデル・マッピング概要

RAML typeをAPI契約の正本とします。詳細な項目マッピングは詳細設計で定義します。

| API Type | Source / Target | RAML | 備考 |
|---|---|---|---|
| `Customer` | Customer System customer response | `raml/sample-customer-api/v1/types/Customer.raml` | 顧客の基本情報 |
| `CustomerSearchRequest` | API request | `raml/sample-customer-api/v1/types/CustomerSearchRequest.raml` | 検索条件 |
| `CustomerSearchResponse` | Customer System search response | `raml/sample-customer-api/v1/types/CustomerSearchResponse.raml` | 検索結果 |
| `ErrorResponse` | API error response | `raml/common/types/ErrorResponse.raml` | アプリケーション内共通エラーモデル |

## 7. エラー定義

| HTTP Status | Error Code | 区分 | RAML Trait | 備考 |
|---:|---|---|---|---|
| 400 | `BAD_REQUEST` | Client error | `common-errors` | 入力値不正 |
| 404 | `RESOURCE_NOT_FOUND` | Business error | `common-errors` | 指定されたリソースが存在しない |
| 500 | `INTERNAL_ERROR` | System error | `common-errors` | 想定外の内部エラー |
| 503 | `SERVICE_UNAVAILABLE` | System error | `common-errors` | 接続先システムが利用不可 |
| 504 | `GATEWAY_TIMEOUT` | System error | `common-errors` | 接続先システムのタイムアウト |

## 8. API管理設計

| API ID | Autodiscovery Property | Flow Ref | Policies | Consumers |
|---|---|---|---|---|
| `sample-customer-api-v1` | `api.sampleCustomer.instanceId` | `sample-customer-api-main-flow` | Client ID enforcement, rate limiting | `sample-web-frontend` |

## 9. 設計トレーサビリティ

| 対象 | 正本 | 関連設計 | 実装・テスト |
|---|---|---|---|
| Application一覧 | `applications/` 直下のディレクトリ | `applications/sample-domain-sapi-v1/design-index.yaml` | `applications/sample-domain-sapi-v1/pom.xml` |
| API一覧 | `applications/sample-domain-sapi-v1/design-index.yaml` | `raml/sample-customer-api/v1/sample-customer-api.raml` | `sample-customer-api-main-flow` |
| `sample-customer-api-v1.customer.getById` | `raml/sample-customer-api/v1/resources/get_customer-get-by-id.raml` | `application-detail-design.md`, `operations/sample-customer-api-v1/get_customer-get-by-id/operation-detail-design.md` | `get-customer-by-id-flow`, `customer-get-by-id-response.dwl`, `customer-get-by-id-*` MUnit |
| `sample-customer-api-v1.customer.search` | `raml/sample-customer-api/v1/resources/post_customer-search.raml` | `application-detail-design.md`, `operations/sample-customer-api-v1/post_customer-search/operation-detail-design.md` | `search-customers-flow`, `customer-search-request.dwl`, `customer-search-response.dwl`, `customer-search-*` MUnit |
