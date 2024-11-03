---
toc: true
title:  "ComfyUI を docker compose で構築する"
date:   2024-09-05 09:00:00 +0900
categories: プログラム
tags:
- python
- 機械学習
- ollama
- OpenWebUI
- llm
---
[Stable Diffusion 3 Medium を実行する環境を ComfyUI で用意した手順][comfyui] では、コンテナを立ち上げて手動で ComfyUI を構築していました。

[Ollama と OpenWebUI でローカル LLM][openwebui] では、docker compose を使いましたが、これが便利だったので、ComfyUI も docker compose で構築してみます。

### 準備
[WSL2 の Docker で CUDA する][docker] を参考に、GPU を使えるように設定済みのコンテナを使います。

ComfyUI 用に以下の内容を記述したファイルを、compose.yml のファイル名で作成します。なお、デフォルトで使用されるファイル名が同じなのと、フォルダ構成が競合すると面倒なので、OpenWebUI とは別フォルダにファイルを配置するようにしています。

{% highlight yaml %}
services:
  comfyui:
    image: nvcr.io/nvidia/tensorrt:24.06-py3
    volumes:
      - ./ComfyUI:/ComfyUI
    command:
    - /bin/bash
    - -c
    - |
      apt update
      apt install -y git python3-pip libgl1-mesa-dev libglib2.0-0
      pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
      cd /ComfyUI/
      pip install -r requirements.txt
      python main.py --listen
    deploy:
      resources:
        reservations:
          devices:
          - driver: nvidia
            capabilities: [gpu]
    ports:
      - "8188:8188"
{% endhighlight %}

TensorRT 拡張を使うことも考慮して tensorrt のイメージを使用すること、8188 番ポートを転送すること、GPU を有効にするための設定を記述しています。また、`command` ディレクティブにパッケージのインストールを記述しています。この書き方だとコンテナを stop/start する度にこの処理が走ってしまいますが、気になるほどの時間はかからないのでこのようにしています。

さらに同じディレクトリに ComfyUI のリポジトリをクローンしておきます。また、同じく必須と思われる　ComfyUI-Manager のリポジトリもクローンしておきます。これは、compose.yml を作成したディレクトリで以下コマンドを実行することで可能です。

{% highlight bash %}
git clone https://github.com/comfyanonymous/ComfyUI.git
pushd ComfyUI/custom_nodes/
git clone https://github.com/ltdrdata/ComfyUI-Manager.git
popd
{% endhighlight %}

この状態で、以下のディレクトリ構成になっているはずです。

{% highlight bash %}
.
|- ComfyUI/
   |- api_Server/
   |- app/
   |- ...
   |- custom_nodes/
      |- ComfyUI-Manager/
   |- input/
   |- ...
-- compose.yml
{% endhighlight %}

### 起動
ファイル、ディレクトリの準備ができたら、以下のコマンドを実行します。
{% highlight bash %}
docker compose up
{% endhighlight %}

しばらく待つとターミナルに `Starting server / To see the GUI go to: http://0.0.0.0:8188` と表示されるので、ブラウザで URL にアクセスします。

![startup][img1]

これで WebUI を開くことができます。

![signup][img2]

モデルはコンテナ外からでも `ComfyUI/models` 以下へ配置できます。UI に反映されない場合はメニューの `Refresh` をクリックすれば読み込まれると思います。生成した画像等も `ComfyUI/output` へ出力されるので、コンテナ外からのアクセスが容易です。

docker compose 一回で WebUI の起動まで可能になるので、便利になりました。



[docker]:{% post_url 2024/2024-06-22-docker-and-cuda %}
[comfyui]:{% post_url 2024/2024-06-20-comfyui %}
[openwebui]:{% post_url 2024/2024-08-27-openwebio %}

[img1]:/assets/images/2024/09/ss-20240905-01.png
[img2]:/assets/images/2024/09/ss-20240905-02.png
