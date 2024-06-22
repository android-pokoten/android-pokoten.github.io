---
layout: post
title:  "WSL2 の Docker で CUDA する"
date:   2024-06-22 00:00:00 +0900
categories: プログラム
tags:
- python
- 機械学習
- WSL
- Docker
- CUDA
---
生成 AI 等をローカルで実行しようとすると、いろいろな依存関係の競合が発生します。Python だけでいえば venv や anaconda といったツールを使うことである程度回避できますが、それだけでは吸収できないような場合もあります。

そんな時は、コンテナを使ってその中に環境を閉じ込めてしまうことで、他への影響が少なくなります。ただ、コンテナの中からハードウェア (CUDA) をたたくためには、少し設定が必要になります。

今回は Windows 11 をベースに、WSL2 の Ubuntu で Docker を使って環境を用意してみます。

### Windows の設定
GeForce のドライバをインストールします。常に最新にしておく必要もないですが、セットアップの時点で最新のドライバにしておいた方がいいと思います。(特に根拠はありません)

WSL2 を有効にして、Ubuntu をインストールします。管理者権限で PowerShell を開いて、以下コマンドを実行したら Windows を再起動します。
{% highlight bash %}
wsl --install
{% endhighlight %}

再起動してきたら再度 PowerShell を開き (管理者として実行はしません)、以下コマンドを実行して Ubuntu 24.04 をインストールします。
{% highlight bash %}
wsl --install -d Ubuntu-24.04
{% endhighlight %}

Windows 側には GeForce のドライバを入れるだけで、Python や CUDA をインストールする必要はありません。(Windows でそれらを使いたい場合は別ですが)

### Ubuntu の設定
NVIDIA のサイトにある [Installing the NVIDIA Container Toolkit][installing] に沿って実施します。なお、WSL の Ubuntu に、GeForce のドライバ等を入れる必要はありません。

{% highlight bash %}
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
{% endhighlight %}

そのうえで、nvidia-container-toolkit をインストールします。

{% highlight bash %}
sudo apt update
sudo apt-get install -y nvidia-container-toolkit
{% endhighlight %}

nvidia-container-toolkit をインストールしたら、Docker をインストールします。これは [Docker のサイト][docker] の手順で行います。
Docker Desktop は使わずに、Ubuntu の apt でインストールします。

{% highlight bash %}
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
{% endhighlight %}

この状態だとコンテナの操作 (docker コマンドの実行) に root 権限が必要になってしまうので、rootless で動作するように設定します。以下の手順に沿って実施します。

[Run the Docker daemon as a non-root user (Rootless mode)][rootless]

まずはシステム権限で動作する docker サービスを無効にします。

{% highlight bash %}
sudo systemctl disable --now docker.service docker.socket
sudo rm /var/run/docker.sock
{% endhighlight %}

必要なパッケージをインストールします。

{% highlight bash %}
sudo apt install -y uidmap
{% endhighlight %}

そのうえで、rootless を設定するスクリプトを実行します。このスクリプトは sudo をつけずに実行する必要があります。

{% highlight bash %}
dockerd-rootless-setuptool.sh install
{% endhighlight %}

スクリプトが正常に完了すると、以下のような画面が表示されます。この状態で、rootless で docker コマンドが実行可能です。

![output][img1]

画面に出力されている通り、以下の 2 行を ~/.bashrc の末尾に追記しておくようにします。

※2 行目の「1002」は実行したユーザーの UID に応じて変わると思います

{% highlight bash %}
export PATH=/usr/bin:$PATH
export DOCKER_HOST=unix:///run/user/1002//docker.sock
{% endhighlight %}

これで docker のインストールは終わったので、再度 [Installing the NVIDIA Container Toolkit][installing] の手順に戻ります。Configuration 項の rootless の手順を実施します。

{% highlight bash %}
nvidia-ctk runtime configure --runtime=docker --config=$HOME/.config/docker/daemon.json
systemctl --user restart docker
sudo nvidia-ctk config --set nvidia-container-cli.no-cgroups --in-place
{% endhighlight %}

これで Ubuntu の設定も完了です。

### コンテナ実行
[サンプルのコンテナ][sample] を実行してみましょう。

{% highlight bash %}
docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
{% endhighlight %}

以下のように GPU が表示されていれば、コンテナから GPU を参照できてます。

![output][img2]


以上で準備は完了です。この環境で、`--gpus all` オプションをつけてコンテナを起動すれば、コンテナ内から GPU を使えるので、システムワイドに環境を分離することがやりやすくなりました。

このブログ内でも、GPU を使えるようにした Docker を使うシーンが多々発生しますが、その場合はこのように構成した WSL2 を使っています。


[installing]:https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html
[docker]:https://docs.docker.com/engine/install/ubuntu/
[rootless]:https://docs.docker.com/engine/security/rootless/
[sample]:https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/sample-workload.html

[img1]:/assets/images/2024/06/ss-20240621-01.png
[img2]:/assets/images/2024/06/ss-20240621-02.png
