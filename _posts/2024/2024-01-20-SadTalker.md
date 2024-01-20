---
layout: post
title:  "音声にあわせて一枚絵を動かす SadTalker - Stable Diffusion WebUI の拡張機能"
date:   2024-01-20 00:00:00 +0900
categories: F1
tags:
- python
- 機械学習
- StableDiffusion
- extensions
- img2video
---
1枚の顔写真と音声データを組み合わせて、その音声をしゃべっているかのように顔を動かせる SadTalker を使ってみます。これは単独のアプリケーションでもありますが、Stable Diffusion の WebUI に拡張機能として使うことが可能なので、この方法で使ってみます。

### 準備
Docker コンテナを起動します。
{% highlight bash %}
>$ docker run -it --gpus=all --rm -p 7860:7860 -v /home/tadashi/work:/work nvidia/cuda:11.8.0-base-ubuntu22.04 /bin/bash
{% endhighlight %}

apt で必要なパッケージをインストールします。
{% highlight bash %}
># apt update
># apt install -y git python3-venv libgl1-mesa-dev libglib2.0-0 ffmpeg wget
{% endhighlight %}

git で WebUI をダウンロードします。また、コンテナの中だと root ユーザーになってしまい、WebUI の起動がブロックされるので、それを回避します。
{% highlight bash %}
># git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
># cd stable-diffusion-webui/
># sed -i -e s/can_run_as_root=0/can_run_as_root=1/ webui.sh
{% endhighlight %}

### SadTalker 拡張機能のインストール
WebUI を起動します。外部の拡張の読み込みを有効にするためのオプションも追加する必要があります。
{% highlight bash %}
># COMMANDLINE_ARGS="--listen --enable-insecure-extension-access" ./webui.sh
{% endhighlight %}

pip などで環境構築が走るので、場合によっては 10 分程度待つと、以下のように URL が表示されるので、ブラウザでアクセスします。
![](/assets/images/2024/ss_20240120_01.png)

「Extension」＞「Install from URL」を選択し、「URL for extension's git repository」欄に`https://github.com/OpenTalker/SadTalker.git`と入力して「Install」をクリックします。
![](/assets/images/2024/ss_20240120_02.png)

少し待って Install ボタンの下に`Installed into /stable-diffusion-webui/extensions/SadTalker. ～`と表示されることを確認し、いったんターミナルから Ctrl+c を押して webui.sh を停止します。
![](/assets/images/2024/ss_20240120_03.png)

SadTaker のモデルをダウンロードします。上記で表示された、SadTalker をインストールしたディレクトリへ移動して、ダウンロード用のコマンドを実行します。
{% highlight bash %}
># pushd extensions/SadTalker/
># bash scripts/download_models.sh
># popd
{% endhighlight %}

### 動画の生成

再度 WebUI を起動します。SadTalker が有効化されるので、ここの起動も少し時間がかかります。
{% highlight bash %}
># COMMANDLINE_ARGS="--listen --enable-insecure-extension-access" ./webui.sh
{% endhighlight %}

「SadTalker」タブができているので、これをクリックします。
![](/assets/images/2024/ss_20240120_04.png)

`Source image`に画像ファイルを、`Input audio`に音声ファイルを指定し、`Generate`をクリックすると、動画が生成されます。

30秒弱の音声データを使った場合、GeForce RTX 2070 でも約2分程度で動画が生成されました。