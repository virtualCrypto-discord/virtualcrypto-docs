# 警告
他人から渡されたトークンをDescriptionに書かないでください。  
あなたのBotにVirtualCrypto上のアプリケーションが**予期せず紐付けられる**恐れがあります。

# VirtualCryptoのBotの検証
ダッシュボードのアプリケーションページから詳細ページへ移動し、Discord Botと紐付けるボタンを押してください。

## VirtualCryptoのBotの検証の動作
1. VirtualCryptoはセッションに紐づく(セッションはdiscordのユーザーに紐付けられています)、トークンを発行します。
2. VirtualCryptoは指定されたサーバーIDのIntegrationを取得します。
3. 取得したIntegrationの中から指定されたIDのIntegrationを見つけます。
4. 見つけたIntegrationのDescriptionにtokenが含まれていることを確認します。
