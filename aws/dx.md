# AWS Direct Connect（DX）：専用線を使った閉域接続サービス
利用者がキャリアから調達する専用線の片端とAWSCloudをDirectConnectロケーションで接続するサービス。

![image](https://github.com/user-attachments/assets/5ddf43c3-eb1e-4f7d-8d86-c5d10364b5f3)


#### ■ 特徴・機能
- 専用線（高帯域、低遅延・少ジッタ、高セキュリティ）
- デリバリーパートナー（通信キャリア）の回線サービスを経由して接続する ※DXロケーション内での直接接続も可
- 帯域：50Mbps～100Gbps
- BGPを利用して接続
- BFDサポート
- CloudWatchによるモニタリング
- ポート使用量（VIF）＋ データ転送料（アウトバウント）


#### ■ 仕組み(コンポーネント)
- DirectConnectロケーション：AWSの物理ルータ（DXエンドポイント）が設置してある、データセンター事業者のデータセンター
- DirectConnect接続：物理的な接続
- DirectConnectエンドポイント：DirectConnectロケーション内に設置されたAWS側ネットワーク機器。利用者側の回線を終端する。
- 仮想インタフェース（VIF）：
- DirectConnectGateway（DXGW）：DirectConnectエンドポイントから複数のVPCに接続したい場合に利用する機能。DXGWを介して複数VPCと相互接続できる。
<br><br>

### DirectConnect接続：物理接続
物理的な専用線接続。回線の利用形態により２つの接続タイプ。
### Dedicated接続（占有接続）
物理的に一つの回線（ポート）を占有して使用する形態。利用者自身が回線を手配する。
- 利用者自身が物理接続を手配（コロケーション内の利用者側機器～DXエンドポイント間の回線敷設） 
- 帯域：1Gbps／10Gbps／100Gbpsから選択
- 複数のVIFを作成可能

### Hosted接続（ホスト型接続）
DirectConnectデリバリパートナー（通信キャリア）を通じてDXを利用する形態。
- パートナーが所有する物理接続を複数のユーザで共有（論理分割）する形態（利用者側で回線敷設は不要）
- Connectionを所有しているAWSアカウントから利用者側のAWSアカウントに対してVIFを提供（クロスアカウント利用）
- 利用帯域：50Mbps／100Mbps／200Mbps／300Mbps／400Mbps／500Mbps, 1Gbps／2Gbps／5Gbps／10Gbps
- ユーザ側で利用可能なVIF（HostedVIF）は1つ
- ユーザ側でVIFの削除、再作成が可能
<br><br>

## 仮想インタフェース（VIF）：BGP接続用の仮想インタフェース
Connection（接続）を通してBGPピアを確立するための論理インターフェイス。IPアドレスを持つVIFをルータ（ゲートウェイ）にアタッチして利用するイメージ。
- 作成したVIFをゲートウェイ（DXGW or VGW）にアタッチして利用する
- 利用目的に応じた３つのVIFタイプを選択（プライベート、パブリック、トラジット）
- VIF作成時に必要な情報（VIF共通）
  - VIFを作成する物理接続（DX接続） 
  - アタッチ先のゲートウェイ（※パブリックVFI除く）
  - AS番号（アタッチ先のゲートウェイで設定）
  - カスタマー側AS番号
  - VLAN ID
  - IPアドレス（利用者側、カスタマー側双方）
  - BGP MD5シークレット
- CloudWatchによるモニタリング（VIF送受信パケット量やパケットレート）

### プライベートVIF：VPC接続用VIF
- プライベートアドレスでVPCへ接続するためのVIF
- DXGWまたはVGWにアタッチ可能（DXGWへの接続推奨）
- プライベートVIFが受信可能なBGP経路数は最大100（それ以上はBGPコネクションがダウン）

### パブリックVIF：グローバルアドレスでAWSサービスにアクセスするためのVIF
- インターネット上で公開されているAWSサービス（S3等）を利用可能
- グローバルアドレスが必要

### トランジットVIF：TGW接続用VIF
- （DXGW経由で）TGWと接続するためのVIF（プライベートVIFはTGWと通信不可）
- 占有接続の場合は1つのみ、ホスト型接続の場合は1Gbps以上の場合のみ作成可
  - **パートナーによる共有型接続では利用できない**
<br><br>

## Direct Connect Gateway（DXGW）：VPCの集約接続
DXGWに複数のVPCをアタッチすることで、オンプレから単一のDX接続で複数のVPCに接続ができる。
- グローバルなフルマネージドリソース
- AS番号を持つ（DXGWのAS番号は重複OK）
- VIF-VGW間の転送機能
  - 許可されたプリフィックスをVIFを通じてカスタマー側に広報する ※VPC CIDRと同じか広いレンジのCIDRのみ許可
  - カスタマー側からの経路をVGWを通じてVPCに伝搬する
- 複数のVIFとVGWをアタッチ可能
  - 最大10VPCをアタッチ可（DXGWとVPCが同一アカウントである必要はない＝マルチアカウント対応可） 
- 折り返し通信不可（DXGWを経由したオンプレ間通信やVPC間通信）
- 利用料なし

## Transit Gateway（TGW）：VPCの集約HUB
DXGWに複数のVPCをアタッチすることで、オンプレから単一のDX接続で複数のVPCに接続ができる。
- グローバルなフルマネージドリソース
- AS番号を持つ（DXGWのAS番号は重複OK）
- VIF-VGW間の転送機能
  - 許可されたプリフィックスをVIFを通じてカスタマー側に広報する ※VPC CIDRと同じか広いレンジのCIDRのみ許可
  - カスタマー側からの経路をVGWを通じてVPCに伝搬する
- 複数のVIFとVGWをアタッチ可能
  - 最大10VPCをアタッチ可（DXGWとVPCが同一アカウントである必要はない＝マルチアカウント対応可） 
- 折り返し通信不可（DXGWを経由したオンプレ間通信やVPC間通信）
- 利用料なし



## 参考
[[AWS Black Belt Online Seminar] AWS Direct Connect 資料及び QA 公開](https://aws.amazon.com/jp/blogs/news/webinar-bb-awsdirectconnect-2021/)
  
