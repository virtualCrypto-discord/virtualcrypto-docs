# REST API
以下のエンドポイントについて、特に記載がない場合はアプリケーションのアクセストークンによる認証が必要です。  
また、リクエストのパラメータについて、特に記載のない場合`GET`についてはクエリパラメータによって引き渡されるものとし、その他の場合はJSONによるものとします。   
リクエストがJSONをBodyとしてもつ場合`Content-Type`ヘッダーにMIME type、`application/json`を指定しなければなりません。  
レスポンスについては記載のない場合、JSONとなります。  
未知のフィールドは無視しなければなりません(レスポンスへのフィールドの追加や、リクエストへのOptionalなフィールドの追加は破壊的な変更とみなされません)。
`error_description`フィールドの内容、およびその存在の有無は仕様の範囲外です。


---

## Versions
| Version | Path    | Status |
| ------- | ------- | ------ |
| v1      | /api/v1 | Beta   |

Beta期間中は破壊的な変更が予告なく行われる可能性があります。

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
| Parameter Name | Parameter Type | Parameter Description                                             |
| -------------- | -------------- | ----------------------------------------------------------------- |
| id             | String         | 請求id                                                            |
| amount         | String         | 請求額                                                            |
| claimant       | User           | 請求者                                                            |
| payer          | User           | 被請求者                                                          |
| currency       | Currency       | 請求されている通貨。`total_amount`フィールドは存在しない。        |
| status         | String         | 請求の状態。`pending`、`approved`、`canceled`、`denied`のいずれか |
| created_at     | String         | 請求作成日                                                        |
| updated_at     | String         | 請求更新日                                                        |

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
  GET /api/v1/currencies?unit=n HTTP/1.1
  Accept: application/json
  Host: vcrypto.sumidora.com
```
#### Get Currency By Query Response
通貨が見つかった場合、ステータスコード`200`で[Currency](#type-curreny)がレスポンスとして返却されます(ただし、`total_amount`が必ず存在します)。

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
#### Get Currency By Id Request
e.g.
```http
  GET /api/v1/currencie/1 HTTP/1.1
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
現在アクティブ(`status`が`pending`)な請求の一覧を取得します。
#### List Claims Request
`/users/@me/claims` へ`GET`を行ってください。
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
  "created_at": "2021-02-07T05:46:15",
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
  "status": "pending",
  "updated_at": "2021-02-07T05:46:15"
},
...
]
```

### Get Cliam By Id
請求idから請求を取得します。
### Get Cliam By Id Request
`/users/@me/claims/:id`へ`GET`を行ってください。

#### Get Cliam By Id Response

[Claim](#type-claim)が返却されます。

#### Get Cliam By Id Error Response
閲覧権限がない場合や存在しないidを指定した場合`404`が返却されます。

## Update Claim
請求の承認や拒否とキャンセルが可能です。
### Update Claim Request
以下のフィールドを持つJSONをBodyとして上記のURLへ`PATCH`リクエストを行ってください。

| Parameter Name | Parameter Type | Parameter Description                                             |
| -------------- | -------------- | ----------------------------------------------------------------- |
| status         | String         | 請求の状態。`approved`、`canceled`、`denied`のいずれか |

### Update Claim Response

更新後の[Claim](#type-claim)が返却されます。

### Update Claim Error Response
すでに状態が`pending`以外に遷移している場合`404`が返却されます。

#### Update Claim Error Response Not Enough Amount
承認しようとしたとき、通貨が不足していた場合、ステータスコード`400`で以下のレスポンスが返却されます。

```json
{
  "error": "invalid_request",
  "error_info": "not_enough_amount"
}
```

