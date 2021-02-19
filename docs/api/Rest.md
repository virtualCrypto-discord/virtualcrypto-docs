# REST API
## Versions
| Version | Path    | Status |
| ------- | ------- | ------ |
| v1      | /api/v1 | Beta   |

Beta期間中は破壊的な変更が予告なく行われる可能性があります。
## Currencies
通貨の情報を扱います。
## Get Currency By Query
`/currencies` に適当なクエリパラメータを付与し`GET`リクエストを行い、通貨に関する情報を取得します。
### Get Currency By Query Request
| Parameter Name | Parameter Description |
| -------------- | --------------------- |
| unit           | 通貨単位*1            |
| guild          | 発行元guild*1         |
| name           | 通貨名*1              |
*1: いずれかから1つのみを指定する必要があります。  

e.g.
```
  GET /api/v1/currencies?unit=n HTTP/1.1
  Accept: application/json
  Host: vcrypto.sumidora.com
```
### Get Currency By Query Response
通貨が見つかった場合、ステータスコード`200`で以下のようなレスポンスが返却されます。
| Parameter Name | Parameter Description                            |
| -------------- | ------------------------------------------------ |
| unit           | 通貨単位                                         |
| guild          | 発行元guild                                      |
| name           | 通貨名                                           |
| pool_amount    | プールの残通貨量                                 |
| total_amount   | 通貨流通量(全ユーザーの所有量を足し合わせたもの) |
```json
{
  "guild": "494780225280802817",
  "name": "nyan",
  "pool_amount": "500",
  "total_amount": "100000",
  "unit": "n"
}
```
### Get Currency By Query Error Response
通貨が見つからなかった場合、ステータスコード`404`で以下のレスポンスが返却されます。
```json
{
  "error": "not_found",
  "error_description": "not_found"
}
```
## Get Currency By Id
`/currencies/:id`へ`GET`リクエストを行い通貨に関する情報を取得します。
### Get Currency By Id Request
e.g.
```
  GET /api/v1/currencie/1 HTTP/1.1
  Accept: application/json
  Host: vcrypto.sumidora.com
```

