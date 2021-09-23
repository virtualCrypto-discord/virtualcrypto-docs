# Webhook API
アプリケーションに関連するイベントを伝えるため、VirtualCryptoはWebhookを用いたイベントの通知に対応しています。
現在、次のイベントの通知が可能です。
- 請求が承認された。
- 請求が拒否された。
[Discordと同様](https://discord.com/developers/docs/interactions/receiving-and-responding#security-and-authorization)に、TimestampとBodyをバイナリとして結合したものにed25519を用いて署名します。
アプリケーションはこれを検証する必要があります。
アプリケーションがこれを検証しているか、VirtualCryptoは登録時に検証し、また、定期的に検証します。

## Security and Authorization
[Discordと同様](https://discord.com/developers/docs/interactions/receiving-and-responding#security-and-authorization)に、TimestampとBodyをバイナリとして結合したものにed25519を用いて署名します。
`X-Signature-Ed25519` HTTP Headerに署名が16進数でエンコードされたものが格納されています。
`X-Signature-Timestamp` にタイムスタンプの数値の文字列が格納されています。
`X-Signature-Timestamp`の値と、HTTP Bodyのバイナリ列を結合し、これを検証してください。
検証失敗時はHttp Status Code 401を返却する必要があります。
Discordの[Community Resourceのinteractions](https://discord.com/developers/docs/topics/community-resources#interactions)の項の一部ライブラリが使用可能です。
### Example
#### JavaScript Example
```sh
npm install discord-interactions
```

```js
const { verifyKey } = require("discord-interactions");

const signature = req.get('X-Signature-Ed25519');
const timestamp = req.get('X-Signature-Timestamp');
const isValidRequest = verifyKey(req.rawBody, signature, timestamp, 'MY_CLIENT_PUBLIC_KEY');
if (!isValidRequest) {
  return res.status(401).end('Bad request signature');
}
```
### Python Examples
```bash
pip install discord-interactions
```

```py
from discord_interactions import verify_key 

if verify_key(request.data, signature, timestamp, 'my_client_public_key'):
    print('Signature is valid')
else:
    print('Signature is invalid')
```
## Webhook Body
Content-Typeは`application/json`です。
未知のtypeのwebhook eventは無視しなければなりません。
| Parameter Name | Parameter Type | Parameter Description                               |
| -------------- | -------------- | --------------------------------------------------- |
| type           | Number         | 1がPING、2がCLAIM UPDATE                            |
| data           | any            | 1ならばundefined。2ならばclaim update eventの配列。 |
### Claim Update Event
| Parameter Name | Parameter Type | Parameter Description                            |
| -------------- | -------------- | ------------------------------------------------ |
| id             | Number         | 請求のid                                         |
| status         | String         | 請求の状態。`approved`または`denied`。           |
| amount         | String         | 請求額                                           |
| updated_at     | String         | 請求の更新日時                                   |
| metadata       | Object         | メタデータ                                       |
| payer          | User           | 被請求者                                         |
| currency       | Currency       | 通貨。ただし、現時点ではtotal_amountを含まない。 |

