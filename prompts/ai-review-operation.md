# AIレビュー用プロンプト: Operation単位

1つのOperationをレビューするためのプロンプトです。

```text
appId: {appId}, apiId: {apiId}, apiFolder: {apiFolder}, version: {version}, operationId: {operationId} をレビューしてください。

最初にルートの design-index.yaml と applications/{appId}/design-index.yaml を読んでください。
その後、対象OperationのRAML fragment、関連するRAML type / example、application-detail-design.md、対象Operationのoperation-detail-design.mdを読んでください。

以下を確認してください。
- Operation RAML fragmentが applications/{appId}/raml/{apiFolder}/{version}/resources/{operationRaml} に存在すること。
- operationRamlが `{method}_{operation-name}.raml` の命名になっていること。
- Methodとpathが applications/{appId}/design-index.yaml とRAMLで一致していること。
- 共通traitを使う場合、applications/{appId}/raml/common/ の部品を参照していること。
- Operation詳細設計が applications/{appId}/operations/{apiId}/{method}_{operationName}/operation-detail-design.md に存在すること。
- Request body、Response body、Header、Status codeが明確であること。
- RAMLのエラー応答とError Handler設計が整合していること。
- Operation詳細設計の詳細シーケンス図とProcessor表が実装入力として十分であること。
- 変換が必要な場合、DataWeaveのsource-to-target項目マッピングが定義されていること。
- Connector呼び出し詳細でmethod、path、header、body、timeout、error mappingが定義されていること。
- MUnitテストケース詳細で正常系と主要異常系の入力、mock、assertが定義されていること。

ファイルは変更しないでください。指摘事項と修正案を報告してください。
```
