# AIレビュー用プロンプト: Application単位

Muleアプリケーション全体の設計をレビューするためのプロンプトです。

```text
最初に README.md、AGENTS.md、ルートの design-index.yaml を読んでください。
その後、対象Applicationの applications/{appId}/design-index.yaml を読み、apis[] 配下に定義されているすべてのAPIとOperationをレビューしてください。

以下を確認してください。
- 対象Applicationの資産が applications/{appId}/ 配下にまとまっていること。
- アプリケーション内共通のRAML部品が applications/{appId}/raml/common/ に閉じていること。
- すべてのAPIにroot RAMLファイルが存在すること。
- すべてのOperationにOperation RAML fragmentが存在すること。
- すべてのOperation RAML fragmentが対象APIの resources/{operationRaml} として存在すること。
- すべてのOperationが一意のoperationIdを持つこと。
- RAML、基本設計、詳細設計、Flow、DataWeave、MUnitの参照関係を追跡できること。
- API ManagerとAutodiscoveryの設定が、必要なAPIごとに定義されていること。
- どれか1つの成果物にしか存在しないOperationがないこと。

不整合はOperation ID単位で報告してください。
```
