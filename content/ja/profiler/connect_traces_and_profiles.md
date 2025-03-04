---
aliases:
- /ja/tracing/profiler/connect_traces_and_profiles/
further_reading:
- link: トレーシング
  tag: ドキュメント
  text: APM 分散型トレーシング
- link: /profiler/enabling
  tag: ドキュメント
  text: アプリケーションの継続的なプロファイラーを有効にする
- link: getting_started/profiler
  tag: ドキュメント
  text: プロファイラーの概要
kind: ドキュメント
title: 遅いトレースやエンドポイントを調査する
---

本番環境においてアプリケーションのパフォーマンスに問題がある場合、分散型トレーシングとプロファイリングによるコードスタックトレースのベンチマークを統合することで、パフォーマンスのボトルネックを特定する強力な方法となります。APM 分散型トレーシングと Continuous Profiler の両方が有効になっているアプリケーションプロセスは、自動的にリンクされます。

Code Hotspots タブでスパン情報からプロファイリングデータに直接移動し、パフォーマンス問題に関連する特定のコード行を見つけることができます。同様に、低速でリソースを消費するエンドポイントも、プロファイリング UI で直接デバッグできます。

## 遅いトレースのコードのホットスポットを特定する

{{< img src="profiler/code_hotspots_tab-2.mp4" alt="Code Hotspots タブで APM トレーススパンのプロファイリング情報を確認" video=true >}}

### 前提条件

{{< programming-lang-wrapper langs="java,python,go,ruby,dotnet,php" >}}
{{< programming-lang lang="java" >}}
[Java サービスのプロファイリングを起動する][1]と、コードのホットスポット識別がデフォルトで有効化されます。手動でインスツルメントされたコードの場合、Continuous Profiler はスパンのスコープアクティベーションを要求します。

```java
final Span span = tracer.buildSpan("ServicehandlerSpan").start();
try (final Scope scope = tracer.activateSpan(span)) { // mandatory for Datadog continuous profiler to link with span
    // worker thread impl
  } finally {
    // Step 3: Finish Span when work is complete
    span.finish();
  }

```

以下が必要です:
- OpenJDK 11+ および `dd-trace-java` バージョン 0.65.0+、または
- OpenJDK 8: 8u282+ および `dd-trace-java` バージョン 0.77.0+。

[1]: /ja/profiler/enabling/java
{{< /programming-lang >}}
{{< programming-lang lang="python" >}}

[Python サービスのプロファイリングを起動する][1]と、コードのホットスポット識別がデフォルトで有効化されます。

`dd-trace-py` バージョン 0.44.0+ が必要です。

[1]: /ja/profiler/enabling/python
{{< /programming-lang >}}
{{< programming-lang lang="ruby" >}}

[Ruby サービスのプロファイリングを起動する][1]と、コードのホットスポット識別がデフォルトで有効化されます。

`dd-trace-rb` バージョン 0.49.0+ が必要です。

[1]: /ja/profiler/enabling/ruby
{{< /programming-lang >}}
{{< programming-lang lang="go" >}}

[Go サービスのプロファイリングを起動する][1]と、コードのホットスポット識別がデフォルトで有効化されます。

`dd-trace-go` バージョン 1.37.0+ が必要です。

**注:** この機能は、Go バージョン 1.18 以降で最もよく機能します。Go 1.17 以下にはいくつかのバグがあり、特に CGO を多用する場合、この機能の精度が落ちることがあります ([GH-35057][2]、[GH-48577][3]、[CL-369741][4]、および [CL-369983][5] 参照)。

[1]: /ja/profiler/enabling/go
[2]: https://github.com/golang/go/issues/35057
[3]: https://github.com/golang/go/issues/48577
[4]: https://go-review.googlesource.com/c/go/+/369741/
[5]: https://go-review.googlesource.com/c/go/+/369983/
{{< /programming-lang >}}
{{< programming-lang lang="dotnet" >}}

[.NET サービスのプロファイリングを起動する][1]と、コードのホットスポット識別がデフォルトで有効化されます。

`dd-trace-dotnet` バージョン 2.7.0+ が必要です。

[1]: /ja/profiler/enabling/dotnet
{{< /programming-lang >}}
{{< programming-lang lang="php" >}}

[PHP サービスのプロファイリングを起動する][1]と、コードのホットスポット識別がデフォルトで有効化されます。

`dd-trace-php` バージョン 0.71+ が必要です。

[1]: /ja/profiler/enabling/php
{{< /programming-lang >}}
{{< /programming-lang-wrapper >}}

### スパンからプロファイリングデータにリンクする

各トレースのビューから、選択したスパンの範囲内のプロファイリングデータが Code Hotspots タブに表示されます。

左側の値は、選択したスパンにおいて、そのメソッド呼び出しに費やされた時間です。ランタイムと言語によって、このタイプのリストは異なります。

- **メソッドの実行時間**は、コードの各メソッドが実行された全体の時間を示します。
- **CPU** は、CPU タスクを実行するのに費やした時間を示します。
- **Synchronization** は、モニター待ちの時間、スレッドがスリープしている時間、パークしている時間などを表示します。
- **VM オペレーション** (Java のみ) は、ガベージコレクションに関連しない (ヒープダンプなど) VM のオペレーション待機に費やした時間を示します。
- **ファイル I/O** は、ディスクの読み取り/書き込み動作の実行待ちに費やした時間を示します。
- **ソケット I/O** は、ネットワークの読み取り/書き込み動作の実行待ちに費やした時間を示します。
- **Monitor enter** は、スレッドがロックでブロックされている時間を表示します。
- **Uncategorized** は、上記のカテゴリーに分類できないスパンの実行に要した時間を表示します。

プラスアイコン `+` をクリックすると、スタックトレースをそのメソッドに**逆順**に展開します。値の上にカーソルを置くと、カテゴリー別に説明される時間の割合が表示されます。

### プロファイルをトレースから閲覧する

{{< img src="profiler/flamegraph_view-1.mp4" alt="フレームグラフでプロファイルビューを開く" video=true >}}

内訳のデータタイプごとに、**View In Full Page** をクリックすると、同じデータが新しいページで表示されます。そこから、フレームグラフに視覚化を変更することができます。
**Focus On** セレクタをクリックすると、データの範囲を定義することができます。

- **Span & Children** は、選択したスパンと同じサービス内のすべての子孫スパンにプロファイリングデータをスコープします。
- **Span only** は、プロファイリングデータをあらかじめ選択されたスパンにのみスコープします。
- **Span time period** は、スパンがアクティブだった期間中のすべてのスレッドにプロファイリングデータをスコープします。
- **Full profile** は、データのスコープを以前に選択されたスパンを実行したサービスプロセス全体の 60 秒に設定します。

## コードのパフォーマンスを API エンドポイントごとに分解する

### 前提条件

{{< programming-lang-wrapper langs="python,go,ruby,dotnet,php" >}}
{{< programming-lang lang="python" >}}

[Python サービスのプロファイリングを起動する][1]と、エンドポイントプロファイリングがデフォルトで有効化されます。

`dd-trace-py` バージョン 0.54.0+ が必要です。

[1]: /ja/profiler/enabling/python
{{< /programming-lang >}}
{{< programming-lang lang="go" >}}
[Go サービスのプロファイリングを起動する][1]と、エンドポイントプロファイリングがデフォルトで有効化されます。

`dd-trace-go` バージョン 1.37.0+ が必要です。

**注:** この機能は、Go バージョン 1.18 以降で最もよく機能します。Go 1.17 以下にはいくつかのバグがあり、特に CGO を多用する場合、この機能の精度が落ちることがあります ([GH-35057][2]、[GH-48577][3]、[CL-369741][4]、および [CL-369983][5] 参照)。

[1]: /ja/profiler/enabling/go
[2]: https://github.com/golang/go/issues/35057
[3]: https://github.com/golang/go/issues/48577
[4]: https://go-review.googlesource.com/c/go/+/369741/
[5]: https://go-review.googlesource.com/c/go/+/369983/
{{< /programming-lang >}}
{{< programming-lang lang="ruby" >}}

[Ruby サービスのプロファイリングを起動する][1]と、エンドポイントプロファイリングがデフォルトで有効化されます。

`dd-trace-rb` バージョン 0.54.0+ が必要です。

[1]: /ja/profiler/enabling/ruby
{{< /programming-lang >}}
{{< programming-lang lang="dotnet" >}}

[.NET サービスのプロファイリングを起動する][1]と、エンドポイントプロファイリングがデフォルトで有効化されます。

`dd-trace-dotnet` バージョン 2.15.0+ が必要です。

[1]: /ja/profiler/enabling/dotnet
{{< /programming-lang >}}
{{< programming-lang lang="php" >}}

[PHP サービスのプロファイリングを起動する][1]と、エンドポイントプロファイリングがデフォルトで有効化されます。

`dd-trace-php` バージョン 0.79.0+ が必要です。

[1]: /ja/profiler/enabling/php
{{< /programming-lang >}}
{{< /programming-lang-wrapper >}}

### フレームグラフをエンドポイントごとにスコープする

エンドポイントプロファイリングは、Web サービスの任意のエンドポイントでフレームグラフをスコープし、遅いエンドポイント、レイテンシーが多いエンドポイント、エンドユーザーエクスペリエンスが悪い原因となっているエンドポイントを見つけることができます。これらのエンドポイントは、デバッグが難しく、なぜ遅いのかを理解するのが困難な場合があります。遅い原因は、エンドポイントが多くの CPU サイクルを消費するなど、意図しない大量のリソースを消費している可能性があります。

エンドポイントプロファイリングを利用すると、以下のことが可能になります。

- エンドポイント全体のレスポンスタイムを遅くしているボトルネックとなるメソッドを特定する。
- CPU やウォールタイムなどの貴重なリソースを消費する上位のエンドポイントを切り分ける。これは、一般的にパフォーマンスを向上させるためにサービスを最適化しようとしている場合に特に役立ちます。
- サードパーティのコードやランタイムライブラリが、エンドポイントの速度低下やリソース消費の重さの原因になっているかどうかを把握する。

{{< img src="profiler/endpoint_agg_gif.mp4" alt="エンドポイント集計による遅いエンドポイントのトラブルシューティング" video=true >}}


### 最もリソースを消費しているエンドポイントを追跡する

CPU やウォールタイムなどの貴重なリソースを消費している上位のエンドポイントを追跡することは価値があります。このリストは、エンドポイントが回帰していないか、あるいは新たに導入したエンドポイントが大幅にリソースを消費してサービス全体の速度を低下させていないかどうかを確認するのに役立ちます。

{{< img src="profiler/endpoint_metric.mp4" alt="上位のエンドポイントの消費リソースのグラフ化" video=true >}}

## その他の参考資料

{{< partial name="whats-next/whats-next.html" >}}

[1]: /ja/profiler/profiler_troubleshooting#reduce-overhead-from-default-setup
[2]: /ja/tracing/trace_collection/custom_instrumentation/java#manually-creating-a-new-span