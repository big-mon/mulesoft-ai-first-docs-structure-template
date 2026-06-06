# AIレビュー用プロンプト: API単位

1つのAPIをレビューするためのプロンプトです。

```text
apiId: {apiId} のAPIをレビューしてください。

最初に design-index.yaml を読んでください。
その後、対象APIのroot RAML、すべてのOperation RAML fragment、基本設計書と詳細設計書の関連セクションを読んでください。

以下を確認してください。
- root RAMLが design-index.yaml に定義されたすべてのOperationを含んでいること。
- RAML内のOperation IDが design-index.yaml と一致していること。
- Request typeとResponse typeが正しく定義・参照されていること。
- 共通エラー応答が一貫して適用されていること。
- API ManagerのPolicy、利用者、Autodiscovery flowRefが定義されていること。
- 詳細設計にすべてのOperationのOperation-to-Flow対応が定義されていること。

指摘事項はOperation ID単位で整理して報告してください。
```
