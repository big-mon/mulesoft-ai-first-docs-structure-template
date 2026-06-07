# Applications

このディレクトリは、Muleアプリケーション単位の設計資産を格納する場所です。

このREADMEは探索用の案内であり、Application、API、Operationの正本ではありません。リポジトリに存在するApplication一覧は、`applications/` 直下のディレクトリを参照してください。

## 読み方

1. `applications/` 直下のディレクトリから対象Applicationを特定します。
2. 対象Applicationの `applications/{appId}/README.md` を読みます。
3. 対象Applicationの `applications/{appId}/design-index.yaml` を読みます。
4. 対象Applicationのdesign indexに従って、基本設計、RAML、アプリケーション詳細設計、Operation詳細設計、実装資産を確認します。

## ルール

- Application資産は `applications/{appId}/` 配下に閉じます。
- `applications/` 直下のディレクトリ名を `appId` として扱ってください。
- Application間でRAML、設計書、実装、テスト資産を混在させないでください。
- READMEには仕様を重複記載せず、探索順と正本への導線だけを記載します。
