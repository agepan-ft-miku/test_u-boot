

# **ゼロコピービデオパイプラインの設計：LinuxにおけるGStreamerのDMA-BUFハンドリング詳説**

---

## **Part I: 高性能メディアパイプラインの基礎概念**

### **第1章: ゼロコピー処理の必要性**

現代のマルチメディアアプリケーション、特に組込みシステムや高性能サーバーにおいては、ビデオデータの効率的な処理が不可欠です。高解像度・高フレームレートのビデオストリームは膨大なデータ量を伴い、従来のCPU中心のデータ転送手法では深刻なパフォーマンスボトルネックを引き起こします。この課題を解決する鍵となるのが「ゼロコピー」という概念です。本章では、GStreamerにおけるゼロコピーの定義、その必要性、そしてそれを実現するための技術スタックの概要について解説します。

#### **1.1 GStreamerコンテキストにおける「ゼロコピー」の定義**

「ゼロコピー」という用語は一般的に広く使われますが、GStreamerの文脈ではより厳密な定義が存在します。それは、「あるエレメントによって生成されたデータが、別のエレメントによって、冪等変換（idempotent transform）を必要とせずに使用されること」です 1。

冪等変換とは、同じ操作を何度繰り返しても結果が変わらない変換を指します。この定義が意味するのは、単にCPUによるメモリコピー（memcpy）を避けるだけでなく、エレメント間でデータを受け渡す際に、内容を実質的に変更しないデータ変換処理（例えば、ハードウェアが直接扱えるメモリレイアウトを維持したままの処理）も不要であることを含意します。真のゼロコピーパイプラインでは、データはビデオキャプチャデバイスからハードウェアエンコーダ、そしてディスプレイコントローラへと、物理メモリ上の一貫した場所にとどまり、各ハードウェアユニットがそのメモリ領域に直接アクセスします。このアプローチにより、システム全体の効率が劇的に向上します。

#### **1.2 パフォーマンスボトルネック：CPUベースのメモリコピー**

ゼロコピーがなぜ重要なのかを理解するためには、従来のデータ処理におけるボトルネックを認識する必要があります。高精細ビデオストリーム、例えば1080p60や4K解像度の非圧縮ビデオは、1フレームあたり数メガバイトのデータを生成します。これをCPUのメインメモリにコピーし、さらにGPUやVPU（Video Processing Unit）などのハードウェアアクセラレータに転送するプロセスは、以下のような深刻な問題を引き起こします 2。

* **CPUサイクルの浪費:** memcpyのような操作はCPUリソースを大量に消費します。これにより、アプリケーションの他の重要なタスクに割り当てられるべきCPU時間が奪われます。  
* **メモリ帯域幅の圧迫:** CPUと各種ハードウェア間でのデータ転送は、システムのメモリバスに大きな負荷をかけます。これは、システム全体のパフォーマンス低下につながります。  
* **レイテンシの増大:** データのコピーには時間がかかります。リアルタイム性が要求されるビデオ会議やストリーミングアプリケーションにおいて、この遅延は致命的となる可能性があります。  
* **消費電力の増加:** CPUとメモリバスの活動が増加することは、特にバッテリー駆動の組込みデバイスにおいて、消費電力の増大に直結します 2。

開発者コミュニティでは、この問題は「ユーザ空間での信じられないほどの量のバイトプッシュ」と表現され、そのパフォーマンスへのペナルティは「甚大」であると認識されています 3。このボトルネックを解消し、ハードウェアの性能を最大限に引き出すことが、ゼロコピーアーキテクチャの主要な目的です。

#### **1.3 アーキテクチャ概要：ハードウェア、カーネル、GStreamerスタック**

Linux環境でゼロコピーを実現するためには、ハードウェア、カーネル、そしてユーザ空間フレームワークが協調して動作する必要があります。その全体像は以下のような階層構造で理解できます。

1. **ハードウェア層:** カメラセンサー、ビデオエンコーダ/デコーダ、GPU、ディスプレイコントローラなどの物理デバイスが存在します。これらのデバイスは、それぞれが直接アクセス可能な専用のメモリ領域を持つか、システムメモリ上の特定の領域にDMA（Direct Memory Access）でアクセスする能力を持っています。  
2. **カーネル層:** Linuxカーネルは、これらのハードウェアデバイスを抽象化し、制御するためのドライバとサブシステムを提供します。ゼロコピーにおいて特に重要なのは以下の3つです。  
   * **V4L2 (Video4Linux2):** カメラやハードウェアコーデックなどのビデオデバイスを制御するための標準APIです 7。  
   * **DRM (Direct Rendering Manager) / KMS (Kernel Mode Setting):** GPUやディスプレイコントローラを管理し、画面表示を制御するための現代的なフレームワークです 8。  
   * **DMA-BUF:** これらの異なるドライバ（V4L2, DRMなど）間で、物理メモリをコピーすることなくバッファを共有するための統一的なフレームワークです。これがゼロコピーの技術的な中核を担います。  
3. **ユーザ空間層:** GStreamerは、この層に位置するマルチメディアフレームワークです 2。GStreamerは、カーネルが提供するV4L2やDRMのAPIを利用してハードウェアを制御し、パイプラインを構築します。GStreamerの役割は、DMA-BUFを通じてカーネルレベルで共有されているバッファの「ハンドル」（ファイルディスクリプタ）をエレメントからエレメントへと受け渡し、データフロー全体をオーケストレーションすることです。データそのものはカーネル空間にとどまり、GStreamerは制御情報のみを扱います。

このレポートでは、これらの各層がどのように連携し、GStreamerがどのようにして効率的なゼロコピーパイプラインを構築するのかを詳細に解き明かしていきます。

### **第2章: GStreamerのメモリ管理アーキテクチャ**

GStreamerが多様なハードウェアと連携し、ゼロコピーのような高度な機能を実現できるのは、その柔軟で拡張性の高いメモリ管理アーキテクチャによるものです。このアーキテクチャは、GstBuffer、GstMemory、GstAllocatorという3つの主要なコンポーネントを中心に構築されています。本章では、これらのオブジェクトがどのように連携し、DMAバッファのような特殊なメモリを抽象化するのかを解説します。

#### **2.1 GstBuffer: データフローの原子単位**

GStreamerパイプラインを流れるデータの基本単位は GstBuffer です 9。これは単なるデータへのポインタではありません。

GstBuffer は、GStreamerの基本オブジェクトモデルである GstMiniObject を継承した構造体であり、メディアデータを処理するために必要な豊富な情報を含んでいます 9。

* **タイミング情報:** 各バッファは、PTS（Presentation Timestamp）とDTS（Decoding Timestamp）を持ちます。PTSはバッファの内容が表示されるべき時刻を、DTSはデコードされるべき時刻を示し、単位はナノ秒です 9。  
* **メタデータ:** GstBuffer には、GstMeta と呼ばれる任意のメタデータを付加できます。これにより、ビデオのクロッピング情報や色空間、あるいはDMAバッファ固有の情報など、標準フィールド以外のデータをエレメント間で伝達できます 9。  
* **メモリコンテナ:** GstBuffer は、実際のメディアデータを直接保持するわけではありません。代わりに、1つまたは複数の GstMemory オブジェクトのリストを内包します。この分離により、GStreamerはデータの物理的な格納場所から独立して、バッファの論理的なプロパティを管理できます 9。

#### **2.2 GstMemory: 多様なメモリ領域の抽象化**

GstMemory は、GStreamerのメモリ管理における中心的な抽象化レイヤーです 11。

GstMemory の役割は、特定のメモリ領域をカプセル化し、そのメモリがどこに（例えば、通常のシステムRAM、共有メモリ、ハードウェアデバイス上のメモリなど）、どのように確保されたかに関わらず、統一されたアクセス方法を提供することです。

この統一されたアクセスは、gst\_memory\_map() と gst\_memory\_unmap() という一対の関数によって実現されます 11。開発者は、

GstMemory オブジェクトに対して gst\_memory\_map() を呼び出すことで、そのメモリ領域へのポインタを取得し、データの読み書きを行うことができます。処理が完了したら gst\_memory\_unmap() を呼び出してアクセスを終了します。

この抽象化こそが、DMA-BUFの統合を可能にする鍵です。GstMemory は、DMA-BUFのファイルディスクリプタによって参照される物理メモリをラップできます。その場合、gst\_memory\_map() の内部では、CPUがそのメモリ領域にアクセスできるようにするための特別な処理（例えば、カーネルへのシステムコール）が実行されます。しかし、GStreamerプラグインの開発者から見れば、それは通常のシステムメモリをマップするのと何ら変わらないAPIとして見えます。

#### **2.3 GstAllocator: 特殊なメモリのためのファクトリ**

GstMemory オブジェクトは、GstAllocator によって生成されます 11。

GstAllocator は、特定の種類のメモリを割り当てる責務を持つ「ファクトリ」オブジェクトです。GStreamerは、標準的なシステムメモリ用のデフォルトアロケータを提供していますが、フレームワークの真の力はその拡張性にあります。

新しい種類のメモリ（例えば、特定のベンダーのGPUメモリや、DMA-BUFで共有されるメモリ）をサポートするためには、カスタムの GstAllocator を実装します 11。DMA-BUFの場合、

GstDmaBufAllocator という専用のアロケータが使用されます。このアロケータの alloc 関数は、実際には新しいメモリを確保しません。代わりに、既存のDMA-BUFファイルディスクリプタを受け取り、それをラップする GstMemory オブジェクトを作成します 13。この仕組みにより、カーネル空間で管理されているハードウェアバッファが、GStreamerのメモリ管理フレームワークにシームレスに統合されるのです。

#### **2.4 参照カウントとバッファプールの重要な役割**

GStreamerは、GstBuffer や GstMemory といった GstMiniObject のライフサイクルを管理するために、参照カウント方式を採用しています 9。オブジェクトが生成されると参照カウントは1になり、

gst\_buffer\_ref() が呼ばれるたびにインクリメントされ、gst\_buffer\_unref() が呼ばれるたびにデクリメントされます。参照カウントが0になると、そのオブジェクトに関連するリソース（メモリを含む）が解放されます。

この仕組みは、特にメモリリークの防止において極めて重要です。しかし、GStreamerにおけるメモリ問題は、単純な参照の解放忘れだけが原因ではありません。パイプラインの設計上の問題が、意図しないメモリ使用量の増大を引き起こすことがしばしばあります。

例えば、ある開発者が「メモリリーク」に直面し、プロセスのメモリ使用量がOOM（Out of Memory）キラーによって強制終了されるまで増大し続けるという問題がありました 16。調査の結果、これは伝統的な意味でのメモリリーク（解放されない

malloc）ではないことが判明しました。原因は、パイプラインの一部が後続の処理遅延により詰まってしまい、その上流にあるキュー（この場合は interpipesrc の内部キュー）が制限なくバッファを溜め込み続けたことでした 16。キューに保持されている各バッファは参照カウントが1以上であるため、決して解放されません。これはメモリを「リーク」しているのではなく、パイプラインのフロー制御の不備によってバッファを「溜め込んでいる」状態です。

この事例は、GStreamerのメモリ問題をデバッグする際のアプローチが、単に解放漏れを探すのではなく、パイプライン全体のデータフローとバックプレッシャーを分析することにあることを示唆しています。

この問題を回避し、パフォーマンスを最適化するために、GstBufferPool が使用されます 11。バッファプールは、同じサイズやプロパティを持つバッファを効率的に再利用するための仕組みです。特にハードウェアが割り当てるメモリは確保・解放のコストが高いため、ソースエレメントが一度確保したバッファをプールに返し、次のフレームで再利用することは、パフォーマンスを維持する上で不可欠です。

---

## **Part II: カーネルレベルのDMA-BUFフレームワーク**

GStreamerがユーザ空間でゼロコピーを実現できるのは、その土台となるLinuxカーネルの強力なバッファ共有メカニズムが存在するからです。その中核をなすのがDMA-BUFサブシステムです。本パートでは、DMA-BUFの基本原理と、マルチメディア処理に不可欠なV4L2およびDRM/KMSといった主要なカーネルサブシステムとの関係について掘り下げます。

### **第3章: Linux DMA-BUFサブシステムの紹介**

#### **3.1 コア原則：デバイス境界を越えたバッファ共有**

DMA-BUFは、Linuxカーネル内に存在する、ハードウェアDMAアクセス用のバッファを複数のデバイスドライバやサブシステム間で効率的に共有するための統一フレームワークです 17。例えば、V4L2カメラドライバがキャプチャしたビデオフレームを、GPUドライバやディスプレイコントローラドライバがメモリコピーなしで直接利用することを可能にします 13。

このフレームワークが登場する以前は、デバイス間でのバッファ共有はアドホックな独自実装に頼っており、非効率で断片化されていました。DMA-BUFは、この問題を解決するために、バッファを表現し共有するための標準化されたAPIとオブジェクトモデルを提供します。

#### **3.2 Exporter、Importer、およびユーザ空間ハンドル（ファイルディスクリプタ）モデル**

DMA-BUFのアーキテクチャは、3つの主要な役割に基づいています 18。

1. **Exporter（エクスポーター）:** バッファを生成し、他のデバイスが利用できるように提供する側のドライバです。例えば、ビデオキャプチャデバイスのV4L2ドライバがこれにあたります。エクスポーターは、自身が管理する物理メモリ領域に対して dma\_buf\_export() APIを呼び出し、カーネル内に struct dma\_buf という抽象オブジェクトを生成します 17。  
2. **ユーザ空間ハンドル（ファイルディスクリプタ）:** struct dma\_buf が生成されると、カーネルは dma\_buf\_fd() APIを通じて、そのバッファへのハンドルとしてファイルディスクリプタ（FD）をユーザ空間アプリケーション（GStreamerなど）に提供します 17。このFDは、通常のファイルと同様にプロセス間で受け渡しが可能であり、これにより異なるデバイスを制御するアプリケーションコンポーネント間でバッファを安全に共有できます。  
3. **Importer（インポーター）:** ユーザ空間からFDを受け取り、そのバッファを利用する側のドライバです。例えば、ディスプレイコントローラのDRMドライバがこれにあたります。インポーターは、まず dma\_buf\_get() APIでFDから struct dma\_buf への参照を取得します。次に、dma\_buf\_attach() を呼び出して、そのバッファを自身のデバイスコンテキストに関連付けます。これにより、インポーターはバッファの物理的な詳細（メモリレイアウトなど）をエクスポーターから取得し、自身のハードウェアがアクセスできるように準備します 17。

このモデルにより、バッファの物理的な所有権はエクスポーターが保持しつつ、複数のインポーターが安全かつ効率的にそのバッファを共有することが可能になります。

#### **3.3 同期プリミティブ：dma-fenceの役割**

ハードウェアデバイスによるメモリアクセスは本質的に非同期です。例えば、GPUがあるバッファにレンダリングを行っている最中に、ディスプレイコントローラがそのバッファを読み込んで表示しようとすると、表示が乱れる（ティアリングなど）可能性があります。

DMA-BUFフレームワークは、単にメモリを共有するだけでなく、こうした非同期アクセスを管理するための同期メカニズムも提供します。その中心となるのが dma-fence です 17。

dma-fence は、特定の非同期ハードウェア操作の完了を通知するための軽量な同期プリミティブです。

* バッファに書き込みを行うデバイス（例：GPU）は、書き込み操作を開始する際に dma-fence を生成し、バッファに関連付けます。  
* バッファから読み込みを行うデバイス（例：ディスプレイコントローラ）は、読み込みを開始する前に、関連付けられた dma-fence がシグナル状態になる（つまり、書き込み操作が完了する）のを待ちます。

これにより、データの整合性が保証され、複数のデバイスが協調して単一のバッファを正しく利用できるようになります。この同期機構はカーネルレベルで透過的に処理されるため、ユーザ空間アプリケーションは複雑な同期処理を意識する必要がありません。

### **第4章: マルチメディアのための主要なカーネルドライバとサブシステム**

DMA-BUFは汎用的なフレームワークですが、その真価は具体的なデバイスドライバサブシステムと統合されることで発揮されます。ビデオ処理パイプラインにおいては、V4L2とDRM/KMSが最も重要な役割を果たします。

#### **4.1 Video4Linux2 (V4L2): ビデオキャプチャとハードウェアコーデックのためのインターフェース**

Video4Linux2（V4L2）は、カメラ、ビデオキャプチャカード、ハードウェアベースのビデオエンコーダ/デコーダなど、ビデオストリームを扱うデバイスのための標準LinuxカーネルAPIです 22。V4L2は以下の機能を提供します。

* デバイスの列挙と能力の問い合わせ（サポートするフォーマット、解像度など）。  
* ビデオフォーマット、解像度、フレームレートなどのパラメータ設定。  
* バッファの管理とデータストリーミングの制御。

ゼロコピーの文脈において、V4L2ドライバはDMA-BUFの主要なエクスポーターまたはインポーターとなります。ドライバがDMA-BUFをサポートしているかどうかは、VIDIOC\_REQBUFS ioctlを V4L2\_MEMORY\_DMABUF メモリタイプで呼び出すことで確認できます 23。サポートされている場合、アプリケーションはV4L2ドライバに対してDMA-BUFのインポート（バッファをドライバに渡す）またはエクスポート（ドライバからバッファを受け取る）を要求できます。

#### **4.2 Direct Rendering Manager (DRM): 現代的なLinuxグラフィックススタック**

Direct Rendering Manager（DRM）は、GPUやディスプレイコントローラといった現代的なグラフィックスハードウェアを管理するためのカーネルサブシステムです 8。これは、古いフレームバッファデバイス（

/dev/fb0）を置き換えるもので、より高度で柔軟な機能を提供します。

##### **4.2.1 Kernel Mode Setting (KMS)**

KMSはDRMサブシステムの一部であり、ディスプレイのモード（解像度、リフレッシュレートなど）を設定し、画面表示パイプラインを制御するためのAPIです 8。KMSは以下の主要な抽象概念を扱います。

* **CRTC (Cathode Ray Tube Controller):** ディスプレイコントローラを表し、フレームバッファからピクセルデータを読み出し、タイミング信号を生成してコネクタに送ります。  
* **Plane (プレーン):** 画面上に表示される個々の画像レイヤーです。多くのハードウェアは、ベースとなるプライマリプレーンの上に、ビデオやUIを重ねて表示するためのオーバーレイプレーンを持っています。ゼロコピー表示では、ビデオフレームをこのオーバーレイプレーンに直接表示することで、合成処理のオーバーヘッドを削減します。  
* **Connector (コネクタ):** HDMI、DisplayPort、VGAなどの物理的な出力ポートを表します。

GStreamerの kmssink のようなエレメントは、このKMS APIを直接利用して、X11やWaylandといったディスプレイサーバーを介さずに、ビデオフレームを特定のプレーンに直接表示します。

##### **4.2.2 GEMとPRIME**

* **GEM (Graphics Execution Manager):** DRMにおけるメモリ管理フレームワークです。GPUが使用するバッファオブジェクト（GEMオブジェクト）の確保、解放、CPUとのマッピングなどを管理します 26。  
* **PRIME:** GEMとDMA-BUFを統合するためのフレームワークです 13。PRIMEは、GEMオブジェクトをDMA-BUFファイルディスクリプタとしてエクスポートしたり、逆にDMA-BUFファイルディスクリプタをGEMオブジェクトとしてインポートしたりする機能を提供します。ユーザ空間からは、  
  DRM\_IOCTL\_PRIME\_HANDLE\_TO\_FD や DRM\_IOCTL\_PRIME\_FD\_TO\_HANDLE といったioctlを通じて利用されます 21。

このPRIMEの仕組みにより、V4L2デバイスがエクスポートしたDMA-BUFを、DRMドライバがシームレスにインポートして画面に表示することが可能になります。

#### **表1: カーネルサブシステムの役割と責務**

これらのカーネル技術の関係性は複雑に見えることがありますが、それぞれの役割を明確に区別することが重要です。以下の表は、各サブシステムの責務をまとめたものです。

| サブシステム | 主な役割 | 主要な抽象化 | GStreamerとの連携 |
| :---- | :---- | :---- | :---- |
| **DMA-BUF** | 汎用的なバッファ共有と同期のフレームワーク | struct dma\_buf, dma\_fence, ファイルディスクリプタ (FD) | GstDmaBufAllocator がFDを GstMemory にラップする。 |
| **V4L2** | ビデオデバイス（キャプチャ、コーデック）の制御 | /dev/videoX, VIDIOC\_\* ioctl, V4L2\_MEMORY\_DMABUF | v4l2src, v4l2h264enc などがV4L2 ioctlを使いDMA-BUF FDをエクスポート/インポートする。 |
| **DRM/KMS** | ディスプレイとGPUの制御 | CRTC, Plane, Connector, Framebuffer (fb-id) | kmssink がKMS ioctlを使いDMA-BUF FDからフレームバッファを作成し、プレーンに表示する。 |

---

## **Part III: GStreamerとカーネルの橋渡し**

GStreamerがカーネルレベルのDMA-BUFフレームワークを活用するためには、両者の世界観を繋ぐための洗練されたメカニズムが必要です。Part IIIでは、GStreamerがどのようにしてカーネルから受け取ったDMA-BUFファイルディスクリプタを自身のメモリオブジェクトに変換し、パイプライン内で効率的にネゴシエーションとデータ転送を行うのか、その具体的な仕組みを解説します。

### **第5章: GStreamerとDMA-BUFの統合**

#### **5.1 GstDmaBufAllocator: ファイルディスクリプタをGstMemoryにラップする**

カーネルとGStreamerを繋ぐ最も直接的な接点が GstDmaBufAllocator です。前述の通り、v4l2src のようなソースエレメントがV4L2ドライバからDMA-BUFのファイルディスクリプタ（FD）を取得すると、そのFDをGStreamerパイプラインで扱えるように変換する必要があります。この変換プロセスは以下の手順で行われます 13。

1. gst\_dmabuf\_allocator\_new() を呼び出して、DMA-BUF専用のアロケータインスタンスを取得します。  
2. 次に、gst\_dmabuf\_allocator\_alloc\_with\_flags() を呼び出します。この関数にFDとバッファサイズを渡すと、アロケータは内部でFDを保持し、それを表現する GstMemory オブジェクトを生成して返します。このGstMemoryオブジェクトは、DMA-BUFによってバックアップされている物理メモリへの参照をカプセル化します。  
3. 最後に、この生成された GstMemory オブジェクトを gst\_buffer\_insert\_memory() を使って新しい GstBuffer に追加します。

こうして、カーネル空間に存在する物理バッファが、GStreamerの抽象化レイヤーを通じて、パイプライン内を流れる標準的な GstBuffer として扱えるようになります。パイプラインの下流にあるエレメント（例：kmssink）は、この GstBuffer から GstMemory を取り出し、その中から元のFDを抽出して、対応するカーネルサブシステム（例：DRM）に渡すことができます。

#### **5.2 ネゴシエーションの仕組み：memory:DMABuf Caps Feature**

GStreamerパイプラインでは、エレメント同士が接続される前に「Capsネゴシエーション」というプロセスが行われます。これは、エレメントのパッド（データの入出力点）間で、処理可能なデータフォーマット（解像度、フレームレート、ピクセルフォーマットなど）を合意する手続きです。

ゼロコピーパイプラインを構築するためには、単にピクセルフォーマットが一致するだけでは不十分です。バッファがDMA-BUFとして物理メモリを共有できる形式であることも合意する必要があります。このために、GStreamerは GstCapsFeatures という仕組みを提供しています。

エレメントのパッドがDMA-BUFを扱えることを示すために、そのパッドのCapsに memory:DMABuf というフィーチャーが追加されます 13。ネゴシエーションの際、GStreamerは両方のパッドがこのフィーチャーをサポートしていることを確認します。合意が成立すれば、エレメント間でDMA-BUFを直接受け渡すゼロコピーパスが確立されます。合意できなければ、GStreamerはデータをCPUメモリにコピーするなどのフォールバックパスを探します。

#### **5.3 データフロー：DMA-BUF GstBufferがパイプラインを横断する仕組み**

ゼロコピーパイプラインにおける GstBuffer のライフサイクルを追うことで、全体の流れをより深く理解できます。

1. **ネゴシエーション:** パイプラインが PAUSED 状態に遷移する際、v4l2src の src パッドと、それに接続された下流エレメント（例：v4l2h264enc）の sink パッドがCapsネゴシエーションを行います。両者が video/x-raw(memory:DMABuf) のようなCapsで合意します。  
2. **バッファ確保とラップ:** v4l2src はV4L2ドライバに対し、V4L2\_MEMORY\_DMABUF モードでバッファを要求し、エクスポートされたFDを取得します。  
3. **プッシュ:** PLAYING 状態になると、v4l2src はキャプチャしたフレームに対応するFDを GstDmaBufAllocator を使って GstMemory にラップし、GstBuffer に格納して下流にプッシュします。  
4. **パイプライン通過:** バッファはパイプラインを流れます。v4l2convert や nvvidconv のようなハードウェアアクセラレーションを利用するエレメントは、バッファからFDを抽出し、変換処理を行うハードウェアに直接そのFDを渡します。処理後の出力も同様に新しいDMA-BUFとして生成され、次のエレメントに渡されます。この間、ピクセルデータがCPUのメインメモリにコピーされることはありません。  
5. **レンダリング:** 最終的に kmssink がバッファを受け取ります。kmssink はバッファからFDを抽出し、DRM/KMSのioctlを呼び出してそのFDからDRMフレームバッファを作成し、ディスプレイコントローラに表示を指示します。  
6. **解放:** kmssink がバッファの表示を終えると、gst\_buffer\_unref() を呼び出します。バッファの参照カウントが0になると、GstMemory オブジェクトが解放されます。この時、GstDmaBufAllocator のデストラクタが働き、ラップしていたFDがクローズされます。これにより、カーネル内の struct dma\_buf の参照カウントも減少し、最終的にV4L2ドライバが物理メモリを再利用できるようになります。

### **第6章: 実践におけるDMA-BUF：ソース、シンク、アプリケーションエレメント**

理論的な仕組みを理解した上で、次にGStreamerの主要なエレメントでDMA-BUFを実際にどのように利用するのかを見ていきます。v4l2src、kmssink、そしてカスタムアプリケーションと連携するための appsrc/appsink が中心となります。

#### **6.1 ソース：v4l2srcとio-modeプロパティ**

v4l2src エレメントは、V4L2デバイスからのキャプチャを行う際の標準的なソースです。このエレメントでDMA-BUFを有効化するための最も重要なプロパティが io-mode です 28。このプロパティは、V4L2ドライバとの間でどのようにバッファを交換するかを決定します。

#### **表2: v4l2src のI/Oモードプロパティ**

io-mode プロパティはゼロコピーを実現する上で非常に重要ですが、各モードの違いが分かりにくいことがあります。以下の表は、主要なモードの機能とユースケースをまとめたものです。

| io-mode の値 | 説明 | バッファアロケータ | 主なユースケース |
| :---- | :---- | :---- | :---- |
| mmap (デフォルト) | V4L2ドライバがバッファを確保し、v4l2src がそれをユーザ空間にメモリマップする。他のハードウェアデバイスにデータを渡すには**CPUコピーが必要**。 | V4L2ドライバ | レガシーなパイプライン、またはCPUがピクセルデータにアクセスする必要がある場合。 |
| userptr | アプリケーションがメモリを確保し、そのポインタをドライバに渡す。非効率であり、DMA-BUFに取って代わられ、現在は非推奨。 | アプリケーション/GStreamer | レガシーシステム。一般的に推奨されない。 |
| dmabuf | V4L2ドライバがバッファを確保し、DMA-BUF FDとして**エクスポート**する。v4l2src はこのFDをラップする。ゼロコピーキャプチャの標準的なモード。 | V4L2ドライバ | カメラキャプチャからハードウェアエンコード/ディスプレイへ。v4l2src がプロデューサーとなる場合。 32 |
| dmabuf-import | v4l2src が、下流エレメントによって確保されたDMA-BUF FDを（ALLOCATION クエリ経由で）**インポート**する。これにより、下流エレメント（例：エンコーダ）がメモリ要件を決定できる。 | 下流のGStreamerエレメント | コンシューマが厳しいメモリ制約（特定のアライメントやタイリングなど）を持つ、密結合されたハードウェアパイプライン。 33 |

#### **6.2 シンク：kmssinkとダイレクトディスプレイレンダリング**

kmssink は、ゼロコピービデオシンクの代表例です 35。このエレメントは、X11やWaylandのようなコンポジタをバイパスし、カーネルのDRM/KMS APIを直接叩くことで、最小限のレイテンシでビデオフレームを画面に表示します 8。

kmssink の動作フローは以下の通りです。

1. memory:DMABuf フィーチャーを持つ GstBuffer を受け取ります。  
2. バッファ内の GstMemory オブジェクトからDMA-BUF FDを抽出します。  
3. DRMのioctl（DRM\_IOCTL\_MODE\_ADDFB2 など）を呼び出し、FDからDRMフレームバッファ（fb-id）を作成します。  
4. KMSのioctlを呼び出し、作成したフレームバッファを特定のDRMプレーンに「ページフリップ」することで表示を更新します。

マルチディスプレイ環境や特定のオーバーレイプレーンを使用する際には、connector-id、plane-id、bus-id といったプロパティを適切に設定する必要があります 35。

#### **6.3 アプリケーション：appsrcとappsinkでのDMA-BUF利用**

GStreamerパイプラインの外部で生成されたDMA-BUF（例えば、別のアプリケーションやCUDAカーネルで生成されたバッファ）をパイプラインに投入したり、パイプラインからDMA-BUFを取り出してカスタム処理を行ったりする場合には、appsrc と appsink を使用します。

* **appsrc での投入:** 外部で取得したDMA-BUF FDを、GstDmaBufAllocator を使って手動で GstBuffer にラップし、gst\_app\_src\_push\_buffer() を使ってパイプラインにプッシュします 14。この際、メモリのライフサイクル管理に注意が必要です。バッファがGStreamerに渡された後、元のアプリケーションがFDをクローズしてしまわないように、またGStreamerがバッファを使い終えたときに適切にリソースが解放されるように、カスタムの  
  GDestroyNotify 関数を提供することが一般的です 39。  
* **appsink での取得:** appsink から gst\_app\_sink\_pull\_sample() で GstSample を取得し、そこから GstBuffer と GstMemory を取り出します。GstMemory がDMA-BUFメモリであれば、そのFDを抽出して外部のAPI（例：CUDAやVA-API）に渡して処理することができます。

これらのエレメントを使うことで、GStreamerの強力なパイプライン機能を活用しつつ、外部のDMA-BUF対応ライブラリとの間でゼロコピー連携を実現できます。

### **第7章: 最新ハードウェアのための高度なネゴシエーション**

初期のDMA-BUF実装は、主にリニアな（線形的な）メモリレイアウトを前提としていました。しかし、現代のSoC（System on a Chip）は、メモリ帯域幅の効率を最大化するために、より複雑なメモリレイアウトを採用しています。この章では、これらの高度なレイアウトをGStreamerで扱うための「DRMフォーマットモディファイア」の概念と、そのネゴシエーションメカニズムについて解説します。

#### **7.1 モディファイアの必要性：リニアメモリレイアウトを超えて**

リニアレイアウトとは、画像のピクセルデータがメモリ上で単純に行ごとに連続して配置される形式です。これはCPUにとって最も扱いやすい形式ですが、ハードウェアにとっては必ずしも最適ではありません。

現代のGPUやビデオコーデックは、パフォーマンスを向上させるために、以下のような特殊なメモリレイアウトを使用します 13。

* **タイリング (Tiling):** ピクセルを2Dのタイル（ブロック）単位でメモリ上に配置します。これにより、2D空間でのメモリアクセスの局所性が高まり、キャッシュ効率が向上します。  
* **圧縮 (Compression):** フレームバッファの内容を可逆または非可逆で圧縮し、メモリ帯域幅の消費を削減します。

これらの特殊なレイアウトを持つバッファを、リニアレイアウトであると仮定して読み込もうとすると、ピクセルの順序がめちゃくちゃになり、画面に表示される映像は完全に乱れてしまいます 43。これは、GStreamerの初期のDMA-BUFサポートにおける大きな課題でした。

#### **7.2 DRMフォーマットモディファイアの役割**

この問題を解決するために、LinuxカーネルのDRMサブシステムは「フォーマットモディファイア」という概念を導入しました。DRMフォーマットモディファイアは、64ビットの整数値であり、特定のベンダーやハードウェアに固有のメモリレイアウトを一意に識別します 13。

バッファを完全に記述するためには、従来のピクセルフォーマット（FourCCで表現される、例：NV12）に加えて、このモディファイアが必要になります。例えば、「IntelのY-tiledレイアウトを持つNV12フォーマット」といった具体的な情報を、フォーマットとモディファイアのペアで表現できるようになります。

#### **7.3 GStreamer Caps内でのモディファイアのネゴシエーション**

GStreamer 1.24以降、このDRMフォーマットモディファイアを正式にサポートするための、より高度なCapsネゴシエーションメカニズムが導入されました 43。

この新しいメカニズムは、memory:DMABuf フィーチャーを持つCapsを拡張するものです。

* **新しいフォーマット DMA\_DRM:** Capsの format フィールドに、DMA\_DRM という新しい値が導入されました。  
* **新しいCapsフィールド drm-format:** format が DMA\_DRM の場合、drm-format という新しいフィールドが使用されます。このフィールドには、DRMフォーマットFourCC:DRMモディファイア という形式の文字列（またはそのリスト）が格納されます。例：drm-format=(string)NV12:0x0100000000000002 43。

この拡張により、GStreamerエレメントは、ピクセルフォーマットだけでなく、物理的なメモリレイアウトについても明示的にネゴシエーションを行うことができます。これにより、以前は「映像が乱れる」ためにDMA-BUFサポートを無効にせざるを得なかった多くのシナリオで、堅牢なゼロコピーが実現可能になりました 43。

この進化は、Linuxマルチメディアスタックの成熟を象徴しています。当初のゼロコピー実装は、フォーマットが同じであれば接続できるというヒューリスティック（経験則）に依存しており、脆弱でした。例えば、あるデバイスの「NV12」がリニアで、別のデバイスの「NV12」がタイル形式である可能性があり、この不一致が原因でデバッグの困難な問題を引き起こしていました。モディファイアの導入は、メモリレイアウトを暗黙の仮定から明示的なネゴシエーションの対象へと昇格させました。これにより、現代の多様なSoCの複雑な要件に対応できる、標準化された堅牢なシステムが構築され、高性能なビデオ製品の開発がより現実的なものとなりました。

---

## **Part IV: 実装、デバッグ、およびベストプラクティス**

これまでの章で解説した理論的背景を踏まえ、本パートではゼロコピーパイプラインの具体的な実装方法、発生しがちな問題のトラブルシューティング、そして効果的なデバッグ手法について、実践的な観点から詳述します。

### **第8章: ゼロコピーパイプラインの構築と分析**

#### **8.1 gst-launch-1.0による注釈付きサンプルパイプライン**

gst-launch-1.0 は、パイプラインのプロトタイピングとテストに非常に便利なコマンドラインツールです。以下に、典型的なゼロコピーパイプラインの例をいくつか示します。

* **カメラプレビュー（ゼロコピー表示）:**  
  Bash  
  gst-launch-1.0 v4l2src device=/dev/video0 io-mode=dmabuf\! 'video/x-raw(memory:DMABuf),width=1920,height=1080'\! kmssink

  このパイプラインは、最も基本的なゼロコピーの例です。V4L2キャプチャデバイス (/dev/video0) から io-mode=dmabuf を使ってDMA-BUFとしてフレームを取得し、それを一切のCPUコピーなしで kmssink を通じて直接ディスプレイに表示します 29。Capsフィルタで  
  memory:DMABuf を明示することにより、ゼロコピーパスが確実に選択されるようにしています。  
* **カメラからビデオエンコード（ゼロコピー処理）:**  
  Bash  
  gst-launch-1.0 v4l2src device=/dev/video0 io-mode=dmabuf\! 'video/x-raw,width=1280,height=720'\! v4l2h264enc output-io-mode=dmabuf\! filesink location=test.h264

  この例では、キャプチャしたビデオフレームをハードウェアH.264エンコーダ (v4l2h264enc) に渡します。v4l2src の io-mode と v4l2h264enc の output-io-mode の両方で dmabuf を指定することで、キャプチャからエンコードまでの一連の処理がゼロコピーで実行されます 32。エンコードされたデータは最終的にファイルに保存されます。  
* **フォーマット変換を含む完全なゼロコピー:**  
  Bash  
  gst-launch-1.0 v4l2src device=/dev/video0 io-mode=dmabuf\! v4l2convert capture-io-mode=dmabuf output-io-mode=dmabuf\! 'video/x-raw(memory:DMABuf),format=NV12'\! kmssink

  より複雑なケースとして、ハードウェアベースのフォーマット変換エレメント (v4l2convert) をパイプラインに挿入する例です。カメラが v4l2convert が直接扱えないフォーマット（例：YUYV）で出力し、後段の kmssink が NV12 を要求する場合などに使用します。v4l2convert の入出力両方でDMA-BUFモードを指定することで、変換処理自体はハードウェアで行われ、データは一貫してDMA-BUFとして扱われるため、パイプライン全体がゼロコピーを維持します 30。

#### **8.2 完全なゼロコピーパイプラインにおけるデータフローの分析**

最初のカメラプレビューの例を使って、DMA-BUFファイルディスクリプタ（FD）のライフサイクルを追ってみましょう。

1. **生成:** v4l2src は内部でV4L2カーネルドライバを呼び出し、ビデオフレームをキャプチャするためのDMA-BUFを確保させ、そのFDを取得します。  
2. **ラップ:** v4l2src は取得したFDを GstDmaBufAllocator を使って GstMemory オブジェクトにラップし、タイムスタンプなどの情報と共に GstBuffer に格納します。  
3. **転送:** この GstBuffer はパイプラインを流れて kmssink に到達します。この間、バッファの参照カウントはGStreamerによって管理されますが、物理データは移動しません。  
4. **アンラップと消費:** kmssink は受け取った GstBuffer から GstMemory を取り出し、その中からFDを抽出します。  
5. **表示:** kmssink は抽出したFDをDRM/KMSカーネルサブシステムに渡します。カーネルはFDをインポートし、対応する物理メモリをDRMフレームバッファとしてディスプレイコントローラに登録し、画面に表示させます。  
6. **解放:** 表示が終わると、kmssink はバッファへの参照を解放します（gst\_buffer\_unref）。参照カウントが0になると、FDがクローズされ、カーネルリソースが解放され、最終的にV4L2ドライバがそのメモリを再利用できるようになります。

### **第9章: 一般的なDMA-BUFパイプラインの問題のトラブルシューティング**

ゼロコピーパイプラインは高性能ですが、その設定は複雑であり、様々な問題が発生する可能性があります。ここでは、よくある問題とその診断・解決方法について解説します。

#### **9.1 ネゴシエーションの失敗と「映像の乱れ」の診断**

* **症状:** パイプラインのリンクに失敗する、またはリンクは成功するものの表示される映像がタイル状に乱れたり、色が不正になったりする。  
* **原因:** これは多くの場合、第7章で述べたDRMフォーマットモディファイアの不一致が原因です 43。ソースとシンクが同じピクセルフォーマット（例：NV12）をサポートしていても、物理的なメモリレイアウト（リニアかタイルかなど）が異なると、データが正しく解釈されません。  
* **解決策:**  
  1. GST\_DEBUG="GST\_CAPS:5" を設定してパイプラインを実行し、Capsネゴシエーションの詳細なログを確認します。  
  2. ログの中で、エレメント間が video/x-raw(memory:DMABuf), format=DMA\_DRM でネゴシエーションしようとしているか、そして drm-format フィールドが一致しているかを確認します。  
  3. 一致していない場合、どちらかのエレメントがモディファイアに対応していないか、あるいは互換性のないモディファイアを提案している可能性があります。videoconvert や imxvideoconvert\_g2d のような変換エレメントを間に挟むことで解決できる場合がありますが、これによりゼロコピーの利点が失われる可能性もあります。

#### **9.2 ストライドとアライメントの不一致の特定と解決**

* **症状:** パイプラインのリンクに失敗する、または映像の下部がずれたり、線が入ったりする。  
* **原因:** あるドライバ（例：エンコーダ）が、特定のストライド（画像の1行あたりのバイト数）やメモリアライメントを要求するのに対し、別のドライバ（例：カメラ）がそれに対応できない場合に発生します 34。ストライドは画像の幅とピクセルあたりのバイト数から計算されますが、ハードウェアの要件により、幅よりも大きい値（パディング）が要求されることがあります。  
* **解決策:**  
  1. この問題は、バッファのコンシューマ（消費者）がプロデューサー（生産者）に対してメモリ要件を伝えることで解決できます。  
  2. GStreamerでは、ALLOCATION クエリという仕組みを使います。コンシューマ（例：v4l2h264enc）は、自身が必要とするアライメント情報を持つバッファプールを提案します。  
  3. プロデューサー（例：v4l2src）は、io-mode=dmabuf-import に設定されている場合、この提案を受け入れ、コンシューマのプールからバッファを取得してデータを書き込みます 11。これにより、コンシューマの要件に完全に合致したバッファが使用され、問題が解決します。

#### **9.3 パイプラインのハングとストールのデバッグ**

* **症状:** パイプラインが再生を開始しない、または再生中に突然停止し、データフローが止まる。  
* **原因:** 下流のエレメントが何らかの理由でバッファを受け付けなくなり、上流のキューが満杯になることで発生します。原因は、ドライバのデッドロック、同期フェンスの喪失、あるいはGStreamerのバグなど多岐にわたります。  
* **解決策:**  
  1. パイプラインの要所に queue エレメントを配置し、silent=false プロパティを設定します。ハングが発生した際に、どのキューが満杯になっているか（current-level-buffers が増え続ける）を確認することで、ボトルネックの場所を特定できます 48。  
  2. アプリケーションコードにシグナルハンドラ（例：SIGUSR1）を実装し、その中で GST\_DEBUG\_BIN\_TO\_DOT\_FILE\_WITH\_TS() を呼び出すようにします。パイプラインがハングした際にシグナルを送ることで、その瞬間のパイプラインの状態をグラフとしてダンプでき、どのエレメントが詰まっているかを視覚的に確認できます 48。

#### **9.4 メモリリークの発見と修正**

* **症状:** パイプラインを長時間実行すると、システムのメモリ使用量が徐々に増加し、最終的にOOMキラーによってプロセスが終了させられる。  
* **原因:** 第2章で述べたように、これは伝統的なメモリリークではなく、バッファの「溜め込み」であることが多いです 16。パイプラインのどこかで処理が追いつかず、制限のないキューにバッファが溜まり続けることが原因です。  
* **解決策:**  
  1. valgrind などのツールは、GStreamerのカスタムアロケータや参照カウントモデルのため、直接的な原因特定には役立ちにくい場合があります。  
  2. まず、ハングのデバッグと同様に、キューのバッファレベルを監視してボトルネックを特定します。  
  3. パイプライン内のすべての queue エレメントに、max-size-buffers や max-size-bytes といった適切な上限値を設定します。  
  4. キューが満杯になった際の挙動を leaky プロパティで制御します（例：leaky=downstream で古いバッファを破棄する）。これにより、OOMキラーによる強制終了を防ぎ、アプリケーションの安定性を保つことができます 16。  
  5. appsrc/appsink を使用している場合は、gst\_buffer\_unref() の呼び出し忘れがないか、またカスタムの GDestroyNotify が正しく実装されているかを徹底的に確認します 39。

### **第10章: 不可欠なデバッグツールとテクニック**

効果的なトラブルシューティングには、適切なツールの使用が不可欠です。GStreamerは、パイプラインの内部状態を詳細に調査するための強力なデバッグ機能を提供しています。

#### **10.1 GST\_DEBUGを活用したDMA-BUFパイプラインのデバッグ**

GStreamerのデバッグ出力は、環境変数 GST\_DEBUG によって制御されます。この変数は、category:level のペアをカンマ区切りでリストしたものです 49。

category はGStreamerのサブシステムやエレメント名を、level は0（エラーのみ）から9（メモリダンプ）までのデバッグレベルを指定します。

#### **表3: ゼロコピーデバッグのための主要なGST\_DEBUGカテゴリ**

DMA-BUF関連の問題をデバッグする際には、以下のカテゴリを監視することが特に有効です。

| カテゴリ | サブシステム | 推奨レベル | 確認すべき内容 |
| :---- | :---- | :---- | :---- |
| v4l2 | Video4Linux2エレメント | 6 (LOG) | ドライバのプロービング、io-modeの選択、バッファのインポート/エクスポート操作。 |
| kms | KMS Sinkエレメント | 6 (LOG) | DRMデバイスのオープン、コネクタ/プレーンの選択、FDからのフレームバッファ作成。 |
| dmabuf | DMA-BUFアロケータ/メモリ | 7 (TRACE) | FDのラップ/アンラップ、メモリマッピング操作。 |
| GST\_CAPS | GStreamer Capabilities | 5 (DEBUG) | エレメント間のCapsネゴシエーションの詳細。memory:DMABufとdrm-formatの合意状況。 |
| GST\_BUFFER\_POOL | バッファプール操作 | 6 (LOG) | ALLOCATIONクエリの結果、プールの設定、バッファの確保/解放。 |

例えば、export GST\_DEBUG=v4l2:6,kms:6,GST\_CAPS:5 のように設定することで、V4L2とKMSの詳細な動作、およびそれらの間のCapsネゴシエーションに焦点を当てたログを取得できます。

#### **10.2 dmesgによるカーネルメッセージの解釈**

GStreamerエレメントはカーネルドライバのラッパーであるため、問題の根本原因がカーネル側にあることも少なくありません。パイプラインの実行中に問題が発生した場合は、dmesg コマンドを実行してカーネルログを確認することが重要です。

v4l2、drm、dma-buf といったキーワードでログを検索し、エラーメッセージや警告を探します 17。例えば、「

Failed to allocate DLIST entry」 52 や 「

GspRmAlloc failed」 53 のようなメッセージは、それぞれDRMドライバやNVIDIAドライバ内部のリソース確保失敗を示しており、GStreamerの設定ではなく、カーネルやドライバレベルの問題であることを示唆しています。

#### **10.3 グラフダンプによるパイプライン状態の可視化**

複雑な動的パイプラインでは、現在のエレメントの接続状態や各エレメントの状態を把握することが困難になる場合があります。GStreamerは、パイプラインの構造と状態をGraphvizの.dot形式でファイルに出力する機能を提供しています。

環境変数 GST\_DEBUG\_DUMP\_DOT\_DIR にディレクトリパスを設定すると、パイプラインの状態が変化するたび（例：NULLからPAUSED、PAUSEDからPLAYINGへ）に、その時点のパイプラインのグラフが自動的に指定されたディレクトリに保存されます 54。

これらの.dotファイルは dot コマンドで画像（PNGやSVGなど）に変換でき、パイプラインのトポロジー、各エレメントの状態、そしてネゴシエートされたCapsを視覚的に確認できます 55。これは、意図しないリンクが生成されている場合や、動的に追加・削除したパッドの状態を確認する際に非常に強力なツールとなります 48。

---

### **結論**

本レポートでは、GStreamerマルチメディアフレームワークにおけるDMA-BUFの取り扱いと、それを支えるLinuxカーネルの機能について、基礎概念から実践的な実装、デバッグ手法に至るまでを包括的に解説しました。

分析を通じて、現代の高性能マルチメディアパイプラインの構築が、単一の技術ではなく、ハードウェア、カーネルサブシステム（DMA-BUF, V4L2, DRM/KMS）、そしてユーザ空間フレームワーク（GStreamer）が緊密に連携するエコシステムによって成り立っていることが明らかになりました。

主要な結論として、以下の点が挙げられます。

1. **ゼロコピーは抽象化の賜物である:** GStreamerの GstMemory と GstAllocator によるメモリ抽象化は、カーネルレベルのDMA-BUFファイルディスクリプタを、フレームワーク内で統一的に扱えるようにするための鍵です。この抽象化により、プラグイン開発者は物理メモリの複雑な詳細を意識することなく、高性能なパイプラインを構築できます。  
2. **ネゴシエーションが堅牢性の核である:** 当初は脆弱であったDMA-BUFのサポートは、memory:DMABuf Caps Feature、そして特にDRMフォーマットモディファイアの導入によって大きく進化しました。これにより、複雑なメモリレイアウトを持つ最新のSoC上でも、信頼性の高いゼロコピーパイプラインの構築が可能となりました。これは、暗黙の仮定から明示的な合意形成へとパラダイムがシフトした結果です。  
3. **問題解決にはスタック全体の理解が不可欠である:** DMA-BUFパイプラインで発生する問題（ネゴシエーションの失敗、映像の乱れ、ハング、メモリ使用量の増大など）は、GStreamer層だけでなく、V4L2やDRMドライバ、さらにはカーネルのDMA-BUFサブシステム自体に起因することがあります。効果的なデバッグには、GST\_DEBUG によるGStreamer内部のログ分析と、dmesg によるカーネルメッセージの解釈を組み合わせ、問題の切り分けを行う能力が求められます。

GStreamerとLinuxカーネルの進化は続いており、将来的にはさらにシームレスなハードウェア連携と、より簡素化されたゼロコピーパイプラインの構築が可能になることが期待されます。本レポートが、高性能なマルチメディアアプリケーションを設計・実装する開発者にとって、その複雑なエコシステムを理解し、その能力を最大限に引き出すための一助となれば幸いです。

#### **引用文献**

1. Nicolas Dufresne \- Zero-Copy Pipelines in GStreamer, 9月 10, 2025にアクセス、 [https://gstreamer.freedesktop.org/data/events/gstreamer-conference/2017/Nicolas%20Dufresne%20-%20Zero-Copy%20Pipelines%20in%20GStreamer.pdf](https://gstreamer.freedesktop.org/data/events/gstreamer-conference/2017/Nicolas%20Dufresne%20-%20Zero-Copy%20Pipelines%20in%20GStreamer.pdf)  
2. GStreamer \- Wikipedia, 9月 10, 2025にアクセス、 [https://en.wikipedia.org/wiki/GStreamer](https://en.wikipedia.org/wiki/GStreamer)  
3. How to pass DMA buffers from a Driver thru GStreamer 'v4l2src' plugin to Freescale IPU/VPU plugins \- NXP Community, 9月 10, 2025にアクセス、 [https://community.nxp.com/t5/i-MX-Processors/How-to-pass-DMA-buffers-from-a-Driver-thru-GStreamer-v4l2src/m-p/255388](https://community.nxp.com/t5/i-MX-Processors/How-to-pass-DMA-buffers-from-a-Driver-thru-GStreamer-v4l2src/m-p/255388)  
4. How to pass DMA buffers from a Driver thru GStreamer 'v4l2src' plugin to Freescale IPU/VPU plugins \- NXP Community, 9月 10, 2025にアクセス、 [https://community.nxp.com/t5/i-MX-Processors/How-to-pass-DMA-buffers-from-a-Driver-thru-GStreamer-v4l2src/td-p/255388](https://community.nxp.com/t5/i-MX-Processors/How-to-pass-DMA-buffers-from-a-Driver-thru-GStreamer-v4l2src/td-p/255388)  
5. How to pass DMA buffers from a Driver thru GStreamer 'v4l2src' plugin to Freescale IPU/VPU plugins \- NXP Community, 9月 10, 2025にアクセス、 [https://community.nxp.com/t5/i-MX-Processors/How-to-pass-DMA-buffers-from-a-Driver-thru-GStreamer-v4l2src/m-p/255411/highlight/true](https://community.nxp.com/t5/i-MX-Processors/How-to-pass-DMA-buffers-from-a-Driver-thru-GStreamer-v4l2src/m-p/255411/highlight/true)  
6. How to pass DMA buffers from a Driver thru GStreamer 'v4l2src' plugin to Freescale IPU/VPU plugins \- NXP Community, 9月 10, 2025にアクセス、 [https://community.nxp.com/t5/i-MX-Processors/How-to-pass-DMA-buffers-from-a-Driver-thru-GStreamer-v4l2src/m-p/255418](https://community.nxp.com/t5/i-MX-Processors/How-to-pass-DMA-buffers-from-a-Driver-thru-GStreamer-v4l2src/m-p/255418)  
7. 5.1. Software Architecture of the Platform \- GitHub Pages, 9月 10, 2025にアクセス、 [https://xilinx.github.io/vck190-base-trd/2021.2/html/arch/arch-sw.html](https://xilinx.github.io/vck190-base-trd/2021.2/html/arch/arch-sw.html)  
8. A GStreamer Video Sink using KMS, 9月 10, 2025にアクセス、 [https://blogs.igalia.com/vjaquez/a-gstreamer-video-sink-using-kms/](https://blogs.igalia.com/vjaquez/a-gstreamer-video-sink-using-kms/)  
9. Gst.Buffer – gstreamer-1.0 \- Valadoc, 9月 10, 2025にアクセス、 [https://valadoc.org/gstreamer-1.0/Gst.Buffer.html](https://valadoc.org/gstreamer-1.0/Gst.Buffer.html)  
10. Gst.Buffer \- Structures \- Gst 1.0, 9月 10, 2025にアクセス、 [https://lazka.github.io/pgi-docs/Gst-1.0/classes/Buffer.html](https://lazka.github.io/pgi-docs/Gst-1.0/classes/Buffer.html)  
11. Memory allocation \- GStreamer, 9月 10, 2025にアクセス、 [https://gstreamer.freedesktop.org/documentation/plugin-development/advanced/allocation.html](https://gstreamer.freedesktop.org/documentation/plugin-development/advanced/allocation.html)  
12. GstMemory \- GStreamer, 9月 10, 2025にアクセス、 [https://gstreamer.freedesktop.org/documentation/additional/design/memory.html](https://gstreamer.freedesktop.org/documentation/additional/design/memory.html)  
13. DMA buffers \- GStreamer, 9月 10, 2025にアクセス、 [https://gstreamer.freedesktop.org/documentation/additional/design/dmabuf.html](https://gstreamer.freedesktop.org/documentation/additional/design/dmabuf.html)  
14. Dmabuf wrapped to GstBuffer for appsrc \- General Discussion \- GStreamer Discourse, 9月 10, 2025にアクセス、 [https://discourse.gstreamer.org/t/dmabuf-wrapped-to-gstbuffer-for-appsrc/1538](https://discourse.gstreamer.org/t/dmabuf-wrapped-to-gstbuffer-for-appsrc/1538)  
15. GstBuffer \- GStreamer, 9月 10, 2025にアクセス、 [https://gstreamer.freedesktop.org/documentation/gstreamer/gstbuffer.html](https://gstreamer.freedesktop.org/documentation/gstreamer/gstbuffer.html)  
16. Debugging a Memory Leak in a Large and Complex GStreamer Pipeline, 9月 10, 2025にアクセス、 [https://www.thegoodpenguin.co.uk/blog/finding-a-memory-leak-in-a-large-and-complex-gstreamer-pipeline/](https://www.thegoodpenguin.co.uk/blog/finding-a-memory-leak-in-a-large-and-complex-gstreamer-pipeline/)  
17. Buffer Sharing and Synchronization (dma-buf) \- The Linux Kernel documentation, 9月 10, 2025にアクセス、 [https://docs.kernel.org/driver-api/dma-buf.html](https://docs.kernel.org/driver-api/dma-buf.html)  
18. Buffer Sharing and Synchronization — The Linux Kernel documentation, 9月 10, 2025にアクセス、 [https://www.kernel.org/doc/html/v4.16/driver-api/dma-buf.html](https://www.kernel.org/doc/html/v4.16/driver-api/dma-buf.html)  
19. dma-buf-sharing.txt, 9月 10, 2025にアクセス、 [https://landley.net/kdocs/Documentation/dma-buf-sharing.txt](https://landley.net/kdocs/Documentation/dma-buf-sharing.txt)  
20. Buffer Sharing and Synchronization (dma-buf) — The Linux Kernel documentation \- DRI, 9月 10, 2025にアクセス、 [https://dri.freedesktop.org/docs/drm/driver-api/dma-buf.html](https://dri.freedesktop.org/docs/drm/driver-api/dma-buf.html)  
21. GStreamer and dmabuf, 9月 10, 2025にアクセス、 [https://gstreamer.freedesktop.org/data/events/gstreamer-conference/2012/omap-dmabuf-gstcon2012.pdf](https://gstreamer.freedesktop.org/data/events/gstreamer-conference/2012/omap-dmabuf-gstcon2012.pdf)  
22. How to pass DMA buffers from a Driver thru GStreamer 'v4l2src' plugin to Freescale IPU/VPU plugins \- Page 2 \- NXP Community, 9月 10, 2025にアクセス、 [https://community.nxp.com/t5/i-MX-Processors/How-to-pass-DMA-buffers-from-a-Driver-thru-GStreamer-v4l2src/m-p/255421](https://community.nxp.com/t5/i-MX-Processors/How-to-pass-DMA-buffers-from-a-Driver-thru-GStreamer-v4l2src/m-p/255421)  
23. 3.4. Streaming I/O (DMA buffer importing) \- The Linux Kernel Archives, 9月 10, 2025にアクセス、 [https://www.kernel.org/doc/html/v4.8/media/uapi/v4l/dmabuf.html](https://www.kernel.org/doc/html/v4.8/media/uapi/v4l/dmabuf.html)  
24. 3.4. Streaming I/O (DMA buffer importing) \- The Linux Kernel documentation, 9月 10, 2025にアクセス、 [https://docs.kernel.org/userspace-api/media/v4l/dmabuf.html](https://docs.kernel.org/userspace-api/media/v4l/dmabuf.html)  
25. DRM Internals \- The Linux Kernel documentation, 9月 10, 2025にアクセス、 [https://docs.kernel.org/gpu/drm-internals.html](https://docs.kernel.org/gpu/drm-internals.html)  
26. DRM Memory Management — The Linux Kernel documentation, 9月 10, 2025にアクセス、 [https://kernel.org/doc/html/v4.12/gpu/drm-mm.html](https://kernel.org/doc/html/v4.12/gpu/drm-mm.html)  
27. DRM Memory Management — The Linux Kernel 5.2.0-rc6-next-20190626+ documentation, 9月 10, 2025にアクセス、 [https://www.infradead.org/\~mchehab/rst\_conversion/gpu/drm-mm.html](https://www.infradead.org/~mchehab/rst_conversion/gpu/drm-mm.html)  
28. 4x 1080p60 V4L2src Capture+Encode gStreamer Performance Issues \- Jetson TX2, 9月 10, 2025にアクセス、 [https://forums.developer.nvidia.com/t/4x-1080p60-v4l2src-capture-encode-gstreamer-performance-issues/67399](https://forums.developer.nvidia.com/t/4x-1080p60-v4l2src-capture-encode-gstreamer-performance-issues/67399)  
29. v4l2src \- GStreamer, 9月 10, 2025にアクセス、 [https://gstreamer.freedesktop.org/documentation/video4linux2/v4l2src.html](https://gstreamer.freedesktop.org/documentation/video4linux2/v4l2src.html)  
30. gstreamer v4l2convert and v4l2h264enc performance \- Raspberry ..., 9月 10, 2025にアクセス、 [https://forums.raspberrypi.com/viewtopic.php?t=388862](https://forums.raspberrypi.com/viewtopic.php?t=388862)  
31. AM5728: DMA memory allocation fails with dmabuf \- Processors forum \- TI E2E, 9月 10, 2025にアクセス、 [https://e2e.ti.com/support/processors-group/processors/f/processors-forum/628671/am5728-dma-memory-allocation-fails-with-dmabuf](https://e2e.ti.com/support/processors-group/processors/f/processors-forum/628671/am5728-dma-memory-allocation-fails-with-dmabuf)  
32. zero copy Gstreamer pipline CPU usage differences in 4.14 and 5.10.35 BSPs, 9月 10, 2025にアクセス、 [https://community.nxp.com/t5/i-MX-Processors/zero-copy-Gstreamer-pipline-CPU-usage-differences-in-4-14-and-5/m-p/1349875](https://community.nxp.com/t5/i-MX-Processors/zero-copy-Gstreamer-pipline-CPU-usage-differences-in-4-14-and-5/m-p/1349875)  
33. Bug 787593 – kmssink: Export DMABuf \- GNOME Bugzilla, 9月 10, 2025にアクセス、 [https://bugzilla.gnome.org/show\_bug.cgi?id=787593](https://bugzilla.gnome.org/show_bug.cgi?id=787593)  
34. Bug 792034 – implement dmabuf-import negotiation with upstream \- GNOME Bugzilla, 9月 10, 2025にアクセス、 [https://bugzilla.gnome.org/show\_bug.cgi?id=792034](https://bugzilla.gnome.org/show_bug.cgi?id=792034)  
35. kmssink \- GStreamer, 9月 10, 2025にアクセス、 [https://gstreamer.freedesktop.org/documentation/kms/index.html](https://gstreamer.freedesktop.org/documentation/kms/index.html)  
36. AM62A7: gstreamer, Whether dual-layer (plane\_id=31,plane\_id=41 in DRM) display can be implemented \- Processors forum \- TI E2E, 9月 10, 2025にアクセス、 [https://e2e.ti.com/support/processors-group/processors/f/processors-forum/1278406/am62a7-gstreamer-whether-dual-layer-plane\_id-31-plane\_id-41-in-drm-display-can-be-implemented](https://e2e.ti.com/support/processors-group/processors/f/processors-forum/1278406/am62a7-gstreamer-whether-dual-layer-plane_id-31-plane_id-41-in-drm-display-can-be-implemented)  
37. Output to both HDMI ports simultaneously using gstreamer and kmssink, 9月 10, 2025にアクセス、 [https://forums.raspberrypi.com/viewtopic.php?t=372496](https://forums.raspberrypi.com/viewtopic.php?t=372496)  
38. How To Push DMA nvbuffer In To Gstreamer Pipeline? \- NVIDIA Developer Forums, 9月 10, 2025にアクセス、 [https://forums.developer.nvidia.com/t/how-to-push-dma-nvbuffer-in-to-gstreamer-pipeline/216193](https://forums.developer.nvidia.com/t/how-to-push-dma-nvbuffer-in-to-gstreamer-pipeline/216193)  
39. Memory Leackage in Gstreamer with DMA buffer \- Jetson TX2 \- NVIDIA Developer Forums, 9月 10, 2025にアクセス、 [https://forums.developer.nvidia.com/t/memory-leackage-in-gstreamer-with-dma-buffer/303752](https://forums.developer.nvidia.com/t/memory-leackage-in-gstreamer-with-dma-buffer/303752)  
40. DMA-BUF Sharing \- PipeWire, 9月 10, 2025にアクセス、 [https://docs.pipewire.org/page\_dma\_buf.html](https://docs.pipewire.org/page_dma_buf.html)  
41. Efficient Streaming using Linux DRM Modifiers | GStreamer Conference 2023 \- YouTube, 9月 10, 2025にアクセス、 [https://www.youtube.com/watch?v=P3wxuCQYWpY](https://www.youtube.com/watch?v=P3wxuCQYWpY)  
42. GStreamer 1.26: Improved hardware efficiency, the MPEG-5 LCEVC codec, and more, 9月 10, 2025にアクセス、 [https://www.collabora.com/news-and-blog/news-and-events/gstreamer-126-improved-hardware-efficiency.html](https://www.collabora.com/news-and-blog/news-and-events/gstreamer-126-improved-hardware-efficiency.html)  
43. GStreamer 1.24 release notes \- Freedesktop.org, 9月 10, 2025にアクセス、 [https://gstreamer.freedesktop.org/releases/1.24/](https://gstreamer.freedesktop.org/releases/1.24/)  
44. DMABuf modifier negotiation in GStreamer, 9月 10, 2025にアクセス、 [https://blogs.igalia.com/vjaquez/dmabuf-modifier-negotiation-in-gstreamer/](https://blogs.igalia.com/vjaquez/dmabuf-modifier-negotiation-in-gstreamer/)  
45. GstVideoInfoDmaDrm \- GStreamer, 9月 10, 2025にアクセス、 [https://gstreamer.freedesktop.org/documentation/video/video-info-dma-drm.html](https://gstreamer.freedesktop.org/documentation/video/video-info-dma-drm.html)  
46. Gstreamer drm sink, zero copy \- Raspberry Pi Forums, 9月 10, 2025にアクセス、 [https://forums.raspberrypi.com/viewtopic.php?t=375855](https://forums.raspberrypi.com/viewtopic.php?t=375855)  
47. Bug 699382 – v4l2: dmabuf handling is not complete \- GNOME Bugzilla, 9月 10, 2025にアクセス、 [https://bugzilla.gnome.org/show\_bug.cgi?id=699382](https://bugzilla.gnome.org/show_bug.cgi?id=699382)  
48. How to debug pipeline hangs \- General Discussion \- GStreamer Discourse, 9月 10, 2025にアクセス、 [https://discourse.gstreamer.org/t/how-to-debug-pipeline-hangs/3889](https://discourse.gstreamer.org/t/how-to-debug-pipeline-hangs/3889)  
49. gst-launch-1.0(1) — gstreamer1.0-tools — Debian experimental, 9月 10, 2025にアクセス、 [https://manpages.debian.org/experimental/gstreamer1.0-tools/gst-launch-1.0.1.en.html](https://manpages.debian.org/experimental/gstreamer1.0-tools/gst-launch-1.0.1.en.html)  
50. Basic tutorial 11: Debugging tools \- GStreamer, 9月 10, 2025にアクセス、 [https://gstreamer.freedesktop.org/documentation/tutorials/basic/debugging-tools.html](https://gstreamer.freedesktop.org/documentation/tutorials/basic/debugging-tools.html)  
51. GST\_DEBUG \- UG1449, 9月 10, 2025にアクセス、 [https://docs.amd.com/r/en-US/ug1449-multimedia/GST\_DEBUG](https://docs.amd.com/r/en-US/ug1449-multimedia/GST_DEBUG)  
52. vc4-drm gpu: \[drm\] \*ERROR\* Failed to allocate DLIST entry: \-28 \- Raspberry Pi Forums, 9月 10, 2025にアクセス、 [https://forums.raspberrypi.com/viewtopic.php?t=357826](https://forums.raspberrypi.com/viewtopic.php?t=357826)  
53. Can't unload nvidia-drm module: Freeze with "GspRmAlloc failed" error in dmesg \- Linux, 9月 10, 2025にアクセス、 [https://forums.developer.nvidia.com/t/cant-unload-nvidia-drm-module-freeze-with-gsprmalloc-failed-error-in-dmesg/335527](https://forums.developer.nvidia.com/t/cant-unload-nvidia-drm-module-freeze-with-gsprmalloc-failed-error-in-dmesg/335527)  
54. Running and debugging GStreamer Applications, 9月 10, 2025にアクセス、 [https://gstreamer.freedesktop.org/documentation/gstreamer/running.html](https://gstreamer.freedesktop.org/documentation/gstreamer/running.html)  
55. Gst.Debug – gstreamer-1.0 \- Valadoc, 9月 10, 2025にアクセス、 [https://valadoc.org/gstreamer-1.0/Gst.Debug.html](https://valadoc.org/gstreamer-1.0/Gst.Debug.html)