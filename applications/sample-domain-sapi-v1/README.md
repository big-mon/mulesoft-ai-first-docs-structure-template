# sample-domain-sapi-v1

このREADMEは、`sample-domain-sapi-v1` の設計資産を探索するための案内です。

このアプリケーションは、アプリケーション名に `sapi` を含むSAPIサンプルです。Application、API、Operationの対応関係の正本は `design-index.yaml` です。

## 読み方

1. `design-index.yaml` でApplication、API、Operation、RAML、Flow、DataWeave、MUnitの対応を確認します。
2. `application-basic-design.md` でアプリケーション責務、API基本設計、Operation概要を確認します。
3. `design-index.yaml` に記載されたroot RAMLとOperation RAML fragmentを確認します。
4. `application-detail-design.md` でAPIkit Router、共通Flow、共通Connector、Error Handler、DataWeave、MUnitの一覧を確認します。
5. `operations/{apiId}/{method}_{operationName}/operation-detail-design.md` でOperation別の詳細シーケンス、Processor表、Flow詳細、DataWeave、MUnit観点を確認します。
6. 実装が存在する場合は `src/` 配下のMule XML、DataWeave、MUnitを確認します。

## ディレクトリ

| Path | 用途 |
|---|---|
| `design-index.yaml` | Application / API / Operation / RAML / Flow / DataWeave / MUnit の対応関係 |
| `application-basic-design.md` | 基本設計 |
| `application-detail-design.md` | アプリケーション共通の詳細設計とOperation詳細設計への導線 |
| `operations/{apiId}/{method}_{operationName}/operation-detail-design.md` | Operation別の詳細設計 |
| `raml/common/` | アプリケーション内共通のRAML type、trait、example |
| `raml/{apiFolder}/{version}/` | API root RAML、Operation fragment、API固有type、Operation固有example |
| `src/` | Mule実装、DataWeave、MUnit |

## 注意

- API一覧とOperation一覧は `design-index.yaml` を参照してください。
- READMEには仕様を重複記載しません。
- `design-index.yaml` の `apis[]` に未登録のAPIを、このアプリケーションの設計対象として扱わないでください。
