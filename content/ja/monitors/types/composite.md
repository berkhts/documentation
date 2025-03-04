---
aliases:
- /ja/guides/composite_monitors
- /ja/monitors/monitor_types/composite
- /ja/monitors/create/types/composite/
description: 複数のモニターを組み合わせた式に対してアラートする
further_reading:
- link: /monitors/notify/
  tag: ドキュメント
  text: モニター通知の設定
- link: /monitors/notify/downtimes/
  tag: ドキュメント
  text: モニターをミュートするダウンタイムのスケジュール
- link: /monitors/manage/status/
  tag: ドキュメント
  text: モニターステータスを確認
kind: documentation
title: 複合条件モニター
---


## 概要

複合条件モニターは、個別モニターを組み合わせて 1 つのモニターにし、さらに詳細なアラート条件を定義します。

既存のモニターを選択して、複合条件モニターを作成します。たとえば、モニター `A` とモニター `B` とします。次に、`A && B` などのブール演算子を使用してトリガー条件を設定します。複合条件モニターは、個別モニターが同時に複合条件モニターのトリガー条件を真にする値を持つときにトリガーします。

{{< img src="monitors/monitor_types/composite/overview.jpg" alt="複合条件の例" style="width:80%;">}}

構成の目的上、複合条件モニターは構成モニターに依存しません。複合条件モニターの通知ポリシーは、構成モニターのポリシーに影響することなく変更できます。その逆も同様です。さらに、複合条件モニターを削除しても、構成モニターは削除されません。複合条件モニターは他のモニターを所有せずに、その結果を使用するだけです。複数の複合条件モニターが同じ個別モニターを参照することもあります。

**注**

- `個別モニター`、`構成モニター`、`非複合条件モニター` という用語はすべて、複合条件モニターがそのステータスを計算するために使用するモニターを指します。
- 複合条件結果には、共通のグループ化が必要です。共通のグループ化がないモニターを選択した場合、式で選択したモニターが複合条件結果にならない場合があります。
- 複合条件モニターは、他の複合条件モニターに基づくことはできません。

## モニターの作成

Datadog で[複合条件モニター][1]を作成するには、メインナビゲーションを使用して次のように移動します: *Monitors --> New Monitor --> Composite*。

### モニターの選択とトリガー条件の設定

#### モニターの選択

複合条件モニターで使用する個別モニターを最大 **10** 個まで選択できます。異なるアラートタイプのモニターでもかまいません (シンプルアラートのみ、マルチアラートのみ、または両方の組み合わせ)。複合条件モニターを個別モニターとして使用することはできません。最初のモニターを選択すると、UI にそのアラートタイプと現在のステータスが表示されます。

マルチアラートモニターを選択すると、UI にはモニターの group-by 節と、現在レポートされている一意のソースの数が表示されます (例: `Returns 5 host groups`)。マルチアラートモニターを組み合わせる場合、この情報は自然にペアになるモニターを選択するのに役立ちます。

{{< img src="monitors/monitor_types/composite/composite_example.jpg" alt="複合条件の例"style="width:80%;">}}

同じグループを持つモニターを選択する必要があります。そうしなかった場合、UI には、このような複合条件モニターは決してトリガーしない旨の警告が表示されます。

{{< img src="monitors/monitor_types/composite/composite_common_group.jpg" alt="複合条件の共通グループ" style="width:80%;">}}


同じグループのマルチアラートモニターを選択した場合でも、モニターに共通のレポートソース (共通のグループ化とも呼ばれる) がない場合、`Group Matching Error` が表示されることがあります。共通のレポートソースがない場合、Datadog は複合条件モニターのステータスを計算できず、トリガーしません。 ただし、警告を無視して、とにかくモニターを作成することは_可能_です。詳細については、[複合条件モニターが共通の報告元ソースを選択する方法](#select-monitors-and-set-triggering-conditions)を参照してください。

警告が表示されないように 2 つ目のモニターを選択すると、**Trigger when** フィールドにデフォルトのトリガー条件 `a && b` が挿入され、提案された複合条件モニターのステータスが表示されます。

#### トリガー条件を設定する

**Trigger when** フィールドに、ブール演算子を使用して目的のトリガー条件を記述します。個別モニターは、フォーム内のラベル (`a`、`b`、`c` など) で識別されます。括弧を使用して演算子の優先度を制御すると、複雑な条件を作成できます。

以下はどれも有効なトリガー条件です。

```text
!(a && b)
a || b && !c
(a || b) && (c || d)
```

#### 高度なアラート条件

##### No Data

複合条件モニターがデータなしの状態にあることを `Do not notify`（通知しない）か `Notify`（通知する）かを選択します。ここでどちらを選択しても、個別モニターの `Notify no data` 設定には影響しませんが、複合条件がデータなしでアラートを出すためには、個別モニターと複合モニターの両方をデータが欠落していることを `Notify`（通知する）に設定する必要があります。

##### その他のオプション

高度なアラートオプション (自動解決など) の詳細な手順については、[モニターコンフィギュレーション][2]ページを参照してください。

### 通知

複合条件モニターの構成モニターのテンプレート変数を通知に使用する方法については、[複合条件モニター変数][4]を参照してください。**Say what's happening** セクションと **Notify your team** セクションの詳細な説明は、[通知][3]のページを参照してください。

### API

API を使用している場合、複合条件モニターのクエリはモニター ID を使用しているサブモニターについて定義されます。

たとえば、2 つの非複合条件モニターに次のクエリと ID があるとします。

```text
"avg(last_1m):avg:system.mem.free{role:database} < 2147483648" # Monitor ID: 1234
"avg(last_1m):avg:system.cpu.system{role:database} > 50" # Monitor ID: 5678
```

上記のモニターを組み合わせた複合条件モニターのクエリは、`"1234 && 5678"` や `"!1234 || 5678"` などになります。

## 複合条件モニターの仕組み

このセクションでは、いくつかのサンプルを使用してトリガー条件が計算される方法と、さまざまなシナリオで受け取るアラートの数について説明します。

### トリガー条件の計算

複合条件モニターには 4 種類のステータスがあり、そのうち 3 種類はアラート対象です。

| ステータス    | アラート対象         |重大度           |
|-----------|----------------------|-------------------|
| `Alert`   | 真                 |4 (最も重大)    |
| `Warn`    | 真                 |3                  |
| `No Data` | 真                 |2                  |
| `Ok`      | 偽                |1 (最も重大ではない)   |

使用されているブール演算子 (`&&`、`||`、`!`) は、複合条件モニターステータスのアラート対象を操作するものです。

* `A && B` がアラート対象である場合、結果は A と B 間の**最も重大ではない**ステータスとなります。
* `A || B` がアラート対象である場合、結果は A と B 間の**最も重大な**ステータスとなります。　
* `A` が `No Data` の場合、`!A` は `No Data`
* `A` がアラート対象の場合、`!A` は `OK`　
* `A` がアラート対象でない場合、`!A` は `Alert`

2 つの個別モニター（`A` および `B`）を考えます。次の表は、トリガー条件 (`&&` または `||`) を与えられた複合条件モニター、および個別モニターにさまざまなステータスが与えられた場合の複合条件モニターの結果ステータスです。アラート対象かどうかは、T または F で示されます。

| モニター A   | モニター B   | 条件   | 通知データなし   | 複合条件ステータス | アラートがトリガーされるか？ |
|-------------|-------------|-------------|------------------|------------------|------------------|
| Alert (T)   | Warn (T)    | `A && B`    |                  | Warn (T)         | {{< X >}}        |
| Alert (T)   | Warn (T)    | `A \|\| B`    |                  | Alert (T)        | {{< X >}}        |
| Warn (T)    | Ok (F)      | `A && B`    |                  | OK (F)           |                  |
| Warn (T)    | Ok (F)      | `A \|\| B`    |                  | Warn (T)         | {{< X >}}        |
| データなし (T) | Warn (T)    | `A && B`    | 真             | データなし (T)      | {{< X >}}        |
| データなし (T) | Warn (T)    | `A \|\| B`    | 真             | Warn (T)         | {{< X >}}        |
| データなし (T) | Warn (T)    | `A && B`    | 偽            | 最後の既知データ       |                  |
| データなし (T) | Warn (T)    | `A \|\| B`    | 偽            | Warn (T)         | {{< X >}}        |
| データなし (T) | OK (F)      | `A && B`    | 偽            | OK (F)           |                  |
| データなし (T) | OK (F)      | `A \|\| B`    | 偽            | 最後の既知データ       |                  |
| データなし (T) | OK (F)      | `A && B`    | 真             | OK (F)           |                  |
| データなし (T) | OK (F)      | `A \|\| B`    | 真             | データなし (T)      | {{< X >}}        |
| データなし (T) | データなし (T) | `A && B`    | 真             | データなし (T)      | {{< X >}}        |
| データなし (T) | データなし (T) | `A \|\| B`    | 真             | データなし (T)      | {{< X >}}        |

**注**: 複合条件の `notify_no_data` が false で、サブモニターの評価結果がその複合条件に対して `No Data` ステータスになった場合、複合条件は代わりに最後の既知の状態を使用します。

### 複合条件とダウンタイム

複合条件モニターとその個別モニターは互いに独立しています。

#### 複合条件モニターでのダウンタイム

`A || B`の条件を持つ 2 つの個別モニターからなる複合条件モニター `C` を考えてみましょう。複合条件モニターにダウンタイムを作成すると、`C` からの通知のみが抑制されます。

モニター `A` または `B` がそれぞれのモニター構成でサービスやチームに通知する場合、複合条件 `C` のダウンタイムは `A` または `B` によって引き起こされた通知をミュートすることはありません。`A` または `B` からの通知をミュートするには、これらのモニターにダウンタイムを設定します。

#### 複合条件モニターで使用している個別モニターでのダウンタイム

複合条件モニター内で使用されている個々のモニター `A` にダウンタイムを作成しても、複合条件モニターはミュートされません。

例えば、ダウンタイムが発生すると、モニター `A`、特にそのグループ `env:staging` はミュートされます。グループ `env:staging` がアラート対応状態になると、個々のモニターから来る通知は抑制され、複合条件モニターはアラート通知を送信します。

### アラートの数

受け取るアラートの数は、個別モニターのアラートタイプによって異なります。
すべての個別モニターがシンプルアラートなら、複合条件モニターもシンプルアラートタイプになります。`A` および `B` のクエリがすべて同時に `true` の場合に、複合条件モニターは 1 つの通知をトリガーします。

個別モニターが 1 つでもマルチアラートなら、複合条件モニターもマルチアラートです。同時にいくつのアラートが送信されるかは、複合条件モニターが使用しているマルチアラートモニターが 1 つか複数かに依存します。


### 共通の報告元ソース

多くのマルチアラートモニターを使用する複合条件モニターでは、個別モニターの*共通の報告元ソース*のみが考慮されます。

**マルチアラートの例**

モニター `A` および `B` がマルチアラートで、ホスト別にグループ化されているシナリオで考えてみましょう。

* `host:web01` から `host:web05` のホストはモニター `A` のレポートを作成しています。
* `host:web04` から `host:web09` のホストはモニター `B` のレポートを作成しています。

複合条件モニターは、共通ソース (`web04` と `web05`) _のみ_ を考慮します。1 回の評価サイクルで最大 2 つのアラートを受信できます。

**異なるグループ名の共通グループ評価**

複合条件モニターはタグ*値* (`web04`) のみを参照し、タグ*キー* (`host`) は参照しません。
上記の例に、単一の報告元ソース `service:web04` を持つ `service` によってグループ化されたマルチアラートモニター `C` が含まれている場合、複合条件モニターは `web04` を `A`、`B`、`C` 間の単一の共通の報告元ソースと見なします。

**2 つ以上のディメンションのあるモニターグループ**

2 つ以上のタグで分割されたマルチアラートモニターの場合、モニターグループはタグの組み合わせ全体に対応します。
たとえば、モニター `1` が `device,host` に基づくマルチアラートであり、モニター `2` が `host` に基づくマルチアラートである場合、複合条件モニターはモニター `1` とモニター `2` を組み合わせることができます。

{{< img src="monitors/monitor_types/composite/multi-alert-1.png" alt="書き込み通知" style="width:80%;">}}

ただし、`host,url` に基づくマルチアラート、モニター `3` を考えると、モニター `1` とモニター `3` の組み合わせは、グルーピングが大きく異なるために、複合条件の結果にならない可能性があります。
{{< img src="monitors/monitor_types/composite/multi-alert-2.png" alt="書き込み通知" style="width:80%;">}}


## その他の参考資料

{{< partial name="whats-next/whats-next.html" >}}

[1]: https://app.datadoghq.com/monitors#create/composite
[2]: /ja/monitors/configuration/#advanced-alert-conditions
[3]: /ja/monitors/notify/
[4]: /ja/monitors/notify/variables/?tab=is_alert#composite-monitor-variables