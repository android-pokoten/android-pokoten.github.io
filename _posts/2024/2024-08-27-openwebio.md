---
layout: post
title:  "Ollama と OpenWebUI でローカル LLM"
date:   2024-08-27 09:00:00 +0900
categories: プログラム
tags:
- python
- 機械学習
- ollama
- OpenWebUI
- llm
---
ローカルで LLM を動かす際に便利な、[OpenWebUI][openwebui] を使ってみます。なお、OpenWebUI はフロントエンドで、バックエンドには [Ollama][ollama] や OpenAI API を使うことができます。

そのため、すでに Ollama を使っていたり、OpenAI API のクレジットを所有していればそれらを活用できるのですが、どちらも持っていません。この場合、Ollama を含む OpenWebUI の Docker イメージを使うことで、セットアップの手間を省けます。

動画も作ってみました。ほとんど待ち時間なのであまり面白みはありませんが・・・
<iframe width="560" height="315" src="https://www.youtube.com/embed/nJPcCcLrKhg?si=8MZllINn_1n2sy-c" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


### 準備
[WSL2 の Docker で CUDA する][docker] を参考に、GPU を使えるように設定済みのコンテナを使います。

コマンドだと引数をつけ忘れたりするので、Dcker Compose で操作するようにします。以下の内容を記述したファイルを、compose.yml のファイル名で作成します。

{% highlight yaml %}
services:
  openwebui:
    image: ghcr.io/open-webui/open-webui:ollama
    volumes:
      - ./ollama:/root/.ollama
      - ./data:/app/backend/data
    deploy:
      resources:
        reservations:
          devices:
          - driver: nvidia
            capabilities: [gpu]
    ports:
      - "8080:8080"
{% endhighlight %}

イメージに「oolama」タグをつけることで、ollama も含まれます。また、8080 番ポートを転送することと、GPU を有効にするための設定も記述します。

さらに同じディレクトリに `oolama` と `data` というディレクトリを作成しておきます。内容は空で大丈夫です。このディレクトリをコンテナにマウントして、ダウンロードしたモデルや WebUI の設定をコンテナ外に保管するようにします。こうすることで、不要になったらコンテナを削除することが可能になり、かつ再度利用する際にスムーズに作業を再開することが可能になります。

この状態で、以下のディレクトリ構成になっているはずです。

{% highlight bash %}
.
|- ollama/
|- data/
-- compose.yml
{% endhighlight %}

### 起動
ファイル、ディレクトリの準備ができたら、以下のコマンドを実行します。
{% highlight bash %}
docker compose up
{% endhighlight %}

しばらく待つとターミナルに `Application startup complete.` と表示され URL が表示されるので、ブラウザで URL にアクセスします。

![startup][img1]

ログイン画面が表示されますが、初回は何もアカウントがないはずなので、「サインアップ」をクリックします。

![signup][img2]

アカウント作成画面になりますが、ローカルに保存するだけのアカウントなので、メールアドレスなどは適当で問題ありません。一応、@ がついたメールアドレス形式になっていないと登録できませんが、メールアドレスの確認なども無いので適当に入力して「アカウントを作成」をクリックします。

![signup][img3]

これで WebUI を開くことができます。

![ui][img4]

### モデルのダウンロード
初期状態だとモデルが無いので、まずはモデルをダウンロードします。UI 左上の「モデルを選択」をクリックし、検索欄に　`llama3` と入力します。すると、`Ollama.com から "llama3" をプル` と表示されるので、これをクリックします。

![model][img5]

モデルによりますが、数 GB のダウンロードを行うので、それなりの時間がかかります。

![downloading][img6]

ダウンロードが完了すると、`モデル "llama3" が正常にダウンロードされました` と表示されます。

![complete][img7]

今回は llama3 をダウンロードしましたが、他にどんなモデルがあるかは以下のサイトで探すことができます。

[ollama.com/models][models]

### 生成
モデルのダウンロードが終わったら、「モデルを選択」からダウンロード済みのモデルを選択します。

![select][img8]

この状態で、チャットが可能になっています。

![chat][img9]

### モデル(キャラ付け?)
open-webui の Community ページでも、さまざまなモデルをダウンロードすることが可能です。

[OpenWebUI Community][community]

なお、こちらは Ollama.com のモデルに対してカスタマイズしたような形のようです。そのため、動作にはベースモデルとして Ollama.com からモデルをダウンロードする必要がありそうです。

なお、同じように自分でカスタマイズすることも可能です。OpenWebUI の「ワークスペース」タブでできそうなのですが、まだそこまで手を付けられていません。


[openwebui]:https://github.com/open-webui/open-webui
[community]:https://openwebui.com/
[ollama]:https://ollama.com/
[models]:https://ollama.com/library

[docker]:{% post_url 2024/2024-06-22-docker-and-cuda %}

[img1]:/assets/images/2024/08/ss-20240828-01.png
[img2]:/assets/images/2024/08/ss-20240828-02.png
[img3]:/assets/images/2024/08/ss-20240828-03.png
[img4]:/assets/images/2024/08/ss-20240828-04.png
[img5]:/assets/images/2024/08/ss-20240828-05.png
[img6]:/assets/images/2024/08/ss-20240828-06.png
[img7]:/assets/images/2024/08/ss-20240828-07.png
[img8]:/assets/images/2024/08/ss-20240828-08.png
[img9]:/assets/images/2024/08/ss-20240828-09.png
