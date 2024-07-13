---
layout: post
title:  "Stable Audio Open 1.0 を ComfyUI で使ってみる"
date:   2024-07-14 09:00:00 +0900
categories: プログラム
tags:
- python
- 機械学習
- StableAudioOpen
- ComfyUI
---
少し前になりますが、[Stable Audio Open 1.0 がリリース][release] されてました。これを ComfyUI で実行してみました。

### ComfyUI 環境準備
[Stable Diffusion 3 Medium を実行する環境を ComfyUI で用意した手順][comfyui] を実行し、コンテナで ComfyUI を実行できるようにします。

### Stable Audio Open 1.0 の準備
引き続き、追加のパッケージをインストールします。まずは apt で `cuda-toolkit-12-1` と `libsndfile1` をインストールします。

{% highlight bash %}
apt install -y cuda-toolkit-12-1 libsndfile1
{% endhighlight %}

次に pip で `stable-audio-tools` をインストールします。

{% highlight bash %}
pip install stable-audio-tools
{% endhighlight %}

[Hugging Face のモデルのページ][release] から、以下のファイルをダウンロードして、ComfyUI の `models/audio_checkpoints/` ディレクトリに配置します。このディレクトリが存在しない場合は、ディレクトリを作成してファイルを配置します。
+ model_config.json
+ stable-audio-open-1.0_model.safetensors

### 起動 & Custom Node 導入
ComfyUI のディレクトリで以下コマンドを実行し、起動します。

{% highlight bash %}
python3 main.py
{% endhighlight %}

ブラウザで UI を開いたら、右下の `Manager` をクリックします。

![ui][img-2]

ComfyUI Manager Menu が開くので、`Custom Node Manager` をクリックします。

![customnode][img1]

上部の「Search」欄に `stableaudio` と入力します。すると `ComfyUI Stable Audio Open 1.0 Sampler` があるので、それをインストールします。

![Stable Audio Open 1.0 Sampler][img2]

なお、上記画面はすでにインストール済みの画面となっていますが、本来なら「Try update」などのボタンが並んでいる箇所が「Install」になっているので、それをクリックしてインストールします。また、インストール後は ComfyUI の再起動と、ブラウザのリロードが必要になります。

### 生成
Custom Node が導入できたら、以下のようにノードをつなげます。そのうえでメニューの `Queue Prompt` をクリックすると、10秒ほどで曲が生成されます。

![Node][img3]

注意点としては以下がありました。

+ 「Load Stable Audio Model」ノードで、ダウンロードした `model_config.json` と `stable-audio-open-1.0_model.safetensors` が認識していることを確認します
+ 「Stable Audio Pre-Conditioning」ノードの `seconds_total` が曲の長さですが、GeForce RTX 2070 (VRAM 8GB) では `15 (秒)` ぐらいが上限でした。それ以上にすると、CUDAoutofmemory が発生して生成に失敗します
+ 「Stable Audio Sampler」の `save` を `true` にしておかないと、処理がエラーとなって生成に失敗します
+ 「Stable Audio Sampler」の `control_afer_generate` は初期値が `fixed` になっており、seed 値が固定です。手動で変更する場合はこれで問題ないのですが、いろいろ試したい場合はここを `randomize` にしておくと、生成後に seed 値をランダムに変更するようになります
+ 「Stable Audio Sampler」は生成後に音楽を再生してくれます。この後に Preview のノードをつなげなくても確認可能です。ローカルにダウンロードしたい場合は、`ComfyUI/output` ディレクトリに出力されたファイルをダウンロードすればよさそうです。


[release]:https://huggingface.co/stabilityai/stable-audio-open-1.0
[comfyui]:{% post_url 2024/2024-06-20-comfyui %}

[img-2]:/assets/images/2024/06/ss-20240620-02.png
[img1]:/assets/images/2024/07/ss-20240714-01.png
[img2]:/assets/images/2024/07/ss-20240714-02.png
[img3]:/assets/images/2024/07/ss-20240714-03.png

