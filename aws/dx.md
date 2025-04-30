# Direct Connect（DX）：専用線を使った閉域接続サービス
利用者がキャリアから調達する専用線の片端とAWSCloudをDirectConnectロケーションで接続するサービス。

![image](https://github.com/user-attachments/assets/5ddf43c3-eb1e-4f7d-8d86-c5d10364b5f3)


#### 特徴・機能
- 専用線（高帯域、低遅延・少ジッタ、高セキュリティ）
- デリバリーパートナー（通信キャリア）の回線サービスを経由して接続する ※DXロケーション内での直接接続も可
- 帯域：50Mbps～100Gbps
- BGPを利用して接続

#### ■ 仕組み(コンポーネント)
- DirectConnectロケーション：AWSの物理ルータ（DXエンドポイント）が設置してある、データセンター事業者のデータセンター
- DirectConnect接続：物理的な接続
- DirectConnectエンドポイント：DirectConnectロケーション内に設置されたAWS側ネットワーク機器。利用者側の回線を終端する。
- 仮想インタフェース（VIF）：
- DirectConnectGateway（DXGW）：DirectConnectエンドポイントから複数のVPCに接続したい場合に利用する機能。DXGWを介して複数VPCと相互接続できる。
<br><br>

## DirectConnect接続：物理接続
物理的な専用線接続。回線の利用形態により２つの接続タイプ。
### Dedicated接続（占有接続）
物理的に一つの回線（ポート）を占有して使用する形態。利用者自身が回線を手配する。
- 利用者自身が物理接続を手配（コロケーション内の利用者側機器～DXエンドポイント間の回線敷設） 
- 帯域：1Gbps／10Gbps／100Gbpsから選択
- 複数のVIFを作成可能
<br><br>
### Hosted接続（ホスト型接続）
DirectConnectデリバリパートナー（通信キャリア）を通じてDXを利用する形態。
- パートナーが所有する物理接続を複数のユーザで共有（論理分割）する形態（利用者側で回線敷設は不要）
- Connectionを所有しているAWSアカウントから利用者側のAWSアカウントに対してVIFを提供（クロスアカウント利用）
- 利用帯域：50Mbps／100Mbps／300Mbps／500Mbps／1Gbps
- ユーザ側で利用可能なVIF（HostedVIF）は1つ
<br><br>

## 仮想インタフェース（VIF）：BGP接続用の仮想インタフェース
Connection（接続）上でBGPピアを確立するための論理インターフェイス。IPアドレスを持つVIFをルータ（ゲートウェイ）にアタッチして利用するイメージ。
- 利用目的に応じた３つのVIFタイプを選択（プライベート、パブリック、トラジット）
- VIF作成時に必要な情報（VIF共通）
  - VIFを作成する物理接続（DX接続） 
  - アタッチ先のゲートウェイ（※パブリックVFI除く）
  - AS番号（アタッチ先のゲートウェイで設定）
  - カスタマー側AS番号
  - VLAN ID
  - IPアドレス（利用者側、カスタマー側双方）
  - BGP MD5シークレット


  - プライベートVIF：VPCへプライベートIPを介した接続を提供
  - パブリックVIF：AWSの全リージョンへパブリックIPを介した接続を提供
  - トランジットVIF：Transit Gateway用のDirect Connectゲートウェイへ接続を提供
    - DirectConnect 1/2/5/10Gbps接続でのみ作成可能（**パートナーによる共有型接続では利用できない**）
ｰ クロスアカウント利用：Connectionを所有しているAWSアカウントから、他のAWSアカウントに対してVIFを提供することが可能
  - データ転送料については、各アカウントに課金

### プライベート接続：Direct Connectゲートウェイ(DXGW)タイプ(推奨)

![image](https://github.com/user-attachments/assets/0da1f15e-073a-4e95-8708-5ead25832338)

- プライベートVIFを使用して複数のVPCへ接続を提供
- お客様ルータでBGP, MD5認証, IEEE802.1q VLANのサポートが必要
- VPCのCIDR(IPv4,IPv6)がすべてAWSから広告される（フィルタリング可能）
- Jumbo Frame(MTU=9001)をサポート

### パブリック接続
### トランジット接続


## AWS Direct Connectゲートウェイ


  
  
