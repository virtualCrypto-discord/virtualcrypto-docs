# REST API
以下のエンドポイントについて、特に記載がない場合はアプリケーションのアクセストークンによる認証が必要です。  
また、リクエストのパラメータについて、特に記載のない場合`GET`についてはクエリパラメータによって引き渡されるものとし、その他の場合はJSONによるものとします。   
リクエストがJSONをBodyとしてもつ場合`Content-Type`ヘッダーにMIME type、`application/json`を指定しなければなりません。  
レスポンスについては記載のない場合、JSONとなります。  
未知のフィールドは無視しなければなりません(レスポンスへのフィールドの追加や、リクエストへのOptionalなフィールドの追加は破壊的な変更とみなされません)。
`error_description`フィールドの内容、およびその存在の有無は仕様の範囲外です。


---

## Versions
| Version | Path    | Status     |
| ------- | ------- | ---------- |
| v1      | /api/v1 | deprecated |
| v2      | /api/v2 | stable     |

Beta期間中は破壊的な変更が予告なく行われる可能性があります。

---

## Changes
### v2
- [PR(#242)](https://github.com/virtualCrypto-discord/virtualCrypto2/pull/242)
- ClaimおよびPayでのエラーを適正に
- /currencies/:idのidの解釈を修正
- 日付時刻(created_atおよびupdated_at)にタイムゾーンを追加(Zをつけただけ…)
- /moneysの廃止
---
## Type Definitions
### Type User
| Parameter Name | Parameter Type             | Parameter Description                                                  |
| -------------- | -------------------------- | ---------------------------------------------------------------------- |
| id             | String                     | VirtualCryptoにおけるユーザーのid                                      |
| discord        | DiscordUser,null,undefined | discord上でアカウントを持たないアプリケーションではnullまたはundefined |
### Type DiscordUser
[DiscordのUser Structure](https://discord.com/developers/docs/resources/user#user-object)に従いますが、 `id`、`username`、`discriminator`、`avatar`、`bot`、`system`、`mfa_enabled`、`premium_type`、`public_flags`以外のフィールドが提供されることはありません。(セキュリティ上の理由で前述したフィールド以外のものを削除しています)
### Type Currency

| Parameter Name |                  | Parameter Description                              |
| -------------- | ---------------- | -------------------------------------------------- |
| unit           | String           | 通貨単位                                           |
| guild          | String           | 発行元guild                                        |
| name           | String           | 通貨名                                             |
| pool_amount    | String           | プールの残通貨量                                   |
| total_amount   | String,undefined | 通貨流通量(全ユーザーの所有量を足し合わせたもの)。 |

### Type Claim
| Parameter Name | Parameter Type | Parameter Description                                                                                                                                                   |
| -------------- | -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id             | String         | 請求id                                                                                                                                                                  |
| amount         | String         | 請求額                                                                                                                                                                  |
| claimant       | User           | 請求者                                                                                                                                                                  |
| payer          | User           | 被請求者                                                                                                                                                                |
| currency       | Currency       | 請求されている通貨。`total_amount`フィールドは存在しない。                                                                                                              |
| status         | String         | 請求の状態。`pending`、`approved`、`canceled`、`denied`のいずれか                                                                                                       |
| metadata       | Object         | 40文字(コードポイント)以下の任意のキー、値として、500文字以下の文字列を持つ大きさ50以下のObject。これは関連するユーザーごとにプライベートであり、他者に公開されません。 |
| created_at     | String         | 請求作成日                                                                                                                                                              |
| updated_at     | String         | 請求更新日                                                                                                                                                              |



---
## Currencies
通貨の情報を扱います。
このエンドポイントは認証不要です。
### Get Currency By Query
`/currencies` に適当なクエリパラメータを付与し`GET`リクエストを行い、通貨に関する情報を取得します。
#### Get Currency By Query Request
| Parameter Name | Parameter Description |
| -------------- | --------------------- |
| unit           | 通貨単位*1            |
| guild          | 発行元guild*1         |
| name           | 通貨名*1              |

*1: いずれかから1つのみを指定する必要があります。  

e.g.
```http
  GET /api/v2/currencies?unit=n HTTP/1.1
  Accept: application/json
  Host: vcrypto.sumidora.com
```
#### Get Currency By Query Response
通貨が見つかった場合、ステータスコード`200`で[Currency](#type-currency)がレスポンスとして返却されます(ただし、`total_amount`が必ず存在します)。

e.g.
```json
{
  "guild": "494780225280802817",
  "name": "nyan",
  "pool_amount": "500",
  "total_amount": "100000",
  "unit": "n"
}
```
#### Get Currency By Query Error Response
通貨が見つからなかった場合、ステータスコード`404`で以下のレスポンスが返却されます。
```json
{
  "error": "not_found",
  "error_description": "not_found"
}
```
### Get Currency By Id
`/currencies/:id`へ`GET`リクエストを行い通貨に関する情報を取得します。
v1ではここでidがギルドidとしてあやまって解釈されていました。
v2では通貨のidとして解釈されます。

#### Get Currency By Id Request
e.g.
```http
  GET /api/v2/currencies/1 HTTP/1.1
  Accept: application/json
  Host: vcrypto.sumidora.com
```

#### Get Currency By Id Response
Queryに同じ。
#### Get Currency By Id Error Response
Queryに同じ。

---

## User Transactions
ユーザーの支払いについて扱います。  
認証が必要です。
### Create User Transactions(Do Pay)
`/users/@me/transactions`へ`POST`リクエストを行い、支払いを行います。
#### Create User Transactions(Do Pay) Idempotency
また、リクエストを冪等にするため`v2`以降、Idempotency-Key HTTP Headerをサポートしています。 

名前空間はJWTのissごとです。

`Idempotency-Key`は次に示す規則に一致しなければなりません。(参考: [The Idempotency HTTP Header Field draft-idempotency-header-01](https://tools.ietf.org/html/draft-idempotency-header-01#section-2.1))
```abnf
Idempotency-Key       = idempotency-key-value
idempotency-key-value = opaque-value
opaque-value          = DQUOTE *idempotencyvalue DQUOTE
idempotencyvalue      = %x21 / %x23-7E / obs-text
      ; VCHAR except double quotes, plus obs-text
```
加えて、`opaque-value`は`"`を除いて0文字以上256文字以下である必要があります。

レスポンスには`Idempotency-Status`ヘッダが含まれます。  
これは、[Idempotency | Worldpay Developers](https://developer.worldpay.com/docs/wpg/idempotency)の影響を受けていますが、細部が異なります。
`Idempotency-Status`で`Invalid Key`が通知されることはありません。

リクエストの処理が完了する前に同一ユーザー、同一`Idempotency-Key`のリクエストが行われた場合、ステータスコード`409`で以下のレスポンスが返却されます。
```json
{
  "error": "processing"
}
```

`Idempotency-Key`は最低7日間保持されます(それ以上保持されることもありえます)。

#### Create User Transactions(Do Pay) Request
以下のフィールドを持つオブジェクトまたはその配列をBodyとして上記のURLへ`POST`リクエストを行ってください。
| Parameter Name      | Parameter Type | Parameter Description     |
| ------------------- | -------------- | ------------------------- |
| unit                | String         | 通貨単位                  |
| receiver_discord_id | String         | 受取人のdiscordにおけるid |
| amount              | String         | 支払額                    |

#### Create User Transactions(Do Pay) Response
成功時は`2xx`が返却されます。

#### Create User Transactions(Do Pay) Error Response 
##### Create User Transactions(Do Pay) Error Response Not Enough Amount
支払おうとしたときに通貨が不足していた場合ステータスコード`409`で以下のレスポンスが返却されます。
###### v2
```json
{
  "error": "conflict",
  "error_info": "not_enough_amount"
}
```

###### v1
支払おうとしたときに通貨が不足していた場合ステータスコード`400`で以下のレスポンスが返却されます。
```json
{
  "error": "invalid_request",
  "error_info": "not_enough_amount"
}
```

---

## Claims
請求の作成、確認、承認や拒否、キャンセルが可能です。

### List Claims
請求の一覧を取得します。
#### List Claims Request
`/users/@me/claims` へ`GET`を行ってください。
| Parameter Name          | Parameter Type | Parameter Description                                                                                                                 |
| ----------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| statuses                | String[]?      | 請求の状態(`pending`、`canceled`、`approved`、`denied`のいずれか)。デフォルトは`pending`。                                            |
| related_discord_user_id | String?        | 相手方のdiscordにおけるid(`related_vc_user_id`と排他)。指定されなかった場合すべてのユーザーが対象。                                   |
| related_vc_user_id      | String?        | 相手方のVirtualCryptoにおけるid(`related_discord_user_id`と排他)。指定されなかった場合すべてのユーザーが対象。                        |
| type                    | String?        | 請求の種類(`claimed`、`received`、`all`)デフォルトは`all`。                                                                           |
| limit                   | Number?        | 返却する請求の個数の上限。デフォルトは無制限ですがこの挙動に依存しないでください。                                                    |
| next                    | String?        | ページネーション用のパラメータ。このパラメータは開区間(区切りを含まない)。`on_next`と排他。どちらも指定されない場合先頭を取得します。 |
| on_next                 | String?        | ページネーション用のパラメータ。このパラメータは閉区間(区切りを含む)。`next`と排他。どちらも指定されない場合先頭を取得します。        |
| order                   | String?        | 請求の順序(`asc_claim_id`、`desc_claim_id`)。デフォルトは`desc_claim_id`。                                                            |

##### List Claims Request URL Examples
- `/users/@me/claims?statuses[]=pending&statuses[]=approved&statuses[]=canceled&statuses[]=denied&limit=10&next=87`
- `/users/@me/claims?statuses[]=pending&statuses[]=approved&statuses[]=canceled&statuses[]=denied&limit=10&on_next=87`
- `/users/@me/claims?statuses[]=pending&statuses[]=approved&statuses[]=canceled&statuses[]=denied&order=asc_claim_id&limit=10&next=87`
- `/users/@me/claims?statuses[]=pending&statuses[]=approved&statuses[]=canceled&statuses[]=denied&order=asc_claim_id&limit=10&related_discord_user_id=408939071289688064`
- `/users/@me/claims?statuses[]=pending&statuses[]=approved&statuses[]=canceled&statuses[]=denied&order=asc_claim_id&limit=10&related_discord_user_id=408939071289688064&type=claimed`

##### next、on_nextの値について
next、on_nextの値は以下の表に従います。
| order                           | value |
| ------------------------------- | ----- |
| `asc_claim_id`、`desc_claim_id` | `id`  |

#### List Claims Response
[Claim](#type-claim)の配列が返却されます。

e.g.
```json
[{
  "amount": "100",
  "claimant": {
    "discord": {
      "avatar": "9940018bc861cf7a1ca308228d4a7be8",
      "discriminator": "2552",
      "id": "408939071289688064",
      "public_flags": 64,
      "username": "tig"
    },
    "id": "1"
  },
  "created_at": "2021-02-07T05:46:15Z",
  "currency": {
    "guild": "494780225280802817",
    "name": "nyan",
    "pool_amount": "76",
    "unit": "n"
  },
  "id": "1",
  "payer": {
    "discord": {
      "avatar": "9940018bc861cf7a1ca308228d4a7be8",
      "discriminator": "2552",
      "id": "408939071289688064",
      "public_flags": 64,
      "username": "tig"
    },
    "id": "1"
  },
  "metadata": {
    "X" => "y"
  },
  "status": "pending",
  "updated_at": "2021-02-07T05:46:15Z"
},
...
]
```
##### List Claims Response Link Header
```http
link: <https://localhost:4000/api/v2/users/@me/claims?type=claimed&order=asc_claim_id&next=10&limit=10&related_discord_user_id=408939071289688064&statuses%5B%5D=pending&statuses%5B%5D=approved&statuses%5B%5D=canceled&statuses%5B%5D=denied>; rel="next"
```
以上のような`Link` Headerを返却します。この場合<>のURLにアクセスすることで次ページを取得することができます。なお、必ず次ページがあることを保証しません。空の配列が返却される場合があります。

### Get Claim By Id
請求idから請求を取得します。
### Get Claim By Id Request
`/users/@me/claims/:id`へ`GET`を行ってください。

#### Get Claim By Id Response

[Claim](#type-claim)が返却されます。

#### Get Claim By Id Error Response
##### v2
存在しないidを指定した場合`404`が返却されます。
```json
{
  "error": "not_found",
  "error_description": "not_found"
}
```
閲覧権限がない場合は`403`が返却されます。

```json
{
  "error": "forbidden",
  "error_description": "invalid_operator"
}
```

##### v1
閲覧権限がない場合や存在しないidを指定した場合`404`が返却されます。


## Create Claim
請求の作成が可能です。

### Create Claim Request
以下のフィールドを持つJSONをBodyとして上記のURLへ`POST`リクエストを行ってください。

| Parameter Name   | Parameter Type        | Parameter Description                             |
| ---------------- | --------------------- | ------------------------------------------------- |
| payer_discord_id | String                | 請求先のユーザーのdiscordのid                     |
| unit             | String                | 請求する通貨の`unit`                              |
| amount           | String                | 請求額                                            |
| metadata         | Object,null,undefined | メタデータ。nullとundefinedは同じ挙動を示します。 |
### Create Claim Response

作成後の[Claim](#type-claim)がステータスコード`201`で返却されます。

### Create Claim Error Response
不正な入力以外の要因で失敗しません。

## Update Claim
請求の承認や拒否とキャンセルが可能です。
### Update Claim Request
以下のフィールドを持つJSONをBodyとして上記のURLへ`PATCH`リクエストを行ってください。

| Parameter Name | Parameter Type        | Parameter Description                                                                                                                                                                 |
| -------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| status         | String,undefined      | 請求の状態。`approved`、`canceled`、`denied`のいずれか                                                                                                                                |
| metadata       | Object,null,undefined | メタデータ。nullとundefinedは異なる挙動を示します。nullが指定された場合、請求に紐づくメタデータがすべて削除されます。また、値にnullを指定することでそのキーに紐づく値を削除できます。 |

### Update Claim Response

更新後の[Claim](#type-claim)が返却されます。

### Update Claim Error Response

#### v2
すでに状態が`pending`以外に遷移している場合`409`が返却されます。

```json
{
  "error": "conflict",
  "error_info": "invalid_status"
}
```
#### v1
すでに状態が`pending`以外に遷移している場合`400`が返却されます。


#### Update Claim Error Response Not Enough Amount
##### v2
承認しようとしたとき、通貨が不足していた場合、ステータスコード`409`で以下のレスポンスが返却されます。
```json
{
  "error": "conflict",
  "error_info": "not_enough_amount"
}
```
##### v1
承認しようとしたとき、通貨が不足していた場合、ステータスコード`400`で以下のレスポンスが返却されます。

```json
{
  "error": "invalid_request",
  "error_info": "not_enough_amount"
}
```

