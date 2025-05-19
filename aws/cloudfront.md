# CloudFrontとは
AWSのフルマネージドCDNサービス（グローバルな多層プロキシサービス）
![image](https://github.com/user-attachments/assets/7cf027b9-a39a-4aa2-8dfe-9a01d5b18ed7)
<br><br>
- グローバルサービス：
- エッジロケーションサービス（配信高速化、コンテンツキャッシュ、リージョナルエッジキャッシュ）
- 動的コンテンツもサポート（ストリーミングデータ、APIレスポンス、データベースクエリに基づくデータ、ユーザーごとに異なるHTMLページなど）
- 多様なオリジンをサポート（S3バケットをオリジン、それ以外をカスタムオリジンとして設定可能）
- HTTPヘッダーの付加・変更やCookieのフォワードのよる動的配信のサポート（標準ヘッダー、カスタムヘッダー）
- LambdaEdge：エッジロケーションでカスタムコードを実行し、リクエストおよびレスポンスを動的に処理（かなり色々できる
- ログ取得：アクセスログ（S3出力）やリアルタイムメトリクス（CloudWatch）取得可能

## AWSエッジサービス
地理的にユーザーに近いエッジロケーションでリクエストの処理・転送・キャッシュ・制御などを行うことで、レイテンシの削減やネットワークパフォーマンスの向上を図るAWSのサービス群。
<br><br>
![image](https://github.com/user-attachments/assets/2161ddde-4922-4e3a-9185-2de9f279cb1e)
<br><br>

## CloudFrontの基本機能
### ■ 階層的キャッシュ構造（エッジ + リージョナルエッジキャッシュ）
### ■ AWSサービスのネイティブ統合
### ■ AWSグローバルバックボーンネットワーク
### ■ 




### ■ HTTPS通信対応（SSL/TLS）
自動でHTTPS対応（SNIベース）＋ ACMや独自証明書の利用可能
### ■ CDN機能群
- CDN（エッジロケーション）：ユーザに近いエッジロケーションから直接コンテンツ配信
- キャッシュストレージ：オリジンの応答をエッジでキャッシュして再利用
- キャッシュビヘイビア：パス・クエリ・ヘッダ・Cookie単位でキャッシュ戦略を調整
- コンテンツ圧縮：コンテンツをエッジ側で自動圧縮し、転送量削減
- AWSグローバルバックボーン：CloudFrontはAWS内部ネットワークを活用し、インターネット経由より安定＆高速な伝送経路を確保
- オリジングルーピング＆Failover：最適なオリジンに振り分けたり、障害時の代替オリジンでスピード維持
### ■ エッジ処理
CloudFront Functions や Lambda@Edge によって、エッジでのレスポンス操作・認証処理などが可能
### ■ セキュリティ統合
AWS WAF（L7ファイアウォール）、Origin Access Control（OAC）、署名付きURL/Cookieによる制限
<br><br>

## CloudFrontの基本コンポーネント
### ■ ディストリビューション（Distribution）
CloudFrontの配信管理単位。<br>
どのコンテンツを、どのオリジンから、どういうルールで、どこに届けるかを一括で管理する設定のかたまり。<br>
通常、1サイト or 1APIごとに1ディストリビューションを作成する。
### ■ オリジン（Origin）
CloudFrontがコンテンツを取得するバックエンドの場所。<br>
S3、ALB、EC2、API Gateway、または外部のHTTPサーバーなどを指定可能。
### ■ オリジングループ（Origin Group）
オリジンの冗長化（フェイルオーバー）構成。<br>
プライマリのオリジンがダウンしたとき、自動的にセカンダリへ切り替えて配信を継続。
### ■ キャッシュビヘイビア（Cache Behavior）
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

- [ランダム⽂字列].cloudfront.net がドメイン名として割り当てられる
- CNAMEエイリアスを利⽤して代替ドメイン名の指定が可能
- ACM連携によるTSL/SSL証明書サポート
  - 事前定義されたセキュリティポリシー（カスタム定義不可）
- HTTP/1.0, HTTP/1.1, HTTP/2, WebSocket 対応
- AWS WAF連携と連携可能
- CloudWatchと連携したモニタリングやアラーム
  - メトリクス：リクエスト量、転送量、キャッシュヒット率、オリジンレイテンシ、ステータスコードエラー率など。。 






<br><br>

# Origin

# Behavior


## 参考資料・引用
[ Amazon CloudFront deep dive [AWS Black Belt Online Seminar]](https://d1.awsstatic.com/webinars/jp/pdf/services/20201028_BlackBelt_Amazon_CloudFront_deep_dive.pdf)
