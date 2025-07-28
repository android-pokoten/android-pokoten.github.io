---
toc: true
title:  "WSL2 の Podman で CUDA する"
date:   2025-07-27 00:00:00 +0900
categories: プログラム
tags:
- python
- 機械学習
- WSL
- podman
- CUDA
- VScode
- コンテナ
---
[WSL2 の Docker で CUDA する][docker] でコンテナから GPU を使う環境を作りました。今度は Podman で構成してみます。基本的な流れは Docker の場合と同じですが、Podman 特有の手順もありました。

# WSL で Ubuntu をインストール
こちらは普通に PowerShell から `wsl --install` コマンドを実行して Ubuntu をインストールします。今回はバージョン 24.04 を使用しました。

``` bash
wsl --install -d Ubuntu-24.04
```

![wsl][img01]

# Ubuntu に Nvidia Container Toolkit をインストール
まずは Nvidia Container Toolkit をインストールします。先にリポジトリを追加します。

``` bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```
![container toolkit][img02]

パッケージ情報を更新して、`nvidia-container-toolkit` パッケージをインストールします。

``` bash
sudo apt update
sudo apt-get install -y nvidia-container-toolkit
```

![install toolkit][img03]


# Ubuntu に Podman をインストール
apt コマンドを使ってインストールします。`podman` と `podman-compose` パッケージをインストールしました。

![apt podman][img04]

![apt podman-compose][img05]

インストールすると podman がサービスで動作していますが、rootless で動作させるため、以下のコマンドを実行してユーザー権限で起動するように変更します。

``` bash
sudo systemctl stop podman.socket
sudo systemctl disable podman.socket
systemctl --user enable podman.socket
systemctl --user start podman.socket
```

![systemctl][img06]


# Container Device Interface (CDI) を設定
Docker と Podman が異なるところで、GPU をコンテナ内から使用するための設定がありました。Podman の場合は CDI というものを利用するようです。`nvidia-ctk` コマンドを使ってデバイス情報を書き出しておき、コンテナ作成時にデバイスを割り当てることでコンテナ内からも利用できるようになります。

## デバイス情報の書き出し
[Support for Container Device Interface][cdi] に手順も含めて情報があります。デバイス情報を書き出すには、以下のコマンドを実行します。

``` bash
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

![cdi generate][img07]

念のため確認します。以下のコマンドを実行して、GPU がリストアップされていれば大丈夫です。

``` bash
nvidia-ctk cdi list
```

![cdi list][img08]

# コンテナ起動
## run
podman コマンドでコンテナを作成するときは、`--device nvidia.com/gpu=all` を引数にすれば GPU を使えます。引数は、CDI のリストに表示された文字列を使えばよさそうです。

``` bash
podman run --rm \
--device nvidia.com/gpu=all \
--security-opt=label=disable \
ubuntu nvidia-smi -L
```

![podman run][img09]

このように GPU 名が出力されれば OK です。


## compose
compose.yml ファイルは以下のように記述します。

``` yaml
serivce:
    test-gpu:
        image: ubuntu
        devices:
            - "nvidia.com/gpu=all"
        security_opt:
            - "label=disable"
        command: nvidia-smi
```

compose.yml ファイルがあるディレクトリで `podman-compose up` を実行すると、GPU の情報が出力されます。

![compose1][img10]
![compose2][img11]

※`podman-docker` パッケージをインストールしていると、`podman compose` の構文が使えますが、こちらの実行方法だと CDI が認識されないのか、GPU の記述でエラーになります。そのため、`podman-compose` コマンドを使う必要がありました。

これで podman でも問題なく GPU を使った処理ができそうです。

# 背景
![environment][img12]

ちょっとわかりにくいのですが、手元では T480 を使っています。GeForce 2070 を乗せたデスクトップは別の部屋にあり、リモート接続で使用しています。Windows を使う場合は RDP で、Ubuntu を使うときは SSH で接続しています。Ubuntu に接続するときは、その上のコンテナを使って何かやる場合が多いので、VScode のリモート接続 (Remote - SSH) 拡張を使って接続することが多いです。また、T480 でもコンテナを使っていて、ブログの更新用の Jekyll 環境など、軽めの処理はこちらでやっていました。

これらの操作には、どうせコードやテキストを書くので、VScode から接続していました。ただ、デスクトップと T480 は環境を作ったタイミングが違っていて、コンテナ管理が Docker と Podman で異なるツールを使う状態になっていました。

この場合、VScode の Dev Containers 拡張で不便なことがあり、「開発コンテナ」一覧が表示できない場合があります。これは設定の `Dev Containers: Docker Path` 設定によるものなのですが、この設定の初期値は `docker` になっています。この場合、Docker を使うデスクトップのコンテナは表示できるのですが、Podman を使う T480 のコンテナは表示できません。 `Dev Containers: Docker Path` 設定を `podman` にすると、T480 のコンテナは表示できてデスクトップのコンテナが表示できなくなります。

そして、設定画面にもあるとおり、この `Dev Containers: Docker Path` 設定はワークスペースごとではなく、すべてのプロファイルで共通の設定らしく、Docker か Podman かどちらかを選ばないといけない状態でした。

設定を一か所変更すればいいだけ、ですが、やはり面倒なのでどちらかに寄せたいと思っていましたが、Docker を rootless で動かそうとするとそこそこ面倒だった記憶のほうが強いので、今回は Podman で統一するほうへ設定してみました。



[cdi]:https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/cdi-support.html

[docker]:{% post_url 2024/2024-06-22-docker-and-cuda %}

[img01]:/assets/images/2025/07/ss-20250727-01.png
[img02]:/assets/images/2025/07/ss-20250727-02.png
[img03]:/assets/images/2025/07/ss-20250727-03.png
[img04]:/assets/images/2025/07/ss-20250727-04.png
[img05]:/assets/images/2025/07/ss-20250727-05.png
[img06]:/assets/images/2025/07/ss-20250727-06.png
[img07]:/assets/images/2025/07/ss-20250727-07.png
[img08]:/assets/images/2025/07/ss-20250727-08.png
[img09]:/assets/images/2025/07/ss-20250727-09.png
[img10]:/assets/images/2025/07/ss-20250727-10.png
[img11]:/assets/images/2025/07/ss-20250727-11.png
[img12]:/assets/images/2025/07/ss-20250727-12.png


