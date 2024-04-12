---
layout: post
title:  "Gentoo Linux を WSL に導入してみた"
date:   2024-04-12 09:00:00 +0900
categories: プログラム
tags:
- python
- 機械学習
---
WSL は各種ディストリビューションが用意されているので、コマンド 1 つで簡単にそれらをインストールすることができます。WSL ではほぼ標準といえる Ubuntu を使うことが多いですが、WSL が実装される前は実機に [Gentoo Linux][home] を入れて使っていました。portage を使った柔軟なパッケージ管理が便利でしたが、インストールにも非常に柔軟性があり、よほどメインで使うつもりでないとインストールまで手が出ない感じでした。

しかし、WSL に Gentoo をインストールする手順が、Gentoo 公式ドキュメントにあることを知りました。

[Gentoo in WSL][gentoo]

これはぜひ試してみましょう。


## ダウンロード
手順は基本的に前述の [Gentoo in WSL][gentoo] に沿って行けば問題ありません。まずは Stage3 ファイルをダウンロードします。

![ダウンロード][img01]

Windows PC なので、arm ではなく amd64 の方を使います。openrc を使いましたが、使いたいアプリケーションが systemd 必須とかあれば systemd の Stage3 ファイルでも手順はほぼ同じはずです。

![展開][img02]

ダウンロードしたファイルは、tar.xz 拡張子になっています。xz で圧縮されていると WSL にインポートできないので展開する必要があります。Windows11 はエクスプローラー上で xz も展開できるようになっていますが、この機能は xz だけ展開ということができません。つまり、tar も展開してしまいます。しかし、WSL でインポートするには tar になっている必要があるので、ここは 7zip などのユーティリティを使うか、別途 Ubuntu 等を用意して xz コマンドで展開しておきます。


## インポート
Windows Terminal などで PowerShell のプロンプトを開き、以下コマンドを実行します。

{% highlight bash %}
PS >wsl --import Gentoo ${ENV:LocalAppdata}\WSL\Gentoo\ .\stage3-amd64-openrc-20211121T170545Z.tar --version 2
{% endhighlight %}

あとは、以下コマンドで Gentoo Linux に入れます。
{% highlight bash %}
PS >wsl -d Gentoo
{% endhighlight %}

簡単ですね。ルートファイルシステムを固めた tar ファイルがあれば WSL にインポートできるので、もともとそういう形式で配布していた Gentoo Linux は、意外と相性が良かったようです。


## インストール後設定
インストール後も、[Gentoo in WSL][gentoo] の Basic system and configuration の項に沿って設定していけば、ほとんど迷うことなく設定できると思います。ただ、ここに記載のない Windows Terminal に登録する方法を紹介します。

![ターミナル][img03]

Windows Terminal の設定画面から新しいプロファイルを追加し、上の画面のように設定します。コマンドラインは `-d` オプションで Gentoo を指定しています。ws.conf でデフォルトのユーザーを指定していますが、ここのコマンドラインオプション(`-u`)でユーザーを指定することも可能です。

また、開始ディレクトリは、指定しないと Windows のユーザープロファイルのディレクトリになってしまいます。Gentoo 側のホームディレクトリにしたい場合は、`\\wsl.localhost\Gentoo\home\xxx` のように `\\wsl.localhost` を使うことで、WSL 内のディレクトリを指定可能でした。


実機に入れると、ブートローダーの設定にミスって Live ISO から再度 chroot しなくてはいけなかったり、カーネルオプションが不足していて / をマウントできない、デバイスが認識しないなど、無事にログインできるまでで一苦労ですが、そういうところを気にせずに、Gentoo Linux がインストールできることを知りました。


[gentoo]:https://wiki.gentoo.org/wiki/Gentoo_in_WSL
[home]:https://www.gentoo.org/

[img01]:/assets/images/2024/04/ss-20240412-01.png
[img02]:/assets/images/2024/04/ss-20240412-02.png
[img03]:/assets/images/2024/04/ss-20240412-03.png
