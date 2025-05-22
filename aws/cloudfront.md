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
<br><br>

## 1. 配信最適化系の機能
### ■ 最適なエッジロケーションへの誘導
エンドユーザのリクエストは、もっとも小さいレイテンシのエッジロケーションにルーティングされる。
![image](https://github.com/user-attachments/assets/97d1dee7-e77b-4a38-bc0f-531b7ec5fc5c)
<br><br>
### ■ キャッシュ機能
オリジンから取得したファイルをエッジロケーションでキャッシュ。
- 階層的キャッシュ構造（エッジ + リージョナルエッジキャッシュ）
- TTLやキャッシュポリシーによりキャッシュの有効期間を制御
### ■ AWSグローバルバックボーンネットワーク
CloudFront - オリジン間はAWS専用のグローバルバックボーンネットワークを利用。
- パブリックネットワークに比べて高帯域・低レイテンシ
### ■ エッジでのコンテンツ圧縮
オリジンから取得した非圧縮コンテンツをエッジで圧縮してキャッシュ可能。
- gzip / Brotli 圧縮方式
- 対象：HTML, CSS, JavaScript, JSON, XML などのテキストファイル
- ブラウザが `Accept-Encoding: gzip, br` を送ってきた場合に適用
- デフォルトでは無効（手動で有効化が必要）
### ■ Price Class
利用するエッジロケーションを制限することでコスト削減が可能。
- Price Class 100：北米・欧州（最も安い）
- Price Class 200：100 + アジア（中程度）
- Price Class All：全世界対応（最も高性能）
<br><br>
## 2. セキュリティ・アクセス制御系の機能
### ■ HTTPS通信対応（SSL/TLS）
- 自動HTTPS（SNIベース）対応
- ACM（無料DV証明書）やカスタム証明書（OV/EV）対応
- セキュリティポリシーによる通信要件の制御
  - Viewer Protocol Policy（ユーザー → CloudFront）
  - Origin Security Policy（CloudFront → オリジン）
### ■ オリジンアクセスコントロール (OAC)
S3への直接アクセスを防ぎ、CloudFront経由のみでアクセスさせる。
- CloudFrontがSigV4署名付きリクエストを送信
- S3は署名検証 + IAMベースの認可で制御
- CloudFrontサービスプリンシパルに基づいたバケットポリシー記述
- ディストリビューション単位でのアクセス制御が可能
### ■ 署名付きURL/Cookieによるアクセス制御
認証済みユーザのみに限定コンテンツを配信可能。
- アプリ側で署名付きURL/Cookieを発行し、CloudFrontが検証
- 有効期限やIP制限などの条件も指定可能
- 公開鍵はKey GroupとしてCloudFrontに登録
### ■ 地理的制限（Geo-restriction）
国レベルでのアクセス制御が可能。
- CloudFrontの標準機能で実現
- 詳細な地理制限はサードパーティの位置情報サービスと併用可
<br><br>
## 3. 可用性・耐障害性機能
### ■ オリジングループによるフェイルオーバー
複数のオリジンをグループ化し、プライマリ障害時にセカンダリへ自動ルーティング。
- ヘルスチェックベースの自動切り替え
- オリジンステータスが失敗（例：5xx）と判断されたときのみ発動
<br><br>
## 4. エッジ処理・カスタマイズ系機能
### ■ Lambda\@Edge / CloudFront Functions（CFF）
CloudFrontリクエスト/レスポンスの各タイミングでコード実行が可能。
#### Lambda\@Edge
- Viewer/Origin の Request/Response タイミングで実行可能
- Node.js, Pythonランタイム、外部ライブラリも利用可
- 複雑な認証・署名検証・外部API呼び出しなどに対応
#### CloudFront Functions
- Viewer Request/Response のみ対応
- JavaScriptベース、超高速軽量
- 軽微な処理（UA判定、リダイレクト、ヘッダ追加など）に最適
<br><br>

## 5. 運用管理・モニタリング系機能
### ■ 標準アクセスログ
ディストリビューションごとにS3バケットへ配信ログを保存できる。
- 各リクエストごとの詳細（IP、User-Agent、ステータスコード、バイト数など）を確認可能
- 数分〜数十分の遅延あり
### ■ リアルタイムログ
Kinesis Data Streamsと連携してリアルタイムでリクエストログを取得
- サブミリ秒単位の分析やセキュリティ監視に対応
- 有料オプション
### メトリクス監視（CloudWatch）
CloudWatchに標準メトリクスを送信可能
- キャッシュヒット率、エラー数、転送量など
- アラームとダッシュボードによる監視が可能
### アクセス制限レポート
WAFを併用して、IPブロックやGeo制限のトラフィックを分析可能。<br>
WAFログやログ分析ツールと組み合わせて攻撃傾向の可視化も可能
### ■ フィードレベル暗号による保護
### ■ AWS WAF によるコンテンツ保護
<br><br>

## CloudFrontの基本コンポーネント
### ■ Viewer
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
CloudFrontの「リクエスト仕分け＆処理ルール」。
リクエスト毎に「どう返すか／どこに行かせるか／キャッシュさせるか」を全部決める（※レスポンス時のルールではない）



## 参考資料・引用
[ Amazon CloudFront deep dive [AWS Black Belt Online Seminar]](https://d1.awsstatic.com/webinars/jp/pdf/services/20201028_BlackBelt_Amazon_CloudFront_deep_dive.pdf)
