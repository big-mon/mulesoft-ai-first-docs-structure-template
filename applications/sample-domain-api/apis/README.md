# RAML構成

RAMLは、APIのrequest / response契約の正本です。

各APIはroot RAMLファイルを持ちます。各OperationはRAML method fragmentとして定義し、root RAMLから `!include` で参照します。

```text
applications/{appId}/
  common/
    raml/
      types/
      traits/
      examples/
  apis/
    {apiId}/
      raml/
        v1/
          {api-id}.raml
          types/
      operations/
        {operationFolder}/
          {operation}.raml
          examples/
```

## ルール

- root RAMLはAPI契約の入口を表します。
- Operation RAML fragmentはmethod単位の契約を表します。
- 共通のエラー定義やヘッダー定義は `common/raml/` 配下に配置します。
- API固有のドメインtypeは、原則として各APIディレクトリ配下に配置します。
- Operation RAML fragmentとOperation固有exampleは、該当APIの `operations/{operationFolder}/` 配下に配置します。
- RAMLを正本とする契約情報は、設計書側に重複記載しないでください。
