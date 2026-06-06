# AGENTS.md

このリポジトリは、人間の開発者とAIエージェントの双方が利用することを前提にしています。

AIエージェントは、このファイルを作業ガイドとして読み、設計理解、レビュー、変更作業を行ってください。

## 必須の読み順

変更またはレビューを行う前に、次の順番でファイルを確認してください。

1. `README.md`
2. `design-index.yaml`
3. `docs/application-basic-design.md`
4. 関連するRAML rootファイル
5. 関連するOperation RAML fragment
6. `docs/application-detail-design.md`
7. 実装が存在する場合は、関連するMule XML、DataWeave、MUnitファイル

## 基本階層

```text
Application
  └─ API
       └─ Operation
```

- Application = Muleアプリ / jar / repository / デプロイ単位
- API = RAML root / API Manager単位 / APIkit Router単位
- Operation = HTTP Method + Path / RAML fragment / Mule Flow単位

## 正本ルール

| トピック | 正本 |
|---|---|
| Application、API、Operationの一覧 | `design-index.yaml` |
| APIのrequest / response契約 | RAML |
| 基本設計 | `docs/application-basic-design.md` |
| FlowとProcessorの設計 | `docs/application-detail-design.md` |
| Mule実装 | `src/main/mule/`, `src/main/resources/dwl/` |
| 単体テスト実装 | `src/test/munit/` |

## パス合成ルール

パスを検証する際は、次のルールを使用してください。

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

## 変更ルール

Operation RAML fragmentを変更する場合は、あわせて次のファイル・資産を確認してください。

- `design-index.yaml`
- RAML rootファイル
- `docs/application-basic-design.md`
- `docs/application-detail-design.md`
- DataWeaveのマッピングまたは実装
- MUnitのテスト設計または実装

明示的に指示されていない限り、`operationId` を変更しないでください。

実装だけを根拠に、新しいAPI仕様や振る舞いを推測しないでください。

RAMLと実装が矛盾している場合は、勝手に解決せず、矛盾として報告してください。

## 命名規約

| 資産 | 命名ルール | 例 |
|---|---|---|
| API ID | `{domain}-api-v{version}` | `sample-customer-api-v1` |
| Operation ID | `{apiId}.{resource}.{action}` | `sample-customer-api-v1.customer.getById` |
| Operation RAML | kebab-case | `customer-get-by-id.raml` |
| Flow | kebab-case + `-flow` | `get-customer-by-id-flow` |
| DataWeave | Operationベース | `customer-get-by-id-response.dwl` |
| MUnit | Operation + シナリオ | `customer-get-by-id-success-test` |

## AIレビュー観点

各Operationについて、次を確認してください。

- `design-index.yaml` にOperationが存在すること
- Operation RAML fragmentが存在すること
- RAML rootからOperation fragmentが参照されていること
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
- `design-index.yaml` の `apis[]` 以外に新しいAPIを追加しないでください。
