# MuleSoft AI-first設計ドキュメント構造テンプレート

このリポジトリは、複数のMuleSoftアプリケーションの設計資産を、人間とAIエージェントの双方が追跡しやすい形で管理するためのテンプレートです。

目的は、ドキュメントの数を増やすことではありません。Application、API、Operationの責務とトレーサビリティを明確にし、API契約、Mule実装、DataWeave、MUnitを一貫して追跡できる状態にすることです。

## 基本コンセプト

このテンプレートでは、次の三層で設計資産を整理します。

```text
Repository
  └─ Application
       └─ API
            └─ Operation
```

| 階層 | 物理パス | 意味 | 主な責務 |
|---|---|---|---|
| Repository | `design-index.yaml` | 複数Muleアプリケーションを束ねる単位 | Application一覧、アプリケーションルートへの導線 |
| Application | `applications/{appId}/` | Muleアプリ / jar / デプロイ単位。`appId` は `sample-domain-api-v1` のようにバージョンを含める | アプリ設計、共通Flow、共通設定、外部接続、Mule実装 |
| API | `applications/{appId}/raml/{apiFolder}/{version}/` | RAML root / API Manager / APIkit Router単位。`apiFolder` は `sample-customer-api` のようにバージョンを重複させない | API契約、APIポリシー、利用者、base path |
| Operation | `applications/{appId}/raml/{apiFolder}/{version}/resources/{operationRaml}` | HTTP method + path / RAML method fragment / Mule Flow単位。`operationRaml` は `get_customer-get-by-id.raml` のようにHTTP methodとOperation名を `_` で区切る | 入出力、Flow設計、マッピング、エラー処理、MUnit |

## リポジトリ構成

```text
.
├─ design-index.yaml
├─ AGENTS.md
├─ prompts/
└─ applications/
   └─ sample-domain-api-v1/
      ├─ design-index.yaml
      ├─ application-basic-design.md
      ├─ application-detail-design.md
      ├─ pom.xml
      ├─ raml/
      │  └─ sample-customer-api/
      │     └─ v1/
      │        ├─ sample-customer-api.raml
      │        ├─ resources/
      │        ├─ types/
      │        ├─ traits/
      │        └─ examples/
      └─ src/
```

| Path | 責務 |
|---|---|
| `README.md` | 人間とAIエージェント向けの入口 |
| `AGENTS.md` | AIエージェント向けの読み順、正本ルール、レビュー観点 |
| `design-index.yaml` | リポジトリに含まれるApplication一覧 |
| `applications/{appId}/design-index.yaml` | Application / API / Operation / RAML / Flow / DataWeave / MUnit の対応関係 |
| `applications/{appId}/application-basic-design.md` | アプリケーション基本設計のベースライン |
| `applications/{appId}/application-detail-design.md` | Mule実装の入力となる詳細設計 |
| `applications/{appId}/raml/{apiFolder}/{version}/` | API root RAML、Operation fragment、type、trait、example |
| `applications/{appId}/src/main/mule/` | Mule XML実装 |
| `applications/{appId}/src/main/resources/dwl/` | DataWeave実装 |
| `applications/{appId}/src/test/munit/` | MUnitテスト |
| `prompts/` | AIレビュー用プロンプト |

## 読む順番

1. `README.md`
2. `AGENTS.md`
3. ルートの `design-index.yaml`
4. 対象アプリケーションの `applications/{appId}/design-index.yaml`
5. `applications/{appId}/application-basic-design.md`
6. 関連するAPI root RAML
7. 関連するOperation RAML fragment
8. `applications/{appId}/application-detail-design.md`
9. 実装が存在する場合は `applications/{appId}/src/` 配下のMule XML、DataWeave、MUnit

## 正本ルール

| 情報 | 正本 |
|---|---|
| リポジトリ内のApplication一覧 | ルートの `design-index.yaml` |
| Application / API / Operation の対応関係 | `applications/{appId}/design-index.yaml` |
| APIのrequest/response契約 | RAML |
| 基本設計上の判断 | `applications/{appId}/application-basic-design.md` |
| Flow / Processor / Error Handler設計 | `applications/{appId}/application-detail-design.md` |
| Mule実装 | `applications/{appId}/src/main/mule/` |
| DataWeave実装 | `applications/{appId}/src/main/resources/dwl/` |
| 単体テスト実装 | `applications/{appId}/src/test/munit/` |

## パス合成ルール

アプリケーション内の `design-index.yaml` では、APIパスを次のルールで扱います。

```text
full API path = api.basePath + operation.path
```

- `api.basePath` はRAML rootの `baseUri` のパス部分と一致させます。
- `operation.path` は該当OperationのRAMLリソースパスと一致させます。
- 同じリソースセグメントを `api.basePath` と `operation.path` の両方に重複して書かないでください。

## フェーズ別の更新ルール

| フェーズ | 主な更新対象 | 目的 |
|---|---|---|
| 要件定義 | ルート `design-index.yaml`, `applications/{appId}/design-index.yaml` | Application、API、Operation候補を整理する |
| 基本設計 | `applications/{appId}/application-basic-design.md`, `applications/{appId}/raml/**`, `applications/{appId}/design-index.yaml` | API契約、責務、API管理、シーケンス、エラー方針を定義する |
| 詳細設計 | `applications/{appId}/application-detail-design.md`, `applications/{appId}/design-index.yaml` | Mule Flow、Processor、Connector、DataWeave、Error Handler、MUnit観点を定義する |
| 実装 | `applications/{appId}/src/main/mule/**`, `applications/{appId}/src/main/resources/dwl/**`, `applications/{appId}/src/test/munit/**` | Muleアプリとテストを実装する |
| 変更管理 | 影響するRAML、設計書、index、実装、テスト | Operation ID単位で影響を追跡する |

## 使い始め方

1. このテンプレートリポジトリをコピーします。
2. 新しいMuleアプリケーションごとに `applications/{appId}/` を作成します。アプリケーションをバージョン単位で管理する場合は、`sample-domain-api-v1` のようにアプリケーションフォルダへバージョンを含めます。
3. ルートの `design-index.yaml` にApplicationを追加します。
4. `applications/{appId}/design-index.yaml` にAPIとOperationを定義します。API IDは `sample-customer-api-v1` のように契約識別子として保持し、APIフォルダは `sample-customer-api` のようにバージョンなしで定義します。
5. API契約は `applications/{appId}/raml/{apiFolder}/{version}/` に配置します。
6. 基本設計と詳細設計は `applications/{appId}/` 直下に記載します。
7. AI支援レビューを行う場合は `prompts/` 配下のプロンプトを利用します。
