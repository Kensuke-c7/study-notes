# Direct Connect（DX）：専用線を使った閉域接続サービス
利用者がキャリアから調達する専用線の片端とAWSCloudをDirectConnectロケーションで接続するサービス。
![image](https://github.com/user-attachments/assets/5ddf43c3-eb1e-4f7d-8d86-c5d10364b5f3)


#### 特徴・機能
- 専用線（高帯域、低遅延・少ジッタ、高セキュリティ）
- デリバリーパートナー（通信キャリア）の回線サービスを経由して接続する
- サービス提供形態
  - 専用接続：物理的に一つの回線（ポート）を占有して使用する形態
    - 1Gbps／10Gbps／100Gbpsから選択
    - 複数のVIFを作成可能
  - Hosted接続：DirectConnectパートナーの物理的な回線を複数のユーザで共有する形態（クロスアカウント利用）
    - Connectionを所有しているAWSアカウントから他のAWSアカウントに対してVIFを提供することが可能
    - 利用帯域：50Mbps／100Mbps／300Mbps／500Mbps／1Gbps
    - 利用可能なVIFは1つ
- BGPを利用して接続
-  

#### ■ 仕組み(コンポーネント)
- DirectConnectロケーション：オンプレミスとAWSデータセンターとの接続ポイント（DirectConnectパートナーやAWSが提供する物理的なデータセンター）
- DirectConnect接続：物理的な専用線接続
- DirectConnectエンドポイント：DirectConnectロケーション内に設置されたAWS側ネットワーク機器。利用者側の回線を終端する。
- 仮想インタフェース（VIF）：
- DirectConnectGateway（DXGW）：DirectConnectエンドポイントから複数のVPCに接続したい場合に利用する機能。DXGWを介して複数VPCと相互接続できる。
<br><br>

## 仮想インタフェース（VIF）
Connection（接続）を通してAWSリソースにアクセスするための論理インターフェイス。
- AWSとカスタマルータの間でBGPピアを確立し経路交換をするために必要
- VLAN IDをもつ
- 利用目的に応じたVIFのタイプを選択する（プライベート、パブリック、トラジット）
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


  
  
