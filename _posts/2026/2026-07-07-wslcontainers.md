---
toc: true
title:  "WSL Containers でコンテナ実行"
date:   2026-07-07 09:00:00 +0900
categories: プログラム
tags:
- Windows11
- WSL
- WSLcontainers
---
2026年6月に行われた Microsoft BUILD で発表された、WSL Containers を試してみました。

# 前提
Windows11 で、すでに WSL は使用している環境です。Windows Insider には参加していませんが、更新プログラムは最新化しています。

# インストール
2026年7月時点では、プレビュー版の WSL をインストールする必要がありそうです。これは PowerShell を開いて以下コマンドを実行するとインストールできます。

``` bash
wsl --update --pre-release
```
{% include thumb.html path="2026/07/ss-20260707-01.png" alt="インストール" %}

このようにバージョン 2.9.3 に更新されると、wslc コマンドもインストールされます。ただ、この時点では実行ファイルが認識されないので、新たに PowerShell  プロンプトを開き、

``` bash
wsl --help
```

と実行するとコマンドが実行され、ヘルプが表示されます。

{% include thumb.html path="2026/07/ss-20260707-02.png" alt="wslc" %}

コマンド体系は Docker や Podman と変わらないようですね。

# コンテナ起動
## 通常のコンテナ
以下のように PowerShell プロンプトから実行することが可能です。

``` bash
wslc run --rm -it python:3.12-slim bash
```

{% include thumb.html path="2026/07/ss-20260707-03.png" alt="wslc run" %}

自動的にコンテナイメージを pull しています。pull が終わるとコンテナ内のシェルが起動します。

{% include thumb.html path="2026/07/ss-20260707-04.png" alt="コンテナで Python を実行" %}

## GPU パススルー
wslc run のヘルプを確認すると、`--gpus` というオプションがあります。

{% include thumb.html path="2026/07/ss-20260707-05.png" alt="wslc run -?" %}

では、GeForce RTX 2070 を乗せた PC でこのオプションを付けてコンテナを起動してみます。

``` bash
wslc run --rm --gpus all ubuntu nvidia-smi
```

{% include thumb.html path="2026/07/ss-20260707-06.png" alt="nvidia-smi" %}

特段何も設定していませんが、このようにコンテナから GPU が認識されています。これは Linux コンテナを実行しやすくなりますね。

# まとめ
これでコンテナ実行環境をあれこれ工夫しなくても、コマンドから実行しやすくなります。あとは git コマンドが標準で用意されていれば・・・

あと、compose はまだサポートされていないようです。そのうち実装されるようにも見えます。
