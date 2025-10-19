---
toc: true
title:  "Podman で GPU が使えない"
date:   2025-10-19 00:00:08 +0900
categories: トラブル
tags:
- Windows
- WSL
- podman
- CUDA
- コンテナ
---
### 事象
[WSL2 の Podman で CUDA する][prev] で作った環境で、コンテナ内から GPU にアクセスできなくなっていることに気が付きました。OpenWeb UI を起動すると、GPU を使ってくれず CPU のみで処理をしていますし、ComfyUI にいたってはコンテナが起動すらしません。以前は GPU が使えたはずのそれらのコンテナの中から `nvidia-smi` コマンドを実行してみると、

`Failed to initialize NVML: N/A`

と表示され、GPU 情報が表示されませんでした。

### 対策
この事象が出る前に、Windows11 側の NVIDIA App よりドライバー更新のお知らせがあったので、更新をしていました。この場合、ドライバーのパスなどが変更される (ことがある？) ため、`sudo nvidia-ctk cdi generate --output /etc/cdi/nvidia.yaml` を再実行し、正しいパスで定義しなおす必要がありました。

なお、Dokcer の場合は GPU を認識させる仕組みが違うため、ドライバーを更新してもコンテナ内から GPU にアクセスすることが可能でした。

### 詳細
事象に気が付いた後に、[WSL2 の Podman で CUDA する][prev] の記事で動作確認用に実行した、

``` bash
podman run --rm \
--device nvidia.com/gpu=all \
--security-opt=label=disable \
ubuntu nvidia-smi -L
```

を実行すると、以下のような結果になっていました。

![init fail][img01]

このとき、同じ PC 内の別の WSL ディストリビューション (Docker を残したままにしていた環境) では、以下のように GPU の情報が出力されていたので、WSL 側へ認識できていることは間違いなさそうです。

![docker][img02]

なお、Podman が入っている WSL 環境で、コンテナの外で `nvidia-smi` コマンドを実行すると GPU の情報が表示されたので、Podman で GPU の情報がうまくコンテナ内から参照できない状態のように見えます。

nvidia-container-toolkit パッケージの再インストールもしてみましたが、結果は変わりませんでした。NVIDIA のドライバーのバージョンアップでエラーになる、という症状は検索して見つけたのですが、その場合はバージョンが違うとかのエラーメッセージのようだったので、今回のエラーとは違うのかな、と思っていました。

しかし、`nvidia-ctk` コマンドで出力する `/etc/cdi/nvidia.yaml` ファイルの内容を参照してみると、以下のように `nvmdi.inf_amd64_8ae620a6c455be0a` を参照しています。 

![nvidia.yaml][img03]

このパスを見てみると、`nvmdi.inf_amd64_8ae620a6c455be0a` と `nvmdi.inf_amd64_f088ae99b5a2f5fd` の 2 つのディレクトリがあり、明らかに後者が最近更新されたパスのようです。

![driver timestamp][img04]

そのため、`sudo nvidia-ctk cdi generate --output /etc/cdi/nvidia.yaml` を実行して、ファイルを再作成してみました。

![cdi generate][img05]

出力を見ると、タイムスタンプが新しかった `nvmdi.inf_amd64_f088ae99b5a2f5fd` を認識しているようです。この状態で動作確認してみると、

![gpu enabled][img06]

正常に `nvidia-smi` コマンドが実行でき、GPU 情報も取得できるようになりました。NVIDIA のドライバーを更新したときは、Podman は CDI の定義をし直したほうがよさそうです。


[prev]:{% post_url 2025/2025-07-27-podman-cuda %}

[img01]:/assets/images/2025/10/ss-20251019-01.png
[img02]:/assets/images/2025/10/ss-20251019-02.png
[img03]:/assets/images/2025/10/ss-20251019-03.png
[img04]:/assets/images/2025/10/ss-20251019-04.png
[img05]:/assets/images/2025/10/ss-20251019-05.png
[img06]:/assets/images/2025/10/ss-20251019-06.png
