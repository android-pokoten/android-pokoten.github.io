---
layout: post
title:  "2 ドライバー間のテレメトリデータを重ねてグラフ化する [自作の fastf1 便利ライブラリ]"
date:   2024-01-13 00:00:00 +0900
categories: F1
tags:
- python
- fastf1
- fastf1lib
- data
---
自作ライブラリのメソッドを紹介します。まずは`datachart_compare`です。前回の記事の通り、`ff1`オブジェクトを作成しておくのと、セッションを読み込んでおいてからの実行となります。

### 実行例
![](/assets/images/2024/datachart_compare.png)
速度、スロットル開度、ブレーキ、エンジン回転数、ギアポジション、DRS状態をグラフ化します。横軸が距離になります。

各ドライバーの最速ラップをそれぞれ比較します。そのため、レースよりも予選向けのグラフです。

### 実行方法
{% highlight python %}
ff1.datachart_compare(
    session=session_qual,
    driver1='LEC',
    driver2='VER'
    )
{% endhighlight %}
セッションオブジェクトと、比較したいドライバーを指定します。ドライバーは3文字略称や、カーナンバーでの指定が可能です。


### 処理内容(抜粋)
{% highlight python %}
abb1 = session.laps.pick_driver(driver1)['Driver'].iloc[0]
color1 = fastf1.plotting.driver_color(abb1)

lap1 = session.laps.pick_driver(driver1).pick_fastest()
tel1 = lap1.get_telemetry()
{% endhighlight %}
それほど特筆するような処理もないです。`fastf1.plotting.driver_color()`でドライバーを表す色を取得し、その色(`color1`)でグラフを描写します。

さらに`laps`の`pick_fastest()`メソッドで最速ラップを取得し、`get_telemetry()`でテレメトリデータを取得し、種類ごとにグラフ化しているだけです。

この作りだと、3人以上のデータをグラフ化しようとすると大改修になってしまいますが、あまり何本もグラフを重ねても見えづらいので、これはこれでいいかな、と思っています。

任意のラップを指定できるようにしてもいいかも、とは思っています。それほど手間はかからず実装できそうですが、今のところそういう必要性も感じていないので後回しにしています。