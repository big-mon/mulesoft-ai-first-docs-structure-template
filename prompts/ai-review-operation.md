# AIレビュー用プロンプト: Operation単位

1つのOperationをレビューするためのプロンプトです。

```text
appId: {appId}, apiId: {apiId}, apiFolder: {apiFolder}, version: {version}, operationId: {operationId} をレビューしてください。

最初にルートの design-index.yaml と applications/{appId}/design-index.yaml を読んでください。
その後、対象OperationのRAML fragment、関連するRAML type / example、詳細設計書の該当Operationセクションを読んでください。

以下を確認してください。
- Operation RAML fragmentが applications/{appId}/raml/{apiFolder}/{version}/resources/{operationRaml} に存在すること。
- operationRamlが `{method}_{operation-name}.raml` の命名になっていること。
- Methodとpathが applications/{appId}/design-index.yaml とRAMLで一致していること。
- Request body、Response body、Header、Status codeが明確であること。
- RAMLのエラー応答とError Handler設計が整合していること。
- Flow図とProcessor明細が実装入力として十分であること。
- 変換が必要な場合、DataWeaveマッピングが定義されていること。
- MUnitシナリオが正常系と主要異常系をカバーしていること。

ファイルは変更しないでください。指摘事項と修正案を報告してください。
```
