---
layout: post
title:  "複数ドライバーのラップタイム推移を比較 [自作の fastf1 便利ライブラリ]"
date:   2024-02-28 09:00:00 +0900
categories: F1
tags:
- python
- fastf1
- fastf1lib
- data
---
自作ライブラリのメソッドを紹介します。今回は`laptime_comperition`です。以下の記事の通り、`ff1`オブジェクトを作成しておくのと、セッションを読み込んでおいてからの実行となります。

[自作の fastf1 便利ライブラリ][prev]

### 実行例
![2/22のペース][img02]

プレシーズンテストの記事でも使用したグラフですが、このようにドライバーごとに色分けしてラップタイム推移を表示します。

### 実行方法
{% highlight python %}
ff1.laptime_comperition(
    session=session_day2,
    drivers=['11', '55'],
    min_sec=93,
    max_sec=101
    )
{% endhighlight %}
セッションオブジェクトと、比較したいドライバーをリストで指定します。ドライバーは3文字略称や、カーナンバーでの指定が可能です。

min_sec と max_sec でグラフに表示するタイムの幅を決めます。そうしないと、ピットストップやセーフティーカーなどでペースが極端に変わったときにグラフが見えづらくなってしまうことを避けるために、指定するようにしました。



### 処理内容(抜粋)
{% highlight python %}
        ax.set_ylim(np.timedelta64(min_sec, 's'), np.timedelta64(max_sec, 's'))
{% endhighlight %}
実行方法でも言及した通り、min_sec と max_sec を指定していますが、int で指定した引数をそのまま `set_ylim` へ渡しても意図したグラフにならずに、少し苦労しました。

このように numpy の timedelta64 で秒に変換することで実現できました。



プレシーズンテストの図でもそうでしたが、現在の処理だとセッション全体をグラフ化してしまうので、ラップの範囲を指定したり、異なるセッション間でグラフ化できるようにすると、より便利になりそうな気はしています。

[prev]:{% post_url 2023/2023-12-30-myfastf1lib-01 %}
[img02]:/assets/images/2024/output-2024022500.png
