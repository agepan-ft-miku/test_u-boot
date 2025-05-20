GStreamer の `rtpjitterbuffer` エレメントは、ネットワークソースから受信した RTP (Real-time Transport Protocol) パケットのジッター（到着時間の揺らぎ）を吸収し、パケットの並べ替えや重複パケットの削除を行う重要なコンポーネントです。これにより、メディアストリームの再生品質を向上させます。

### `rtpjitterbuffer` の主な機能と詳細

`rtpjitterbuffer` エレメントの主な機能と、関連するソースコード内の記述は以下の通りです。

* **パケットの並べ替えと重複排除**:
    * ネットワーク経由で受信される RTP パケットは、順序通りに到着するとは限りません。また、同じパケットが複数回届くこともあります。`rtpjitterbuffer` は、これらのパケットをシーケンス番号に基づいて並べ替え、重複したパケットを破棄します。

* **ジッター吸収とレイテンシ**:
    * `latency` プロパティで設定された時間（デフォルトでは 200 ミリ秒）だけパケットをバッファリングし、ネットワークジッターを吸収します。
    * `drop-on-latency` プロパティが TRUE の場合、バッファがレイテンシで設定された上限に達すると、最も古いパケットから破棄します。

* **欠落パケットの処理 (`do-lost`)**:
    * `do-lost` プロパティが TRUE の場合、パケットが設定されたレイテンシ時間内に到着せず、欠落したと判断されると、`GstRTPPacketLost` という名前のカスタムシリアライズイベントを下流に送信します。
    * このイベントは、デペイローダーなどのエレメントがパケット欠損時のデータを補間生成するなどの処理に利用されます。

* **再送要求 (`do-retransmission`)**:
    * `do-retransmission` プロパティが TRUE の場合、パケットが期待される時間に到着しないと判断されると、`GstRTPRetransmissionRequest` という名前のカスタムアップストリームイベントを送信し、再送を要求します。
    * 再送要求のタイミングやリトライ間隔、リトライ期間などは、以下のプロパティで細かく制御できます:
        * `rtx-delay`: パケットが期待通りに到着しなかった場合に、再送イベントを送信するまでの追加待機時間。
        * `rtx-min-delay`: 再送イベントを送信するまでの最小追加待機時間。
        * `rtx-delay-reorder`: この値以上のパケット並び替えが発生した場合に再送イベントを送信するとみなす。
        * `rtx-retry-timeout`: 再送イベント送信後、この時間内にパケットが受信されなかった場合に再送イベントを再試行するタイムアウト。
        * `rtx-min-retry-timeout`: `rtx-retry-timeout` が自動(-1)の場合の、リトライタイムアウトの最小間隔。
        * `rtx-retry-period`: 再送を取得しようとする期間。
        * `rtx-max-retries`: 再送要求の最大リトライ回数。
        * `rtx-deadline`: 有効な RTX リクエストのデッドライン（ミリ秒）。

* **タイムスタンプ処理**:
    * 入力バッファの DTS (Decoding Time Stamp) または PTS (Presentation Time Stamp) と、RTP パケット内の RTP タイムスタンプを使用して、出力バッファの PTS を生成します。
    * `ts-offset` プロパティにより、出力バッファのタイムスタンプを調整できます。これは主にストリーム間の同期に使用されます。
    * `max-ts-offset-adjustment` プロパティで、フレームごとのタイムスタンプオフセットの最大調整量を設定できます。

* **クロックレートの取得**:
    * RTP ペイロードのクロックレート情報を必要とします。これは、シンクパッドの Caps から取得するか、Caps がない場合は `request-pt-map` シグナルを通じて取得します。
    * `clear-pt-map` シグナルで以前のペイロードタイプマップをクリアできます。

* **統計情報 (`stats`)**:
    * `stats` プロパティを通じて、プッシュされたパケット数、損失パケット数、遅延パケット数、重複パケット数、再送要求数、再送成功数などの統計情報を取得できます。

* **モード (`mode`)**:
    * `mode` プロパティで、バッファリングとタイムスタンプのアルゴリズムを制御できます。デフォルトは `slave` モードです。

* **`rtpbin` との連携**:
    * `rtpjitterbuffer` は、RTP セッション管理を行う `rtpbin` エレメントの内部で自動的に使用されます。

### パッド

`rtpjitterbuffer` エレメントは以下のパッドを持ちます:

* **`sink`**: RTP パケットを受信する常時接続のシンクパッド。`application/x-rtp` の Caps を持ちます。
* **`sink_rtcp`**: RTCP (RTP Control Protocol) パケットを受信する要求パッド。`application/x-rtcp` の Caps を持ちます。
* **`src`**: ジッター補正および並べ替え処理後の RTP パケットを出力する常時接続のソースパッド。`application/x-rtp` の Caps を持ちます。

### 使用例

`rtpjitterbuffer` は、RTP ストリームを受信する際に、ネットワークの揺らぎを吸収し、安定したメディア再生を実現するために不可欠なエレメントです。テストコード (`tests/check/elements/rtpjitterbuffer.c`) では、様々なシーケンスのパケット（順序通り、逆順、順不同など）をプッシュし、`rtpjitterbuffer` が正しく並べ替えて出力することを確認するテストケースが含まれています。また、パケットロスイベントや再送要求イベントのテストも行われています。

この情報は、提供された GStreamer プラグインリポジトリの `gst-plugins-good-1.16.3` ブランチのコードに基づいています。


> rtpjitterbufferにおける、出力バッファのPTSの計算方法を解説してください。

`rtpjitterbuffer` エレメントにおける出力バッファの PTS (Presentation Time Stamp) の計算は、いくつかの要素と設定モードに基づいて行われます。主な処理は内部の `RTPJitterBuffer` オブジェクトの `rtp_jitter_buffer_calculate_pts` 関数と、それを利用する `gstrtpjitterbuffer.c` 内のロジックによって実行されます。

以下にその計算方法の主要なステップと考慮事項を解説します。

### 1. 基本的なタイムスタンプの取得と変換

* **入力タイムスタンプ**:
    * 入力バッファの DTS (Decoding Time Stamp) が主に使用されます。DTS がない場合は PTS が使用されます。
    * これらのタイムスタンプが存在しない場合（例: ネットワークから受信したばかりのパケット）、現在のクロックから取得したランニングタイムが `dts` として使用されることがあります。
    * `dts` は、エレメントのセグメント情報に基づいてランニングタイムに変換されます。
* **RTP タイムスタンプ**:
    * RTP パケットヘッダ内の `rtptime` が使用されます。
    * この `rtptime` は、`clock-rate` を用いて GStreamer の時間単位 (ナノ秒) に変換された `gstrtptime` となります。
    * 連続する RTP タイムスタンプのラップアラウンドを処理するために、拡張 RTP タイムスタンプ (`ext_rtptime`) が内部で維持されます。

### 2. `rtp_jitter_buffer_calculate_pts` 関数によるコア計算 (rtpjitterbuffer.c)

この関数が PTS 計算の中心的な役割を担います。

* **初期同期とベースタイムスタンプ**:
    * 最初に受信したパケット、またはリセット後の最初のパケットで、`jbuf->base_time` (GStreamer 時間の基準点) と `jbuf->base_rtptime` (RTP 時間の基準点、`gstrtptime` 形式) が設定されます。
    * これらのベースタイムスタンプは、以降のパケットの相対的な時間を計算するための基準となります。
    * `jbuf->need_resync` フラグが立っている場合や、特定の条件下（例: RTP タイムスタンプの大きなジャンプ）で、これらのベースタイムスタンプは再同期（リセット）されます。

* **モード (`jbuf->mode`) に応じた処理**:
    * **`RTP_JITTER_BUFFER_MODE_NONE` / `RTP_JITTER_BUFFER_MODE_BUFFER`**:
        * 最初のパケットの `dts` は 0 とされ、それ以降のパケットの `dts` は -1 (無効) として扱われます。これにより、RTP タイムスタンプに基づいて 0 から始まるタイムスタンプが生成されます。`BUFFER` モードでは、バッファリング時間に応じてさらに調整が加わることがあります。
    * **`RTP_JITTER_BUFFER_MODE_SYNCED`**:
        * 最初のパケットの `dts` を基準とし、それ以降は RTP タイムスタンプに基づいて補間します。
    * **`RTP_JITTER_BUFFER_MODE_SLAVE` (デフォルト)**:
        * 受信したパケットの `dts` を使用して、送信側のクロックに対する受信側クロックのずれ（スキュー）を推定し、補正しようとします。

* **クロックスキューの計算と補正 (`calculate_skew` 関数)**:
    * `RTP_JITTER_BUFFER_MODE_SLAVE` モードなどで重要になります。
    * 送信側での経過時間 (`send_diff` = 現在の `gstrtptime` - `jbuf->base_rtptime`) と、受信側での経過時間 (`recv_diff` = 現在の `dts` - `jbuf->base_time`) の差 (`delta`) を計算します。
    * この `delta` の値を平滑化（ウィンドウ内の最小値を使用し、さらに平均化）してクロックスキュー (`jbuf->skew`) を推定します。
    * 計算された PTS は `jbuf->base_time + send_diff + jbuf->skew` のように補正されます。
    * 急激なスキューの変化はタイムスタンプのリセットを引き起こす可能性があります。

* **RFC7273 同期**:
    * `rfc7273-sync` プロパティが有効で、かつメディアクロック (`media_clock`) とそのオフセット (`media_clock_offset`) が提供されている場合、`rtpjitterbuffer` は受信ストリームを RFC7273 で定義された外部クロックに同期させようとします。
    * このモードでは、NTP や PTP などの外部クロックソースから取得した時刻情報と RTP タイムスタンプを関連付けて PTS を計算します。これにより、ネットワーク遅延の変動によらない、より正確な同期が期待できます。

* **タイムスタンプの単調増加の保証**:
    * 計算された PTS が直前の出力 PTS (`jbuf->prev_out_time`) よりも小さい場合（タイムスタンプが逆行する場合）は、`jbuf->prev_out_time` が使用され、単調増加が保証されます。
    * 同様に、入力 `dts` に対して計算された PTS が著しく早い場合（例: `pts + jbuf->delay < dts`）、サーバーが一時停止した可能性などを考慮し、ジッターバッファがリセットされ、`pts` が `dts` に設定されることがあります。

### 3. `gstrtpjitterbuffer.c` での最終調整と出力

`rtpjitterbuffer.c` で計算された PTS は、`gstrtpjitterbuffer.c` の `pop_and_push_next` 関数内でさらに調整されます。

* **セグメント変換**:
    * 計算されたランニングタイムとしての PTS は、現在のセグメント (`priv->segment`) を用いて、ストリームタイムに変換されます。
* **オフセットの適用 (`apply_offset` 関数)**:
    * **`ts-offset`**: `ts-offset` プロパティで設定された値が PTS に加算されます。これは主にストリーム間の同期（例: オーディオとビデオ）のために使用されます。
    * **`out_offset`**: ジッターバッファがバッファリングモードからアクティブモードに移行する際などに計算される内部的なオフセットが加算されます。
    * `max-ts-offset-adjustment` プロパティが設定されている場合、`ts-offset` の急激な変化を避けるために、フレームごとに段階的に調整が行われます。
* **バッファフラグ**:
    * パケットロスなどにより不連続が発生した場合は、出力バッファに `GST_BUFFER_FLAG_DISCONT` フラグが設定されます。
    * タイムスタンプの大きな不連続や再同期が必要な場合は `GST_BUFFER_FLAG_RESYNC` フラグが設定されることがあります。

### まとめ

`rtpjitterbuffer` の PTS 計算は、入力パケットの RTP タイムスタンプと到着時刻 (DTS)、設定された動作モード、クロックレート、オプションの外部メディアクロック、そして `ts-offset` などのプロパティ値を総合的に考慮して行われます。主な目的は、ネットワークジッターを吸収し、クロックスキューを補正し、他のストリームとの同期を取りながら、滑らかで正確なタイミングで下流にメディアバッファを供給することです。


> rtpjitterbufferにおけるlatencyプロパティの機能と処理に与える影響についてソースコードを例示しながら解説してください。


`rtpjitterbuffer` エレメントの `latency` プロパティは、ネットワークジッターを吸収するために RTP パケットをバッファリングする最大時間をミリ秒単位で指定します。この値は、エレメントがパケットをどれくらいの期間保持し、順序の入れ替えや欠落パケットの到着待ちを行うかを決定する上で非常に重要です。

デフォルト値は `200` ミリ秒です。

### `latency` プロパティの機能

1.  **ジッター吸収**: ネットワーク遅延の変動（ジッター）により、パケットの到着間隔が一定でなくなることがあります。`latency` プロパティで設定された時間だけパケットを内部バッファに保持することで、これらの到着時間の揺らぎを平滑化し、より安定したメディアストリームを下流に供給します。

2.  **パケット並べ替えのための待機時間**: RTP パケットはネットワークの状況により順序通りに到着しないことがあります。`latency` は、後続のパケットが到着するのを待つ時間の上限となり、この時間内に到着したパケットはシーケンス番号に基づいて並べ替えられます。

3.  **欠落パケットの判断基準**:

      * `do-lost` プロパティが `TRUE` の場合、パケットが `latency` で設定された時間内に到着しないと「欠落した」と判断されます。
      * この場合、`GstRTPPacketLost` という名前のカスタムイベントが下流に送信されます。

4.  **再送要求のタイムアウト基準 (間接的)**:

      * `do-retransmission` プロパティが `TRUE` の場合、`latency` は再送要求 (`rtx-retry-period`) の期間を決定する際の要素の一つとなり得ます。具体的には、`rtx-retry-period` が自動設定 (-1) の場合、`latency` と観測されたラウンドトリップタイムに基づいて再送を試みる期間が推定されます。

5.  **バッファ管理 (`drop-on-latency`)**:

      * `drop-on-latency` プロパティが `TRUE` の場合、ジッターバッファ内のデータ量が `latency` で指定された時間分を超過しないように、最も古いパケットから破棄します。
      * `gstrtpjitterbuffer.c` の `gst_rtp_jitter_buffer_chain` 関数内で、`drop-on-latency` が有効で、かつ内部バッファ (`priv->jbuf`) の保持期間 (`rtp_jitter_buffer_get_ts_diff`) が設定された `latency_ns` を超えた場合に、古いアイテムをドロップする処理があります。

    <!-- end list -->

    ```c
    /* gstrtpjitterbuffer.c */
    if (priv->latency_ms && priv->drop_on_latency) {
        latency_ts =
            gst_util_uint64_scale_int (priv->latency_ms, priv->clock_rate, 1000);

        if (G_UNLIKELY (rtp_jitter_buffer_get_ts_diff (priv->jbuf) >= latency_ts)) {
          RTPJitterBufferItem *old_item;

          old_item = rtp_jitter_buffer_peek (priv->jbuf);

          if (IS_DROPABLE (old_item)) { // IS_DROPABLE はバッファかLOSTイベントかをチェック
            old_item = rtp_jitter_buffer_pop (priv->jbuf, &percent);
            GST_DEBUG_OBJECT (jitterbuffer, "Queue full, dropping old packet %p",
                old_item);
            priv->next_seqnum = (old_item->seqnum + old_item->count) & 0xffff;
            free_item (old_item); // free_item でバッファを解放
          }
          /* we might have removed some head buffers, signal the pushing thread to
           * see if it can push now */
          JBUF_SIGNAL_EVENT (priv);
        }
      }
    ```

### `latency` プロパティが処理に与える影響

  * **遅延の増加**: `latency` の値が大きいほど、パイプライン全体の遅延は増加します。これは、パケットをより長くバッファリングするためです。リアルタイム性が重要なアプリケーションでは、この値を適切に設定する必要があります。
  * **ジッター耐性の向上**: `latency` の値が大きいほど、より大きなネットワークジッターを吸収できます。これにより、ネットワーク状況が不安定な場合でも、スムーズな再生が期待できます。
  * **パケットロスと品質**:
      * `latency` が小さい場合、パケットが少し遅れただけでも「欠落」と判断されやすくなり、結果として画質や音質が低下する可能性があります（特に `do-lost` が有効で下流が欠損処理をしない場合）。
      * `latency` が大きい場合、より多くのパケットの到着を待つため、欠落と判断されるパケットは減りますが、遅延が増加します。
  * **リソース消費**: バッファリングするパケットが増えるため、`latency` を大きくするとメモリ使用量が増加する可能性があります。

### ソースコードにおける `latency` の設定と使用

  * **プロパティ定義 (`gstrtpjitterbuffer.c`)**:
    `latency` プロパティは `g_object_class_install_property` を通じて定義され、デフォルト値は `DEFAULT_LATENCY_MS` (200ミリ秒) です。

    ```c
    /* gstrtpjitterbuffer.c */
    g_object_class_install_property (gobject_class, PROP_LATENCY,
        g_param_spec_uint ("latency", "Buffer latency in ms",
            "Amount of ms to buffer", 0, G_MAXUINT, DEFAULT_LATENCY_MS,
            G_PARAM_READWRITE | G_PARAM_STATIC_STRINGS));
    ```

  * **プロパティ設定時 (`gst_rtp_jitter_buffer_set_property`)**:
    `latency` プロパティが設定されると、`priv->latency_ms` と `priv->latency_ns` (ナノ秒単位) が更新され、内部の `RTPJitterBuffer` オブジェクトにも `rtp_jitter_buffer_set_delay` を通じてナノ秒単位の遅延時間が設定されます。

    ```c
    /* gstrtpjitterbuffer.c */
    static void
    gst_rtp_jitter_buffer_set_property (GObject * object,
        guint prop_id, const GValue * value, GParamSpec * pspec)
    {
      GstRtpJitterBuffer *jitterbuffer;
      GstRtpJitterBufferPrivate *priv;

      jitterbuffer = GST_RTP_JITTER_BUFFER (object);
      priv = jitterbuffer->priv;

      switch (prop_id) {
        case PROP_LATENCY:
        {
          guint new_latency, old_latency;

          new_latency = g_value_get_uint (value);

          JBUF_LOCK (priv); // ジッターバッファのロック
          old_latency = priv->latency_ms;
          priv->latency_ms = new_latency;
          priv->latency_ns = priv->latency_ms * GST_MSECOND; // ミリ秒からナノ秒へ変換
          rtp_jitter_buffer_set_delay (priv->jbuf, priv->latency_ns); // 内部ジッターバッファへ設定
          JBUF_UNLOCK (priv); // ロック解除

          /* post message if latency changed, this will inform the parent pipeline
           * that a latency reconfiguration is possible/needed. */
          if (new_latency != old_latency) {
            GST_DEBUG_OBJECT (jitterbuffer, "latency changed to: %" GST_TIME_FORMAT,
                GST_TIME_ARGS (new_latency * GST_MSECOND));

            gst_element_post_message (GST_ELEMENT_CAST (jitterbuffer),
                gst_message_new_latency (GST_OBJECT_CAST (jitterbuffer)));
          }
          break;
        }
        // ... (他のプロパティ処理)
      }
    }
    ```

  * **内部 `RTPJitterBuffer` での遅延設定 (`rtpjitterbuffer.c`)**:
    `rtp_jitter_buffer_set_delay` 関数は、`jbuf->delay` を設定するとともに、バッファリングモード (`RTP_JITTER_BUFFER_MODE_BUFFER`) で使用される低水位 (`low_level`) および高水位 (`high_level`) の閾値を計算します。これらはバッファの充填率を監視し、バッファリング状態の遷移を制御するために使用されます。

    ```c
    /* rtpjitterbuffer.c */
    void
    rtp_jitter_buffer_set_delay (RTPJitterBuffer * jbuf, GstClockTime delay)
    {
      jbuf->delay = delay;
      jbuf->low_level = (delay * 15) / 100; // 遅延の15%を低水位
      /* the high level is at 90% in order to release packets before we fill up the
       * buffer up to the latency */
      jbuf->high_level = (delay * 90) / 100; // 遅延の90%を高水位

      GST_DEBUG ("delay %" GST_TIME_FORMAT ", min %" GST_TIME_FORMAT ", max %"
          GST_TIME_FORMAT, GST_TIME_ARGS (jbuf->delay),
          GST_TIME_ARGS (jbuf->low_level), GST_TIME_ARGS (jbuf->high_level));
    }
    ```

  * **タイマー処理での利用**:
    `latency` は、欠落パケットや期待されるパケットのタイムアウトをスケジュールする際にも間接的に影響します。例えば、`do_deadline_timeout` では、最初のパケットに対するデッドラインタイマーが発火した際に `priv->next_seqnum` を更新し、イベントをシグナルして処理ループを進行させます。このデッドラインは `latency` に基づいて設定された時間後にパケットをプッシュするためのものです。
    また、`calculate_expected` 関数内では、大きなギャップがある場合に、`total_duration > priv->latency_ns` のような形で `latency` が欠落パケット数の判断に使われます。

`latency` プロパティは、`rtpjitterbuffer` の動作の根幹に関わる重要なパラメータであり、ネットワーク状況やアプリケーションの要件に応じて適切に調整する必要があります。

> rtpjitterbufferにおけるjbuf->delayとjbuf->low_level, jbuf->high_levelの使用箇所とそれぞれの役割を教えてください。


`rtpjitterbuffer` エレメントにおける `jbuf->delay`、`jbuf->low_level`、および `jbuf->high_level` は、主に `mode` プロパティが `RTP_JITTER_BUFFER_MODE_BUFFER` (バッファリングモード) に設定されている場合の動作を制御するために、内部の `RTPJitterBuffer` 構造体 (`jbuf`) で使用されます。

これらの値は、`rtpjitterbuffer.c` 内の `rtp_jitter_buffer_set_delay` 関数で、`latency` プロパティの値に基づいて設定されます。

```c
/* rtpjitterbuffer.c */
void
rtp_jitter_buffer_set_delay (RTPJitterBuffer * jbuf, GstClockTime delay)
{
  jbuf->delay = delay; // レイテンシプロパティから設定された遅延時間 (ナノ秒)
  jbuf->low_level = (delay * 15) / 100; // 遅延の15%
  /* the high level is at 90% in order to release packets before we fill up the
   * buffer up to the latency */
  jbuf->high_level = (delay * 90) / 100; // 遅延の90%

  GST_DEBUG ("delay %" GST_TIME_FORMAT ", min %" GST_TIME_FORMAT ", max %"
      GST_TIME_FORMAT, GST_TIME_ARGS (jbuf->delay),
      GST_TIME_ARGS (jbuf->low_level), GST_TIME_ARGS (jbuf->high_level));
}
```

それぞれの役割と使用箇所は以下の通りです。

### `jbuf->delay`

  * **役割**:

      * `latency` プロパティによって設定された、ジッターバッファがパケットを保持する目標の最大時間（ナノ秒単位）。
      * バッファリングモードにおいて、この時間分のデータをバッファリングしようとします。
      * 他のモード (`RTP_JITTER_BUFFER_MODE_SLAVE` など) でも、PTS 計算時に `pts + jbuf->delay < dts` のような条件で、出力タイムスタンプが入力DTSに対して早すぎる場合にリセットを行う判断基準の一部として参照されることがあります (下記 `rtp_jitter_buffer_calculate_pts` 内のコード断片を参照)。

  * **主な使用箇所**:

      * **`rtp_jitter_buffer_set_delay` (`rtpjitterbuffer.c`)**: `latency` プロパティから値が設定され、`low_level` と `high_level` もこの値に基づいて計算されます。
      * **`rtp_jitter_buffer_calculate_pts` (`rtpjitterbuffer.c`)**:
        ```c
        /* rtpjitterbuffer.c */
        if (dts != -1 && pts + jbuf->delay < dts) {
          /* if we are going to produce a timestamp that is later than the input
           * timestamp, we need to reset the jitterbuffer. Likely the server paused
           * temporarily */
          GST_DEBUG ("out %" GST_TIME_FORMAT " + %" G_GUINT64_FORMAT " < time %"
              GST_TIME_FORMAT ", reset jitterbuffer", GST_TIME_ARGS (pts),
              jbuf->delay, GST_TIME_ARGS (dts));
          rtp_jitter_buffer_resync (jbuf, dts, gstrtptime, ext_rtptime, TRUE);
          pts = dts;
        }
        ```
        ここでは、計算された出力PTS (`pts`) に `jbuf->delay` を加えた値が、入力DTS (`dts`) よりも小さい場合 (つまり、バッファリングを考慮しても出力が早すぎる場合) に、タイムスタンプの再同期を行うかどうかを判断しています。
      * `gstrtpjitterbuffer.c` の `gst_rtp_jitter_buffer_src_event` 内で、下流からの `GST_EVENT_LATENCY` を処理する際に、バッファリングモードであれば `rtp_jitter_buffer_set_delay` を呼び出して内部の `jbuf->delay` を更新する可能性があります。

### `jbuf->low_level` (低水位マーク)

  * **役割**:

      * バッファリングモード (`RTP_JITTER_BUFFER_MODE_BUFFER`) において、ジッターバッファ内のデータ量がこの値を下回った場合に、再度バッファリングを開始するための閾値です。
      * 通常、`jbuf->delay` の 15% に設定されます。
      * これにより、ネットワーク状態が一時的に改善してバッファが消費された後、再びジッターが増加した場合に備えてバッファを再充填します。

  * **主な使用箇所**:

      * **`rtp_jitter_buffer_set_delay` (`rtpjitterbuffer.c`)**: `jbuf->delay` に基づいて計算・設定されます。
      * **`update_buffer_level` (`rtpjitterbuffer.c`)**:
        ```c
        /* rtpjitterbuffer.c */
        static void
        update_buffer_level (RTPJitterBuffer * jbuf, gint * percent)
        {
          // ... (level の計算) ...

          if (jbuf->buffering) { // 現在バッファリング中の場合
            post = TRUE;
            if (level >= jbuf->high_level) { // バッファレベルが高水位マーク以上に達したら
              GST_DEBUG ("buffering finished");
              jbuf->buffering = FALSE; // バッファリングを終了
            }
          } else { // 現在バッファリング中でない場合
            if (level < jbuf->low_level) { // バッファレベルが低水位マーク未満になったら
              GST_DEBUG ("buffering started");
              jbuf->buffering = TRUE; // バッファリングを開始
              post = TRUE;
            }
          }
          // ... (percent の計算と GstMessage の処理) ...
        }
        ```
        この関数は、バッファ内のデータ量 (`level`) を監視し、`level` が `jbuf->low_level` を下回ると、`jbuf->buffering` フラグを `TRUE` に設定してバッファリング状態に移行します。

### `jbuf->high_level` (高水位マーク)

  * **役割**:

      * バッファリングモード (`RTP_JITTER_BUFFER_MODE_BUFFER`) において、ジッターバッファ内のデータ量がこの値に達した場合に、バッファリングを完了したとみなし、パケットの出力を開始 (または再開) するための閾値です。
      * 通常、`jbuf->delay` の 90% に設定されます。90% に設定されているのは、`latency` で指定された時間ちょうどまで待つのではなく、少し手前で出力を開始することで、バッファが完全に一杯になる前にスムーズな再生を開始するためです。

  * **主な使用箇所**:

      * **`rtp_jitter_buffer_set_delay` (`rtpjitterbuffer.c`)**: `jbuf->delay` に基づいて計算・設定されます。
      * **`update_buffer_level` (`rtpjitterbuffer.c`)**: 上記の `update_buffer_level` 関数のコード断片に示されるように、バッファ内のデータ量 (`level`) が `jbuf->high_level` に達すると、`jbuf->buffering` フラグを `FALSE` に設定してバッファリング状態を解除し、パケットの供給を開始します。
      * **`rtp_jitter_buffer_get_percent` (`rtpjitterbuffer.c`)**: バッファの使用率を計算する際に、`jbuf->high_level` を分母として使用します。

これらの値は、特に `rtpjitterbuffer` が `mode=buffer` (GEnumValue `RTP_JITTER_BUFFER_MODE_BUFFER`) で動作している場合に、動的なバッファリング制御の中心的な役割を果たします。ネットワークの状況に応じてバッファリングを開始したり停止したりすることで、遅延を最小限に抑えつつ、再生の途切れを防ぐことを目指します。他のモード (例: `slave` モード) では、これらの水位マークに基づくバッファリング制御はアクティブにはならず、主に `jbuf->delay` がタイムアウトや PTS 計算の基準として参照されます。

> rtpjitterbufferにおけるlatencyプロパティの機能と処理に与える影響についてソースコードを例示しながら解説してください。

`rtpjitterbuffer` エレメントの `latency` プロパティは、ネットワークジッターを吸収するために RTP パケットをバッファリングする最大時間をミリ秒単位で指定します。この値は、エレメントがパケットをどれくらいの期間保持し、順序の入れ替えや欠落パケットの到着待ちを行うかを決定する上で非常に重要です。

デフォルト値は `200` ミリ秒です。

### `latency` プロパティの機能

1.  **ジッター吸収**: ネットワーク遅延の変動（ジッター）により、パケットの到着間隔が一定でなくなることがあります。`latency` プロパティで設定された時間だけパケットを内部バッファに保持することで、これらの到着時間の揺らぎを平滑化し、より安定したメディアストリームを下流に供給します。

2.  **パケット並べ替えのための待機時間**: RTP パケットはネットワークの状況により順序通りに到着しないことがあります。`latency` は、後続のパケットが到着するのを待つ時間の上限となり、この時間内に到着したパケットはシーケンス番号に基づいて並べ替えられます。

3.  **欠落パケットの判断基準**:

      * `do-lost` プロパティが `TRUE` の場合、パケットが `latency` で設定された時間内に到着しないと「欠落した」と判断されます。
      * この場合、`GstRTPPacketLost` という名前のカスタムイベントが下流に送信されます。

4.  **再送要求のタイムアウト基準 (間接的)**:

      * `do-retransmission` プロパティが `TRUE` の場合、`latency` は再送要求 (`rtx-retry-period`) の期間を決定する際の要素の一つとなり得ます。具体的には、`rtx-retry-period` が自動設定 (-1) の場合、`latency` と観測されたラウンドトリップタイムに基づいて再送を試みる期間が推定されます。

5.  **バッファ管理 (`drop-on-latency`)**:

      * `drop-on-latency` プロパティが `TRUE` の場合、ジッターバッファ内のデータ量が `latency` で指定された時間分を超過しないように、最も古いパケットから破棄します。
      * `gstrtpjitterbuffer.c` の `gst_rtp_jitter_buffer_chain` 関数内で、`drop-on-latency` が有効で、かつ内部バッファ (`priv->jbuf`) の保持期間 (`rtp_jitter_buffer_get_ts_diff`) が設定された `latency_ns` を超えた場合に、古いアイテムをドロップする処理があります。

    <!-- end list -->

    ```c
    /* gstrtpjitterbuffer.c */
    if (priv->latency_ms && priv->drop_on_latency) {
        latency_ts =
            gst_util_uint64_scale_int (priv->latency_ms, priv->clock_rate, 1000);

        if (G_UNLIKELY (rtp_jitter_buffer_get_ts_diff (priv->jbuf) >= latency_ts)) {
          RTPJitterBufferItem *old_item;

          old_item = rtp_jitter_buffer_peek (priv->jbuf);

          if (IS_DROPABLE (old_item)) { // IS_DROPABLE はバッファかLOSTイベントかをチェック
            old_item = rtp_jitter_buffer_pop (priv->jbuf, &percent);
            GST_DEBUG_OBJECT (jitterbuffer, "Queue full, dropping old packet %p",
                old_item);
            priv->next_seqnum = (old_item->seqnum + old_item->count) & 0xffff;
            free_item (old_item); // free_item でバッファを解放
          }
          /* we might have removed some head buffers, signal the pushing thread to
           * see if it can push now */
          JBUF_SIGNAL_EVENT (priv);
        }
      }
    ```

### `latency` プロパティが処理に与える影響

  * **遅延の増加**: `latency` の値が大きいほど、パイプライン全体の遅延は増加します。これは、パケットをより長くバッファリングするためです。リアルタイム性が重要なアプリケーションでは、この値を適切に設定する必要があります。
  * **ジッター耐性の向上**: `latency` の値が大きいほど、より大きなネットワークジッターを吸収できます。これにより、ネットワーク状況が不安定な場合でも、スムーズな再生が期待できます。
  * **パケットロスと品質**:
      * `latency` が小さい場合、パケットが少し遅れただけでも「欠落」と判断されやすくなり、結果として画質や音質が低下する可能性があります（特に `do-lost` が有効で下流が欠損処理をしない場合）。
      * `latency` が大きい場合、より多くのパケットの到着を待つため、欠落と判断されるパケットは減りますが、遅延が増加します。
  * **リソース消費**: バッファリングするパケットが増えるため、`latency` を大きくするとメモリ使用量が増加する可能性があります。

### ソースコードにおける `latency` の設定と使用

  * **プロパティ定義 (`gstrtpjitterbuffer.c`)**:
    `latency` プロパティは `g_object_class_install_property` を通じて定義され、デフォルト値は `DEFAULT_LATENCY_MS` (200ミリ秒) です。

    ```c
    /* gstrtpjitterbuffer.c */
    g_object_class_install_property (gobject_class, PROP_LATENCY,
        g_param_spec_uint ("latency", "Buffer latency in ms",
            "Amount of ms to buffer", 0, G_MAXUINT, DEFAULT_LATENCY_MS,
            G_PARAM_READWRITE | G_PARAM_STATIC_STRINGS));
    ```

  * **プロパティ設定時 (`gst_rtp_jitter_buffer_set_property`)**:
    `latency` プロパティが設定されると、`priv->latency_ms` と `priv->latency_ns` (ナノ秒単位) が更新され、内部の `RTPJitterBuffer` オブジェクトにも `rtp_jitter_buffer_set_delay` を通じてナノ秒単位の遅延時間が設定されます。

    ```c
    /* gstrtpjitterbuffer.c */
    static void
    gst_rtp_jitter_buffer_set_property (GObject * object,
        guint prop_id, const GValue * value, GParamSpec * pspec)
    {
      GstRtpJitterBuffer *jitterbuffer;
      GstRtpJitterBufferPrivate *priv;

      jitterbuffer = GST_RTP_JITTER_BUFFER (object);
      priv = jitterbuffer->priv;

      switch (prop_id) {
        case PROP_LATENCY:
        {
          guint new_latency, old_latency;

          new_latency = g_value_get_uint (value);

          JBUF_LOCK (priv); // ジッターバッファのロック
          old_latency = priv->latency_ms;
          priv->latency_ms = new_latency;
          priv->latency_ns = priv->latency_ms * GST_MSECOND; // ミリ秒からナノ秒へ変換
          rtp_jitter_buffer_set_delay (priv->jbuf, priv->latency_ns); // 内部ジッターバッファへ設定
          JBUF_UNLOCK (priv); // ロック解除

          /* post message if latency changed, this will inform the parent pipeline
           * that a latency reconfiguration is possible/needed. */
          if (new_latency != old_latency) {
            GST_DEBUG_OBJECT (jitterbuffer, "latency changed to: %" GST_TIME_FORMAT,
                GST_TIME_ARGS (new_latency * GST_MSECOND));

            gst_element_post_message (GST_ELEMENT_CAST (jitterbuffer),
                gst_message_new_latency (GST_OBJECT_CAST (jitterbuffer)));
          }
          break;
        }
        // ... (他のプロパティ処理)
      }
    }
    ```

  * **内部 `RTPJitterBuffer` での遅延設定 (`rtpjitterbuffer.c`)**:
    `rtp_jitter_buffer_set_delay` 関数は、`jbuf->delay` を設定するとともに、バッファリングモード (`RTP_JITTER_BUFFER_MODE_BUFFER`) で使用される低水位 (`low_level`) および高水位 (`high_level`) の閾値を計算します。これらはバッファの充填率を監視し、バッファリング状態の遷移を制御するために使用されます。

    ```c
    /* rtpjitterbuffer.c */
    void
    rtp_jitter_buffer_set_delay (RTPJitterBuffer * jbuf, GstClockTime delay)
    {
      jbuf->delay = delay;
      jbuf->low_level = (delay * 15) / 100; // 遅延の15%を低水位
      /* the high level is at 90% in order to release packets before we fill up the
       * buffer up to the latency */
      jbuf->high_level = (delay * 90) / 100; // 遅延の90%を高水位

      GST_DEBUG ("delay %" GST_TIME_FORMAT ", min %" GST_TIME_FORMAT ", max %"
          GST_TIME_FORMAT, GST_TIME_ARGS (jbuf->delay),
          GST_TIME_ARGS (jbuf->low_level), GST_TIME_ARGS (jbuf->high_level));
    }
    ```

  * **タイマー処理での利用**:
    `latency` は、欠落パケットや期待されるパケットのタイムアウトをスケジュールする際にも間接的に影響します。例えば、`do_deadline_timeout` では、最初のパケットに対するデッドラインタイマーが発火した際に `priv->next_seqnum` を更新し、イベントをシグナルして処理ループを進行させます。このデッドラインは `latency` に基づいて設定された時間後にパケットをプッシュするためのものです。
    また、`calculate_expected` 関数内では、大きなギャップがある場合に、`total_duration > priv->latency_ns` のような形で `latency` が欠落パケット数の判断に使われます。

`latency` プロパティは、`rtpjitterbuffer` の動作の根幹に関わる重要なパラメータであり、ネットワーク状況やアプリケーションの要件に応じて適切に調整する必要があります。


> rtpjitterbufferにおけるjbuf->delayとjbuf->low_level, jbuf->high_levelの使用箇所とそれぞれの役割を教えてください。

`rtpjitterbuffer` エレメントにおける `jbuf->delay`、`jbuf->low_level`、および `jbuf->high_level` は、主に `mode` プロパティが `RTP_JITTER_BUFFER_MODE_BUFFER` (バッファリングモード) に設定されている場合の動作を制御するために、内部の `RTPJitterBuffer` 構造体 (`jbuf`) で使用されます。

これらの値は、`rtpjitterbuffer.c` 内の `rtp_jitter_buffer_set_delay` 関数で、`latency` プロパティの値に基づいて設定されます。

```c
/* rtpjitterbuffer.c */
void
rtp_jitter_buffer_set_delay (RTPJitterBuffer * jbuf, GstClockTime delay)
{
  jbuf->delay = delay; // レイテンシプロパティから設定された遅延時間 (ナノ秒)
  jbuf->low_level = (delay * 15) / 100; // 遅延の15%
  /* the high level is at 90% in order to release packets before we fill up the
   * buffer up to the latency */
  jbuf->high_level = (delay * 90) / 100; // 遅延の90%

  GST_DEBUG ("delay %" GST_TIME_FORMAT ", min %" GST_TIME_FORMAT ", max %"
      GST_TIME_FORMAT, GST_TIME_ARGS (jbuf->delay),
      GST_TIME_ARGS (jbuf->low_level), GST_TIME_ARGS (jbuf->high_level));
}
```

それぞれの役割と使用箇所は以下の通りです。

### `jbuf->delay`

  * **役割**:

      * `latency` プロパティによって設定された、ジッターバッファがパケットを保持する目標の最大時間（ナノ秒単位）。
      * バッファリングモードにおいて、この時間分のデータをバッファリングしようとします。
      * 他のモード (`RTP_JITTER_BUFFER_MODE_SLAVE` など) でも、PTS 計算時に `pts + jbuf->delay < dts` のような条件で、出力タイムスタンプが入力DTSに対して早すぎる場合にリセットを行う判断基準の一部として参照されることがあります (下記 `rtp_jitter_buffer_calculate_pts` 内のコード断片を参照)。

  * **主な使用箇所**:

      * **`rtp_jitter_buffer_set_delay` (`rtpjitterbuffer.c`)**: `latency` プロパティから値が設定され、`low_level` と `high_level` もこの値に基づいて計算されます。
      * **`rtp_jitter_buffer_calculate_pts` (`rtpjitterbuffer.c`)**:
        ```c
        /* rtpjitterbuffer.c */
        if (dts != -1 && pts + jbuf->delay < dts) {
          /* if we are going to produce a timestamp that is later than the input
           * timestamp, we need to reset the jitterbuffer. Likely the server paused
           * temporarily */
          GST_DEBUG ("out %" GST_TIME_FORMAT " + %" G_GUINT64_FORMAT " < time %"
              GST_TIME_FORMAT ", reset jitterbuffer", GST_TIME_ARGS (pts),
              jbuf->delay, GST_TIME_ARGS (dts));
          rtp_jitter_buffer_resync (jbuf, dts, gstrtptime, ext_rtptime, TRUE);
          pts = dts;
        }
        ```
        ここでは、計算された出力PTS (`pts`) に `jbuf->delay` を加えた値が、入力DTS (`dts`) よりも小さい場合 (つまり、バッファリングを考慮しても出力が早すぎる場合) に、タイムスタンプの再同期を行うかどうかを判断しています。
      * `gstrtpjitterbuffer.c` の `gst_rtp_jitter_buffer_src_event` 内で、下流からの `GST_EVENT_LATENCY` を処理する際に、バッファリングモードであれば `rtp_jitter_buffer_set_delay` を呼び出して内部の `jbuf->delay` を更新する可能性があります。

### `jbuf->low_level` (低水位マーク)

  * **役割**:

      * バッファリングモード (`RTP_JITTER_BUFFER_MODE_BUFFER`) において、ジッターバッファ内のデータ量がこの値を下回った場合に、再度バッファリングを開始するための閾値です。
      * 通常、`jbuf->delay` の 15% に設定されます。
      * これにより、ネットワーク状態が一時的に改善してバッファが消費された後、再びジッターが増加した場合に備えてバッファを再充填します。

  * **主な使用箇所**:

      * **`rtp_jitter_buffer_set_delay` (`rtpjitterbuffer.c`)**: `jbuf->delay` に基づいて計算・設定されます。
      * **`update_buffer_level` (`rtpjitterbuffer.c`)**:
        ```c
        /* rtpjitterbuffer.c */
        static void
        update_buffer_level (RTPJitterBuffer * jbuf, gint * percent)
        {
          // ... (level の計算) ...

          if (jbuf->buffering) { // 現在バッファリング中の場合
            post = TRUE;
            if (level >= jbuf->high_level) { // バッファレベルが高水位マーク以上に達したら
              GST_DEBUG ("buffering finished");
              jbuf->buffering = FALSE; // バッファリングを終了
            }
          } else { // 現在バッファリング中でない場合
            if (level < jbuf->low_level) { // バッファレベルが低水位マーク未満になったら
              GST_DEBUG ("buffering started");
              jbuf->buffering = TRUE; // バッファリングを開始
              post = TRUE;
            }
          }
          // ... (percent の計算と GstMessage の処理) ...
        }
        ```
        この関数は、バッファ内のデータ量 (`level`) を監視し、`level` が `jbuf->low_level` を下回ると、`jbuf->buffering` フラグを `TRUE` に設定してバッファリング状態に移行します。

### `jbuf->high_level` (高水位マーク)

  * **役割**:

      * バッファリングモード (`RTP_JITTER_BUFFER_MODE_BUFFER`) において、ジッターバッファ内のデータ量がこの値に達した場合に、バッファリングを完了したとみなし、パケットの出力を開始 (または再開) するための閾値です。
      * 通常、`jbuf->delay` の 90% に設定されます。90% に設定されているのは、`latency` で指定された時間ちょうどまで待つのではなく、少し手前で出力を開始することで、バッファが完全に一杯になる前にスムーズな再生を開始するためです。

  * **主な使用箇所**:

      * **`rtp_jitter_buffer_set_delay` (`rtpjitterbuffer.c`)**: `jbuf->delay` に基づいて計算・設定されます。
      * **`update_buffer_level` (`rtpjitterbuffer.c`)**: 上記の `update_buffer_level` 関数のコード断片に示されるように、バッファ内のデータ量 (`level`) が `jbuf->high_level` に達すると、`jbuf->buffering` フラグを `FALSE` に設定してバッファリング状態を解除し、パケットの供給を開始します。
      * **`rtp_jitter_buffer_get_percent` (`rtpjitterbuffer.c`)**: バッファの使用率を計算する際に、`jbuf->high_level` を分母として使用します。

これらの値は、特に `rtpjitterbuffer` が `mode=buffer` (GEnumValue `RTP_JITTER_BUFFER_MODE_BUFFER`) で動作している場合に、動的なバッファリング制御の中心的な役割を果たします。ネットワークの状況に応じてバッファリングを開始したり停止したりすることで、遅延を最小限に抑えつつ、再生の途切れを防ぐことを目指します。他のモード (例: `slave` モード) では、これらの水位マークに基づくバッファリング制御はアクティブにはならず、主に `jbuf->delay` がタイムアウトや PTS 計算の基準として参照されます。

> rtpjitterbufferのlatencyプロパティによる出力バッファの遅延とPTSの関係を教えてください。

`rtpjitterbuffer` の `latency` プロパティは、出力バッファの**実際の遅延時間 (出力されるタイミング)** と、そのバッファに付与される **PTS (Presentation Time Stamp)** の両方に影響を与えます。この2つは密接に関連していますが、同一ではありません。

以下にその関係性を解説します。

### 1\. 出力バッファの遅延 (出力タイミング) への影響

  * **基本的な役割**: `latency` プロパティは、`rtpjitterbuffer` がRTPパケットを内部に保持する目標時間を定義します。つまり、受信したパケットは、この `latency` の期間が経過するまで（あるいは他の条件が満たされるまで）出力されません。
  * **ジッター吸収と並べ替え**: この遅延時間を利用して、ネットワークジッター（パケット到着時間の揺らぎ）を吸収し、順序が入れ替わって到着したパケットを正しい順序に並べ替えます。
  * **パケットのリリース**:
      * 通常、パケットは `latency` の期間が経過するのを待ってから、適切なタイミングで下流にプッシュされます。
      * `do-lost=true` の場合、`latency` を超えても到着しないパケットは欠落とみなされ、そのタイミングで（もしキューの先頭にあれば）次の処理（欠落イベントの発行や後続パケットのプッシュ）に進みます。
      * `faststart-min-packets` が設定されている場合、`latency` を待たずに、指定された数の連続したパケットが到着すれば、より早くパケットの出力を開始することがあります。この場合でも、PTSの計算ロジックは維持されますが、実際の出力タイミングが早まります。

ソースコードでは、主に `gstrtpjitterbuffer.c` の `gst_rtp_jitter_buffer_loop` 関数が、内部のタイマー (`wait_next_timeout` で管理) やイベントに基づいて、バッファをいつ下流にプッシュするかを決定しています。このタイマーの期限切れのタイミングは、`latency` (や `rtx-retry-period` など他の設定) に影響を受けます。

```c
/* gstrtpjitterbuffer.c の wait_next_timeout 関数内 */
// (len は priv->timers の長さ)
for (i = 0; i < len;) {
  TimerData *test = &g_array_index (priv->timers, TimerData, i);
  // get_timeout は latency_ns (latency プロパティのナノ秒表現) を考慮してタイムアウト値を計算
  GstClockTime test_timeout = get_timeout (jitterbuffer, test);
  // ...
  if (test->type == TIMER_TYPE_LOST &&
      (test_timeout == -1 || test_timeout <= now)) {
    // ... 欠落タイムアウト処理 ...
  }
  // ...
  if (timer_timeout == -1 || timer_timeout <= now || priv->eos) {
    // ... 即時タイムアウト処理 (DEADLINE や EOS など) ...
  }
  // ...
  // gst_clock_new_single_shot_id と gst_clock_id_wait で実際の遅延を発生させる
  id = priv->clock_id = gst_clock_new_single_shot_id (clock, sync_time);
  // ...
  ret = gst_clock_id_wait (id, &clock_jitter);
  // ...
}
```

この `sync_time` の計算には、タイマーの種類に応じて `latency` が考慮された `timer_timeout` が使われます。

### 2\. 出力バッファの PTS への影響

`latency` プロパティの値が直接的に出力PTSの計算式に含まれるわけではありませんが、間接的に影響を与えます。

  * **PTS計算の基本**:
      * `rtpjitterbuffer` のPTS計算 (`rtp_jitter_buffer_calculate_pts` 関数) は、主に受信パケットのRTPタイムスタンプと、そのパケットの到着時刻 (DTS)、そしてエレメントのベースタイム (`jbuf->base_time`) やRTPベースタイム (`jbuf->base_rtptime`) を用いて行われます。
      * 目標は、送信側で意図された提示時刻を、受信側で可能な限り正確に再現することです。
  * **`latency` の間接的な役割**:
      * **基準時刻の安定化**: `latency` の期間バッファリングすることで、初期のパケット到着の揺らぎの影響を受けにくくなり、PTS計算の基準となる `jbuf->base_time` や `jbuf->base_rtptime` がより安定した値で設定されることに寄与します。
      * **スキュー計算への影響**: `mode=slave` の場合、送信側と受信側のクロックスキューを計算します。`latency` によってバッファリングされることで、より多くのパケットサンプルに基づいてスキューが計算され、結果としてPTSの精度向上に繋がる可能性があります。
      * **タイムスタンプの単調増加**: `rtp_jitter_buffer_calculate_pts` 内では、計算されたPTSが前の出力PTSよりも逆行しないように調整するロジックがあります。
        ```c
        /* rtpjitterbuffer.c の rtp_jitter_buffer_calculate_pts 関数内 */
        if (G_LIKELY (pts != -1 && jbuf->prev_out_time != -1
                && jbuf->prev_send_diff != -1)) {
          if (G_UNLIKELY (
                  /* if the server timestamps went up and the out_time backwards */
                  (gstrtptime - jbuf->base_rtptime > jbuf->prev_send_diff
                      && pts < jbuf->prev_out_time) ||
                  /* if the server timestamps went backwards and the out_time forwards */
                  (gstrtptime - jbuf->base_rtptime < jbuf->prev_send_diff
                      && pts > jbuf->prev_out_time) ||
                  /* if the server timestamps did not change */
                  gstrtptime - jbuf->base_rtptime == jbuf->prev_send_diff)) {
            GST_DEBUG ("backwards timestamps, using previous time");
            pts = jbuf->prev_out_time; // 前の出力時刻を使用
          }
        }
        ```
        `latency` が適切に設定されていると、このような逆行が発生しにくくなり、よりスムーズなPTS列が生成されます。
      * **リセット条件**: `rtp_jitter_buffer_calculate_pts` 内で、計算されたPTSに `jbuf->delay` (latencyプロパティから設定) を加えた値が、入力DTSよりも著しく早い場合 (サーバーが一時停止した可能性など)、ジッターバッファをリセットし、PTSをDTSに設定する処理があります。
        ```c
        /* rtpjitterbuffer.c の rtp_jitter_buffer_calculate_pts 関数内 */
        if (dts != -1 && pts + jbuf->delay < dts) {
          GST_DEBUG ("out %" GST_TIME_FORMAT " + %" G_GUINT64_FORMAT " < time %"
              GST_TIME_FORMAT ", reset jitterbuffer", GST_TIME_ARGS (pts),
              jbuf->delay, GST_TIME_ARGS (dts));
          rtp_jitter_buffer_resync (jbuf, dts, gstrtptime, ext_rtptime, TRUE);
          pts = dts;
        }
        ```
        この `jbuf->delay` は `latency` プロパティから来ています。このチェックは、過度なバッfaリングによる不自然なタイムスタンプのずれを防ぎます。

### まとめ

  * **出力遅延**: `latency` プロパティは、出力バッファが実際に下流にプッシュされるまでの**物理的な遅延時間**を直接的に決定します。これによりジッターを吸収します。
  * **PTS**:
      * `latency` は、出力バッファのPTS計算式に直接項として現れるわけではありません。PTSは主にRTPタイムスタンプと到着時刻に基づいて計算されます。
      * しかし、`latency` によるバッファリングは、PTS計算の基準となる値の安定化や、スキュー計算の精度向上に寄与し、結果としてより正確で滑らかなPTS列の生成を助けます。
      * また、PTSが入力DTSに対して早すぎる場合の補正処理において、`latency` (内部的には `jbuf->delay`) が判断基準として使われます。

したがって、`latency` プロパティは、単に出力タイミングを遅らせるだけでなく、タイムスタンプの連続性や正確性にも間接的に良い影響を与えるように設計されています。ただし、`latency` を大きくしすぎると、全体の遅延が許容範囲を超える可能性があるため、適切なバランスが求められます。

> rtpjitterbufferのlatencyプロパティが影響するタイマーの処理をソースコードを例示しながら解説してください。

`rtpjitterbuffer` の `latency` プロパティは、内部で管理されるタイマーの動作にいくつかの形で影響を与えます。これらのタイマーは、パケットの到着遅延、欠落、再送要求などを処理するために使用されます。

`latency` プロパティの値 (`priv->latency_ns` としてナノ秒単位で保持) は、主に以下のタイマー関連の処理で考慮されます。

1.  **デッドラインタイマー (初期バッファリングおよび最初のパケット出力)**

      * 最初にパケットを受信した際、または大きなギャップの後にストリームが再開された際、`rtpjitterbuffer` はすぐにはパケットを出力しません。代わりに、「デッドラインタイマー」を設定します。

      * このタイマーのタイムアウト値は、最初の有効なパケットの計算済みPTSに `priv->latency_ns` を加えた時刻になることが一般的です。これにより、`latency` で指定された時間だけバッファリングを行い、その間に後続のパケットが到着して順序が整うのを待ちます。

      * `gstrtpjitterbuffer.c` の `gst_rtp_jitter_buffer_chain` 関数内で、最初のパケット (`expected == -1` の場合) に対して `set_timer` が呼び出され、`TIMER_TYPE_DEADLINE` のタイマーが設定されます。このタイマーの `timeout` は、`rtp_jitter_buffer_calculate_pts` で計算されたPTSです。

        ```c
        /* gstrtpjitterbuffer.c の gst_rtp_jitter_buffer_chain 関数内 */
        if (G_UNLIKELY (expected == -1)) { // expected は priv->next_in_seqnum
          GST_DEBUG_OBJECT (jitterbuffer, "First buffer #%d", seqnum);

          /* calculate a pts based on rtptime and arrival time (dts) */
          pts =
              rtp_jitter_buffer_calculate_pts (priv->jbuf, dts, estimated_dts,
              rtptime, gst_element_get_base_time (GST_ELEMENT_CAST (jitterbuffer)));

          /* we don't know what the next_in_seqnum should be, wait for the last
           * possible moment to push this buffer, maybe we get an earlier seqnum
           * while we wait */
          // この set_timer で TIMER_TYPE_DEADLINE が設定される。
          // timeout 値は計算された pts。実際のタイムアウト処理は wait_next_timeout で latency を考慮。
          set_timer (jitterbuffer, TIMER_TYPE_DEADLINE, seqnum, pts);

          do_next_seqnum = TRUE;
          /* take rtptime and pts to calculate packet spacing */
          priv->ips_rtptime = rtptime;
          priv->ips_pts = pts;
        }
        ```

      * そして、`wait_next_timeout` 関数内でこのタイマーが処理される際、`get_timeout` 関数が呼び出されます。`TIMER_TYPE_DEADLINE` の場合、計算されたPTS (`timer->timeout`) に `priv->latency_ns` が加算されて実際のタイムアウト時刻 (`test_timeout`) が決定されます。

        ```c
        /* gstrtpjitterbuffer.c の get_timeout 関数内 */
        static GstClockTime
        get_timeout (GstRtpJitterBuffer * jitterbuffer, TimerData * timer)
        {
          GstRtpJitterBufferPrivate *priv = jitterbuffer->priv;
          GstClockTime test_timeout;

          if ((test_timeout = timer->timeout) == -1) // timer->timeout は pts
            return -1;

          if (timer->type != TIMER_TYPE_EXPECTED) { // DEADLINE や LOST の場合
            /* add our latency and offset to get output times. */
            test_timeout = apply_offset (jitterbuffer, test_timeout);
            test_timeout += priv->latency_ns; // ここで latency が加算される
          }
          return test_timeout;
        }
        ```

      * このタイマーが満了すると (`do_deadline_timeout` が呼ばれる)、`priv->next_seqnum` が確定し、パケットの出力処理が開始されます。

2.  **欠落パケットタイマー (`TIMER_TYPE_LOST`)**

      * パケットが期待通りに到着せず、ギャップが検出された場合、その欠落パケットに対するタイマーが設定されます。

      * `do-lost=TRUE` の場合、このタイマーが満了すると（つまり、`latency` で設定された期間を考慮してもパケットが到着しないと判断されると）、`GstRTPPacketLost` イベントが生成され、内部的に `ITEM_TYPE_LOST` としてキューに追加されます。

      * `gstrtpjitterbuffer.c` の `calculate_expected` 関数では、ギャップがある場合に `add_timer` を呼び出し、`TIMER_TYPE_LOST` (または `do-retransmission=TRUE` なら `TIMER_TYPE_EXPECTED`) のタイマーを設定します。

      * このタイマーのタイムアウト計算 (`get_timeout` 内) も、上記と同様に `priv->latency_ns` を考慮します。これにより、`latency` の期間だけパケットの到着を待ってから「欠落」と判断します。

        ```c
        /* gstrtpjitterbuffer.c の calculate_expected 関数内 */
        if (priv->do_retransmission) {
            // ... type = TIMER_TYPE_EXPECTED; ...
        } else {
            type = TIMER_TYPE_LOST; // do_retransmission=FALSE なら LOST タイマー
        }
        // ...
        while (gst_rtp_buffer_compare_seqnum (expected, seqnum) > 0) {
            add_timer (jitterbuffer, type, expected, 0, expected_pts, delay, duration);
            // ...
        }
        ```

      * `wait_next_timeout` 内でこの `TIMER_TYPE_LOST` のタイマーが満了すると (`do_lost_timeout` が呼ばれる)、パケットが実際に失われたと判断され、対応する処理が行われます。

3.  **再送要求期間 (`rtx-retry-period`) の自動計算への影響**

      * `do-retransmission=TRUE` かつ `rtx-retry-period` プロパティが `-1` (自動) に設定されている場合、`rtpjitterbuffer` は再送要求を試みる期間を推定します。

      * この推定において、`priv->latency_ns` が考慮されます。具体的には、`get_rtx_retry_period` 関数内で、観測されたRTT (Round Trip Time) が `latency_ns` より大きい場合は再送期間を0とし、そうでなければ `latency_ns` からRTTを引いた値を再送期間とします。

        ```c
        /* gstrtpjitterbuffer.c の get_rtx_retry_period 関数内 */
        static GstClockTime
        get_rtx_retry_period (GstRtpJitterBufferPrivate * priv,
            GstClockTime rtx_retry_timeout)
        {
          GstClockTime rtx_retry_period;

          if (priv->rtx_retry_period == -1) { // rtx-retry-period が自動設定の場合
            /* we retry up to the configured jitterbuffer size but leaving some
             * room for the retransmission to arrive in time */
            if (rtx_retry_timeout > priv->latency_ns) { // RTT が latency より大きい
              rtx_retry_period = 0;
            } else {
              rtx_retry_period = priv->latency_ns - rtx_retry_timeout; // latency から RTT を引く
            }
          } else {
            rtx_retry_period = priv->rtx_retry_period * GST_MSECOND;
          }
          return rtx_retry_period;
        }
        ```

      * これにより、`latency` が大きいほど、より長い期間、欠落パケットの再送を試みることができます。

4.  **`drop-on-latency` の動作**

      * `drop-on-latency=TRUE` の場合、ジッターバッファ内のRTPタイムスタンプの範囲 (`rtp_jitter_buffer_get_ts_diff`) が `latency` プロパティで指定された時間 (`latency_ts`) を超えると、最も古いパケットが破棄されます。

      * このチェックは `gst_rtp_jitter_buffer_chain` 関数内で行われ、直接的なタイマー処理ではありませんが、`latency` に基づいてバッファの最大保持期間を制限するものです。

        ```c
        /* gstrtpjitterbuffer.c の gst_rtp_jitter_buffer_chain 関数内 */
        if (priv->latency_ms && priv->drop_on_latency) {
            latency_ts = // latency_ms を clock_rate に基づいて RTP タイムスタンプ単位に変換
                gst_util_uint64_scale_int (priv->latency_ms, priv->clock_rate, 1000);

            if (G_UNLIKELY (rtp_jitter_buffer_get_ts_diff (priv->jbuf) >= latency_ts)) {
              // バッファ内の期間が latency_ts を超えたら古いパケットをドロップ
              // ... (ドロップ処理) ...
            }
        }
        ```

これらの処理を通じて、`latency` プロパティは、パケットがいつ出力されるか、いつ欠落と見なされるか、そして再送がどの程度の期間試みられるかといった、`rtpjitterbuffer` の時間に関連する多くの側面に影響を与えます。

> rtpjitterbufferが出力バッファに設定したPTSが下流のエレメントで無視される場合、latencyプロパティで設定された遅延時間はどのような影響を与えますか。

`rtpjitterbuffer` が出力バッファに設定したPTS (Presentation Time Stamp) が、下流のエレメント (例: デコーダやシンク) で**完全に無視される**というシナリオを考えます。この場合、`latency` プロパティで設定された遅延時間は、主に以下の点でパイプライン全体の動作に影響を与えます。

1.  **出力バッファの物理的なタイミング**:

      * たとえPTSが無視されたとしても、`rtpjitterbuffer` は `latency` プロパティに基づいて、受信したRTPパケットを一定時間内部で保持（バッファリング）します。
      * つまり、`rtpjitterbuffer` からバッファが下流にプッシュされる**物理的なタイミング**は、`latency` の設定値によって遅延します。
      * 下流のエレメントがPTSを見ずに、到着した順にバッファを処理する場合、`latency` が大きいほど、最初のデータが表示/再生されるまでの初期遅延が大きくなります。

2.  **ジッター吸収とパケット並べ替えの恩恵**:

      * PTSが無視される場合でも、`latency` によって確保されたバッファリング期間は、ネットワークジッターの吸収や、順序が入れ替わったパケットの並べ替えに役立ちます。

      * これにより、下流のエレメントが受信するバッファの順序はより正しくなり、データ自体の連続性が向上する可能性があります。例えば、ビデオフレームが正しい順序でデコーダに渡れば、表示の乱れを防ぐことができます。

      * `gstrtpjitterbuffer.c` の `gst_rtp_jitter_buffer_chain` 関数内でパケットが内部の `RTPJitterBuffer` (`priv->jbuf`) に挿入される際、シーケンス番号に基づいてソートされます。このソート処理が有効に働くためには、ある程度の `latency` が必要です。

        ```c
        /* gstrtpjitterbuffer.c の gst_rtp_jitter_buffer_chain 関数内 */
        // ... (seqnum, rtptime などの取得) ...
        if (G_UNLIKELY (!rtp_jitter_buffer_insert (priv->jbuf, item, &head,
                          &percent))) {
            // ... (重複パケット処理) ...
        }
        ```

        `rtp_jitter_buffer_insert` (実際には `rtpjitterbuffer.c` の `rtp_jitter_buffer_insert`) は、シーケンス番号順にアイテムを挿入します。

3.  **欠落パケットの処理 (`do-lost`)**:

      * `do-lost` プロパティが `TRUE` の場合、`latency` の期間内に到着しなかったパケットは欠落とみなされ、`GstRTPPacketLost` イベントが発行されます。
      * 下流のエレメントがPTSを無視するとしても、この欠落イベント自体は受信する可能性があります。デペイローダーなどがこのイベントを解釈して、欠落パケットの補間処理（例えば、音声なら無音挿入、映像なら前フレームの表示維持など）を行う場合、`latency` の設定は「いつ欠落と判断するか」の基準として依然として重要です。
      * `wait_next_timeout` 関数内で `TIMER_TYPE_LOST` のタイマーが処理される際 (`do_lost_timeout` が呼ばれる)、`latency` を考慮したタイムアウト計算がなされています。

4.  **再送要求 (`do-retransmission`)**:

      * `do-retransmission` プロパティが `TRUE` の場合、パケットが期待通りに到着しないと判断された際に再送要求イベントがアップストリームに送信されます。
      * この「期待通りに到着しない」の判断や、再送を試みる期間 (`rtx-retry-period` の自動計算) には、`latency` が間接的に関わっています。
      * 下流がPTSを無視するとしても、`rtpjitterbuffer` が欠落パケットを再送によって取得できれば、下流に渡されるデータの完全性は向上します。`latency` の設定は、再送パケットが到着するのを待つ時間的な余裕に影響します。

5.  **パイプライン全体のライブ性 (Live Synchronization)**:

      * `rtpjitterbuffer` はライブエレメントとして動作し、パイプラインのレイテンシ計算に自身の `latency` を加算します。

      * 下流がPTSを無視していても、GStreamerのパイプライン全体の同期メカニズム（クロック同期やライブエレメント間のレイテンシネゴシエーション）は、`rtpjitterbuffer` が報告する `latency` を考慮する場合があります。これにより、例えばオーディオとビデオの同期（リップシンク）が、PTSとは別のレベルで行われようとする場合、`latency` がその挙動に影響を与える可能性があります。

      * `gstrtpjitterbuffer.c` の `gst_rtp_jitter_buffer_src_query` 関数内で `GST_QUERY_LATENCY` を処理する際、自身の `latency_ns` をピアレイテンシに加算して報告しています。

        ```c
        /* gstrtpjitterbuffer.c の gst_rtp_jitter_buffer_src_query 関数内 */
        case GST_QUERY_LATENCY:
        {
          // ... (ピアレイテンシを取得) ...
          JBUF_LOCK (priv);
          priv->peer_latency = min_latency;
          our_latency = priv->latency_ns; // ここで自身の latency を使用
          JBUF_UNLOCK (priv);

          min_latency += our_latency; // 自身の latency を加算
          max_latency = -1;

          gst_query_set_latency (query, TRUE, min_latency, max_latency);
          // ...
        }
        ```

### まとめ

下流のエレメントが `rtpjitterbuffer` の出力PTSを無視する場合でも、`latency` プロパティは以下の点で影響を与え続けます。

  * **データの到着タイミング**: パケットが物理的に下流に渡されるタイミングが遅延します。
  * **データの品質**: ジッター吸収とパケット並べ替えによって、下流に渡されるデータの順序性と連続性が向上する可能性があります。
  * **欠落・再送処理**: パケットがいつ欠落したと判断されるか、いつ再送要求が行われるかの基準時間に影響し、結果的に下流に渡るデータの完全性に影響します。
  * **パイプライン同期**: GStreamerフレームワークレベルでの同期（特にライブソースの場合）に影響を与える可能性があります。

したがって、たとえPTSが直接利用されないシナリオであっても、`latency` プロパティはストリームの安定性や品質、そしてパイプライン全体の挙動に対して重要な役割を果たします。

> latencyプロパティがパイプラインに影響を与えないという現象はどのような条件の場合発生しますか。

`rtpjitterbuffer` の `latency` プロパティは、受信したRTPパケットを処理し、下流に送り出す際の遅延時間やタイミング制御に深く関わっています。しかし、特定の条件下では、この `latency` プロパティがパイプラインの**体感的な遅延や最終的な出力品質にほとんど影響を与えない、あるいは意図した通りに機能しない**ように見える現象が発生し得ます。

いくつかの条件を以下に挙げます。

1.  **ネットワーク状態が極めて良好な場合 (ジッターやパケットロスがほぼ無い)**:

      * `latency` プロパティの主な目的の一つは、ネットワークの揺らぎ（ジッター）を吸収し、パケットの順序を整えることです。
      * ネットワークが非常に安定しており、パケットがほぼ時間通りに、かつ正しい順序で到着する場合、`rtpjitterbuffer` が行うべき補正処理はほとんどありません。
      * このような状況では、`latency` を大きな値に設定しても、実際にその時間分だけパケットが保持されるわけではなく、`faststart-min-packets` の条件を満たせば、または他のタイマーロジックにより、比較的早期にパケットが出力される可能性があります。結果として、`latency` の設定値が実際の遅延に直結しないように見えることがあります。
      * ソースコードの `gst_rtp_jitter_buffer_fast_start` 関数や、`wait_next_timeout` 内の各種タイマー処理がこれに関わります。特にデッドラインタイマーは、最初のパケット群が揃えば `latency` 期間待たずに動き出す可能性があります。

2.  **`mode` プロパティが `RTP_JITTER_BUFFER_MODE_NONE` または `RTP_JITTER_BUFFER_MODE_BUFFER` 以外の場合**:

      * **`RTP_JITTER_BUFFER_MODE_SLAVE` (デフォルト)**: このモードでは、受信側クロックを送信側クロックに同期させようとします。`latency` は依然としてパケット保持の目標時間や欠落判断の基準として機能しますが、クロックスキューの補正が主目的となるため、純粋な遅延時間としての意味合いは `BUFFER` モードほど直接的ではありません。実際の遅延は、検出されたスキューやパケットの到着タイミングに大きく左右されます。
      * **`RTP_JITTER_BUFFER_MODE_SYNCED`**: 送信側と受信側のクロックが既に同期していることを前提とします。このモードでも `latency` はパケットの並べ替えや一時的な保持には使われますが、積極的な遅延追加というよりは、同期されたタイムスタンプに基づいた出力が優先されます。

3.  **`drop-on-latency` プロパティが `FALSE` (デフォルト) で、かつ下流が非常に高速にデータを消費する場合**:

      * `drop-on-latency` が `FALSE` の場合、ジッターバッファは `latency` を超えてもパケットを保持し続けることができます（メモリ上限に達するまで）。
      * もし下流のエレメント（デコーダやシンク）が非常に高速にデータを引き取り、`rtpjitterbuffer` の内部キューが常に空に近い状態であれば、パケットは `latency` で設定された時間まで待たずに、到着次第すぐに処理されて下流に渡される可能性があります。
      * この場合、`latency` プロパティはバッファの最大許容期間としての意味合いが薄れ、実際の遅延への影響は小さくなります。
      * `gst_rtp_jitter_buffer_loop` 関数内で `handle_next_buffer` が呼び出され、キューに処理すべきバッファがあれば速やかに出力しようとします。

4.  **`faststart-min-packets` プロパティによる早期開始**:

      * `faststart-min-packets` が0より大きい値に設定されている場合、指定された数の連続したパケットが到着すると、`latency` の期間が経過するのを待たずにパケットの出力を開始します。

      * これにより、初期遅延を削減できますが、`latency` プロパティが意図する遅延時間よりも早くデータが流れ始めるため、`latency` の影響が限定的に見えることがあります。

      * `gst_rtp_jitter_buffer_chain` 関数内の `gst_rtp_jitter_buffer_fast_start` の呼び出しがこれに該当します。

        ```c
        /* gstrtpjitterbuffer.c の gst_rtp_jitter_buffer_chain 関数内 */
        if (gst_rtp_jitter_buffer_fast_start (jitterbuffer)) // faststart の条件をチェック
          head = TRUE; // head が TRUE になると、ループが早く動き出す可能性がある

        if (G_UNLIKELY (head)) {
          if (G_LIKELY (priv->active))
            JBUF_SIGNAL_EVENT (priv); // イベントを通知して出力ループを起動

          if (G_UNLIKELY (priv->clock_id)) {
            GST_DEBUG_OBJECT (jitterbuffer, "Unscheduling waiting new buffer");
            unschedule_current_timer (jitterbuffer); // 既存のタイマーを解除
          }
        }
        ```

5.  **EOS (End Of Stream) 状態**:

      * ストリームの終わりに近づき、EOS が送信されると、`rtpjitterbuffer` はバッファリング動作を変更し、残っているパケットを速やかに排出しようとします。
      * `rtp_jitter_buffer_disable_buffering (priv->jbuf, TRUE);` が `GST_EVENT_EOS` 受信時に呼び出されます。
      * また、`wait_next_timeout` 内で `priv->eos` が `TRUE` の場合、タイマーのタイムアウト値に関わらず処理を進めるロジックがあります。
      * この状態では、`latency` プロパティによる遅延制御よりも、ストリームの確実な終了が優先されます。

6.  **パイプラインの他の部分にボトルネックがある場合**:

      * `rtpjitterbuffer` の下流に処理が重いエレメントがあり、そこがボトルネックとなってデータの流れが滞っている場合、`rtpjitterbuffer` が `latency` に基づいてパケットを保持していても、パイプライン全体の遅延は主にそのボトルネックによって決定されます。
      * この場合、`rtpjitterbuffer` の `latency` を変更しても、全体の体感遅延には大きな変化が見られないことがあります。

7.  **`latency` が極端に小さい値に設定されている場合**:

      * `latency` が非常に小さい値（例: 0に近い値）に設定されている場合、ジッター吸収やパケット並べ替えのための十分な時間が確保できません。
      * パケットはほぼ到着した順に出力され、`latency` プロパティによる遅延効果はほとんど見られなくなります。ただし、この設定ではジッターバッファとしての本来の機能は十分に発揮されません。

これらの条件下では、`latency` プロパティが直接的にパイプラインの遅延や動作に影響を与えているとは感じられないかもしれません。しかし、内部的には `latency` の値に基づいて各種タイマーやバッファ管理の閾値が設定されており、間接的にはストリームの安定性や品質に寄与している点を理解することが重要です。多くの場合、`latency` は他の要因との組み合わせでその効果を発揮します。
