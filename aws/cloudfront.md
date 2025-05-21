# CloudFrontとは
AWSのフルマネージドCDNサービス（グローバルな多層プロキシサービス）
![image](https://github.com/user-attachments/assets/7cf027b9-a39a-4aa2-8dfe-9a01d5b18ed7)
<br><br>
- エンドユーザを最寄りのエッジロケーションに誘導し、高速・低遅延な配信を実現
- エッジサーバでコンテンツのキャッシュや圧縮による配信効率化とオリジンサーバの負荷軽減
- パスやHTTPヘッダなどの条件に応じて、複数のオリジン間でルーティング（動的オリジン選択）
- ヘッダの追加・変換・削除により、キャッシュ分割、認証連携、CORS対応など柔軟な制御が可能
- エッジでJavaScriptコードを実行し、動的にリクエストやレスポンスを操作可能（Lambda@Edge、CloudFrontFunctions）
- HTTPSによる通信暗号化（SSL/TLS）および、機密データのField-Level Encryptionに対応
- 署名付きURL/Cookie、オリジンアクセス制御（OAC）、カスタムヘッダ、Geo制限など多様なアクセス制御機能に対応
- AWS WAF連携によるWebアプリ保護、およびAWS ShieldによるDDoS対策をネイティブにサポート
- S3への標準アクセスログ出力、CloudWatchによるリアルタイムメトリクス、アラーム監視に対応

## CloudFrontの基本機能
### ■ エッジロケーションへの誘導
エンドユーザのリクエストはもっと適切なエッジロケーション（通常は最も小さいレイテンシ）にルーティングされる。
![image](https://github.com/user-attachments/assets/97d1dee7-e77b-4a38-bc0f-531b7ec5fc5c)
<br><br>
### ■ 階層的キャッシュ構造（エッジ + リージョナルエッジキャッシュ）
### ■ AWSサービスのネイティブ統合
### ■ AWSグローバルバックボーンネットワーク
## コンテンツの保護とアクセス制限
### ■ HTTPS通信対応（SSL/TLS）
- 自動でHTTPS対応（SNIベース）＋ ACM（DV証明書）や独自証明書（OV、EV）を利用可能
- 事前定義されたセキュリティポリシーを利用
  - ユーザ - CloufFront間のセキュリティポリシー（Viewer Protocol Policy）
  - CloufFront ｰ オリジン間のセキュリティポリシー（Origin Security Policy）
### ■ 地理的制限（Geoブロッキング）
コンテンツに対して特定地域のユーザーのアクセス拒否ができる。
- CloudFrontの地理的制限機能：ディストリビューションに関連するすべてのファイルへのアクセスを制限。国レベルでアクセスを制限する場合に利用。
- サードパーティーの位置情報サービス：国レベルより詳細なレベルでアクセスを制限が可能。
### ■ 署名付きURL/Cookieによるアクセス制御
CDNサービスで一般的な署名付きURLおよび署名付きCookieによるアクセス制御に対応している。
これにより、認証済みユーザのみに限定コンテンツを配信することが可能となる。
- アプリケーション（認証サーバ）側で署名付きのURLまたはCookieを生成し、ユーザに付与する
- CloudFrontは事前に登録された公開鍵（Key Group）を使用して署名の正当性を検証
- 有効期限（開始時刻・終了時刻）や、アクセス元IPアドレスなどの制限条件を署名に含めることで、柔軟なアクセス制御が可能
### ■ オリジンアクセスコントロール (OAC) によるアクセス制御
S3へのアクセスをCloufFront経由に制限する機能。
OACを利用することで、バケットポリシーよりも柔軟なアクセス制御とセキュリティ強化ができる。
- IAMと連携してS3オリジンと認証を行う
- 包括的な HTTP メソッドのサポート



### ■ フィードレベル暗号による保護
### ■ AWS WAF によるコンテンツ保護




<br><br>

## CloudFrontの基本コンポーネント
### Viewer
Webブラウザなどのクライアント
### ■ ディストリビューション（Distribution）
CloudFrontにおけるコンテンツの配信管理単位。<br>
どのコンテンツを、どのオリジンから、どういうルールで、どこに届けるかを一括で管理する設定のかたまり。<br>
通常、1サイト or 1APIごとに1ディストリビューションを作成する。
### ■ オリジン（Origin）
コンテンツの配信元。<br>
S3、ALB、EC2、API Gateway、または外部のHTTPサーバーなどを指定可能。
### ■ オリジングループ（Origin Group）
オリジンの冗長化（フェイルオーバー）構成。<br>
プライマリのオリジンがダウンしたとき、自動的にセカンダリへ切り替えて配信を継続。
### ■ ビヘイビア（Behavior）
リクエストパス（例：/images/*）ごとの処理ルールを定義。<br>
以下のような設定が可能：
- どのオリジンへルーティングするか
- キャッシュキー（ヘッダー、クエリ、Cookieなど）
- CORSやHTTPSの強制などのオプション
### ■ エッジロケーション（Edge Location）
CloudFrontがコンテンツをキャッシュして配信する拠点（データセンター）。<br>
アベイラビリティゾーン（AZ）とは別で、よりユーザーに近い場所に配置されている。<br>
現在、400以上のエッジロケーションと、**13のリージョン別キャッシュ（Regional Edge Caches）**が存在。
- CloudFrontやRoute53などのグローバルサービスで利用
- AWS Global Acceleratorの通信経路上にも利用されることがある
<br><br>

# Distribution
![image](https://github.com/user-attachments/assets/446aa5b9-db76-4630-a6be-b35646a4be70)
<br><br>
- [ランダム⽂字列].cloudfront.net がドメイン名として割り当てられる
- CNAMEエイリアスを利⽤して代替ドメイン名の指定が可能
- ACM連携によるTSL/SSL証明書サポート
  - 事前定義されたセキュリティポリシー（カスタム定義不可）
- HTTP/1.0, HTTP/1.1, HTTP/2, WebSocket 対応
- AWS WAFと連携可能（Web Application Security）
- CloudWatchと連携したモニタリングやアラーム
  - メトリクス：リクエスト量、転送量、キャッシュヒット率、オリジンレイテンシ、ステータスコードエラー率など。。 
<br><br>

# Origin
![image](https://github.com/user-attachments/assets/2bca5dfd-3136-443e-922a-e1344f07adb1)
<br><br>
- S3オリジンとカスタムオリジン（s3以外のオリジン）
- オブジェクトをエッジにキャッシュして高速配信
- HTTPSでCloudFrontと通信可能（TLS1.2など）
- CloudFrontの機能として同様にモニタリング・ログ出力可能

### S3オリジン
S3バケットをオリジンとしたもの。他のオリジンと異なりS3向けに最適化（独自機能と制限）。
- S3の内部APIを使って、効率よくファイルを取得（AWSファミリーだから特別扱いでAPIで取りに行く）
- 静的オブジェクト（画像・HTML等）向けに最適化
- 独自のアクセス制御機能（署名付きURL、OAC、s3バケットポリシー）
- 限定的なHTTPメソッド（PUT/POSTなどは制限）
- ヘルスチェック機能なし
- ヘッダ制御の自由度が低い（CloufFront側で自動制御）
### カスタムオリジン
S3以外のオリジン（ALB、EC2、API Gateway等）で通常のHTTPサーバとしての扱い。
- 通常のHTTP/HTTPSでGETなど送信（HTTPサーバだから普通にGET投げる）
- 動的コンテンツやAPI応答にも使える
- GET/POST/PUT/DELETE など幅広く対応
- ヘッダ制御の自由度が高い
  - リクエストのHTTPヘッダ操作やカスタムヘッダ挿入等の柔軟な制御ができる
  - レスポンスヘッダをオリジンサーバ側で自由に設定できる
- 独自のアクセス制御機能なし
- ヘルスチェック機能なし



# Behavior


## 参考資料・引用
[ Amazon CloudFront deep dive [AWS Black Belt Online Seminar]](https://d1.awsstatic.com/webinars/jp/pdf/services/20201028_BlackBelt_Amazon_CloudFront_deep_dive.pdf)
