---
toc: true
title:  "Windows11 の WSL2 (WSLg) で 日本語入力 [2024/8/12 一部修正]"
date:   2024-07-02 09:00:00 +0900
categories: プログラム
tags:
- Windows11
- WSL
- WSLg
- fcitx
---
ChromeOS の Linux コンテナで GUI アプリを実行した場合、ChromeOS 側の日本語入力をそのまま利用することが可能でした。WSL で実行する Linux の場合、GUI アプリには Windows の IME が反応してくれません。そのため、WSL の Linux に対して日本語入力環境を構成する必要があります。Windows 側とは分離した環境を WSL 内で用意できるといえばメリットがありそうですが、Ubuntu のバージョンなども相まって、決定版といえる手順がないため、手間がかかりそうに見えます。

とりあえず自分用に、2024年07月時点で、Windows 11 に Ubuntu 24.04 を導入した場合の手順です。それ以外のバージョンでは手順やコマンドが異なる可能性があります。うまくいかない点に気が付いたら、順次修正していこうと思います。

### 必要なパッケージのインストール
最初に、念のため ibus 関連のパッケージを削除します。おそらく何もインストールされていないと思いますが、念のためです。

{% highlight bash %}
sudo apt remove --purge ibus*
{% endhighlight %}

続いて、fcitx をインストールします。

{% highlight bash %}
sudo apt install -y fcitx5-mozc
{% endhighlight %}


### 設定

ホームディレクトリの `.profile` ファイルの末尾に以下内容を追記します。

{% highlight bash %}
export GTK_IM_MODULE=fcitx5
export QT_IM_MODULE=fcitx5
export XMODIFIERS=@im=fcitx5
export INPUT_METHOD=fcitx5
export DefaultIMModule=fcitx5
if [ $SHLVL = 1 ] ; then
  (fcitx5 --disable=wayland -d --verbose '*'=0 &)
fi
{% endhighlight %}

その後、`im-config` を実行して入力メソッドを fcitx に設定します。
{% highlight bash %}
im-config -n fcitx5
{% endhighlight %}

設定ツールを起動します。

{% highlight bash %}
fcitx5-configtool
{% endhighlight %}

初期状態では「現在の入力メソッド」欄に「キーボード - English」や「キーボード - Japanese」があるので、それらを削除して、「有効な入力メソッド」欄から「Mozc」を追加します。以下のような画面になれば OK です。

![configtool][img1]

※当初、WSL を再起動してから設定ツールを起動していましたが、その順番だと設定ツールの内容が初期化されてしまい、毎回設定ツールで Mozc を追加しないと日本語入力ができない状態でした。これは、Fcitx5 の仕様上、デーモン (`-d` をつけて起動した状態) だと設定ファイルの内容が保存されないためのようです。

PowerShell のプロンプトから、WSL を再起動します。

{% highlight bash %}
wsl -t Ubuntu-24.04
{% endhighlight %}

この状態で、WSL の Ubuntu 24.04 にインストールした gedit や Firefox などの GUI アプリを起動すると、mozc による日本語入力が可能になります。ただ、ツールバーが表示されないので、現在のモードが日本語入力なのか、直接入力なのかキーを打ってみないとわからないのが不便なところです。

#### 2024/8/12 一部修正
fcitx5-configtool を実行するタイミングを変更


[img1]:/assets/images/2024/07/ss-20240702.png
