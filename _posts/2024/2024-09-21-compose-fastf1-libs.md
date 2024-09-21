---
layout: post
title:  "自作の fastf1-libs の実行環境を docker compose で構築する"
date:   2024-09-21 09:00:00 +0900
categories: プログラム
tags:
- python
- compose
- F1
- fastf1-libs
---
[自作の fastf1 便利ライブラリ][fastf1-libs] を紹介しましたが、こちらも docker compose で簡単・確実に環境を作れるようにします。ついでにいろいろ試すには便利な、JuputerLab を起動するようにします。

### 準備
[WSL2 の Docker で CUDA する][docker] を参考に、GPU を使えるように設定済みのコンテナを使います。

fastf1-libs 用に以下の内容を記述したファイルを、compose.yml のファイル名で作成します。

{% highlight yaml %}
services:
  fastf1-libs:
    image: jupyter/base-notebook
    volumes:
      - ./fastf1-libs:/fastf1-libs/
      - ./data:/data/
      - ./cache:/data/cache/
    command:
    - /bin/bash
    - -c
    - |
      cd /fastf1-libs/
      pip install -r requirements-dev.txt
      pip install fastf1_lib/
      start-notebook.py --ip=0.0.0.0 --notebook-dir=/data
    ports:
      - "8888:8888"
{% endhighlight %}

特段 GPU の支援は必要ないので、設定がすっきりしていますね。コンテナイメージも、JupyterLab のイメージを使っているので、自作のライブラリをインストールすれば環境としては完成です。

別マシンから接続する形になるので、JupyterLab の起動オプションに `--ip=0.0.0.0` をつけないと接続を拒否してしまうのと、ノートブックの初期ディレクトリを指定するようにしています。

さらに同じディレクトリに fastf1-libs のリポジトリをクローンしておきます。これは、compose.yml を作成したディレクトリで以下コマンドを実行することで可能です。

{% highlight bash %}
git clone https://github.com/android-pokoten/fastf1-libs.git
{% endhighlight %}

また、ノートブックを保存しておくパス (`./data`) と、セッションデータをキャッシュするディレクトリ (`./cache`) を作成しておきます。

この状態で、以下のディレクトリ構成になっているはずです。

{% highlight bash %}
.
|- fastf1-libs/
   |- cache/
   |- data/
   |- fastf1-libs/
     |- ...
-- compose.yml
{% endhighlight %}

`cache` と `data` の中は、最初は空です。コンテナイメージの更新などで作り直しても、データは残せるようにします。

### 起動
ファイル、ディレクトリの準備ができたら、以下のコマンドを実行します。
{% highlight bash %}
docker compose up
{% endhighlight %}

しばらく待つとターミナルに `http://127.0.0.1/lab?token=xxx` と表示されるので、その URL をブラウザにコピーして開きます。

![startup][img1]

これで JupyterLab を開くことができます。`/data` ディレクトリを開いているので、そこでファイルを作成していくことが可能です。

![signup][img2]

コードの実行に関して2点ほど。fastf1-libs のリポジトリを `/fastf1-libs` にマウントしているので、ライブラリの読み込みは以下のようにパスを指定すれば OK です。

{% highlight python %}
import sys
import importlib
sys.path.append("/fastf1-libs/fastf1_lib")
from fastf1lib import myFastf1

ff1 = myFastf1()
{% endhighlight %}

キャッシュは `/data/cache` にマウントしているので、セッションをロードする `load_session_o` は以下のように実行できます。

{% highlight python %}
session_fp1 = ff1.load_session_o(
    name="singapore", 
    year=2024, 
    s='FP1',
    cache='/data/cache/'
    )
{% endhighlight %}

仮に JupyterLab の更新をするなど、コンテナイメージを更新を指定場合は、`docker compose down` でコンテナを削除し、`docker pull jupyter/base-notebook` でイメージを最新化します。そのうえで `docker compose up` で起動するだけです。作成したノートブックやキャッシュデータはそのままです。

[docker]:{% post_url 2024/2024-06-22-docker-and-cuda %}
[fastf1-libs]:{% post_url 2023/2023-12-30-myfastf1lib-01 %}

[img1]:/assets/images/2024/09/ss-20240921-01.png
[img2]:/assets/images/2024/09/ss-20240921-02.png
