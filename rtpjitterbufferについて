GStreamer の `rtpjitterbuffer` エレメントは、ネットワーク経由で受信される RTP (Real-time Transport Protocol) パケットの到着時刻の揺らぎ（ジッター）、順序の入れ替わり、重複、ロスといった問題を緩和し、スムーズなメディア再生を実現するための重要なコンポーネントです。

提供されたソースコード (`gstrtpjitterbuffer.c`/`h` および `rtpjitterbuffer.c`/`h`) に基づいて、その仕組みと機能を解説します。

### `rtpjitterbuffer` エレメントの構造

このエレメントは、2つの主要な部分から構成されています。

1.  **`GstRtpJitterBuffer` (`gstrtpjitterbuffer.c`/`h`)**:
    * GStreamer のエレメントとしてのラッパー部分です。`GstElement` を継承し、プロパティ、シグナル、パッド（`sink` と `src`）の管理、状態変化への対応、イベント処理など、GStreamerフレームワークとの連携を担当します。
    * 内部にコアとなるジッターバッファライブラリのインスタンス (`priv->jbuf`) を保持します。

2.  **`RTPJitterBuffer` (`rtpjitterbuffer.c`/`h`)**:
    * 実際のジッターバッファリング、パケットの並べ替え、タイムスタンプ処理、同期ロジックなどを実装するコアライブラリです。
    * `GstRtpJitterBuffer` エレメントから独立して使用することも可能なように設計されている汎用的なジッターバッファです。

### 主な目的と機能

`rtpjitterbuffer` の主な目的は、不安定なネットワーク環境でも高品質なRTPストリーミングを実現することです。そのために以下の機能を提供します。

1.  **ジッター吸収 (Jitter Absorption)**:
    * 受信したRTPパケットを一定期間内部のバッファに保持することで、ネットワーク遅延の変動（ジッター）を吸収します。これにより、パケットの到着間隔が平滑化され、再生がスムーズになります。
    * バッファリングする時間は `latency` プロパティで設定できます。

2.  **パケットの並べ替え (Reordering)**:
    * RTPパケットはネットワーク経路の違いなどにより、送信された順序とは異なる順序で到着することがあります。
    * `rtpjitterbuffer` は、各RTPパケットのヘッダに含まれるシーケンス番号（Sequence Number）を監視し、正しい順序に並べ替えてから下流のエレメントに渡します。

3.  **重複パケットの除去 (Duplicate Removal)**:
    * 同じシーケンス番号を持つパケットが複数到着した場合、重複とみなして破棄します。

4.  **タイムスタンプに基づくスケジューリングと同期**:
    * RTPパケットのRTPタイムスタンプと、RTCP Sender Report (SR) パケットに含まれるNTPタイムスタンプおよび対応するRTPタイムスタンプのマッピング情報を使用して、各パケットが再生されるべき「絶対時刻」を計算します。
    * 計算された時刻になるまでパケットの送出を遅延させ、送信側のクロックレートと受信側の再生クロックを同期させようとします。これにより、長期間の再生でも音ズレや映像の早送り/スローダウンを防ぎます。
    * この機能は主に `mode` プロパティが `GST_RTP_JITTER_BUFFER_MODE_SLAVE` (デフォルト) の場合に有効です。

5.  **パケットロス処理 (Packet Loss Handling)**:
    * シーケンス番号の抜けを検出することでパケットロスを認識します。
    * `do-lost` プロパティが `TRUE` の場合、失われたパケットの代わりに「lost」とマークされた空のバッファを下流に送出し、下流のエレメント（例: デコーダ）がロスを認識してエラーコンシールメント処理などを行えるようにします。

### `GstRtpJitterBuffer` エレメントの動作

1.  **初期化 (`gst_rtp_jitter_buffer_class_init`, `gst_rtp_jitter_buffer_init`)**:
    * プロパティ（後述）やパッドテンプレートなどを定義します。
    * 内部で `RTPJitterBuffer` オブジェクト (`priv->jbuf`) を `rtp_jitter_buffer_new()` で作成します。

2.  **パケット受信 (シンクパッドのチェーン関数: `gst_rtp_jitter_buffer_chain`)**:
    * 上流からRTPパケット (`GstBuffer`) を受信すると、まず `gst_rtp_buffer_map()` を使ってRTPパケット情報を解析します（シーケンス番号、タイムスタンプ、SSRCなど）。
    * 解析した情報と共にパケットを `rtp_jitter_buffer_add(priv->jbuf, buffer, seqnum, timestamp, ...)` 関数を使って内部の `RTPJitterBuffer` に追加します。

3.  **イベント処理**:
    * **シンクパッド (`gst_rtp_jitter_buffer_sink_event`)**:
        * 特に `GST_EVENT_RTCP_BUFFER` をリッスンし、RTCP SR (Sender Report) パケットを受信すると、その情報を `rtp_jitter_buffer_set_syncinfo(priv->jbuf, ssrc, clock_rate, sr_ntptime, sr_rtptime, ...)` に渡して、内部の同期情報を更新します。これにより、RTPタイムスタンプとNTP実時間とのマッピング精度が向上します。
        * `GST_EVENT_CAPS` を受信すると、ペイロードタイプからクロックレートを取得し、`rtp_jitter_buffer_set_clock_rate()` で内部バッファに設定します。
    * **ソースパッド (`gst_rtp_jitter_buffer_src_event`)**:
        * 下流からの `GST_EVENT_CUSTOM_DOWNSTREAM` イベント（"GstRTPJitterBufferLatency", "GstRTPJitterBufferMode"）を処理し、動的に `latency` や `mode` を変更できます。

4.  **パケット送出ループ (`gst_rtp_jitter_buffer_loop`)**:
    * このエレメントは、通常、独自の内部スレッド（`priv->task`）で `gst_rtp_jitter_buffer_loop()` 関数を実行します。
    * このループ関数は、定期的に `RTPJitterBuffer` (`priv->jbuf`) の状態を確認します。
    * `rtp_jitter_buffer_get_packet(priv->jbuf, &next_release_time, &is_lost)` を呼び出し、次に出力すべきパケットとその再生時刻 `next_release_time` を取得します。
    * 現在のパイプラインクロック時刻が `next_release_time` に達するまで待機します（または、`next_release_time` が過去の場合は即座に）。
    * 時刻が来たら、取得したパケットを下流にプッシュします (`gst_pad_push(GST_ELEMENT_SRCPAD(jbuf), outbuf)`)。
    * パケットのPTSは、`rtp_jitter_buffer_calculate_pts()` を使って、RTPタイムスタンプと同期情報に基づいて計算されます。

### `RTPJitterBuffer` コアライブラリの主要な仕組み

* **`RTPJitterBufferItem`**:
    * 個々のRTPパケットと、それに関連する情報（シーケンス番号、RTPタイムスタンプ、受信時刻、ヘッダ情報など）を格納する構造体です。
    * 内部のリスト（`jbuf->packets`、`GList` で実装）にソートされて格納されます。

* **パケットの追加 (`rtp_jitter_buffer_add_item`)**:
    * 新しい `RTPJitterBufferItem` を作成し、シーケンス番号の拡張（ラップアラウンド対応）、重複チェック、順序の確認などを行います。
    * 適切な位置に挿入します。`max_misorder_time` を超えるほど古いパケットはドロップされることがあります。

* **同期情報の管理 (`jbuf->base_time`, `jbuf->base_extrtp`, `jbuf->skew`, `jbuf->clock_rate`)**:
    * `base_time`: 最初に有効なパケットを受信したとき、または最初のRTCP SRに基づいて設定される、GStreamerのクロック時間（基準点）。
    * `base_extrtp`: 上記 `base_time` に対応する拡張RTPタイムスタンプ（基準点）。
    * `skew`: 送信側と受信側のクロックのずれや、ネットワーク伝搬遅延の初期推定値などを補正するための値。
    * `clock_rate`: RTPストリームのクロックレート（例: オーディオなら8000Hzや48000Hz、ビデオなら90000Hz）。
    * これらの値は、主に `rtp_jitter_buffer_set_syncinfo()` や最初のパケット到着時に初期化/更新されます。

* **PTS計算 (`rtp_jitter_buffer_calculate_pts`)**:
    * この関数がRTPタイムスタンプをGStreamerのPTS (Presentation TimeStamp) に変換する核心部分です。
    * `pts = base_time + skew + ((rtptime - base_extrtp) * GST_SECOND / clock_rate)` のような計算式が基本となります。
    * `rtptime` は現在のパケットの拡張RTPタイムスタンプです。
    * この計算により、RTPタイムスタンプの相対的な時間間隔が、GStreamerパイプラインの絶対的な時間軸（ランニングタイム）に正しくマッピングされます。

* **出力パケットの決定 (`rtp_jitter_buffer_get_packet_internal`)**:
    * バッファリング状態 (`jbuf->buffering_active`)、設定された `latency`、パケットのRTPタイムスタンプ、そして上記同期情報に基づいて、次に出力すべきパケットとその理想的なリリース時刻 (`next_release_time`) を決定します。
    * `max_dropout_time` を超えるほどパケットが到着しない場合は、強制的にバッファリングを解除して出力を開始することもあります。
    * `faststart_min_packets` の条件を満たせば、初期のバッファリング時間を短縮して早く再生を開始します。

### 主要なプロパティとその役割

`GstRtpJitterBuffer` エレメントは多くのプロパティを持ち、動作を細かく制御できます。以下はその一部です。

* **`latency` (デフォルト: 200 ms)**: 目標とするバッファリング遅延時間。この時間分のパケットをバッファに保持しようとします。
* **`mode` (デフォルト: `GST_RTP_JITTER_BUFFER_MODE_SLAVE` (0))**:
    * `GST_RTP_JITTER_BUFFER_MODE_SLAVE`: RTCP SR に基づいてクロック同期を行うモード。最も一般的なモード。
    * `GST_RTP_JITTER_BUFFER_MODE_BUFFER` (1): 単純なバッファリングと並べ替えのみを行い、タイムスタンプの再同期は行いません。送信側と受信側のクロックが既に非常に正確に同期している場合や、RTCPが利用できない場合に使用されることがあります。
    * `GST_RTP_JITTER_BUFFER_MODE_SYNCED` (2): 外部から提供される同期情報に追従するモード（特殊な用途）。
* **`drop-on-latency` (デフォルト: `FALSE`)**: `latency` プロパティで設定された時間内に処理できないほど遅れて到着したパケットをドロップするかどうか。`TRUE` にすると、古いパケットを捨ててリアルタイム性を優先します。
* **`do-lost` (デフォルト: `FALSE`)**: パケットロスを検出した場合、`GST_BUFFER_FLAG_GAP | GST_BUFFER_FLAG_DROPPABLE` フラグが設定された空のバッファを生成して下流に送るかどうか。
* **`rtcp-sync-send-time` (デフォルト: `TRUE`)**: RTCP SRのNTPタイムスタンプを「送信時刻」として信頼するかどうか。ネットワークの対称性が仮定できない場合は `FALSE` にすることが推奨されることもあります。
* **`max-dropout-time` (デフォルト: 0, 無効)**: この時間、連続してパケットが途切れた場合、バッファリングを強制的に終了して利用可能なパケットの出力を開始するまでの最大時間。
* **`max-misorder-time` (デフォルト: 2000 ms)**: シーケンス番号が大きく前後しているパケット（misordered packet）を、ドロップせずにどれだけの期間保持するか。
* **`rfc7273-sync` (デフォルト: `FALSE`)**: RFC7273 (RTPとPTPの同期) を使用するかどうか。
* **`faststart-min-packets` (デフォルト: 0, 無効)**: ストリーム開始時に、この数のパケットが連続して到着したら、`latency` で設定された時間まで待たずに再生を開始する。

### まとめ

`rtpjitterbuffer` は、GStreamerでRTPストリーミングを扱う上で非常に重要な役割を果たすエレメントです。ネットワークの不安定性（ジッター、ロス、順序入れ替え）を吸収し、RTPタイムスタンプとRTCP SR情報を活用して送信側のクロックに同期することで、高品質で安定したメディア再生を実現します。その動作は多くのプロパティによって細かく調整可能であり、アプリケーションの要件（低遅延優先か、安定性優先かなど）に応じて最適化することができます。

> latencyプロパティの設定値と処理への影響を詳しく解説してください。

`rtpjitterbuffer` エレメントの `latency` プロパティは、ネットワークジッターを吸収するためにエレメントが内部でRTPパケットをどれだけの期間バッファリングしようとするかの目標値をミリ秒単位で指定します。この設定値は、リアルタイムストリーミングの品質（スムーズさ）と遅延（リアルタイム性）の間のトレードオフを調整する上で非常に重要です。

提供されたソースコード (`gstrtpjitterbuffer.c` および `rtpjitterbuffer.c`) を基に、`latency` プロパティの設定値が処理に与える影響を詳しく解説します。

### `latency` プロパティの定義と役割

  * **定義**: `gstrtpjitterbuffer.c` の `gst_rtp_jitter_buffer_class_init()` 関数内で、`latency` プロパティは以下のように定義されています（デフォルト値は200ミリ秒）。

    ```c
    // gstrtpjitterbuffer.c
    g_object_class_install_property (gobject_class, PROP_LATENCY,
        g_param_spec_uint ("latency", "Latency",
            "Amount of ms to buffer",
            0, G_MAXUINT, DEFAULT_LATENCY,
            G_PARAM_READWRITE | G_PARAM_STATIC_STRINGS |
            GST_PARAM_MUTABLE_PLAYING));
    ```

    `DEFAULT_LATENCY` は通常 200 (ミリ秒) です。

  * **役割**:

      * **ジッター吸収**: ネットワーク経由でRTPパケットを受信する際、各パケットの到着間隔は一定ではなく、揺らぎ（ジッター）が発生します。`latency` プロパティで指定された時間だけパケットをバッファに保持することで、これらの到着時間のばらつきを吸収し、下流のエレメント（デコーダやシンク）にはより均一な間隔でパケットを供給することを目指します。
      * **再生タイミングの基準**: `rtpjitterbuffer` は、バッファリングされたパケットをすぐには下流に送出せず、`latency` で指定された目標遅延時間と、パケットのRTPタイムスタンプおよびRTCP SRからの同期情報に基づいて計算された「理想的な再生時刻」を考慮して送出タイミングを決定します。

### `latency` プロパティ設定時の内部処理

`latency` プロパティが設定されると、`gstrtpjitterbuffer.c` の `gst_rtp_jitter_buffer_set_property()` 関数が呼び出されます。

```c
// gstrtpjitterbuffer.c
static void
gst_rtp_jitter_buffer_set_property (GObject * object, guint prop_id,
    const GValue * value, GParamSpec * pspec)
{
  GstRtpJitterBuffer *jbuf = GST_RTP_JITTER_BUFFER (object);
  GstRtpJitterBufferPrivate *priv = jbuf->priv;

  GST_OBJECT_LOCK (jbuf); // エレメントのロック

  switch (prop_id) {
    // ... (他のプロパティの処理) ...
    case PROP_LATENCY:
      priv->latency = g_value_get_uint (value); // GstRtpJitterBuffer のプライベート構造体に値を保存
      if (priv->jbuf) // コアの RTPJitterBuffer インスタンスが存在すれば
        rtp_jitter_buffer_set_latency (priv->jbuf, priv->latency); // コアライブラリの関数を呼び出し
      GST_DEBUG_OBJECT (jbuf, "latency set to %u ms", priv->latency);
      break;
    // ... (他のプロパティの処理) ...
    default:
      G_OBJECT_WARN_INVALID_PROPERTY_ID (object, prop_id, pspec);
      break;
  }
  GST_OBJECT_UNLOCK (jbuf);
}
```

このコードからわかるように、`latency` プロパティの値は `GstRtpJitterBuffer` のプライベート構造体 `priv->latency` に保存され、さらにコアのジッターバッファライブラリのインスタンス `priv->jbuf` に対して `rtp_jitter_buffer_set_latency()` 関数を呼び出して設定されます。

`rtpjitterbuffer.c` の `rtp_jitter_buffer_set_latency()` 関数は、受け取った値を内部の `RTPJitterBuffer` 構造体の `latency` フィールドに格納します。

```c
// rtpjitterbuffer.c
void
rtp_jitter_buffer_set_latency (RTPJitterBuffer * jbuf, guint latency)
{
  g_return_if_fail (RTP_IS_JITTER_BUFFER (jbuf));

  JBUF_LOCK (jbuf); // RTPJitterBuffer 内部のロック
  jbuf->latency = latency;
  JBUF_UNLOCK (jbuf);
}
```

この `jbuf->latency` の値は、後述するパケット送出ロジックで重要な役割を果たします。

### `latency` の値が処理に与える影響

`latency` の設定値は、主にパケットの送出タイミング決定ロジック (`rtpjitterbuffer.c` の `rtp_jitter_buffer_get_packet_internal()` や、それを呼び出す `rtp_jitter_buffer_get_packet()`) に影響を与えます。

1.  **バッファリングの深さ**:

      * `latency` の値が大きいほど、ジッターバッファはより多くのパケットを内部に保持しようとします（時間的に）。これにより、より大きなネットワークジッターや、一時的なパケット順序の入れ替わりを吸収できる可能性が高まります。
      * 結果として、再生が途切れたりカクついたりする可能性が低減されます。

2.  **再生開始までの遅延**:

      * `latency` の値が大きいほど、最初のパケットを受信してから実際に再生が開始されるまでの遅延（初期遅延）が大きくなる傾向があります。これは、エレメントが `latency` 分のデータをバッファしようとするためです。
      * ただし、`faststart-min-packets` プロパティが設定されている場合、指定された数の連続したパケットが到着すれば、`latency` に達していなくても再生を開始することがあります。

3.  **リアルタイム性**:

      * `latency` の値が大きいほど、エンドツーエンドの遅延（送信者がメディアを送信してから受信者がそれを再生するまでの時間）が増加します。これは、インタラクティブなアプリケーション（ビデオ会議など）では問題となる可能性があります。
      * 逆に `latency` の値を小さくすると、遅延は減少しますが、ジッター吸収能力が低下し、ネットワーク状態が悪い場合には再生品質が低下しやすくなります。

4.  **パケットの送出タイミング (`rtp_jitter_buffer_get_packet_internal`)**:

      * この関数は、次に出力すべきパケットとその理想的なリリース時刻 (`next_release_time`) を決定します。
      * `mode` が `GST_RTP_JITTER_BUFFER_MODE_SLAVE` の場合、`next_release_time` は主にパケットのRTPタイムスタンプ、RTCP SRからの同期情報 (`base_time`, `base_extrtp`, `skew`, `clock_rate`)、そしてこの `latency` を考慮して計算されます。
      * 具体的には、バッファ内の最初のパケットの理想的な再生時刻を計算し、その時刻に `latency` を加味した時刻を、パケットを保持すべき目標時刻とします。
        ```c
        // rtpjitterbuffer.c の rtp_jitter_buffer_get_packet_internal() 内のロジックの一部 (概念)
        // 理想的な最初のパケットのリリース時刻 ideal_release_time を計算
        // ...
        // ターゲットリリース時刻 target_time = ideal_release_time + GST_MSECOND * jbuf->latency;
        // ...
        // 現在時刻が target_time に達していなければ、まだパケットを出さない (待つ)
        // ただし、バッファが空になりそうな場合や、max_dropout_time を超えた場合は別
        ```
      * `mode` が `GST_RTP_JITTER_BUFFER_MODE_BUFFER` の場合は、より単純に `latency` 時間分のバッファを維持しようとします。

5.  **`drop-on-latency` プロパティとの連携**:

      * `drop-on-latency` が `TRUE` の場合、あるパケットの理想的な再生時刻に `latency` を加えた時刻よりも、現在のパイプラインクロック時刻がさらに進んでしまっている（つまり、そのパケットは「遅すぎる」）と判断されると、そのパケットはドロップされます。
      * `drop-on-latency` が `FALSE` (デフォルト) の場合は、遅れて到着したパケットも可能な限り再生しようとしますが、これにより全体の遅延が増加する可能性があります。

### `latency` 設定値の選択ガイドライン

適切な `latency` の値は、ネットワーク環境とアプリケーションの要件によって異なります。

  * **ネットワーク状態が良い (ジッターが小さい) 場合**:
      * `latency` を比較的小さな値（例: 数十ミリ秒）に設定しても、スムーズな再生が期待できます。これにより、リアルタイム性が向上します。
  * **ネットワーク状態が悪い (ジッターが大きい) 場合**:
      * `latency` を大きな値（例: 200ミリ秒以上）に設定することで、ジッター吸収能力を高め、再生の途切れを防ぐことができます。ただし、遅延は増加します。
  * **インタラクティブなアプリケーション (例: ビデオ会議、オンラインゲーム)**:
      * 遅延を最小限に抑えることが重要なので、許容できる範囲で `latency` を小さく設定する必要があります。ネットワーク品質の確保がより重要になります。
  * **ストリーミング再生 (例: オンデマンドビデオ、IPTV)**:
      * 多少の初期遅延は許容できる場合が多いので、安定した再生のために `latency` を比較的高めに設定することが一般的です。

GStreamer のドキュメントや `rtpjitterbuffer` のデフォルト値 (200ms) は、多くの一般的なユースケースで妥当な出発点となりますが、実際の環境でテストし、最適な値を見つけることが推奨されます。`gst-launch-1.0` の `-v` オプションや、`GST_DEBUG="rtpjitterbuffer:5"` のようなデバッグ出力を有効にすることで、ジッターバッファの動作状況（バッファリングレベル、ドロップされたパケットなど）を確認しながら調整することができます。

また、`rtpjitterbuffer` は `num-lost` や `avg-jitter` などの統計情報をGStreamerメッセージとしてバスにポストすることがあるため、アプリケーション側でこれらの情報を監視し、動的に `latency` を調整するような高度な制御も可能です（ただし、そのような動的調整ロジックはアプリケーション側で実装する必要があります）。

> ジッターの計算式とlatencyプロパティの関係を教えてください

`rtpjitterbuffer` エレメントにおける「ジッター」の計算式そのものは、ソースコード内に特定の「この式でジッター値を算出しています」という形で明示されているわけではありません。`rtpjitterbuffer` は、ジッターそのものを数値として計算し続けるというよりは、**ジッターによって発生する現象（パケット到着時刻の揺らぎ、順序の入れ替わり）を吸収・緩和する**ことを目的として動作します。

しかし、RFC 3550 (RTP: A Transport Protocol for Real-Time Applications) で定義されている**RTCP Receiver Report (RR) で報告されるジッター値の計算方法**が、間接的に `rtpjitterbuffer` の動作や、QoS (Quality of Service) メッセージの生成、統計情報の提供などに関連してきます。

そして、`latency` プロパティは、このジッターをどれだけ許容し、吸収するためのバッファリング時間（遅延）を設けるか、という点で深く関わってきます。

### RFC 3550 におけるジッター計算

RFC 3550 (Section 6.4.1) では、受信側が計算するジッター値について以下のように定義されています。これは、連続するパケット間の到着時刻の差の統計的な分散（具体的には平均偏差）を表します。

1.  あるパケット `i` がRTPタイムスタンプ $R\_i$ で送信され、受信時刻 $S\_i$ (ローカルクロック時間) で到着したとします。

2.  同様に、その直前のパケット `i-1` はRTPタイムスタンプ $R\_{i-1}$ で送信され、受信時刻 $S\_{i-1}$ で到着したとします。

3.  パケット間の伝送時間差 $D(i-1, i)$ は以下のように計算されます。
    $D(i-1, i) = (S\_i - S\_{i-1}) - (R\_i - R\_{i-1})$

      * $(S\_i - S\_{i-1})$: パケット $i-1$ と $i$ の受信時刻の間隔。
      * $(R\_i - R\_{i-1})$: パケット $i-1$ と $i$ の送信時刻の間隔（RTPタイムスタンプの差をクロックレートで割って秒単位に換算したもの）。
      * 理想的なネットワークでは、$D(i-1, i)$ は常に0に近いはずです。これが大きく変動するのがジッターです。

4.  この $D$ の値を使って、受信ジッター $J\_i$ は以下の平滑化アルゴリズム（EWMA: Exponentially Weighted Moving Average）で更新されます。
    $J\_i = J\_{i-1} + (|D(i-1, i)| - J\_{i-1}) / 16$
    または、整数演算で書くと:
    $J\_i = J\_{i-1} + ((|D(i-1, i)| \<\< 4) - J\_{i-1} + 8) \>\> 4$

この計算されたジッター値 $J\_i$ (RTPタイムスタンプ単位) は、RTCP RRブロックの `interarrival jitter` フィールドに設定されて送信者に報告されます。

### `rtpjitterbuffer` とジッター計算・`latency` の関係

`rtpjitterbuffer` のソースコード (`rtpjitterbuffer.c`) を見ると、`rtp_jitter_buffer_add_item()` 関数内で、新しく到着したパケットの情報を元に、まさに上記RFC 3550の定義に近い形でジッター計算の準備が行われている部分があります。

```c
// rtpjitterbuffer.c の rtp_jitter_buffer_add_item 関数内の一部 (簡略化)
static RTPJitterBufferItem *
rtp_jitter_buffer_add_item (RTPJitterBuffer * jbuf, RTPJitterBufferItem * item)
{
  // ... (他の処理) ...

  // ジッター計算のための情報更新
  if (GST_CLOCK_TIME_IS_VALID (jbuf->last_arrival) &&
      GST_CLOCK_TIME_IS_VALID (item->arrival) &&
      jbuf->clock_rate != 0 && jbuf->received_any) {
    GstClockTime transit_diff;
    GstClockTime arrival_diff, rtp_time_diff;
    guint32 jitter;

    arrival_diff = item->arrival - jbuf->last_arrival;
    rtp_time_diff =
        (item->ext_rtptime -
        jbuf->last_rtptime) * (GstClockTime) GST_SECOND / jbuf->clock_rate;

    if (rtp_time_diff > arrival_diff)
      transit_diff = rtp_time_diff - arrival_diff;
    else
      transit_diff = arrival_diff - rtp_time_diff;

    // RFC3550のジッター計算 (D(i-1, i) に相当する部分)
    // D = (S_i - S_{i-1}) - (R_i - R_{i-1})
    // ここでは transit_diff が |D| に近い値 (絶対値を取る前の差の絶対値)
    // jbuf->jitter はRTPタイムスタンプ単位
    jitter = jbuf->jitter;
    jitter += (guint32) ((transit_diff * jbuf->clock_rate / GST_SECOND) -
        jitter + 8) / 16; // EWMA の更新
    jbuf->jitter = jitter;

    GST_LOG_OBJECT (jbuf,
        "arrival %" GST_TIME_FORMAT " (+%" G_GINT64_FORMAT "), rtp %"
        G_GUINT64_FORMAT " (+%" G_GINT64_FORMAT "), jitter %u",
        GST_TIME_ARGS (item->arrival),
        GST_CLOCK_DIFF_ARGS (arrival_diff), item->ext_rtptime,
        item->ext_rtptime - jbuf->last_rtptime, jbuf->jitter);
  }
  jbuf->last_arrival = item->arrival;
  jbuf->last_rtptime = item->ext_rtptime;
  jbuf->received_any = TRUE;

  // ... (パケットをリストに追加する処理など) ...

  return item;
}
```

このコードでは、連続するパケットの到着時刻の差 (`arrival_diff`) とRTPタイムスタンプの差 (`rtp_time_diff`) を比較し、その差の絶対値 (`transit_diff`) を使って `jbuf->jitter` を更新しています。この `jbuf->jitter` がRTCP RRで報告されるジッター値の基になります。

**`latency` プロパティとの関係性**:

1.  **ジッターの許容量**: `latency` プロパティは、「これくらいの時間分のジッター（到着時刻の揺らぎ）までは吸収できるようにバッファリングする」という目標値をエレメントに与えます。つまり、**`latency` はジッターそのものを計算する式には直接現れませんが、計算されたジッターや観測される到着時間の揺らぎに対して、システムがどれだけ耐性を持つかを決定するパラメータ**です。

2.  **バッファリング判断への影響**:

      * `rtpjitterbuffer` は、`latency` で指定された時間分のデータを保持しようとします。
      * ネットワークのジッターが大きい場合、パケットの到着間隔は大きく変動します。`latency` が十分に大きければ、遅れて到着したパケットもバッファ内に保持され、再生順序が来たときに下流に渡されるため、再生が途切れるのを防ぐことができます。
      * `latency` が小さい場合、大きなジッターによって遅れたパケットは、すでに再生されるべき時刻を過ぎていると判断され、ドロップされるか（`drop-on-latency = TRUE` の場合）、あるいは後から再生されて全体の遅延を増加させる可能性があります。

3.  **再生開始タイミング**:

      * `rtpjitterbuffer` は、バッファ内のパケットのRTPタイムスタンプとRTCP SRからの同期情報（`base_time`, `base_extrtp`, `skew`, `clock_rate`）を使って、各パケットが再生されるべき理想的な時刻を計算します。
      * `latency` の値は、この理想的な再生時刻に対して、さらにどれだけ追加で遅延（バッファリング）を許容するかというオフセットとして機能します。
      * `rtp_jitter_buffer_get_packet_internal()` 内で、バッファからパケットを取り出すかどうかを判断する際に、この `latency` が考慮されます。具体的には、バッファの先頭パケットの理想的な再生時刻に `latency` を加算した時刻 (`target_time`) を計算し、現在のパイプラインクロック時刻がこの `target_time` に達するまでパケットの送出を待つ、というようなロジックが含まれています。

4.  **QoSメッセージ**:

      * `rtpjitterbuffer` は、観測されたジッターやバッファリング状態（アンダーフローやオーバーフローの可能性）に基づいて、QoSメッセージをGStreamerのバスにポストすることがあります。
      * このQoSメッセージには、現在のジッターレベルやバッファ占有率などの情報が含まれることがあり、アプリケーションはこれを利用してネットワーク状態を把握したり、`latency` プロパティを動的に調整したりすることができます。

**まとめると**:

  * `rtpjitterbuffer` は内部的にRFC 3550に準拠した方法でパケット到着間隔の変動（ジッターの指標となる値）を追跡・計算しています。この計算されたジッター値は主にRTCP RRでの報告に使われます。
  * `latency` プロパティは、この観測されるジッターに対して、エレメントがどれだけのバッファリング時間（遅延）を設けて対応するかの目標値を設定します。
  * `latency` が大きいほど、大きなジッターを吸収してスムーズな再生を維持しやすくなりますが、全体の遅延は増加します。
  * `latency` が小さいほど、遅延は減りますが、ジッターに対する耐性が低下し、再生が不安定になる可能性があります。

したがって、`latency` はジッターの「計算式」に直接組み込まれるのではなく、ジッターという現象に対してエレメントがどのように振る舞うか（どれだけバッファして吸収するか）を制御するための重要なパラメータとなります。
