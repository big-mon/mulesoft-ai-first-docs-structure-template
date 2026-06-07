# design-index field definition

このドキュメントは、ルート `design-index.yaml` と `applications/{appId}/design-index.yaml` のフィールド意味を定義します。

目的は、AIエージェントと人間レビューアが `design-index.yaml` を同じ意味で解釈できるようにし、フィールド名の揺れや推測による補完を避けることです。

## 1. 基本方針

- ルート `design-index.yaml` は、リポジトリ全体のApplication探索ルール、構造ルール、パス規約を定義します。
- Application一覧は `applications/` 直下のディレクトリを正本とします。
- `applications/{appId}/design-index.yaml` は、そのApplication内のAPI、Operation、RAML、設計資産、MUnitテスト名の対応関係を定義します。
- READMEは探索案内であり、`design-index.yaml` のフィールド意味を上書きしません。
- RAML request / response契約はRAMLを正本とします。`design-index.yaml` はRAML、設計書、実装、テストへの対応関係を保持します。

## 2. ルート design-index.yaml

| Field | Required | Meaning | Rule |
|---|---:|---|---|
| `schemaVersion` | true | design index形式のバージョン | 互換性を壊す構造変更時に更新します。 |
| `repository.repositoryId` | true | 親リポジトリの識別子 | Git repository名またはリポジトリ管理上のIDを設定します。Application IDではありません。 |
| `repository.description` | true | リポジトリ全体の説明 | 個別Applicationの説明を書きません。 |
| `repository.applicationRoot` | true | Application群を格納するルートディレクトリ | 通常は `applications` とします。 |
| `applicationDiscovery.mode` | true | Application発見方式 | filesystem discoveryの場合は `filesystem` とします。 |
| `applicationDiscovery.root` | true | Applicationを探索するディレクトリ | `repository.applicationRoot` と一致させます。 |
| `applicationDiscovery.appIdSource` | true | `appId` の取得元 | `directoryName` の場合、Applicationディレクトリ名を `appId` とします。 |
| `applicationDiscovery.requiredFiles` | true | Applicationとして扱うために必要なファイル | `README.md`、`design-index.yaml`、`application-basic-design.md`、`application-detail-design.md` を含めます。 |
| `applicationDiscovery.rules` | false | Application探索時の補足ルール | 自動チェックとAIレビューで参照する短いルールを記載します。 |
| `structure.*PathPattern` | true | 標準パスの合成ルール | `{appId}`、`{apiId}`、`{apiFolder}`、`{version}`、`{operationRaml}` などの変数を使用します。 |

## 3. Application design-index.yaml

### 3.1 application

| Field | Required | Meaning | Rule |
|---|---:|---|---|
| `application.appId` | true | Application識別子 | `applications/{appId}` のディレクトリ名と一致させます。 |
| `application.appName` | true | 表示用Application名 | 人間が読む名称です。IDやパスの正本にはしません。 |
| `application.appRoot` | true | Applicationの物理ルート | `applications/{appId}` と一致させます。Git repository名ではありません。 |
| `application.artifactName` | true | Mule artifact名 | jarデプロイ単位の場合は `{appId}.jar` を基本とします。 |
| `application.deploymentUnit` | true | デプロイ単位 | 例: `jar`。 |
| `application.muleRuntime` | false | Mule runtime version | 実装やPOMと矛盾する場合は矛盾として報告します。 |
| `application.javaVersion` | false | Java version | 実装やPOMと矛盾する場合は矛盾として報告します。 |
| `application.ownerTeam` | false | 所管チーム | 連絡先や責任境界のためのメタデータです。 |

### 3.2 documents

| Field | Required | Meaning | Rule |
|---|---:|---|---|
| `documents.basicDesign` | true | 基本設計ファイル | Application rootからの相対パスで指定します。 |
| `documents.detailDesign` | true | アプリケーション詳細設計ファイル | Application rootからの相対パスで指定します。 |
| `documents.operationDetailDesignPattern` | true | Operation詳細設計の標準パターン | `operations/{apiId}/{method}_{operationName}/operation-detail-design.md` を基本とします。 |

### 3.3 common

| Field | Required | Meaning | Rule |
|---|---:|---|---|
| `common.errorModel` | true | 共通エラーモデルRAML | Application rootからの相対パスで指定します。 |
| `common.commonTypes` | false | Application内で再利用するRAML type一覧 | `raml/common/types/` 配下を基本とします。 |
| `common.commonTraits` | false | Application内で再利用するRAML trait一覧 | `raml/common/traits/` 配下を基本とします。 |
| `common.commonExamples` | false | Application内で再利用するexample一覧 | `raml/common/examples/` 配下を基本とします。 |

### 3.4 apis[]

| Field | Required | Meaning | Rule |
|---|---:|---|---|
| `apis[].apiId` | true | API契約ID | versionを含めます。例: `sample-customer-api-v1`。 |
| `apis[].apiFolder` | true | RAML物理フォルダ名 | versionを含めません。例: `sample-customer-api`。 |
| `apis[].apiName` | true | 表示用API名 | 人間が読む名称です。IDやパスの正本にはしません。 |
| `apis[].version` | true | RAML version folder | `raml/{apiFolder}/{version}` の `{version}` と一致させます。例: `v1`。 |
| `apis[].description` | false | API概要 | 責務を短く記載します。 |
| `apis[].rootRaml` | true | RAML rootファイル | Application rootからの相対パスで指定します。 |
| `apis[].basePath` | true | API base path | RAML rootの `baseUri` のpath部分と一致させます。 |
| `apis[].entryFlow` | true | API entry flow | APIkit Routerを含む入口Flow名です。 |
| `apis[].apiKitConfig` | true | APIkit config名 | Mule XML実装と一致させます。 |
| `apis[].autodiscovery` | false | API Manager autodiscovery設定 | API Manager管理対象の場合に設定します。 |
| `apis[].apiManager` | false | API Manager管理設定 | policy、consumerなどAPI単位の管理情報を記載します。 |
| `apis[].requestHeaders` | false | API共通request header一覧 | 全Operationに適用されるヘッダーのみ記載します。Operation固有ヘッダーはOperation側へ記載します。 |
| `apis[].operations` | true | API配下のOperation一覧 | このAPIの設計対象Operationだけを記載します。 |

### 3.5 apis[].requestHeaders[]

| Field | Required | Meaning | Rule |
|---|---:|---|---|
| `name` | true | Header名 | RAML traitのheader名と一致させます。 |
| `required` | true | 必須有無 | RAML上のrequiredと一致させます。 |
| `source` | false | ヘッダー要求の由来 | 例: `Client ID enforcement`。 |
| `scope` | true | 適用範囲 | API共通の場合は `api` とします。 |
| `validationOwner` | false | 検証責務の所在 | API Manager policyが検証する場合は `api-manager-policy` とし、Flow内の手動Validationと混同しないようにします。 |
| `constraints` | false | 入力制約 | RAML traitのpattern、minLengthなどと一致させます。 |

### 3.6 apis[].operations[]

| Field | Required | Meaning | Rule |
|---|---:|---|---|
| `operationId` | true | Operation識別子 | 明示指示なしに変更しません。命名は `{apiId}.{resource}.{action}` を基本とします。 |
| `method` | true | HTTP method | RAML fragmentのmethodと一致させます。 |
| `path` | true | Operation resource path | RAML resource pathと一致させます。`api.basePath` と重複したセグメントを書きません。 |
| `summary` | true | Operation概要 | 業務的な目的を短く記載します。 |
| `operationRaml` | true | Operation RAML fragment名 | `resources/` 配下のファイル名です。例: `get_customer-get-by-id.raml`。 |
| `raml` | true | Operation RAML fragmentへのパス | Application rootからの相対パスで指定します。 |
| `detailDesign` | true | Operation詳細設計へのパス | Application rootからの相対パスで指定します。 |
| `flow` | true | Operation flow名 | Mule XML実装のFlow名と一致させます。 |
| `requestType` | false | Request body type | Request bodyがない場合は `null` とします。 |
| `responseType` | true | Response body type | RAML response typeと一致させます。 |
| `errorModel` | true | Error response type | 共通エラーモデルを使う場合は `ErrorResponse` とします。 |
| `dataweave` | false | Operationで使用するDWL | request / responseなど用途別にApplication rootからの相対パスで指定します。 |
| `munit` | true | MUnitテスト名一覧 | 正常系、主要な入力不正、主要な接続先異常、timeoutなどを省略しません。 |
| `lifecycle` | false | 設計フェーズ別ステータス | requirement / basicDesign / detailDesign の状態を追跡します。 |

## 4. Path consistency rules

| Check | Rule |
|---|---|
| Application directory | `application.appId` と `application.appRoot` の末尾ディレクトリ名を一致させます。 |
| API path | `full API path = apis[].basePath + apis[].operations[].path` として扱います。 |
| RAML root | `apis[].rootRaml` は `raml/{apiFolder}/{version}/` 配下に置きます。 |
| Operation RAML | `apis[].operations[].raml` は `raml/{apiFolder}/{version}/resources/{operationRaml}` と一致させます。 |
| Operation detail design | `apis[].operations[].detailDesign` は `operations/{apiId}/{method}_{operationName}/operation-detail-design.md` と一致させます。 |
| API Manager policy | API単位のpolicyは `apis[].apiManager.policies` に記載し、Operation固有の挙動として重複記載しません。 |
| MUnit | Operation詳細設計に定義した主要シナリオと `apis[].operations[].munit` を一致させます。 |

## 5. Review checklist

- `appId` はApplication folder名と一致しているか。
- `appRoot` は `applications/{appId}` と一致しているか。
- `apiId` はversionを含み、`apiFolder` はversionを含んでいないか。
- `version` はRAML配下のversion folderと一致しているか。
- `basePath` はRAML rootの `baseUri` path部分と一致しているか。
- `path` はRAML resource pathと一致しているか。
- `operationRaml` は `resources/` 配下に存在するか。
- `detailDesign` はOperation詳細設計ファイルに到達するか。
- `munit` はOperation詳細設計の主要異常系を省略していないか。
- RAML、設計書、index、実装が矛盾する場合、推測で解決せず矛盾として報告しているか。
