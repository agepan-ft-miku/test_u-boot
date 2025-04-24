# SPDX-License-Identifier: GPL-2.0+
#
# (C) Copyright 2000 - 2013
# Wolfgang Denk, DENX Software Engineering, wd@denx.de.

概要:
========

このディレクトリには、U-Boot のソースコードが含まれています。U-Boot は、PowerPC、ARM、MIPS およびその他いくつかのプロセッサをベースにした組み込みボード用のブートローダーであり、ブート ROM にインストールしてハードウェアの初期化やテスト、またはアプリケーションコードのダウンロードと実行に使用できます。

U-Boot の開発は Linux と密接に関連しています。ソースコードの一部は Linux のソースツリーに由来し、共通のヘッダーファイルもいくつかあり、Linux イメージのブートをサポートするために特別な準備がなされています。

このソフトウェアを容易に設定可能かつ拡張可能にするために、いくつかの注意が払われています。 例えば、すべてのモニターコマンドは同じ呼び出しインターフェースで実装されているため、新しいコマンドを追加するのは非常に簡単です。 また、めったに使用されないコード（例えばハードウェアテストユーティリティ）をモニターに恒久的に追加する代わりに、動的にロードして実行することができます。

ステータス:
=======

一般的に、Makefile に設定オプションが存在するすべてのボードは、ある程度テストされており、「動作している」と見なすことができます。 実際、それらの多くは生産システムで使用されています。

問題が発生した場合は、CHANGELOG ファイルを参照して、特定のポートを誰が貢献したかを確認してください。 さらに、U-Boot ソース全体に散在する様々な MAINTAINERS ファイルがあり、様々なボードやサブシステムを担当する人物や企業が特定されています。

注意: 2010年8月現在、実際の U-Boot ソースツリーには CHANGELOG ファイルはもうありません。 しかし、以下のコマンドを使用して Git のログから動的に生成することができます。

	make CHANGELOG

ヘルプの入手先:
==================

U-Boot に関する質問、問題、または貢献がある場合は、<u-boot@lists.denx.de> の U-Boot メーリングリストにメッセージを送信してください。
メーリングリストの過去のやり取りのアーカイブもあります - FAQ を尋ねる前にアーカイブを検索してください。
https://lists.denx.de/pipermail/u-boot および https://marc.info/?l=u-boot を参照してください。

ソースコードの入手先:
=========================

U-Boot のソースコードは https://source.denx.de/u-boot/u-boot.git の Git リポジトリで管理されています。
https://source.denx.de/u-boot/u-boot でオンラインで閲覧できます。

このページの「Tags」リンクから、興味のある任意のバージョンの tarball をダウンロードできます。公式リリースは、HTTPS または FTP を介して DENX ファイルサーバーからも入手可能です。
https://ftp.denx.de/pub/u-boot/
ftp://ftp.denx.de/pub/u-boot/

これまでの経緯:
===================

-   8xxrom ソースから開始
-   PPCBoot プロジェクトを作成 (https://sourceforge.net/projects/ppcboot)
-   コードのクリーンアップ
-   カスタムボードの追加を容易にする
-   他の [PowerPC] CPU の追加を可能にする
-   機能拡張、特に:
    * Linux ブートローダーへの拡張インターフェースの提供
    * S-Record ダウンロード
    * ネットワークブート
    * ATA ディスク / SCSI ... ブート
-   ARMBoot プロジェクトを作成 (https://sourceforge.net/projects/armboot)
-   他の CPU ファミリを追加 (ARM から開始)
-   U-Boot プロジェクトを作成 (https://sourceforge.net/projects/u-boot)
-   現在のプロジェクトページ: https://www.denx.de/wiki/U-Boot を参照

名称とスペル:
===================

このプロジェクトの「公式」名称は "Das U-Boot" です。
スペル "U-Boot" は、すべての記述テキスト（ドキュメント、ソースファイル内のコメントなど）で使用されるものとします。 例:

	これは U-Boot プロジェクトの README ファイルです。

ファイル名などは文字列 "u-boot" に基づくものとします。 例:

	include/asm-ppc/u-boot.h

	#include <asm/u-boot.h>

変数名、プリプロセッサ定数などは、文字列 "u_boot" または "U_BOOT" のいずれかに基づくものとします。 例:

	U_BOOT_VERSION		u_boot_logo
	IH_OS_U_BOOT		u_boot_hush_start

バージョニング:
===========

2008年10月のリリース以降、リリースの名称は、深い意味のない数値のリリース番号から、タイムスタンプベースの番号付けに変更されました。
通常リリースは、リリース日の暦年と月で構成される名前で識別されます。 追加のフィールド（存在する場合）は、リリース候補または「安定版」メンテナンストゥリーでのバグ修正リリースを示します。 例:
	U-Boot v2009.11	    - 2009年11月リリース
	U-Boot v2009.11.1   - バージョン 2009年11月安定版ツリーのリリース1
	U-Boot v2010.09-rc1 - 2010年9月リリースのリリース候補1

ディレクトリ階層:
====================
/arch			アーキテクチャ固有ファイル
/arc			ARC アーキテクチャ汎用ファイル
/arm			ARM アーキテクチャ汎用ファイル
/m68k			m68k アーキテクチャ汎用ファイル
/microblaze		MicroBlaze アーキテクチャ汎用ファイル
/mips			MIPS アーキテクチャ汎用ファイル
/nds32		NDS32 アーキテクチャ汎用ファイル
/nios2		Altera NIOS2 アーキテクチャ汎用ファイル
/powerpc		PowerPC アーキテクチャ汎用ファイル
/riscv		RISC-V アーキテクチャ汎用ファイル
/sandbox		HW 非依存「サンドボックス」汎用ファイル
/sh			SH アーキテクチャ汎用ファイル
/x86			x86 アーキテクチャ汎用ファイル
/xtensa		Xtensa アーキテクチャ汎用ファイル
/api			外部アプリ用マシン/アーキテクチャ非依存 API
/board			ボード依存ファイル
/cmd			U-Boot コマンド関数
/common			様々なアーキテクチャ非依存関数
/configs		ボードデフォルト設定ファイル
/disk			ディスクドライブパーティション処理用コード
/doc			ドキュメント (ReST と README の混在)
/drivers		デバイスドライバ
/dts			内部 U-Boot fdt ビルド用 Makefile
/env			環境変数サポート
/examples		スタンドアロンアプリケーション等のサンプルコード
/fs			ファイルシステムコード (cramfs, ext2, jffs2, etc.)
/include		ヘッダーファイル
/lib			全アーキテクチャ汎用ライブラリルーチン
/Licenses		各種ライセンスファイル
/net			ネットワークコード
/post			Power On Self Test (自己診断)
/scripts		各種ビルドスクリプトと Makefile
/test			各種ユニットテストファイル
/tools			FIT イメージ等のビルド・署名ツール


ソフトウェア設定:
=======================

設定は通常、C プリプロセッサ定義を使用して行われます。 その背後にある理論的根拠は、可能な限りデッドコードを避けることです。 設定変数には2つのクラスがあります。

* 設定 **OPTIONS**:
    これらはユーザーが選択可能で、名前は "CONFIG_" で始まります。
* 設定 **SETTINGS**:
    これらはハードウェアなどに依存し、何をしているか理解していない場合はいじるべきではありません。 名前は "CONFIG_SYS_" で始まります。

以前は、すべての設定は手作業で行われ、シンボリックリンクの作成や設定ファイルの手動編集が必要でした。最近では、U-Boot は Linux カーネルで使用される Kbuild インフラストラクチャを追加し、「make menuconfig」コマンドを使用してビルドを設定できるようになりました。

プロセッサアーキテクチャとボードタイプの選択:
---------------------------------------------------

サポートされているすべてのボードには、すぐに使用できるデフォルト設定が用意されています。「make <board_name>_defconfig」と入力するだけです。 例: TQM823L モジュールの場合:

	cd u-boot
	make TQM823L_defconfig

注意: 以前は存在していたはずなのに見つからないボードのデフォルト設定ファイルを探している場合は、サポートされなくなったボードのリストについて doc/README.scrapyard ファイルを確認してください。

サンドボックス環境:
--------------------

U-Boot は 'sandbox' ボードを使用して、Linux ホスト上で実行するためにネイティブにビルドできます。 これにより、ボードやアーキテクチャに固有でない機能開発をネイティブプラットフォームで行うことができます。 サンドボックスは、U-Boot のテストの一部を実行するためにも使用されます。

詳細は doc/arch/sandbox.rst を参照してください。

ボード初期化フロー:
--------------------------

これはボードの意図された起動フローです。 これは SPL と U-Boot 本体（つまり、両方とも同じルールに従う）の両方に適用されるべきです。 注意: "SPL" は "Secondary Program Loader" の略で、このファイルの後半で詳しく説明されています。 現在、SPL はほとんど別のコードパスを使用していますが、各関数の名前と役割は同じです。 一部のボードやアーキテクチャはこれに準拠していない場合があります。 少なくとも CONFIG_SPL_FRAMEWORK を使用するほとんどの ARM ボードはこれに準拠しています。

実行は通常、アーキテクチャ固有（そしておそらく CPU 固有）の start.S ファイルから始まります。例:

	- arch/arm/cpu/armv7/start.S
	- arch/powerpc/cpu/mpc83xx/start.S
	- arch/mips/cpu/start.S

など。そこから3つの関数が呼び出されます。これらの各関数の目的と制限は以下の通りです。

`lowlevel_init()`:
	- 目的: `board_init_f()` に実行が到達できるようにするための必須の初期化
	- `global_data` や BSS なし
	- スタックなし（ARMv7 にはあるかもしれないが、間もなく削除される予定）
	- SDRAM の設定やコンソールの使用は禁止
	- `board_init_f()` へ実行を続けるための最低限のことだけを行うこと
	- これはほとんどの場合不要
	- この関数から正常にリターンすること

`board_init_f()`:
	- 目的: `board_init_r()` を実行する準備が整うようにマシンを設定する: つまり SDRAM とシリアル UART
	- `global_data` が利用可能
	- スタックは SRAM 内
	- BSS は利用不可、したがってグローバル/静的変数は使用できず、スタック変数と `global_data` のみ使用可能

	非 SPL 固有の注意点:
	- DRAM を設定するために `dram_init()` が呼び出される。 SPL で既に実行済みの場合は何もしなくてもよい。

	SPL 固有の注意点:
	- 必要に応じて `board_init_f()` 関数全体を独自のバージョンでオーバーライドできる。
	- 極端な場合、`preloader_console_init()` をここで呼び出すことができる
	- SDRAM と、UART が動作するために必要なものを設定する必要がある
	- BSS をクリアする必要はない、それは crt0.S によって行われる
	- 特定のアーキテクチャ上の特定のシナリオでは、早期 BSS が利用可能になる場合がある（`board_init_f()` に入る前に BSS のクリアを移動することによる `CONFIG_SPL_EARLY_BSS` 経由）が、そうすることは推奨されない。 代わりに、コードベース全体で互換性と一貫性を維持するために、この README の他のセクションで示されているように、`board_init_f()` 中に BSS が利用可能であることに依存しないように、コードの変更や追加を設計することが強く推奨される。
	- この関数から正常にリターンする必要がある（`board_init_r()` を直接呼び出さないこと）

ここで BSS がクリアされる。
SPL の場合、`CONFIG_SPL_STACK_R` が定義されている場合、この時点でスタックと `global_data` は `CONFIG_SPL_STACK_R_ADDR` の下に再配置される。 非 SPL の場合、U-Boot はメモリの最上部に再配置されて実行される。

`board_init_r()`:
	- 目的: メインの実行、共通コード
	- `global_data` が利用可能
	- SDRAM が利用可能
	- BSS が利用可能、すべての静的/グローバル変数が使用可能
	- 実行は最終的に `main_loop()` に続く

	非 SPL 固有の注意点:
	- U-Boot はメモリの最上部に再配置され、そこから実行されている。
	SPL 固有の注意点:
	- `CONFIG_SPL_STACK_R` が定義され、`CONFIG_SPL_STACK_R_ADDR` が SDRAM を指している場合、スタックはオプションで SDRAM 内にある
	- `preloader_console_init()` はここで呼び出すことができる - 通常、これは `CONFIG_SPL_BOARD_INIT` を選択し、この呼び出しを含む `spl_board_init()` 関数を提供することによって行われる
	- U-Boot または (ファルコンモードでは) Linux をロードする

設定オプション:
----------------------

設定はボードと CPU タイプの組み合わせに依存します。 すべての情報は設定ファイル "include/configs/<board_name>.h" に保持されます。

例: TQM823L モジュールの場合、すべての設定は "include/configs/TQM823L.h" にあります。
オプションの多くは、対応する Linux カーネル設定オプションとまったく同じ名前が付けられています。 その意図は、後で設定ツールを構築しやすくするためです。

-   ARM プラットフォームバスタイプ(CCI):
    CoreLink Cache Coherent Interconnect (CCI) は、マルチコア CPU の2つのクラスター間の完全なキャッシュコヒーレンシと、デバイスおよび I/O マスターの I/O コヒーレンシを提供する ARM BUS です。

    `CONFIG_SYS_FSL_HAS_CCI400`

    キャッシュコヒーレントインターコネクト CCN-400 を持つ SoC 向けに定義

    `CONFIG_SYS_FSL_HAS_CCN504`

    キャッシュコヒーレントインターコネクト CCN-504 を持つ SoC 向けに定義

以下のオプションを設定する必要があります:

-   CPU タイプ: 正確に1つ定義します。例: `CONFIG_MPC85XX`。

-   ボードタイプ: 正確に1つ定義します。例: `CONFIG_MPC8540ADS`。

-   85xx CPU オプション:
    `CONFIG_SYS_PPC64`

    コアが 64 ビット PowerPC 実装であることを指定します（Power ISA の "64" カテゴリを実装）。 これは、他の考えられる理由の中でも特に ePAPR 準拠に必要です。

    `CONFIG_SYS_FSL_TBCLK_DIV`

    システムクロックと比較したコアタイムベースクロックの分周比を定義します。ほとんどの PQ3 デバイスでは 8 ですが、新しい QorIQ デバイスでは 16 または 32 になることがあります。 比率は SoC ごとに異なります。

    `CONFIG_SYS_FSL_PCIE_COMPAT`

    指定されたプラットフォームの PCIe デバイスツリーノードを照合しようとするときに使用する文字列を定義します。

    `CONFIG_SYS_FSL_ERRATUM_A004510`

    エラッタ A004510 の回避策を有効にします。設定されている場合、`CONFIG_SYS_FSL_ERRATUM_A004510_SVR_REV` と `CONFIG_SYS_FSL_CORENET_SNOOPVEC_COREONLY` を設定する必要があります。

    `CONFIG_SYS_FSL_ERRATUM_A004510_SVR_REV`
    `CONFIG_SYS_FSL_ERRATUM_A004510_SVR_REV2` (オプション)

    A004510 回避策を適用すべき SoC リビジョン（SVR の下位 8 ビット）を1つまたは2つ定義します。 SVR の残りの部分は、エラッタが存在するかどうかの決定には関係ない（例: p2040 対 p2041）か、`CONFIG_SYS_FSL_ERRATUM_A004510` が設定されているかどうかを制御するビルドターゲットによって暗黙的に示されます。 このエラッタに関する詳細については、Freescale App Note 4493 を参照してください。

    `CONFIG_A003399_NOR_WORKAROUND`
    IFC エラッタ A003399 の回避策を有効にします。これは NOR ブート中にのみ必要です。

    `CONFIG_A008044_WORKAROUND`
    T1040/T1042 エラッタ A008044 の回避策を有効にします。これは NAND ブート中にのみ必要であり、Rev 1.0 SoC リビジョンに対して有効です。

    `CONFIG_SYS_FSL_CORENET_SNOOPVEC_COREONLY`

    これは A004510 回避策に従って CCSR オフセット 0x18600 に書き込む値です。

    `CONFIG_SYS_FSL_DSP_DDR_ADDR`
    この値は、DSP コア専用に接続されている DDR メモリの開始オフセットを示します。

    `CONFIG_SYS_FSL_DSP_M2_RAM_ADDR`
    この値は、DSP コアに直接接続されている M2 メモリの開始オフセットを示します。

    `CONFIG_SYS_FSL_DSP_M3_RAM_ADDR`
    この値は、DSP コアに直接接続されている M3 メモリの開始オフセットを示します。

    `CONFIG_SYS_FSL_DSP_CCSRBAR_DEFAULT`
    この値は、DSP CCSR 空間の開始オフセットを示します。

    `CONFIG_SYS_FSL_SINGLE_SOURCE_CLK`
    シングルソースクロックは、一部の FSL SoC に存在するクロックモードです。 このモードでは、単一の差動クロックがシステムクロック、DDR クロック、USB クロックにクロックを供給するために使用されます。

    `CONFIG_SYS_CPC_REINIT_F`
    この CONFIG は、U-Boot エントリ時に CPC が SRAM として設定されており、再初期化が必要な場合に定義されます。

    `CONFIG_DEEP_SLEEP`
    この SoC がディープスリープ機能をサポートしていることを示します。ディープスリープがサポートされている場合、コアはウェイクアップ時に uboot の実行を開始します。

-   汎用 CPU オプション:
    `CONFIG_SYS_BIG_ENDIAN`, `CONFIG_SYS_LITTLE_ENDIAN`

    CPU のエンディアンを定義します。これらの値の実装はアーキテクチャ固有です。

    `CONFIG_SYS_FSL_DDR`
    Freescale DDR ドライバが使用中です。このタイプの DDR コントローラは、mpc83xx、mpc85xx、および一部の ARM コア SoC に搭載されています。

    `CONFIG_SYS_FSL_DDR_ADDR`
    Freescale DDR メモリマップドレジスタベース。

    `CONFIG_SYS_FSL_DDR_EMU`
    DDR のエミュレータサポートを指定します。デスクュートレーニングなどの一部の DDR 機能は利用できません。

    `CONFIG_SYS_FSL_DDRC_GEN1`
    Freescale DDR1 コントローラ。

    `CONFIG_SYS_FSL_DDRC_GEN2`
    Freescale DDR2 コントローラ。

    `CONFIG_SYS_FSL_DDRC_GEN3`
    Freescale DDR3 コントローラ。

    `CONFIG_SYS_FSL_DDRC_GEN4`
    Freescale DDR4 コントローラ。

    `CONFIG_SYS_FSL_DDRC_ARM_GEN3`
    ARM ベース SoC 用 Freescale DDR3 コントローラ。

    `CONFIG_SYS_FSL_DDR1`
    DDR1 を使用するためのボード設定。これは、ボードの実装に応じて、Freescale DDR1 または DDR2 コントローラを持つ SoC に対して有効にできます。

    `CONFIG_SYS_FSL_DDR2`
    DDR2 を使用するためのボード設定。これは、ボードの実装に応じて、Freescale DDR2 または DDR3 コントローラを持つ SoC に対して有効にできます。

    `CONFIG_SYS_FSL_DDR3`
    DDR3 を使用するためのボード設定。これは、Freescale DDR3 または DDR3L コントローラを持つ SoC に対して有効にできます。

    `CONFIG_SYS_FSL_DDR3L`
    DDR3L を使用するためのボード設定。これは、DDR3L コントローラを持つ SoC に対して有効にできます。

    `CONFIG_SYS_FSL_DDR4`
    DDR4 を使用するためのボード設定。これは、DDR4 コントローラを持つ SoC に対して有効にできます。

    `CONFIG_SYS_FSL_IFC_BE`
    IFC コントローラレジスタ空間をビッグエンディアンとして定義します。

    `CONFIG_SYS_FSL_IFC_LE`
    IFC コントローラレジスタ空間をリトルエンディアンとして定義します。

    `CONFIG_SYS_FSL_IFC_CLK_DIV`
    プラットフォームクロック（IFC コントローラへのクロック入力）の分周器を定義します。

    `CONFIG_SYS_FSL_LBC_CLK_DIV`
    プラットフォームクロック（eLBC コントローラへのクロック入力）の分周器を定義します。

    `CONFIG_SYS_FSL_PBL_PBI`
    ビルドイメージに RCW（Power on reset configuration）の追加を有効にします。 詳細については doc/README.pblimage を参照してください。

    `CONFIG_SYS_FSL_PBL_RCW`
    u-boot ビルドイメージに PBI（プリブート命令）コマンドを追加します。 PBI コマンドは、実行を開始する前に SoC を設定するために使用できます。 詳細については doc/README.pblimage を参照してください。

    `CONFIG_SYS_FSL_DDR_BE`
    DDR コントローラレジスタ空間をビッグエンディアンとして定義します。

    `CONFIG_SYS_FSL_DDR_LE`
    DDR コントローラレジスタ空間をリトルエンディアンとして定義します。

    `CONFIG_SYS_FSL_DDR_SDRAM_BASE_PHY`
    DDR コントローラから見た物理アドレス。 これは、すべての Power SoC において `CONFIG_SYS_DDR_SDRAM_BASE` と同じです。しかし、ARM SoC では異なる場合があります。

    `CONFIG_SYS_FSL_DDR_INTLV_256B`
    256 バイトでの DDR コントローラインターリービング。これは、ARM コアを持つ Freescale Layerscape SoC 向けに Dickens によって処理される特別なインターリービングモードです。

    `CONFIG_SYS_FSL_DDR_MAIN_NUM_CTRLS`
    メインメモリとして使用されるコントローラの数。

    `CONFIG_SYS_FSL_OTHER_DDR_NUM_CTRLS`
    メインメモリ以外に使用されるコントローラの数。

    `CONFIG_SYS_FSL_HAS_DP_DDR`
    SoC が DPAA 用に使用される DP-DDR を持っていることを定義します。

    `CONFIG_SYS_FSL_SEC_BE`
    SEC コントローラレジスタ空間をビッグエンディアンとして定義します。

    `CONFIG_SYS_FSL_SEC_LE`
    SEC コントローラレジスタ空間をリトルエンディアンとして定義します。

-   MIPS CPU オプション:
    `CONFIG_SYS_INIT_SP_OFFSET`

    初期スタックポインタのための `CONFIG_SYS_SDRAM_BASE` からのオフセット。これは再配置前の一時スタックに必要です。

    `CONFIG_XWAY_SWAP_BYTES`

    NOR フラッシュからのブートに必要な Lantiq XWAY SoC 用の tools/xway-swap-bytes のコンパイルを有効にします。 フラッシュプログラマを使用する場合、U-Boot イメージをスワップする必要があります。

-   ARM オプション:
    `CONFIG_SYS_EXCEPTION_VECTORS_HIGH`

    ARM コアの高位例外ベクターを選択します。例: CP15 の c1 レジスタの V ビットをクリアしません。

    `COUNTER_FREQUENCY`
    汎用タイマークロックソース周波数。

    `COUNTER_FREQUENCY_REAL`
    実際のクロックが `COUNTER_FREQUENCY` と異なり、実行時にしか決定できない場合の汎用タイマークロックソース周波数。

-   Tegra SoC オプション:
    `CONFIG_TEGRA_SUPPORT_NON_SECURE`

    非セキュア（NS）モードでの U-Boot の実行をサポートします。CPU が NS モードの場合、ARM アーキテクチャタイマ初期化など、特定の不可能なアクションはスキップされます。

-   Linux カーネルインターフェース:
    `CONFIG_MEMSIZE_IN_BYTES`		[MIPS のみ関連]

    memsize パラメータを Linux に転送する際、一部のバージョンではバイト単位、他のバージョンでは MB 単位を期待します。バイト単位にするには `CONFIG_MEMSIZE_IN_BYTES` を定義します。

    `CONFIG_OF_LIBFDT`

    新しいカーネルバージョンは、ファームウェア設定がフラット化されたデバイスツリー（オープンファームウェアコンセプトに基づく）を使用して渡されることを期待しています。

    `CONFIG_OF_LIBFDT`
     * 新しい libfdt ベースのサポート
     * "fdt" コマンドを追加
     * bootm コマンドが fdt を自動的に更新

    `OF_TBCLK` - タイムベース周波数。 QUICC エンジンを持つボードでは、UCC MAC アドレスを設定するために `OF_QE` が必要です。

    `CONFIG_OF_BOARD_SETUP`

    ボードコードには、カーネルに渡す前にフラットデバイスツリーに加えたい追加の変更があります。

    `CONFIG_OF_SYSTEM_SETUP`

    他のコードには、カーネルに渡す前にフラットデバイスツリーに加えたい追加の変更があります。これにより、カーネルをブートする前に `ft_system_setup()` が呼び出されます。

    `CONFIG_OF_IDE_FIXUP`

    U-Boot は IDE デバイスが存在するかどうかを検出できます。 存在しない場合、この新しい設定オプションが有効になっていると、U-Boot は Linux をブートする前に DTS から ATA ノードを削除します。これにより、Linux IDE ドライバがデバイスをプローブしてクラッシュするのを防ぎます。 これは、信号 IDE5V_DD7 にプルダウン抵抗が接続されていないバグのあるハードウェア（uc101）で必要です。

    `CONFIG_MACH_TYPE`	[ARM のみ関連][必須]

    この設定は、マシンタイプが1つしかないすべてのボードで必須であり、ARM マシンレジストリに表示されるマシンタイプ番号を指定するために使用する必要があります（https://www.arm.linux.org.uk/developer/machines/ を参照）。 単一の設定ファイルで複数のマシンタイプがサポートされており、マシンタイプが実行時に検出可能なボードのみ、この設定を使用する必要はありません。

-   vxWorks ブートパラメータ:

    bootvx は、以下の環境変数を使用して有効なブートラインを構築します: bootdev, bootfile, ipaddr, netmask, serverip, gatewayip, hostname, othbootargs。 bootfile が指す vxWorks イメージをロードします。

    注意: "bootargs" 環境変数が定義されている場合、それは上記のデフォルトを上書きします。

-   キャッシュ設定:
    `CONFIG_SYS_L2CACHE_OFF`- U-Boot で L2 キャッシュを有効にしません

-   ARM のキャッシュ設定:
    `CONFIG_SYS_L2_PL310` - ARM PL310 L2 キャッシュコントローラのサポートを有効にします
    `CONFIG_SYS_PL310_BASE` - PL310 コントローラレジスタ空間の物理ベースアドレス

-   シリアルポート:
    `CONFIG_PL011_SERIAL`

    Amba PrimeCell PL011 UART のサポートが必要な場合にこれを定義します。

    `CONFIG_PL011_CLOCK`

    Amba PrimeCell PL011 UART がある場合、この変数を UART のクロック速度に設定します。

    `CONFIG_PL01x_PORTS`

    ボードに Amba PrimeCell PL010 または PL011 UART がある場合、これを各（サポートされている）ポートのベースアドレスのリストに定義します。 例: include/configs/versatile.h を参照。

    `CONFIG_SERIAL_HW_FLOW_CONTROL`

    シリアルドライバでハードウェアフロー制御を有効にするには、この変数を定義します。このオプションの現在のユーザーは drivers/serial/nsl16550.c ドライバです。

-   自動ブートコマンド:
    `CONFIG_BOOTCOMMAND`
    `CONFIG_BOOTDELAY` が有効な場合にのみ必要です。 リセット後、「ブート遅延」内にコンソールインターフェイスで文字が読み取られなかった場合に自動的に実行されるコマンド文字列を定義します。

    `CONFIG_RAMBOOT` および `CONFIG_NFSBOOT`
    これらの値は、それぞれ "ramboot" および "nfsboot" として環境変数に入り、RAM からのブートと NFS からのブートを切り替える際の便宜のために使用できます。

-   シリアルダウンロードエコーモード:
    `CONFIG_LOADS_ECHO`
    1 に定義されている場合、シリアルダウンロード（"loads" コマンドを使用）中に受信されたすべての文字がエコーバックされます。 これは一部のターミナルエミュレーション（"cu" など）で必要になる場合がありますが、他のエミュレーションでは単に時間がかかるだけかもしれません。 この設定は、"loads_echo" 環境変数の初期値を #define します。

-   Kgdb シリアルボーレート: (`CONFIG_CMD_KGDB` が定義されている場合)
    `CONFIG_KGDB_BAUDRATE`
    以下に示す `CONFIG_SYS_BAUDRATE_TABLE` にリストされているボーレートのいずれかを選択します。

-   コマンドの削除
    ブートにコマンドが必要ない場合は、`CONFIG_CMDLINE` を無効にして削除できます。 この場合、コマンドラインは利用できなくなり、U-Boot がブートコマンドを実行しようとするとき（起動時）、代わりに `board_run_command()` を呼び出します。 これにより、非常に単純なブート手順のイメージサイズを大幅に削減できます。

-   正規表現サポート:
    `CONFIG_REGEX`
    この変数が定義されている場合、U-Boot は SLRE（Super Light Regular Expression）ライブラリに対してリンクされ、"env grep" や "setexpr" などの一部のコマンドに正規表現サポートが追加されます。

-   デバイスツリー:
    `CONFIG_OF_CONTROL`
    この変数が定義されている場合、U-Boot はボードファイル内の静的にコンパイルされた #define に依存する代わりに、デバイスツリーを使用してデバイスを設定します。 このオプションは実験的であり、一部のボードでのみ利用可能です。 デバイスツリーはグローバルデータ `gd->fdt_blob` として利用可能です。
    U-Boot はどこかからデバイスツリーを取得する必要があります。これは、以下の3つのオプションのいずれかを使用して行うことができます。

    `CONFIG_OF_EMBED`
    この変数が定義されている場合、U-Boot はデバイスツリーバイナリをイメージに埋め込みます。 このデバイスツリーファイルはボードディレクトリにあり、`<soc>-<board>.dts` という名前である必要があります。 バイナリファイルは `board_init_f()` で取得され、グローバルデータ構造体 `gd->fdt_blob` を介して利用可能になります。

    `CONFIG_OF_SEPARATE`
    この変数が定義されている場合、U-Boot はデバイスツリーバイナリをビルドします。それは u-boot.dtb という名前になります。 アーキテクチャ固有のコードが実行時にそれを見つけます。一般的に、これは次のように機能します。

    		cat u-boot.bin u-boot.dtb >image.bin

    そして実際、U-Boot はこれを行い、一般的なケースで役立つ u-boot-dtb.bin というファイルを生成します。 より特殊なものが必要な場合は、個々のファイルを引き続き使用できます。

    `CONFIG_OF_BOARD`
    この変数が定義されている場合、U-Boot はイメージに1つ埋め込む代わりに、実行時にボードによって提供されるデバイスツリーを使用します。 `board_fdt_blob_setup()` を定義しているボードのみがこのオプションをサポートします（include/fdtdec.h ファイルを参照）。

-   ウォッチドッグ:
    `CONFIG_WATCHDOG`
    この変数が定義されている場合、SoC のウォッチドッグサポートが有効になります。SoC 固有のコードにウォッチドッグのサポートが必要です。 8xx CPU の場合、SIU ウォッチドッグ機能が SYPCR レジスタで有効になります。 特定の SoC でサポートが利用可能な場合、それを使用するために追加のボード固有コードは必要ありません。

    `CONFIG_HW_WATCHDOG`
    使用する SoC の外部にあるウォッチドッグ回路を使用する場合、この変数を定義し、「hw_watchdog_reset」関数のためのボード固有コードを提供します。

    `CONFIG_SYS_WATCHDOG_FREQ`
    一部のプラットフォームでは、タイマー割り込みハンドラから `CONFIG_SYS_WATCHDOG_FREQ` 回の割り込みごとに `WATCHDOG_RESET()` を自動的に呼び出します。 ボード設定ファイルで設定されていない場合、デフォルトの `CONFIG_SYS_HZ/2`（つまり 500）が使用されます。 `CONFIG_SYS_WATCHDOG_FREQ` を 0 に設定すると、タイマー割り込みからの `WATCHDOG_RESET()` の呼び出しが無効になります。

-   リアルタイムクロック:

    `CONFIG_CMD_DATE` が選択されている場合、RTC のタイプも選択する必要があります。 以下のオプションのいずれか1つを正確に定義します。

    `CONFIG_RTC_PCF8563`	- Philips PCF8563 RTC を使用
    `CONFIG_RTC_MC13XXX`	- MC13783 または MC13892 RTC を使用
    `CONFIG_RTC_MC146818`	- MC146818 RTC を使用
    `CONFIG_RTC_DS1307`	- Maxim, Inc. DS1307 RTC を使用
    `CONFIG_RTC_DS1337`	- Maxim, Inc. DS1337 RTC を使用
    `CONFIG_RTC_DS1338`	- Maxim, Inc. DS1338 RTC を使用
    `CONFIG_RTC_DS1339`	- Maxim, Inc. DS1339 RTC を使用
    `CONFIG_RTC_DS164x`	- Dallas DS164x RTC を使用
    `CONFIG_RTC_ISL1208`	- Intersil ISL1208 RTC を使用
    `CONFIG_RTC_MAX6900`	- Maxim, Inc. MAX6900 RTC を使用
    `CONFIG_RTC_DS1337_NOOSC`	- DS1337 の OSC 出力をオフにする
    `CONFIG_SYS_RV3029_TCR`	- RV3029 RTC のトリクルチャージャーを有効にする。
    RTC が I2C を使用する場合、I2C インターフェースも設定する必要があることに注意してください。以下の I2C サポートを参照してください。

-   GPIO サポート:
    `CONFIG_PCA953X`		- NXP の PCA953X シリーズ I2C GPIO を使用

    `CONFIG_SYS_I2C_PCA953X_WIDTH` オプションは、特定のチップでサポートされるピン数を PCA953X ドライバに伝えるチップ-ngpio ペアのリストを指定します。 GPIO デバイスが I2C を使用する場合、I2C インターフェースも設定する必要があることに注意してください。以下の I2C サポートを参照してください。

-   I/O トレース:
    `CONFIG_IO_TRACE` が選択されている場合、U-Boot はすべての I/O アクセスをインターセプトし、それらをチェックサムするか、メモリにリストを書き出すことができます。 詳細については 'iotrace' コマンドを参照してください。これは、コード変更前後でドライバが同じように動作することを確認できるため、デバイスドライバのテストに役立ちます。 現在、これは sandbox と arm でサポートされています。アーキテクチャにサポートを追加するには、`arch/<arch>/include/asm/io.h` の最後に `#include <iotrace.h>` を追加してテストします。 'iotrace stats' コマンドからの出力例を以下に示します。トレースバッファが使い果たされても、チェックサムは引き続き動作することに注意してください。
    ```
    iotrace is enabled
    	Start:  10000000	(buffer start address)
    	Size:   00010000	(buffer size)
    	Offset: 00000120	(current buffer offset)
    	Output: 10000120	(start + offset)
    	Count:  00000018	(number of trace records)
    	CRC32:  9526fb66	(CRC32 of all trace records)
    ```
-   タイムスタンプサポート:

    `CONFIG_TIMESTAMP` が選択されている場合、イメージのタイムスタンプ（日付と時刻）が bootm や iminfo などのイメージコマンドによって表示されます。このオプションは、`CONFIG_CMD_DATE` を選択すると自動的に有効になります。

-   サポートされるパーティションラベル (disklabel):
    以下のゼロ個以上:
    `CONFIG_MAC_PARTITION`   Apple の MacOS パーティションテーブル。
    `CONFIG_ISO_PARTITION`   ISO パーティションテーブル、CDROM などで使用されます。
    `CONFIG_EFI_PARTITION`   GPT パーティションテーブル、EFI がブートローダーの場合に一般的です。 2TB パーティション制限に注意。disk/part_efi.c を参照してください。
    `CONFIG_SCSI`）の場合、少なくとも1つの非 MTD パーティションタイプのサポートも設定する必要があります。

-   IDE リセットメソッド:
    `CONFIG_IDE_RESET_ROUTINE` - これはいくつかのボード設定ファイルで定義されていますが、どこにも使用されていません！
    `CONFIG_IDE_RESET` - これが定義されている場合、IDE リセットは関数 `ide_set_reset(int reset)` を呼び出すことによって実行されます。これはボード固有のファイルで定義する必要があります。

-   ATAPI サポート:
    `CONFIG_ATAPI`

    ATAPI サポートを有効にするにはこれを設定します。

-   LBA48 サポート
    `CONFIG_LBA48`

    137GB より大きいディスクのサポートを有効にするにはこれを設定します。`CONFIG_SYS_64BIT_LBA` も参照してください。 これらがない場合、LBA48 サポートは 32 ビット変数を使用し、「わずか」2.1TB までのディスクしかサポートしません。

    `CONFIG_SYS_64BIT_LBA`:
    	有効にすると、IDE サブシステムが 64 ビットセクタアドレスを使用するようになります。デフォルトは 32 ビットです。

-   SCSI サポート:
    `CONFIG_SYS_SCSI_MAX_LUN` [8]、`CONFIG_SYS_SCSI_MAX_SCSI_ID` [7]、および `CONFIG_SYS_SCSI_MAX_DEVICE` [`CONFIG_SYS_SCSI_MAX_SCSI_ID` * `CONFIG_SYS_SCSI_MAX_LUN`] は、LUN、SCSI ID、およびターゲットデバイスの最大数を定義するように調整できます。 環境変数 'scsidevs' は、最後のスキャン中に見つかった SCSI デバイスの数に設定されます。

-   ネットワークサポート (PCI):
    `CONFIG_E1000`
    Intel 8254x/8257x ギガビットチップのサポート。

    `CONFIG_E1000_SPI`
    Intel 8257x 上の SPI バスへの直接アクセス用ユーティリティコード。`CONFIG_CMD_E1000` または `CONFIG_E1000_SPI_GENERIC` の少なくとも1つを設定しない限り、これは役に立ちません。

    `CONFIG_E1000_SPI_GENERIC`
    Intel 8257x 上の SPI バスへの汎用アクセスを許可します。例えば "sspi" コマンドを使用します。

    `CONFIG_NATSEMI`
    National dp83815 チップのサポート。

    `CONFIG_NS8382X`
    National dp8382[01] ギガビットチップのサポート。

-   ネットワークサポート (その他):

    `CONFIG_DRIVER_AT91EMAC`
    AT91RM9200 EMAC のサポート。

    	`CONFIG_RMII`
    		Reduced MII インターフェースを使用するにはこれを定義します

    		`CONFIG_DRIVER_AT91EMAC_QUIET`
    		これが定義されている場合、ドライバは静かになります。ドライバはリンクステータスメッセージを表示しません。

    `CONFIG_CALXEDA_XGMAC`
    Calxeda XGMAC デバイスのサポート

    `CONFIG_LAN91C96`
    SMSC の LAN91C96 チップのサポート。

    	`CONFIG_LAN91C96_USE_32_BIT`
    		32 ビットアドレス指定を有効にするにはこれを定義します

    `CONFIG_SMC91111`
    SMSC の LAN91C111 チップのサポート

    		`CONFIG_SMC91111_BASE`
    		デバイスの物理アドレス（I/O 空間）を保持するためにこれを定義します

    		`CONFIG_SMC_USE_32_BIT`
    		データバスが 32 ビットの場合にこれを定義します

    		`CONFIG_SMC_USE_IOFUNCS`
    		マクロの代わりに I/O 関数を使用するにはこれを定義します（一部のハードウェアはマクロでは動作しません）

    		`CONFIG_SYS_DAVINCI_EMAC_PHY_COUNT`
    		PHY が3つ以上ある場合にこれを定義します。

    `CONFIG_FTGMAC100`
    Faraday の FTGMAC100 ギガビット SoC イーサネットのサポート

    		`CONFIG_FTGMAC100_EGIGA`
    		ギガビット PHY で GE リンク更新を使用するにはこれを定義します。FTGMAC100 がギガビット PHY に接続されている場合にこれを定義します。 システムに 10/100 PHY しかない場合、誤った動作が発生しない可能性があります。 なぜなら、PHY は通常、ギガビットステータスおよびギガビット制御レジスタをポーリングするときにタイムアウトまたは無用なデータを返すからです。 この動作は、10/100 リンク速度更新の正確さには影響しません。

    `CONFIG_SH_ETHER`
    ルネサス オンチップ イーサネットコントローラのサポート

    		`CONFIG_SH_ETHER_USE_PORT`
    		使用するポート数を定義します

    		`CONFIG_SH_ETHER_PHY_ADDR`
    		ETH PHY のアドレスを定義します

    		`CONFIG_SH_ETHER_CACHE_WRITEBACK`
    		このオプションが設定されている場合、ドライバはキャッシュフラッシュを有効にします。

-   TPM サポート:
    `CONFIG_TPM`
    TPM デバイスをサポートします。

    `CONFIG_TPM_TIS_INFINEON`
    Infineon i2c バス TPM デバイスのサポート。現時点ではシステムごとに1つのデバイスのみがサポートされます。

    		`CONFIG_TPM_TIS_I2C_BURST_LIMITATION`
    		バーストカウントバイトの上限を定義します

    `CONFIG_TPM_ST33ZP24`
    STMicroelectronics TPM デバイスのサポート。DM_TPM サポートが必要です。

    		`CONFIG_TPM_ST33ZP24_I2C`
    		STMicroelectronics ST33ZP24 I2C デバイスのサポート。TPM_ST33ZP24 と I2C が必要です。

    		`CONFIG_TPM_ST33ZP24_SPI`
    		STMicroelectronics ST33ZP24 SPI デバイスのサポート。TPM_ST33ZP24 と SPI が必要です。

    `CONFIG_TPM_ATMEL_TWI`
    Atmel TWI TPM デバイスのサポート。I2C サポートが必要です。

    `CONFIG_TPM_TIS_LPC`
    汎用パラレルポート TPM デバイスのサポート。現時点ではシステムごとに1つのデバイスのみがサポートされます。

    		`CONFIG_TPM_TIS_BASE_ADDRESS`
    		汎用 TPM デバイスがマップされるベースアドレス。現代の x86 システムでは通常 0xfed40000 にマップされます。

    `CONFIG_TPM`
    一部の TPM コマンドへの機能インターフェースを提供する TPM サポートライブラリを有効にするにはこれを定義します。TPM デバイスのサポートが必要です。

    `CONFIG_TPM_AUTH_SESSIONS`
    TPM ライブラリで認証済み機能を有効にするにはこれを定義します。`CONFIG_TPM` と `CONFIG_SHA1` が必要です。

-   USB サポート:
    現時点では UHCI ホストコントローラのみがサポートされています（PIP405、MIP405）。有効にするには `CONFIG_USB_UHCI` を定義します。 USB キーボードを有効にするには `CONFIG_USB_KEYBOARD` を定義し、USB ストレージデバイスを有効にするには `CONFIG_USB_STORAGE` を定義します。 注意: サポートされているのは USB キーボードと USB フロッピードライブ（TEAC FD-05PUB）です。

    `CONFIG_USB_EHCI_TXFIFO_THRESH` は、リセット時に EHCI コントローラの txfilltuning フィールドの設定を有効にします。

    `CONFIG_USB_DWC2_REG_ADDR` は DWC2 HW モジュールレジスタの物理 CPU アドレスです。

-   USB デバイス:
    USB コンソールを使用したい場合は以下を定義します。 ファームウェアがシリアルコンソールから再構築されたら、コマンド "setenv stdin usbtty; setenv stdout usbtty" を発行し、USB ケーブルを接続します。 Unix コマンド "dmesg" は、新しいデバイスが見つかったことを表示するはずです。 環境変数 `usbtty` を `gserial` または `cdc_acm` に設定すると、デバイスが Linux `gserial` デバイスまたは Common Device Class Abstract Control Model シリアルデバイスとして USB ホストに表示されるようになります。 `usbtty = gserial` を選択した場合、Linux ホストを列挙できるはずです
    	`# modprobe usbserial vendor=0xVendorID product=0xProductID`
    それ以外の場合、`cdc_acm` を使用する場合は、環境変数 `usbtty` を `cdc_acm` に設定するだけで十分なはずです。 以下は YourBoardName.h で定義される可能性があります

    		`CONFIG_USB_DEVICE`
    		UDC デバイスをビルドするにはこれを定義します

    		`CONFIG_USB_TTY`
    		UDC デバイスと通信するために利用可能な tty タイプのデバイスを持つにはこれを定義します

    		`CONFIG_USBD_HS`
    		USB デバイスと usbtty の高速サポートを有効にするにはこれを定義します。この機能が有効になっている場合、列挙が高速またはフルスピードで成功したかどうかを動的にポーリングするために、ドライバによってルーチン `int is_usbd_high_speed(void)` も定義する必要があります。

    		`CONFIG_SYS_CONSOLE_IS_IN_ENV`
    		stdin、stdout、stderr を usbtty に設定したい場合にこれを定義します。

    USB-IF 割り当ての VendorID がある場合は、BoardName.h または直接 usbd_vendor_info.h で独自のベンダー固有値を定義することをお勧めします。 `CONFIG_USBD_MANUFACTURER`、`CONFIG_USBD_PRODUCT_NAME`、`CONFIG_USBD_VENDORID`、`CONFIG_USBD_PRODUCTID` を定義しない場合、U-Boot はターゲットホストに対して Linux デバイスであるかのように振る舞うはずです。

    		`CONFIG_USBD_MANUFACTURER`
    		これをあなたの会社の名前として文字列で定義します
    		- `CONFIG_USBD_MANUFACTURER "my company"`

    		`CONFIG_USBD_PRODUCT_NAME`
    		これをあなたの製品名として文字列で定義します
    		- `CONFIG_USBD_PRODUCT_NAME "acme usb device"`

    		`CONFIG_USBD_VENDORID`
    		これを USB Implementors Forum から割り当てられた Vendor ID として定義します。USB 名前空間を汚染しないように、これは**本物の** Vendor ID でなければなりません。
    		- `CONFIG_USBD_VENDORID 0xFFFF`

    		`CONFIG_USBD_PRODUCTID`
    		これをあなたのデバイスのユニークな Product ID として定義します
    		- `CONFIG_USBD_PRODUCTID 0xFFFF`

-   ULPI レイヤーサポート:
    ULPI (UTMI Low Pin (count) Interface) PHY は、汎用 ULPI レイヤーを介してサポートされます。 汎用レイヤーはプラットフォームビューポートを介して ULPI PHY にアクセスするため、汎用レイヤーとビューポートの両方を有効にする必要があります。 現在、Chipidea/ARC ベースのビューポートのみがサポートされています。 ULPI レイヤーサポートを有効にするには、ボード設定ファイルで `CONFIG_USB_ULPI` と `CONFIG_USB_ULPI_VIEWPORT` を定義します。 ULPI PHY が標準の 24 MHz とは異なる参照クロックを必要とする場合は、`CONFIG_ULPI_REF_CLK` を適切な値 (Hz) に定義する必要があります。

-   MMC サポート:
    Intel PXA の MMC コントローラがサポートされています。これを有効にするには `CONFIG_MMC` を定義します。 MMC は、フラッシュと同様にデバイスを物理メモリにマッピングすることで、ブートプロンプトからアクセスできます。 コマンドラインは `CONFIG_CMD_MMC` で有効になります。MMC ドライバは FAT fs でも動作します。これは `CONFIG_CMD_FAT` で有効になります。

    `CONFIG_SH_MMCIF`
    ルネサス オンチップ MMCIF コントローラのサポート

    		`CONFIG_SH_MMCIF_ADDR`
    		MMCIF レジスタのベースアドレスを定義します

    		`CONFIG_SH_MMCIF_CLK`
    		MMCIF のクロック周波数を定義します

-   USB Device Firmware Update (DFU) クラスサポート:
    `CONFIG_DFU_OVER_USB`
    DFU USB クラスの USB 部分を有効にします

    `CONFIG_DFU_NAND`
    DFU 経由で NAND デバイスを公開するサポートを有効にします。

    `CONFIG_DFU_RAM`
    DFU 経由で RAM を公開するサポートを有効にします。注意: DFU 仕様は不揮発性メモリの使用を参照していますが、仕様の範囲を超える使用法（ここでは RAM の使用、主に開発者の助けになるもの）を許可しています。

    `CONFIG_SYS_DFU_DATA_BUF_SIZE`
    Dfu 転送は、生のストレージデバイスにデータを書き込む前にバッファを使用します。このバッファのサイズ（バイト単位）を設定可能にします。 このバッファのサイズは、"dfu_bufsiz" 環境変数を介しても設定可能です。

    `CONFIG_SYS_DFU_MAX_FILE_SIZE`
    生のストレージデバイスではなくファイルを更新する場合、ファイルをコピーするための静的バッファを使用し、ファイル全体が与えられたらバッファを書き込みます。これをバッファの最大ファイルサイズ（バイト単位）に定義します。 未定義の場合、デフォルトは 4 MiB です。

    `DFU_DEFAULT_POLL_TIMEOUT`
    ポーリングタイムアウト [ms] は、デバイスがホストに送信できるタイムアウトです。 ホストは、デバイスに後続の DFU_GET_STATUS 要求を送信する前に、このタイムアウトを待機する必要があります。

    `DFU_MANIFEST_POLL_TIMEOUT`
    ポーリングタイムアウト [ms] は、デバイスが dfuMANIFEST 状態に入るときにホストに送信します。 ホストはこのタイムアウトを待機してから、デバイスに再度 USB 要求を送信します。

-   ジャーナリングフラッシュファイルシステムサポート:
    `CONFIG_JFFS2_NAND`
    NAND デバイス上のデフォルトパーティション用にこれらを定義します

    `CONFIG_SYS_JFFS2_FIRST_SECTOR`,
    `CONFIG_SYS_JFFS2_FIRST_BANK`, `CONFIG_SYS_JFFS2_NUM_BANKS`
    NOR デバイス上のデフォルトパーティション用にこれらを定義します

-   キーボードサポート:
    利用可能なキーボードドライバについては Kconfig ヘルプを参照してください。

    `CONFIG_KEYBOARD`

    カスタムキーボードサポートを有効にするにはこれを定義します。これは単に `drv_keyboard_init()` を呼び出します。これはボード固有ファイルで定義する必要があります。 このオプションは非推奨であり、novena でのみ使用されます。新しいボードでは、代わりにドライバモデルを使用してください。

-   ビデオサポート:
    `CONFIG_FSL_DIU_FB`
    Freescale DIU ビデオドライバを有効にします。DIU を持つ SOC のリファレンスボードは、DIU サポートを有効にするためにこのマクロを定義し、以下の他のマクロも定義する必要があります。

    		`CONFIG_SYS_DIU_ADDR`
    		`CONFIG_VIDEO`
    		`CONFIG_CFB_CONSOLE`
    		`CONFIG_VIDEO_SW_CURSOR`
    		`CONFIG_VGA_AS_SINGLE_DEVICE`
    		`CONFIG_VIDEO_LOGO`
    		`CONFIG_VIDEO_BMP_LOGO`

    DIU ドライバは 'video-mode' 環境変数を探し、定義されている場合、ブート中に DIU をコンソールとして有効にします。 この変数の説明については、ドキュメントファイル doc/README.video を参照してください。

-   LCD サポート: `CONFIG_LCD`

    LCD サポートを有効にするにはこれを定義します（LCD ディスプレイへの出力用）。 また、以下のいずれかを定義して、サポートされているディスプレイの1つを選択します。

    `CONFIG_ATMEL_LCD`:

    	HITACHI TX09D70VM1CCA, 3.5", 240x320.

    `CONFIG_NEC_NL6448AC33`:

    	NEC NL6448AC33-18. アクティブ、カラー、シングルスキャン。

    `CONFIG_NEC_NL6448BC20`

    	NEC NL6448BC20-08. 6.5", 640x480. アクティブ、カラー、シングルスキャン。

    `CONFIG_NEC_NL6448BC33_54`

    	NEC NL6448BC33-54. 10.4", 640x480. アクティブ、カラー、シングルスキャン。

    `CONFIG_SHARP_16x9`

    	Sharp 320x240. アクティブ、カラー、シングルスキャン。16x9 ではなく、何であるかは不明です。

    `CONFIG_SHARP_LQ64D341`

    	Sharp LQ64D341 ディスプレイ, 640x480. アクティブ、カラー、シングルスキャン。

    `CONFIG_HLD1045`

    	HLD1045 ディスプレイ, 640x480. アクティブ、カラー、シングルスキャン。

    `CONFIG_OPTREX_BW`

    	Optrex	 CBL50840-2 NF-FW 99 22 M5
    	または
    	Hitachi	 LMG6912RPFC-00T
    	または
    	Hitachi	 SP14Q002

    	320x240. 白黒。

    `CONFIG_LCD_ALIGNMENT`

    通常、LCD はページアラインされます（通常 4KB）。これが定義されている場合、LCD は代わりにこの値にアラインされます。ARM の場合、セクションごとにデータキャッシュ設定を変更する方が安価であるため、ここで `MMU_SECTION_SIZE` を使用すると便利な場合があります。

    `CONFIG_LCD_ROTATION`

    例えばディスプレイが縦向きモードで取り付けられている場合や、横向きでも180度回転して取り付けられている場合など、表示内容をフレームバッファに対して回転させる必要がある場合があります。これにより、ユーザーは表示されるメッセージを読むことができます。 `CONFIG_LCD_ROTATION` が定義されると、lcd_console はボード固有のコードによって提供される "vidinfo_t" の "vl_rot" から与えられた回転で初期化されます。 vl_rot の値は以下のようにコード化されます（fbcon=rotate:<n> linux-kernel コマンドラインに一致）:
    0 = 回転なし、それぞれ 0 度
    1 = 90 度回転
    2 = 180 度回転
    3 = 270 度回転

    `CONFIG_LCD_ROTATION` が定義されていない場合、コンソールは 0 度回転で初期化されます。

    `CONFIG_LCD_BMP_RLE8`

    LCD 上での RLE8 圧縮ビットマップの描画をサポートします。

    `CONFIG_I2C_EDID`

    接続された LCD ディスプレイから I2C 経由で EDID 情報を読み取ることができる 'i2c edid' コマンドを有効にします。

-   MII/PHY サポート:
    `CONFIG_PHY_CLOCK_FREQ` (ppc4xx)

    MII バスのクロック周波数

    `CONFIG_PHY_RESET_DELAY`

    Intel LXT971A のような一部の PHY は、リセット後、MII レジスタアクセスが可能になる前に余分な遅延が必要です。このような PHY の場合、このオプションを必要な遅延時間 (usec) に設定します。 (LXT971A の場合は最低 300usec)

    `CONFIG_PHY_CMD_DELAY` (ppc4xx)

    Intel LXT971A のような一部の PHY は、コマンド発行後、MII ステータスレジスタが読み取れるようになる前に余分な遅延が必要です

-   IP アドレス:
    `CONFIG_IPADDR`

    デフォルトのイーサネットインターフェースに使用する IP アドレスのデフォルト値を定義します。これは、例えば bootp によって決定されない場合に備えます。(環境変数 "ipaddr")

-   サーバー IP アドレス:
    `CONFIG_SERVERIP`

    "tftboot" コマンドを使用するときに接続する TFTP サーバーの IP アドレスのデフォルト値を定義します。(環境変数 "serverip")

    `CONFIG_KEEP_SERVERADDR`

    サーバーの MAC アドレスを env 'serveraddr' に保持し、bootargs に渡します (Linux の netconsole オプションなど)。

-   ゲートウェイ IP アドレス:
    `CONFIG_GATEWAYIP`

    他のネットワークへのパケットが送信されるデフォルトルーターの IP アドレスのデフォルト値を定義します。(環境変数 "gatewayip")

-   サブネットマスク:
    `CONFIG_NETMASK`

    IP アドレスがローカルサブネットに属するか、ルーター経由で転送する必要があるかを判断するために使用されるサブネットマスク（またはルーティングプレフィックス）のデフォルト値を定義します。(環境変数 "netmask")

-   BOOTP リカバリモード:
    `CONFIG_BOOTP_RANDOM_DELAY`

    ネットワーク内に BOOTP を使用してブートしようとするターゲットが多数ある場合、すべてのシステムがまったく同じ瞬間に BOOTP 要求を送信するのを避けたい場合があります（例えば、停電からの復旧時に、すべてのシステムがブートしようとし、BOOTP サーバーをフラッディングする場合など）。`CONFIG_BOOTP_RANDOM_DELAY` を定義すると、BOOTP 要求を送信する前にランダムな遅延が挿入されます。その場合、以下の遅延が挿入されます。

    	1 番目の BOOTP 要求:	遅延 0 ... 1 秒
    	2 番目の BOOTP 要求:	遅延 0 ... 2 秒
    	3 番目の BOOTP 要求:	遅延 0 ... 4 秒
    	4 番目以降の
    	BOOTP 要求:		遅延 0 ... 8 秒

    `CONFIG_BOOTP_ID_CACHE_SIZE`

    BOOTP パケットは 32 ビット ID を使用して一意に識別されます。 サーバーはクライアント要求から応答へ ID をコピーし、U-Boot はこれを使用して受信応答の宛先が自分であるかどうかを判断します。一部のサーバーは、アドレスを割り当てる前に使用中でないことを確認します（通常は ARP ping を使用）。そのため、応答に数百ミリ秒かかる場合があります。 ネットワークの輻輳も、応答がクライアントに戻るまでの時間に影響を与える可能性があります。 その時間が長すぎると、U-Boot は要求を再送信します。 これらの再送信後にも以前の応答を受け入れられるようにするため、U-Boot の BOOTP クライアントは ID の小さなキャッシュを保持します。 `CONFIG_BOOTP_ID_CACHE_SIZE` は、このキャッシュのサイズを制御します。 デフォルトでは、最大4つの未解決の要求の ID を保持します。 これを増やすと、U-Boot は異常に高いレイテンシを持つネットワークで BOOTP クライアントからのオファーを受け入れることができます。

-   DHCP 詳細オプション:
    `CONFIG_BOOTP_*` シンボルを定義することで、DHCP 機能を微調整できます。

    	`CONFIG_BOOTP_NISDOMAIN`
    	`CONFIG_BOOTP_BOOTFILESIZE`
    	`CONFIG_BOOTP_NTPSERVER`
    	`CONFIG_BOOTP_TIMEOFFSET`
    	`CONFIG_BOOTP_VENDOREX`
    	`CONFIG_BOOTP_MAY_FAIL`

    `CONFIG_BOOTP_SERVERIP` - TFTP サーバーは BOOTP サーバーではなく、serverip 環境変数になります。

    `CONFIG_BOOTP_MAY_FAIL` - 設定されたリトライ回数の後に DHCP サーバーが見つからない場合、最初からやり直す代わりに呼び出しは失敗します。これは、DHCP サーバーが利用できない場合にリンクローカル IP アドレス設定にフェイルオーバーするために使用できます。

    `CONFIG_BOOTP_DHCP_REQUEST_DELAY`

    「DHCP オファー」を受信してから「DHCP 要求」を送信するまでの遅延のための 32 ビット値（マイクロ秒単位）。 これは、「DHCP 要求」に 100% 応答しない特定の DHCP サーバーの問題を修正します。例えば、180MHz で動作する AT91RM9200 プロセッサでは、Windows Server 2003 DHCP サーバーが 100% 応答するようになるまでに、この遅延が*少なくとも* 15,000 usec 必要でした。 安全のため、少なくとも 50,000 usec を推奨します。 代替策は、リトライの1つが成功することを期待することですが、DHCP タイムアウトとリトライプロセスはこの遅延よりも長くかかることに注意してください。

-   リンクローカル IP アドレスネゴシエーション:
    ローカルネットワーク上の他のリンクローカルクライアントと、明示的な設定を必要としないアドレスについてネゴシエートします。 これは、デバイスが動作する必要のあるすべての環境で DHCP サーバーが存在することが保証できない場合に特に役立ちます。 詳細については doc/README.link-local を参照してください。

-   環境変数からの MAC アドレス

    `FDT_SEQ_MACADDR_FROM_ENV`

    環境変数から順次取得した MAC アドレスでデバイスツリーを修正します。この設定は、デバイスツリーの使用不可能なイーサネットノードが存在しないか、またはそれらのステータスが "disabled" とマークされているという仮定で機能します。

-   CDP オプション:
    `CONFIG_CDP_DEVICE_ID`

    CDP トリガーフレームで使用されるデバイス ID。

    `CONFIG_CDP_DEVICE_ID_PREFIX`

    デバイスの MAC アドレスの前に付けられる2文字の文字列。

    `CONFIG_CDP_PORT_ID`

    ポートの ASCII 名を含む printf フォーマット文字列。通常は "eth%d" に設定され、最初のイーサネットには eth0、2番目には eth1 などが設定されます。

    `CONFIG_CDP_CAPABILITIES`

    デバイスの能力を示す 32 ビット整数。転送しない通常のホストの場合は 0x00000010。

    `CONFIG_CDP_VERSION`

    ソフトウェアのバージョンを含む ASCII 文字列。

    `CONFIG_CDP_PLATFORM`

    プラットフォームの名前を含む ASCII 文字列。

    `CONFIG_CDP_TRIGGER`

    トリガーで送信される 32 ビット整数。

    `CONFIG_CDP_POWER_CONSUMPTION`

    デバイスの消費電力をミリワットの 0.1 単位で含む 16 ビット整数。

    `CONFIG_CDP_APPLIANCE_VLAN_TYPE`

    VLAN の ID を含むバイト。

-   ステータス LED: `CONFIG_LED_STATUS`

    いくつかの設定では、LED を使用して現在のステータスを表示できます。例えば、U-Boot コードを実行中は LED が速く点滅し、BOOTP 要求への応答を受信するとすぐに点滅を停止し、Linux カーネルが実行されるとゆっくり点滅し始めます（Linux カーネルのステータス LED ドライバによってサポートされています）。 `CONFIG_LED_STATUS` を定義すると、U-Boot でこの機能が有効になります。

    追加オプション:

    `CONFIG_LED_STATUS_GPIO`
    ステータス LED は GPIO ピンに接続できます。このような場合、gpio_led ドライバをステータス LED バックエンド実装として使用できます。 gpio_led ドライバを U-Boot バイナリに含めるには `CONFIG_LED_STATUS_GPIO` を定義します。

    `CONFIG_GPIO_LED_INVERTED_TABLE`
    一部の GPIO 接続 LED は極性が反転している場合があり、その場合 GPIO high 値は LED オフ状態に対応し、GPIO low 値は LED オン状態に対応します。このような場合、`CONFIG_GPIO_LED_INVERTED_TABLE` を、極性が反転している GPIO LED のリストで定義できます。

-   I2C サポート: `CONFIG_SYS_I2C_LEGACY`

    注意: これはドライバモデルを優先して非推奨です。代わりに `CONFIG_DM_I2C` を使用してください。
    これはレガシー i2c サブシステムを有効にし、u-boot コマンドラインで i2c コマンドを使用できるようになります（速度とスレーブアドレスを定義するために `CONFIG_SYS_I2C_SOFT_SPEED` と `CONFIG_SYS_I2C_SOFT_SLAVE` を設定している限り
      - 2番目のバスを `I2C_SOFT_DECLARATIONS2` で有効にし、`CONFIG_SYS_I2C_SOFT_SPEED_2` と `CONFIG_SYS_I2C_SOFT_SLAVE_2` を定義して速度とスレーブアドレスを定義します
      - 3番目のバスを `I2C_SOFT_DECLARATIONS3` で有効にし、`CONFIG_SYS_I2C_SOFT_SPEED_3` と `CONFIG_SYS_I2C_SOFT_SLAVE_3` を定義して速度とスレーブアドレスを定義します
      - 4番目のバスを `I2C_SOFT_DECLARATIONS4` で有効にし、`CONFIG_SYS_I2C_SOFT_SPEED_4` と `CONFIG_SYS_I2C_SOFT_SLAVE_4` を定義して速度とスレーブアドレスを定義します）

    -   drivers/i2c/fsl_i2c.c:
        -   `CONFIG_SYS_I2C_FSL` で i2c ドライバを有効にします
        -   レジスタオフセットを設定するために `CONFIG_SYS_FSL_I2C_OFFSET` を定義します
        -   i2c 速度のために `CONFIG_SYS_FSL_I2C_SPEED` を、最初のバスのスレーブアドレスのために `CONFIG_SYS_FSL_I2C_SLAVE` を定義します。
        -   ボードが2番目の fsl i2c バスをサポートしている場合は、レジスタオフセットのために `CONFIG_SYS_FSL_I2C2_OFFSET` を、速度のために `CONFIG_SYS_FSL_I2C2_SPEED` を、2番目のバスのスレーブアドレスのために `CONFIG_SYS_FSL_I2C2_SLAVE` を定義します。

    -   drivers/i2c/tegra_i2c.c:
        -   `CONFIG_SYS_I2C_TEGRA` でこのドライバを有効にします
        -   このドライバは、固定速度 100000 とスレーブアドレス 0 で 4 つの i2c バスを追加します！

    -   drivers/i2c/ppc4xx_i2c.c
        -   `CONFIG_SYS_I2C_PPC4XX` でこのドライバを有効にします
        -   `CONFIG_SYS_I2C_PPC4XX_CH0` はハードウェアチャネル 0 を有効にします
        -   `CONFIG_SYS_I2C_PPC4XX_CH1` はハードウェアチャネル 1 を有効にします

    -   drivers/i2c/i2c_mxc.c
        -   `CONFIG_SYS_I2C_MXC` でこのドライバを有効にします
        -   `CONFIG_SYS_I2C_MXC_I2C1` でバス 1 を有効にします
        -   `CONFIG_SYS_I2C_MXC_I2C2` でバス 2 を有効にします
        -   `CONFIG_SYS_I2C_MXC_I2C3` でバス 3 を有効にします
        -   `CONFIG_SYS_I2C_MXC_I2C4` でバス 4 を有効にします
        -   `CONFIG_SYS_MXC_I2C1_SPEED` でバス 1 の速度を定義します
        -   `CONFIG_SYS_MXC_I2C1_SLAVE` でバス 1 のスレーブを定義します
        -   `CONFIG_SYS_MXC_I2C2_SPEED` でバス 2 の速度を定義します
        -   `CONFIG_SYS_MXC_I2C2_SLAVE` でバス 2 のスレーブを定義します
        -   `CONFIG_SYS_MXC_I2C3_SPEED` でバス 3 の速度を定義します
        -   `CONFIG_SYS_MXC_I2C3_SLAVE` でバス 3 のスレーブを定義します
        -   `CONFIG_SYS_MXC_I2C4_SPEED` でバス 4 の速度を定義します
        -   `CONFIG_SYS_MXC_I2C4_SLAVE` でバス 4 のスレーブを定義します
        これらの定義が設定されていない場合、デフォルト値は速度が 100000、スレーブが 0 です。

    -   drivers/i2c/rcar_i2c.c:
        -   `CONFIG_SYS_I2C_RCAR` でこのドライバを有効にします
        -   このドライバは 4 つの i2c バスを追加します

    -   drivers/i2c/sh_i2c.c:
        -   `CONFIG_SYS_I2C_SH` でこのドライバを有効にします
        -   このドライバは 2 から 5 つの i2c バスを追加します

        -   `CONFIG_SYS_I2C_SH_BASE0` はレジスタチャネル 0 を設定します
        -   `CONFIG_SYS_I2C_SH_SPEED0` は速度チャネル 0 を設定します
        -   `CONFIG_SYS_I2C_SH_BASE1` はレジスタチャネル 1 を設定します
        -   `CONFIG_SYS_I2C_SH_SPEED1` は速度チャネル 1 を設定します
        -   `CONFIG_SYS_I2C_SH_BASE2` はレジスタチャネル 2 を設定します
        -   `CONFIG_SYS_I2C_SH_SPEED2` は速度チャネル 2 を設定します
        -   `CONFIG_SYS_I2C_SH_BASE3` はレジスタチャネル 3 を設定します
        -   `CONFIG_SYS_I2C_SH_SPEED3` は速度チャネル 3 を設定します
        -   `CONFIG_SYS_I2C_SH_BASE4` はレジスタチャネル 4 を設定します
        -   `CONFIG_SYS_I2C_SH_SPEED4` は速度チャネル 4 を設定します
        -   `CONFIG_SYS_I2C_SH_NUM_CONTROLLERS` は i2c バスの数を設定します

    -   drivers/i2c/omap24xx_i2c.c
        -   `CONFIG_SYS_I2C_OMAP24XX` でこのドライバを有効にします
        -   `CONFIG_SYS_OMAP24_I2C_SPEED` 速度チャネル 0
        -   `CONFIG_SYS_OMAP24_I2C_SLAVE` スレーブアドレスチャネル 0
        -   `CONFIG_SYS_OMAP24_I2C_SPEED1` 速度チャネル 1
        -   `CONFIG_SYS_OMAP24_I2C_SLAVE1` スレーブアドレスチャネル 1
        -   `CONFIG_SYS_OMAP24_I2C_SPEED2` 速度チャネル 2
        -   `CONFIG_SYS_OMAP24_I2C_SLAVE2` スレーブアドレスチャネル 2
        -   `CONFIG_SYS_OMAP24_I2C_SPEED3` 速度チャネル 3
        -   `CONFIG_SYS_OMAP24_I2C_SLAVE3` スレーブアドレスチャネル 3
        -   `CONFIG_SYS_OMAP24_I2C_SPEED4` 速度チャネル 4
        -   `CONFIG_SYS_OMAP24_I2C_SLAVE4` スレーブアドレスチャネル 4

    -   drivers/i2c/s3c24x0_i2c.c:
        -   `CONFIG_SYS_I2C_S3C24X0` でこのドライバを有効にします
        -   このドライバは、固定速度 100000 とスレーブアドレス 0 で i2c バス（Exynos5250、Exynos5420 では 11、Exynos4 では 9、Samsung の S3C24X0 SoC では 1）を追加します！

    -   drivers/i2c/ihs_i2c.c
        -   `CONFIG_SYS_I2C_IHS` でこのドライバを有効にします
        -   `CONFIG_SYS_I2C_IHS_CH0` はハードウェアチャネル 0 を有効にします
        -   `CONFIG_SYS_I2C_IHS_SPEED_0` 速度チャネル 0
        -   `CONFIG_SYS_I2C_IHS_SLAVE_0` スレーブアドレスチャネル 0
        -   `CONFIG_SYS_I2C_IHS_CH1` はハードウェアチャネル 1 を有効にします
        -   `CONFIG_SYS_I2C_IHS_SPEED_1` 速度チャネル 1
        -   `CONFIG_SYS_I2C_IHS_SLAVE_1` スレーブアドレスチャネル 1
        -   `CONFIG_SYS_I2C_IHS_CH2` はハードウェアチャネル 2 を有効にします
        -   `CONFIG_SYS_I2C_IHS_SPEED_2` 速度チャネル 2
        -   `CONFIG_SYS_I2C_IHS_SLAVE_2` スレーブアドレスチャネル 2
        -   `CONFIG_SYS_I2C_IHS_CH3` はハードウェアチャネル 3 を有効にします
        -   `CONFIG_SYS_I2C_IHS_SPEED_3` 速度チャネル 3
        -   `CONFIG_SYS_I2C_IHS_SLAVE_3` スレーブアドレスチャネル 3
        -   `CONFIG_SYS_I2C_IHS_DUAL` でデュアルチャネルを有効にします
        -   `CONFIG_SYS_I2C_IHS_SPEED_0_1` 速度チャネル 0_1
        -   `CONFIG_SYS_I2C_IHS_SLAVE_0_1` スレーブアドレスチャネル 0_1
        -   `CONFIG_SYS_I2C_IHS_SPEED_1_1` 速度チャネル 1_1
        -   `CONFIG_SYS_I2C_IHS_SLAVE_1_1` スレーブアドレスチャネル 1_1
        -   `CONFIG_SYS_I2C_IHS_SPEED_2_1` 速度チャネル 2_1
        -   `CONFIG_SYS_I2C_IHS_SLAVE_2_1` スレーブアドレスチャネル 2_1
        -   `CONFIG_SYS_I2C_IHS_SPEED_3_1` 速度チャネル 3_1
        -   `CONFIG_SYS_I2C_IHS_SLAVE_3_1` スレーブアドレスチャネル 3_1

    追加の定義:

    `CONFIG_SYS_NUM_I2C_BUSES`
    使用したい i2c バスの数を保持します。

    `CONFIG_SYS_I2C_DIRECT_BUS`
    ハードウェア上で i2c マックスを使用しない場合にこれを定義します。 `CONFIG_SYS_I2C_MAX_HOPS` が定義されていないか == 0 の場合、この定義は省略できます。

    `CONFIG_SYS_I2C_MAX_HOPS`
    1つの i2c バス上に最大で連続して接続されているマックスの数を定義します。i2c マックスを使用しない場合は、この定義を省略します。

    `CONFIG_SYS_I2C_BUSES`
    使用したいバスのリストを保持します。`CONFIG_SYS_I2C_DIRECT_BUS` が定義されていない場合にのみ使用されます。例えば、`CONFIG_SYS_I2C_MAX_HOPS = 1` および `CONFIG_SYS_NUM_I2C_BUSES = 9` のボードの場合:

    ```
    CONFIG_SYS_I2C_BUSES	{{0, {I2C_NULL_HOP}}, \
    				{0, {{I2C_MUX_PCA9547, 0x70, 1}}}, \
    				{0, {{I2C_MUX_PCA9547, 0x70, 2}}}, \
    				{0, {{I2C_MUX_PCA9547, 0x70, 3}}}, \
    				{0, {{I2C_MUX_PCA9547, 0x70, 4}}}, \
    				{0, {{I2C_MUX_PCA9547, 0x70, 5}}}, \
    				{1, {I2C_NULL_HOP}}, \
    				{1, {{I2C_MUX_PCA9544, 0x72, 1}}}, \
    				{1, {{I2C_MUX_PCA9544, 0x72, 2}}}, \
    				}
    ```

    これは以下を定義します
    	バス 0 はアダプタ 0 上、マックスなし
    	バス 1 はアダプタ 0 上、アドレス 0x70 ポート 1 に PCA9547 マックスあり
    	バス 2 はアダプタ 0 上、アドレス 0x70 ポート 2 に PCA9547 マックスあり
    	バス 3 はアダプタ 0 上、アドレス 0x70 ポート 3 に PCA9547 マックスあり
    	バス 4 はアダプタ 0 上、アドレス 0x70 ポート 4 に PCA9547 マックスあり
    	バス 5 はアダプタ 0 上、アドレス 0x70 ポート 5 に PCA9547 マックスあり
    	バス 6 はアダプタ 1 上、マックスなし
    	バス 7 はアダプタ 1 上、アドレス 0x72 ポート 1 に PCA9544 マックスあり
    	バス 8 はアダプタ 1 上、アドレス 0x72 ポート 2 に PCA9544 マックスあり

    ボードに i2c マックスがない場合は、この定義を省略します。

- レガシー I2C サポート:
    ソフトウェア i2c インターフェース (CONFIG_SYS_I2C_SOFT) を使用する場合、以下のマクロを定義する必要があります (例は include/configs/lwmon.h から):

    `I2C_INIT`

    (オプション)。I2C コントローラを有効にするか、ポートを設定するために必要なコマンド。
    例: `#define I2C_INIT (immr->im_cpm.cp_pbdir |= PB_SCL)`

    `I2C_ACTIVE`

    I2C データラインをアクティブ (駆動) にするために必要なコード。データラインがオープンドレインの場合、この定義は null にできます。
    例: `#define I2C_ACTIVE (immr->im_cpm.cp_pbdir |= PB_SDA)`

    `I2C_TRISTATE`

    I2C データラインをトライステート (非アクティブ) にするために必要なコード。データラインがオープンドレインの場合、この定義は null にできます。
    例: `#define I2C_TRISTATE (immr->im_cpm.cp_pbdir &= ~PB_SDA)`

    `I2C_READ`

    I2C データラインが High の場合は true、Low の場合は false を返すコード。
    例: `#define I2C_READ ((immr->im_cpm.cp_pbdat & PB_SDA) != 0)`

    `I2C_SDA(bit)`

    `<bit>` が true の場合、I2C データラインを High に設定します。false の場合は、クリアします (Low)。
    例:
    ```c
    #define I2C_SDA(bit) \
        if(bit) immr->im_cpm.cp_pbdat |=  PB_SDA; \
        else    immr->im_cpm.cp_pbdat &= ~PB_SDA
    ```


    `I2C_SCL(bit)`

    `<bit>` が true の場合、I2C クロックラインを High に設定します。false の場合は、クリアします (Low)。
    例:
    ```c
    #define I2C_SCL(bit) \
        if(bit) immr->im_cpm.cp_pbdat |=  PB_SCL; \
        else    immr->im_cpm.cp_pbdat &= ~PB_SCL
    ```


    `I2C_DELAY`

    この遅延はクロックサイクルごとに4回呼び出されるため、データ転送レートを制御します。したがって、データレートは 1 / (I2C_DELAY * 4) となります。しばしば次のように定義されます:

    ```c
    #define I2C_DELAY  udelay(2)
    ```


    `CONFIG_SOFT_I2C_GPIO_SCL / CONFIG_SOFT_I2C_GPIO_SDA`

    アーキテクチャが汎用 GPIO フレームワーク (asm/gpio.h) をサポートしている場合、代わりに SCL / SDA として使用される 2 つの GPIO を定義できます。前述の I2C_xxx マクロのいずれも、必要に応じて GPIO ベースのデフォルト値が割り当てられます。これらは、汎用 GPIO 関数に直接与えられる GPIO 値として定義する必要があります。

    `CONFIG_SYS_I2C_INIT_BOARD`

    i2c バス転送中にボードがリセットされた場合、チップは現在の転送がまだ進行中であると考える可能性があります。一部のボードでは、プロセッサピンを GPIO として使用するか、バスに接続された 2 番目のピンを持つことによって、i2c SCLK ラインに直接アクセスすることが可能です。このオプションが定義されている場合、boards/xxx/board.c 内のカスタム `i2c_init_board()` ルーチンがブートシーケンスの早い段階で実行されます。

    `CONFIG_I2C_MULTI_BUS`

    このオプションは、それぞれがコントローラを持つ必要がある複数の I2C バスの使用を許可します。任意の時点でアクティブなバスは 1 つだけです。異なるバスに切り替えるには、'i2c dev' コマンドを使用します。バス番号は 0 ベースであることに注意してください。

    `CONFIG_SYS_I2C_NOPROBES`

    このオプションは、'i2c probe' コマンドが発行されたときにスキップされる I2C デバイスのリストを指定します。`CONFIG_I2C_MULTI_BUS` が設定されている場合は、バス-デバイスペアのリストを指定します。それ以外の場合は、デバイスアドレスの 1 次元配列を指定します。

    例:
    ```c
    #undef  CONFIG_I2C_MULTI_BUS
    #define CONFIG_SYS_I2C_NOPROBES {0x50,0x68}
    ```
    これは、1つのI2Cバスを持つボードでアドレス 0x50 と 0x68 をスキップします。

    ```c
    #define CONFIG_I2C_MULTI_BUS
    #define CONFIG_SYS_I2C_NOPROBES {{0,0x50},{0,0x68},{1,0x54}}
    ```
    これは、バス 0 でアドレス 0x50 と 0x68 を、バス 1 でアドレス 0x54 をスキップします。

    `CONFIG_SYS_SPD_BUS_NUM`

    定義されている場合、これは DDR SPD 用の I2C バス番号を示します。定義されていない場合、U-Boot は SPD が I2C バス 0 にあると想定します。

    `CONFIG_SYS_RTC_BUS_NUM`

    定義されている場合、これは RTC 用の I2C バス番号を示します。定義されていない場合、U-Boot は RTC が I2C バス 0 にあると想定します。

    `CONFIG_SOFT_I2C_READ_REPEATED_START`

    これを定義すると、soft_i2c ドライバの `i2c_read()` 関数は、アドレスポインタの書き込みとデータの読み取りの間に I2C リピーテッドスタートを実行するように強制されます。この定義が省略された場合、ストップスタートシーケンスを実行するデフォルトの動作が使用されます。ほとんどの I2C デバイスはどちらの方法でも使用できますが、一部のデバイスはどちらか一方を必要とします。

- SPI サポート: `CONFIG_SPI`

    SPI ドライバを有効にします (これまでのところ SPI EEPROM でのみテスト済み、SACSng ボード上の Crystal A/D および D/A でもインスタンスが動作します)。

    `CONFIG_SOFT_SPI`

    ハードウェアサポートを使用する代わりに、ソフトウェア (ビットバン) SPI ドライバを有効にします。これは、機能するために 3 つの汎用 I/O ポートピン (2 つの出力、1 つの入力) のみを必要とする汎用ドライバです。これが定義されている場合、ボード設定はいくつかの SPI 設定項目 (使用するポートピンなど) を定義する必要があります。例については、include/configs/sacsng.h を参照してください。

    `CONFIG_SYS_SPI_MXC_WAIT`
    SPI 転送が完了するまでの待機タイムアウト。
    デフォルト: (CONFIG_SYS_HZ/100) /* 10 ms */

- FPGA サポート: `CONFIG_FPGA`

    FPGA サブシステムを有効にします。

    `CONFIG_FPGA_<vendor>`

    特定のチップベンダーのサポートを有効にします。
    (ALTERA, XILINX)

    `CONFIG_FPGA_<family>`

    FPGA ファミリのサポートを有効にします。
    (SPARTAN2, SPARTAN3, VIRTEX2, CYCLONE2, ACEX1K, ACEX)

    `CONFIG_FPGA_COUNT`

    サポートする FPGA デバイスの数を指定します。

    `CONFIG_SYS_FPGA_PROG_FEEDBACK`

    FPGA 設定中にハッシュマークの出力を有効にします。

    `CONFIG_SYS_FPGA_CHECK_BUSY`

    設定関数による FPGA 設定インターフェースのビジー状態のチェックを有効にします。このオプションでは、ボードまたはデバイス固有の関数を作成する必要があります。

    `CONFIG_FPGA_DELAY`

    定義されている場合、FPGA 設定ドライバで遅延を提供する関数。

    `CONFIG_SYS_FPGA_CHECK_CTRLC`
    Control-C で FPGA 設定を中断できるようにします。

    `CONFIG_SYS_FPGA_CHECK_ERROR`

    FPGA ビットファイルロード中の設定エラーをチェックします。例えば、Virtex II 設定中に INIT_B ラインが Low になった場合 (CRC エラーを示す) に中断します。

    `CONFIG_SYS_FPGA_WAIT_INIT`

    Virtex II FPGA 設定シーケンス中に PROB_B がデアサートされた後、INIT_B ラインがデアサートされるのを待つ最大時間。デフォルト時間は 500 ms です。

    `CONFIG_SYS_FPGA_WAIT_BUSY`

    Virtex II FPGA 設定中に BUSY がデアサートされるのを待つ最大時間。デフォルトは 5 ms です。

    `CONFIG_SYS_FPGA_WAIT_CONFIG`

    FPGA 設定後に待機する時間。デフォルトは 200 ms です。

- 設定管理:

    `CONFIG_IDENT_STRING`

    定義されている場合、この文字列は U-Boot バージョン情報 (U_BOOT_VERSION) に追加されます。

- ベンダーパラメータ保護:

    U-Boot は、環境変数 "serial#" (ボードシリアル番号) と "ethaddr" (イーサネットアドレス) の値を、ボードベンダー / 製造元によって一度設定されるパラメータと見なし、ユーザーによる安易な変更からこれらの変数を保護します。一度設定されると、これらの変数は読み取り専用になり、書き込みまたは削除の試みは拒否されます。
    この動作は変更できます:

    設定ファイルで `CONFIG_ENV_OVERWRITE` が #define されている場合、ベンダーパラメータの書き込み保護は完全に無効になります。誰でもこれらのパラメータを変更または削除できます。

    あるいは、デフォルト環境で `ethaddr` と `CONFIG_OVERWRITE_ETHADDR_ONCE` の両方を定義すると、デフォルトのイーサネットアドレスが環境にインストールされ、ユーザーはそれを **一度だけ** 変更できます。[serial# はこれによる影響を受けません、つまり読み取り専用のままです。]

    同じことは、".flags" 変数または `CONFIG_ENV_FLAGS_LIST_STATIC` を定義することで、これらの変数へのアクセスタイプを設定することにより、任意の変数に対してより柔軟な方法で実現できます。

- 保護 RAM:
    `CONFIG_PRAM`

    "保護 RAM"、つまり U-Boot によって上書きされない RAM の予約を有効にするには、この変数を定義します。pRAM 用に予約したい kB 数を保持するために `CONFIG_PRAM` を定義します。環境変数 "pram" を予約したい kB 数に定義することで、このデフォルト値を上書きできます。ボード情報構造体は依然として RAM の全量を表示することに注意してください。pRAM が予約されている場合、新しい環境変数 "mem" が自動的に定義され、Linux へのブート引数として渡すことができる形式で残りの RAM の量を保持します。例えば、次のようにします:

        setenv bootargs ... mem=\${mem}
        saveenv

    このようにして、Linux にもこのメモリを使用しないように指示でき、結果として再起動の影響を受けないメモリ領域ができます。

    *警告* ボード設定が RAM サイズの自動検出を使用する場合、このメモリテストが非破壊的であることを確認する必要があります。これまでのところ、以下のボード設定が "pRAM-clean" であることがわかっています:

        IVMS8, IVML24, SPD8xx,
        HERMES, IP860, RPXlite, LWMON,
        FLAGADM

- 物理メモリ領域 (> 4GB) へのアクセス
    U-Boot が通常アクセスできないメモリ上の操作に対する基本的なサポートが提供されています - 例えば、一部のアーキテクチャでは、物理アドレス拡張などを使用して 32 ビットマシンで 4GB を超えるメモリへのアクセスをサポートしています。この基本的なサポートにアクセスするには `CONFIG_PHYSMEM` を定義します。これは現在、メモリのクリアのみをサポートしています。

- エラー回復:
    `CONFIG_NET_RETRY_COUNT`

    この変数は、ARP、RARP、TFTP、または BOOTP などのネットワーク操作で、操作を断念するまでのリトライ回数を定義します。定義されていない場合、デフォルト値の 5 が使用されます。

    `CONFIG_ARP_TIMEOUT`

    ARP 応答を待つタイムアウト時間 (ミリ秒)。

    `CONFIG_NFS_TIMEOUT`

    NFS プロトコルで使用されるタイムアウト時間 (ミリ秒)。
    nfs コマンドで "ERROR: Cannot umount" が発生する場合は、より長いタイムアウト (例: `#define CONFIG_NFS_TIMEOUT 10000UL`) を試してください。

    注:

        現在の実装では、ローカル変数スペースとグローバル環境変数スペースは分離されています。ローカル変数は、単に `name=value` と入力して定義するものです。後でローカル変数にアクセスするには、`$name` または `${name}` と書く必要があります。変数の内容を直接実行するには、コマンドプロンプトで `$name` と入力します。
        グローバル環境変数は、setenv/printenv を使用して操作するものです。そのような変数に格納されたコマンドを実行するには、run コマンドを使用する必要があり、アクセスするために '$' 記号を使用してはなりません。
        コマンドや特殊文字を変数に格納するには、セミコロンや特殊記号の前のバックスラッシュの代わりに、変数のテキスト全体を二重引用符で囲んでください。

- コマンドライン編集と履歴:
    `CONFIG_CMDLINE_PS_SUPPORT`

    実行時にコマンドプロンプト文字列を変更するサポートを有効にします。これまでのところ静的文字列のみがサポートされています。文字列は環境変数 PS1 および PS2 から取得されます。

- デフォルト環境:
    `CONFIG_EXTRA_ENV_SETTINGS`

    ブートイメージにコンパイルされるデフォルト環境の一部となる、任意の数の NULL 終端文字列 (変数 = 値 のペア) を含むようにこれを定義します。例えば、ボードの設定ファイルに次のようなものを配置します:

    ```c
    #define CONFIG_EXTRA_ENV_SETTINGS \
        "myvar1=value1\0" \
        "myvar2=value2\0"
    ```

    警告: この方法は、環境が U-Boot コードによってどのように格納されるかという内部形式に関する知識に基づいています。これは公式にエクスポートされたインターフェースではありません！ この形式がすぐに変更される可能性は低いですが、保証もありません。ここで何をしているのかをよく理解しておく必要があります。

    注: デフォルト環境の過度な (乱)用は推奨されません。"source" コマンドや boot コマンドなど、環境をプリセットする他の方法をまず確認してください。

    `CONFIG_DELAY_ENVIRONMENT`

    通常、環境はボードが初期化されるときにロードされ、U-Boot で利用可能になります。これはそれを抑制し、後で U-Boot コードによって明示的にロードされるまで環境が利用できなくなります。`CONFIG_OF_CONTROL` が有効な場合、これは代わりに `/config/load-environment` の値によって制御されます。

- TFTP 固定 UDP ポート:
    `CONFIG_TFTP_PORT`

    これが定義されている場合、環境変数 `tftpsrcp` が TFTP UDP ソースポート値を提供するために使用されます。`tftpsrcp` が定義されていない場合、通常の擬似乱数ポート番号ジェネレータが使用されます。また、環境変数 `tftpdstp` が TFTP UDP 宛先ポート値を提供するために使用されます。`tftpdstp` が定義されていない場合、通常のポート 69 が使用されます。

    `tftpsrcp` の目的は、TFTP サーバーが事前に設定されたターゲット IP アドレスと UDP ポートを使用して、TFTP 転送をブラインドで開始できるようにすることです。これにより、(Windows XP) ファイアウォールを「突破」し、残りの TFTP 転送を正常に続行できます。より良い解決策はファイアウォールを適切に設定することですが、それが許可されない場合もあります。

    `CONFIG_STANDALONE_LOAD_ADDR`

    このオプションは、スタンドアロンプログラムがロードされるアドレスのボード固有の値を定義し、アーキテクチャ依存のデフォルト設定を上書きします。

- フレームバッファアドレス:
    `CONFIG_FB_ADDR`

    フレームバッファに特定のアドレスを使用したい場合は、`CONFIG_FB_ADDR` を定義します。これは通常、個別のビデオメモリを持つグラフィックコントローラを使用する場合です。U-Boot は、設定されたパネルサイズに応じてフレームバッファ用のメモリを取得する `lcd_setmem()` を呼び出してシステム RAM 内に動的に予約する代わりに、指定されたアドレスにフレームバッファを配置します。board_init_f 関数を参照してください。

- TFTP サーバー経由の自動ソフトウェア更新
    `CONFIG_UPDATE_TFTP`
    `CONFIG_UPDATE_TFTP_CNT_MAX`
    `CONFIG_UPDATE_TFTP_MSEC_MAX`

    これらのオプションは、自動更新機能を有効にし、制御します。詳細については、doc/README.update を参照してください。

- MTD サポート (mtdparts コマンド, UBI サポート)
    `CONFIG_MTD_UBI_WL_THRESHOLD`
    このパラメータは、UBI デバイスの消去ブロックの最も高い消去カウンタ値と最も低い消去カウンタ値の間の最大差を定義します。このしきい値を超えると、UBI は低い消去カウンタを持つ消去ブロックから高い消去カウンタを持つ消去ブロックへデータを移動することにより、ウェアレベリングを実行し始めます。デフォルト値は、SLC NAND フラッシュ、NOR フラッシュ、および消去ブロックのライフサイクルが 100000 以上のその他のフラッシュに適しているはずです。ただし、通常 10000 未満の消去ブロックライフサイクルを持つ MLC NAND フラッシュの場合、しきい値を小さくする必要があります (例: 128 または 256、ただし 2 のべき乗である必要はありません)。
    デフォルト: 4096

    `CONFIG_MTD_UBI_BEB_LIMIT`
    このオプションは、UBI が MTD デバイス上で期待する最大不良物理消去ブロック数 (1024 消去ブロックあたり) を指定します。基礎となるフラッシュが不良消去ブロックを許容しない場合 (例: NOR フラッシュ)、この値は無視されます。NAND データシートは、フラッシュの耐久寿命に対する最小および最大の NVM (有効ブロック数) をしばしば指定します。1024 消去ブロックあたりの最大期待不良消去ブロック数は、"1024 * (1 - MinNVB / MaxNVB)" として計算でき、これはほとんどの NAND で 20 になります (MaxNVB は基本的にチップ上の消去ブロックの総数です)。言い換えると、この値が 20 の場合、UBI は不良ブロック処理のために物理消去ブロックの約 1.9% を予約しようとします。そして、それは UBI がアタッチする MTD パーティションだけでなく、NAND チップ全体の消去ブロックの 1.9% になります。これは、例えば、最大 40 個の不良消去ブロックを許容する NAND フラッシュチップがあり、それが同じサイズの 2 つの MTD パーティションに分割されている場合、UBI はパーティションをアタッチするときに 40 個の消去ブロックを予約することを意味します。
    デフォルト: 20

    `CONFIG_MTD_UBI_FASTMAP`
    Fastmap は、ほぼ一定時間で UBI デバイスをアタッチできるメカニズムです。MTD デバイス全体をスキャンする代わりに、デバイス上のチェックポイント (fastmap と呼ばれる) を見つけるだけで済みます。オンフラッシュ fastmap には、デバイスをアタッチするために必要なすべての情報が含まれています。fastmap の使用は、スキャンによるアタッチに時間がかかる大規模デバイスでのみ意味があります。UBI は古いイメージに自動的に fastmap をインストールしませんが、必要に応じて UBI パラメータ `CONFIG_MTD_UBI_FASTMAP_AUTOCONVERT` を 1 に設定できます。fastmap 対応イメージは、fastmap サポートのない UBI 実装でも引き続き使用可能であることに注意してください。典型的なフラッシュデバイスでは、fastmap 全体が 1 つの PEB に収まります。UBI は 2 つの fastmap を保持するために PEB を予約します。

    `CONFIG_MTD_UBI_FASTMAP_AUTOCONVERT`
    fastmap のないイメージで fastmap を自動的に有効にするには、このパラメータを設定します。
    デフォルト: 0

    `CONFIG_MTD_UBI_FM_DEBUG`
    UBI fastmap デバッグを有効にします。
    デフォルト: 0

- SPL フレームワーク
    `CONFIG_SPL`
    SPL のビルドをグローバルに有効にします。

    `CONFIG_SPL_LDSCRIPT`
    SPL バイナリをリンクするための LDSCRIPT。

    `CONFIG_SPL_MAX_FOOTPRINT`
    SPL に割り当てられるメモリの最大サイズ (BSS を含む)。定義されている場合、リンカは _start から __bss_end まで SPL が実際に使用するメモリがそれを超えないことをチェックします。
    `CONFIG_SPL_MAX_FOOTPRINT` と `CONFIG_SPL_BSS_MAX_SIZE` は、同時に定義してはなりません。

    `CONFIG_SPL_MAX_SIZE`
    SPL イメージの最大サイズ (text, data, rodata, およびリンカリストセクション、BSS を除く)。定義されている場合、リンカは実際のサイズがそれを超えないことをチェックします。

    `CONFIG_SPL_RELOC_TEXT_BASE`
    リロケート先のアドレス。指定されていない場合、これは `CONFIG_SPL_TEXT_BASE` と等しくなります (つまり、リロケーションは行われません)。

    `CONFIG_SPL_BSS_START_ADDR`
    SPL バイナリ内の BSS のリンクアドレス。

    `CONFIG_SPL_BSS_MAX_SIZE`
    SPL BSS に割り当てられるメモリの最大サイズ。
    定義されている場合、リンカは __bss_start から __bss_end まで SPL が実際に使用するメモリがそれを超えないことをチェックします。
    `CONFIG_SPL_MAX_FOOTPRINT` と `CONFIG_SPL_BSS_MAX_SIZE` は、同時に定義してはなりません。

    `CONFIG_SPL_STACK`
    SPL が使用するスタックの開始アドレス。

    `CONFIG_SPL_PANIC_ON_RAW_IMAGE`
    定義されている場合、ロードしたイメージに署名がない場合、SPL は panic() します。これは、SPL でイメージをロードするコードが、絶対にすべての読み取りエラーが検出されることを保証できない場合に役立ちます。例としては、LPC32XX MLC NAND ドライバがあります。これは、完全に読み取り不可能な NAND ブロックは不良と見なし、したがって静かにスキップされるべきであると考えます。

    `CONFIG_SPL_RELOC_STACK`
    リロケーション後に SPL が使用するスタックの開始アドレス。指定されていない場合、これは `CONFIG_SPL_STACK` と等しくなります。

    `CONFIG_SYS_SPL_MALLOC_START`
    SPL で使用される malloc プールの開始アドレス。
    このオプションが設定されている場合、SPL で完全な malloc が使用され、spl_init() によってセットアップされます。それ以前は、`CONFIG_SYS_MALLOC_F` が定義されていれば、単純な malloc() を使用できます。

    `CONFIG_SYS_SPL_MALLOC_SIZE`
    SPL で使用される malloc プールのサイズ。

    `CONFIG_SPL_OS_BOOT`
    SPL から直接 OS をブートすることを有効にします。
    参照: doc/README.falcon

    `CONFIG_SPL_DISPLAY_PRINT`
    ARM の場合、実行中のシステムに関する詳細情報を表示するオプションの関数を有効にします。

    `CONFIG_SPL_INIT_MINIMAL`
    アーキテクチャの初期化コードは、非常に小さなイメージ用にビルドされるべきです。

    `CONFIG_SYS_MMCSD_RAW_MODE_U_BOOT_PARTITION`
    MMC が raw モードで使用されている場合に U-Boot をロードする MMC 上のパーティション。

    `CONFIG_SYS_MMCSD_RAW_MODE_KERNEL_SECTOR`
    MMC が raw モードで使用されている場合にカーネル uImage をロードするセクタ (Falcon モード用)。

    `CONFIG_SYS_MMCSD_RAW_MODE_ARGS_SECTOR`,
    `CONFIG_SYS_MMCSD_RAW_MODE_ARGS_SECTORS`
    MMC が raw モードで使用されている場合にカーネル引数パラメータをロードするセクタとセクタ数 (Falcon モード用)。

    `CONFIG_SPL_FS_LOAD_PAYLOAD_NAME`
    ファイルシステムから読み込むときに U-Boot をロードするために読み取るファイル名。

    `CONFIG_SPL_FS_LOAD_KERNEL_NAME`
    ファイルシステムから読み込むときにカーネル uImage をロードするために読み取るファイル名 (Falcon モード用)。

    `CONFIG_SPL_FS_LOAD_ARGS_NAME`
    ファイルシステムから読み込むときにカーネル引数パラメータをロードするために読み取るファイル名 (Falcon モード用)。

    `CONFIG_SPL_MPC83XX_WAIT_FOR_NAND`
    PPC mpc83xx ターゲットの NAND SPL に対してこれを設定します。これにより、start.S は SPL の残りがロードされるのを待ってから続行します (ハードウェアは最初のページをロードしただけで実行を開始し、完全な 4K ではありません)。

    `CONFIG_SPL_SKIP_RELOCATE`
    SPL のリロケーションを回避します。

    `CONFIG_SPL_NAND_IDENT`
    SPL はチップ ID リストを使用して NAND フラッシュを識別します。
    `CONFIG_SPL_NAND_BASE` が必要です。

    `CONFIG_SPL_UBI`
    軽量 UBI (fastmap) スキャナとローダのサポート。

    `CONFIG_SPL_NAND_RAW_ONLY`
    raw u-boot.bin イメージのみをブートするサポート。スペースを節約する必要がある場合にのみ使用してください。

    `CONFIG_SPL_COMMON_INIT_DDR`
    SPL バイナリでのシリアルプレゼンスデテクトによる共通 DDR 初期化用に設定します。

    `CONFIG_SYS_NAND_5_ADDR_CYCLE`, `CONFIG_SYS_NAND_PAGE_COUNT`,
    `CONFIG_SYS_NAND_PAGE_SIZE`, `CONFIG_SYS_NAND_OOBSIZE`,
    `CONFIG_SYS_NAND_BLOCK_SIZE`, `CONFIG_SYS_NAND_BAD_BLOCK_POS`,
    `CONFIG_SYS_NAND_ECCPOS`, `CONFIG_SYS_NAND_ECCSIZE`,
    `CONFIG_SYS_NAND_ECCBYTES`
    SPL が U-Boot を読み取るために使用する NAND のサイズと動作を定義します。

    `CONFIG_SYS_NAND_U_BOOT_OFFS`
    NAND 内で U-Boot を読み取る場所。

    `CONFIG_SYS_NAND_U_BOOT_DST`
    メモリ内で U-Boot をロードする場所。

    `CONFIG_SYS_NAND_U_BOOT_SIZE`
    ロードするイメージのサイズ。

    `CONFIG_SYS_NAND_U_BOOT_START`
    ロードされたイメージ内でジャンプするエントリポイント。

    `CONFIG_SYS_NAND_HW_ECC_OOBFIRST`
    最初に OOB を読み取り、次にデータを読み取る必要がある場合にこれを定義します。これは、例えば davinci プラットフォームで使用されます。

    `CONFIG_SPL_RAM_DEVICE`
    SPL バイナリで、RAM に既に存在するイメージを実行するためのサポート。

    `CONFIG_SPL_PAD_TO`
    SPL ペイロードを追加する前に SPL をパディングするイメージオフセット。デフォルトでは、これは `CONFIG_SPL_MAX_SIZE` として定義されるか、`CONFIG_SPL_MAX_SIZE` が未定義の場合は 0 です。`CONFIG_SPL_PAD_TO` は、0 (パディングなしで SPL ペイロードを追加することを意味する) または `>= CONFIG_SPL_MAX_SIZE` である必要があります。

    `CONFIG_SPL_TARGET`
    SPL とペイロードを含む最終ターゲットイメージ。一部の SPL は、代わりにアーキテクチャ固有の makefile フラグメントを使用します。例えば、複数のイメージを生成する必要がある場合などです。

    `CONFIG_SPL_FIT_PRINT`
    FIT イメージに関する情報を表示すると、SPL にかなりの量のコードが追加されます。したがって、これは通常 SPL では無効になっています。これを再度有効にするには、このオプションを使用します。これは、FIT イメージをブートするときの bootm コマンドの出力に影響します。

- TPL フレームワーク
    `CONFIG_TPL`
    TPL のビルドをグローバルに有効にします。

    `CONFIG_TPL_PAD_TO`
    TPL ペイロードを追加する前に TPL をパディングするイメージオフセット。デフォルトでは、これは `CONFIG_SPL_MAX_SIZE` として定義されるか、`CONFIG_SPL_MAX_SIZE` が未定義の場合は 0 です。`CONFIG_SPL_PAD_TO` は、0 (パディングなしで SPL ペイロードを追加することを意味する) または `>= CONFIG_SPL_MAX_SIZE` である必要があります。

- 割り込みサポート (PPC):

    すべての PPC アーキテクチャに共通の `interrupt_init()` と `timer_interrupt()` があります。`interrupt_init()` は、CPU 固有の初期化のために `interrupt_init_cpu()` を呼び出します。`interrupt_init_cpu()` は `decrementer_count` を適切な値に設定する必要があります。CPU が割り込み後にデクリメンタを自動的にリセットする場合 (ppc4xx)、`decrementer_count` をゼロに設定する必要があります。`timer_interrupt()` は CPU 固有の処理のために `timer_interrupt_cpu()` を呼び出します。ボードにウォッチドッグ / ステータス LED / その他のアクティビティモニターがある場合、一般的な `timer_interrupt()` から自動的に動作します。

ボード初期化設定:
------------------------------

初期化中、U-Boot は、ドライバが初期化される前のボード固有の前提条件 (例: ピン設定) の準備を可能にするために、多数のボード固有の関数を呼び出します。これらのコールバックを有効にするには、以下の設定マクロを定義する必要があります。これは現在アーキテクチャ固有であるため、通常は `board_init_f()` および `board_init_r()` 内の `arch/your_architecture/lib/board.c` を確認してください。

- `CONFIG_BOARD_EARLY_INIT_F`: `board_early_init_f()` を呼び出す
- `CONFIG_BOARD_EARLY_INIT_R`: `board_early_init_r()` を呼び出す
- `CONFIG_BOARD_LATE_INIT`: `board_late_init()` を呼び出す
- `CONFIG_BOARD_POSTCLK_INIT`: `board_postclk_init()` を呼び出す

設定設定:
-----------------------

- `MEM_SUPPORT_64BIT_DATA`: 64 ビットとしてコンパイルされた場合に自動的に定義されます。オプションで、64 ビットメモリーコマンドをサポートするために定義できます。

- `CONFIG_SYS_LONGHELP`: 長いヘルプメッセージを含めたい場合に定義します。メモリが不足している場合は未定義にします。

- `CONFIG_SYS_HELP_CMD_WIDTH`: 'help' コマンドの出力にリストされるコマンドのデフォルト幅を上書きしたい場合に定義します。

- `CONFIG_SYS_PROMPT`: ユーザー入力を促すために U-Boot がコンソールに出力するものです。

- `CONFIG_SYS_CBSIZE`: コンソールからの入力用バッファサイズ。

- `CONFIG_SYS_PBSIZE`: コンソール出力用バッファサイズ。

- `CONFIG_SYS_MAXARGS`: モニターコマンドで受け入れられる引数の最大数。

- `CONFIG_SYS_BARGSIZE`: アプリケーション (通常は Linux カーネル) がブートされるときに渡されるブート引数用のバッファサイズ。

- `CONFIG_SYS_BAUDRATE_TABLE`: このボードで有効なボーレート設定のリスト。

- `CONFIG_SYS_MEM_RESERVE_SECURE`
    今のところ ARMv8 でのみ実装されています。
    定義されている場合、`CONFIG_SYS_MEM_RESERVE_SECURE` サイズのメモリが合計 RAM から差し引かれ、OS に報告されません。このメモリはセキュアメモリとして使用できます。変数 `gd->arch.secure_ram` が場所を追跡するために使用されます。RAM ベースがゼロでないシステム、または RAM がバンクに分割されているシステムでは、アドレスを取得するためにこの変数を再計算する必要があります。

- `CONFIG_SYS_MEM_TOP_HIDE`:
    `CONFIG_SYS_MEM_TOP_HIDE` がボード設定ヘッダーで定義されている場合、指定されたメモリ領域が RAM の最上位 (末尾) から差し引かれ、U-Boot によってまったく "触れられ" ません。`gd->ram_size` を修正することにより、Linux カーネルは「修正された」メモリサイズを渡され、それにも触れないはずです。これは arch/ppc および arch/powerpc で機能するはずです。SDRAM コントローラの設定からメモリサイズを再計算する bootwrapper サポートを備えた arch/powerpc の Linux ボードポートのみ、Linux でも追加で修正する必要があります。このオプションは、SDRAM の最後の 256 バイトに触れてはならない 440EPx/GRx CHIP 11 エラッタの回避策として使用できます。
    警告: この値が Linux のページサイズ (通常 4k) の倍数であることを確認してください。そうでない場合、Linux メモリの終了アドレスがページサイズにアラインされていないアドレスに配置され、これが大きな問題を引き起こす可能性があります。

- `CONFIG_SYS_LOADS_BAUD_CHANGE`:
    シリアルダウンロード中に一時的なボーレート変更を有効にします。

- `CONFIG_SYS_SDRAM_BASE`:
    SDRAM の物理開始アドレス。ここでは 0 でなければなりません。

- `CONFIG_SYS_FLASH_BASE`:
    フラッシュメモリの物理開始アドレス。

- `CONFIG_SYS_MONITOR_BASE`:
    ブートモニターコードの物理開始アドレス (リンク時に使用されるテキストベースアドレス (CONFIG_SYS_TEXT_BASE) と同じになるように make config ファイルによって設定されます) - フラッシュからブートする場合は `CONFIG_SYS_FLASH_BASE` と同じです。

- `CONFIG_SYS_MONITOR_LEN`:
    モニターコード用に予約されたメモリのサイズ。環境が U-Boot イメージ内に埋め込まれているか、別のフラッシュセクタにあるかをコンパイル時に (!) 判断するために使用されます。

- `CONFIG_SYS_MALLOC_LEN`:
    malloc() の使用のために予約された DRAM のサイズ。

- `CONFIG_SYS_MALLOC_F_LEN`
    リロケーション前に使用する malloc() プールのサイズ。これが定義されている場合、リロケーション前に非常に単純な malloc() 実装が利用可能になります。アドレスはグローバルデータのすぐ下にあり、スタックはスペースを確保するために下に移動されます。この機能は、領域内で増加するアドレスを持つ領域を割り当てます。calloc() はサポートされていますが、realloc() は利用できません。free() はサポートされていますが、何もしません。メモリは、U-Boot が自身をリロケートするときに解放されます (実際には忘れ去られるだけです)。

- `CONFIG_SYS_MALLOC_SIMPLE`
    SPL で完全な malloc (CONFIG_SYS_SPL_MALLOC_START で有効化) を使用しないボード向けに、単純で小さな malloc() および calloc() を提供します。

- `CONFIG_SYS_NONCACHED_MEMORY`:
    非キャッシュメモリ領域のサイズ。このメモリ領域は通常、malloc() 領域のすぐ下に配置され、MMU でキャッシュされずにマップされます。これは、そうでなければ多くの明示的なキャッシュメンテナンスを必要とするドライバに役立ちます。一部のドライバでは、キャッシュを適切に維持することも不可能です。例えば、フラッシュする必要がある領域がキャッシュラインサイズの倍数ではなく、*かつ* それらをアラインするために領域間にパディングを割り当てることができない場合 (つまり、HW が連続した領域配列を必要とし、各領域のサイズがキャッシュアラインされていない場合)、1 つの領域のフラッシュが、ハードウェアが同じキャッシュライン内の別の領域に書き込んだデータを上書きする可能性があります。これは、例えば、バッファのディスクリプタが通常 CPU キャッシュラインよりも小さい (例: 16 バイト対 32 または 64 バイト) ネットワークドライバで発生する可能性があります。非キャッシュメモリは現在 32 ビット ARM でのみサポートされています。

- `CONFIG_SYS_BOOTM_LEN`:
    通常、圧縮された uImage は、非圧縮サイズが 8 MBytes に制限されています。これで十分でない場合は、ボード設定ファイルで `CONFIG_SYS_BOOTM_LEN` を定義して、この設定をニーズに合わせて調整できます。

- `CONFIG_SYS_BOOTMAPSZ`:
    Linux カーネルのスタートアップコードによってマップされるメモリの最大サイズ。Linux カーネルによって処理されなければならないすべてのデータ (bd_info、ブート引数、使用されている場合は FDT blob) は、"bootm_low" 環境変数が定義されていてゼロでない限り、この制限以下に配置する必要があります。そのような場合、Linux カーネル用のすべてのデータは "bootm_low" と "bootm_low" + `CONFIG_SYS_BOOTMAPSZ` の間にある必要があります。「bootm_mapsize」環境変数は `CONFIG_SYS_BOOTMAPSZ` の値を上書きします。`CONFIG_SYS_BOOTMAPSZ` が未定義の場合、代わりに「bootm_size」の値が使用されます。

- `CONFIG_SYS_BOOT_RAMDISK_HIGH`:
    initrd_high 機能を有効にします。定義されている場合、initrd_high 機能が有効になり、bootm ramdisk サブコマンドが有効になります。

- `CONFIG_SYS_BOOT_GET_CMDLINE`:
    "bootm_low" と "bootm_low" + BOOTMAPSZ の間のスペースにカーネルコマンドラインを割り当てて保存することを有効にします。

- `CONFIG_SYS_BOOT_GET_KBD`:
    "bootm_low" と "bootm_low" + BOOTMAPSZ の間のスペースに bd_info のカーネルコピーを割り当てて保存することを有効にします。

- `CONFIG_SYS_MAX_FLASH_BANKS`:
    フラッシュメモリバンクの最大数。

- `CONFIG_SYS_MAX_FLASH_SECT`:
    フラッシュチップ上のセクタの最大数。

- `CONFIG_SYS_FLASH_ERASE_TOUT`:
    フラッシュ消去操作のタイムアウト (ms)。

- `CONFIG_SYS_FLASH_WRITE_TOUT`:
    フラッシュ書き込み操作のタイムアウト (ms)。

- `CONFIG_SYS_FLASH_LOCK_TOUT`
    フラッシュセクタロックビット設定操作のタイムアウト (ms)。

- `CONFIG_SYS_FLASH_UNLOCK_TOUT`
    フラッシュロックビットクリア操作のタイムアウト (ms)。

- `CONFIG_SYS_FLASH_PROTECTION`
    定義されている場合、U-Boot ソフトウェア保護の代わりにハードウェアフラッシュセクタ保護が使用されます。

- `CONFIG_SYS_DIRECT_FLASH_TFTP`:

    フラッシュメモリへの直接 TFTP 転送を有効にします。このオプションがない場合、そのようなダウンロードは 2 つのステップで実行する必要があります: (1) RAM へのダウンロード、(2) RAM からフラッシュへのコピー。通常、2 ステップアプローチの方が信頼性が高くなります。フラッシュを消去する前にダウンロードが機能したかどうかを確認できるためですが、状況によっては (システム RAM がダウンロードされたイメージの一時コピーを許可するには制限されすぎている場合)、このオプションが非常に役立つ場合があります。

- `CONFIG_SYS_FLASH_CFI`:
    フラッシュドライバがフラッシュジオメトリを格納するために共通フラッシュ構造に追加要素を使用する場合に定義します。

- `CONFIG_FLASH_CFI_DRIVER`
    このオプションは、ドライバディレクトリ内の cfi_flash ドライバのビルドも有効にします。

- `CONFIG_FLASH_CFI_MTD`
    このオプションは、ドライバディレクトリ内の cfi_mtd ドライバのビルドを有効にします。ドライバは CFI フラッシュを MTD レイヤーにエクスポートします。

- `CONFIG_SYS_FLASH_USE_BUFFER_WRITE`
    フラッシュへのバッファ書き込みを使用します。

- `CONFIG_FLASH_SPANSION_S29WS_N`
    s29ws-n MirrorBit フラッシュには、バッファ書き込みコマンド用の非標準アドレスがあります。

- `CONFIG_SYS_FLASH_QUIET_TEST`
    このオプションが定義されている場合、共通 CFI フラッシュは認識されない FLASH バンクに対する警告を出力しません。これは、構成されたバンクの一部がオプションでしか利用できない場合に役立ちます。

- `CONFIG_FLASH_SHOW_PROGRESS`
    定義されている場合 (整数である必要があります)、カウントダウンの数字とドットを出力します。推奨値: 80 カラム表示の場合は 45 (9..1)、40 カラム表示の場合は 15 (3..1)。

- `CONFIG_FLASH_VERIFY`
    定義されている場合、書き込み操作後にフラッシュ (宛先) の内容がソースと比較されます。内容が同一でない場合、エラーメッセージが出力されます。このようなフラッシュプログラミングエラーは通常、保護解除/消去/プログラミング中に早期に検出されるため、このオプションはほとんどの場合役に立たないことに注意してください。本当に何をしているのかを理解している場合にのみ、このオプションを有効にしてください。

- `CONFIG_SYS_RX_ETH_BUFFER`:
    イーサネット受信バッファの数を定義します。一部のイーサネットコントローラでは、高いイーサネットトラフィックでインターフェイスを有効にした直後にすべてのバッファがいっぱいになる可能性があるため、この値を 8 以上 (EEPRO100 または 405 EMAC) に設定することをお勧めします。定義されていない場合、デフォルトは 4 です。

- `CONFIG_ENV_MAX_ENTRIES`

    環境設定を内部的に格納するために使用されるハッシュテーブルのエントリの最大数。デフォルト設定は寛大であると想定されており、ほとんどの場合機能するはずです。この設定は動作を調整するために使用できます。詳細については lib/hashtable.c を参照してください。

- `CONFIG_ENV_FLAGS_LIST_DEFAULT`
- `CONFIG_ENV_FLAGS_LIST_STATIC`
    env set を呼び出すときに環境変数に与えられた値の検証を有効にします。変数は、10 進数、16 進数、またはブール値のみに制限できます。`CONFIG_CMD_NET` も定義されている場合、変数は IP アドレスまたは MAC アドレスにも制限できます。
    リストの形式は次のとおりです:
        type_attribute = [s|d|x|b|i|m]
        access_attribute = [a|r|o|c]
        attributes = type_attribute[access_attribute]
        entry = variable_name[:attributes]
        list = entry[,list]

    type 属性は次のとおりです:
        s - 文字列 (デフォルト)
        d - 10 進数
        x - 16 進数
        b - ブール値 ([1yYtT|0nNfF])
        i - IP アドレス
        m - MAC アドレス

    access 属性は次のとおりです:
        a - 任意 (デフォルト)
        r - 読み取り専用
        o - 一度だけ書き込み可能
        c - デフォルトの変更

    - `CONFIG_ENV_FLAGS_LIST_DEFAULT`
        デフォルトまたは埋め込み環境で ".flags" 環境変数を定義するために、これをリスト (文字列) に定義します。
    - `CONFIG_ENV_FLAGS_LIST_STATIC`
        ".flags" 環境変数にエントリが見つからない場合に実行されるべき検証を定義するために、これをリスト (文字列) に定義します。静的リストの設定を上書きするには、".flags" 変数に同じ変数名のエントリを追加するだけです。

    `CONFIG_REGEX` が定義されている場合、上記の variable_name は正規表現として評価されます。これにより、各変数に明示的にリストすることなく、複数の変数が同じフラグを定義できます。

以下の定義は、環境データ (変数領域) の配置と管理を扱います。一般に、以下の設定をサポートしています:

- `CONFIG_BUILD_ENVCRC`:

    ターゲット環境で envcrc をビルドアップし、外部ユーティリティがそれを簡単に抽出して最終的な U-Boot イメージに埋め込むことができるようにします。
    注意してください！ 環境への最初のアクセスは、U-Boot の初期化のかなり早い段階 (コンソールボーレートの設定を取得しようとするとき) に発生します。その時点で NVRAM 領域をマップしておく必要があります。そうしないと U-Boot はハングします。NVRAM を使用する場合でも、RAM 内の環境のコピーを使用することに注意してください。NVRAM で直接作業することもできますが、誰かが現在の設定を保存するために "saveenv" を使用する場合を除き、常にそこの設定を変更しないようにしたいからです。
    注意してください！ いくつかの特殊なケースでは、ローカルデバイスは "saveenv" コマンドを使用できません。例えば、ローカルデバイスは SRIO または PCIE リンクによってリモート NOR フラッシュに格納された環境を取得しますが、SRIO または PCIE インターフェースによってこの NOR フラッシュを消去、書き込みすることはできません。

- `CONFIG_NAND_ENV_DST`

    nand_spl コードが環境をコピーするべき RAM 内のアドレスを定義します。冗長環境が使用されている場合、それは `CONFIG_NAND_ENV_DST + CONFIG_ENV_SIZE` にコピーされます。
    モニターが RAM にリロケートされ、環境の RAM コピーが作成されるまで、環境は読み取り専用であることに注意してください。また、EEPROM を使用する場合、それまでは環境変数を読み取るために `env_get_f()` を使用する必要があります。
    環境は CRC32 チェックサムによって保護されています。モニターが RAM にリロケートされる前に、CRC が不正な結果として、コンパイル時に組み込まれたデフォルト環境で作業することになります - *静かに*！！！ [これは必要です。なぜなら、最初に必要な環境変数はコンソールの "baudrate" 設定だからです - CRC が不正な場合、文句を言うことができるデバイスがまだありません。]

    注: モニターがリロケートされると、デフォルト環境が使用されている場合に文句を言うようになります。「saveenv」コマンドを使用して有効な環境を格納するとすぐに、新しい CRC が計算されます。

- `CONFIG_SYS_FAULT_ECHO_LINK_DOWN`:
    反転されたイーサネットリンク状態をフォルト LED にエコーします。
    注: このオプションがアクティブな場合、`CONFIG_SYS_FAULT_MII_ADDR` も定義する必要があります。

- `CONFIG_SYS_FAULT_MII_ADDR`:
    イーサネットリンク状態をチェックするための PHY の MII アドレス。

- `CONFIG_NS16550_MIN_FUNCTIONS`:
    drivers/serial/ns16550.c にあるシリアルドライバの NS16550_init および NS16550_putc 関数のみを使用したい場合にこれを定義します。このオプションは、NAND_SPL 設定を含むがこれに限定されない、すでに大幅に制限されたイメージのスペースを節約するのに役立ちます。

- `CONFIG_DISPLAY_BOARDINFO`
    U-Boot が起動するときに、U-Boot が実行されているボードに関する情報を表示します。ボード関数 `checkboard()` がこれを実行するために呼び出されます。

- `CONFIG_DISPLAY_BOARDINFO_LATE`
    前のオプションと同様ですが、stdio が実行され、出力が LCD (存在する場合) に送られるようになった後、この情報を後で表示します。

- `CONFIG_BOARD_SIZE_LIMIT`:
    U-Boot イメージの最大サイズ。定義されている場合、ビルドシステムは実際のサイズがそれを超えないことをチェックします。

低レベル (ハードウェア関連) 設定オプション:
---------------------------------------------------

- `CONFIG_SYS_CACHELINE_SIZE`:
    CPU のキャッシュラインサイズ。

- `CONFIG_SYS_CCSRBAR_DEFAULT`:
    Freescale PowerPC SOC 上の CCSR のデフォルト (パワーオンリセット) 物理アドレス。

- `CONFIG_SYS_CCSRBAR`:
    CCSR の仮想アドレス。32 ビットビルドでは、これは通常 `CONFIG_SYS_CCSRBAR_DEFAULT` と同じ値です。

- `CONFIG_SYS_CCSRBAR_PHYS`:
    CCSR の物理アドレス。必要に応じて、CCSR は新しい物理アドレスに再配置できます。この場合、このマクロはそのアドレスに設定する必要があります。それ以外の場合は、`CONFIG_SYS_CCSRBAR_DEFAULT` と同じ値に設定する必要があります。例えば、CCSR は通常 36 ビットビルドで再配置されます。このマクロは _HIGH および _LOW マクロを介して定義することをお勧めします:

    ```c
    #define CONFIG_SYS_CCSRBAR_PHYS ((CONFIG_SYS_CCSRBAR_PHYS_HIGH * 1ull) << 32 | CONFIG_SYS_CCSRBAR_PHYS_LOW)
    ```


- `CONFIG_SYS_CCSRBAR_PHYS_HIGH`:
    `CONFIG_SYS_CCSRBAR_PHYS` のビット 33-36。この値は通常、0 (32 ビットビルド) または 0xF (36 ビットビルド) のいずれかです。このマクロはアセンブリコードで使用されるため、型キャストや整数サイズの接尾辞 (例: "ULL") を含んではなりません。

- `CONFIG_SYS_CCSRBAR_PHYS_LOW`:
    `CONFIG_SYS_CCSRBAR_PHYS` の下位 32 ビット。このマクロはアセンブリコードで使用されるため、型キャストや整数サイズの接尾辞 (例: "ULL") を含んではなりません。

- `CONFIG_SYS_CCSR_DO_NOT_RELOCATE`:
    このマクロが定義されている場合、`CONFIG_SYS_CCSRBAR_PHYS` は CCSR が再配置されないことを保証する値に強制されます。

- `CONFIG_IDE_AHB`:
    ほとんどの IDE コントローラは PCI インターフェースで接続するように設計されていました。AHB インターフェース用に設計されたものは少数です。ソフトウェアが IDE-AHB コントローラを介して IDE デバイスへの ATA コマンドおよびデータ転送を行う場合、これらの種類の IDE-AHB コントローラへのいくつかの追加レジスタアクセスが必要です。

- `CONFIG_SYS_IMMR`:	内部メモリの物理アドレス。
    正確に何をしているのかを理解していない限り、変更しないでください！ (11-4) [MPC8xx システムのみ]

- `CONFIG_SYS_INIT_RAM_ADDR`:

    初期データとスタックに使用できるメモリ領域の開始アドレス。特別な初期化なしで動作する書き込み可能なメモリでなければならないことに注意してください。つまり、メモリコントローラをプログラミングし、特定の初期化シーケンスを実行した後にのみ利用可能になる通常の RAM を使用することはできません。
    U-Boot は以下のメモリタイプを使用します:
    - MPC8xx: IMMR (CPU の内部メモリ)

- `CONFIG_SYS_GBL_DATA_OFFSET`:

    `CONFIG_SYS_INIT_RAM_ADDR` で定義されたメモリ領域内の初期データ構造のオフセット。通常、`CONFIG_SYS_GBL_DATA_OFFSET` は、初期データが利用可能なスペースの最後に配置されるように選択され (時には (CONFIG_SYS_INIT_RAM_SIZE - GENERATED_GBL_DATA_SIZE) と書かれます)、初期スタックはその領域のすぐ下にあります ((CONFIG_SYS_INIT_RAM_ADDR + CONFIG_SYS_GBL_DATA_OFFSET) から下向きに成長します)。

    注:
        MPC824X (または初期メモリにデータキャッシュを使用する他のシステム) では、`CONFIG_SYS_INIT_RAM_ADDR` に選択されたアドレスは基本的に任意です - RAM の最上位と PCI スペースの開始の間にある、それ以外では未使用のアドレス空間を指す必要があります。

- `CONFIG_SYS_SCCR`:	システムクロックおよびリセットコントロールレジスタ (15-27)

- `CONFIG_SYS_OR_TIMING_SDRAM`:
    SDRAM タイミング

- `CONFIG_SYS_MAMR_PTA`:
    リフレッシュ用の周期タイマー

- `FLASH_BASE0_PRELIM`, `FLASH_BASE1_PRELIM`, `CONFIG_SYS_REMAP_OR_AM`,
  `CONFIG_SYS_PRELIM_OR_AM`, `CONFIG_SYS_OR_TIMING_FLASH`, `CONFIG_SYS_OR0_REMAP`,
  `CONFIG_SYS_OR0_PRELIM`, `CONFIG_SYS_BR0_PRELIM`, `CONFIG_SYS_OR1_REMAP`, `CONFIG_SYS_OR1_PRELIM`,
  `CONFIG_SYS_BR1_PRELIM`:
    メモリコントローラ定義: BR0/1 および OR0/1 (FLASH)

- `SDRAM_BASE2_PRELIM`, `SDRAM_BASE3_PRELIM`, `SDRAM_MAX_SIZE`,
  `CONFIG_SYS_OR_TIMING_SDRAM`, `CONFIG_SYS_OR2_PRELIM`, `CONFIG_SYS_BR2_PRELIM`,
  `CONFIG_SYS_OR3_PRELIM`, `CONFIG_SYS_BR3_PRELIM`:
    メモリコントローラ定義: BR2/3 および OR2/3 (SDRAM)

- `CONFIG_SYS_SRIO`:
    チップに SRIO があるかないか

- `CONFIG_SRIO1`:
    ボードに SRIO 1 ポートが利用可能

- `CONFIG_SRIO2`:
    ボードに SRIO 2 ポートが利用可能

- `CONFIG_SRIO_PCIE_BOOT_MASTER`
    ボードは SRIO および PCIE からのブートのマスター機能をサポートできます

- `CONFIG_SYS_SRIOn_MEM_VIRT`:
    SRIO ポート 'n' メモリ領域の仮想アドレス

- `CONFIG_SYS_SRIOn_MEM_PHYxS`:
    SRIO ポート 'n' メモリ領域の物理アドレス

- `CONFIG_SYS_SRIOn_MEM_SIZE`:
    SRIO ポート 'n' メモリ領域のサイズ

- `CONFIG_SYS_NAND_BUSWIDTH_16BIT`
    NAND チップが 16 ビットバスを使用していることを NAND コントローラに伝えるために定義されます。すべての NAND ドライバがこのシンボルを使用するわけではありません。
    それを使用するドライバの例:
    - drivers/mtd/nand/raw/ndfc.c
    - drivers/mtd/nand/raw/mxc_nand.c

- `CONFIG_SYS_NDFC_EBC0_CFG`
    NDFC の EBC0_CFG レジスタを設定します。定義されていない場合、デフォルト値が使用されます。

- `CONFIG_SPD_EEPROM`
    I2C EEPROM から DDR タイミング情報を取得します。SODIMM などのプラグ可能なメモリモジュールで一般的です。

  `SPD_EEPROM_ADDRESS`
    SPD EEPROM の I2C アドレス。

- `CONFIG_SYS_SPD_BUS_NUM`
    SPD EEPROM が最初の I2C バス以外にある場合、ここで指定します。値はドライバが処理できるものに解決される必要があることに注意してください。

- `CONFIG_SYS_DDR_RAW_TIMING`
    SPD 以外から DDR タイミング情報を取得します。SPD なしでオンボードにはんだ付けされた DDR チップで一般的です。DDR raw タイミングパラメータはデータシートから抽出され、ヘッダーファイルまたはボード固有のファイルにハードコーディングされます。

- `CONFIG_FSL_DDR_INTERACTIVE`
    インタラクティブな DDR デバッグを有効にします。doc/README.fsl-ddr を参照してください。

- `CONFIG_FSL_DDR_SYNC_REFRESH`
    複数のコントローラの更新の同期を有効にします。

- `CONFIG_FSL_DDR_BIST`
    Freescale DDR コントローラ用の組み込みメモリテストを有効にします。

- `CONFIG_SYS_83XX_DDR_USES_CS0`
    83xx システムのみ。指定されている場合、DDR は CS2 および CS3 の代わりに CS0 および CS1 を使用して構成する必要があります。

- `CONFIG_RMII`
    すべての FEC に対して RMII モードを有効にします。
    これはグローバルオプションであり、1 つの FEC が標準 MII モードで、もう 1 つが RMII モードであることはできないことに注意してください。

- `CONFIG_CRC32_VERIFY`
    crc32 コマンドに検証オプションを追加します。
    構文は次のとおりです:

    ```
    => crc32 -v <address> <count> <crc32>
    ```

    ここで、address/count はメモリエリアを示し、crc32 はエリアが持つべき正しい crc32 です。

- `CONFIG_LOOPW`
    "loopw" メモリコマンドを追加します。これは、メモリコマンドがグローバルにアクティブ化されている (CONFIG_CMD_MEMORY) 場合にのみ有効になります。

- `CONFIG_CMD_MX_CYCLIC`
    "mdc" および "mwc" メモリコマンドを追加します。これらは周期的な "md/mw" コマンドです。
    例:

    ```
    => mdc.b 10 4 500
    ```
    このコマンドは、500 ms ごとに 4 バイト (10,11,12,13) を表示します。

    ```
    => mwc.l 100 12345678 10
    ```
    このコマンドは、10 ms ごとにアドレス 100 に 12345678 を書き込みます。
    これは、メモリコマンドがグローバルにアクティブ化されている (CONFIG_CMD_MEMORY) 場合にのみ有効になります。

- `CONFIG_SKIP_LOWLEVEL_INIT`
    [ARM, NDS32, MIPS, RISC-V のみ] この変数が定義されている場合、特定の低レベル初期化 (メモリコントローラの設定など) が省略され、および/または U-Boot は自身を RAM に再配置しません。通常、この変数は定義してはなりません。唯一の例外は、U-Boot がこれらの初期化を自身で実行する他のブートローダーまたはデバッガによって (RAM に) ロードされる場合です。

- `CONFIG_SKIP_LOWLEVEL_INIT_ONLY`
    [ARM926EJ-S のみ] これにより、`lowlevel_init()` の呼び出しのみをスキップできます。通常の CP15 初期化 (命令キャッシュの有効化など) は依然として実行されます。

- `CONFIG_SPL_BUILD`
    現在実行中のコンパイルが、(TPL や U-Boot 本体とは対照的に) SPL に最終的に含まれる成果物用である場合に設定されます。ステージ固有の動作を必要とするコードは、これをチェックする必要があります。

- `CONFIG_TPL_BUILD`
    現在実行中のコンパイルが、(SPL や U-Boot 本体とは対照的に) TPL に最終的に含まれる成果物用である場合に設定されます。ステージ固有の動作を必要とするコードは、これをチェックする必要があります。

- `CONFIG_SYS_MPC85XX_NO_RESETVEC`
    85xx システムのみ。この変数が指定されている場合、セクション .resetvec は保持されず、セクション .bootpg は .text セクションの前の 4k に配置されます。

- `CONFIG_ARCH_MAP_SYSMEM`
    一般に U-Boot (特に md コマンド) は実効アドレスを使用します。したがって、U-Boot アドレスを物理アドレスに変換する必要がある仮想アドレスとして見なす必要はありません。ただし、sandbox はこれを必要とします。なぜなら、すべてのアドレス指定可能なメモリを含む独自の小さな RAM バッファを維持するからです。このオプションにより、一部のメモリアクセスが `map_sysmem()` / `unmap_sysmem()` を介してマップされます。

- `CONFIG_X86_RESET_VECTOR`
    定義されている場合、x86 リセットベクタコードが含まれます。これは、U-Boot が Coreboot から実行されている場合には必要ありません。

- `CONFIG_SYS_NAND_NO_SUBPAGE_WRITE`
    NAND ドライバでサブページ書き込みを無効にするオプション。
    これを使用するドライバ:
    drivers/mtd/nand/raw/davinci_nand.c

Freescale QE/FMAN ファームウェアサポート:
-----------------------------------

Freescale QUICCEngine (QE) および Frame Manager (FMAN) は、両方とも QE ファームウェアバイナリ形式でエンコードされた「ファームウェア」のロードをサポートします。このファームウェアは、しばしば U-Boot のブート中にロードする必要があるため、ストレージデバイス (NOR フラッシュ、SPI など) とそのデバイス内のアドレスを識別するためにマクロが使用されます。

- `CONFIG_SYS_FMAN_FW_ADDR`
    FMAN マイクロコードが配置されているストレージデバイス内のアドレス。このアドレスの意味は、どの `CONFIG_SYS_QE_FMAN_FW_IN_xxx` マクロも指定されているかによって異なります。

- `CONFIG_SYS_QE_FW_ADDR`
    QE マイクロコードが配置されているストレージデバイス内のアドレス。このアドレスの意味は、どの `CONFIG_SYS_QE_FMAN_FW_IN_xxx` マクロも指定されているかによって異なります。

- `CONFIG_SYS_QE_FMAN_FW_LENGTH`
    ファームウェアの最大可能サイズ。ファームウェアバイナリ形式には、ファームウェアの実際のサイズを指定するフィールドがありますが、ファームウェア全体を保持するためのローカルストレージが最初に割り当てられない限り、ファームウェアのどの部分も読み取ることができない場合があります。

- `CONFIG_SYS_QE_FMAN_FW_IN_NOR`
    QE/FMAN ファームウェアが NOR フラッシュに配置され、LBC を介して通常のアドレス指定可能なメモリとしてマップされることを指定します。`CONFIG_SYS_FMAN_FW_ADDR` は NOR フラッシュ内の仮想アドレスです。

- `CONFIG_SYS_QE_FMAN_FW_IN_NAND`
    QE/FMAN ファームウェアが NAND フラッシュに配置されることを指定します。`CONFIG_SYS_FMAN_FW_ADDR` は NAND フラッシュ内のオフセットです。

- `CONFIG_SYS_QE_FMAN_FW_IN_MMC`
    QE/FMAN ファームウェアがプライマリ SD/MMC デバイスに配置されることを指定します。`CONFIG_SYS_FMAN_FW_ADDR` はそのデバイス上のバイトオフセットです。

- `CONFIG_SYS_QE_FMAN_FW_IN_REMOTE`
    QE/FMAN ファームウェアがリモート (マスター) メモリ空間に配置されることを指定します。`CONFIG_SYS_FMAN_FW_ADDR` は、スレーブ TLB->スレーブ LAW->スレーブ SRIO または PCIE アウトバウンドウィンドウ->マスターインバウンドウィンドウ->マスター LAW->マスターのメモリ空間内の ucode アドレスにマップできる仮想アドレスです。

Freescale Layerscape Management Complex ファームウェアサポート:
---------------------------------------------------------
Freescale Layerscape Management Complex (MC) は「ファームウェア」のロードをサポートします。このファームウェアは、しばしば U-Boot のブート中にロードする必要があるため、ストレージデバイス (NOR フラッシュ、SPI など) とそのデバイス内のアドレスを識別するためにマクロが使用されます。

- `CONFIG_FSL_MC_ENET`
    Layerscape SoC 用の MC ドライバを有効にします。

Freescale Layerscape デバッグサーバーサポート:
-------------------------------------------
Freescale Layerscape デバッグサーバーサポートは、「デバッグサーバーファームウェア」のロードと SP boot-rom のトリガーをサポートします。このファームウェアは、しばしば U-Boot のブート中にロードする必要があります。

- `CONFIG_SYS_MC_RSV_MEM_ALIGN`
    MC が必要とする予約メモリのアライメントを定義します。

再現可能なビルド
-------------------

再現可能なビルドを実現するためには、U-Boot ビルドプロセスで使用されるタイムスタンプを固定値に設定する必要があります。これは `SOURCE_DATE_EPOCH` 環境変数を使用して行われます。`SOURCE_DATE_EPOCH` は、U-Boot の設定オプションや U-Boot の環境変数としてではなく、ビルドホストのシェルで設定する必要があります。`SOURCE_DATE_EPOCH` は、エポックからの秒数を UTC で設定する必要があります。

ソフトウェアのビルド:
======================

U-Boot のビルドは、いくつかのネイティブビルド環境と多くの異なるクロス環境でテストされています。もちろん、すべての (潜在的に廃止された) バージョンのクロス開発ツールのすべての存在する可能性のあるバージョンをサポートすることはできません。ツールチェーンの問題が発生した場合、U-Boot のビルドとテストに広く使用されている ELDK (https://www.denx.de/wiki/DULG/ELDK を参照) を使用することをお勧めします。
ネイティブ環境を使用していない場合、GNU クロスコンパイルツールがパスで利用可能であると想定されます。この場合、シェルで環境変数 `CROSS_COMPILE` を設定する必要があります。Makefile や他のソースファイルを変更する必要はないことに注意してください。例えば、4xx CPU で ELDK を使用する場合、次のように入力してください:

    $ CROSS_COMPILE=ppc_4xx-
    $ export CROSS_COMPILE

U-Boot は簡単にビルドできるように意図されています。ソースをインストールした後、U-Boot を特定のボードタイプ用に設定する必要があります。これは次のように入力して行います:

    make NAME_defconfig

ここで、"NAME_defconfig" は既存の設定の名前の 1 つです。サポートされている名前については configs/*_defconfig を参照してください。
注: 一部のボードでは特別な設定名が存在する場合があります。ボードベンダーから追加情報が入手可能かどうかを確認してください。例えば、TQM823L システムは LCD サポートなし (標準) またはありで利用可能です。設定を選択するときに、そのような追加の「機能」を選択できます。つまり:

    make TQM823L_defconfig
    - プレーンな TQM823L 用に設定します、つまり LCD サポートなし

    make TQM823L_LCD_defconfig
    - LCD 上の U-Boot コンソールを備えた TQM823L 用に設定します

    など


最後に、"make all" と入力すると、システムへのダウンロード / インストール準備ができた動作する U-Boot イメージが得られるはずです:

- "u-boot.bin" は raw バイナリイメージです
- "u-boot" は ELF バイナリ形式のイメージです
- "u-boot.srec" は Motorola S-Record 形式です

デフォルトでは、ビルドはローカルで実行され、オブジェクトはソースディレクトリに保存されます。この動作を変更し、U-Boot を外部ディレクトリにビルドするには、次の 2 つの方法のいずれかを使用できます:

1. make コマンドライン呼び出しに O= を追加します:

    ```bash
    make O=/tmp/build distclean
    make O=/tmp/build NAME_defconfig
    make O=/tmp/build all
    ```

2. 環境変数 KBUILD_OUTPUT を目的の場所を指すように設定します:

    ```bash
    export KBUILD_OUTPUT=/tmp/build
    make distclean
    make NAME_defconfig
    make all
    ```

コマンドラインの "O=" 設定は KBUILD_OUTPUT 環境変数を上書きすることに注意してください。
ユーザー固有の CPPFLAGS、AFLAGS、CFLAGS は、対応する環境変数 KCPPFLAGS、KAFLAGS、KCFLAGS を設定することでコンパイラに渡すことができます。例えば、すべてのコンパイラ警告をエラーとして扱うには:

    make KCFLAGS=-Werror

Makefile は GNU make を使用していることを想定しているため、例えば NetBSD ではネイティブの "make" の代わりに "gmake" を使用する必要がある場合があることに注意してください。
お持ちのシステムボードがリストにない場合、U-Boot をハードウェアプラットフォームに移植する必要があります。これを行うには、次の手順に従います:

1. ボード固有のコードを保持するための新しいディレクトリを作成します。必要なファイルを追加します。ボードディレクトリには、少なくとも "Makefile" と "<board>.c" が必要になります。
2. ボード用の新しい設定ファイル "include/configs/<board>.h" を作成します。
3. U-Boot を新しい CPU に移植する場合は、CPU 固有のコードを保持するための新しいディレクトリも作成します。必要なファイルを追加します。
4. 新しい名前で "make <board>_defconfig" を実行します。
5. "make" と入力すると、ターゲットシステムにインストールする準備ができた動作する "u-boot.srec" ファイルが得られるはずです。
6. 発生する可能性のある問題をデバッグして解決します。[もちろん、この最後のステップは言うほど簡単ではありません。]


U-Boot の変更、新しいハードウェアへの移植などのテスト:
==============================================================

U-Boot ソースを変更した場合 (例えば、新しいボードや新しいデバイスのサポート、新しい CPU などを追加した場合)、他の開発者にフィードバックを提供することが期待されます。フィードバックは通常、「パッチ」、つまり U-Boot ソースの特定の (最新の公式または git リポジトリの最新) バージョンに対するコンテキスト diff の形式を取ります。
しかし、そのようなパッチを提出する前に、変更が既存のコードを壊していないことを確認してください。少なくとも、サポートされている *すべて* のボードが *いかなる* コンパイラ警告 *なし* でコンパイルされることを確認してください。これを行うには、サポートされているすべてのシステムに対して U-Boot を設定およびビルドする buildman スクリプト (tools/buildman/buildman) を実行するだけです。これには時間がかかることに注意してください。buildman README を参照するか、ドキュメントのために 'buildman -H' を実行してください。


以下の「U-Boot 移植ガイド」も参照してください。

モニターコマンド - 概要:
============================

go      - アドレス 'addr' でアプリケーションを開始
run     - 環境変数内のコマンドを実行
bootm   - メモリからアプリケーションイメージをブート
bootp   - BootP/TFTP プロトコルを使用してネットワーク経由でイメージをブート
bootz   - メモリから zImage をブート
tftpboot- TFTP プロトコルと環境変数 "ipaddr" および "serverip"
          (および場合によっては "gatewayip") を使用してネットワーク経由でイメージをブート
tftpput - TFTP プロトコルを使用してネットワーク経由でファイルをアップロード
rarpboot- RARP/TFTP プロトコルを使用してネットワーク経由でイメージをブート
diskboot- IDE デバイスからブート
bootd   - デフォルトでブート、つまり 'bootcmd' を実行
loads   - シリアルライン経由で S-Record ファイルをロード
loadb   - シリアルライン (kermit モード) 経由でバイナリファイルをロード
md      - メモリ表示
mm      - メモリ変更 (自動インクリメント)
nm      - メモリ変更 (定数アドレス)
mw      - メモリ書き込み (フィル)
ms      - メモリ検索
cp      - メモリコピー
cmp     - メモリ比較
crc32   - チェックサム計算
i2c     - I2C サブシステム
sspi    - SPI ユーティリティコマンド
base    - アドレスオフセットの表示または設定
printenv- 環境変数の表示
pwm     - pwm チャネルの制御
setenv  - 環境変数の設定
saveenv - 環境変数を永続ストレージに保存
protect - FLASH 書き込み保護の有効化または無効化
erase   - FLASH メモリの消去
flinfo  - FLASH メモリ情報の表示
nand    - NAND メモリ操作 (doc/README.nand を参照)
bdinfo  - ボード情報構造体の表示
iminfo  - アプリケーションイメージのヘッダ情報の表示
coninfo - コンソールデバイスと情報の表示
ide     - IDE サブシステム
loop    - アドレス範囲での無限ループ
loopw   - アドレス範囲での無限書き込みループ
mtest   - 簡単な RAM テスト
icache  - 命令キャッシュの有効化または無効化
dcache  - データキャッシュの有効化または無効化
reset   - CPU のリセットを実行
echo    - 引数をコンソールにエコー
version - モニターバージョンの表示
help    - オンラインヘルプの表示
?       - 'help' のエイリアス


モニターコマンド - 詳細説明:
========================================

TODO。

今のところ: "help <command>" と入力してください。

環境変数:
======================

U-Boot は、環境変数を使用したユーザー設定をサポートしており、フラッシュメモリに保存することで永続化できます。環境変数は "setenv" を使用して設定され、"printenv" を使用して表示され、"saveenv" を使用してフラッシュに保存されます。値なしで "setenv" を使用すると、環境から変数を削除できます。環境を保存しない限り、メモリ内のコピーで作業しています。環境を含むフラッシュ領域が誤って消去された場合に備えて、デフォルト環境が提供されます。一部の設定オプションは、環境変数を使用して設定できます。

環境変数のリスト (おそらく完全ではありません):

  baudrate    - CONFIG_BAUDRATE を参照

  bootdelay   - CONFIG_BOOTDELAY を参照

  bootcmd     - CONFIG_BOOTCOMMAND を参照

  bootargs    - RTOS イメージをブートするときのブート引数

  bootfile    - TFTP でロードするイメージの名前

  bootm_low   - bootm コマンドでのイメージ処理に利用可能なメモリ範囲を制限できます。この変数は 16 進数として与えられ、bootm コマンドで使用が許可される最も低いアドレスを定義します。"bootm_size" 環境変数も参照してください。"bootm_low" によって定義されたアドレスは、Linux カーネルの初期メモリマッピングのベースでもあります -- CONFIG_SYS_BOOTMAPSZ および bootm_mapsize の説明を参照してください。

  bootm_mapsize - Linux カーネルの初期メモリマッピングのサイズ。この変数は 16 進数として与えられ、ベースアドレス bootm_low から始まる、早期ブート中に Linux カーネルがアクセス可能なメモリ領域のサイズを定義します。設定されていない場合、定義されていれば CONFIG_SYS_BOOTMAPSZ がデフォルト値として使用され、それ以外の場合は bootm_size が使用されます。

  bootm_size    - bootm コマンドでのイメージ処理に利用可能なメモリ範囲を制限できます。この変数は 16 進数として与えられ、bootm コマンドで使用が許可される領域のサイズを定義します。"bootm_low" 環境変数も参照してください。

  bootstopkeysha256, bootdelaykey, bootstopkey  - README.autoboot を参照

  updatefile  - 自動ソフトウェア更新機能で使用される、TFTP サーバー上のソフトウェア更新ファイルの場所。詳細については doc/README.update のドキュメントを参照してください。

  autoload    - "no" ( 'n' で始まる任意の文字列) に設定されている場合、"bootp" は BOOTP サーバーから設定のルックアップを実行するだけで、TFTP を使用してイメージをロードしようとはしません。

  autostart   - "yes" に設定されている場合、"bootp"、"rarpboot"、"tftpboot"、または "diskboot" コマンドを使用してロードされたイメージは自動的に開始されます (内部的に "bootm" を呼び出すことによって)。

              - "no" に設定されている場合、"bootm" コマンドに渡されたスタンドアロンイメージはロードアドレスにコピーされます (そして最終的に展開されます) が、開始されません。これは、任意のデータをロードして展開するために使用できます。

  fdt_high    - 設定されている場合、ブート時にフラット化デバイスツリーがコピーされる最大アドレスを制限します。例えば、物理アドレス 0x10000000 に 1 GB のメモリがあるシステムで、Linux カーネルが最初の 704 MB のみを低位メモリとして認識する場合、デバイスツリー BLOB が 704 MB の低位メモリの最大アドレスにコピーされるように fdt_high を 0x3C000000 に設定する必要がある場合があります。これにより、Linux カーネルはブート手順中にアクセスできます。これが特別な値 0xFFFFFFFF に設定されている場合、fdt はブート時にまったくコピーされません。これが機能するためには、書き込み可能なメモリに存在し、U-Boot が必要な情報を追加するための十分なパディングが末尾にあり、メモリがカーネルからアクセス可能である必要があります。

  fdtcontroladdr- 設定されている場合、これは CONFIG_OF_CONTROL が定義されているときに U-Boot が使用する制御用フラット化デバイスツリーのアドレスです。

  i2cfast     - (PPC405GP|PPC405EP のみ) 'y' に設定されている場合、Linux I2C ドライバを高速モード (400kHZ) 用に設定します。この環境変数は初期化コードで使用されます。したがって、変更を有効にするには、保存してボードをリセットする必要があります。

  initrd_high - initrd イメージの配置を制限します:
              - この変数が設定されていない場合、initrd イメージは RAM 内の可能な限り高いアドレスにコピーされます。これにより initrd サイズが最大になるため、通常はこれが望ましい動作です。何らかの理由で initrd イメージが `CONFIG_SYS_BOOTMAPSZ` の制限以下にロードされることを確認したい場合は、この環境変数を "no"、"off"、または "0" の値に設定できます。あるいは、使用する最大上位アドレスに設定することもできます (U-Boot は依然として U-Boot スタックとデータを上書きしないことをチェックします)。例えば、16 MB の RAM を持つシステムがあり、Linux による使用から 4 MB を予約したい場合は、「bootargs」変数の値に "mem=12M" を追加することでこれを行うことができます。ただし、今度は initrd イメージも最初の 12 MB に配置されることを確認する必要があります - これは次のようにして行うことができます:

                `setenv initrd_high 00c00000`

              - `initrd_high` を 0xFFFFFFFF に設定した場合、これはフラッシュメモリ内のアドレスを含むすべてのアドレスが Linux カーネルにとって有効であることを U-Boot に示します。この場合、U-Boot は RAM ディスクをまったくコピーしません。これはシステムのブート時間を短縮するのに役立つ場合がありますが、この機能が Linux カーネルでサポートされている必要があります。

  ipaddr      - IP アドレス; tftpboot コマンドに必要

  loadaddr    - "bootp"、"rarpboot"、"tftpboot"、"loadb"、または "diskboot" などのコマンドのデフォルトロードアドレス

  loads_echo  - CONFIG_LOADS_ECHO を参照

  serverip    - TFTP サーバー IP アドレス; tftpboot コマンドに必要

  bootretry   - CONFIG_BOOT_RETRY_TIME を参照

  bootdelaykey - CONFIG_AUTOBOOT_DELAY_STR を参照

  bootstopkey - CONFIG_AUTOBOOT_STOP_STR を参照

  ethprime    - 最初にどのインターフェイスを使用するかを制御します。

  ethact      - 現在アクティブなインターフェイスを制御します。例えば、次のようにできます

                => setenv ethact FEC
                => ping 192.168.0.1 # トラフィックは FEC で送信されます
                => setenv ethact SCC
                => ping 10.0.0.1    # トラフィックは SCC で送信されます

  ethrotate   - "no" に設定すると、U-Boot は利用可能なすべてのネットワークインターフェイスを巡回しません。現在選択されているインターフェイスにとどまります。

  netretry    - "no" に設定すると、各ネットワーク操作はリトライなしで成功または失敗します。"once" に設定すると、利用可能なすべてのネットワークインターフェイスが一度試行されて成功しなかった場合にネットワーク操作は失敗します。リトライ操作を自分で制御するスクリプトに役立ちます。

  npe_ucode   - NPE マイクロコードのロードアドレスを設定します

  silent_linux  - 設定されている場合、コンソールを空に変更することで、Linux にサイレントにブートするように指示します。"yes" の場合はサイレントになります。"no" の場合はサイレントになりません。設定されていない場合は、U-Boot コンソールがサイレントであればサイレントになります。

  tftpsrcp    - これが設定されている場合、値は TFTP の UDP ソースポートに使用されます。

  tftpdstp    - これが設定されている場合、値は Well Know Port 69 の代わりに TFTP の UDP 宛先ポートに使用されます。

  tftpblocksize - TFTP 転送に使用するブロックサイズ。設定されていない場合は、TFTP サーバーのデフォルトのブロックサイズを使用します

  tftptimeout - TFTP パケットの再送信タイムアウト (ミリ秒、最小値は 1000 = 1 秒)。パケットが失われたと見なされ、再送信する必要がある時期を定義します。デフォルトは 5000 = 5 秒です。この値を小さくすると、パケット損失率が高いネットワークや信頼性の低い TFTP サーバーでのダウンロードがより速く成功する可能性があります。

  tftptimeoutcountmax - TFTP タイムアウトの最大回数 (単位なし、最小値 = 0)。単一のファイル転送中に、その転送が中止されるまでに発生できるタイムアウトの回数を定義します。デフォルトは 10 で、0 は「タイムアウトは許可されない」ことを意味します。この値を大きくすると、パケット損失率が高い場合や、信頼性の低い TFTP サーバーまたはクライアントハードウェアでのダウンロードが成功するのに役立つ場合があります。

  tftpwindowsize - これが設定されている場合、値は RFC 7440 で説明されている TFTP のウィンドウサイズに使用されます。これは、サーバーに ack を送信する前に受信できるブロック数を意味します。

  vlan        - 4095 未満の値に設定されている場合、イーサネット経由のトラフィックは 802.1q VLAN タグ付きフレームを介してカプセル化/受信されます。

  bootpretryperiod - BOOTP/DHCP がリトライを送信する期間。符号なし値、ミリ秒単位。設定されていない場合、期間はデフォルト (28000) または、定義されていれば CONFIG_NET_RETRY_COUNT に基づく値のいずれかになります。この値は CONFIG_NET_RETRY_COUNT に基づく値よりも優先されます。

  memmatches  - 最後の 'ms' コマンドで見つかった一致の数、16 進数

  memaddr     - 最後の 'ms' コマンドで見つかった最後の一致のアドレス、16 進数、または見つからなかった場合は 0

  mempos      - 最後の 'ms' コマンドで見つかった最後の一致のインデックス位置、検索のサイズ (.b, .w, .l) 単位

  zbootbase   - (x86 のみ) bzImage 'setup' ブロックのベースアドレス

  zbootaddr   - (x86 のみ) ロードされた bzImage のアドレス、通常は BZIMAGE_LOAD_ADDR であり 0x100000

以下のイメージロケーション変数は、ブートで使用されるイメージの場所を含みます。
"Image" 列はイメージの役割を示し、環境変数名ではありません。他の列は環境変数名です。
"File Name" は TFTP サーバー上のファイル名を示し、"RAM Address" はイメージがロードされる RAM 内の場所を示し、"Flash Location" は NOR フラッシュ内のイメージのアドレスまたは NAND フラッシュ内のオフセットを示します。
*注* - これらの変数はすべてのボードで定義する必要はありません。

一部のボードは現在、これらの目的のために他の変数を使用しており、一部のボードはこれらの変数を他の目的で使用しています。

Image           File Name        RAM Address       Flash Location

u-boot          u-boot           u-boot_addr_r     u-boot_addr
Linux kernel    bootfile         kernel_addr_r     kernel_addr
device tree blob fdtfile        fdt_addr_r        fdt_addr
ramdisk         ramdiskfile      ramdisk_addr_r    ramdisk_addr



以下の環境変数は、ネットワークブートコマンド ("bootp" および "rarpboot") によって使用され、ブートサーバーによって提供される情報に応じて自動的に更新される場合があります:

  bootfile    - 上記を参照
  dnsip       - ドメインネームサーバーの IP アドレス
  dnsip2      - セカンダリドメインネームサーバーの IP アドレス
  gatewayip   - 使用するゲートウェイ (ルーター) の IP アドレス
  hostname    - ターゲットホスト名
  ipaddr      - 上記を参照
  netmask     - サブネットマスク
  rootpath    - NFS サーバー上のルートファイルシステムのパス名
  serverip    - 上記を参照


2 つの特別な環境変数があります:

  serial#     - タイプ文字列やシリアル番号などのハードウェア識別情報を含む
  ethaddr     - イーサネットアドレス

これらの変数は一度だけ設定できます (通常はボードの製造時)。U-Boot は、一度設定されたこれらの変数を削除または上書きすることを拒否します。

さらに特別な環境変数:

  ver         - "version" コマンドで表示される U-Boot バージョン文字列を含む。この変数は読み取り専用です (CONFIG_VERSION_VARIABLE を参照)。


一部の設定パラメータへの変更は、次回のブート後にのみ有効になる場合があることに注意してください (はい、Windoze のようなものです :-)。

環境変数のコールバック関数:
---------------------------------------------

一部の環境変数については、値が変更されたときに u-boot の動作を変更する必要があります。この機能により、任意の変数に関数を関連付けることができます。作成、上書き、または削除時に、コールバックは何らかの副作用が発生する機会、または変更が拒否される機会を提供します。コールバックは名前が付けられ、ボードまたはドライバコード内の U_BOOT_ENV_CALLBACK マクロを使用して関数に関連付けられます。これらのコールバックは、次の 2 つの方法のいずれかで変数に関連付けられます。静的リストは、ボード設定で `CONFIG_ENV_CALLBACK_LIST_STATIC` を関連付けのリストを定義する文字列に定義することによって追加できます。リストは次の形式である必要があります:

    entry = variable_name[:callback_name]
    list = entry[,list]

コールバック名が指定されていない場合、コールバックは削除されます。リスト内のどこにでもスペースを使用することもできます。

コールバックは、".callbacks" 変数を上記の同じリスト形式で定義することによっても関連付けることができます。".callbacks" 内の関連付けは、静的リスト内の関連付けを上書きします。デフォルトまたは埋め込み環境で ".callbacks" 環境変数を定義するために、`CONFIG_ENV_CALLBACK_LIST_DEFAULT` をリスト (文字列) に定義できます。

`CONFIG_REGEX` が定義されている場合、上記の variable_name は正規表現として評価されます。これにより、すべてを明示的にリストすることなく、複数の変数を同じコールバックに接続できます。

コールバック関数のシグネチャは次のとおりです:

    int callback(const char *name, const char *value, enum env_op op, int flags)

* name - 変更された環境変数
* value - 環境変数の新しい値
* op - 操作 (作成、上書き、または削除)
* flags - 環境変数変更の属性、include/search.h のフラグ H_* を参照

戻り値は、変数の変更が受け入れられた場合は 0、それ以外の場合は 1 です。

冗長イーサネットインターフェイスに関する注意:
=======================================

一部のボードには冗長なイーサネットインターフェイスが付属しています。U-Boot はそのような構成をサポートし、必要に応じて「動作中の」インターフェイスを自動的に選択できます。MAC の割り当ては次のように機能します:

ネットワークインターフェイスには eth0, eth1, eth2, ... と番号が付けられます。対応する MAC アドレスは、環境に "ethaddr" (=>eth0), "eth1addr" (=>eth1), "eth2addr", ... として格納できます。

ネットワークインターフェイスが有効な MAC アドレス (例えば SROM 内) を格納している場合、環境に対応する設定がない場合はこれがデフォルトアドレスとして使用されます。対応する環境変数が設定されている場合、これはカードの設定を上書きします。つまり:

o SROM に有効な MAC アドレスがあり、環境にアドレスがない場合、SROM のアドレスが使用されます。
o SROM に有効なアドレスがなく、環境に定義が存在する場合、環境変数の値が使用されます。
o SROM と環境の両方に MAC アドレスが含まれ、両方のアドレスが同じ場合、この MAC アドレスが使用されます。
o SROM と環境の両方に MAC アドレスが含まれ、アドレスが異なる場合、環境の値が使用され、警告が表示されます。
o SROM にも環境にも MAC アドレスが含まれていない場合、エラーが発生します。`CONFIG_NET_RANDOM_ETHADDR` が定義されている場合、この場合、ランダムなローカル割り当て MAC が使用されます。

イーサネットドライバが 'write_hwaddr' 関数を実装している場合、有効な MAC アドレスは初期化プロセスの一部としてハードウェアにプログラムされます。これは、適切な 'ethmacskip' 環境変数を設定することによってスキップできます。命名規則は次のとおりです:
"ethmacskip" (=>eth0), "eth1macskip" (=>eth1) など。

イメージ形式:
==============

U-Boot は、次の 2 つの形式のイメージをブート (およびその他の補助操作を実行) できます:

新しい uImage 形式 (FIT)
-----------------------

Flattened Image Tree -- FIT (Flattened Device Tree に類似) に基づく柔軟で強力な形式。複数のコンポーネント (いくつかのカーネル、ramdisk など) を持つイメージの使用を可能にし、内容は SHA1、MD5、または CRC32 によって保護されます。詳細については、doc/uImage.FIT ディレクトリを参照してください。


古い uImage 形式
-----------------

古いイメージ形式は、基本的に何でもかまいませんが、特別なヘッダーが先行するバイナリファイルに基づいています。詳細については include/image.h の定義を参照してください。基本的に、ヘッダーは次のイメージプロパティを定義します:

* ターゲットオペレーティングシステム (OpenBSD, NetBSD, FreeBSD, 4.4BSD, Linux, SVR4, Esix, Solaris, Irix, SCO, Dell, NCR, VxWorks, LynxOS, pSOS, QNX, RTEMS, INTEGRITY 用の準備あり。現在サポートされているのは: Linux, NetBSD, VxWorks, QNX, RTEMS, LynxOS, INTEGRITY)。
* ターゲット CPU アーキテクチャ (Alpha, ARM, Intel x86, IA64, MIPS, NDS32, Nios II, PowerPC, IBM S390, SuperH, Sparc, Sparc 64 Bit 用の準備あり。現在サポートされているのは: ARM, Intel x86, MIPS, NDS32, Nios II, PowerPC)。
* 圧縮タイプ (非圧縮, gzip, bzip2)
* ロードアドレス
* エントリポイント
* イメージ名
* イメージタイムスタンプ

ヘッダーは特別なマジックナンバーでマークされ、イメージのヘッダー部分とデータ部分の両方が CRC32 チェックサムによって破損から保護されています。

Linux サポート:
==============

U-Boot はどの OS やスタンドアロンアプリケーションも簡単にサポートするはずですが、U-Boot の設計中の主な焦点は常に Linux にありました。U-Boot には、これまでのところ Linux カーネル内のいくつかの特別な「ブートローダー」コードの一部であった多くの機能が含まれています。また、使用される「initrd」イメージは、もはや 1 つの大きな Linux イメージの一部ではありません。代わりに、カーネルと「initrd」は別々のイメージです。この実装はいくつかの目的を果たします:

- 同じ機能を他の OS やスタンドアロンアプリケーションに使用できます (例えば: フラッシュメモリフットプリントを削減するために圧縮イメージを使用する)

- U-Boot によって多くの低レベル、ハードウェア依存の処理が行われるため、新しい Linux カーネルバージョンを移植するのがはるかに簡単になります

- 同じ Linux カーネルイメージを異なる「initrd」イメージで使用できるようになりました。もちろん、これは異なるカーネルイメージを同じ「initrd」で実行できることも意味します。これによりテストが容易になります (「initrd」内のファイルを変更しただけで新しい「zImage.initrd」Linux イメージをビルドする必要はありません)。また、ソフトウェアのフィールドアップグレードも容易になりました。

Linux HOWTO:
============

U-Boot ベースのシステムへの Linux の移植:
---------------------------------------

U-Boot は、ターゲットハードウェアで使用するために Linux デバイスドライバを設定するために必要なすべての変更を行うことからあなたを救うことはできません (いいえ、Linux に完全な仮想マシンインターフェイスを提供するつもりはありません :-)。しかし、今ではすべてのブートローダーコード (arch/powerpc/mbxboot 内) を無視できます。マシン固有のヘッダーファイル (例えば include/asm-ppc/tqm8xx.h) が include/asm-<arch>/u-boot.h で定義するのと同じボード情報構造体の定義を含んでいることを確認し、IMAP_ADDR の定義が CONFIG_SYS_IMMR の U-Boot 設定と同じ値を使用していることを確認してください。

U-Boot には現在、ドライバモデル、ドライバの統一モデルがあることに注意してください。新しいドライバを追加する場合は、ドライバモデルに組み込んでください。利用可能な uclass がない場合は、作成することをお勧めします。doc/driver-model を参照してください。


Linux カーネルの設定:
-----------------------------

U-Boot の特定の要件はありません。ターゲットシステム用に何らかのルートデバイス (初期 RAM ディスク、NFS) があることを確認してください。

Linux イメージのビルド:
-----------------------

U-Boot では、「zImage」や「bzImage」のような「通常の」ビルドターゲットは使用されません。最近のカーネルソースを使用する場合、新しいビルドターゲット「uImage」が存在し、U-Boot で使用可能なイメージを自動的にビルドします。ほとんどの古いカーネルには、「pImage」ターゲットのサポートもあります。これは、私たちの前身プロジェクト PPCBoot のために導入され、100% 互換性のある形式を使用します。

例:

    make TQM850L_defconfig
    make oldconfig
    make dep
    make uImage

「uImage」ビルドターゲットは、圧縮された Linux カーネルイメージをヘッダー情報、CRC32 チェックサムなどでカプセル化して U-Boot で使用するための特別なツール (「tools/mkimage」内) を使用します。私たちが行っていることは次のとおりです:

* 標準の「vmlinux」カーネルイメージをビルドします (ELF バイナリ形式):

* カーネルを raw バイナリイメージに変換します:

    `${CROSS_COMPILE}-objcopy -O binary \
                 -R .note -R .comment \
                 -S vmlinux linux.bin`

* バイナリイメージを圧縮します:

    `gzip -9 linux.bin`

* U-Boot 用に圧縮されたバイナリイメージをパッケージ化します:

    `mkimage -A ppc -O linux -T kernel -C gzip \
        -a 0 -e 0 -n "Linux Kernel Image" \
        -d linux.bin.gz uImage`


「mkimage」ツールは、U-Boot で使用するための ramdisk イメージを作成するためにも使用でき、Linux カーネルイメージから分離するか、1 つのファイルに結合します。「mkimage」は、ターゲットアーキテクチャ、オペレーティングシステム、イメージタイプ、圧縮方法、エントリポイント、タイムスタンプ、CRC32 チェックサムなどの情報を含む 64 バイトヘッダーでイメージをカプセル化します。

「mkimage」は 2 つの方法で呼び出すことができます: 既存のイメージを検証してヘッダー情報を表示するか、新しいイメージをビルドします。最初の形式 ( "-l" オプション付き) では、mkimage は既存の U-Boot イメージのヘッダーに含まれる情報をリストします。これにはチェックサム検証が含まれます:

    tools/mkimage -l image
      -l ==> イメージヘッダー情報をリスト表示

2 番目の形式 ( "-d" オプション付き) は、イメージペイロードとして使用される「データファイル」から U-Boot イメージをビルドするために使用されます:

    tools/mkimage -A arch -O os -T type -C comp -a addr -e ep \
                  -n name -d data_file image
      -A ==> アーキテクチャを 'arch' に設定
      -O ==> オペレーティングシステムを 'os' に設定
      -T ==> イメージタイプを 'type' に設定
      -C ==> 圧縮タイプを 'comp' に設定
      -a ==> ロードアドレスを 'addr' (16 進数) に設定
      -e ==> エントリポイントを 'ep' (16 進数) に設定
      -n ==> イメージ名を 'name' に設定
      -d ==> 'datafile' からイメージデータを使用

現在、PowerPC システム用のすべての Linux カーネルは同じロードアドレス (0x00000000) を使用しますが、エントリポイントアドレスはカーネルバージョンによって異なります:

- 2.2.x カーネルのエントリポイントは 0x0000000C です、
- 2.3.x 以降のカーネルのエントリポイントは 0x00000000 です。

したがって、U-Boot イメージをビルドするための典型的な呼び出しは次のようになります:

    -> tools/mkimage -n '2.4.4 kernel for TQM850L' \
    > -A ppc -O linux -T kernel -C gzip -a 0 -e 0 \
    > -d /opt/elsk/ppc_8xx/usr/src/linux-2.4.4/arch/powerpc/coffboot/vmlinux.gz \
    > examples/uImage.TQM850L
    Image Name:   2.4.4 kernel for TQM850L
    Created:      Wed Jul 19 02:34:59 2000
    Image Type:   PowerPC Linux Kernel Image (gzip compressed)
    Data Size:    335725 Bytes = 327.86 kB = 0.32 MB
    Load Address: 0x00000000
    Entry Point:  0x00000000

イメージの内容を検証する (または破損をチェックする) には:

    -> tools/mkimage -l examples/uImage.TQM850L
    Image Name:   2.4.4 kernel for TQM850L
    Created:      Wed Jul 19 02:34:59 2000
    Image Type:   PowerPC Linux Kernel Image (gzip compressed)
    Data Size:    335725 Bytes = 327.86 kB = 0.32 MB
    Load Address: 0x00000000
    Entry Point:  0x00000000

注: ブート時間が重要な組み込みシステムでは、速度とメモリをトレードオフして、代わりに非圧縮イメージをインストールできます。これにはフラッシュのスペースが多く必要ですが、展開する必要がないため、はるかに高速にブートします:

    -> gunzip /opt/elsk/ppc_8xx/usr/src/linux-2.4.4/arch/powerpc/coffboot/vmlinux.gz
    -> tools/mkimage -n '2.4.4 kernel for TQM850L' \
    > -A ppc -O linux -T kernel -C none -a 0 -e 0 \
    > -d /opt/elsk/ppc_8xx/usr/src/linux-2.4.4/arch/powerpc/coffboot/vmlinux \
    > examples/uImage.TQM850L-uncompressed
    Image Name:   2.4.4 kernel for TQM850L
    Created:      Wed Jul 19 02:34:59 2000
    Image Type:   PowerPC Linux Kernel Image (uncompressed)
    Data Size:    792160 Bytes = 773.59 kB = 0.76 MB
    Load Address: 0x00000000
    Entry Point:  0x00000000


同様に、カーネルが初期 RAM ディスクを使用することを意図している場合、「ramdisk.image.gz」ファイルから U-Boot イメージをビルドできます:

    -> tools/mkimage -n 'Simple Ramdisk Image' \
    > -A ppc -O linux -T ramdisk -C gzip \
    > -d /LinuxPPC/images/SIMPLE-ramdisk.image.gz examples/simple-initrd
    Image Name:   Simple Ramdisk Image
    Created:      Wed Jan 12 14:01:50 2000
    Image Type:   PowerPC Linux RAMDisk Image (gzip compressed)
    Data Size:    566530 Bytes = 553.25 kB = 0.54 MB
    Load Address: 0x00000000
    Entry Point:  0x00000000

「dumpimage」ツールは、mkimage によってビルドされたイメージの逆アセンブルまたは内容のリスト表示に使用できます。詳細については dumpimage のヘルプ出力 (-h) を参照してください。

Linux イメージのインストール:
-------------------------

シリアル (コンソール) インターフェース経由で U-Boot イメージをダウンロードするには、イメージを S-Record 形式に変換する必要があります:

    objcopy -I binary -O srec examples/image examples/image.srec

「objcopy」は U-Boot イメージヘッダー内の情報を理解しないため、結果の S-Record ファイルはアドレス 0x00000000 に対して相対的になります。特定のアドレスにロードするには、「loads」コマンドでターゲットアドレスを「offset」パラメータとして指定する必要があります。

例: イメージをアドレス 0x40100000 (TQM8xxL では最初のフラッシュバンク内) にインストールします:

    => erase 40100000 401FFFFF

    .......... done
    Erased 8 sectors

    => loads 40100000
    ## Ready for S-Record download ...
    ~>examples/image.srec
    1 2 3 4 5 6 7 8 9 10 11 12 13 ...
    ...
    15989 15990 15991 15992
    [file transfer complete]
    [connected]
    ## Start Addr = 0x00000000


「iminfo」コマンドを使用してダウンロードの成功を確認できます。これにはチェックサム検証が含まれているため、データ破損が発生していないことを確認できます:

    => imi 40100000

    ## Checking Image at 40100000 ...
       Image Name:     2.2.13 for initrd on TQM850L
       Image Type:     PowerPC Linux Kernel Image (gzip compressed)
       Data Size:      335725 Bytes = 327 kB = 0 MB
       Load Address: 00000000
       Entry Point:  0000000c
       Verifying Checksum ... OK


Linux のブート:
-----------

「bootm」コマンドは、メモリ (RAM またはフラッシュ) に格納されているアプリケーションをブートするために使用されます。Linux カーネルイメージの場合、「bootargs」環境変数の内容がパラメータとしてカーネルに渡されます。この変数は、「printenv」および「setenv」コマンドを使用して確認および変更できます:


    => printenv bootargs
    bootargs=root=/dev/ram

    => setenv bootargs root=/dev/nfs rw nfsroot=10.0.0.2:/LinuxPPC nfsaddrs=10.0.0.99:10.0.0.2

    => printenv bootargs
    bootargs=root=/dev/nfs rw nfsroot=10.0.0.2:/LinuxPPC nfsaddrs=10.0.0.99:10.0.0.2

    => bootm 40020000
    ## Booting Linux kernel at 40020000 ...
       Image Name:     2.2.13 for NFS on TQM850L
       Image Type:     PowerPC Linux Kernel Image (gzip compressed)
       Data Size:      381681 Bytes = 372 kB = 0 MB
       Load Address: 00000000
       Entry Point:  0000000c
       Verifying Checksum ... OK
       Uncompressing Kernel Image ... OK
    Linux version 2.2.13 (wd@denx.local.net) (gcc version 2.95.2 19991024 (release)) #1 Wed Jul 19 02:35:17 MEST 2000
    Boot arguments: root=/dev/nfs rw nfsroot=10.0.0.2:/LinuxPPC nfsaddrs=10.0.0.99:10.0.0.2
    time_init: decrementer frequency = 187500000/60
    Calibrating delay loop... 49.77 BogoMIPS
    Memory: 15208k available (700k kernel code, 444k data, 32k init) [c0000000,c1000000]
    ...

初期 RAM ディスクを使用して Linux カーネルをブートしたい場合は、カーネルと initrd イメージ (PPBCOOT 形式!) の両方のメモリアドレスを「bootm」コマンドに渡します:

    => imi 40100000 40200000

    ## Checking Image at 40100000 ...
       Image Name:     2.2.13 for initrd on TQM850L
       Image Type:     PowerPC Linux Kernel Image (gzip compressed)
       Data Size:      335725 Bytes = 327 kB = 0 MB
       Load Address: 00000000
       Entry Point:    0000000c
       Verifying Checksum ... OK

    ## Checking Image at 40200000 ...
       Image Name:     Simple Ramdisk Image
       Image Type:     PowerPC Linux RAMDisk Image (gzip compressed)
       Data Size:      566530 Bytes = 553 kB = 0 MB
       Load Address: 00000000
       Entry Point:    00000000
       Verifying Checksum ... OK

    => bootm 40100000 40200000
    ## Booting Linux kernel at 40100000 ...
       Image Name:     2.2.13 for initrd on TQM850L
       Image Type:     PowerPC Linux Kernel Image (gzip compressed)
       Data Size:      335725 Bytes = 327 kB = 0 MB
       Load Address: 00000000
       Entry Point:    0000000c
       Verifying Checksum ... OK
       Uncompressing Kernel Image ... OK
    ## Loading RAMDisk Image at 40200000 ...
       Image Name:     Simple Ramdisk Image
       Image Type:     PowerPC Linux RAMDisk Image (gzip compressed)
       Data Size:      566530 Bytes = 553 kB = 0 MB
       Load Address: 00000000
       Entry Point:    00000000
       Verifying Checksum ... OK
       Loading Ramdisk ... OK
    Linux version 2.2.13 (wd@denx.local.net) (gcc version 2.95.2 19991024 (release)) #1 Wed Jul 19 02:32:08 MEST 2000
    Boot arguments: root=/dev/ram
    time_init: decrementer frequency = 187500000/60
    Calibrating delay loop... 49.77 BogoMIPS
    ...
    RAMDISK: Compressed image found at block 0
    VFS: Mounted root (ext2 filesystem).
    bash#

フラットデバイスツリーを渡して Linux をブート:
-----------

まず、U-Boot を適切な define でコンパイルする必要があります。詳細については、上記の「Linux カーネルインターフェイス」セクションを参照してください。以下は、カーネルを起動し、更新されたフラットデバイスツリーを渡す方法の例です:

    => print oftaddr
    oftaddr=0x300000
    => print oft
    oft=oftrees/mpc8540ads.dtb
    => tftp $oftaddr $oft
    Speed: 1000, full duplex
    Using TSEC0 device
    TFTP from server 192.168.1.1; our IP address is 192.168.1.101
    Filename 'oftrees/mpc8540ads.dtb'.
    Load address: 0x300000
    Loading: #
    done
    Bytes transferred = 4106 (100a hex)
    => tftp $loadaddr $bootfile
    Speed: 1000, full duplex
    Using TSEC0 device
    TFTP from server 192.168.1.1; our IP address is 192.168.1.2
    Filename 'uImage'.
    Load address: 0x200000
    Loading:############
    done
    Bytes transferred = 1029407 (fb51f hex)
    => print loadaddr
    loadaddr=200000
    => print oftaddr
    oftaddr=0x300000
    => bootm $loadaddr - $oftaddr
    ## Booting image at 00200000 ...
       Image Name:   Linux-2.6.17-dirty
       Image Type:   PowerPC Linux Kernel Image (gzip compressed)
       Data Size:    1029343 Bytes = 1005.2 kB
       Load Address: 00000000
       Entry Point:  00000000
       Verifying Checksum ... OK
       Uncompressing Kernel Image ... OK
    Booting using flat device tree at 0x300000
    Using MPC85xx ADS machine description
    Memory CAM mapping: CAM0=256Mb, CAM1=256Mb, CAM2=0Mb residual: 0Mb
    [snip]


U-Boot イメージタイプの詳細:
------------------------------

U-Boot は以下のイメージタイプをサポートしています:

   "Standalone Programs" は U-Boot によって提供される環境で直接実行可能です。それらが (行儀よく) 動作する場合、Standalone Program から戻った後も U-Boot で作業を続けることができると期待されます。

   "OS Kernel Images" は通常、完全に制御を引き継ぐ組み込み OS のイメージです。通常、これらのプログラムは独自の例外ハンドラ、デバイスドライバをインストールし、MMU を設定するなどを行います - これは、CPU をリセットすることによってのみ U-Boot に再入できることを意味します。

   "RAMDisk Images" は多かれ少なかれ単なるデータブロックであり、それらのパラメータ (アドレス、サイズ) は開始される OS カーネルに渡されます。

   "Multi-File Images" は、通常 OS (Linux) カーネルイメージと 1 つ以上のデータイメージ (RAMDisks など) の複数のイメージを含みます。この構造は、例えば BOOTP などを使用してネットワーク経由でブートする場合に役立ちます。ブートサーバーは単一のイメージファイルのみを提供しますが、例えば OS カーネルと RAMDisk イメージを取得したい場合などです。

   "Multi-File Images" はイメージサイズのリストで始まり、各イメージサイズ (バイト単位) はネットワークバイトオーダーの "uint32_t" で指定されます。このリストは "(uint32_t)0" で終端されます。終端の 0 の直後に、イメージが 1 つずつ続き、すべて "uint32_t" 境界にアラインされます (サイズは 4 バイトの倍数に切り上げられます)。

   "Firmware Images" は、通常フラッシュメモリにプログラムされるファームウェア (U-Boot や FPGA イメージなど) を含むバイナリイメージです。

   "Script files" は U-Boot のコマンドインタプリタによって実行されるコマンドシーケンスです。この機能は、コマンドインタプリタとして実際のシェル (hush) を使用するように U-Boot を設定する場合に特に役立ちます。

Linux zImage のブート:
-------------------------

一部のプラットフォームでは、Linux zImage をブートすることが可能です。これは「bootz」コマンドを使用して行われます。"bootz" コマンドの構文は "bootm" コマンドの構文と同じです。`CONFIG_SUPPORT_RAW_INITRD` を定義すると、ユーザーはカーネルに raw initrd イメージを提供できます。構文はわずかに異なり、initrd のアドレスにはそのサイズを次の形式で追加する必要があります: "<initrd アドレス>:<initrd サイズ>"。

スタンドアロン HOWTO:
=================

U-Boot の特徴の 1 つは、「スタンドアロン」アプリケーションを動的にロードして実行できることです。これらは、コンソール I/O 関数や割り込みサービスなど、U-Boot の一部のリソースを使用できます。2 つの簡単な例がソースに含まれています:

"Hello World" デモ:
-------------------

'examples/hello_world.c' には、小さな "Hello World" デモアプリケーションが含まれています。これは U-Boot をビルドするときに自動的にコンパイルされます。アドレス 0x00040004 で実行するように設定されているので、次のように試すことができます:

    => loads
    ## Ready for S-Record download ...
    ~>examples/hello_world.srec
    1 2 3 4 5 6 7 8 9 10 11 ...
    [file transfer complete]
    [connected]
    ## Start Addr = 0x00040004

    => go 40004 Hello World! This is a test.
    ## Starting application at 0x00040004 ...
    Hello World
    argc = 7
    argv[0] = "40004"
    argv[1] = "Hello"
    argv[2] = "World!"
    argv[3] = "This"
    argv[4] = "is"
    argv[5] = "a"
    argv[6] = "test."
    argv[7] = "<NULL>"
    Hit any key to exit ...

    ## Application terminated, rc = 0x0

U-Boot コードで CPM 割り込みハンドラを登録する方法を示す別の例は、'examples/timer.c' にあります。ここでは、CPM タイマーが 1 秒ごとに割り込みを生成するように設定されています。割り込みサービスルーチンは些細なもので、'.' 文字を表示するだけですが、これは単なるデモプログラムです。アプリケーションは次のキーで制御できます:

    ? - CPM タイマーレジスタの現在の値を表示
    b - 割り込みを有効にしてタイマーを開始
    e - タイマーを停止して割り込みを無効化
    q - アプリケーションを終了

    => loads
    ## Ready for S-Record download ...
    ~>examples/timer.srec
    1 2 3 4 5 6 7 8 9 10 11 ...
    [file transfer complete]
    [connected]
    ## Start Addr = 0x00040004

    => go 40004
    ## Starting application at 0x00040004 ...
    TIMERS=0xfff00980
    Using timer 1
      tgcr @ 0xfff00980, tmr @ 0xfff00990, trr @ 0xfff00994, tcr @ 0xfff00998, tcn @ 0xfff0099c, ter @ 0xfff009b0

'b' を押す:
    [q, b, e, ?] Set interval 1000000 us
    Enabling timer
'?' を押す:
    [q, b, e, ?] ........
    tgcr=0x1, tmr=0xff1c, trr=0x3d09, tcr=0x0, tcn=0xef6, ter=0x0
'?' を押す:
    [q, b, e, ?] .
    tgcr=0x1, tmr=0xff1c, trr=0x3d09, tcr=0x0, tcn=0x2ad4, ter=0x0
'?' を押す:
    [q, b, e, ?] .
    tgcr=0x1, tmr=0xff1c, trr=0x3d09, tcr=0x0, tcn=0x1efc, ter=0x0
'?' を押す:
    [q, b, e, ?] .
    tgcr=0x1, tmr=0xff1c, trr=0x3d09, tcr=0x0, tcn=0x169d, ter=0x0
'e' を押す:
    [q, b, e, ?] ...Stopping timer
'q' を押す:
    [q, b, e, ?] ## Application terminated, rc = 0x0


Minicom に関する警告:
================

時間が経つにつれて、多くの人々がシリアルダウンロードに "minicom" 端末エミュレーションプログラムを使用しようとするときに問題を報告しています。私 (wd) は minicom が壊れていると考えており、使用しないことをお勧めします。Unix では、汎用用途 (特に kermit バイナリプロトコルダウンロード ("loadb" コマンド) には C-Kermit を使用し、S-Record ダウンロード ("loads" コマンド) には "cu" を使用することをお勧めします。kermit のヘルプについては https://www.denx.de/wiki/view/DULG/SystemSetup#Section_4.3 を参照してください。


それにもかかわらず、どうしてもそれを使用したい場合は、「ファイル転送プロトコル」セクションにこの設定を追加してみてください:

       Name      Program                         Name U/D FullScr IO-Red. Multi
    X  kermit  /usr/bin/kermit -i -l %l -s     Y    U    Y       N      N
    Y  kermit  /usr/bin/kermit -i -l %l -r     N    D    Y       N      N


NetBSD に関する注意:
=============

バージョン 0.9.2 から、U-Boot はホスト (U-Boot をビルド) とターゲットシステム (NetBSD/mpc8xx をブート) の両方として NetBSD をサポートします。

ビルドにはクロス環境が必要です。NetBSD/i386 上で cross-powerpc-netbsd-1.3 パッケージで動作することが知られています (Makefile は BSD make と互換性がないため、gmake も必要になります)。cross-powerpc パッケージはインクルードファイルをインストールしないことに注意してください。<machine/ansi.h> が見つからないため、U-Boot のビルドは失敗します。このファイルは手動でインストールしてパッチを適用する必要があります:

    # cd /usr/pkg/cross/powerpc-netbsd/include
    # mkdir powerpc
    # ln -s powerpc machine
    # cp /usr/src/sys/arch/powerpc/include/ansi.h powerpc/ansi.h
    # ${EDIT} powerpc/ansi.h    ## __va_list, _BSD_VA_LIST を削除する必要がある

ネイティブビルドは、ネイティブと U-Boot のインクルードファイルの非互換性のため動作しません。

ブートは、ブートされるイメージ (の最初の部分) がステージ 2 ローダーであり、それが次にカーネル本体をロードして呼び出すことを前提としています。ローダーソースは最終的に NetBSD ソースツリー (おそらく sys/arc/mpc8xx/stand/u-boot_stage2/ 内) に現れます。それまでの間は、ftp://ftp.denx.de/pub/u-boot/ppcboot_stage2.tar.gz を参照してください。


実装内部:
=========================

以下は、すべての実装の詳細を完全に説明することを意図したものではありません。ただし、U-Boot の内部動作を理解し、カスタムハードウェアへの移植を容易にするのに役立つはずです。

初期スタック、グローバルデータ:
---------------------------

U-Boot の実装は、U-Boot が通常、システム RAM へのアクセスなしに ROM (フラッシュメモリ) から実行を開始するという事実によって複雑になります (メモリコントローラがまだ初期化されていないため)。これは、書き込み可能なデータまたは BSS セグメントがなく、BSS がゼロとして初期化されていないことを意味します。C 環境をまったく動作させるためには、少なくとも最小限のスタックを割り当てる必要があります。これに対する実装オプションは、使用される CPU によって定義され、制限されます: 一部の CPU モデルはオンチップメモリ (MPC8xx および MPC826x プロセッサ上の IMMR 領域など) を提供し、他のモデルではデータキャッシュ (の一部) をロックしてメモリとして (誤って) 使用できます。

    Chris Hallinan は、これらの問題の良い要約を U-Boot メーリングリストに投稿しました:

    件名: RE: [U-Boot-Users] RE: More On Memory Bank x (nothingness)?
    差出人: "Chris Hallinan" <clh@net1plus.com>
    日付: Mon, 10 Feb 2003 16:43:46 -0500 (22:43 MET)
    ...

    皆さん、もし間違っていたら訂正してください。私の理解では、スタックなどの初期 RAM として DCACHE を使用することは、キャッシュをバックアップする物理 RAM を必要としません。巧妙なのは、SDRAM コントローラがセットアップされる前に必要なストレージの一時的な供給源としてキャッシュが使用されていることです。詳細を説明するのはこのリストの範囲を超えていますが、アーキテクチャおよびプロセッサ固有のマニュアルでキャッシュアーキテクチャと操作を研究することによって、これがどのように機能するかがわかります。

    OCM はオンチップメモリであり、405GP には 4K あると思います。これは、SDRAM が利用可能になる前に、システム設計者が初期スタック/RAM 領域として使用するための別のオプションです。どちらのオプションも機能するはずです。CS 4 を使用することは、ボード設計者が初期ブート中に問題を引き起こすようなものに使用していなければ問題ありません！ しばしば使用されません。

    CONFIG_SYS_INIT_RAM_ADDR は、プロセッサ/ボード/システム設計を妨げないどこかに設定する必要があります。最近の u-boot ディストリビューションの walnut.h にあるデフォルト値は機能するはずです。SDRAM モジュールよりも大きな値に設定することをお勧めします。64MB の SDRAM モジュールがある場合は、400_0000 より上に設定してください。ボードにそのアドレスに応答することになっているリソースがないことを確認してください！ start.S のそのコードはしばらく前から存在しており、設定が正しくなればそのまま動作するはずです。
    -Chris Hallinan
    DS4.COM, Inc.

初期化手順の C コードにいくつかの影響があるため、これを覚えておくことが不可欠です:

* 初期化されたグローバルデータ (データセグメント) は読み取り専用です。書き込もうとしないでください。

* 初期化されていないグローバルデータ (または暗黙的にゼロデータとして初期化されたもの - BSS セグメント) はまったく使用しないでください - これは未定義であり、初期化は後で (RAM へのリロケーション時) 実行されます。

* スタックスペースは非常に限られています。大きなデータバッファなどは避けてください。

書き込み可能なメモリとしてスタックしかないということは、コード間で情報を共有するために通常のグローバルデータを使用できないことを意味します。しかし、グローバルデータ構造体 (gd_t) をすべての関数で利用可能にすることで、U-Boot の実装を大幅に簡素化できることがわかりました。このデータへのポインタを *すべて* の関数に引数として渡すこともできますが、これによりコードが肥大化します。代わりに、データを共有するために GCC コンパイラの機能 (グローバルレジスタ変数) を使用します: この目的のために予約したレジスタに、グローバルデータへのポインタ (gd) を配置します。そのような目的でレジスタを選択する場合、現在のアーキテクチャの関連する (E)ABI 仕様と GCC の実装によって制限されます。

PowerPC の場合、次のレジスタには特定の用途があります:
    R1:     スタックポインタ
    R2:     システム使用のために予約
    R3-R4:  パラメータ渡しと戻り値
    R5-R10: パラメータ渡し
    R13:    スモールデータエリアポインタ
    R30:    GOT ポインタ
    R31:    フレームポインタ

    (U-Boot は内部 GOT ポインタとして R12 も使用します。r12 は揮発性レジスタであるため、アセンブリと C の間を行き来するときに r12 をリセットする必要があります)

    ==> U-Boot は R2 を使用してグローバルデータへのポインタを保持します

    注: PPC では、静的イニシャライザを使用することもできます (グローバルデータ構造体のアドレスはコンパイル時に既知であるため)。しかし、レジスタを予約すると、コードが多少小さくなることがわかりました - コードの節約はそれほど大きくはありませんが (すべてのボードで平均して U-Boot イメージ全体で 752 バイト、624 テキスト + 127 データ)。

ARM の場合、次のレジスタが使用されます:

    R0:     関数引数ワード/整数結果
    R1-R3:  関数引数ワード
    R9:     プラットフォーム固有
    R10:    スタック制限 (スタックチェックが有効な場合のみ使用)
    R11:    引数 (フレーム) ポインタ
    R12:    一時的なワークスペース
    R13:    スタックポインタ
    R14:    リンクレジスタ
    R15:    プログラムカウンタ

    ==> U-Boot は R9 を使用してグローバルデータへのポインタを保持します

    注: ARM では、R_ARM_RELATIVE リロケーションのみがサポートされています。

Nios II の場合、ABI はここで文書化されています:
    https://www.altera.com/literature/hb/nios2/n2cpu_nii51016.pdf

    ==> U-Boot は gp を使用してグローバルデータへのポインタを保持します

    注: Nios II では、gcc に "-G0" オプションを与え、スモールデータセクションにアクセスするために gp を使用しないため、gp は自由です。

NDS32 の場合、次のレジスタが使用されます:

    R0-R1:  引数/戻り値
    R2-R5:  引数
    R15:    アセンブラ用の一時レジスタ
    R16:    トランポリンレジスタ
    R28:    フレームポインタ (FP)
    R29:    グローバルポインタ (GP)
    R30:    リンクレジスタ (LP)
    R31:    スタックポインタ (SP)
    PC:     プログラムカウンタ (PC)

    ==> U-Boot は R10 を使用してグローバルデータへのポインタを保持します

注: DECLARE_GLOBAL_DATA_PTR はファイルグローバルスコープで使用する必要があります。そうしないと、現在のバージョンの GCC がコードを過度に「最適化」する可能性があります。

RISC-V の場合、次のレジスタが使用されます:

    x0: ハードワイヤードゼロ (zero)
    x1: 戻りアドレス (ra)
    x2: スタックポインタ (sp)
    x3: グローバルポインタ (gp)
    x4: スレッドポインタ (tp)
    x5: リンクレジスタ (t0)
    x8: フレームポインタ (fp)
    x10-x11: 引数/戻り値 (a0-1)
    x12-x17: 引数 (a2-7)
    x28-31: 一時レジスタ (t3-6)
    pc: プログラムカウンタ (pc)

    ==> U-Boot は gp を使用してグローバルデータへのポインタを保持します

メモリ管理:
------------------

U-Boot はシステム状態で実行され、物理アドレスを使用します。つまり、MMU はアドレスマッピングにもメモリ保護にも使用されません。利用可能なメモリは、メモリコントローラを使用して固定アドレスにマップされます。このプロセスでは、いくつかの物理メモリバンクで構成されている場合でも、メモリタイプごと (フラッシュ、SDRAM、SRAM) に連続したブロックが形成されます。

U-Boot は最初のフラッシュバンクの最初の 128 kB にインストールされます (TQM8xxL モジュールでは、これは 0x40000000 ... 0x4001FFFF の範囲です)。ブート後、DRAM のサイズ設定と初期化の後、コードは DRAM の上端に自身を再配置します。U-Boot コードのすぐ下に、malloc() によって使用されるためのメモリが予約されます [CONFIG_SYS_MALLOC_LEN 設定設定を参照]。その下に、グローバルボード情報データを持つ構造体が配置され、その後にスタックが続きます (下向きに成長)。さらに、いくつかの例外ハンドラコードが DRAM の下位 8 kB (0x00000000 ... 0x00001FFF) にコピーされます。

したがって、16 MB の DRAM を持つ典型的なメモリ構成は次のようになります:

    0x0000 0000     例外ベクタコード
          :
    0x0000 1FFF
    0x0000 2000     アプリケーション使用のために空き
          :
          :

          :
          :
    0x00FB FF20     モニター スタック (下向きに成長)
    0x00FB FFAC     ボード情報データとグローバルデータの永続コピー
    0x00FC 0000     Malloc アリーナ
          :
    0x00FD FFFF
    0x00FE 0000     モニターコードの RAM コピー
    ...             場合によっては: LCD またはビデオフレームバッファ
    ...             場合によっては: pRAM (保護 RAM - リセットによって変更されない)
    0x00FF FFFF     [RAM の終わり]


システム初期化:
----------------------

リセット構成では、U-Boot はリセットエントリポイントで開始します (ほとんどの PowerPC システムではアドレス 0x00000100)。CS0# のリセット構成のため、これはオンボードフラッシュメモリのミラーです。メモリを再マップできるようにするために、U-Boot はそのリンクアドレスにジャンプします。初期化コードを C で実装できるようにするために、(小さな!) 初期スタックが内部デュアルポート RAM (そのような機能を提供する CPU の場合など)、またはデータキャッシュのロックされた部分に設定されます。その後、U-Boot は CPU コア、キャッシュ、および SIU を初期化します。

次に、(潜在的に) 利用可能なすべてのメモリバンクが予備マッピングを使用してマップされます。例えば、それらを 512 MB 境界 (0x20000000 の倍数: SDRAM は 0x00000000 と 0x20000000、フラッシュは 0x40000000 と 0x60000000、SRAM は 0x80000000) に配置します。次に、SDRAM アクセス用に UPM A がプログラムされます。一時的な構成を使用して、SDRAM バンクのサイズを決定する簡単なメモリテストが実行されます。複数の SDRAM バンクがあり、バンクのサイズが異なる場合、最大のものが最初にマップされます。サイズが等しい場合、最初のバンク (CS2#) が最初にマップされます。最初のマッピングは常にアドレス 0x00000000 用であり、追加のバンクは 0 から始まる連続したメモリを作成するために直後に続きます。

次に、モニターは SDRAM 領域の上端に自身をインストールし、malloc() による使用およびグローバルボード情報データ用のメモリを割り当てます。また、例外ベクタコードが下位 RAM ページにコピーされ、最終的なスタックが設定されます。

U-Boot 移植ガイド:
----------------------

[U-Boot-Users メーリングリストの Jerry Van Baren によるメッセージに基づく、2002 年 10 月]


```c
int main(int argc, char *argv[])
{
    sighandler_t no_more_time;
    signal(SIGALRM, no_more_time);
    alarm(PROJECT_DEADLINE - toSec (3 * WEEK));

    if (available_money > available_manpower) {
        Pay consultant to port U-Boot;
        return 0;
    }

    Download latest U-Boot source;

    Subscribe to u-boot mailing list;

    if (clueless)
        email("Hi, I am new to U-Boot, how do I get started?");
    while (learning) {
        Read the README file in the top level directory;
        Read [https://www.denx.de/wiki/bin/view/DULG/Manual](https://www.denx.de/wiki/bin/view/DULG/Manual);
        Read applicable doc/README.*;
        Read the source, Luke;
        /* find . -name "*.[chS]" | xargs grep -i <keyword> */
    }

    if (available_money > toLocalCurrency ($2500))
        Buy a BDI3000;
    else
        Add a lot of aggravation and time;

    if (a similar board exists) {   /* hopefully... */
        cp -a board/<similar> board/<myboard>
        cp include/configs/<similar>.h include/configs/<myboard>.h
    } else {
        Create your own board support subdirectory;
        Create your own board include/configs/<myboard>.h file;
    }
    Edit new board/<myboard> files
    Edit new include/configs/<myboard>.h

    while (!accepted) {
        while (!running) {
            do {
                Add / modify source code;
            } until (compiles);
            Debug;
            if (clueless)
                email("Hi, I am having problems...");
        }
        Send patch file to the U-Boot email list;
        if (reasonable critiques)
            Incorporate improvements from email list code review;
        else
            Defend code as written;
    }

    return 0;
}

void no_more_time (int sig)
{
      hire_a_guru();
}

コーディング標準:
----------------------

U-Boot へのすべての貢献は、Linux カーネルのコーディングスタイルに準拠する必要があります。
カーネルコーディングスタイルガイド (https://www.kernel.org/doc/html/latest/process/coding-style.html) および Linux カーネルソースディレクトリ内のスクリプト "scripts/Lindent" を参照してください 。   

異なるプロジェクト (例えば MTD サブシステム) から派生したソースファイルは、通常これらのガイドラインから免除され、それらのソースの新しいバージョンへの後続の移行を容易にするために再フォーマットされません 。   

U-Boot は C (および一部はアセンブラ) で実装されていることに注意してください。C++ は使用されていないため、コード内で C++ スタイルのコメント (//) を使用しないでください 。   

また、次のフォーマット規則に従ってください:   

末尾の空白を削除する    
インデントと垂直方向の整列にはスペースではなく TAB 文字を使用する    
DOS の '\r\n' 改行を使用しないようにする    
ソースファイルに 2 行を超える連続した空行を追加しない    
ソースファイルの末尾に空行を追加しない    
標準に準拠しない提出物は、変更の再フォーマットを要求して返却される場合があります 。   

パッチの提出:
----------------------

U-Boot のパッチ数が増加しているため、いくつかのルールを確立する必要があります 。これらのルールに準拠しない提出物は、重要で価値のあるものが含まれていても拒否される場合があります 。詳細については https://www.google.com/search?q=https://www.denx.de/wiki/U-Boot/Patches を参照してください 。   

パッチは u-boot メーリングリスト [メールアドレスを削除しました] に送信する必要があります。https://lists.denx.de/listinfo/u-boot を参照してください 。   

パッチを送信する際には、次の情報を含めてください:

バグ修正の場合: バグの説明とパッチがこのバグをどのように修正するか。パッチが実際に何かを修正することを示す方法を含めるようにしてください 。   
新機能の場合: 機能とその実装の説明 。   
主要な貢献の場合、あなたの情報と関連するファイルおよびディレクトリ参照を含む MAINTAINERS ファイルを追加してください 。   
新しいボードのサポートを追加する場合、boards.cfg ファイルにもメンテナーの電子メールアドレスを追加することを忘れないでください 。   
パッチが新しい設定オプションを追加する場合、README ファイルでこれらを文書化することを忘れないでください 。   
パッチ自体。git (強く推奨されます) を使用している場合は、「git format-patch」を使用して簡単にパッチを生成できます 。その後、「git send-email」を使用して U-Boot メーリングリストに送信すると、他のメールクライアントに関する一般的な問題のほとんどを回避できます 。git を使用できない場合は、「diff -purN OLD NEW」を使用してください 。diff のバージョンがこれらのオプションをサポートしていない場合は、GNU diff の最新バージョンを入手してください 。このコマンドを実行するときのカレントディレクトリは、U-Boot ソースツリーの親ディレクトリである必要があります (つまり、パッチが影響を受けるファイルに十分なディレクトリ情報を含んでいることを確認してください) 。   
プレーンテキストとしてのパッチを推奨します。MIME 添付ファイルは推奨されず、圧縮された添付ファイルは使用してはなりません 。   
1 つの論理的な変更セットが複数のファイルに影響を与えるか作成する場合、これらの変更はすべて単一のパッチファイルで提出する必要があります 。   
異なる、関連性のない変更を含む変更セットは、変更セットごとに 1 つのパッチとして、個別のパッチとして提出する必要があります 。   
注:   

パッチを送信する前に、パッチを適用したソースツリーで buildman スクリプトを実行し、どのボードに対してもエラーや警告が報告されないことを確認してください 。   
変更は必要最小限に抑えてください: いくつかの関連性のない変更や任意の再フォーマットを含むパッチは、再フォーマット/分割の要求とともに返却されます 。   
既存のコードを変更する場合、新しいコードがコードのメモリフットプリントを増やさないようにしてください ;-) Small is beautiful! 新機能を追加する場合、これらは条件付きでのみコンパイルされるべきであり (#ifdef を使用)、新機能が無効になっている結果のコードは、変更なしの古いコードよりも多くのメモリを必要としてはなりません 。   
u-boot メーリングリストにはメッセージごとに 100 kB のサイズ制限があることを忘れないでください 。大きなパッチはモデレートされます 。合理的で大きすぎなければ承認されます 。しかし、サイズ制限を超えるパッチは避けるべきです 。   

<!-- end list -->