

# **Renesas mmngrbufとLinux Contiguous Memory Allocatorのアーキテクチャ分析：R-Car SoC環境における関係性の解明**

## **第1章 ヘテロジニアスコンピューティングSoCにおける連続メモリの課題**

### **1.1. ハードウェアアクセラレータの要件**

現代の組込みシステムオンチップ（SoC）、特にルネサス エレクトロニクス（以下、ルネサス）のR-Carシリーズのような先進的な車載用SoCは、単一のCPUコアだけでなく、多種多様な専用ハードウェアアクセラレータを統合したヘテロジニアス（異種混在）コンピューティングプラットフォームです 1。これらのSoCには、グラフィックス処理ユニット（GPU）、ビデオ・シグナル・プロセッサ（VSP）、ハードウェアビデオコーデック（エンコーダ/デコーダ）、および複数のダイレクトメモリアクセス（DMA）コントローラなどが搭載されています 3。

これらのハードウェアアクセラレータは、特定のタスクをCPUよりもはるかに高効率に実行するように設計されています。例えば、ビデオコーデックは動画の圧縮・伸張を、GPUは3Dグラフィックスのレンダリングを、VSPはカメラからの映像信号処理を専門に担当します。これらの処理は、ビデオフレームやグラフィックステクスチャといった、数メガバイトに及ぶ大規模なデータバッファを対象とします。性能を最大化するため、これらのアクセラレータはCPUの介在なしにメインメモリ（DDR RAM）との間で直接データを転送するDMAを利用します。

DMAが効率的に機能するための基本的な要件は、多くの場合、データバッファが物理メモリ上で連続した領域に配置されていることです 5。ハードウェアアクセラレータの中には、I/Oメモリ管理ユニット（IOMMU）を持たない、あるいは持っていても性能上の理由から利用されないものがあります。IOMMUがない場合、アクセラレータは物理アドレスを直接扱います。データが物理的に断片化（フラグメンテーション）していると、アクセラレータは単一の転送操作でバッファ全体を読み書きできません。一部のDMAコントローラは、散在するメモリ領域をリスト化して扱う「スキャッタ・ギャザー」機能をサポートしていますが、この機能には性能上のオーバーヘッドが伴い、全てのハードウェアでサポートされているわけではありません。したがって、特にリアルタイム性が要求されるマルチメディア処理においては、物理的に連続したメモリ領域を確保することが、システムの性能と安定性を保証する上で不可欠な要件となります。

### **1.2. フラグメンテーション問題**

Linuxカーネルの標準的なメモリ管理機構は、物理メモリを「ページ」という固定サイズの単位（ARM64アーキテクチャでは通常4 KB）で管理します。このページを効率的に割り当て・解放するために、「バディシステム（buddy system）」と呼ばれるアルゴリズムが用いられています。バディシステムは、様々なサイズのメモリ要求に柔軟に対応できますが、システムの稼働時間が長くなるにつれて、物理メモリの「フラグメンテーション（断片化）」を引き起こすという内在的な特性を持っています 5。

フラグメンテーションとは、システムの総空きメモリ量は十分にあるにもかかわらず、連続した大きな空き領域が存在しない状態を指します。これは、様々なサイズのメモリブロックの割り当てと解放が繰り返されることで、空きページが物理メモリ上に点在してしまうために発生します。

この状態は、前述のハードウェアアクセラレータにとって深刻な問題となります。例えば、4K解像度（3840x2160）の非圧縮ビデオフレーム（YUV422フォーマット）を格納するには、約16 MBのメモリが必要です。システム起動直後であれば、このような大きな連続領域を確保することは容易かもしれません。しかし、様々なアプリケーションやサービスが動作し、メモリの確保と解放を繰り返した後では、合計で100 MBの空きメモリがあったとしても、16 MBの連続したブロックを見つけることは極めて困難になります。

その結果、マルチメディア処理を開始しようとした際に連続メモリの割り当てに失敗し、機能が利用できなくなる、あるいは性能が著しく低下するといった、予測不能で不安定なシステム挙動の原因となります。kmalloc()のような標準的なカーネル内メモリアロケータは、この問題を解決するようには設計されていません。したがって、大規模な連続メモリを「確実」に確保するためには、バディシステムとは異なる特別な仕組みが必要となります。

### **1.3. ゼロコピーの必須要件**

組込みシステム、特にバッテリー駆動や厳しい熱設計が求められる車載システムにおいて、性能と電力効率は最重要課題です。マルチメディアパイプライン、例えば「カメラからの映像入力 → VSPによる画像補正 → GPUによるオーバーレイ合成 → H.264エンコーダによる圧縮 → ディスプレイコントローラによる表示」といった一連の処理を考えます。

このパイプラインの各ステージで、もしCPUが巨大なビデオバッファをメモリから読み出し、別のバッファに書き込むというコピー操作を行っていたらどうなるでしょうか。4Kビデオフレーム（約16 MB）を1回コピーするだけでも、CPUリソースを大量に消費し、メモリバス帯域を圧迫し、結果としてシステム全体の性能低下と消費電力の増大を招きます。パイプラインが複雑になれば、このコピー処理のコストは壊滅的になります。

この問題を解決するのが「ゼロコピー（zero-copy）」という設計思想です。ゼロコピーとは、パイプラインの異なるステージ間でデータをCPUによってコピーすることなく、単一のメモリバッファへの参照（ポインタやハンドル）を渡すことで処理を進める手法です。例えば、カメラドライバが確保したバッファにカメラハードウェアが直接映像を書き込み、そのバッファの参照がVSPドライバに渡され、VSPがその場でデータを処理し、次にGPUドライバに渡される、といった具合です。

このゼロコピーを実現するためには、単に連続メモリを確保できるだけでは不十分です。異なるデバイスドライバ間で、安全かつ効率的にバッファを共有するための標準化された仕組みが不可欠です。この仕組みは、あるドライバがバッファを書き込んでいる最中に、別のドライバがそれを読み出そうとすることを防ぐための同期機構も提供する必要があります。

このように、現代のヘテロジニアスSoCが直面するメモリ管理の課題は、単一の問題ではなく、以下の3つの密接に関連した要件から構成されています。

1. **連続性（Contiguity）:** DMAのために物理的に連続したメモリを確保すること。  
2. **信頼性（Reliability）:** メモリのフラグメンテーションに関わらず、要求されたサイズの連続メモリを確実に割り当てられること。  
3. **共有（Sharing）:** 複数のハードウェアアクセラレータ（およびそのドライバ）間で、ゼロコピーを実現するためにバッファを効率的かつ安全に共有すること。

これらの課題を解決するために、Linuxカーネルは階層的なソリューションを提供しており、その中核をなすのが次章で解説するContiguous Memory Allocator（CMA）です。

## **第2章 Linuxカーネルの基礎的解決策：Contiguous Memory Allocator (CMA)**

前章で述べた連続メモリ確保の信頼性という課題に対し、Linuxカーネルが提供する標準的な解決策がContiguous Memory Allocator（CMA）です 6。CMAは、特定のデバイスドライバが必要とする物理的に連続した大きなメモリブロックを、システムの断片化状態に影響されずに提供するために設計されたフレームワークです。

### **2.1. CMAの設計思想**

CMAの核心的な設計思想は、単純ながらも非常に効果的です。それは、「システム起動時の早い段階で、物理メモリの一部を連続メモリ割り当て専用の領域（CMA領域）として予約（reserve）しておく」というものです 5。このアプローチにより、システムが長時間稼働してメモリが断片化する前に、大きな連続領域が確保されます。

しかし、この予約されたメモリをデバイスが使用していない間、完全に遊ばせておくのはメモリリソースの無駄遣いです。CMAの優れた点は、この問題を解決する仕組みにあります。CMA領域は、デバイスドライバからの連続メモリ要求がない間、カーネルのバディシステムに「貸し出され」ます。ただし、どのようなメモリ要求にも応じるわけではありません。CMA領域から割り当てが許可されるのは、ページの内容を別の物理ページに移動（migrate）させることが可能な「移動可能（movable）」なページ要求のみです 6。移動可能なページの代表例としては、ユーザー空間プロセスの匿名メモリ（ヒープやスタック）や、ディスクキャッシュなどが挙げられます。

この設計により、CMAは2つの目的を両立させています。

1. **保証:** デバイスドライバに対して、必要な時にいつでも連続した物理メモリを提供できることを保証する。  
2. **効率:** デバイスがメモリを使用していない間は、その領域をシステム全体で有効活用し、メモリ使用率の低下を防ぐ。

このように、CMAはデバイスの要求とシステム全体のメモリ効率との間のトレードオフを巧みに解決する、バランスの取れたアプローチを採用しています。

### **2.2. 内部メカニズム**

CMAの動作を理解するには、Linuxのメモリ管理における「マイグレートタイプ（migrate type）」と「ページブロック（pageblock）」という概念を把握する必要があります。

* **マイグレートタイプ:** バディシステムは、管理するページをその移動可能性に応じていくつかのタイプに分類します。例えば、MIGRATE\_MOVABLE（移動可能）、MIGRATE\_UNMOVABLE（移動不可能）などです。これにより、同じ種類のページを物理的に近くに配置しようと試み、フラグメンテーションを抑制します 9。  
* **ページブロック:** ページブロックは、物理的に連続したページの集合体です（例：ARM64では通常4 MB）。各ページブロックには、そのブロックに含まれるページの主要なマイグレートタイプが割り当てられています 6。

CMAは、このマイグレートタイプの仕組みを拡張し、MIGRATE\_CMAという新しいタイプを導入します 8。システム起動時に予約されたCMA領域内のページブロックは、この

MIGRATE\_CMAタイプとして設定され、バディシステムに返却されます。バディシステムは、MIGRATE\_CMAタイプのページブロックからは、MIGRATE\_MOVABLEタイプのページ要求しか満たさない、という特別なルールに従います。これにより、CMA領域が移動不可能なページで汚染されるのを防ぎます。

デバイスドライバがCMA領域から連続メモリを要求した際の内部的な処理フローは以下のようになります。

1. **要求受付:** ドライバがdma\_alloc\_from\_contiguous()などのAPIを呼び出すと、CMAフレームワークが起動します。  
2. **領域の隔離:** CMAは、要求されたサイズのブロックを確保できるCMA領域内のページブロック群を特定し、それらのマイグレートタイプを一時的にMIGRATE\_ISOLATEに変更します。これにより、バディシステムがこれらのページブロックにアクセスしなくなります 8。  
3. **ページの移行:** 次に、隔離されたページブロック内に存在する「貸し出し中」の移動可能なページをすべて特定します。CMAはページ移行メカニズムを起動し、これらのページの内容をCMA領域外の別の空き物理ページにコピーし、ページテーブルなどの参照を新しい場所に更新します。このプロセスが完了すると、元の物理ページは空き状態になります 8。  
4. **割り当て:** ターゲット領域内のすべてのページが移行され、完全に空き状態になると、CMAはこの連続した物理ページ群を要求元のドライバに返します。  
5. **隔離の解除:** 最後に、ページブロックのマイグレートタイプをMIGRATE\_CMAに戻し、再びバディシステムの管理下に置きます。

この一連のプロセスにより、CMAは動的に使用されているメモリ領域からでも、オンデマンドで連続した物理メモリを「切り出して」提供することができるのです。

### **2.3. カーネルへの統合と設定**

デバイスドライバの開発者がCMAの内部関数を直接呼び出すことは通常ありません。CMAは、アーキテクチャ層のDMAマッピングAPIと緊密に統合されています 9。ドライバは、

dma\_alloc\_from\_contiguous()やdma\_alloc\_coherent()といった標準的なAPIを使用します。カーネルのコンフィギュレーションでCMAが有効になっている場合、これらのAPIのバックエンドとしてCMAが自動的に利用されます 8。

CMA領域のサイズや場所を設定する方法は、主に2つあります。これは、組込みシステムの開発者にとって非常に重要な知識です。

1. デバイツリー（Device Tree）:  
   組込みLinuxシステムでハードウェア構成を記述する標準的な方法であるデバイツリーを使用して、CMA領域を静的に定義できます。reserved-memoryノード内に、compatible \= "shared-dma-pool";というプロパティを持つサブノードを作成します 11。ルネサスのブートログでも、この方法でCMAメモリプールが作成されていることが確認できます 12。  
   **表1: RenesasプラットフォームにおけるCMA設定**

| 設定方法 | パラメータ/プロパティ | 説明 | 設定例 |
| :---- | :---- | :---- | :---- |
| デバイツリー | compatible | このノードがCMA領域であることを示す。 | compatible \= "shared-dma-pool"; |
| デバイツリー | reusable | デバイスが使用していない時に、カーネルがこの領域を移動可能なページのために再利用できることを示す。 | reusable; |
| デバイツリー | size | 予約するCMA領域のサイズをバイト単位で指定する。 | size \= \<0x0 0x10000000\>; (256 MB) |
| デバイツリー | alignment | 予約領域の物理アドレスのアライメントを指定する。 | alignment \= \<0x0 0x400000\>; (4 MB) |
| デバイツリー | alloc-ranges | 割り当て可能な物理アドレスの範囲を指定する。32ビットアドレス空間にしかアクセスできないデバイスのために使用されることがある。 | alloc-ranges \= \<0x0 0x0 0x0 0xffffffff\>; |
| カーネルコマンドライン | cma | 起動時にCMA領域のサイズを指定する。デバイツリーの設定を上書きできる。 | cma=256M |

2. カーネルコマンドライン:  
   ブートローダ（U-Bootなど）からカーネルに渡す起動パラメータで、CMA領域のサイズを動的に指定することも可能です。cma=256Mのように指定することで、カーネルを再コンパイルすることなく、システムのユースケースに応じてCMAのサイズを柔軟に変更できます 5。

これらの設定を通じて確保されたCMA領域の情報は、/proc/meminfoで確認できます。CmaTotalは予約された総量、CmaFreeはそのうち現在デバイスに割り当てられていない（貸し出し可能または未使用の）量を示します 5。

### **2.4. 運用上の考慮事項**

CMAは強力なフレームワークですが、万能ではありません。いくつかの運用上の注意点が存在します。

* **割り当てレイテンシ:** 2.2節で説明した通り、CMAからのメモリ割り当てはページの移行処理を伴う可能性があります。この移行処理には時間がかかる場合があり、dma\_alloc\_from\_contiguous()の実行時間は非決定的です。そのため、この関数を割り込みコンテキストのようなアトミックな（スリープが許されない）コンテキストから呼び出すことはできません 8。  
* **割り当ての失敗:** ページの移行は、そのページが「ピン留め（pinned）」されていない場合にのみ成功します。何らかの理由でカーネルが一時的にページを移動不可能としてマークしている場合（例えば、ユーザー空間メモリへの直接I/O処理中など）、移行は失敗します。CMA領域内にピン留めされた移動可能なページが存在すると、そのページを含む連続ブロックの割り当て要求は失敗する可能性があります 6。これはCMAの信頼性を損なう可能性のあるシナリオであり、システム設計において考慮が必要です。

これらの考察から導かれる結論は、CMAが連続メモリの「確保」という問題を解決するための、非常に効果的なカーネルの基礎コンポーネントであるということです。しかし、CMAはあくまで物理的なリソース（連続したページ群）を提供する「鈍器」のようなものであり、それらのページがどのように異なるデバイス間で「共有」され、「同期」されるかという、より高レベルな問題には関与しません。CMAはメモリという「モノ」を提供しますが、その使い方に関する「プロトコル」は提供しないのです。このギャップを埋めるために、次の章で説明するdmabufフレームワークが必要となります。

## **第3章 dmabufフレームワーク：デバイス間バッファ共有の標準規格**

前章では、CMAが物理的に連続したメモリを確保するための強力な基盤技術であることを明らかにしました。しかし、CMAが返すのはstruct page \*のようなカーネル内部のメモリオブジェクトであり、これを異なるデバイスドライバ間で安全かつ効率的に共有するための標準化されたプロトコルは提供しません。この「共有と同期」という課題を解決するために導入されたのが、dmabuf（DMA Buffer Sharing）フレームワークです 14。

### **3.1. 存在理由とアーキテクチャ**

dmabufは、Linuxカーネル内において、ハードウェアDMAアクセスを伴うバッファを、複数のデバイスドライバやサブシステム間でゼロコピー共有するための汎用的な仕組みを提供します 14。グラフィックス、ビデオ、カメラ、ディスプレイといったサブシステムが密に連携する現代のマルチメディアSoCにおいて、

dmabufは不可欠なコンポーネントとなっています。

そのアーキテクチャは、特定のドライバやサブシステムに依存しない、抽象化されたバッファオブジェクトに基づいています。これにより、例えばV4L2（Video for Linux 2）サブシステムのカメラドライバが作成したバッファを、DRM（Direct Rendering Manager）サブシステムのディスプレイドライバが直接読み込んで表示する、といった連携が可能になります。

### **3.2. コアコンセプト**

dmabufフレームワークは、いくつかの重要なコンセプトに基づいています。

* エクスポータとインポータ（Exporter/Importer）モデル:  
  バッファを最初に作成し、他のドライバに共有を提供する側を「エクスポータ」と呼びます。エクスポータは、自身が管理するネイティブなバッファ（例えば、CMAから確保したメモリ）を、struct dma\_bufという汎用的なオブジェクトでラップします。一方、共有されたバッファを利用する側を「インポータ」と呼びます。インポータは、struct dma\_bufに「アタッチ」することで、そのバッファへのアクセス権を取得します 14。  
* ファイルディスクリプタによる共有:  
  カーネル空間のstruct dma\_bufオブジェクトを、ユーザー空間のアプリケーションを介して異なるドライバに安全に渡すためのメカニズムとして、「ファイルディスクリプタ（file descriptor）」が使用されます。  
  1. エクスポータドライバは、作成したstruct dma\_bufに対応するファイルディスクリプタを生成し、ユーザー空間のアプリケーション（例：GStreamerやWaylandコンポジタ）に返します。  
  2. アプリケーションは、このファイルディスクリプタを、今度はインポータドライバ（例：ディスプレイドライバ）に対してioctlシステムコールなどを通じて渡します。  
  3. インポータドライバは、受け取ったファイルディスクリプタから、カーネル内の元のstruct dma\_bufオブジェクトへの参照を取得します。  
     このファイルディスクリプタを介したやり取りは、Linuxにおけるプロセス間通信（IPC）の標準的かつセキュアな手法であり、dmabufの堅牢性の基盤となっています 15。  
* 同期機構（dma-fence）:  
  dmabufは、非同期なハードウェア処理の完了を管理するためのdma-fenceフレームワークと密接に統合されています 14。例えば、GPUがバッファへのレンダリングを完了したことを示す  
  dma-fenceを生成し、dmabufに関連付けます。ディスプレイドライバは、そのバッファをスキャンアウト（表示）する前に、このフェンスがシグナル状態になる（つまり、GPUの処理が完了する）のを待つことができます。これにより、データの競合や表示の破損（ティアリングなど）を防ぎ、パイプライン全体の処理を正しく順序付けることができます。

これらのコンセプトを組み合わせることで、dmabufはCMAが提供できなかった、高レベルな共有と同期のプロトコルを実現します。

ここで、CMAとdmabufの関係性を整理すると、非常に明確な二段階の構造が見えてきます。CMAは、物理的なリソース、すなわち「連続したメモリ」を供給する低レベルのメカニズムです。一方、dmabufは、そのリソースを管理し、複数の利用主体間で共有するための高レベルな「論理的なプロトコル」を提供します。

言い換えれば、dmabufエクスポータは、バッファの物理的な記憶領域（backing store）をどこかから調達する必要がありますが、dmabufフレームワーク自体はその調達方法を規定しません 14。DMAアクセスを必要とするデバイスのための

dmabufエクスポータが、その記憶領域としてCMAを利用して確保した連続メモリを使用するのは、極めて自然で論理的な帰結です。

これにより、アプリケーションから物理メモリに至るまでの一貫したアーキテクチャが形成されます。  
アプリケーション → dmabufファイルディスクリプタ → インポータドライバ → エクスポータドライバ → DMA API → CMA → 物理メモリ  
このアーキテクチャパターンこそが、ルネサスのmmngrbufが適合する枠組みです。mmngrbufは、このパターンにおける「エクスポータドライバ」の役割を、R-Car SoCに特化した形で実装したものなのです。

## **第4章 ルネサスの実装：mmngrとmmngrbuf**

これまでの章で、連続メモリの確保（CMA）と共有（dmabuf）に関するLinuxカーネルの標準的なフレームワークを概観しました。本章では、これらの標準技術を基盤として、ルネサスがR-Car SoCプラットフォーム向けにどのように独自のメモリ管理ソリューションを構築しているかを、mmngrとmmngrbufという2つのコンポーネントに焦点を当てて詳述します。

### **4.1. ルネサスメモリマネージャ（mmngr）**

ルネサスのソフトウェアスタックには、mmngr\_drvというカーネルドライバと、それに対応するmmngr\_libというユーザー空間ライブラリが存在します 16。

mmngrは "Memory Manager" の略であり、その名の通り、R-Car SoC上のメモリ管理を担うコンポーネントです。

もし、単にCMAからメモリを確保するだけであれば、このようなベンダー固有の管理レイヤは不要なはずです。mmngrの存在は、R-Car SoCのハードウェアが、標準のCMAだけでは満たせない、より複雑で特殊なメモリ要件を持っていることを示唆しています。R-Car SoCは、VSP、IMP-X（画像認識プロセッサ）、GPUなど、多種多様なハードウェアIPを搭載しており、それぞれが最適な性能を発揮するためには、固有のアライメント要件、キャッシュコヒーレンシの管理ポリシー、あるいは特定の物理メモリバンクへの配置といった制約が存在する可能性があります。

mmngr\_drvは、これらのハードウェアIP固有の要件を熟知した、プラットフォーム専用の抽象化レイヤとして機能します。汎用的なCMAの上に、R-Carのアーキテクチャに最適化されたメモリ割り当てポリシーを実装し、SoC上のマルチメディア関連の特殊なメモリ要求を一元的に管理する役割を担っていると考えられます。ユーザー空間のアプリケーションは、mmngr\_libを介してこの高度なメモリ管理機能にアクセスします。

### **4.2. mmngrbuf: dmabufへの架け橋**

mmngrbufドライバの役割は、その名称とドキュメントから非常に明確です。これはmmngrの「dmabufサポート」を提供するドライバです 16。ルネサスのVerified Linux Package（VLP）のコンポーネントリストにも、

kernel-module-mmngrと並んでkernel-module-mmngrbufが含まれており、両者がセットで機能することが示されています 18。

具体的には、mmngrbufは第3章で説明したdmabufの**エクスポータ**として機能します。その責務は、mmngr\_drvが管理するプラットフォーム固有のメモリオブジェクトを受け取り、それをLinuxエコシステム全体で通用する標準的なstruct dma\_bufオブジェクトとして公開（エクスポート）することです。

このmmngrbufという「架け橋」の存在により、mmngrが管理するR-Carに最適化されたメモリバッファが、V4L2、DRM、GStreamerといった、標準的なdmabufインポータとして実装されたあらゆるソフトウェアコンポーネントからシームレスに利用可能になります。これにより、ルネサスはプラットフォーム固有の性能最適化と、広範なオープンソースソフトウェアエコシステムとの互換性という、2つの重要な目標を両立させています。

### **4.3. mmngrbufとCMAの関係性**

ここまでの分析を統合し、ユーザーの最初の問いである「mmngrbufとLinuxのCMAの関係」に明確な回答を与えます。

その関係性は、\*\*「mmngrbufは、dmabufサブシステムと標準DMA APIを介して、CMAフレームワークを利用するクライアントである」\*\*と要約できます。

両者は競合する技術ではなく、階層的な依存関係にあります。その依存関係の連鎖は以下の通りです。

1. mmngrbufは、mmngr\_drvによって管理されるメモリバッファを、dmabufとしてエクスポートします 16。  
2. mmngr\_drvは、それらのバッファに必要な物理的な記憶領域を確保するために、カーネルの標準DMA APIであるdma\_alloc\_from\_contiguous()を呼び出します 10。  
3. dma\_alloc\_from\_contiguous()への呼び出しは、最終的にカーネルのCMAフレームワークによって処理され、あらかじめ予約されたCMA領域から物理的に連続したページ群が割り当てられます 8。

したがって、mmngrbufが提供するdmabufの根源をたどると、そこにはCMAによって確保された物理メモリが存在します。mmngrbufはCMAを置き換えるものではなく、CMAを基盤として、その上にR-Carプラットフォーム固有の管理機能と、標準規格に準拠した共有インターフェースという付加価値を提供する、より高次のレイヤなのです。

### **4.4. 実用事例：R-Car上のGStreamerパイプライン**

この抽象的なアーキテクチャを具体的に理解するために、R-Carプラットフォームで実際に使用されるGStreamerのコマンドラインを見てみましょう。GStreamerは、マルチメディアのストリーミングパイプラインを構築するためのフレームワークであり、その要素（エレメント）の多くはdmabufをサポートしています。

以下は、R-Car H2やRZ/V2Lなどで見られる典型的なパイプラインの例です 20。

$ gst-launch-1.0 v4l2src device="/dev/video0" io-mode=dmabuf\! video/x-raw,...\! vspmfilter dmabuf-use=true\! video/x-raw,...\! omxh264enc use-dmabuf=true\!...  
このパイプラインの実行前には、多くの場合、modprobe \-a mmngr mmngrbuf vspm\_ifのようなコマンドで、必要なドライバがロードされます 22。これは、これらのドライバがパイプラインの動作に不可欠であることを直接的に示しています。

このパイプラインにおけるデータの流れと各コンポーネントの役割を段階的に分解してみましょう。

1. **v4l2src io-mode=dmabuf:**  
   * v4l2srcは、V4L2デバイス（この場合は/dev/video0、つまりカメラ）から映像をキャプチャするエレメントです。  
   * io-mode=dmabufオプションにより、このエレメントはdmabufの**エクスポータ**として動作します。  
   * 内部では、V4L2ドライバが（おそらくmmngr経由で）CMAを利用して、カメラハードウェアが映像を書き込むためのDMAバッファを確保します。  
   * そして、そのバッファをdmabufとしてラップし、ファイルディスクリプタをパイプラインの後段に渡します。  
2. **vspmfilter dmabuf-use=true:**  
   * vspmfilterは、ルネサスのVSP（Video Signal Processor）を制御するエレメントです。色空間変換やスケーリングなどの処理を行います。  
   * dmabuf-use=trueオプションにより、このエレメントはdmabufの**インポータ**として動作します。  
   * 前段のv4l2srcからdmabufのファイルディスクリプタを受け取り、それにアタッチします。  
   * これにより、VSPドライバはCPUによるメモリコピーを一切行うことなく、カメラが書き込んだバッファに直接アクセスし、その場で（in-place）映像処理を実行できます。これがゼロコピーです 23。  
3. **omxh264enc use-dmabuf=true:**  
   * omxh264encは、OpenMAX IL（OMX）インターフェースを介してハードウェアH.264エンコーダを制御するエレメントです。  
   * use-dmabuf=trueオプションにより、これもまたdmabufの**インポータ**として動作します。  
   * vspmfilterから渡された（あるいはvspmfilterが新たにエクスポートした）dmabufを受け取り、ハードウェアエンコーダがそのバッファの内容を直接読み込んでH.264ストリームに圧縮します。

この一連の流れにおいて、mmngrbufは、v4l2srcやvspmfilterのようなR-Car固有のハードウェアを操作するエレメントが、dmabufをエクスポートするためのバックエンドとして機能しています。mmngrbufがなければ、これらのデバイスドライバは標準的な方法でバッファを共有できず、このような効率的なゼロコピーパイプラインを構築することはできません。

この事例は、CMA、dmabuf、そしてルネサス独自のmmngr/mmngrbufが、どのように連携して一つの高性能なマルチメディアシステムを形成しているかを見事に示しています。

## **第5章 統合的考察：アーキテクチャの全体像と結論**

これまでの分析を通じて、Linuxカーネルの標準的なメモリ管理機能と、ルネサスR-Car SoCに特化したドライバ群が、それぞれ異なる階層で特定の課題を解決するために協調して動作する、洗練されたアーキテクチャが明らかになりました。本章では、このアーキテクチャの全体像を俯瞰し、このような階層化されたアプローチがもたらす価値について考察します。

### **5.1. 完全なメモリ管理スタック**

ルネサスR-Carプラットフォームにおける、マルチメディアアプリケーションから物理メモリに至るまでのメモリ管理スタックは、以下のような明確な階層構造で表現できます。

* **ユーザー空間 (User Space):**  
  * **アプリケーション:** GStreamer、Waylandコンポジタ、OpenCVなど、dmabufを利用する高レベルのアプリケーション。  
  * **ユーザー空間ライブラリ:** mmngr\_lib。アプリケーションがmmngrの高度なメモリ管理機能にアクセスするためのインターフェースを提供。  
* **カーネル境界 (Kernel Boundary):**  
  * **システムコール:** ioctlなど。ユーザー空間とカーネル空間の間でdmabufのファイルディスクリプタを受け渡すための標準的なインターフェース。  
* **カーネル空間：ベンダー固有ドライバ (Kernel Space: Vendor-Specific Drivers):**  
  * **mmngrbuf:** dmabufエクスポータ。mmngrが管理するメモリを標準dmabufオブジェクトに変換する「架け橋」。  
  * **mmngr\_drv:** R-Carプラットフォームのメモリマネージャ。ハードウェアIP固有の要件（アライメント、キャッシュ管理等）を考慮した最適なメモリ割り当てを実行。  
* **カーネル空間：標準フレームワーク (Kernel Space: Standard Frameworks):**  
  * **dmabufサブシステム:** デバイスドライバ間のバッファ共有と同期を管理する汎用フレームワーク。  
  * **DMAマッピングAPI:** dma\_alloc\_from\_contiguous()など。アーキテクチャ非依存のインターフェースを提供。  
* **カーネル空間：コアメモリ管理 (Kernel Space: Core Memory Management):**  
  * **Contiguous Memory Allocator (CMA):** 物理的に連続したメモリページの供給源。  
* **ハードウェア (Hardware):**  
  * **物理メモリ (DDR RAM):** 実際のデータが格納される物理デバイス。

この階層構造は、関心の分離（Separation of Concerns）という優れたソフトウェア設計原則を体現しています。各レイヤは明確に定義された責務を持ち、上下のレイヤとは標準化されたインターフェース（APIやシステムコール）を通じてのみ通信します。これにより、システム全体の複雑さが管理可能になり、各コンポーネントの独立した開発と保守が容易になります。

### **5.2. ベンダー固有抽象化の価値**

なぜ、標準のCMAとdmabufの上に、mmngr/mmngrbufというベンダー固有のレイヤを追加する必要があるのでしょうか。この一見複雑に見えるスタックは、実際には組込みシステム開発において多大な価値を提供します。

**表2: Linuxメモリ割り当てメカニズムの比較**

| 機能 | バディシステム | Contiguous Memory Allocator (CMA) | Renesas MMNGR/MMNGRBUF |
| :---- | :---- | :---- | :---- |
| **主要目的** | 汎用的なページ単位のメモリ割り当て | 断片化に強い、物理的に連続したメモリの**確保** | R-Carハードウェアに最適化されたメモリの**管理**と、標準規格での**共有** |
| **割り当て単位** | ページ (2n個) | 要求されたページ数 | ハードウェア要件に基づくブロック |
| **物理的連続性の保証** | 限定的（高次の割り当ては失敗しやすい） | 高い（予約領域内） | 高い（CMAをバックエンドとして利用） |
| **呼び出しコンテキスト** | スリープ可能/アトミック（GFPフラグによる） | スリープ可能のみ | ユーザー空間/スリープ可能 |
| **典型的なユースケース** | カーネル内のほぼ全ての動的メモリ確保 | DMAバッファ、ハードウェア用バッファ | ゼロコピーマルチメディアパイプライン |
| **主要な抽象化** | struct page | struct page | dmabuf ファイルディスクリプタ |

この表が示すように、各メカニズムは異なるレベルの抽象化と機能を提供します。mmngr/mmngrbufレイヤの付加価値は以下の点に集約されます。

* **抽象化:** アプリケーション開発者は、R-Car SoCのVSPやGPUが持つ複雑なメモリ要件を意識する必要がありません。彼らは標準的なdmabufハンドルを扱うだけでよく、ハードウェアの詳細はmmngrドライバが隠蔽します。  
* **性能:** mmngrは、R-Carのハードウェアアーキテクチャを深く理解しているため、キャッシュコヒーレンシの管理やアライメントの強制など、性能を最大化するための最適なメモリ割り当てを実行できます。  
* **安定性と保守性:** mmngr\_libとioctlインターフェースは、安定したAPIを提供します。将来、SoCの新しいリビジョンでハードウェアの仕様やカーネルの実装が変更されたとしても、このAPIを維持することで、アプリケーション側のコード修正を最小限に抑えることができます。  
* **エコシステムの実現:** 最も重要な点として、mmngrbufは、R-Carプラットフォームをdmabufを前提とする広大なLinuxソフトウェアエコシステム（GStreamer、Wayland、Androidなど）に接続するための鍵となります。これにより、顧客は既存のソフトウェア資産を容易に活用でき、開発期間を大幅に短縮できます。

結論として、ルネサスが採用しているmmngr/mmngrbufとCMAを組み合わせたアーキテクチャは、単一ベンダーの独自ソリューションではありません。これは、TI 13やQualcomm 11といった他の主要なSoCベンダーも採用している、組込みLinux業界における標準的な設計パターンです。すなわち、\*\*「CMAを物理メモリ確保の基盤とし、

dmabufをデバイス間共有の標準プロトコルとし、両者を結びつけるためにベンダー固有の最適化を施したドライバを『糊（glue）』として提供する」\*\*というアプローチです。

このアーキテクチャを理解することは、単にルネサスSoC上の特定のドライバの機能を学ぶだけでなく、現代の高性能組込みシステムにおけるメモリ管理の普遍的な課題とその解決策を把握することに繋がります。mmngrbufとCMAは、それぞれがこの階層的ソリューションの不可欠な一部を担う、協調的な関係にあるのです。

#### **引用文献**

1. R-Car-M2 \- Automotive System-on-Chip (SoC) for Mid-range Car Information Systems, 9月 11, 2025にアクセス、 [https://www.renesas.com/en/products/r-car-m2](https://www.renesas.com/en/products/r-car-m2)  
2. R-Car Automotive System-on-Chips (SoCs) \- Renesas, 9月 11, 2025にアクセス、 [https://www.renesas.com/en/products/automotive-products/automotive-system-chips-socs](https://www.renesas.com/en/products/automotive-products/automotive-system-chips-socs)  
3. R-Car-H3e \- R-Car H3/H3e/H3e-2G High-end Automotive System-on-Chip (SoC) for In-vehicle Infotainment and Integrated Cockpit | Renesas, 9月 11, 2025にアクセス、 [https://www.renesas.com/en/products/r-car-h3e](https://www.renesas.com/en/products/r-car-h3e)  
4. R-Car-M3e \- R-Car M3/M3e/M3e-2G Automotive System-on-Chip (SoC) Ideal for Medium-Class Automotive Computing Systems | Renesas, 9月 11, 2025にアクセス、 [https://www.renesas.com/en/products/r-car-m3e](https://www.renesas.com/en/products/r-car-m3e)  
5. Advanced Large Scale Computing at a Glance ... \- LINUX & HPC, 9月 11, 2025にアクセス、 [https://www.sachinpbuzz.com/2024/09/cma-contiguous-memory-allocator-linux.html](https://www.sachinpbuzz.com/2024/09/cma-contiguous-memory-allocator-linux.html)  
6. 内存之旅——如何提升CMA利用率？ \- OpenHarmony开发者- 博客园, 9月 11, 2025にアクセス、 [https://www.cnblogs.com/openharmony/p/16092537.html](https://www.cnblogs.com/openharmony/p/16092537.html)  
7. 【原创】（十六）Linux内存管理之CMA \- LoyenWang \- 博客园, 9月 11, 2025にアクセス、 [https://www.cnblogs.com/LoyenWang/p/12182594.html](https://www.cnblogs.com/LoyenWang/p/12182594.html)  
8. A deep dive into CMA \[LWN.net\], 9月 11, 2025にアクセス、 [https://lwn.net/Articles/486301/](https://lwn.net/Articles/486301/)  
9. Allocating big chunks of physically contiguous memory, 9月 11, 2025にアクセス、 [https://events.static.linuxfound.org/images/stories/pdf/lceu2012\_nazarwicz.pdf](https://events.static.linuxfound.org/images/stories/pdf/lceu2012_nazarwicz.pdf)  
10. Linux Kernel: drivers/base/dma-contiguous.c File Reference \- Huihoo, 9月 11, 2025にアクセス、 [https://docs.huihoo.com/doxygen/linux/kernel/3.7/dma-contiguous\_8c.html](https://docs.huihoo.com/doxygen/linux/kernel/3.7/dma-contiguous_8c.html)  
11. Configure and manage memory \- Qualcomm Linux Kernel Guide, 9月 11, 2025にアクセス、 [https://docs.qualcomm.com/bundle/publicresource/topics/80-70020-3/memory.html](https://docs.qualcomm.com/bundle/publicresource/topics/80-70020-3/memory.html)  
12. Driver support method for parallel input to CRU \- Forum \- Renesas Engineering Community, 9月 11, 2025にアクセス、 [https://community.renesas.com/rz/f/rz-forum/52562/driver-support-method-for-parallel-input-to-cru](https://community.renesas.com/rz/f/rz-forum/52562/driver-support-method-for-parallel-input-to-cru)  
13. 4.1.19. How to Configure the CMA Size — Processor SDK AM62Px Documentation, 9月 11, 2025にアクセス、 [https://software-dl.ti.com/processor-sdk-linux/esd/AM62PX/latest/exports/docs/linux/How\_to\_Guides/Target/How\_To\_Carve\_Out\_CMA.html](https://software-dl.ti.com/processor-sdk-linux/esd/AM62PX/latest/exports/docs/linux/How_to_Guides/Target/How_To_Carve_Out_CMA.html)  
14. Buffer Sharing and Synchronization (dma-buf) \- The Linux Kernel Archives, 9月 11, 2025にアクセス、 [https://www.kernel.org/doc/html/v6.10/driver-api/dma-buf.html](https://www.kernel.org/doc/html/v6.10/driver-api/dma-buf.html)  
15. Buffer Sharing and Synchronization (dma-buf) \- The Linux Kernel documentation, 9月 11, 2025にアクセス、 [https://docs.kernel.org/driver-api/dma-buf.html](https://docs.kernel.org/driver-api/dma-buf.html)  
16. renesas-rcar/mmngr\_drv: Memory Manager drv \- GitHub, 9月 11, 2025にアクセス、 [https://github.com/renesas-rcar/mmngr\_drv](https://github.com/renesas-rcar/mmngr_drv)  
17. renesas-rcar/mmngr\_lib: Memory Manager lib \- GitHub, 9月 11, 2025にアクセス、 [https://github.com/renesas-rcar/mmngr\_lib](https://github.com/renesas-rcar/mmngr_lib)  
18. RZ/V2L Linux Package Version 1.0.0 Component list \- Renesas Electronics Corporation, 9月 11, 2025にアクセス、 [https://www.renesas.com/en/document/apn/rzv2l-linux-package-version-100-component-list](https://www.renesas.com/en/document/apn/rzv2l-linux-package-version-100-component-list)  
19. RZ/G Verified Linux Package for 64bit kernel Version 1.0.21 Component list \- Renesas, 9月 11, 2025にアクセス、 [https://www.renesas.com/en/document/rln/rzg-verified-linux-package-64bit-kernel-version-1021-component-list](https://www.renesas.com/en/document/rln/rzg-verified-linux-package-64bit-kernel-version-1021-component-list)  
20. GStreamer build and customization \- Forum \- Renesas RZ MPU, 9月 11, 2025にアクセス、 [https://community.renesas.com/rz/f/rz-forum/52446/gstreamer-build-and-customization](https://community.renesas.com/rz/f/rz-forum/52446/gstreamer-build-and-customization)  
21. R car H2 with opencv \- Cockpit Support \- Renesas Engineering Community, 9月 11, 2025にアクセス、 [https://community.renesas.com/automotive-soc/r-car-h3-m3-cockpit/f/cockpit-f/9621/r-car-h2-with-opencv?ReplyFilter=Answers\&ReplySortBy=Answers\&ReplySortOrder=Descending)](https://community.renesas.com/automotive-soc/r-car-h3-m3-cockpit/f/cockpit-f/9621/r-car-h2-with-opencv?ReplyFilter=Answers&ReplySortBy=Answers&ReplySortOrder=Descending\))  
22. How to use H.264 encoding. | Renesas Customer Hub, 9月 11, 2025にアクセス、 [https://en-support.renesas.com/knowledgeBase/19905188](https://en-support.renesas.com/knowledgeBase/19905188)  
23. \[RZ/G1E\] Display image is wrong with VSPM filtering \- Renesas Engineering Community, 9月 11, 2025にアクセス、 [https://community.renesas.com/rz/f/rz-forum/9787/rz-g1e-display-image-is-wrong-with-vspm-filtering](https://community.renesas.com/rz/f/rz-forum/9787/rz-g1e-display-image-is-wrong-with-vspm-filtering)  
24. Video lag on RZG2L H.264 decoding \- Forum \- Renesas RZ MPU, 9月 11, 2025にアクセス、 [https://community.renesas.com/rz/f/rz-forum/18676/video-lag-on-rzg2l-h-264-decoding/62038](https://community.renesas.com/rz/f/rz-forum/18676/video-lag-on-rzg2l-h-264-decoding/62038)  
25. usb \- gadgets \- Forum \- Renesas RZ MPU, 9月 11, 2025にアクセス、 [https://community.renesas.com/rz/f/rz-forum/51180/usb---gadgets](https://community.renesas.com/rz/f/rz-forum/51180/usb---gadgets)