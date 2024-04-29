## 手動設定

Terraformなどで自動化できていない設定手順を記載する。
新規環境を作成するときにのみ実行が必要。

### DNSの初期設定

ネームサーバーには Cloudflare を使用します。（一番の理由はワイルドカードのSSL証明書の利用が簡単なため）
以下の２つのNSレコードを Cloud Domains などで Custom DNS として設定します。

```
lakas.ns.cloudflare.com
luciane.ns.cloudflare.com
```

Cloudflareのコンソールで、上記で確認した静的IPに対し、以下のAレコードを設定します。（dev環境の場合）
Cloudflare ProxyはONにしておきます。
 
- chatforce-dev.com
- *.chatforce-dev.com
- *.my.chatforce-dev.com

SSL/TLSのメニューを開き、encryption modeをFullにします。

### Cloud Run のドメインマッピング

Cloud Run のコンソールでEnvoyに対してカスタムドメインの設定を行います。
`api.dev.chat-force.dev` などを登録する。
 
CloudRunにカスタムドメインを設定するために、webmasterで所有権を確認した後、domainオーナーとして、terraformのサービスアカウントを追加
    - 所有権の確認には、DNSでTXTレコードを登録する
    - https://www.google.com/webmasters/verification/details

### Rewrite Response Code の設定

- CloudFlare の Workers ルート から `rewrite-response-code` のスクリプトを設定し、404を200にrewriteするようにする

### GCP/Firebase の設定

- Firebase console で、Blocking function の設定
- Identity Platform でマルチテナントを有効化
- Firebase console から JSONを取得して、userとservice-adminの `.env-cmdrc.json` に追記
- Cloud Functions のデプロイ後、各Secretがデプロイ時のバージョン固定になるので、GCP のコンソールからlatestにしてデプロイしなおす。
- Authentication -> Template
  - 送信元のドメインをカスタマイズ -> 所有権の確認に必要なレコードをcoruscant.co.jpのDNSに登録

### Stripe の設定

- stripeWebhook のデプロイ後、そのURLをStripeのWebhookに設定する
  - customer と subscription のイベントをすべてサブスクリプションする（そんなにいらないかも）
- Standard, Premium の商品を作成する

### Azureの設定

- Azure OpenAI の Resourceを作成
- 以下のDeploy を作成（名前で参照されるので完全に同じ名前にする必要あり）
  - gpt-35-turbo
  - gpt-4

### その他各種設定

- 管理対象GCPプロジェクトでterraform@ というサービスアカウントを作成して、Ownerロールを与える。JSONをダウンロードし、GOOGLE_CREDENTIALSという名前でTerraform Cloud の Sensitive env var (Terraform Variable) として保存する。
- ci用のサービスアカウントのJSONを取得して、GithubのSecretに追加
