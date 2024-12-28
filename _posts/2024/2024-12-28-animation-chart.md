---
toc: true
title: "Python で動きのあるグラフを描写する [bar_chart_race/FuncAnimation]"
date: 2024-12-28 09:00:00 +0900
categories: プログラム
tags:
- python
- animation
- bar-chart-race
- FuncAnimation
---
Python を使ってグラフを描写する際に、YouTube などでも見かける動きのある表示をさせる方法があるようです。2 種類確認できたので、それぞれの方法で描写してみます。

# データの用意
まずはグラフ化するデータを用意します。2024 年のドライバーズポイントのレースごとの推移を、`result.csv` というファイル名の CSV ファイルで用意することにしました。

![result.csv][img01]

これを以下のコードで pandas の DataFrame にセットします。

``` python
import pandas as pd

df = pd.read_csv('result.csv')
```

これでデータの用意は完了です。

# グラフ描写
## bar_chart_race
[https://www.dexplo.org/bar_chart_race/][bcr]

棒グラフがレースをするように動かせるライブラリです。[チュートリアル][tutorial] にはパラメータでどのようなグラフを描写できるか、動画付きで説明しているので参考になります。今回はパラメータはデフォルトのまま、シンプルに描写してみます。

### インストール
ターミナルから pip コマンドでインストールできます。また、アニメーションを保存する際に ffmpeg を使用するので、apt (Ubuntu の場合) を使ってインストールします。

``` bash
sudo apt install -y ffmpeg
pip install bar_chart_race
```

### コード
``` python
import bar_chart_race as bcr

bcr.bar_chart_race(df=df, n_bars=10)
```

`bar_chart_race` をインポートしたら、描写するデータを `df` パラメータで指定するのみです。`n_bars` はデータの上位件数のみを表示するオプションです。

### 出力
![result.mp4][img02]

あれだけのコードでこんな出力が可能なんですね。

## FuncAnimation
[https://matplotlib.org/stable/api/_as_gen/matplotlib.animation.FuncAnimation.html][FuncAnimation]

matplotlib の関数の一つ、FuncAnimation です。一定間隔で関数を呼び出しながらグラフを描写することができる、というイメージなので、matplotlib で描写できるグラフならアニメーションさせることが可能ですが、bar_chart_race のような追加の動きを描写することは難しいと思います。

### インストール
matplotlib をインストールすれば FuncAnimation も使用可能になります。

``` bash
pip install matplotlib
```

### コード
``` python
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation

d1 = df["VER"]
d2 = df["NOR"]
t = list(range(len(d1)))
x1,y1 = [],[]
x2,y2 = [],[]

fig, ax = plt.subplots()
ax.set_ylim(0, 450)
ax.set_xlim(0, 25)

def animate(i):
    x1.append(t[i])
    y1.append((d1[i]))
    ax.plot(x1,y1, scaley=True, scalex=True, color="blue")
    x2.append(t[i])
    y2.append((d2[i]))
    ax.plot(x2,y2, scaley=True, scalex=True, color="yellow")

ani = FuncAnimation(fig=fig, func=animate, frames=range(24), interval=100)
ani.save("/root/anime.gif", writer="pillow")
```

まず `matplotlib.animation.FuncAnimation` をインポートします。今回はフェルスタッペンとノリスのポイント推移を比較してみるので、CSV を読み込んだ DataFrame から二人のデータを抜き出します。

関数 `animate(i)` が定期的にグラフを描写する処理です。フェルスタッペンのデータを入れた `d1` から青線のグラフを、ノリスのデータを入れた `d2` から黄色線のグラフを描写します。引数 `i` に何フレーム目かを示す数値が入るので、それで何レース目のデータを描写するかを定義している形です。

### 出力
![result.gif][img03]

bar_chart_race を見た後だとシンプルに見えてしまいますが、一枚絵のグラフよりも推移が見えてくる感じがしますね。後半、フェルスタッペンはだいぶ追い詰められていたように感じましたが、こうやって見てみるとそこまで僅差にはなっていなかった、という見方もできる気がします。


# まとめ
同じデータを表現しているだけですが、動きがあることでまた違う見方ができそうですね。今後はこれらも使ってみたいと感じました。また、今回紹介した 2 つは、それぞれ様々なオプションがあるので、表現の仕方もいろいろありそうで興味を惹かれました。


[bcr]:https://www.dexplo.org/bar_chart_race/
[tutorial]:https://www.dexplo.org/bar_chart_race/tutorial/
[FuncAnimation]:https://matplotlib.org/stable/api/_as_gen/matplotlib.animation.FuncAnimation.html


[img01]:/assets/images/2024/12/ss-20241228-01.png
[img02]:/assets/images/2024/12/ss-20241228-02.gif
[img03]:/assets/images/2024/12/ss-20241228-03.gif

