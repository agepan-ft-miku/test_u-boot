

# **Linux 6.12におけるstmmacドライバの解体新書：アーキテクチャの深層分析**

## **序論**

Synopsys社のDesignWare Ethernet MAC IPコアは、半導体業界においてユビキタスなソリューションとして確固たる地位を築いており、組込みシステムオンチップ（SoC）から高性能コンピューティングプラットフォームに至るまで、多岐にわたる製品に搭載されています1。この広範なIPファミリーに対応するLinuxの標準ドライバが

stmmacであり、その重要性は計り知れません4。本レポートの中心的な論点は、

stmmacドライバのアーキテクチャ的成功が、Synopsys IPポートフォリオ内に存在する膨大な多様性を管理しつつ、現代のLinuxカーネルネットワーキングパラダイムとシームレスに統合するために設計された、洗練された多層的な抽象化戦略に懸かっているという点です。

stmmacドライバは、単一のハードウェアを対象としたものではありません。STMicroelectronics、NXP、Intel、Rockchipなど、多数のベンダーがライセンス供与を受けた、世代も機能も異なる多様なIPコアをサポートしています7。このような状況では、各ベンダーが独自に改変したドライバのフォークをメンテナンスするという、組込み業界で頻繁に見られる非効率な開発モデルに陥りがちです。しかし、

stmmacは、これらの差異を単一のアップストリームコードベース内に集約することで、この問題を回避しています。この事実は、ドライバのライフサイクルの初期段階で、拡張性を意図したアーキテクチャ上の決定がなされたことを示唆しています。関数ポインタテーブルやプラットフォームデータ構造を駆使することで、断片化を防ぎ、コミュニティ主導でクリーンなソリューションを維持するという先見の明があったのです。この設計思想こそが、ドライバの長寿命と堅牢性の根幹をなしています。

本レポートでは、Linuxカーネルバージョン6.12.31時点のstmmacドライバの構造を分析し、その内部動作に関する決定的なガイドを提供します11。高レベルのアーキテクチャから始まり、ハードウェア抽象化のメカニズム、中心的なデータ構造、初期化シーケンス、データパスの処理、そして先進的な機能の実装に至るまで、ドライバのあらゆる側面を詳細に解き明かしていきます。

## **第1章 高レベルアーキテクチャと設計原則**

stmmacドライバの構造を俯瞰すると、関心の分離を基本原則とした、明確な階層構造が見えてきます。この設計により、バスインターフェース、コアロジック、ハードウェア固有の実装がそれぞれ独立して進化できる、高いモジュール性が確保されています。

### **1.1 階層化モデル**

ドライバは、主に3つの論理的な層に分割できます。

1. **バス抽象化層:** Linuxのデバイスモデルと直接インターフェースする最上位層です。この層は、主にプラットフォームデバイス（stmmac\_platform.c）とPCIデバイス（stmmac\_pci.c）といった、異なるバスタイプを処理するための個別のモジュールで構成されます。その主な責務は、デバイスツリーやPCIコンフィギュレーション空間からデバイスを発見し、メモリアドレス空間や割り込み番号といった基本的なリソースを取得することです12。  
2. **コアロジック層 (stmmac\_main.c):** ドライバの心臓部であり、バスに依存しない共通ロジックを実装しています。net\_device\_opsのコールバック関数群（パケット送受信、MTU変更など）、NAPI（New API）ポーリング機構、phylinkサブシステムとの連携、そして主要なプローブ/リムーブ処理（stmmac\_dvr\_probe）がこの層に含まれます。コアロジック層は、下位のハードウェア抽象化層を介してハードウェアを操作するオーケストレーターとして機能します。  
3. **ハードウェア抽象化層 (HAL):** ハードウェアレジスタを直接操作する最下層です。この層自体も、Synopsys IPの多様性に対応するためにさらに細分化されています。異なるバージョンのMAC（Media Access Control）、DMA（Direct Memory Access）、およびディスクリプタコントローラに対応する具体的な実装が含まれており、コアロジック層がハードウェアのバージョンの違いを意識することなく動作できるようにしています。

### **1.2 モジュール性とソースコード構成**

stmmacドライバのアーキテクチャは、そのソースコードの構成にも明確に反映されています。drivers/net/ethernet/stmicro/stmmac/ディレクトリ内のファイル群は、それぞれが特定の役割を担っており、前述の階層モデルを具現化しています4。

このファイル構造は、ドライバのアーキテクチャが時間とともに進化したことを物語っています。特に、多数のdwmac-\*.cファイル（例: dwmac-imx.c, dwmac-rockchip.c, dwmac-socfpga.c）の存在は、初期のモノリシックなプラットフォームドライバから、より粒度の細かいSoCファミリー固有のカスタマイズモデルへと移行したことを示しています7。コアとなる

stmmac\_platform.cが汎用的なフレームワークを提供する一方で、これらの「グルー」ファイルが、特定のチップにおけるクロック、リセット、PHY構成といった最終的かつ複雑な詳細を処理します。このアーキテクチャにより、SoCベンダーは、コアロジックの安定性を損なうことなく、新しいdwmac-\<soc\>.cファイルと関連するデバイスツリーバインディングを提供するだけで、新しいチップのサポートを追加できます。これは、ハードウェア統合における「ラストワンマイル問題」に対する、非常にスケーラブルで保守性の高い実践的な解決策です。

以下の表は、主要なソースファイルとその役割をまとめたものです。これは、ドライバのコードベースを理解し、修正やデバッグを行う際の指針となります。

**表1: stmmacの主要ソースファイルとその役割**

| ファイル名 | 主な役割 | 主要な関数/構造体 |
| :---- | :---- | :---- |
| stmmac\_main.c | コアロジック層。net\_deviceの操作、NAPI、データパス、phylink連携、主要な初期化/終了処理。 | stmmac\_dvr\_probe, stmmac\_xmit, stmmac\_poll, stmmac\_netdev\_ops |
| stmmac\_platform.c | バス抽象化層。プラットフォームバス（Device Treeベース）に接続されたデバイスの処理。 | stmmac\_pltfr\_probe, struct platform\_driver |
| stmmac\_pci.c | バス抽象化層。PCI/PCIeバスに接続されたデバイスの処理。 | stmmac\_pci\_probe, struct pci\_driver |
| stmmac\_ethtool.c | ethtoolコマンドのサポート。統計情報、レジスタダンプ、自己診断テストなどの実装。 | stmmac\_ethtool\_ops |
| dwmac\_lib.c | ハードウェア抽象化層。異なるMAC/DMAバージョンのための共通ライブラリ関数。 | dwmac\_probe\_features, dwmac\_dma\_axi |
| dwmac\<version\>\_core.c | ハードウェア抽象化層。特定のMACバージョン（例: dwmac4, dwmac5）のレジスタ操作。 | dwmac4\_setup, dwmac5\_setup |
| dwmac\<version\>\_dma.c | ハードウェア抽象化層。特定のDMAバージョンのレジスタ操作。 | dwmac\_dma\_ops |
| dwmac-\<soc\>.c | プラットフォーム固有のグルーコード。特定のSoCファミリー（例: i.MX, Rockchip）の初期化処理。 | imx\_dwmac\_probe, rk\_gmac\_probe |

## **第2章 ハードウェア抽象化層：多様性の克服**

stmmacが進化し続けるSynopsys IPコアファミリーをサポートできる能力の核心には、ハードウェアの多様性を吸収するための巧妙な抽象化メカニズムが存在します。その中心的な戦略は、C言語における古典的なオブジェクト指向設計パターンである、関数ポインタを含む構造体（オペレーション構造体）の活用です。

### **2.1 オペレーション構造体 (\*\_ops): 間接化による戦略**

ドライバは、ハードウェアの各機能ブロック（MAC、DMAなど）に対するインターフェースとして機能する、複数の「オペレーション構造体」を定義しています。コアロジック層は、ハードウェア固有の関数を直接呼び出すことは決してありません。常にこれらの構造体内の関数ポインタを介してハードウェアにアクセスします。この間接化により、コアロジックは、操作対象のハードウェアの具体的なバージョンや実装の詳細から完全に分離されます。

### **2.2 MACオペレーション (dwmac\_ops)**

MACコントローラ自体の操作を抽象化するのがdwmac\_ops構造体です。

* **機能:** このインターフェースは、MACコアの初期化、MACアドレスの設定、フィルタリング（マルチキャスト、ユニキャスト）の構成、フロー制御の管理、MAC固有の割り込み処理といった、MACに関連する全ての基本操作を網羅します。  
* **実装:** ドライバのプローブ処理中、ハードウェアのバージョンレジスタを読み取り、MACのバージョン（例: GMAC, DWMAC4, DWMAC5, XGMAC）を特定します1。そして、特定されたバージョンに対応する関数群（例:  
  dwmac4\_ops）でdwmac\_opsポインタを初期化します。これにより、以降のMAC操作は、正しいハードウェア実装に対して行われることが保証されます。priv-\>hw-\>mac-\>pcs\_ctrl\_aneのような関数呼び出しは、この抽象化メカニズムが実際に使用されていることを示しています16。

### **2.3 DMAオペレーション (dma\_ops)**

MACと同様に、DMAエンジンの操作もdma\_ops構造体によって抽象化されます。

* **機能:** DMAの初期化、送受信チャネルの開始/停止、割り込みのクリア、DMAバスモード（例: AXIバスパラメータ）の設定などが含まれます8。  
* **実装:** 異なるIPバージョンは、異なるDMA機能やレジスタレイアウトを持っています。これらの差異は、それぞれ異なるdma\_opsの実装によって吸収されます。例えば、priv-\>hw-\>dma-\>set\_tx\_ring\_lenのような呼び出しは、コアロジックがDMAエンジンの具体的な実装を知ることなく、送信リングの長さを設定できることを示しています16。

### **2.4 ディスクリプタとモードの抽象化**

この抽象化は、ハードウェアレジスタの操作だけでなく、ドライバが使用するデータ構造そのものにも及んでいます。

* **ディスクリプタ・オペレーション (stmmac\_desc\_ops):** ドライバのバッファとハードウェアを繋ぐDMAディスクリプタのフォーマットは、IPバージョンによって異なります。ドライバは「ノーマル」と「エンハンスド」の2種類のディスクリプタをサポートしています18。ディスクリプタへのオーナービットの設定、送信のためのディスクリプタ準備、受信後のステータス読み取りといった操作は、  
  stmmac\_desc\_ops構造体を介して抽象化されます。  
* **モード・オペレーション (stmmac\_mode\_ops):** ドライバは、ディスクリプタリストの管理方法として「リングモード」と「チェーンドモード」の両方をサポートしています8。この選択もまた、  
  stmmac\_mode\_opsを介して抽象化されており、コアロジックが基盤となるディスクリプタ管理スキームに依存しないように設計されています。

この多層的な抽象化（mac\_ops, dma\_ops, desc\_ops, mode\_ops）は、設定可能性の強力なマトリックスを生み出します。これにより、Synopsys社が新しいMACコアと古いDMAエンジンを組み合わせたIPや、既存のMACに新しいディスクリプタフォーマットを追加したIPをリリースした場合でも、大規模なドライバの書き換えを必要としません。ドライバは、初期化時に各サブブロックの機能を検出し（多くの場合、HW Capability Registerを介して21）、それぞれのコンポーネントに適切な

ops構造体を割り当てることで、デバイスの正しい「ペルソナ」を動的に組み立てることができます。これは、if (version \== X) {... } else if (version \== Y) {... }のようなモノリシックな構造よりもはるかに柔軟です。新しいハードウェアバリアントのサポートは、変更されたコンポーネントの新しいops実装を提供するだけで済み、ドライバの他の部分には影響を与えません。これこそが、stmmacドライバが広範なハードウェアをサポートできる重要な要因です。

以下の表は、MACとDMAの抽象化インターフェースを構成する主要な関数ポインタの例を示しています。

**表2: dwmac\_opsとdma\_opsの主要な関数ポインタ**

| 関数ポインタ名 | 抽象化層 | 目的の説明 |
| :---- | :---- | :---- |
| core\_init | MAC | MACコアの初期化、リセット、基本設定を行う。 |
| set\_mac\_addr | MAC | MACアドレスをハードウェアレジスタに設定する。 |
| set\_filter | MAC | マルチキャスト/ユニキャストアドレスフィルタリングを設定する。 |
| flow\_ctrl\_set | MAC | ハードウェアのフロー制御（ポーズフレーム）を設定する。 |
| pcs\_ctrl\_ane | MAC | PCS（Physical Coding Sublayer）のオートネゴシエーションを制御する。 |
| dma\_init | DMA | DMAエンジンの初期化、バスモード、ディスクリプタ設定を行う。 |
| start\_tx | DMA | 送信DMAチャネルを有効にする。 |
| stop\_tx | DMA | 送信DMAチャネルを無効にする。 |
| start\_rx | DMA | 受信DMAチャネルを有効にする。 |
| stop\_rx | DMA | 受信DMAチャネルを無効にする。 |
| dma\_interrupt | DMA | DMA割り込みステータスを読み取り、処理する。 |
| dump\_regs | MAC/DMA | デバッグ用にMACおよびDMAの全レジスタをダンプする。 |

## **第3章 コアデータ構造：ドライバの設計図**

stmmacドライバの動作を理解するためには、その状態、設定、および操作ポインタを保持する主要なC言語の構造体を把握することが不可欠です。これらのデータ構造は、ドライバの内部状態を表現する設計図そのものです。

### **3.1 stmmac\_priv構造体：中央ハブ**

stmmac\_privは、このドライバで最も重要なデータ構造です。ネットワークデバイスがインスタンス化されるたびに、この構造体のインスタンスが1つ割り当てられ、そのデバイスに関連するすべての状態情報を集約します。以下に、その主要なメンバを分類して解説します。

* **カーネルへのリンク:**  
  * struct net\_device \*dev: このプライベート構造体が属するnet\_deviceへのバックポインタ。  
  * struct device \*device: プラットフォームデバイスやPCIデバイスなど、基盤となるstruct deviceへのポインタ。  
* **ハードウェア抽象化ポインタ:**  
  * const struct mac\_device\_info \*hw: MACとDMAのオペレーション構造体（dwmac\_ops, dma\_opsなど）へのポインタを保持するコンテナへのポインタ。これがHALへのリンクとなります。  
* **PHY/リンク管理:**  
  * struct phylink \*phylink: 物理層（PHY）とのリンク状態を管理するための、現代的なphylinkサブシステムのインスタンスへのポインタ。  
  * struct phylink\_config phylink\_config: phylinkを設定するためのコンフィギュレーション構造体。  
* **コンフィギュレーション:**  
  * struct plat\_stmmacenet\_data \*plat: デバイスツリーやプラットフォームコードから渡される、プラットフォーム固有の設定データを保持します8。  
  * DMA設定、有効化された機能フラグ（例: tx\_coe, rx\_coe）、フロー制御設定（flow\_ctrl, pause22）などが含まれます。  
* **運用状態:**  
  * struct stmmac\_tx\_queue \*tx\_queues, struct stmmac\_rx\_queue \*rx\_queues: 送受信キューの配列へのポインタ。  
  * struct napi\_struct \*napi: 各受信キューに関連付けられたNAPIインスタンスへのポインタ。  
  * struct clk \*stmmac\_clk, struct reset\_control \*stmmac\_rst: クロックおよびリセットコントローラへのハンドル。  
  * 現在のリンク状態（speed, duplex）など。

stmmac\_priv構造体は、単なるデータコンテナではありません。それはドライバのステートマシンそのものを具現化したものです。そのフィールドは、プローブシーケンス中に段階的に設定され、リンク状態やキューの状態といったその値は、運用中に絶えず更新されます。この構造体のメモリダンプを異なる時点で分析することは、ドライバの健全性を診断するための強力なデバッグスナップショットを提供します。例えば、プローブの初期段階で割り当てられ23、デバイスツリーから

platメンバが設定され、ハードウェアバージョンが特定されるとhwポインタが設定され、PHYのセットアップ中にphylinkポインタが設定される、というように、構造体のライフサイクルはデバイスのライフサイクルと完全に一致します。どの関数がどのフィールドを設定するかを理解することは、ドライバの初期化と運用ロジックの明確なロードマップを提供します。

### **3.2 プラットフォームデータ (plat\_stmmacenet\_data): ボードとドライバの架け橋**

この構造体は、デバイスツリーやプラットフォームコードから解析された設定情報を、stmmac\_priv構造体に完全に統合される前の一時的な保持場所として機能します。PHYのアドレス、インターフェースモード（phy-mode）、クロック情報、リセットハンドル、そしてSoC固有のセットアップ関数（setup, init）へのポインタなどが含まれます8。これにより、ボード固有のハードウェア構成を、コアドライバのロジックから分離することができます。

### **3.3 キューとディスクリプタ管理構造体**

stmmac\_tx\_queueとstmmac\_rx\_queueは、マルチキュー環境における各キューの状態を管理するために不可欠です。これらの構造体は、ディスクリプタリングのベースアドレス、現在の処理位置（cur\_tx, dirty\_txなど）、関連付けられたsk\_buffの配列、そしてDMAマッピングされたアドレスの配列などを保持します。stmmac\_xmitやstmmac\_pollといったデータパスの中核関数は、これらの構造体を操作してディスクリプタを管理します24。

以下の表は、stmmac\_priv構造体の特に重要なメンバとその役割をまとめたものです。

**表3: stmmac\_priv構造体の重要なメンバ**

| メンバ名 | データ型 | 役割/説明 |
| :---- | :---- | :---- |
| dev | struct net\_device \* | このプライベートデータが属するネットワークデバイスへのポインタ。 |
| device | struct device \* | バス上のデバイス（例: platform\_device）へのポインタ。 |
| ioaddr | void \_\_iomem \* | メモリマップされたI/Oレジスタ空間のベースアドレス。 |
| hw | const struct mac\_device\_info \* | MAC/DMAのops構造体など、ハードウェア固有の情報を保持する構造体へのポインタ。 |
| plat | struct plat\_stmmacenet\_data \* | デバイスツリーから読み込まれたプラットフォーム固有の設定データ。 |
| phylink | struct phylink \* | phylinkサブシステムとの連携を管理するインスタンスへのポインタ。 |
| tx\_queues, rx\_queues | struct stmmac\_tx\_queue \*, struct stmmac\_rx\_queue \* | 送受信キューの配列へのポインタ。 |
| napi | struct napi\_struct \*\* | 各受信キューに対応するNAPIインスタンスの配列へのポインタ。 |
| synopsys\_id | u32 | ハードウェアから読み取ったSynopsys IPのバージョンID。 |
| dma\_cap | struct dma\_features | ハードウェアのDMA機能（チェックサム、PTPなど）を示すフラグ。 |
| speed, duplex | int | 現在のリンク速度とデュプレックスモード。 |

## **第4章 初期化の軌跡：プローブから稼働デバイスへ**

カーネルがstmmac互換デバイスを発見した際に何が起こるのか、その一連の処理を追跡します。この初期化シーケンスは、最も汎用的なレベルから始まり、各ステップでより具体的な情報を収集して、最終的に特定のハードウェアインスタンスに完全に適合したドライバの「ペルソナ」を構築する、段階的な特化のプロセスです。

### **4.1 プローブのエントリポイント**

初期化プロセスは、バス固有の層から始まります。SoCで一般的なプラットフォームデバイスの場合、エントリポイントはstmmac\_platform.c内のstmmac\_pltfr\_probe関数です12。この関数は、バス固有のセットアップ（例:

platform\_get\_resourceによるリソース取得）を行った後、コアとなるプローブ関数を呼び出すラッパーとして機能します。

### **4.2 デバイスツリーの解析**

次に、stmmac\_probe\_config\_dt関数が呼び出され、デバイスツリーからハードウェアの構成情報を読み取ります。これは、ドライバを特定のボード設計に適合させるための極めて重要なステップです12。この段階で、以下のプロパティが解析され、

plat\_stmmacenet\_data構造体に格納されます。

* reg: MMIOレジスタ空間のアドレスとサイズ。  
* interrupts: 割り込み番号。  
* clocks, resets: クロックおよびリセットコントローラへのハンドル。  
* phy-mode: PHYとの接続インターフェース（例: rgmii-id, sgmii）。  
* phy-handle: 接続されているPHYデバイスノードへの参照。  
* snps,\* プレフィックスを持つ多数のプロパティ: DMAのPBL（Programmable Burst Length）やAXIバスモードなど、Synopsys IP固有のチューニングパラメータ17。

### **4.3 コアドライバのプローブ (stmmac\_dvr\_probe)**

stmmac\_dvr\_probeは、バスに依存しない主要な初期化関数です23。ここからがドライバの本体と言えます。以下の一連の処理が実行されます。

1. alloc\_etherdevを呼び出し、net\_device構造体と、そのプライベート領域としてstmmac\_priv構造体を割り当てます。  
2. MMIOレジスタ空間をdevm\_ioremap\_resourceでマッピングします。  
3. Synopsys IPのバージョンレジスタを読み取り、ハードウェアを特定します。  
4. 特定されたバージョンに基づき、stmmac\_setup\_mac\_opsやstmmac\_setup\_dma\_opsといった関数を呼び出し、stmmac\_priv-\>hw内のmac\_opsやdma\_opsポインタに適切なHAL実装を設定します。  
5. 設定された\*\_ops関数ポインタを介して、MACコアとDMAエンジンを初期化します。  
6. MDIOバスを初期化し、接続されているPHYデバイスをスキャンします。  
7. phylinkインスタンスをセットアップし、登録します。  
8. 最後に、register\_netdevを呼び出し、net\_deviceをカーネルに登録します。これにより、ethXインターフェースが作成されます。

### **4.4 phylinkによるPHYとリンクの管理**

stmmacドライバの現代的な側面として、phylinkサブシステムへの統合が挙げられます。phylinkは、MACとPHY間のリンクを管理するための汎用的なステートマシンを提供し、ドライバのコードを大幅に簡素化します。stmmacは、古いphylibロジックからphylinkへ移行されています27。

* **phylink\_mac\_ops:** ドライバは、phylinkがMAC側を制御できるように、mac\_config、mac\_link\_up、mac\_link\_downといったコールバック関数群をphylink\_mac\_ops構造体に実装します28。  
  phylinkはPHY側の制御を自動的に行います。  
* **プロセス:** phylinkは、PHYとのオートネゴシエーション（または固定リンク設定）を行い、最適な共通構成を解決します。その後、mac\_configコールバックを呼び出し、stmmacのMACに対して、解決された速度、デュプレックス、インターフェースモードをプログラムします。これにより、複雑なリンク状態の管理ロジックをphylinkに委譲できます。

以下の表は、システムインテグレータがstmmacを使用するボードのデバイスツリーを記述またはデバッグする際に役立つ、主要なプロパティのガイドです。

**表4: stmmac設定に不可欠なデバイスツリープロパティ**

| プロパティ名 | 値の型 | 必須/任意 | 説明と目的 |
| :---- | :---- | :---- | :---- |
| compatible | string | 必須 | ドライバとデバイスを一致させるための文字列。例: "st,stm32mp1-dwmac", "snps,dwmac-4.20a" |
| reg | reg | 必須 | MACコントローラのMMIOレジスタ空間のアドレスとサイズ。 |
| interrupts | interrupts | 必須 | MACのメイン割り込みと、オプションでWake-on-LAN割り込み。 |
| clocks, clock-names | phandle, string | 任意 | MACの動作に必要なクロックへのハンドル。例: "stmmaceth", "pclk" |
| resets, reset-names | phandle, string | 任意 | MACのリセット信号へのハンドル。例: "stmmaceth" |
| phy-mode | string | 必須 | PHYとの物理インターフェースタイプ。例: "rgmii-id", "rmii", "sgmii" |
| phy-handle | phandle | 任意 | 外部PHYを使用する場合の、MDIOバス上のPHYノードへのハンドル。 |
| snps,pbl | u32 | 任意 | DMAのProgrammable Burst Length。パフォーマンスチューニングに使用。 |
| snps,axi-config | phandle | 任意 | AXIバスのパラメータを設定するためのサブノードへのハンドル。 |
| max-speed | u32 | 任意 | インターフェースの最大速度を制限する。例: |

## **第5章 データパス：パケットの旅**

この章では、ネットワークパケットがドライバを通過する際のフローを、送信と受信の両面から追跡します。stmmacのデータパス設計は、CPU効率を最優先事項としています。NAPIによる割り込みのバッチ処理、ゼロコピー受信、Scatter-Gather送信、チェックサム/セグメンテーションオフロードといった主要な決定はすべて、CPUあたりのパケット処理コストを最小限に抑えることを目的としています。これが、このIPとドライバの組み合わせがマルチギガビットのような高速ネットワークに適している理由です。

### **5.1 送信 (stmmac\_xmit)**

ndo\_start\_xmitコールバックとして実装されているstmmac\_xmit関数が、送信処理のエントリポイントです24。ネットワークスタックから

struct sk\_buffを受け取ると、以下のステップが実行されます。

1. **キューの選択:** マルチキューが有効な場合、パケットの分類情報（例: 優先度）に基づいて、適切な送信キュー（stmmac\_tx\_queue）を選択します。  
2. **ディスクリプタの確保:** 送信リング内に利用可能なDMAディスクリプタがあるか確認します。空きがない場合、netif\_tx\_stop\_queueを呼び出して、これ以上パケットが来ないようにネットワークスタックのキューを一時的に停止します。  
3. **DMAマッピング:** sk\_buffのデータ領域をDMA転送用にマッピングします。ドライバはScatter-Gather I/O (NETIF\_F\_SG)をサポートしているため、複数のフラグメントにまたがるsk\_buffも、データをコピーすることなく効率的に処理できます9。  
4. **ディスクリプタの設定:** 1つまたは複数のDMAディスクリプタに、マッピングされたDMAアドレス、データ長、および制御フラグ（チェックサムオフロード要求、タイムスタンプ要求、パケットの終端を示すフラグなど）を設定します。  
5. **所有権の移譲:** ディスクリプタの「オーナー」ビットを書き換えることで、そのディスクリプタの制御をCPUからDMAエンジンに移譲します。  
6. **DMAエンジンの起動:** 多くの場合、特定のレジスタに書き込みを行うことで、DMAエンジンに対して新しいディスクリプタが処理可能であることを通知します。

この一連の処理は、24のソースコードレベルでの分析で詳細に確認できます。

### **5.2 受信とNAPI (stmmac\_poll)**

パフォーマンスの鍵を握るのが、NAPIに基づいた受信パスです。これにより、高負荷時における割り込みストームを防ぎ、効率的なパケット処理を実現します9。

1. **トリガー:** DMAがパケットの受信を完了すると、割り込みが発生します。割り込みハンドラ（ISR）の役割は最小限に抑えられており、さらなる割り込みを無効にし、NAPIのpoll関数を\_\_napi\_scheduleでスケジュールするだけです。  
2. **pollループ:** カーネルは、割り込みコンテキスト外でstmmac\_poll関数を呼び出します。この関数には、一度に処理できる作業量の上限（バジェット）が与えられます。  
   * **送信完了処理:** poll関数はまず、送信キューをクリーンアップします。ハードウェアが送信を完了したディスクリプタを見つけ、そのバッファのDMAマッピングを解除し、対応するsk\_buffを解放します。もし送信キューが停止していた場合は、netif\_tx\_wake\_queueを呼び出して再開させます。  
   * **受信処理:** 次に、受信ディスクリプタリングを巡回します。CPUが所有権を持つ（つまり、受信が完了した）各ディスクリプタに対して、以下の処理を行います。  
     * DMAバッファのDMAマッピングを解除します。  
     * 受信したデータから新しいsk\_buffを構築します。この際、データをコピーしない「ゼロコピー」方式が採用されています18。  
     * napi\_gro\_receiveを介して、sk\_buffをネットワークスタックの上位層に渡します。  
     * 新しいバッファを割り当て、DMA用にマッピングし、そのディスクリプタに割り当て直してから、所有権をDMAエンジンに戻します。  
3. **ループの終了:** このループは、バジェットを使い切るか、処理すべき完了済みディスクリプタがなくなるまで続きます。バジェットを使い切ってもまだ処理すべきディスクリプタが残っている場合、poll関数は再スケジュールを要求します。すべての作業が完了した場合、napi\_completeを呼び出してNAPIサイクルを終了し、割り込みを再有効化します。

このNAPIベースのポーリングメカニズムの詳細は、24の分析で深く掘り下げられています。

## **第6章 先進的機能の実装**

stmmacドライバは、基本的なパケット転送機能にとどまらず、現代のネットワークアプリケーションで要求される高度な機能を多数実装しています。これらの機能は、stmmacが単なるNICドライバを超え、産業用制御ネットワークや車載イーサネット、仮想化ネットワーク機能といった、複雑で高性能かつ時間的制約の厳しいシステムを構築するための基盤コンポーネントであることを示しています。

### **6.1 ハードウェアタイムスタンプとPTP**

ドライバは、IEEE 1588 Precision Time Protocol (PTP)をサポートしており、MACを通過するパケットにハードウェアがタイムスタンプを付与する機能を利用します。

* **メカニズム:** この機能は、タイムスタンプを格納するための専用フィールドを持つ「エンハンスド」ディスクリプタフォーマットを使用することで実現されます18。  
  stmmac\_poll関数内で、ドライバはディスクリプタからこのハードウェアタイムスタンプを抽出し、カーネルのPTPサブシステムに渡します。これにより、マイクロ秒以下の精度でのクロック同期が可能になります。

### **6.2 ハードウェアオフロード**

CPUの負荷を軽減するため、ドライバは様々なオフロード機能に対応しています。

* **チェックサムオフロード (CSUM):** 送受信両方のTCP/UDP/IPパケットに対して、チェックサム計算をハードウェアにオフロードします18。  
* **セグメンテーションオフロード (TSO/GSO):** ドライバはソフトウェアベースのGeneric Segmentation Offload (GSO)をサポートしています。さらに、新しいIPバージョンでは、デバイスツリーのsnps,tsoプロパティを介して、ハードウェアによるTCP Segmentation Offload (TSO)を有効にすることも可能です9。

### **6.3 トラフィック制御 (TC) とサービス品質 (QoS)**

ドライバは、カーネルのTCサブシステムと統合されており、トラフィックシェーピングやスケジューリングをハードウェアにオフロードできます。

* **マルチキュー:** ハードウェアは複数の送受信キューをサポートしており、トラフィックの種類に応じてパケットを異なるキューに振り分けることが可能です8。  
* **スケジューラ:** ドライバは、MQPRIO (Multi-Queue Priority) スケジューラのオフロードに対応しています31。これにより、優先度に基づいた厳密なキューイングが可能になります。また、AVB (Audio Video Bridging) アプリケーション向けに、CBS (Credit-Based Shaper) のような他のスケジューラをサポートする機能もハードウェアには備わっています21。これらの機能により、ハードウェアレベルで帯域幅と遅延の保証が実現できます。

### **6.4 eXpress Data Path (XDP)**

最高のパケット処理性能を求めるユースケースのために、ドライバはXDPのフックを備えています。

* **機能:** XDPは、DMAによるパケット受信直後、sk\_buffが割り当てられるよりも前の段階で、BPF（Berkeley Packet Filter）プログラムを実行することを可能にします。これにより、DDoS攻撃の緩和、ロードバランシング、ラインレートでのフォワーディングといった処理を超低遅延で実現できます。  
* **実装:** ドライバは、AF\_XDPソケットを介したゼロコピー送受信をサポートしており、ユーザー空間アプリケーションが直接DMAバッファにアクセスすることを可能にします32。

これらの先進的な機能ポートフォリオは、市場の要求、特にインテリジェントで高機能なエッジネットワークハードウェアへの需要に直接応える形で投資・開発されたものです。その結果、Synopsys IPを搭載したSoCは、これらの技術的に要求の厳しい市場をターゲットにすることができ、stmmacドライバは単なる接続性の実現者ではなく、先進的な製品カテゴリの実現者としての役割を担っています。

## **結論**

Linuxカーネル6.12におけるstmmacドライバのアーキテクチャ分析を通じて、その成功が、高度にモジュール化され、抽象化された設計に根差していることが明らかになりました。この設計思想は、10年以上にわたってドライバが進化し続けることを可能にし、Synopsysからリリースされる新しいハードウェアバリアントや、Linuxカーネルに導入される新しいソフトウェアパラダイムを、驚くべき柔軟性をもって吸収してきました。

本レポートで詳述した3層構造（バス抽象化、コアロジック、ハードウェア抽象化）は、関心の分離を徹底し、各層が独立して開発・保守されることを可能にしています。特に、dwmac\_opsやdma\_opsといったオペレーション構造体を用いたハードウェア抽象化層は、膨大なIPの多様性を単一のコードベースで管理するための鍵となっています。このアーキテクチャは、初期化シーケンスにおける「段階的な特化」のプロセスにも表れており、デバイスツリー、ハードウェアレジスタ、PHYからの情報を組み合わせることで、各デバイスインスタンスに最適な構成を動的に構築します。

データパスは、NAPI、ゼロコピー、各種オフロード機能を駆使し、CPU効率を最大化するように最適化されています。さらに、PTP、TCオフロード、XDPといった先進的な機能の実装は、stmmacが単なるネットワークインターフェースドライバではなく、時間的制約の厳しいリアルタイムシステムや高性能ネットワーク仮想化環境を支える、重要な基盤技術であることを示しています。

結論として、stmmacドライバは、Linuxエコシステム内における複雑なハードウェアドライバ設計の模範的なモデルとして位置づけられます。広範なハードウェア互換性の必要性と、最先端のネットワーキング機能への要求との間で見事なバランスを取り、多種多様な組込みシステムおよび高性能システムにおいて、その継続的な重要性を確固たるものにしています。

#### **引用文献**

1. Synopsys Ethernet XGMAC IP, 9月 10, 2025にアクセス、 [https://www.synopsys.com/dw/ipdir.php?ds=dwc\_ether\_xgmac](https://www.synopsys.com/dw/ipdir.php?ds=dwc_ether_xgmac)  
2. Synopsys Ethernet GMAC IP, 9月 10, 2025にアクセス、 [https://www.synopsys.com/dw/ipdir.php?ds=dwc\_ether\_mac10\_100\_1000\_unive](https://www.synopsys.com/dw/ipdir.php?ds=dwc_ether_mac10_100_1000_unive)  
3. Synopsys Ethernet IP Solutions, 9月 10, 2025にアクセス、 [https://www.synopsys.com/designware-ip/interface-ip/ethernet.html](https://www.synopsys.com/designware-ip/interface-ip/ethernet.html)  
4. config\_stmmac\_eth \- kernelconfig.io, 9月 10, 2025にアクセス、 [https://www.kernelconfig.io/config\_stmmac\_eth](https://www.kernelconfig.io/config_stmmac_eth)  
5. Ethernet overview \- stm32mpu \- ST wiki, 9月 10, 2025にアクセス、 [https://wiki.st.com/stm32mpu/wiki/Ethernet\_overview](https://wiki.st.com/stm32mpu/wiki/Ethernet_overview)  
6. What is the linux stmmac driver for? \- STMicroelectronics Community, 9月 10, 2025にアクセス、 [https://community.st.com/t5/stm32-mpus-embedded-software-and/what-is-the-linux-stmmac-driver-for/td-p/105905](https://community.st.com/t5/stm32-mpus-embedded-software-and/what-is-the-linux-stmmac-driver-for/td-p/105905)  
7. linux/linux/drivers/net/ethernet/stmicro/stmmac/ Source Tree \- Code Browser, 9月 10, 2025にアクセス、 [https://codebrowser.dev/linux/linux/drivers/net/ethernet/stmicro/stmmac/](https://codebrowser.dev/linux/linux/drivers/net/ethernet/stmicro/stmmac/)  
8. Linux Driver for the Synopsys(R) Ethernet Controllers “stmmac”, 9月 10, 2025にアクセス、 [https://docs.kernel.org/5.10/networking/device\_drivers/ethernet/stmicro/stmmac.html](https://docs.kernel.org/5.10/networking/device_drivers/ethernet/stmicro/stmmac.html)  
9. stmmac.txt, 9月 10, 2025にアクセス、 [https://landley.net/kdocs/Documentation/networking/stmmac.txt](https://landley.net/kdocs/Documentation/networking/stmmac.txt)  
10. RN00161 Real-time Edge Software Release Notes \- NXP Semiconductors, 9月 10, 2025にアクセス、 [https://www.nxp.com/docs/en/release-note/RN00161.pdf](https://www.nxp.com/docs/en/release-note/RN00161.pdf)  
11. Linux 6.12.31 \- LWN.net, 9月 10, 2025にアクセス、 [https://lwn.net/Articles/1023079/](https://lwn.net/Articles/1023079/)  
12. stmmac \- STMicro ethernet driver note \- GitHub Pages, 9月 10, 2025にアクセス、 [https://cwshu.github.io/arm\_virt\_notes/notes/net\_driver/stmmac\_note.html](https://cwshu.github.io/arm_virt_notes/notes/net_driver/stmmac_note.html)  
13. genode-platforms-23-05.pdf, 9月 10, 2025にアクセス、 [https://genode.org/documentation/genode-platforms-23-05.pdf](https://genode.org/documentation/genode-platforms-23-05.pdf)  
14. Synopsys 200G/400G and 800G Ethernet MAC and PCS IP, 9月 10, 2025にアクセス、 [https://www.synopsys.com/dw/ipdir.php?ds=dwc\_ether\_200g\_400g\_800g\_mac\_pcs](https://www.synopsys.com/dw/ipdir.php?ds=dwc_ether_200g_400g_800g_mac_pcs)  
15. Synopsys Ethernet MAC IP, 9月 10, 2025にアクセス、 [https://www.synopsys.com/dw/ipdir.php?ds=dwc\_ether\_mac10\_100\_universal](https://www.synopsys.com/dw/ipdir.php?ds=dwc_ether_mac10_100_universal)  
16. stmmac ethernet in kernel 4.4: coalescing related pauses? \- Google Groups, 9月 10, 2025にアクセス、 [https://groups.google.com/g/linux.kernel/c/N54PQZgus80](https://groups.google.com/g/linux.kernel/c/N54PQZgus80)  
17. Documentation/devicetree/bindings/net/stmmac.txt \- platform/external/linux-kselftest \- Git at Google \- Android GoogleSource, 9月 10, 2025にアクセス、 [https://android.googlesource.com/platform/external/linux-kselftest/+/d97034ccdf0a13ad86f00945df245bbaf0780478/Documentation/devicetree/bindings/net/stmmac.txt](https://android.googlesource.com/platform/external/linux-kselftest/+/d97034ccdf0a13ad86f00945df245bbaf0780478/Documentation/devicetree/bindings/net/stmmac.txt)  
18. Linux Driver for the Synopsys(R) Ethernet Controllers “stmmac”, 9月 10, 2025にアクセス、 [https://www.kernel.org/doc/html/v5.7/networking/device\_drivers/stmicro/stmmac.html](https://www.kernel.org/doc/html/v5.7/networking/device_drivers/stmicro/stmmac.html)  
19. Linux Driver for the Synopsys(R) Ethernet Controllers “stmmac”, 9月 10, 2025にアクセス、 [https://docs.kernel.org/networking/device\_drivers/ethernet/stmicro/stmmac.html](https://docs.kernel.org/networking/device_drivers/ethernet/stmicro/stmmac.html)  
20. stmmac.txt, 9月 10, 2025にアクセス、 [https://www.kernel.org/doc/Documentation/networking/stmmac.txt](https://www.kernel.org/doc/Documentation/networking/stmmac.txt)  
21. Linux Driver for the Synopsys(R) Ethernet Controllers “stmmac”, 9月 10, 2025にアクセス、 [https://www.kernel.org/doc/html/v6.12/networking/device\_drivers/ethernet/stmicro/stmmac.html](https://www.kernel.org/doc/html/v6.12/networking/device_drivers/ethernet/stmicro/stmmac.html)  
22. \[PATCH net-next 1/3\] net: stmmac: clarify priv-\>pause and pause module parameter, 9月 10, 2025にアクセス、 [http://lists.infradead.org/pipermail/linux-amlogic/2025-February/023562.html](http://lists.infradead.org/pipermail/linux-amlogic/2025-February/023562.html)  
23. Linux Kernel: drivers/net/ethernet/stmicro/stmmac/stmmac\_main.c File Reference \- Huihoo, 9月 10, 2025にアクセス、 [https://docs.huihoo.com/doxygen/linux/kernel/3.7/stmmac\_\_main\_8c.html](https://docs.huihoo.com/doxygen/linux/kernel/3.7/stmmac__main_8c.html)  
24. stmmac\_main.c source code \[linux/drivers/net/ethernet/stmicro ..., 9月 10, 2025にアクセス、 [https://codebrowser.dev/linux/linux/drivers/net/ethernet/stmicro/stmmac/stmmac\_main.c.html](https://codebrowser.dev/linux/linux/drivers/net/ethernet/stmicro/stmmac/stmmac_main.c.html)  
25. Documentation/devicetree/bindings/net/stmmac.txt \- kernel/msm \- Git at Google \- Android GoogleSource, 9月 10, 2025にアクセス、 [https://android.googlesource.com/kernel/msm/+/android-7.1.0\_r0.2/Documentation/devicetree/bindings/net/stmmac.txt](https://android.googlesource.com/kernel/msm/+/android-7.1.0_r0.2/Documentation/devicetree/bindings/net/stmmac.txt)  
26. Ethernet | ConnectCore MP15 \- Support Resources \- Digi International, 9月 10, 2025にアクセス、 [https://docs.digi.com/resources/documentation/digidocs/embedded/dey/4.0/ccmp15/bsp-ethernet\_r\_stm.html](https://docs.digi.com/resources/documentation/digidocs/embedded/dey/4.0/ccmp15/bsp-ethernet_r_stm.html)  
27. net: stmmac: platform: Fix MDIO init for platforms without PHY \- Patchwork \- OzLabs, 9月 10, 2025にアクセス、 [https://patchwork.ozlabs.org/patch/1213121/](https://patchwork.ozlabs.org/patch/1213121/)  
28. Phylink and SFP: Going Beyond 1G Copper, 9月 10, 2025にアクセス、 [https://lpc.events/event/2/contributions/104/attachments/102/125/phylink-and-sfp-slides.pdf](https://lpc.events/event/2/contributions/104/attachments/102/125/phylink-and-sfp-slides.pdf)  
29. Documentation/networking/stmmac.txt \- kernel/common.git \- Android GoogleSource, 9月 10, 2025にアクセス、 [https://android.googlesource.com/kernel/common.git/+/ASB-2018-07-05\_4.9-o/Documentation/networking/stmmac.txt](https://android.googlesource.com/kernel/common.git/+/ASB-2018-07-05_4.9-o/Documentation/networking/stmmac.txt)  
30. Documentation/networking/stmmac.txt · v3.12.41 · Eclipse Projects / Oniro Core / linux · GitLab, 9月 10, 2025にアクセス、 [https://gitlab.eclipse.org/eclipse/oniro-core/linux/-/blob/v3.12.41/Documentation/networking/stmmac.txt?ref\_type=tags](https://gitlab.eclipse.org/eclipse/oniro-core/linux/-/blob/v3.12.41/Documentation/networking/stmmac.txt?ref_type=tags)  
31. Ethernet Driver Features in 6.12 \- donaldh.wtf, 9月 10, 2025にアクセス、 [https://donaldh.wtf/2024/11/ethernet-driver-features-in-6.12/](https://donaldh.wtf/2024/11/ethernet-driver-features-in-6.12/)  
32. \[EHL\] Integrated TSN controller (stmmac) driver support \- Launchpad Bugs, 9月 10, 2025にアクセス、 [https://bugs.launchpad.net/bugs/1928387](https://bugs.launchpad.net/bugs/1928387)