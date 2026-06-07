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
| Repository | `design-index.yaml` | 複数Muleアプリケーションを束ねる単位 | Application探索ルール、構造ルール、パス規約 |
| Application | `applications/{appId}/` | Muleアプリ / jar / デプロイ単位。`appId` は `sample-domain-sapi-v1` のようにバージョンを含める | アプリ設計、共通Flow、共通設定、外部接続、Mule実装 |
| API | `applications/{appId}/raml/{apiFolder}/{version}/` | RAML root / API Manager / APIkit Router単位。`apiFolder` は `sample-customer-api` のようにバージョンを重複させない | API契約、APIポリシー、利用者、base path |
| Operation | `applications/{appId}/raml/{apiFolder}/{version}/resources/{operationRaml}`<br>`applications/{appId}/operations/{apiId}/{method}_{operationName}/operation-detail-design.md` | HTTP method + path / RAML method fragment / Mule Flow単位。`operationRaml` とOperation詳細設計フォルダは `get_customer-get-by-id` のようにHTTP methodとOperation名を `_` で区切る | API契約、Flow設計、Processor、詳細シーケンス、マッピング、エラー処理、MUnit |

## リポジトリ構成

```text
.
├─ design-index.yaml
├─ AGENTS.md
├─ docs/
│  └─ design-index-field-definition.md
├─ prompts/
└─ applications/
   ├─ README.md
   └─ sample-domain-sapi-v1/
      ├─ README.md
      ├─ design-index.yaml
      ├─ application-basic-design.md
      ├─ application-detail-design.md
      ├─ pom.xml
      ├─ operations/
      │  └─ sample-customer-api-v1/
      │     ├─ get_customer-get-by-id/
      │     │  └─ operation-detail-design.md
      │     └─ post_customer-search/
      │        └─ operation-detail-design.md
      ├─ raml/
      │  ├─ common/
      │  │  ├─ types/
      │  │  ├─ traits/
      │  │  └─ examples/
      │  └─ sample-customer-api/
      │     └─ v1/
      │        ├─ sample-customer-api.raml
      │        ├─ resources/
      │        ├─ types/
      │        └─ examples/
      └─ src/
```

| Path | 責務 |
|---|---|
| `README.md` | 人間とAIエージェント向けの入口 |
| `AGENTS.md` | AIエージェント向けの読み順、正本ルール、レビュー観点 |
| `design-index.yaml` | リポジトリ全体のApplication探索ルール、構造ルール、パス規約 |
| `docs/design-index-field-definition.md` | ルートおよびApplication design indexのフィールド定義 |
| `applications/README.md` | Applicationディレクトリの探索案内。仕様の正本ではない |
| `applications/{appId}/README.md` | 対象Application内の探索案内。仕様の正本ではない |
| `applications/{appId}/design-index.yaml` | Application / API / Operation / RAML / Flow / DataWeave / MUnit の対応関係 |
| `applications/{appId}/application-basic-design.md` | アプリケーション基本設計のベースライン |
| `applications/{appId}/application-detail-design.md` | アプリケーション共通の詳細設計とOperation詳細設計への導線 |
| `applications/{appId}/operations/{apiId}/{method}_{operationName}/operation-detail-design.md` | Operation別のFlow、Processor、詳細シーケンス、DataWeave項目マッピング、Connector呼び出し詳細、MUnitテストケース詳細 |
| `applications/{appId}/raml/common/` | アプリケーション内の複数APIで共有するRAML type、trait、example |
| `applications/{appId}/raml/{apiFolder}/{version}/` | API root RAML、Operation fragment、API固有type、Operation固有example |
| `applications/{appId}/src/main/mule/` | Mule XML実装 |
| `applications/{appId}/src/main/resources/dwl/` | DataWeave実装 |
| `applications/{appId}/src/test/munit/` | MUnitテスト |
| `deploy_files/` | Jenkins用デプロイ定義。Applicationとして扱わない |
| `_docs/` | 既存リポジトリ由来のlegacy docs。AI-first設計の正本ではない |
| `prompts/` | AIレビュー用プロンプト |

## 読む順番

1. `README.md`
2. `AGENTS.md`
3. ルートの `design-index.yaml`
4. `docs/design-index-field-definition.md`
5. `applications/README.md`
6. 対象アプリケーションの `applications/{appId}/README.md`
7. 対象アプリケーションの `applications/{appId}/design-index.yaml`
8. `applications/{appId}/application-basic-design.md`
9. 関連するAPI root RAML
10. 関連するOperation RAML fragment
11. `applications/{appId}/application-detail-design.md`
12. 関連するOperation詳細設計
13. 実装が存在する場合は `applications/{appId}/src/` 配下のMule XML、DataWeave、MUnit

## 正本ルール

| 情報 | 正本 |
|---|---|
| リポジトリ内のApplication一覧 | `applications/` 直下のディレクトリ |
| design indexのフィールド定義 | `docs/design-index-field-definition.md` |
| Application / API / Operation の対応関係 | `applications/{appId}/design-index.yaml` |
| APIのrequest/response契約 | RAML |
| 基本設計上の判断 | `applications/{appId}/application-basic-design.md` |
| アプリケーション共通のFlow / Connector / Error Handler設計 | `applications/{appId}/application-detail-design.md` |
| Operation別のFlow / Processor / DataWeave / MUnit設計 | `applications/{appId}/operations/{apiId}/{method}_{operationName}/operation-detail-design.md` |
| Mule実装 | `applications/{appId}/src/main/mule/` |
| DataWeave実装 | `applications/{appId}/src/main/resources/dwl/` |
| 単体テスト実装 | `applications/{appId}/src/test/munit/` |

## Legacy Application参照ルール

AI-first管理対象のApplicationは `applications/` 配下に存在するものだけです。

既存リポジトリから段階導入する場合、root直下に残る `*-xapi-v*`、`*-papi-v*`、`*-sapi-v*` 形式のMuleアプリケーションはlegacy applicationとして扱います。legacy applicationは通常のAI-firstレビュー対象外です。

legacy applicationは、移行作業、互換性確認、またはユーザーが明示的に参照を依頼した場合のみ確認します。`deploy_files/` はJenkins用のデプロイ定義置き場であり、Applicationとして扱いません。Application移行時は `deploy_files/` 内の参照パスを確認してください。`_docs/` はlegacy docsとして扱い、AI-first設計の正本にはしません。

## パス合成ルール

アプリケーション内の `design-index.yaml` では、APIパスを次のルールで扱います。

```text
full API path = api.basePath + operation.path
```

- `api.basePath` はRAML rootの `baseUri` のパス部分と一致させます。
- `operation.path` は該当OperationのRAMLリソースパスと一致させます。
- 同じリソースセグメントを `api.basePath` と `operation.path` の両方に重複して書かないでください。

## RAML共通部品の配置ルール

- アプリケーション内の複数APIで再利用するtype、trait、exampleは `applications/{appId}/raml/common/` に配置します。
- API契約に固有のドメインtypeは `applications/{appId}/raml/{apiFolder}/{version}/types/` に配置します。
- Operation固有のrequest / response exampleは `applications/{appId}/raml/{apiFolder}/{version}/examples/` に配置します。
- `common/` はアプリケーション境界を越えて共有しないでください。別アプリケーションで同じ部品が必要な場合も、まずは対象アプリケーション配下に明示的に配置します。

## フェーズ別の更新ルール

| フェーズ | 主な更新対象 | 目的 |
|---|---|---|
| 要件定義 | ルート `design-index.yaml`, `applications/{appId}/design-index.yaml` | Application探索ルールと、API、Operation候補を整理する |
| 基本設計 | `applications/{appId}/application-basic-design.md`, `applications/{appId}/raml/**`, `applications/{appId}/design-index.yaml` | API契約、責務、API管理、シーケンス、エラー方針を定義する |
| 詳細設計 | `applications/{appId}/application-detail-design.md`, `applications/{appId}/operations/**/operation-detail-design.md`, `applications/{appId}/design-index.yaml` | 共通Flow、Connector、Error Handlerと、Operation別のProcessor、詳細シーケンス、DataWeave項目マッピング、Connector呼び出し詳細、MUnitテストケース詳細を定義する |
| 実装 | `applications/{appId}/src/main/mule/**`, `applications/{appId}/src/main/resources/dwl/**`, `applications/{appId}/src/test/munit/**` | Muleアプリとテストを実装する |
| 変更管理 | 影響するRAML、設計書、index、実装、テスト | Operation ID単位で影響を追跡する |

## 使い始め方

1. このテンプレートリポジトリをコピーします。
2. 新しいMuleアプリケーションごとに `applications/{appId}/` を作成します。アプリケーションをバージョン単位で管理する場合は、`sample-domain-sapi-v1` のようにアプリケーションフォルダへバージョンを含めます。
3. `applications/{appId}/` に `README.md`、`design-index.yaml`、`application-basic-design.md`、`application-detail-design.md` を配置します。
4. `applications/{appId}/design-index.yaml` にAPIとOperationを定義します。API IDは `sample-customer-api-v1` のように契約識別子として保持し、APIフォルダは `sample-customer-api` のようにバージョンなしで定義します。
5. API契約は `applications/{appId}/raml/{apiFolder}/{version}/` に配置します。
6. 基本設計は `applications/{appId}/application-basic-design.md` に記載します。
7. アプリケーション共通の詳細設計は `applications/{appId}/application-detail-design.md` に記載し、Operation別詳細は `applications/{appId}/operations/{apiId}/{method}_{operationName}/operation-detail-design.md` に記載します。
8. AI支援レビューを行う場合は `prompts/` 配下のプロンプトを利用します。
