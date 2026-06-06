# RAML構成

RAMLは、APIのrequest / response契約の正本です。

各APIはroot RAMLファイルを持ちます。各OperationはRAML method fragmentとして定義し、root RAMLから `!include` で参照します。

```text
applications/{appId}/
  raml/
    {apiFolder}/
      {version}/
        {api-root-name}.raml
        resources/
        types/
        examples/
        traits/
```

## ルール

- root RAMLはAPI契約の入口を表します。
- APIフォルダは `sample-customer-api` のようにバージョンを含めず、API契約上のバージョンはroot RAMLの `version` と `baseUri`、およびアプリケーションフォルダで表します。
- Operation RAML fragmentは `resources/` 配下に置き、method単位の契約を表します。
- Operation RAMLは `get_customer-get-by-id.raml` や `post_customer-search.raml` のようにHTTP methodとOperation名を `_` で区切ります。
- Request bodyを持つOperationは、Operation RAML fragment内の `body` にtypeとexampleを必ず記載します。
- 共通のエラー定義やヘッダー定義は、API version配下の `traits/` に配置します。
- API固有のドメインtypeは、原則として各APIディレクトリ配下に配置します。
- Operation固有exampleは、API version配下の `examples/` に配置します。
- RAMLを正本とする契約情報は、設計書側に重複記載しないでください。
