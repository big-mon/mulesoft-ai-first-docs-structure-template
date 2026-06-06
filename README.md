# MuleSoft AI-first設計ドキュメント構造テンプレート

このリポジトリは、MuleSoftアプリケーションの設計資産を、人間とAIエージェントの双方が読みやすい形で管理するためのテンプレートです。

目的は、ドキュメントの数を増やすことではありません。設計資産の責務とトレーサビリティを明確にし、API契約、Mule実装、DataWeave、MUnitを一貫して追跡できる状態にすることです。

## 基本コンセプト

このテンプレートでは、次の階層で設計資産を整理します。

```text
Application
  └─ API
       └─ Operation
```

| 階層 | 意味 | 主な責務 |
|---|---|---|
| Application | Muleアプリ / jar / repository / デプロイ単位 | 実行単位、共通Flow、共通設定、外部接続 |
| API | RAML root / API Manager / APIkit Router単位 | API契約、APIポリシー、利用者、base path |
| Operation | HTTP method + path / RAML fragment / Mule Flow単位 | 入出力、Flow設計、マッピング、エラー処理、MUnit |

## リポジトリ構成

| Path | 責務 |
|---|---|
| `README.md` | 人間とAIエージェント向けの入口 |
| `AGENTS.md` | AIエージェント向けの読み順、正本ルール、レビュー観点 |
| `design-index.yaml` | Application / API / Operation / RAML / Flow / DataWeave / MUnit の対応関係を示すトレーサビリティマップ |
| `docs/application-basic-design.md` | 基本設計のベースライン |
| `docs/application-detail-design.md` | Mule実装の入力となる詳細設計 |
| `raml/` | API契約仕様 |
| `raml/common/` | 共通RAML type、trait、example |
| `src/main/mule/` | Mule XML実装 |
| `src/main/resources/dwl/` | DataWeave実装 |
| `src/main/resources/properties/` | 環境別プロパティの配置先 |
| `src/test/munit/` | MUnitテスト |
| `prompts/` | AIレビュー用プロンプト |

## 読む順番

1. `README.md`
2. `AGENTS.md`
3. `design-index.yaml`
4. `docs/application-basic-design.md`
5. RAML rootファイルとOperation fragment
6. `docs/application-detail-design.md`
7. `src/` 配下のMule実装

## 正本ルール

| 情報 | 正本 |
|---|---|
| Application / API / Operation の対応関係 | `design-index.yaml` |
| APIのrequest/response契約 | RAML |
| 基本設計上の判断 | `docs/application-basic-design.md` |
| Flow / Processor / Error Handler設計 | `docs/application-detail-design.md` |
| Mule実装 | `src/main/mule/` |
| DataWeave実装 | `src/main/resources/dwl/` |
| 単体テスト実装 | `src/test/munit/` |

## パス合成ルール

`design-index.yaml` では、APIパスを次のルールで扱います。

```text
full API path = api.basePath + operation.path
```

- `api.basePath` はRAML rootの `baseUri` のパス部分と一致させます。
- `operation.path` は該当OperationのRAMLリソースパスと一致させます。
- 同じリソースセグメントを `api.basePath` と `operation.path` の両方に重複して書かないでください。

例:

| 項目 | 値 |
|---|---|
| RAML `baseUri` | `https://api.example.com/api/v1` |
| `api.basePath` | `/api/v1` |
| `operation.path` | `/customers/{customerId}` |
| Full path | `/api/v1/customers/{customerId}` |

## フェーズ別の更新ルール

| フェーズ | 主な更新対象 | 目的 |
|---|---|---|
| 要件定義 | `design-index.yaml` | Application、API、Operation候補を整理する |
| 基本設計 | `docs/application-basic-design.md`, `raml/**`, `design-index.yaml` | API契約、責務、API管理、シーケンス、エラー方針を定義する |
| 詳細設計 | `docs/application-detail-design.md`, `design-index.yaml` | Mule Flow、Processor、Connector、DataWeave、Error Handler、MUnit観点を定義する |
| 実装 | `src/main/mule/**`, `src/main/resources/dwl/**`, `src/test/munit/**` | Muleアプリとテストを実装する |
| 変更管理 | 影響するRAML、設計書、index、実装、テスト | Operation ID単位で影響を追跡する |

## 重要ルール

RAMLは基本設計で作成し、API契約のベースラインとして扱います。詳細設計または実装中にRAMLを変更する場合は、API契約変更として扱い、関連する設計、マッピング、テスト資産を更新してください。

## 使い始め方

1. このテンプレートリポジトリをコピーします。
2. `design-index.yaml` のサンプルApplication ID、API ID、Operation IDを実案件の値に置き換えます。
3. `raml/{api-id}/v1/` 配下にRAML rootとOperation fragmentを作成または更新します。
4. 基本設計では `docs/application-basic-design.md` を記載します。
5. 詳細設計では `docs/application-detail-design.md` を記載します。
6. AI支援レビューを行う場合は `prompts/` 配下のプロンプトを利用します。
