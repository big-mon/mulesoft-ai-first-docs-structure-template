# AGENTS.md

このリポジトリは、人間の開発者とAIエージェントの双方が利用することを前提にしています。

AIエージェントは、このファイルを作業ガイドとして読み、設計理解、レビュー、変更作業を行ってください。

## 必須の読み順

変更またはレビューを行う前に、次の順番でファイルを確認してください。

1. `README.md`
2. ルートの `design-index.yaml`
3. 対象アプリケーションの `applications/{appId}/design-index.yaml`
4. `applications/{appId}/application-basic-design.md`
5. 関連するRAML rootファイル
6. 関連するOperation RAML fragment
7. `applications/{appId}/application-detail-design.md`
8. 実装が存在する場合は、対象アプリケーション配下のMule XML、DataWeave、MUnitファイル

## 基本階層

```text
Repository
  └─ Application
       └─ API
            └─ Operation
```

- Repository = 複数Muleアプリケーションを格納するリポジトリ単位
- Application = Muleアプリ / jar / repository内のデプロイ単位
- API = RAML root / API Manager単位 / APIkit Router単位
- Operation = HTTP Method + Path / RAML fragment / Mule Flow単位

## 物理パスルール

```text
applications/{appId}/
  design-index.yaml
  application-basic-design.md
  application-detail-design.md
  raml/
    common/
      types/
      traits/
      examples/
    {apiFolder}/
      {version}/
        {api-root}.raml
        resources/
        types/
        examples/
  src/
```

- アプリケーション固有の設計、RAML、Mule実装、テストは必ず `applications/{appId}/` 配下に置いてください。
- アプリケーションをバージョン単位で管理する場合は、`appId` とアプリケーションフォルダに `sample-domain-sapi-v1` のようなバージョンを含めてください。
- APIフォルダは `sample-customer-api` のようにバージョンを重複させず、API契約上のIDは `apiId` として `sample-customer-api-v1` のように保持してください。
- アプリケーション内の複数APIで再利用するRAML type、trait、exampleは `applications/{appId}/raml/common/` 配下に置いてください。
- API固有のRAML root、Operation fragment、type、Operation固有exampleは `applications/{appId}/raml/{apiFolder}/{version}/` 配下に置いてください。
- Operation RAML fragmentは `resources/` 配下に置き、`get_customer-get-by-id.raml` のようにHTTP methodとOperation名を `_` で区切ってください。
- アプリケーション間で資産を混在させないでください。

## 正本ルール

| トピック | 正本 |
|---|---|
| リポジトリ内のApplication一覧 | ルートの `design-index.yaml` |
| Application、API、Operationの一覧 | `applications/{appId}/design-index.yaml` |
| APIのrequest / response契約 | RAML |
| 基本設計 | `applications/{appId}/application-basic-design.md` |
| FlowとProcessorの設計 | `applications/{appId}/application-detail-design.md` |
| Mule実装 | `applications/{appId}/src/main/mule/`, `applications/{appId}/src/main/resources/dwl/` |
| 単体テスト実装 | `applications/{appId}/src/test/munit/` |

## パス合成ルール

パスを検証する際は、次のルールを使用してください。

```text
full API path = api.basePath + operation.path
```

- `api.basePath` はRAML rootの `baseUri` のパス部分と一致させます。
- `operation.path` は該当OperationのRAMLリソースパスと一致させます。
- 同じリソースセグメントを `api.basePath` と `operation.path` の両方に重複して書かないでください。

## 変更ルール

Operation RAML fragmentを変更する場合は、あわせて次のファイル・資産を確認してください。

- ルートの `design-index.yaml`
- 対象アプリケーションの `applications/{appId}/design-index.yaml`
- RAML rootファイル
- `applications/{appId}/application-basic-design.md`
- `applications/{appId}/application-detail-design.md`
- DataWeaveのマッピングまたは実装
- MUnitのテスト設計または実装

明示的に指示されていない限り、`operationId` を変更しないでください。

実装だけを根拠に、新しいAPI仕様や振る舞いを推測しないでください。

RAMLと実装が矛盾している場合は、勝手に解決せず、矛盾として報告してください。

## 設計書構成ルール

`application-basic-design.md` は、次の章構成を標準とします。

```text
1. アプリケーション概要
   1.1 アプリケーション責務
   1.2 jar / repo / deployment単位
   1.3 外部接続先一覧
   1.4 共通処理方針
2. API一覧
3. API別基本設計
4. Operation別概要
5. シーケンス概要
6. データモデル・マッピング概要
7. エラー定義
8. API管理設計
9. 設計トレーサビリティ
```

`application-detail-design.md` は、次の章構成を標準とします。

```text
1. アプリケーション詳細
2. 共通Flow設計
3. 共通Connector設定
4. API別詳細設計
   4.x.1 APIkit Router / entry flow
   4.x.2 Operation - Flow対応表
   4.x.3 Operation別Flow詳細
5. Error Handler詳細
6. DataWeave・Connector・設定一覧
7. MUnitテスト設計
```

- 基本設計と詳細設計には、対象アプリケーションの `design-index.yaml` の `apis[]` に存在するAPIのみ記載してください。
- 構造例として有用なAPIであっても、`design-index.yaml` に未登録であれば設計書本文には追加しないでください。
- XAPI / PAPI / SAPI の判定は `appId` とアプリケーションフォルダ名に含まれる `xapi`、`papi`、`sapi` で行います。
- API一覧やAPI別設計に `Layer` 列、`apiLayer`、`Experience`、`Process`、`System` などの重複情報を追加しないでください。

## 命名規約

| 資産 | 命名ルール | 例 |
|---|---|---|
| API ID | `{domain}-api-v{version}` | `sample-customer-api-v1` |
| API Folder | `{domain}-api` | `sample-customer-api` |
| Operation ID | `{apiId}.{resource}.{action}` | `sample-customer-api-v1.customer.getById` |
| Operation RAML | `{method}_{operation-name}.raml` | `get_customer-get-by-id.raml` |
| Flow | kebab-case + `-flow` | `get-customer-by-id-flow` |
| DataWeave | Operationベース | `customer-get-by-id-response.dwl` |
| MUnit | Operation + シナリオ | `customer-get-by-id-success-test` |

## AIレビュー観点

各Operationについて、次を確認してください。

- 対象アプリケーションの `design-index.yaml` にOperationが存在すること
- Operation RAML fragmentが `resources/{operationRaml}` に存在すること
- RAML rootからOperation fragmentが参照されていること
- 複数APIで再利用されるRAML部品が `raml/common/` にあり、API固有部品と混在していないこと
- `api.basePath + operation.path` がRAML契約上のルートと一致すること
- 詳細設計でFlowが定義されていること
- エラー応答がRAMLと整合していること
- 変換が必要な場合、Mapping / DataWeaveが定義されていること
- MUnitシナリオが正常系と主要異常系をカバーしていること

## 禁止事項

- 明示的な指示なしにOperation IDを変更しないでください。
- API契約を変えるRAML変更を軽微な編集として扱わないでください。
- 矛盾を勝手に解決せず、必ず報告してください。
- サンプル値を本番設計値としてそのまま流用しないでください。
- ルートの `design-index.yaml` の `applications[]` 以外に新しいApplicationを追加しないでください。
- 対象アプリケーションの `design-index.yaml` の `apis[]` 以外に新しいAPIを追加しないでください。
