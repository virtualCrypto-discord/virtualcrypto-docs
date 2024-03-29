# 通常ユーザー用コマンド一覧

## 読み方について

\[\]で囲っている引数は確実に必要なもの、\<\>で囲っている引数は任意のものです。
 
## /help

ヘルプコマンドです。このページへのリンクや招待URLなどが表示されます。

## /invite

招待URLなどを表示します。

## /pay \[unit\] \[user\] \[amount\]

他のユーザーに通貨を送信します。

パラメータ:

- unit  送信する通貨の単位です。
- user  送信先のユーザーです。
- amount 送信する通貨の量です。

### 例

```
/info unit:v user:@すみどら#8931 amount:100
```

## /info \<name\> \<unit\>

通貨の詳細な情報を表示します。

パラメータ:

- name 調べたい通貨の名前です。
- unit 調べたい通貨の単位です。

### 例

```
/info name:vcoin
```

## /bal

自分が所持している通貨を表示します。

## /claim list \<pending\> \<approved\> \<denied\> \<denied\> \<canceled\> \<user\>

自分に関係する請求の一覧を表示します。
状態でフィルターを掛けない場合`pending`な請求のみが表示されます。
状態でフィルターを掛けると`pending`はデフォルトでは`false`となります。

パラメータ:

- pending 未処理の請求を表示します。
- approved 承認済みの請求を表示します。
- denied 拒否した/されたの請求を表示します。
- canceled キャンセルした/された請求を表示します。
- user あるユーザーに関連する請求のみを表示します。

## /claim received \<pending\> \<approved\> \<denied\> \<denied\> \<canceled\> \<user\>

自分の受け取った請求の一覧を表示します。
状態でフィルターを掛けない場合`pending`な請求のみが表示されます。
状態でフィルターを掛けると`pending`はデフォルトでは`false`となります。

パラメータ:

- pending 未処理の請求を表示します。
- approved 承認済みの請求を表示します。
- denied 拒否した請求を表示します。
- canceled キャンセルされた請求のみを表示します。
- user ある請求元のユーザーを指定し、そのユーザーからの請求のみを表示します。

## /claim list \<pending\> \<approved\> \<denied\> \<denied\> \<canceled\> \<user\>

自分の送った請求の一覧を表示します。
状態でフィルターを掛けない場合`pending`な請求のみが表示されます。
状態でフィルターを掛けると`pending`はデフォルトでは`false`となります。

パラメータ:

- pending 未処理の請求を表示します。
- approved 承認済みの請求を表示します。
- denied 拒否されたの請求を表示します。
- canceled キャンセルした請求を表示します。
- user ある請求先のユーザーを指定し、そのユーザーへの請求のみを表示します。

## /claim make \[user\] \[unit\] \[amount\]

請求を作成します。指定したユーザーが`/claim approve`コマンドを実行すると自分に通貨が支払われます。

パラメータ:

- user 請求したいユーザーです。
- unit 請求する通貨の単位です。
- amount 請求する通貨の量です。

### 例

```
/claim make user:@すみどら#8931 unit:v amount:100
```

## /claim approve \[id\]

請求を承諾します。

パラメータ:

- id 承諾する請求のidです。`/claim list`コマンドで確認できます。

### 例

```
/claim approve id:1
```

## /claim deny \[id\]

請求を拒否します。

パラメータ:

- id 拒否する請求のidです。`/claim list`コマンドで確認できます。

### 例

```
/claim deny id:1
```

## /claim cancel \[id\]

自分が出した請求をキャンセルします。

パラメータ:

- id キャンセルする請求のidです。`/claim list`コマンドで確認できます。

### 例

```
/claim cancel id:1
```

  
  
# 運営用コマンド一覧
以下のコマンドの実行にはサーバーの管理の権限が必要です。

## /give \[user\] \[amount\]

指定したユーザーに発行枠から通貨を発行します。

パラメータ:

- user 発行先のユーザーです。
- amount 発行枚数です。


### 例

```
/give user:@すみどら#8931 amount:1000
```

## /create \[name\] \[unit\] \[amount\]

新しい通貨を発行します。運営用のプールの枚数はamountの0.5%になります。

パラメータ:

- name 通貨の名前です。
- unit 通貨の単位です。
- amount 通貨の最初の発行枚数です。発行者に付与されます。

### 例

```
/create name:vcoin unit:v amount:1000000
```


## /delete 

作成した通貨を削除します。通貨の作成後72時間実行可能です。

### 例

```
/delete
```
