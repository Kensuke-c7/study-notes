# AWSグローバルインフラストラクチャ（Global Infrastructure）

## ■リージョン：AWSサービスの地理的提供範囲（国や地域単位）
- 基本的にAWSサービスはリージョン毎に独立して提供される
- リージョン同士は地理的に離れている
- １つのリージョンは複数のAZで構成される
- リージョン間は高速・高帯域でネットワーク接続されている
  - リージョン間データ転送料金：0.09USD/GB	
<br><br>

## ■AZ（アベイラビリティゾーン）：リージョン内の独立したデータセンター群。
- １つのAZは１つ以上のデータセンタで構成される。
- 同一リージョン内の他のAZから論理的に隔離され低レイテンシのネットワークで接続されている。
- AZ内の各データセンタは数十キロ単位で地理的に離れている。
- AZ間通信料金：各方向に0.01USD/GB
<br><br>

## ■エッジロケーション：ユーザーに近い場所に配置されたデータセンター
- 低レイテンシー、高スループットを目的にユーザーに近い場所に配置されたデータセンター（CloudFront、AWS WAF、Global Accelaraorなどで利用）
- 主な用途はキャッシュ配信（CloudFront）やDNS応答高速化（Route53）など。 
<br><br>

## ■ローカルゾーン：特定の都市に近い場所に配置されたAWSのインフラストラクチャ
- 特定の都市に近い場所に配置されたAWSのインフラストラクチャで、低レイテンシーの要件を持つアプリケーションやデータ処理を支援するためのもの。
- 親リージョンと接続されていて、基本的な制御・管理は親リージョン側で行う
<br><br>
<br><br>

# VPC（Amazon Virtual Private Cloud）
AWS上にプライベート（自分専用）な仮想ネットワーク環境を作成するサービス。<br>
VPC＝１個の巨大な仮想ルータで、サブネット＝仮想ルータのI/Fのイメージ（サブネット間はルート設定せずにルーティングが通る）


### ■VPCの特徴
- VPCに対してIPアドレスレンジ（プライマリCIDRブロック）を割り当てる
  - プライマリCIDRブロックは作成後の変更不可（セカンダリCIDRブロックを追加能）
  - プライベートIPアドレス：VPC内でのみ有効なIPアドレス。VPC内のAWSリソースには、パブリックサブネット・プライベートサブネットに関係なく、**すべてプライベートIPアドレスが自動で割り当てられる**
  - パブリックIPアドレス：インターネット接続可能なIPアドレス。インスタンスをパブリックサブネットに作成するときに自動で割り当てされる。
  - Elastic IPアドレス：固定パブリックIPアドレス。AWSアカウントと紐づいており、AWSリソースに手動で割り当てる
    - 割り当てたElastic IPは AWSリソースが停止・起動しても（IPアドレスは）変わらない
    - 割り当てたAWSリソースが終了しても同一のIPアドレスが保有されるので利用料金が発生し続ける
    - 保有しているElastic IPを使用して、同一IPアドレスを別のAWSリソースに再び割り当てできる
- VPCは同一リージョンに複数作成可能（デフォルト最大５）
- VPCは同一リージョン内の全AZに跨る
- VPC同士で接続できる（デフォルトでは不可）
- ルートテーブルでルーティングを行う
- リソースにはENIをアタッチしてネットワークインターフェース機能を付加する
<br><br>

## VPC基本機能
### ■ 専用仮想ネットワーク機能
VPCを複数のサブネットに分割し、VPC内やVPC外（インターネット等）へのルーティングが出来る。
### ■ ネットワークサービス
- DHCP：サブネット内のENIにプライベートIPを自動割り当て
- DNS：VPC内の名前解決を行うDNSリゾルバや、ENIに対するDNS名の自動割当を提供
- NTP：Amazon提供の時刻同期サービスを利用可能（VPC内で169.254.169.123）
- NAT：NAT GatewayやNATインスタンスで、プライベートサブネットからのインターネットアクセスを提供
### ■ ネットワークセキュリティ
- SG（セキュリティグループ）：ENIにアタッチするステートフルファイアウォール
- NACL（ネットワークACL）：サブネット単位で制御するステートレスファイアウォール
- VPC Lattice：VPC内外にまたがるサービス間通信の中継＆制御（認証・通信暗号化）
### ■ エンドポイントサービス（VPC Endpoint）
AWSサービス（S3など）に対し、インターネットを経由せずにプライベートアクセスを提供するエンドポイント
- Gateway型：S3やDynamoDB専用のルートターゲット型エンドポイント
- Interface型：サブネット内にENIとして設置される汎用エンドポイント
### ■ オンプレとのプライベート接続（閉域接続）のサポート
- VPN接続：オンプレミスとVPCをIPsecトンネルで接続（AWS VPN）
- Direct Connect：オンプレミスとAWSを専用線で接続
### ■ VPC間接続のサポート
- VPC Peering：2つのVPC間でプライベートルーティングを可能にする（1対1接続）
- AWS PrivateLink：VPC間でサービスを公開・利用するためのエンドポイントサービス
- Transit Gateway：複数のVPC、VPN、Direct Connect間でトラフィックをハブ型に中継
### ■ 運用管理
- VPC Flow Logs：ENIを通過するトラフィック情報をCloudWatch LogsまたはS3に出力
- VPC Traffic Mirroring：ENIのトラフィックを他のENIにミラーリング
<br><br>

## VPCコンポーネント
### ■ サブネット：VPCをCIDR分割したもの（最小/16 ～ 最大/28）
- サブネットは１つのAZにのみ含まれる（**複数のAZに跨ることはできない**）
- サブネットに対して1つのルートテーブルを関連付ける必要がある（デフォルトでメインルートテーブルが関連づけられる）
- ２つのサブネットタイプ
  - パブリックサブネット：インターネットゲートウェイへの直接ルートがあるサブネット
  - プライベートサブネット：インターネットゲートウェイへの直接ルートがないサブネット
- サブネット毎に１つのVPCルータ（CIDR +1 アドレス）を持つ。
- IPv4、IPv6 対応のサブネット作成可能（デュアルスタック可）
- NACLでネットワークアクセス制御ができる（アタッチ必須）
### ■ ルートテーブル：サブネット単位のルーティングテーブル
ＶＰＣはルートテーブルを使ってルーティングする。VPC内通信（同一VPC内のサブネット間通信）はデフォルトでＯＫだが、**VPC外通信（インターネット、オンプレ、他VPC）はルートテーブルへのルート設定が必要**。ルートは送信先（宛先ＣＩＤＲ）に対してターゲット（IGW、VGW、NGWなど）を指定する。
- メインルートテーブル：VPC作成時に自動作成される。**明示的な関連づけのない全てのサブネットに関連づけされる。**
- カスタムルートテーブル：VPC作成後に手動で作成するテーブル。
- サブネットルートテーブル：サブネットに関連づけされたルートテーブル。
- ルーティング優先度
  - 1.ロンゲストマッチ
  - 2.静的ルート
  - 3.プレフィックスリストルート
  - 4.伝搬されたルート（DirectConnect BGPルートなど）
### ■ ENI（Elastic Network Interface）：VPC内通信のエンドポイント
AWSにおける仮想NIC。VPC内のリソースはENIをアタッチすることでIP通信が可能になる。
- IPアドレス（プラマリ、セカンダリ、パブリック）、MACアドレス、内部DNS名を持ち、**VPC内は内部DNS名を用いて通信する**
- **VPC内リソース間通信の通信エンドポイントになる（IPレイヤレベルではENI同士で通信している）**
- 単一のインスタンスに複数のENIをアタッチ可能（同一AZのENIに限る）
- **1つ以上のセキュリティグループを必ずアタッチする**
- 別リソースへの付け替え可能（障害復旧や冗長構成に利用）
- **VPCフローログはENI単位で出力される**
### ■ SG（セキュリティグループ）：ENIにアタッチするステートフルファイアウォール
- ステートフルなファイアウォール機能を提供
- IN/OUT方向で指定可能
- 許可ルールのみ設定できる（暗黙のDeny）
- 送信元・宛先にSGを指定可能（指定SGにアタッチされている全ENIが対象）
- ENIにアタッチすることで、EC2、RDS、ALB、Lambda（VPC配置時）などのリソース通信を制御（リソース本体に直接アタッチするわけではない）
- → **アタッチされたリソース本体ではなくENIが制御対象**
### ■ NACL（ネットワークアクセスコントロール）：サブネットのアクセスリスト
- サブネットには必ず1つのNACLがアタッチされる（デフォルトは全許可）
- ステートレス（インバウンドとアウトバウンドをそれぞれ明示的に許可しないと遮断される）
- ルール番号順に評価され、最初にマッチしたルールが適用される
### ■ Amazon Route53 Resolver：サブネット内のDNSリゾルバ
- VPCネットワークアドレス +2 がDNSクエリエンドポイントとなる
- enableDnsSupport：VPC内でAmazonProvidedDNSを利用できるかどうかを制御する（デフォルト有効）
- enableDnsHostnames：EC2インスタンスにDNSホスト名（パブリックDNS名）を割り当てるか制御する（デフォルト無効、ただしDefault VPCでは有効）
- サブネットのCIDRブロックの+2番目のIPアドレスがリゾルバアドレスとして提供される（例：10.0.0.0/24なら10.0.0.2）
### ■ インターネットゲートウェイ（IGW）：インターネット接続用
- デフォルトルート（0.0.0.0/0）のターゲットとして指定
- VPC単位でアタッチする
- ルーティング機能のみを提供（グローバルIPへのSNATはしない）
  - ただし、**IGWでプライベートIPとグローバルIPのマッピング**はしている 
- グローバルIP（Elastic IPなど）を持つENIが必要（プライベートIPのみではインターネット通信できない）
### ■ NATゲートウェイ（NGW）：NAT機能を持つマネージドサービス
- プライベートサブネットのインスタンスからインターネットに出るためのターゲット
- サブネット単位で配置する
  - パブリックNATゲートウェイ：パブリックサブネットに配置し、Elastic IPをアタッチする
  - プライベートNATゲートウェイ：プライベートサブネットに配置（インターネット通信不可、他VPC接続用）
- アウトバウンド通信専用（インバウンド通信のNATは非対応）
- AWSフルマネージドサービス（AZ内で冗長化） ※AZ間冗長は利用者側で設定必要
### ■ バーチャルプライベートゲートウェイ（VGW）：AWS以外との接続用
- VPN接続やDX接続のエンドポイントとなる（リモートネットワークからパケットをVPCへルーティング）
- 設定項目はAS番号のみ 
- リージョン内に複数作成可能（**ただしVPCにアタッチできるVGWは1つのみ**）
  - VGW間で同一ASNでもOK（VGW同士が独立したBGPプロセスかつVGW同士でピアを貼らない） 
- いつでもVPCの付け替え可能（アタッチ先のVPCの切替）
- VPCとデタッチ状態でも対向のエンドポイントと経路交換可能
### ■ Gatewayエンドポイント
- ゲートウェイとしてVPCにアタッチし、ルートテーブルのターゲットとして指定
- s3, DyanamoDBのみサポート
- エンドポイントポリシーでアクセス制御
- VPC外から利用できない（IPアドレスを持たないためルート設定ができない）
### ■ インタフェースエンドポイント
- エンドポイントとしてENIをサブネットに配置し、ENIのプライベートIP宛を通じてアクセス
- 内部DNS名（プライベートIP）でアクセスし、SGでアクセス制御
- AZで冗長化が必要（サブネットに配置するため）
- VPC側から利用可能（内部DNS名を解決できれば）
<br><br>
